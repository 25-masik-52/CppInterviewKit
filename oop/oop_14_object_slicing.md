#### **1. Определение + Пример кода**  
**Срезка объекта** — это проблема, возникающая при присваивании объекта производного класса объекту базового класса, что приводит к **потере данных** и **полиморфного поведения**.  

**Пример:**  
```cpp
class Base {
public:
    virtual void print() const { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void print() const override { std::cout << "Derived\n"; }
    int extra_data = 42;  // Дополнительное поле
};

int main() {
    Derived d;
    Base b = d;  // Срезка! Копируется только Base-часть объекта
    b.print();   // Выведет "Base" (а не "Derived")
    // extra_data потеряно
}
```  

#### **2. Почему это происходит?**  
- При копировании/присваивании **только базовая часть** объекта копируется (размер `Base` ≤ размер `Derived`).  
- **Полиморфизм** не работает, так как копирование создаёт новый объект типа `Base`.  

#### **3. Как избежать срезки?**  

##### **1. Работа через указатели/ссылки**  
Использовать указатели (`Base*`) или ссылки (`Base&`), чтобы сохранить полиморфизм:  
```cpp
Derived d;
Base* ptr = &d;  // Указатель на базовый класс
Base& ref = d;   // Ссылка на базовый класс
ptr->print();    // Выведет "Derived" (полиморфизм работает)
```  

##### **2. Запрет копирования**  
Если срезка недопустима, удалите конструктор копирования/присваивания:  
```cpp
class Base {
public:
    Base(const Base&) = delete;  // Запрещаем копирование
    Base& operator=(const Base&) = delete;
};
```  

##### **3. Использование `std::shared_ptr`/`std::unique_ptr`**  
Умные указатели сохраняют полный тип объекта:  
```cpp
std::shared_ptr<Base> ptr = std::make_shared<Derived>();
ptr->print();  // Выведет "Derived"
```  

##### **4. Клонирование объектов**  
Добавьте виртуальный метод `clone()`:  
```cpp
class Base {
public:
    virtual Base* clone() const { return new Base(*this); }
};

class Derived : public Base {
public:
    Derived* clone() const override { return new Derived(*this); }
};

Derived d;
Base* b = d.clone();  // Корректное копирование
```  

#### **4. Подводные камни и ошибки**  
- **Ошибки:**  
  - Передача объекта по значению в функцию:  
    ```cpp
    void process(Base b) { ... }  // Срезка, если передать Derived!
    ```  
    **Решение:** Принимать по ссылке (`const Base&`).  
  - Хранение объектов в контейнере:  
    ```cpp
    std::vector<Base> vec;
    vec.push_back(Derived{});  // Срезка!
    ```  
    **Решение:** Использовать `std::vector<Base*>`.  

- **Антипаттерны:**  
  - Игнорирование срезки при проектировании иерархий классов.  

#### **5. Пример из практики**  
**Ситуация:** Система обработки графических объектов.  
```cpp
class Shape {
public:
    virtual void draw() const = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
    int radius = 10;
public:
    void draw() const override { std::cout << "Circle\n"; }
};

// Опасный код:
std::vector<Shape> shapes;  // Срезка при добавлении Circle!
shapes.push_back(Circle{}); // Circle превращается в Shape

// Правильное решение:
std::vector<std::unique_ptr<Shape>> shapes;
shapes.push_back(std::make_unique<Circle>());
shapes[0]->draw();  // Выведет "Circle"
```  

#### **Ключевые моменты для собеседования:**  
✔ **Срезка** — потеря данных и полиморфизма при копировании производного класса в базовый.  
✔ **Как избежать:**  
  - Использовать указатели/ссылки (`Base*`, `Base&`).  
  - Запретить копирование (`=delete`).  
  - Применять умные указатели (`std::shared_ptr`).  
✔ **Опасные места:**  
  - Передача объектов в функции по значению.  
  - Хранение полиморфных объектов в контейнерах.  

**Ошибка новичка:**  
```cpp
void saveToFile(Base obj) { ... }  // Срезка, если передать Derived!
saveToFile(Derived{});
```  

**Идеальный ответ:**  
*«Срезка объекта — это потеря данных и полиморфного поведения при копировании производного класса в базовый. Например, если `Circle` наследуется от `Shape`, то присваивание `Circle` к `Shape` обрежет все специфичные для `Circle` данные. Чтобы избежать срезки, нужно работать через указатели/ссылки (`Shape*`, `Shape&`), использовать умные указатели или запретить копирование. Особенно опасна срезка при передаче объектов в функции по значению или хранении в контейнерах.»*