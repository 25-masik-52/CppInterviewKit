#### **1. Определение**  
**Наблюдатель (Observer)** — поведенческий паттерн, где объекты (**наблюдатели**) подписываются на события **субъекта** и автоматически уведомляются об изменениях.  
**Без STL** реализуется через ручное управление памятью и структурами данных.

#### **2. Реализация**

##### **Шаг 1: Интерфейс наблюдателя**
```cpp
class Observer {
public:
    virtual ~Observer() = default;
    virtual void update(int value) = 0;  // Метод для уведомлений
};
```

##### **Шаг 2: Субъект с динамическим массивом**
```cpp
class Subject {
private:
    Observer** observers;  // Массив указателей
    int capacity;         // Размер массива
    int count;            // Текущее кол-во наблюдателей

    void resize() {
        capacity *= 2;
        Observer** newArr = new Observer*[capacity];
        for (int i = 0; i < count; ++i) {
            newArr[i] = observers[i];
        }
        delete[] observers;
        observers = newArr;
    }

public:
    Subject(int initialCapacity = 10) 
        : capacity(initialCapacity), count(0) {
        observers = new Observer*[capacity];
    }

    ~Subject() { delete[] observers; }

    void attach(Observer* observer) {
        if (count >= capacity) resize();
        observers[count++] = observer;
    }

    void detach(Observer* observer) {
        for (int i = 0; i < count; ++i) {
            if (observers[i] == observer) {
                for (int j = i; j < count - 1; ++j) {
                    observers[j] = observers[j + 1];
                }
                count--;
                break;
            }
        }
    }

    void notify(int value) {
        for (int i = 0; i < count; ++i) {
            observers[i]->update(value);
        }
    }
};
```

##### **Шаг 3: Конкретный наблюдатель**
```cpp
class ConsoleObserver : public Observer {
public:
    void update(int value) override {
        std::cout << "Received update: " << value << "\n";
    }
};
```

##### **Шаг 4: Использование**
```cpp
int main() {
    Subject subject;
    ConsoleObserver obs1, obs2;

    subject.attach(&obs1);
    subject.attach(&obs2);
    subject.notify(42);  // Уведомление всех

    subject.detach(&obs1);
    subject.notify(100); // Только obs2 получит уведомление
}
```

#### **3. Ключевые особенности**
1. **Динамический массив**  
   - Аналог `std::vector` с ручным управлением памятью (`new[]`, `delete[]`).  
   - Автоматическое расширение при переполнении (`resize()`).  
2. **Безопасность**  
   - Наблюдатели не удаляются субъектом (ответственность за память — на вызывающей стороне).  
3. **Производительность**  
   - Быстрее STL в embedded-системах (нет накладных расходов).  

#### **4. Альтернативные структуры данных**
- **Фиксированный массив** (если известно max число наблюдателей):  
  ```cpp
  Observer* observers[MAX_OBSERVERS];
  ```
- **Односвязный список** (для частых вставок/удалений):  
  ```cpp
  struct Node {
      Observer* observer;
      Node* next;
  };
  Node* head;
  ```

#### **5. Когда использовать?**  
- **Embedded-разработка** (минимальные зависимости).  
- **Высокопроизводительный код** (игры, real-time системы).  
- **Обучение** (понимание работы контейнеров).  

#### **6. Сравнение с STL-версией**

| **Критерий**       | **Без STL**                     | **С STL**                     |
|--------------------|--------------------------------|-------------------------------|
| **Память**         | Ручное управление              | Автоматическое (`std::vector`) |
| **Безопасность**   | Требует аккуратности           | Безопаснее                    |
| **Производительность** | Выше (нет оверхеда STL)    | Удобнее, но медленнее         |

**Идеальный ответ:**  
*"Паттерн 'Наблюдатель' без STL реализуется через:*
*1. **Динамический массив** для хранения наблюдателей (аналог `std::vector`).* 
*2. **Ручное управление памятью** (`new[]`, `delete[]`).*
*3. **Интерфейс `Observer`** с методом `update()`.*

*Пример: субъект (`Subject`) уведомляет наблюдателей при изменении состояния.  
**Плюсы:***
*- Нет зависимостей от STL.*
*- Полный контроль над памятью.*
***Минусы:***
*- Требуется аккуратность в управлении памятью.*

*Оптимально для embedded-систем или проектов с запретом на STL."*