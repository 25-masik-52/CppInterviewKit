#### **1. Основные подходы**
1. **Статический счетчик экземпляров** - базовый способ с подсчетом объектов
2. **Синглтон** - частный случай (лимит = 1)
3. **Фабричный метод + friend-класс** - строгий контроль создания
4. **CRTP** - шаблонный подход для гибких ограничений
5. **Умные указатели** - контроль через shared_ptr/weak_ptr

#### **2. Критерии выбора метода**
| Метод               | Когда использовать                  | Плюсы                        | Минусы                  |
| ------------------- | ----------------------------------- | ---------------------------- | ----------------------- |
| Статический счетчик | Простые случаи, небольшой лимит     | Простота реализации          | Нет контроля в runtime  |
| Синглтон            | Требуется ровно 1 экземпляр         | Гарантия единственности      | Глобальное состояние    |
| Фабрика + friend    | Строгий контроль создания           | Гибкость                     | Усложненная архитектура |
| CRTP                | Гибкие лимиты для семейства классов | Повторное использование кода | Сложность шаблонов      |
| Умные указатели     | Управление временем жизни           | Автоматический контроль      | Накладные расходы       |

#### **3. Ключевые моменты реализации**
1. **Для всех подходов:**
   - Запрет копирования (`= delete`)
   - Потокобезопасность (мьютексы для счетчиков)
   
2. **Особенности:**
   - Синглтон лучше возвращать по ссылке, а не указателю
   - CRTP позволяет задавать лимит через параметр шаблона
   - Фабричный метод дает максимальный контроль

#### **4. Примеры кода**

**1. Статический счетчик:**
```cpp
class Limited {
    static inline int count = 0;  // C++17 inline переменная
    static constexpr int MAX = 3;
public:
    Limited() { if (++count > MAX) throw std::bad_alloc(); }
    ~Limited() { --count; }
};
```

**2. Потокобезопасный синглтон (C++11):**
```cpp
class Singleton {
    static std::atomic<Singleton*> instance;
    static std::mutex mtx;
public:
    static Singleton& get() {
        if (!instance.load()) {
            std::lock_guard lock(mtx);
            if (!instance.load()) {
                instance.store(new Singleton());
            }
        }
        return *instance;
    }
};
```

**3. Использование friend-класса:**
```c++
class Restricted {
private:
    Restricted() = default;
    friend class Factory;

    static int count;
    static const int MAX = 2;
};

class Factory {
public:
    static Restricted* create() {
        if (Restricted::count >= Restricted::MAX) {
            return nullptr;
        }
        ++Restricted::count;
        return new Restricted();
    }
};

int Restricted::count = 0;
```

**4. CRTP (Curiously Recurring Template Pattern):**
```c++
template<unsigned max>
class InstanceLimited {
protected:
    static unsigned count;
    
    InstanceLimited() {
        if (count >= max)
            throw std::bad_alloc();
        ++count;
    }
    
    ~InstanceLimited() { --count; }
};

template<unsigned max>
unsigned InstanceLimited<max>::count = 0;

// Класс с ограничением в 5 экземпляров
class MyClass : public InstanceLimited<5> {
public:
    MyClass() = default;
};
```

**5. Использование std::shared_ptr и weak_ptr:**
```c++
class Controlled {
private:
    static std::weak_ptr<Controlled> instance;
    
    Controlled() = default;

public:
    static std::shared_ptr<Controlled> create() {
        if (auto ptr = instance.lock()) {
            return ptr;
        }
        auto ptr = std::shared_ptr<Controlled>(new Controlled());
        instance = ptr;
        return ptr;
    }
};

std::weak_ptr<Controlled> Controlled::instance;
```

#### **5. Альтернативы**
1. Для простых случаев - **статический счетчик**
2. Для enterprise-решений - **фабрика + dependency injection**
3. В шаблонных библиотеках - **CRTP**
4. Для ресурсоемких объектов - **умные указатели**

**Идеальный ответ:**
*«В C++ есть несколько способов ограничения экземпляров: от простого счетчика до шаблонного CRTP. Выбор зависит от требований - для базовых случаев хватит статического счетчика с проверкой в конструкторе, для сложных сценариев используйте фабрики или умные указатели. Главное - не забывать про потокобезопасность и запрет копирования.»*