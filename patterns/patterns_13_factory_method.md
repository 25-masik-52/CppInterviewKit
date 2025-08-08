#### **1. Определение**  
**Фабричный метод (Factory Method)** — это порождающий паттерн, который определяет **интерфейс для создания объектов**, но делегирует выбор конкретного класса объекта подклассам.  

**Ключевая идея**:  
> *"Пусть подклассы решают, какой объект создавать"*  

#### **2. Пример реализации**  
**Базовый класс продукта:**  
```cpp
class Product {
public:
    virtual ~Product() = default;
    virtual void operation() = 0;
};
```

**Конкретные продукты:**  
```cpp
class ConcreteProductA : public Product {
public:
    void operation() override { 
        std::cout << "Product A operation\n"; 
    }
};

class ConcreteProductB : public Product {
public:
    void operation() override { 
        std::cout << "Product B operation\n"; 
    }
};
```

**Абстрактный создатель (Creator):**  
```cpp
class Creator {
public:
    virtual ~Creator() = default;
    
    // Фабричный метод (может быть чисто виртуальным или иметь реализацию по умолчанию)
    virtual std::unique_ptr<Product> createProduct() = 0;
    
    void someOperation() {
        auto product = createProduct();
        product->operation();
    }
};
```

**Конкретные создатели:**  
```cpp
class ConcreteCreatorA : public Creator {
public:
    std::unique_ptr<Product> createProduct() override {
        return std::make_unique<ConcreteProductA>();
    }
};

class ConcreteCreatorB : public Creator {
public:
    std::unique_ptr<Product> createProduct() override {
        return std::make_unique<ConcreteProductB>();
    }
};
```

**Использование:**  
```cpp
int main() {
    std::unique_ptr<Creator> creator = std::make_unique<ConcreteCreatorA>();
    creator->someOperation();  // Создаст и использует ProductA

    creator = std::make_unique<ConcreteCreatorB>();
    creator->someOperation();  // Создаст и использует ProductB
}
```

#### **3. Когда использовать?**  
- Нужна **гибкость** в создании объектов, т.е. **когда класс заранее не знает, объекты каких типов ему нужно создавать**  
- **Когда нужно дать подклассам контроль над созданием объектов**  
- **Для соблюдения OCP (новые типы добавляются без изменения существующего кода)**

#### **4. Плюсы и минусы**  
| **Преимущества**               | **Недостатки**                  |
|--------------------------------|---------------------------------|
| 🔹 Гибкость: подклассы выбирают тип объекта | 🔹 Увеличивает количество классов |
| 🔹 Соответствует OCP           | 🔹 Избыточен для простых случаев |
| 🔹 Избегает жесткой привязки   | 🔹 Клиент должен знать Creator   |

#### **5. Отличия от других фабрик**  
| **Критерий**       | **Фабричный метод**       | **Абстрактная фабрика**       | **Простая фабрика**       |
|--------------------|--------------------------|------------------------------|--------------------------|
| **Создаёт**        | Один продукт             | Семейство продуктов          | Один продукт             |
| **Гибкость**       | Через подклассы          | Через композицию интерфейсов | Через параметры метода   |
| **Сложность**      | Средняя                  | Высокая                      | Низкая                   |

#### **6. Реальный пример**  
В Qt фабричный метод используется в `QApplication` для создания виджетов:  
```cpp
QPushButton* button = new QPushButton(parent);  // Фабричный метод в действии
```

#### **7. Вариации**  
- **Фабричный метод с параметром** (но нарушает OCP):  
```cpp
std::unique_ptr<Product> createProduct(ProductType type) {
    switch(type) {
        case ProductType::A: return std::make_unique<ConcreteProductA>();
        // ...
    }
}
```

**Идеальный ответ:**  
*"Фабричный метод — это паттерн, который делегирует создание объектов подклассам, позволяя им выбирать конкретный тип продукта. Он полезен, когда система должна оставаться расширяемой без изменения существующего кода (OCP). Реализуется через абстрактный Creator с фабричным методом и конкретные подклассы-создатели. Плюсы: гибкость и соблюдение OCP. Минусы: усложнение иерархии классов. Альтернативы: простая фабрика (для базовых случаев) или абстрактная фабрика (для семейств объектов)."*