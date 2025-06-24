#### **1. Простейший `UniquePtr` (аналог `std::unique_ptr`)**
```cpp
template <typename T>
class UniquePtr {
    T* ptr;
    
public:
    // Конструктор, принимающий сырой указатель
    explicit UniquePtr(T* p = nullptr) : ptr(p) {}
    
    // Запрет копирования
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
    
    // Поддержка перемещения
    UniquePtr(UniquePtr&& other) noexcept : ptr(other.ptr) {
        other.ptr = nullptr;
    }
    
    UniquePtr& operator=(UniquePtr&& other) noexcept {
        if (this != &other) {
            delete ptr;
            ptr = other.ptr;
            other.ptr = nullptr;
        }
        return *this;
    }
    
    // Деструктор
    ~UniquePtr() {
        delete ptr;
    }
    
    // Операторы доступа
    T& operator*() const { return *ptr; }
    T* operator->() const { return ptr; }
    T* get() const { return ptr; }
    
    // Освобождение владения
    T* release() {
        T* p = ptr;
        ptr = nullptr;
        return p;
    }
    
    // Сброс указателя
    void reset(T* p = nullptr) {
        delete ptr;
        ptr = p;
    }
    
    // Проверка на null
    explicit operator bool() const { return ptr != nullptr; }
};
```
Пример использования:
```cpp
UniquePtr<int> ptr(new int(42));
std::cout << *ptr << std::endl; // 42
```

#### **2. Более сложный `SharedPtr` (аналог `std::shared_ptr`)**
```cpp
template <typename T>
class SharedPtr {
    T* ptr;
    size_t* count; // Счетчик ссылок
    
public:
    // Конструктор
    explicit SharedPtr(T* p = nullptr) : ptr(p), count(new size_t(1)) {}
    
    // Копирующий конструктор
    SharedPtr(const SharedPtr& other) : ptr(other.ptr), count(other.count) {
        ++(*count);
    }
    
    // Оператор присваивания копированием
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != &other) {
            release();
            ptr = other.ptr;
            count = other.count;
            ++(*count);
        }
        return *this;
    }
    
    // Деструктор
    ~SharedPtr() {
        release();
    }
    
    // Операторы доступа
    T& operator*() const { return *ptr; }
    T* operator->() const { return ptr; }
    T* get() const { return ptr; }
    
    // Количество владельцев
    size_t use_count() const { return *count; }
    
private:
    void release() {
        if (--(*count) == 0) {
            delete ptr;
            delete count;
        }
    }
};
```
Пример использования:
```cpp
SharedPtr<int> ptr1(new int(42));
{
    SharedPtr<int> ptr2 = ptr1;
    std::cout << *ptr2 << " (count: " << ptr2.use_count() << ")\n"; // 42 (count: 2)
}
std::cout << *ptr1 << " (count: " << ptr1.use_count() << ")\n"; // 42 (count: 1)
```

#### **3. Добавление `WeakPtr` и контрольного блока**
Для полной реализации `shared_ptr/weak_ptr` нужно добавить:
1. **Контрольный блок** (хранит счетчики shared и weak)
2. **Класс WeakPtr** (не увеличивает счетчик shared)
```cpp
template <typename T>
class ControlBlock {
    T* ptr;
    size_t shared_count;
    size_t weak_count;
    // ...
};

template <typename T>
class WeakPtr {
    // Не увеличивает shared_count, но проверяет expired()
};
```

#### **4. Поддержка пользовательских делитеров**
```cpp
template <typename T, typename Deleter = std::default_delete<T>>
class UniquePtrWithDeleter {
    T* ptr;
    Deleter deleter;
    
public:
    // ...
    ~UniquePtrWithDeleter() {
        deleter(ptr);
    }
};
```

#### **5. Поддержка массивов**
```cpp
template <typename T>
class UniquePtr<T[]> {
    T* ptr;
    
public:
    // ...
    ~UniquePtr() {
        delete[] ptr;
    }
    
    T& operator[](size_t index) {
        return ptr[index];
    }
};
```

**Ключевые моменты реализации:**
1. **Управление временем жизни**:
   - `unique_ptr` - эксклюзивное владение
   - `shared_ptr` - подсчет ссылок
   - `weak_ptr` - не влияет на время жизни
2. **Безопасность исключений**:
   - Гарантии, что ресурсы будут освобождены
3. **Правило пяти**:
   - Деструктор
   - Копирующие операции
   - Перемещающие операции
4. **Оптимизации**:
   - Для `shared_ptr` можно объединить объект и контрольный блок (как в `make_shared`)

**Идеальный ответ:**  
*"Для реализации собственного умного указателя необходимо:*  
1. ***Для `UniquePtr`** (эксклюзивное владение):*  
   - *Запретить копирование (удалить копирующие конструкторы)*  
   - *Реализовать семантику перемещения (`move`-операции)*  
   - *Освобождать ресурсы в деструкторе через `delete` или кастомный делитер*  
   - *Добавить операторы `*`, `->` и методы `get()`, `reset()`, `release()`*  

2. ***Для `SharedPtr`** (разделяемое владение):*  
   - *Ввести счетчик ссылок (обычно `size_t*`)*
   - *Увеличивать счетчик при копировании*
   - *Уменьшать счетчик в деструкторе (удалять объект при `count == 0`)*
   - *Реализовать безопасное присваивание с проверкой на самоприсваивание*

3. ***Дополнительно**:*  
   - *Для `WeakPtr` нужен отдельный счетчик слабых ссылок*  
   - *Поддержка массивов требует специализации с `delete[]` и `operator[]`*  
   - *Кастомные делитеры добавляются через шаблонный параметр"*