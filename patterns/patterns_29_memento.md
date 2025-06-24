#### **1. Определение**  
**Паттерн Хранитель (Memento)** — это поведенческий паттерн, позволяющий сохранять и восстанавливать предыдущие состояния объекта без нарушения его инкапсуляции. Основное применение — реализация механизмов отмены (undo) и возврата (redo) действий.

#### **2. Код-пример**  
**Пример: Текстовый редактор с функцией отмены**  
```cpp
#include <iostream>
#include <vector>
#include <memory>

// Создатель (Originator)
class TextDocument {
    std::string content_;
public:
    void write(const std::string& text) {
        content_ += text;
    }
    std::string getContent() const {
        return content_;
    }

    // Внутренний класс Memento
    class Memento {
        std::string state_;
        friend class TextDocument;
        Memento(const std::string& state) : state_(state) {}
    public:
        const std::string& getState() const { return state_; }
    };

    std::unique_ptr<Memento> createMemento() const {
        return std::make_unique<Memento>(content_);
    }

    void restoreFromMemento(const Memento* memento) {
        content_ = memento->getState();
    }
};

// Опекун (Caretaker)
class History {
    std::vector<std::unique_ptr<TextDocument::Memento>> states_;
public:
    void push(std::unique_ptr<TextDocument::Memento> memento) {
        states_.push_back(std::move(memento));
    }
    std::unique_ptr<TextDocument::Memento> pop() {
        if (states_.empty()) return nullptr;
        auto last = std::move(states_.back());
        states_.pop_back();
        return last;
    }
};

int main() {
    TextDocument doc;
    History history;

    doc.write("Hello");
    history.push(doc.createMemento());

    doc.write(" World!");
    std::cout << "Current: " << doc.getContent() << "\n";

    // Отмена последнего изменения
    if (auto memento = history.pop()) {
        doc.restoreFromMemento(memento.get());
        std::cout << "After undo: " << doc.getContent() << "\n";
    }
}
```
**Вывод:**  
```
Current: Hello World!
After undo: Hello
```

#### **3. Компоненты и связи**  
- **Originator (TextDocument)**  
  Объект, состояние которого нужно сохранять и восстанавливать.  
- **Memento**  
  Неизменяемый снимок состояния Originator.  
- **Caretaker (History)**  
  Управляет историей состояний (сохраняет и восстанавливает Memento).  

#### **4. Плюсы паттерна**  
- **Сохранение инкапсуляции**: Доступ к состоянию только у Originator.  
- **Простота отмены операций**: Легко реализовать undo/redo.  
- **Гибкость**: Можно сохранять разные аспекты состояния.  

#### **5. Когда использовать?**  
- Редакторы с историей изменений.  
- Игры с системой сохранений.  
- Транзакционные операции (откат изменений).  

#### **6. Подводные камни и решения**  
**Минусы:**  
- **Потребление памяти**: Хранение многих состояний может быть затратным.  
- **Сложность**: Для сложных объектов создание Memento нетривиально.  

**Способы решения:**  
- Оптимизировать хранение (например, дельта-компрессия).  
- Использовать паттерн **Прототип** для клонирования состояний.  

#### **7. Альтернативы и отличия от других паттернов**  

| **Паттерн**       | **Отличие от Memento**                     |
|-------------------|--------------------------------------------|
| **Команда**       | Хранит действия, а не состояния.           |
| **Прототип**      | Клонирует объекты, а не сохраняет состояния. |
| **Хранитель (Snapshot)** | Оптимизирован для частых сохранений.       |

#### **8. Примеры в реальных проектах**  
- **Графические редакторы**: Photoshop (история изменений).  
- **Игры**: Сохранение позиций персонажей.  
- **Системы контроля версий**: Git (коммиты как Memento).  

**Идеальный ответ:**  
*"Паттерн Memento сохраняет состояния объекта (Originator) в специальных объектах-снимках (Memento), которые управляются опекуном (Caretaker). Это позволяет реализовать отмену операций, сохраняя инкапсуляцию. Пример — текстовый редактор с undo. Плюсы: соблюдение инкапсуляции и простота отмены. Минусы: затраты памяти. Отличается от Команды (хранит действия) и Прототипа (клонирование объектов)."*