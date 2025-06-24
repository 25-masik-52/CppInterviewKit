#### **1. Определение**  
**Паттерн Стратегия (Strategy)** — это поведенческий паттерн, который определяет семейство алгоритмов, инкапсулирует каждый из них и делает их взаимозаменяемыми. Позволяет изменять поведение объекта во время выполнения, делегируя работу инкапсулированному алгоритму.

#### **2. Код-пример**  
**Пример: Система оплаты с разными стратегиями**  
```cpp
#include <iostream>
#include <memory>
#include <string>

// Интерфейс стратегии оплаты
class PaymentStrategy {
public:
    virtual ~PaymentStrategy() = default;
    virtual void pay(double amount) const = 0;
};

// Конкретные стратегии
class CreditCardPayment : public PaymentStrategy {
    std::string cardNumber;
public:
    CreditCardPayment(const std::string& num) : cardNumber(num) {}
    void pay(double amount) const override {
        std::cout << "Paid " << amount << " via Credit Card (" 
                  << cardNumber.substr(cardNumber.length() - 4) << ")\n";
    }
};

class PayPalPayment : public PaymentStrategy {
    std::string email;
public:
    PayPalPayment(const std::string& e) : email(e) {}
    void pay(double amount) const override {
        std::cout << "Paid " << amount << " via PayPal (" << email << ")\n";
    }
};

// Контекст
class PaymentProcessor {
    std::unique_ptr<PaymentStrategy> strategy;
public:
    void setStrategy(std::unique_ptr<PaymentStrategy> s) {
        strategy = std::move(s);
    }
    void executePayment(double amount) const {
        if (strategy) {
            strategy->pay(amount);
        } else {
            std::cout << "Payment method not selected!\n";
        }
    }
};

int main() {
    PaymentProcessor processor;
    
    // Оплата кредитной картой
    processor.setStrategy(std::make_unique<CreditCardPayment>("1234567812345678"));
    processor.executePayment(100.50);
    
    // Смена стратегии на PayPal
    processor.setStrategy(std::make_unique<PayPalPayment>("user@example.com"));
    processor.executePayment(75.25);
}
```
**Вывод:**  
```
Paid 100.5 via Credit Card (5678)  
Paid 75.25 via PayPal (user@example.com)  
```

#### **3. Компоненты и связи**  
- **Strategy (PaymentStrategy)**  
  Интерфейс, определяющий алгоритм (`pay()`).  
- **Concrete Strategy (CreditCardPayment, PayPalPayment)**  
  Конкретные реализации алгоритмов.  
- **Context (PaymentProcessor)**  
  Хранит стратегию и делегирует ей выполнение.  

#### **4. Плюсы паттерна**  
- **Гибкость**: Алгоритмы можно менять во время выполнения.  
- **Инкапсуляция**: Каждая стратегия изолирована в своём классе.  
- **Устранение условных операторов**: Замена `if/switch` на полиморфизм.  

#### **5. Когда использовать?**  
- Когда нужно использовать разные варианты одного алгоритма.  
- Для замены множественных условных операторов.  
- В системах, где поведение должно настраиваться.  

#### **6. Подводные камни и решения**  
**Минусы:**  
- **Увеличение числа классов** (компенсируется гибкостью).  
- **Клиент должен знать о стратегиях** (можно использовать фабрику).  

**Способы решения:**  
- Группировать связанные стратегии в модули.  
- Использовать фабрику для создания стратегий.  

#### **7. Альтернативы и отличия от других паттернов**  

| **Паттерн**       | **Отличие от Strategy**                     |
|-------------------|---------------------------------------------|
| **Состояние**     | Меняет поведение автоматически при изменении состояния. |
| **Команда**       | Инкапсулирует запросы, а не алгоритмы.      |
| **Шаблонный метод** | Определяет скелет алгоритма, стратегия — всю реализацию. |

#### **8. Примеры в реальных проектах**  
- **Платёжные системы**: Выбор способа оплаты.  
- **Навигация**: Разные алгоритмы построения маршрута.  
- **Игры**: Поведение AI противников.  

**Идеальный ответ:**  
*"Паттерн Strategy инкапсулирует семейство алгоритмов, делая их взаимозаменяемыми. Состоит из Strategy (интерфейс), ConcreteStrategy (реализации) и Context (исполнитель). Пример — система оплаты с разными методами. Плюсы: гибкость и устранение условных операторов. Минус: рост числа классов. Отличается от Состояния (автоматическое изменение) и Команды (инкапсуляция запросов)."*