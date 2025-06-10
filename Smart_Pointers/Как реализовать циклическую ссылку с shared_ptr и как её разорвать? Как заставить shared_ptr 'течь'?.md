#### **Циклические ссылки и проблема утечки памяти**

Циклические ссылки между `shared_ptr` могут приводить к утечкам памяти, так как счетчики ссылок никогда не достигают нуля. Рассмотрим реализацию и решение этой проблемы.

**Пример циклической ссылки**
```cpp
#include <memory>
#include <iostream>

class B; // Предварительное объявление

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::shared_ptr<A> a_ptr;
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    // Создаем циклическую ссылку
    a->b_ptr = b;
    b->a_ptr = a;
    
    // При выходе из области видимости деструкторы не вызовутся!
    // Утечка памяти
}
```

#### **Как разорвать циклическую ссылку?**

**1. Использование `weak_ptr`**
`weak_ptr` - это "слабая" ссылка, которая не увеличивает счетчик ссылок:
```cpp
class B;

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::weak_ptr<A> a_ptr; // Используем weak_ptr вместо shared_ptr
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;
    b->a_ptr = a; // weak_ptr не создает циклической ссылки
    
    // Теперь объекты будут корректно уничтожены
}
```

**2. Ручной разрыв ссылки**
```cpp
int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;
    b->a_ptr = a;
    
    // Ручной разрыв цикла перед выходом
    a->b_ptr.reset();
    // или b->a_ptr.reset();
    
    // Теперь объекты уничтожатся
}
```

#### **Как заставить `shared_ptr` "течь" (создать утечку)?**
Для демонстрационных целей можно создать ситуацию, когда память не освобождается:
```cpp
void create_leak() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;
    b->a_ptr = a;
    
    // Утечка: объекты a и b не будут уничтожены,
    // так как имеют циклические ссылки
}

int main() {
    create_leak();
    // Память, выделенная для A и B, не освобождена
}
```

**Дополнительные методы работы с циклическими ссылками**

**3. Использование сырых указателей для "родительских" ссылок**
```cpp
class B;

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    A* a_ptr; // Сырой указатель на родителя
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    
    a->b_ptr = b;
    b->a_ptr = a.get(); // Сырой указатель не увеличивает счетчик
    
    // Нет утечки памяти
}
```

**4. Использование `enable_shared_from_this`**
Для сложных случаев, когда объекту нужно создать `shared_ptr` на самого себя:
```cpp
class A : public std::enable_shared_from_this<A> {
public:
    void setup() {
        b_ptr->a_ptr = shared_from_this(); // Безопасное создание shared_ptr
    }
    std::shared_ptr<B> b_ptr;
};
```

**Идеальный ответ:**  
*"Циклические ссылки между `shared_ptr` возникают, когда объекты взаимно владеют друг другом через `shared_ptr`, что предотвращает освобождение памяти (счётчики ссылок не обнуляются). Для решения проблемы необходимо разорвать цикл, заменив один из `shared_ptr` на `weak_ptr` (который не увеличивает счётчик ссылок) либо вручную вызвав `reset()` перед удалением объектов. Для принудительной утечки памяти достаточно создать такую циклическую зависимость и не разрывать её.
**Пример:** два класса, `A` и `B`, хранящие `shared_ptr` друг на друга, — замена одного из указателей на `weak_ptr` устраняет утечку.
**Вывод:** `weak_ptr` — стандартный способ борьбы с циклическими зависимостями в `shared_ptr`.*"