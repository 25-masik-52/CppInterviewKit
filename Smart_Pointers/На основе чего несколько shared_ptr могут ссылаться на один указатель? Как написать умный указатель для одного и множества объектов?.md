`std::shared_ptr` позволяет **разделять владение** одним объектом между несколькими экземплярами. Это работает благодаря:  
1. **Общему Control Block** (хранит счётчик ссылок).  
2. **Копированию `shared_ptr`** (увеличивает счётчик).  
3. **Автоматическому удалению объекта**, когда счётчик становится `0`.  
**Пример**:  
```cpp
auto ptr1 = std::make_shared<int>(42);  // Control Block создан, strong_refs = 1  
auto ptr2 = ptr1;                       // strong_refs = 2  
auto ptr3 = ptr2;                       // strong_refs = 3  

ptr1.reset();  // strong_refs = 2  
ptr2.reset();  // strong_refs = 1  
// Объект жив, пока ptr3 существует  
```  

**Как написать свой умный указатель?**
Рассмотрим две реализации:  
1. **Аналог `unique_ptr`** (эксклюзивное владение).  
2. **Аналог `shared_ptr`** (разделяемое владение).  

#### **1. Простой `UniquePtr` (аналог `std::unique_ptr`)**  

**Ключевые особенности**:  
- **Единственный владелец** (нельзя копировать).  
- **Передача владения** через `move`.  
- **Автоматическое удаление** при разрушении.  

**Реализация**:  
```cpp
template<typename T>
class UniquePtr {
    T* ptr;
public:
    explicit UniquePtr(T* p = nullptr) : ptr(p) {}
    
    // Запрет копирования
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
    
    // Перемещение
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
    
    ~UniquePtr() { delete ptr; }
    
    T* get() const { return ptr; }
    T& operator*() const { return *ptr; }
    T* operator->() const { return ptr; }
    
    void reset(T* p = nullptr) {
        delete ptr;
        ptr = p;
    }
};

// Использование
UniquePtr<int> ptr1(new int(42));
UniquePtr<int> ptr2 = std::move(ptr1);  // Владение передано
```  

#### **2. Простой `SharedPtr` (аналог `std::shared_ptr`)**  

**Ключевые особенности**:  
- **Разделяемое владение** (счётчик ссылок).  
- **Безопасное удаление** при `ref_count == 0`.  
- **Поддержка `weak_ptr`** (не реализовано здесь для простоты).  

**Реализация**:  
```cpp
template<typename T>
class SharedPtr {
    T* ptr;
    size_t* ref_count;
public:
    explicit SharedPtr(T* p = nullptr) : ptr(p), ref_count(new size_t(1)) {}
    
    // Копирование (увеличивает счётчик)
    SharedPtr(const SharedPtr& other) : ptr(other.ptr), ref_count(other.ref_count) {
        ++(*ref_count);
    }
    
    // Присваивание
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != &other) {
            release();  // Уменьшаем счётчик текущего объекта
            ptr = other.ptr;
            ref_count = other.ref_count;
            ++(*ref_count);
        }
        return *this;
    }
    
    // Перемещение
    SharedPtr(SharedPtr&& other) noexcept : ptr(other.ptr), ref_count(other.ref_count) {
        other.ptr = nullptr;
        other.ref_count = nullptr;
    }
    
    ~SharedPtr() { release(); }
    
    T* get() const { return ptr; }
    T& operator*() const { return *ptr; }
    T* operator->() const { return ptr; }
    
    size_t use_count() const { return ref_count ? *ref_count : 0; }
    
private:
    void release() {
        if (ref_count && --(*ref_count) == 0) {
            delete ptr;
            delete ref_count;
        }
    }
};

// Использование
SharedPtr<int> ptr1(new int(42));
SharedPtr<int> ptr2 = ptr1;  // Счётчик = 2  
```  

#### **3. Как добавить `WeakPtr`?**  
Для полной аналогии с `std::shared_ptr` нужен:  
1. **Второй счётчик (`weak_count`)** в Control Block.  
2. **Отдельный класс `WeakPtr`**, который не увеличивает `strong_refs`.  

**Упрощённая идея**:  
```cpp
template<typename T>
class WeakPtr {
    T* ptr;
    size_t* ref_count;
    size_t* weak_count;
public:
    WeakPtr(const SharedPtr<T>& shared) : ptr(shared.get()), ref_count(shared.ref_count), weak_count(new size_t(1)) {}
    
    SharedPtr<T> lock() const {
        if (ref_count && *ref_count > 0) {
            return SharedPtr<T>(ptr, ref_count);
        }
        return SharedPtr<T>();
    }
    
    ~WeakPtr() {
        if (weak_count && --(*weak_count) == 0 && !ref_count) {
            delete weak_count;
        }
    }
};
```  

**Когда что использовать**:  
- `UniquePtr` — если владелец один (например, внутри класса).  
- `SharedPtr` — если объект разделяется (например, кэш).  
- `WeakPtr` — чтобы избежать утечек (например, observer-паттерн).

**Идеальный ответ:**  
*"`std::shared_ptr` позволяет нескольким экземплярам совместно владеть объектом благодаря общему **Control Block**, содержащему счётчик ссылок (`strong_refs`), который увеличивается при копировании и уменьшается при разрушении `shared_ptr`, автоматически удаляя объект при достижении нуля. Для реализации аналога:*
*- **`UniquePtr`** (эксклюзивное владение) требует запрета копирования и поддержки семантики перемещения (`std::move`), а также автоматического удаления объекта в деструкторе.*
*- **`SharedPtr`** (разделяемое владение) использует счётчик ссылок, увеличиваемый при копировании и уменьшаемый при разрушении, с удалением объекта при обнулении счётчика.*
*- **`WeakPtr`** (опционально) требует отдельного счётчика (`weak_refs`) и методов для безопасного преобразования в `SharedPtr` (например, `lock()`)."*