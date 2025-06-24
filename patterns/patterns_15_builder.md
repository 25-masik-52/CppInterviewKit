#### **1. Определение**
**Строитель** — это порождающий паттерн, который позволяет создавать сложные объекты **пошагово**, разделяя процесс конструирования от его представления. Особенно полезен для объектов с множеством параметров или вариативными конфигурациями.

#### **2. Классическая реализация на C++**
Рассмотрим пример строительства дома (`House`).

**1. Продукт (House)**
```cpp
class House {
public:
    void setWalls(const std::string& walls) { walls_ = walls; }
    void setRoof(const std::string& roof) { roof_ = roof; }
    void setWindows(int windows) { windows_ = windows; }
    
    void describe() const {
        std::cout << "Дом: " << walls_ << " стены, "
                  << roof_ << " крыша, " << windows_ << " окна.\n";
    }
private:
    std::string walls_;
    std::string roof_;
    int windows_ = 0;
};
```

**2. Абстрактный строитель (Builder)**
```cpp
class HouseBuilder {
public:
    virtual ~HouseBuilder() = default;
    virtual void buildWalls() = 0;
    virtual void buildRoof() = 0;
    virtual void buildWindows() = 0;
    virtual House getResult() = 0;
};
```

**3. Конкретные строители**
```cpp
// Деревянный дом
class WoodHouseBuilder : public HouseBuilder {
public:
    void buildWalls() override { house_.setWalls("деревянные"); }
    void buildRoof() override { house_.setRoof("деревянная"); }
    void buildWindows() override { house_.setWindows(4); }
    House getResult() override { return house_; }
private:
    House house_;
};

// Каменный дом
class StoneHouseBuilder : public HouseBuilder {
public:
    void buildWalls() override { house_.setWalls("каменные"); }
    void buildRoof() override { house_.setRoof("черепица"); }
    void buildWindows() override { house_.setWindows(6); }
    House getResult() override { return house_; }
private:
    House house_;
};
```

**4. Директор (Director)**
```cpp
class HouseDirector {
public:
    void setBuilder(HouseBuilder* builder) { builder_ = builder; }
    House construct() {
        builder_->buildWalls();
        builder_->buildRoof();
        builder_->buildWindows();
        return builder_->getResult();
    }
private:
    HouseBuilder* builder_;
};
```

**5. Использование**
```cpp
int main() {
    WoodHouseBuilder woodBuilder;
    StoneHouseBuilder stoneBuilder;
    HouseDirector director;

    director.setBuilder(&woodBuilder);
    House woodHouse = director.construct();
    woodHouse.describe(); // Дом: деревянные стены, деревянная крыша, 4 окна.

    director.setBuilder(&stoneBuilder);
    House stoneHouse = director.construct();
    stoneHouse.describe(); // Дом: каменные стены, черепица крыша, 6 окон.
}
```

#### **3. Плюсы и минусы**
| **Преимущества**                          | **Недостатки**                     |
|-------------------------------------------|------------------------------------|
| ✅ Пошаговое создание сложных объектов    | ❌ Увеличивает количество классов  |
| ✅ Гибкость (разные конфигурации объекта) | ❌ Избыточен для простых объектов  |
| ✅ Изоляция кода сборки от продукта       |                                    |

#### **4. Когда использовать?**
- **Сложные объекты** (GUI, SQL-запросы, конфигурации).
- **Неизменяемые (immutable) объекты**.
- **Когда нужно поддерживать разные варианты объекта**.

#### **5. Вариации**
**1. Fluent Interface (цепочка вызовов)**
```cpp
class HouseBuilder {
public:
    HouseBuilder& withWalls(const std::string& walls) {
        house_.setWalls(walls);
        return *this;
    }
    House build() { return house_; }
private:
    House house_;
};

// Использование:
House house = HouseBuilder()
    .withWalls("кирпич")
    .withRoof("металл")
    .build();
```

**2. Builder без Director**  
Если порядок шагов не важен, клиент может вызывать методы напрямую.

#### **6. Сравнение с другими паттернами**
| **Паттерн**       | **Отличие**                              |
|--------------------|------------------------------------------|
| **Фабрика**        | Создаёт объект в один шаг                |
| **Прототип**       | Копирует существующий объект             |
| **Строитель**      | Контролирует каждый этап создания        |

#### **7. Реальные примеры**
- **`std::stringstream`**: Построение строки через `<<`.
- **Библиотеки HTTP**: Пошаговая настройка запросов (например, cURL).

**Идеальный ответ:**  
*"Паттерн Строитель (Builder) предназначен для пошагового создания сложных объектов. Он разделяет процесс конструирования (Builder) и результат (Product), что особенно полезно при работе с объектами, имеющими множество параметров. Классическая реализация включает: 1) Product (объект для сборки), 2) Builder (абстрактный интерфейс), 3) ConcreteBuilder (конкретные реализации), 4) Director (управляет процессом). Преимущества: гибкость, изоляция кода. Недостатки: усложнение структуры. Используется в GUI, API, конфигураторах."*