#### **1. Определение + Пример кода**  
**Деструктор** — это специальный метод класса, который автоматически вызывается при уничтожении объекта. Он освобождает ресурсы (память, файлы, сокеты и т.д.), выделенные объектом.  

**Пример:**  
```cpp
class FileHandler {
public:
    FileHandler(const std::string& filename) {
        file = fopen(filename.c_str(), "r");
    }

    ~FileHandler() {  // Деструктор
        if (file) {
            fclose(file);  // Освобождение ресурса
            std::cout << "File closed\n";
        }
    }

private:
    FILE* file;
};
```  

#### **2. Виртуальный деструктор**  
**Зачем нужен:**  
- Если класс предназначен для наследования и будет удаляться через указатель на базовый класс, деструктор **должен быть виртуальным**. Иначе вызовется только деструктор базового класса, что приведет к утечке ресурсов.  

**Пример проблемы:**  
```cpp
class Base {
public:
    ~Base() { std::cout << "Base dtor\n"; }  // Не виртуальный!
};

class Derived : public Base {
public:
    ~Derived() { std::cout << "Derived dtor\n"; }
};

int main() {
    Base* obj = new Derived();
    delete obj;  // Вызовется только ~Base() → утечка ресурсов Derived
}
```  

**Исправленный вариант:**  
```cpp
class Base {
public:
    virtual ~Base() { std::cout << "Base dtor\n"; }  // Виртуальный
};

// Теперь delete obj вызовет сначала ~Derived(), потом ~Base().
```  

#### **3. Когда объявлять деструктор виртуальным?**  
1. **Если класс является полиморфным** (содержит хотя бы одну виртуальную функцию).  
2. **Если класс предназначен для наследования** (даже если сейчас у него нет виртуальных методов).  
3. **Если возможна утечка ресурсов** при удалении через указатель на базовый класс.  

**Исключения:**  
- **Финальные классы** (не предназначены для наследования) — виртуальный деструктор не нужен.  
- **Классы без полиморфизма** (например, `std::vector`) — деструктор не виртуальный.  

#### **4. Подводные камни и ошибки**  
- **Ошибки:**  
  - Не сделать деструктор виртуальным в полиморфном классе → утечка ресурсов.  
  - Слишком раннее объявление виртуальности (если класс не будет полиморфным) → лишние накладные расходы (vtable).  
- **Антипаттерны:**  
  - Виртуальный деструктор в классе без виртуальных методов (если нет наследования).  

#### **5. Пример из практики**  
**Ситуация:** Иерархия графических объектов:  
```cpp
class GraphicObject {
public:
    virtual void draw() const = 0;
    virtual ~GraphicObject() = default;  // Виртуальный деструктор
};

class Circle : public GraphicObject {
public:
    void draw() const override { /* ... */ }
    ~Circle() { std::cout << "Circle destroyed\n"; }
};

// Использование:
GraphicObject* obj = new Circle();
delete obj;  // Корректно вызовет ~Circle(), затем ~GraphicObject()
```  

#### **Ключевые моменты для собеседования:**  
✔ **Деструктор** — освобождает ресурсы объекта.  
✔ **Виртуальный деструктор** — обязателен для полиморфных классов.  
✔ **Когда использовать:**  
   - Если класс имеет виртуальные методы.  
   - Если возможна утечка ресурсов при удалении через указатель на базовый класс.  

**Ошибка новичка:**  
```cpp
class Base {
public:
    ~Base() {}  // Не виртуальный
};

class Derived : public Base {
    int* data;
public:
    Derived() : data(new int[100]) {}
    ~Derived() { delete[] data; }  // Не вызовется при delete через Base*
};
```  

**Идеальный ответ:**  
*«Деструктор — это метод, освобождающий ресурсы объекта. Виртуальный деструктор нужен, если класс используется полиморфно: при удалении объекта через указатель на базовый класс он гарантирует вызов деструктора производного класса. Например, в иерархии `Shape -> Circle` деструктор `Shape` должен быть виртуальным, чтобы корректно освобождать ресурсы `Circle`. Виртуальный деструктор объявляют для всех классов, предназначенных для наследования.»*