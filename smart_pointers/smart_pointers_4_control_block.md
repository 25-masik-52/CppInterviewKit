#### **1. Что такое Control Block?**  
**Control Block** — это внутренняя структура, которая хранит:  
1. **Счётчик сильных ссылок (`strong refs`)** — количество `shared_ptr`, владеющих объектом.  
   - Когда `strong refs == 0`, объект уничтожается (вызывается deleter).  
2. **Счётчик слабых ссылок (`weak refs`)** — количество `weak_ptr`, наблюдающих за объектом.  
   - Когда `weak refs == 0`, уничтожается сам Control Block.  
3. **Указатель на управляемый объект** (`T*`).  
4. **Deleter** (если был передан в `shared_ptr`).  
5. **Allocator** (если используется кастомный аллокатор).  

**Где находится Control Block?**  
- Если `shared_ptr` создан через **`make_shared`** → Control Block и объект создаются в **одном выделении памяти** (оптимизация).  
- Если `shared_ptr` создан через **конструктор от `new T`** → Control Block выделяется отдельно.  

#### **2. Как устроен Control Block в памяти?**  
Примерная структура (псевдокод):  
```cpp
struct ControlBlock {
    std::atomic<size_t> strong_refs;  // Количество shared_ptr
    std::atomic<size_t> weak_refs;    // Количество weak_ptr
    T* ptr;                          // Указатель на объект
    Deleter deleter;                  // Делетер (по умолчанию — delete)
    Allocator allocator;              // Аллокатор (опционально)
};
```

**Схема выделения памяти**  

**Случай 1: `make_shared`**  
```  
[ Control Block | Object ]  ← Одно выделение памяти  
```  
✅ **Плюсы**:  
- Лучшая производительность (1 выделение вместо 2).  
- Локализация данных (меньше cache misses).  

**Случай 2: `shared_ptr(new T)`**  
```  
[ Control Block ]  ← Выделение 1  
[ Object ]        ← Выделение 2  
```  
❌ **Минусы**:  
- Два отдельных выделения памяти.  
- Меньшая эффективность.  

#### **3. Как `weak_ptr` взаимодействует с Control Block?**  
1. **При создании `weak_ptr`**:  
   - Увеличивается `weak_refs` в Control Block.  
   - `strong_refs` не меняется.  
2. **При вызове `.lock()`**:  
   - Если `strong_refs > 0`, создаётся новый `shared_ptr` (и `strong_refs++`).  
   - Если `strong_refs == 0`, возвращается `nullptr`.  
3. **При уничтожении `weak_ptr`**:  
   - Уменьшается `weak_refs`.  
   - Если `weak_refs == 0` **и** `strong_refs == 0`, Control Block удаляется.  

#### **4. Жизненный цикл объекта и Control Block**  
Рассмотрим на примере:  
```cpp
auto shared = std::make_shared<int>(42);  // strong_refs=1, weak_refs=0
std::weak_ptr<int> weak = shared;         // strong_refs=1, weak_refs=1
shared.reset();                           // strong_refs=0, weak_refs=1 → объект удалён!
auto locked = weak.lock();                // locked == nullptr
weak.reset();                             // weak_refs=0 → Control Block удалён
```  
**Важно**:  
- Объект удаляется **только когда `strong_refs == 0`**.  
- Control Block удаляется **только когда `strong_refs + weak_refs == 0`**.  

#### **5. Многопоточная безопасность**  
- **Счётчики (`strong_refs`, `weak_refs`)** — атомарные (`std::atomic`).  
- **Доступ к объекту** — не синхронизирован! Если один поток удалил объект (через `shared_ptr`), а другой пытается получить к нему доступ через `.lock()`, поведение безопасно (вернётся `nullptr`), но **сам объект уже мёртв**.  

**Пример гонки (race condition)**:  
```cpp
std::weak_ptr<int> weak;
auto shared = std::make_shared<int>(42);
weak = shared;

// Поток 1:
if (!weak.expired()) {  // Проверка: strong_refs > 0?
    // Между expired() и lock() объект может быть удалён в потоке 2!
    auto locked = weak.lock();  // Может быть nullptr!
}

// Поток 2:
shared.reset();  // Удаляет объект
```  
**Решение**:  
Всегда использовать **`.lock()`** без предварительной проверки:  
```cpp
if (auto locked = weak.lock()) {  // Атомарная проверка + создание shared_ptr
    // Объект точно жив
}
```  

#### **6. Кастомные Deleter и Allocator**  
Control Block поддерживает пользовательские deleter и аллокаторы:  
```cpp
auto deleter = [](int* p) { delete p; };
auto allocator = std::allocator<int>();

// shared_ptr с кастомным deleter и аллокатором
std::shared_ptr<int> shared(new int(42), deleter, allocator);
```  
- **Deleter** хранится в Control Block и вызывается при `strong_refs == 0`.  
- **Allocator** используется для выделения/освобождения памяти под Control Block.  

#### **7. Особые случаи и оптимизации**  

**a) `make_shared` и `allocate_shared`**  
- `make_shared` объединяет объект и Control Block в одну аллокацию.  
- `allocate_shared` позволяет указать кастомный аллокатор.  

**b) `enable_shared_from_this`**  
Позволяет объекту безопасно создавать `shared_ptr` из `this`:  
```cpp
struct Foo : std::enable_shared_from_this<Foo> {
    std::shared_ptr<Foo> get_shared() { return shared_from_this(); }
};
```  
**Как работает**:  
- В Control Block добавляется **weak_ptr** на объект.  
- `shared_from_this()` создаёт `shared_ptr` из этого `weak_ptr`.  

**c) `weak_ptr` к неполному типу**  
Если `T` — неполный тип (например, forward declaration), то:  
- `shared_ptr<T>` **нельзя** создать (нужна полная декларация).  
- `weak_ptr<T>` **можно** создать (но `.lock()` потребует полного типа).  

**Оптимизации**:  
- Всегда предпочитайте `make_shared` (если не нужен кастомный deleter).  
- Избегайте раздельных аллокаций (`shared_ptr(new T)`).  

**Идеальный ответ:**  
*"Control Block — это внутренняя структура `shared_ptr`/`weak_ptr`, хранящая счётчики ссылок (`strong_refs` для владения, `weak_refs` для наблюдения), указатель на объект, deleter и аллокатор. При `strong_refs == 0` объект удаляется, при `weak_refs == 0` — уничтожается сам Control Block. `make_shared` объединяет объект и Control Block в одну аллокацию (оптимизация), тогда как `shared_ptr(new T)` создаёт их раздельно. Для потокобезопасности счётчики атомарны, но доступ к объекту требует синхронизации. Интеграция с `enable_shared_from_this` и поддержка кастомных deleter/аллокаторов делают Control Block гибким инструментом управления памятью."*