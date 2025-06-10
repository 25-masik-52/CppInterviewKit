#### **1. Определение**
Абстрактная фабрика — это порождающий паттерн, который предоставляет интерфейс для создания **семейств связанных или зависимых объектов**, не указывая их конкретных классов.

**Ключевая идея**:
> *"Один интерфейс — множество реализаций для целого семейства продуктов"*

#### **2. Пример реализации**
**1. Абстрактные продукты**
```cpp
// Интерфейс кнопки
class Button {
public:
    virtual ~Button() = default;
    virtual void render() = 0;
};

// Интерфейс чекбокса
class Checkbox {
public:
    virtual ~Checkbox() = default;
    virtual void render() = 0;
};
```

**2. Конкретные продукты (Windows-стиль)**
```cpp
class WinButton : public Button {
public:
    void render() override { 
        std::cout << "Render Windows button\n";
    }
};

class WinCheckbox : public Checkbox {
public:
    void render() override {
        std::cout << "Render Windows checkbox\n";
    }
};
```

**3. Абстрактная фабрика**
```cpp
class GUIFactory {
public:
    virtual ~GUIFactory() = default;
    virtual std::unique_ptr<Button> createButton() = 0;
    virtual std::unique_ptr<Checkbox> createCheckbox() = 0;
};
```

**4. Конкретные фабрики**
```cpp
class WinFactory : public GUIFactory {
public:
    std::unique_ptr<Button> createButton() override {
        return std::make_unique<WinButton>();
    }
    std::unique_ptr<Checkbox> createCheckbox() override {
        return std::make_unique<WinCheckbox>();
    }
};
```

**5. Клиентский код**
```cpp
void Application(const GUIFactory& factory) {
    auto button = factory.createButton();
    auto checkbox = factory.createCheckbox();
    button->render();
    checkbox->render();
}

int main() {
    WinFactory winFactory;
    Application(winFactory);
    // Output:
    // Render Windows button
    // Render Windows checkbox
}
```

#### **3. Когда использовать?**
- **Кроссплатформенные приложения** (создание UI для Windows/macOS)  
- **Работа с разными БД** (семейство SQL-запросов)  
- **Игровые движки** (разные стили персонажей и окружения)  

#### **4. Плюсы и минусы**
| **Преимущества** | **Недостатки** |
|-----------------|---------------|
| 🔹 Гарантирует совместимость продуктов | 🔹 Сложность добавления новых продуктов |
| 🔹 Изолирует клиентский код от конкретных классов | 🔹 Большое количество классов |
| 🔹 Легко заменить целое семейство продуктов | 🔹 Избыточен для простых случаев |

#### **5. Сравнение с другими фабриками**
| **Критерий** | **Простая фабрика** | **Фабричный метод** | **Абстрактная фабрика** |
|-------------|--------------------|--------------------|------------------------|
| **Создаёт** | Один продукт | Один продукт | Семейство продуктов |
| **Гибкость** | Низкая | Средняя | Высокая |
| **Сложность** | Простая | Средняя | Сложная |

#### **6. Реальный пример**
**Документооборот**:
```cpp
class DocFactory {
public:
    virtual std::unique_ptr<Document> createDocument() = 0;
    virtual std::unique_ptr<Formatter> createFormatter() = 0;
};

class PdfFactory : public DocFactory {
    // Реализация для PDF
};

class DocxFactory : public DocFactory {
    // Реализация для DOCX
};
```

**Идеальный ответ**:
*"Абстрактная фабрика — это паттерн для создания семейств связанных объектов. Он полезен, когда система должна работать с различными вариантами продуктов, сохраняя их совместимость. Реализуется через: 1) абстрактные интерфейсы продуктов, 2) конкретные реализации продуктов, 3) абстрактную фабрику и 4) конкретные фабрики. Плюсы: гибкость и изоляция кода. Минусы: сложность и избыточность для простых задач. Используйте её для кроссплатформенных UI, игровых движков и работы с разными БД."*