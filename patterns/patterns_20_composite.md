#### **1. Определение**
**Компоновщик (Composite)** — это структурный паттерн, который организует объекты в **древовидные структуры**, позволяя клиентам единообразно работать как с отдельными объектами, так и с их группами.

**Ключевая идея**:
> *"Одинаковый интерфейс для отдельных объектов и их композиций"*

#### **2. Реализация на C++ (Файловая система)**

**1. Базовый компонент**
```cpp
class FileSystemComponent {
public:
    virtual ~FileSystemComponent() = default;
    virtual void display(int depth = 0) const = 0;
    virtual void add(std::unique_ptr<FileSystemComponent>)
        { throw std::runtime_error("Не поддерживается"); }
};
```

**2. Лист (файл)**
```cpp
class File : public FileSystemComponent {
    std::string name_;
public:
    File(std::string name) : name_(std::move(name)) {}

    void display(int depth) const override {
        std::cout << std::string(depth, ' ') << "📄 " << name_ << "\n";
    }
};
```

**3. Композит (папка)**
```cpp
class Folder : public FileSystemComponent {
    std::string name_;
    std::vector<std::unique_ptr<FileSystemComponent>> children_;

public:
    Folder(std::string name) : name_(std::move(name)) {}

    void add(std::unique_ptr<FileSystemComponent> component) override {
        children_.push_back(std::move(component));
    }

    void display(int depth) const override {
        std::cout << std::string(depth, ' ') << "📁 " << name_ << "\n";
        for (const auto& child : children_) {
            child->display(depth + 2);
        }
    }
};
```

**4. Клиентский код**
```cpp
int main() {
    auto root = std::make_unique<Folder>("Root");
    auto docs = std::make_unique<Folder>("Documents");
    auto img = std::make_unique<Folder>("Images");

    docs->add(std::make_unique<File>("resume.pdf"));
    img->add(std::make_unique<File>("photo.jpg"));
    
    root->add(std::move(docs));
    root->add(std::move(img));
    root->display();
}
```

**Вывод**:
```
📁 Root
  📁 Documents
    📄 resume.pdf
  📁 Images
    📄 photo.jpg
```

#### **3. Плюсы и минусы**

| **Преимущества**                   | **Недостатки**                                 |
| ---------------------------------- | ---------------------------------------------- |
| ✅ Упрощает клиентский код          | ❌ Нарушает SRP (единый интерфейс)              |
| ✅ Легко добавлять новые компоненты | ❌ Может скрывать различия между объектами      |
| ✅ Гибкость структуры               | ❌ Ограниченный контроль над типами компонентов |

#### **4. Когда использовать?**
- **Графические редакторы** (группировка фигур)  
- **UI-интерфейсы** (вложенные компоненты)  
- **Игровые объекты** (сцены, инвентарь)  

#### **5. Отличие от других паттернов**
| **Паттерн** | **Ключевое отличие** |
|------------|---------------------|
| **Декоратор** | Добавляет функциональность |
| **Посетитель** | Отделяет алгоритмы от структуры |
| **Фасад** | Упрощает сложную систему |

#### **6. Реальные примеры**
1. **UI-фреймворки**:
```cpp
class Widget {
    std::vector<std::unique_ptr<Widget>> children;
};
```

2. **Игровые движки**:
```cpp
class GameObject {
    std::vector<GameObject*> children;
};
```

#### **7. Best Practices**
1. Используйте **std::unique_ptr** для управления памятью
2. Реализуйте **метод remove()** для удаления компонентов
3. Добавьте **итератор** для обхода дерева

**Идеальный ответ**:
*"Паттерн Компоновщик позволяет единообразно работать с древовидными структурами, где одни объекты могут содержать другие. Состоит из: 1) Component (базовый интерфейс), 2) Leaf (конечные объекты), 3) Composite (группы объектов). Преимущества: упрощение кода клиента, гибкость структуры. Недостатки: общий интерфейс может нарушать SRP. Применяется в файловых системах, UI и игровых объектах."*