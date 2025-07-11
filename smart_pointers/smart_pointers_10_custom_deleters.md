#### **1. Определение**

Делитеры (deleters) в умных указателях нужны для **гибкого управления освобождением ресурсов**. Они позволяют:  
1. **Использовать нестандартные способы удаления** (например, `free()`, `fclose()`, деаллокаторы из сторонних библиотек).  
2. **Вызывать кастомные методы очистки** (например, `Release()` для COM-объектов).  
3. **Работать с памятью, выделенной не через `new`** (например, аллокация через `mmap`, shared memory).  

#### **2. Примеры использования**:  
```cpp
// 1. Для FILE* (нужен fclose вместо delete)
std::unique_ptr<FILE, decltype(&fclose)> filePtr(fopen("test.txt", "r"), &fclose);

// 2. Для malloc/free
std::unique_ptr<int, decltype(&free)> mallocPtr((int*)malloc(sizeof(int)), &free);

// 3. Кастомный deleter (лямбда)
auto deleter = [](int* p) { std::cout << "Удаляем " << p << "\n"; delete p; };
std::unique_ptr<int, decltype(deleter)> customPtr(new int(42), deleter);
```  

#### **3. Когда в умных указателях не вызывается `delete`?**  
Умные указатели **не всегда** используют `delete`. Вот случаи, когда `delete` не вызывается:  
**1. Используется кастомный deleter**  
Если передан свой deleter, то `delete` **не вызывается** — вместо него выполняется deleter.  
```cpp
// Здесь delete не вызывается — будет fclose
std::unique_ptr<FILE, decltype(&fclose)> file(fopen("file.txt", "r"), &fclose);
```  

**2. Объект создан не через `new`**  
Если память выделена не через `new` (например, стек, `malloc`, пул объектов), то:  
- **Для `unique_ptr`/`shared_ptr`** — нужно передать кастомный deleter.  
- **Иначе — UB** (неопределённое поведение).  
```cpp
int x = 10;
std::unique_ptr<int> ptr(&x);  // Ошибка: вызовет delete для стека → UB!

// Правильно (если очень нужно):
std::unique_ptr<int, void(*)(int*)> ptr(&x, [](int*){});  // Ничего не удаляет
```  

**3. Указатель `nullptr`**  
Если `ptr == nullptr`, то deleter **не вызывается**.  
```cpp
std::shared_ptr<int> ptr(nullptr);  // Ничего не удалит
```  

**4. Передача владения (`release`, `move`)**  
Если владение передано (например, через `release()`), то `delete` не вызывается.  
```cpp
auto ptr = std::make_unique<int>(42);
int* raw = ptr.release();  // Теперь удалять вручную!
```  

**5. Циклические зависимости (`shared_ptr`)**
Если есть циклы из `shared_ptr`, объекты **никогда не удалятся** (утечка памяти).  
```cpp
struct Node {
    std::shared_ptr<Node> next;
};
auto a = std::make_shared<Node>();
auto b = std::make_shared<Node>();
a->next = b;
b->next = a;  // Утечка: никто не удалится!
```  
**Решение**: использовать `weak_ptr`.  

**Идеальный ответ:**  
*"Делитеры (deleters) в умных указателях обеспечивают **гибкое управление освобождением ресурсов**, позволяя заменять стандартный `delete` на кастомные операции (например, `fclose` для файлов, `free` для `malloc`, или методы сторонних библиотек). `delete` не вызывается в следующих случаях: (1) при использовании кастомного делитера, (2) если объект создан не через `new` (стек, пулы памяти — требует ручной настройки делитера), (3) для `nullptr`, (4) при передаче владения (`release()`), (5) при циклических зависимостях `shared_ptr` (утечка памяти).
**Пример:** `unique_ptr` с `fclose` для файлового дескриптора.
**Вывод:** делитеры расширяют сферу применения умных указателей, но требуют аккуратного использования при нестандартном управлении памятью."*