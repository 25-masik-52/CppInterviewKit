#### **1. Определение**  
**Фабрика без `if`/`switch`** — это реализация паттерна "Фабрика", где создание объектов происходит через:  
- **Регистрацию типов** (например, в `std::map`).  
- **Шаблонные методы** (статическая фабрика).  
- **CRTP** (полностью статическое решение).  

#### **2. Способы реализации**  

##### **1. Регистрация фабричных функций в `std::map`**  
**Идея:** Хранить фабричные лямбды в ассоциативном контейнере.  

```cpp
class ShapeFactory {
    std::map<std::string, std::function<std::unique_ptr<Shape>()>> creators_;
public:
    void registerShape(const std::string& name, auto creator) {
        creators_[name] = creator;
    }

    std::unique_ptr<Shape> create(const std::string& name) const {
        if (auto it = creators_.find(name); it != creators_.end()) {
            return it->second();
        }
        return nullptr;
    }
};

// Использование:
ShapeFactory factory;
factory.registerShape("Circle", [] { return std::make_unique<Circle>(); });
auto circle = factory.create("Circle");
```

**Плюсы:**  
✅ Динамическая регистрация типов.  
✅ Нет `if`/`switch` по типам.  

**Минусы:**  
❌ Незначительный оверхед на поиск в `std::map`.  

##### **2. Шаблонная фабрика**  
**Идея:** Использовать шаблоны для регистрации типов на этапе компиляции.  

```cpp
template <typename T>
class ShapeFactory {
public:
    static std::unique_ptr<Shape> create() {
        return std::make_unique<T>();
    }
};

// Использование:
auto circle = ShapeFactory<Circle>::create();
```

**Плюсы:**  
✅ Нет runtime-накладных расходов.  
✅ Статическая типобезопасность.  

**Минусы:**  
❌ Не поддерживает динамическую регистрацию.  

##### **3. CRTP (Статический полиморфизм)**  
**Идея:** Использовать шаблонный базовый класс для создания объектов.  

```cpp
template <typename Derived>
class Shape {
public:
    static std::unique_ptr<Shape> create() {
        return std::make_unique<Derived>();
    }
};

class Circle : public Shape<Circle> { /* ... */ };

// Использование:
auto circle = Shape<Circle>::create();
```

**Плюсы:**  
✅ Максимальная производительность.  
✅ Нет виртуальных вызовов.  

**Минусы:**  
❌ Жёсткая привязка типов на этапе компиляции.  

#### **3. Когда что использовать?**  
- **`std::map` + лямбды**: GUI, плагины, конфигурируемые фабрики.  
- **Шаблонная фабрика**: Библиотеки с жёсткой типизацией.  
- **CRTP**: Высокопроизводительные системы (игры, embedded).  

#### **4. Сравнение методов**  

| **Критерий**                 | **`std::map` + лямбды** | **Шаблонная фабрика** | **CRTP**              |
| ---------------------------- | ----------------------- | --------------------- | --------------------- |
| **Гибкость**                 | Высокая (runtime)       | Средняя               | Низкая (compile-time) |
| **Производительность**       | Умеренная               | Высокая               | Максимальная          |
| **Динамическая регистрация** | Да                      | Нет                   | Нет                   |

**Идеальный ответ:**  
*"Паттерн 'Фабрика' без условных операторов можно реализовать тремя способами:* 
*1. **Через `std::map` и лямбды** — гибкое runtime-решение с регистрацией типов.*
*2. **Шаблонную фабрику** — типобезопасный вариант для compile-time.*
*3. **CRTP** — максимально производительный статический метод.*
*Пример с `std::map` универсален для плагинов, шаблоны — для строгой типизации, а CRTP — для embedded-систем. Выбор зависит от требований к гибкости и производительности."*