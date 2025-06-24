#### **1. Определение**  
**Паттерн Цепочка ответственности (Chain of Responsibility)** — это поведенческий паттерн, позволяющий передавать запросы по цепочке обработчиков. Каждый обработчик решает, может ли он обработать запрос, и передаёт его следующему, если не может.  

#### **2. Код-пример**  
**Пример: Система логирования с разными уровнями**  
```cpp
#include <iostream>
#include <memory>

enum LogLevel { INFO, WARNING, ERROR };

// Базовый обработчик
class Logger {
protected:
    LogLevel level_;
    std::unique_ptr<Logger> next_;

public:
    Logger(LogLevel level) : level_(level) {}

    void setNext(std::unique_ptr<Logger> next) {
        next_ = std::move(next);
    }

    virtual void log(LogLevel severity, const std::string& msg) {
        if (level_ <= severity) {
            write(msg);
        }
        if (next_) {
            next_->log(severity, msg);
        }
    }

    virtual void write(const std::string& msg) = 0;
};

// Конкретные обработчики
class ConsoleLogger : public Logger {
public:
    ConsoleLogger(LogLevel level) : Logger(level) {}
    void write(const std::string& msg) override {
        std::cout << "Console: " << msg << "\n";
    }
};

class FileLogger : public Logger {
public:
    FileLogger(LogLevel level) : Logger(level) {}
    void write(const std::string& msg) override {
        std::cout << "File: " << msg << "\n";
    }
};

class EmailLogger : public Logger {
public:
    EmailLogger(LogLevel level) : Logger(level) {}
    void write(const std::string& msg) override {
        std::cout << "Email: " << msg << "\n";
    }
};

int main() {
    auto console = std::make_unique<ConsoleLogger>(INFO);
    auto file = std::make_unique<FileLogger>(WARNING);
    auto email = std::make_unique<EmailLogger>(ERROR);

    console->setNext(std::move(file));
    file->setNext(std::move(email));

    console->log(INFO, "Info message");
    console->log(ERROR, "Critical error");
}
```
**Вывод:**  
```
Console: Info message  
Console: Critical error  
File: Critical error  
Email: Critical error  
```

#### **3. Компоненты и связи**  
- **Handler (Logger)**  
  Базовый интерфейс с методом `log()` и ссылкой на следующий обработчик.  
- **Concrete Handler (ConsoleLogger, FileLogger, EmailLogger)**  
  Реализует обработку запроса (логирование) для своего уровня.  
- **Client**  
  Строит цепочку и инициирует запрос.  

#### **4. Плюсы паттерна**  
- **Гибкость**: Динамическое изменение цепочки.  
- **Разделение ответственности**: Каждый обработчик решает свою задачу.  
- **Слабая связанность**: Отправитель не знает конечного обработчика.  

#### **5. Когда использовать?**  
- Обработка событий с разными уровнями важности (логирование, исключения).  
- Фильтрация HTTP-запросов (middleware).  
- Проверка данных через цепочку валидаторов.  

#### **6. Подводные камни и решения**  
**Минусы:**  
- **Нет гарантии обработки**: Запрос может пройти всю цепочку без обработки.  
- **Производительность**: Рекурсивные вызовы могут быть затратными.  

**Способы решения:**  
- Добавить обработчик-заглушку для "непойманных" запросов.  
- Оптимизировать цепочку (например, кешировать результаты).  

#### **7. Альтернативы и отличия от других паттернов**  

| **Паттерн**       | **Отличие от Chain of Responsibility**                     |
|-------------------|-----------------------------------------------------------|
| **Команда**       | Инкапсулирует запрос в объект, а не передаёт по цепочке.  |
| **Посредник**     | Централизует логику, а не распределяет её.                |
| **Декоратор**     | Добавляет функциональность объекту, а не обрабатывает запросы. |

#### **8. Примеры в реальных проектах**  
- **Логирование**: Фильтрация сообщений по уровню (log4j).  
- **Веб-фреймворки**: Middleware в Express.js или Django.  
- **Игры**: Обработка событий ввода (клавиатура, мышь).  

**Идеальный ответ:**  
*"Паттерн Chain of Responsibility позволяет передавать запросы по цепочке обработчиков, где каждый решает, может ли он обработать запрос. Используется для логирования, валидации или middleware. Состоит из базового обработчика (Handler), конкретных обработчиков (Concrete Handler) и клиента. Пример — система логирования с разными уровнями (INFO, ERROR). Плюсы: гибкость и слабая связанность. Минус: нет гарантии обработки. Отличается от Команды (инкапсуляция запроса) и Посредника (централизация логики)."*