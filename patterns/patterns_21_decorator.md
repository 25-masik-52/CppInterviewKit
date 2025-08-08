#### **1. Определение**
**Декоратор (Decorator)** — это структурный паттерн, который позволяет **динамически добавлять** новые функциональные возможности объектам, оборачивая их в специальные классы-декораторы. Это более гибкая альтернатива классическому наследованию.

#### **2. Когда использовать?**
- Когда нужно **добавлять/удалять** обязанности объектов **во время выполнения**
- Когда **наследование невозможно** или нецелесообразно
- Для **расширения legacy-кода** без его модификации

#### **3. Классическая реализация (на примере системы уведомлений)**

**1. Базовый компонент**
```cpp
class Notification {
public:
    virtual ~Notification() = default;
    virtual void send() const = 0;
};
```

**2. Конкретный компонент**
```cpp
class EmailNotification : public Notification {
public:
    void send() const override {
        std::cout << "Отправка email-уведомления\n";
    }
};
```

**3. Базовый декоратор**
```cpp
class NotificationDecorator : public Notification {
protected:
    Notification* notification;
public:
    explicit NotificationDecorator(Notification* notif) 
        : notification(notif) {}
    ~NotificationDecorator() { delete notification; }
    
    void send() const override {
        notification->send();
    }
};
```

**4. Конкретные декораторы**
```cpp
class SMSDecorator : public NotificationDecorator {
public:
    explicit SMSDecorator(Notification* notif)
        : NotificationDecorator(notif) {}
        
    void send() const override {
        NotificationDecorator::send();
        std::cout << "Отправка SMS-уведомления\n";
    }
};

class SlackDecorator : public NotificationDecorator {
public:
    explicit SlackDecorator(Notification* notif)
        : NotificationDecorator(notif) {}
        
    void send() const override {
        NotificationDecorator::send();
        std::cout << "Отправка Slack-уведомления\n";
    }
};
```

**5. Использование**
```cpp
int main() {
    Notification* notif = new EmailNotification();
    notif = new SMSDecorator(notif);
    notif = new SlackDecorator(notif);
    
    notif->send();
    /* Вывод:
       Отправка email-уведомления
       Отправка SMS-уведомления
       Отправка Slack-уведомления
    */
    
    delete notif;
}
```

#### **4. Плюсы и минусы**

| **Преимущества**                             | **Недостатки**                     |
| -------------------------------------------- | ---------------------------------- |
| ✅ Гибкость (динамическое добавление функций) | ❌ Много мелких классов             |
| ✅ Соответствует OCP (открыт для расширения)  | ❌ Сложность конфигурации           |
| ✅ Позволяет комбинировать поведения          | ❌ Нет контроля порядка декораторов |

#### **5. Сравнение с другими паттернами**

| **Паттерн** | **Отличие** |
|------------|------------|
| **Стратегия** | Заменяет поведение целиком |
| **Компоновщик** | Работает с иерархиями объектов |
| **Адаптер** | Изменяет интерфейс объекта |

#### **6. Реальные примеры**
1. **Потоки ввода/вывода**:
```cpp
std::ifstream file("data.txt");
std::istream& encrypted = decryptStream(file);
```

2. **Веб-фреймворки**:
```python
# Flask/Python пример
@app.route('/')
@login_required  # Декоратор
@cache(300)      # Еще декоратор
def index(): pass
```

#### **7. Best Practices**
1. Используйте **умные указатели** для управления памятью
2. Реализуйте **интерфейс идентичный** оригинальному компоненту
3. Избегайте **чрезмерного количества** декораторов

**Идеальный ответ**:
*"Декоратор позволяет динамически добавлять объектам новые обязанности через композицию. Состоит из: 1) Component (базовый интерфейс), 2) ConcreteComponent (основной класс), 3) Decorator (обертка), 4) ConcreteDecorator (реализация доп.функций). Преимущества: гибкость, соответствие OCP. Недостатки: множество классов, сложность конфигурации. Применяется в системах уведомлений, потоках данных и middleware."*