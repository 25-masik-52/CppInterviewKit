#### **1. Определение**  
**Паттерн Приспособленец (Flyweight)** — это структурный паттерн проектирования, который оптимизирует использование памяти за счёт разделения общего состояния между множеством объектов. Он применяется, когда программа работает с большим количеством объектов, имеющих повторяющиеся данные.  

#### **2. Код-пример**  
**Пример: Текстовый редактор (символы)**  
```cpp
#include <iostream>
#include <unordered_map>
#include <vector>

// Flyweight (разделяемое состояние)
class Character {
private:
    char symbol_;
    int fontSize_;

public:
    Character(char symbol, int fontSize) : symbol_(symbol), fontSize_(fontSize) {}

    void print() const {
        std::cout << "Symbol: " << symbol_ << ", FontSize: " << fontSize_ << std::endl;
    }
};

// Flyweight Factory (управление кешем)
class CharacterFactory {
private:
    std::unordered_map<char, Character*> characters_;

public:
    Character* getCharacter(char symbol, int fontSize) {
        if (characters_.find(symbol) == characters_.end()) {
            characters_[symbol] = new Character(symbol, fontSize);
        }
        return characters_[symbol];
    }

    ~CharacterFactory() {
        for (auto& pair : characters_) {
            delete pair.second;
        }
    }
};

// Клиент (использование Flyweight)
class TextDocument {
private:
    CharacterFactory& factory_;
    std::vector<Character*> characters_;

public:
    TextDocument(CharacterFactory& factory) : factory_(factory) {}

    void addCharacter(char symbol, int fontSize) {
        characters_.push_back(factory_.getCharacter(symbol, fontSize));
    }

    void print() const {
        for (auto* character : characters_) {
            character->print();
        }
    }
};

int main() {
    CharacterFactory factory;
    TextDocument doc(factory);

    doc.addCharacter('A', 12);
    doc.addCharacter('B', 14);
    doc.addCharacter('A', 12); // Берётся из кеша

    doc.print();
}
```
**Вывод:**  
```
Symbol: A, FontSize: 12  
Symbol: B, FontSize: 14  
Symbol: A, FontSize: 12  
```  

#### **3. Компоненты и связи**  
- **Flyweight (Character)**  
  Содержит **внутреннее состояние** (разделяемое, например, символ и размер шрифта). 
- **Concrete Flyweight**  
  Конкретная реализация (в примере — класс `Character`).  
- **Flyweight Factory (CharacterFactory)**  
  Управляет созданием и кешированием объектов.  
- **Client (TextDocument)**  
  Хранит **внешнее состояние** (позиция символа) и использует Flyweight.  

#### **4. Плюсы паттерна**  
- **Экономия памяти**: Общее состояние хранится один раз.  
- **Ускорение работы**: Меньше объектов — меньше накладных расходов.  
- **Масштабируемость**: Позволяет работать с тысячами объектов.  

#### **5. Когда использовать?**  
- Большое количество объектов с повторяющимся состоянием (например, деревья в игре).  
- Оптимизация ОЗУ (кеширование ресурсов: текстуры, шрифты).  
- Графические редакторы, текстовые процессоры.  

#### **6. Подводные камни и решения**  
**Минусы:**  
- **Усложнение кода**: Требуется разделение состояния на внутреннее/внешнее.  
- **Потенциальные утечки памяти**: Если Flyweight содержит ссылки на внешние ресурсы. 

**Способы решения:**  
- Использовать умные указатели (`std::shared_ptr`) для автоматического управления памятью.  
- Чётко разделять ответственность между клиентом и Flyweight.

#### **7. Альтернативы**
|**Паттерн**|**Отличие от Flyweight**|
|---|---|
|**Синглтон (Singleton)**|Гарантирует единственный экземпляр на всю программу, а Flyweight — множество объектов с общим состоянием.|
|**Фабрика (Factory)**|Создаёт объекты, но не управляет их повторным использованием (нет кеширования).|
|**Прототип (Prototype)**|Клонирует объекты, а Flyweight разделяет их состояние.|
|**Кеш (Cache)**|Flyweight — частный случай кеша, но специализирован для объектов с разделяемым состоянием.|
|**Адаптер (Adapter)**|Связывает несовместимые интерфейсы, а Flyweight оптимизирует память.|
|**Стратегия (Strategy)**|Меняет поведение объекта, а Flyweight — структуру хранения данных.|
|**Фасад (Facade)**|Упрощает сложную систему, а Flyweight — уменьшает нагрузку на память.|

**Идеальный ответ:**  
*"Паттерн Flyweight оптимизирует память, разделяя общее состояние между множеством объектов. Он состоит из Flyweight (внутреннее состояние), Flyweight Factory (управление кешем) и Client (внешнее состояние). Применяется для работы с большим количеством объектов, например, в текстовых редакторах или играх. Основные плюсы — экономия памяти и ускорение работы, но требует аккуратного разделения состояния. В C++ можно реализовать через фабрику с кешированием, как в примере с символами."*