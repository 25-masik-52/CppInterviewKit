#### **1. Определение**  
**Паттерн Посетитель (Visitor)** — это поведенческий паттерн, который позволяет добавлять новые операции к объектам, не изменяя их классов. Он разделяет алгоритмы и структуры данных, вынося логику операций в отдельные классы-посетители.

#### **2. Код-пример**  
**Пример: Генерация отчётов для документов**  
```cpp
#include <iostream>
#include <vector>

// Предварительные объявления
class PDFDocument;
class WordDocument;

// Интерфейс посетителя
class DocumentVisitor {
public:
    virtual ~DocumentVisitor() = default;
    virtual void visit(PDFDocument& doc) = 0;
    virtual void visit(WordDocument& doc) = 0;
};

// Базовый класс документа
class Document {
public:
    virtual ~Document() = default;
    virtual void accept(DocumentVisitor& visitor) = 0;
};

// Конкретные документы
class PDFDocument : public Document {
public:
    void accept(DocumentVisitor& visitor) override {
        visitor.visit(*this);
    }
    std::string getContent() const { return "PDF content"; }
};

class WordDocument : public Document {
public:
    void accept(DocumentVisitor& visitor) override {
        visitor.visit(*this);
    }
    std::string getContent() const { return "Word content"; }
};

// Конкретные посетители
class HTMLExporter : public DocumentVisitor {
public:
    void visit(PDFDocument& doc) override {
        std::cout << "Exporting PDF to HTML: " << doc.getContent() << "\n";
    }
    void visit(WordDocument& doc) override {
        std::cout << "Exporting Word to HTML: " << doc.getContent() << "\n";
    }
};

class TextExporter : public DocumentVisitor {
public:
    void visit(PDFDocument& doc) override {
        std::cout << "Exporting PDF to text: " << doc.getContent() << "\n";
    }
    void visit(WordDocument& doc) override {
        std::cout << "Exporting Word to text: " << doc.getContent() << "\n";
    }
};

int main() {
    PDFDocument pdf;
    WordDocument word;

    HTMLExporter html;
    TextExporter text;

    pdf.accept(html);  // Exporting PDF to HTML
    word.accept(text); // Exporting Word to text
}
```
**Вывод:**  
```
Exporting PDF to HTML: PDF content  
Exporting Word to text: Word content  
```

#### **3. Компоненты и связи**  
- **Visitor (DocumentVisitor)**  
  Интерфейс с методами `visit` для каждого типа элемента.  
- **Concrete Visitor (HTMLExporter, TextExporter)**  
  Реализует операции для каждого типа элемента.  
- **Element (Document)**  
  Определяет метод `accept`, принимающий посетителя.  
- **Concrete Element (PDFDocument, WordDocument)**  
  Вызывает соответствующий метод посетителя.  

#### **4. Плюсы паттерна**  
- **Открытость/закрытость**: Новые операции без изменения классов.  
- **Локализация кода**: Все операции одного типа в одном классе.  
- **Обход сложных структур**: Удобен для деревьев и графов.  

#### **5. Когда использовать?**  
- При множестве несвязанных операций над объектами.  
- Для генерации отчётов/экспорта в разных форматах.  
- В компиляторах (анализ AST).  

#### **6. Подводные камни и решения**  
**Минусы:**  
- **Нарушение инкапсуляции**: Посетитель может требовать доступа к приватным полям.  
- **Сложность добавления элементов**: Новый тип элемента требует изменений во всех посетителях.  

**Способы решения:**  
- Использовать дружественные классы для контролируемого доступа.  
- Применять только для стабильных иерархий элементов.  

#### **7. Альтернативы и отличия от других паттернов**  

| **Паттерн**       | **Отличие от Visitor**                     |
|-------------------|--------------------------------------------|
| **Итератор**      | Обходит элементы без выполнения операций.  |
| **Стратегия**     | Меняет алгоритм одного объекта.            |
| **Декоратор**     | Добавляет функциональность объекту.        |

#### **9. Примеры в реальных проектах**  
- **Компиляторы**: Анализ абстрактного синтаксического дерева.  
- **Генераторы кода**: Преобразование структур данных в SQL/XML.  
- **Графические редакторы**: Операции над элементами canvas.  

**Идеальный ответ:**  
*"Паттерн Visitor позволяет добавлять операции к объектам, не изменяя их классов. Состоит из Visitor (интерфейс операций), ConcreteVisitor (реализации операций), Element (метод accept) и ConcreteElement. Пример — экспорт документов в разные форматы. Плюсы: гибкость добавления операций. Минусы: нарушение инкапсуляции. Отличается от Итератора (обход без операций) и Стратегии (один алгоритм)."*