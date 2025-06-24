#### **1. Определение**  
**Паттерн Итератор (Iterator)** — это поведенческий паттерн, предоставляющий унифицированный интерфейс для последовательного доступа к элементам коллекции без раскрытия её внутренней структуры.  

#### **2. Код-пример**  
**Пример: Итератор для пользовательского массива**  
```cpp
#include <iostream>
#include <algorithm>

// Интерфейс итератора
template <typename T>
class Iterator {
public:
    virtual ~Iterator() = default;
    virtual T next() = 0;
    virtual bool hasNext() const = 0;
};

// Коллекция (массив)
template <typename T>
class ArrayCollection {
    T* data_;
    size_t size_;
    size_t capacity_;

    void resize() {
        capacity_ *= 2;
        T* newData = new T[capacity_];
        std::copy(data_, data_ + size_, newData);
        delete[] data_;
        data_ = newData;
    }

public:
    ArrayCollection() : size_(0), capacity_(10), data_(new T[capacity_]) {}
    ~ArrayCollection() { delete[] data_; }

    void add(const T& item) {
        if (size_ >= capacity_) resize();
        data_[size_++] = item;
    }

    Iterator<T>* createIterator() const;
};

// Конкретный итератор
template <typename T>
class ArrayIterator : public Iterator<T> {
    const ArrayCollection<T>& collection_;
    size_t index_ = 0;

public:
    ArrayIterator(const ArrayCollection<T>& coll) : collection_(coll) {}

    T next() override {
        return collection_.data_[index_++];
    }

    bool hasNext() const override {
        return index_ < collection_.size_;
    }
};

template <typename T>
Iterator<T>* ArrayCollection<T>::createIterator() const {
    return new ArrayIterator<T>(*this);
}

int main() {
    ArrayCollection<std::string> collection;
    collection.add("Iterator");
    collection.add("Pattern");
    collection.add("Demo");

    Iterator<std::string>* it = collection.createIterator();
    while (it->hasNext()) {
        std::cout << it->next() << " ";
    }
    delete it;
}
```
**Вывод:**  
```
Iterator Pattern Demo 
```  

#### **3. Компоненты и связи**  
- **Iterator**  
  Абстрактный интерфейс с методами `next()` и `hasNext()`.  
- **Concrete Iterator (ArrayIterator)**  
  Реализует логику обхода конкретной коллекции.  
- **Aggregate (ArrayCollection)**  
  Интерфейс для создания итератора (`createIterator()`).  
- **Concrete Aggregate**  
  Коллекция, возвращающая конкретный итератор.  

#### **4. Плюсы паттерна**  
- **Унификация доступа**: Единый интерфейс для разных коллекций.  
- **Инкапсуляция**: Скрывает внутреннюю структуру данных.  
- **Гибкость**: Поддержка разных алгоритмов обхода (прямой/обратный, фильтрация).  

#### **5. Когда использовать?**  
- Обход сложных структур (деревья, графы).  
- Необходимость скрыть реализацию коллекции.  
- Ленивая загрузка данных (например, из базы данных).  

#### **6. Подводные камни и решения**  
**Минусы:**  
- **Оверхед**: Для простых коллекций может быть избыточным.  
- **Безопасность**: Итераторы могут стать невалидными при изменении коллекции.  

**Способы решения:**  
- Использовать **STL-итераторы** для стандартных коллекций.  
- Реализовать **механизм инвалидации** итераторов при изменении данных.  

#### **7. Альтернативы и отличия от других паттернов**  

| **Паттерн**     | **Отличие от Iterator**                   |
| --------------- | ----------------------------------------- |
| **Посетитель**  | Добавляет операции к элементам коллекции. |
| **Компоновщик** | Работает с древовидными структурами.      |
| **Фабрика**     | Создаёт объекты, а не инструменты обхода. |

#### **8. Примеры в реальных проектах**  
- **STL**: Итераторы для `std::vector`, `std::map`.  
- **Базы данных**: Обход результатов запроса.  
- **Графические движки**: Обход объектов сцены.  

**Идеальный ответ:**  
*"Паттерн Iterator предоставляет единый интерфейс для обхода коллекций, скрывая их внутреннюю структуру. Состоит из Iterator (интерфейс), ConcreteIterator (реализация обхода), Aggregate (интерфейс коллекции) и ConcreteAggregate. Пример — итератор для пользовательского массива. Плюсы: унификация доступа и инкапсуляция. Минусы: оверхед для простых случаев. В C++ уже реализован в STL (begin(), end())."*