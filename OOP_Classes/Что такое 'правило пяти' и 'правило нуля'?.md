#### **1. Определение + Пример кода**  

##### **Правило пяти (Rule of Five)**  
Если класс требует **явного определения** одного из следующих методов, то, скорее всего, ему нужны **все пять**:  
1. **Деструктор** (`~T()`)  
2. **Конструктор копирования** (`T(const T&)`)  
3. **Оператор копирующего присваивания** (`T& operator=(const T&)`)  
4. **Конструктор перемещения** (`T(T&&)`)  
5. **Оператор перемещающего присваивания** (`T& operator=(T&&)`)  

**Пример:**  
```cpp
class ResourceHolder {
public:
    // 1. Деструктор
    ~ResourceHolder() { delete[] data; }

    // 2. Конструктор копирования
    ResourceHolder(const ResourceHolder& other) : size(other.size) {
        data = new int[size];
        std::copy(other.data, other.data + size, data);
    }

    // 3. Оператор копирующего присваивания
    ResourceHolder& operator=(const ResourceHolder& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }

    // 4. Конструктор перемещения
    ResourceHolder(ResourceHolder&& other) noexcept 
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }

    // 5. Оператор перемещающего присваивания
    ResourceHolder& operator=(ResourceHolder&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }

private:
    int* data = nullptr;
    size_t size = 0;
};
```  

##### **Правило нуля (Rule of Zero)**  
Если класс **не управляет ресурсами напрямую**, то **не нужно** определять ни один из пяти специальных методов — их работу выполнят стандартные (`=default`) или компиляторные версии.  

**Пример:**  
```cpp
class Person {
public:
    Person(std::string name, int age) : name(std::move(name)), age(age) {}

    // Ничего не определяем — компилятор сгенерирует всё сам:
    // ~Person() = default;
    // Person(const Person&) = default;
    // Person& operator=(const Person&) = default;
    // Person(Person&&) = default;
    // Person& operator=(Person&&) = default;

private:
    std::string name;  // Управляет памятью самостоятельно
    int age;           // Тривиально копируется
};
```  

#### **2. Зачем это нужно и где применяется?**  
- **Правило пяти:**  
  - Для классов, **владеющих ресурсами** (память, файлы, сокеты).  
  - Если определен хотя бы один из пяти методов, остальные **могут понадобиться** для корректной работы.  
- **Правило нуля:**  
  - Для классов, **не владеющих ресурсами напрямую** (использующих RAII-обёртки: `std::string`, `std::vector`, умные указатели).  
  - Упрощает код и уменьшает вероятность ошибок.  

#### **3. Подводные камни и ошибки**  
- **Ошибки в "правиле пяти":**  
  - Забыть **noexcept** для перемещающих операций → потеря оптимизаций.  
  - Не проверить **самоприсваивание** в операторах → утечки/двойное освобождение.  
- **Антипаттерны:**  
  - Определять только часть методов из пяти → риск некорректного копирования/перемещения.  
  - Игнорировать **"правило нуля"** и писать лишний код.  

#### **4. Пример из практики**  
**Ситуация 1 (Правило пяти):** Класс для работы с файлом.  
```cpp
class File {
public:
    File(const char* filename) : handle(fopen(filename, "r")) {}
    ~File() { if (handle) fclose(handle); }

    // Остальные 4 метода обязательны!
    File(const File&) = delete;  // Запрещаем копирование
    File& operator=(const File&) = delete;
    File(File&& other) noexcept : handle(other.handle) { other.handle = nullptr; }
    File& operator=(File&& other) noexcept { 
        if (this != &other) {
            if (handle) fclose(handle);
            handle = other.handle;
            other.handle = nullptr;
        }
        return *this;
    }

private:
    FILE* handle;
};
```  

**Ситуация 2 (Правило нуля):** Обёртка для данных.  
```cpp
class DataWrapper {
public:
    DataWrapper(std::vector<int> data) : data(std::move(data)) {}
    // Всё генерируется автоматически — вектор сам управляет памятью.

private:
    std::vector<int> data;
};
```  

#### **5. Сравнение подходов**  
| **Критерий**           | **Правило пяти**                   | **Правило нуля**                      |
| ---------------------- | ---------------------------------- | ------------------------------------- |
| **Когда использовать** | Класс владеет ресурсами            | Класс использует RAII-обёртки         |
| **Сложность**          | Требует ручного управления         | Автоматическая генерация компилятором |
| **Риск ошибок**        | Высокий (утечки, двойное удаление) | Низкий                                |

#### **Ключевые моменты для собеседования:**  
✔ **Правило пяти:** Если класс управляет ресурсами — определяйте **все 5 методов**.  
✔ **Правило нуля:** Если ресурсы уже управляются RAII-объектами — **ничего не определяйте**.  
✔ **Исключения:**  
   - Если копирование запрещено — используйте `=delete`.  
   - Если перемещение не нужно — удалите его (`T(T&&) = delete`).  

**Ошибка новичка:**  
```cpp
class Buffer {
public:
    Buffer(size_t size) : data(new int[size]) {}
    ~Buffer() { delete[] data; }
    // Забыли конструктор копирования/перемещения → проблемы при копировании!
private:
    int* data;
};
```  

**Идеальный ответ:**  
*«Правило пяти требует явного определения деструктора, конструкторов и операторов копирования/перемещения, если класс владеет ресурсами. Например, для класса с сырым указателем нужно реализовать все пять методов. Правило нуля, наоборот, рекомендует не определять их, если класс использует RAII-типы (как std::vector), которые сами управляют ресурсами. Это уменьшает объем кода и предотвращает ошибки.»*