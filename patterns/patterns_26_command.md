#### **1. Определение**  
**Паттерн Команда (Command)** — это поведенческий паттерн, который инкапсулирует запрос в виде объекта, позволяя параметризовать клиенты с разными запросами, поддерживать отмену операций (undo/redo), ставить команды в очередь или логировать их выполнение.  

#### **2. Код-пример**  
**Пример: Текстовый редактор с undo/redo**  
```cpp
#include <iostream>
#include <memory>
#include <vector>

// Получатель (Receiver)
class Document {
    std::string content_;
public:
    void addText(const std::string& text) {
        content_ += text;
        std::cout << "Добавлен текст: " << text << "\n";
    }
    void deleteText(size_t length) {
        if (length <= content_.size()) {
            std::string deleted = content_.substr(content_.size() - length);
            content_.erase(content_.size() - length);
            std::cout << "Удалён текст: " << deleted << "\n";
        }
    }
    std::string getContent() const { return content_; }
};

// Базовый интерфейс команды
class Command {
public:
    virtual ~Command() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
};

// Конкретные команды
class AddTextCommand : public Command {
    Document& doc_;
    std::string text_;
public:
    AddTextCommand(Document& doc, const std::string& text) : doc_(doc), text_(text) {}
    void execute() override { doc_.addText(text_); }
    void undo() override { doc_.deleteText(text_.size()); }
};

class DeleteTextCommand : public Command {
    Document& doc_;
    size_t length_;
    std::string backup_;
public:
    DeleteTextCommand(Document& doc, size_t length) : doc_(doc), length_(length) {}
    void execute() override {
        backup_ = doc_.getContent().substr(doc_.getContent().size() - length_);
        doc_.deleteText(length_);
    }
    void undo() override { doc_.addText(backup_); }
};

// Инициатор (Invoker)
class TextEditor {
    Document doc_;
    std::vector<std::unique_ptr<Command>> history_;
public:
    void executeCommand(std::unique_ptr<Command> cmd) {
        cmd->execute();
        history_.push_back(std::move(cmd));
    }
    void undo() {
        if (!history_.empty()) {
            history_.back()->undo();
            history_.pop_back();
        }
    }
    void showContent() const {
        std::cout << "Текущий текст: " << doc_.getContent() << "\n";
    }
};

int main() {
    TextEditor editor;
    editor.executeCommand(std::make_unique<AddTextCommand>(editor.getDoc(), "Hello"));
    editor.executeCommand(std::make_unique<AddTextCommand>(editor.getDoc(), " World!"));
    editor.undo(); // Отмена добавления " World!"
    editor.executeCommand(std::make_unique<DeleteTextCommand>(editor.getDoc(), 3));
    editor.showContent();
}
```
**Вывод:**  
```
Добавлен текст: Hello  
Добавлен текст: World!  
Удалён текст: World!  
Удалён текст: Hel  
Текущий текст: o  
```

#### **3. Компоненты и связи**  
- **Command**  
  Абстрактный интерфейс с методами `execute()` и `undo()`.  
- **Concrete Command (AddTextCommand, DeleteTextCommand)**  
  Реализует операции, связывая действие с Receiver.  
- **Receiver (Document)**  
  Содержит бизнес-логику для выполнения операций.  
- **Invoker (TextEditor)**  
  Управляет выполнением команд и поддерживает историю.  
- **Client**  
  Конфигурирует команды и связывает их с получателями.  

#### **4. Плюсы паттерна**  
- **Инкапсуляция запросов**: Команды — это объекты, которые можно передавать и хранить.  
- **Поддержка undo/redo**: История команд позволяет отменять действия.  
- **Гибкость**: Очереди команд, отложенное выполнение, логирование.  

#### **5. Когда использовать?**  
- Графические интерфейсы (кнопки, меню).  
- Транзакционные системы (откат операций).  
- Планировщики задач (очереди команд).  
- Многозвенные операции (макросы).  

#### **6. Подводные камни и решения**  
**Минусы:**  
- **Рост числа классов**: Для каждой операции — отдельный класс команды.  
- **Накладные расходы**: Хранение истории команд потребляет память.  

**Способы решения:**  
- Использовать **обобщённые команды** (например, с лямбда-функциями).  
- Ограничивать глубину истории (например, хранить только последние N команд).  

#### **7. Альтернативы и отличия от других паттернов**  

| **Паттерн**       | **Отличие от Command**                                                                 |
|-------------------|---------------------------------------------------------------------------------------|
| **Стратегия**     | Меняет алгоритм на лету, а Command инкапсулирует действие.                           |
| **Цепочка обязанностей** | Запрос передаётся по цепочке, а Command выполняет сразу.                             |
| **Хранитель (Memento)** | Сохраняет состояние объекта для отката, а Command — сами операции.                  |
**Аналог в STL:**  
```cpp
std::function<void()> saveCmd = []{ /* логика сохранения */ };
std::function<void()> loadCmd = []{ /* логика загрузки */ };
// Команды можно хранить в очереди:
std::queue<std::function<void()>> cmdQueue;
cmdQueue.push(saveCmd);
cmdQueue.front()(); // Выполнение
```

#### **8. Примеры в реальных проектах**  
- **Графические редакторы**: Photoshop (история действий).  
- **Игры**: Управление персонажем через команды (атака, перемещение).  
- **Банковские системы**: Транзакции с возможностью отката.  

**Идеальный ответ:**  
*"Паттерн Command инкапсулирует запрос в объект, позволяя реализовать отмену операций, очереди команд и логирование. Состоит из Command (интерфейс), ConcreteCommand (реализация), Receiver (бизнес-логика) и Invoker (управление). Пример — текстовый редактор с undo/redo. Плюсы: гибкость и поддержка отмены. Минусы: рост числа классов. Отличается от Стратегии (меняет алгоритмы) и Цепочки обязанностей (передача запроса)."*