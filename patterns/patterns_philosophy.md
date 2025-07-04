#### **Паттерны и принципы проектирования**

**Что такое MVC?**  
> *"MVC — это как театральная постановка: Модель — сценарист (данные), View — сцена (отображение), Controller — режиссёр (логика). Каждый играет свою роль, не мешая другим."*  
> — **Trygve Reenskaug**, создатель MVC  

**Что такое MVP?**  
> *"MVP — это переводчик между Моделью и View. Presenter берёт всю логику на себя, оставляя View лишь показывать результат."*  
> — **Martin Fowler**, *«Patterns of Enterprise Application Architecture»*  

**Что такое SOLID?**  
> *"SOLID — это пять заповедей архитектора кода:
> - **S**ingle Responsibility (Один класс — одна цель)
> - **O**pen/Closed (Открыт для расширения, закрыт для изменений)
> - **L**iskov Substitution (Дети не должны ломать родителей)
> - **I**nterface Segregation (Много специализированных интерфейсов)
> - **D**ependency Inversion (Зависи от абстракций)
> Нарушишь — получишь хрупкий код."*  
> — **Роберт Мартин**, *«Clean Code»*  

**Что такое RAII?**  
> *"RAII — это как хороший родитель: дал ресурс — научил им правильно распоряжаться. Деструктор — последний урок перед уходом."*  
> — **Бьерн Страуструп**, *«The C++ Programming Language»*  

**Что такое CRTP?**  
> *"CRTP — это магия статического полиморфизма: класс смотрит в зеркало и видит своё отражение шаблонным параметром."*  
> — **Александреску**, *«Modern C++ Design»*  

**Что такое DRY?**  
> *"DRY — это как золотое правило морали для кода: не повторяйся, ибо повторение — мать всех ошибок."*  
> — **Эндрю Хант и Дейв Томас**, *«The Pragmatic Programmer»*  

**Что такое KISS?**  
> *"KISS — это 'Будь проще'. Сложные решения редко бывают правильными."*  
> — **Келли Джонсон**, инженер Lockheed  

**Что такое YAGNI?**  
> *"YAGNI — это 'Тебе это не понадобится'. Будущее слишком туманно, чтобы писать код 'на всякий случай'."*  
> — **Рон Джеффрис**, экстремальное программирование  

**Что такое PIMPL?**  
> *"PIMPL — это как чёрный ящик: спрячь реализацию, чтобы мир видел только интерфейс."*  
> — **Герб Саттер**, *«Exceptional C++»*  

**Что такое SFINAE?**  
> *"SFINAE — это искусство вежливого отказа: 'Если не подходит — тихо отступим, не вызывая ошибок'."*  
> — **Совет от C++ шаблонных магов**  

**Виды шаблонов проектирования?**  
> *"Шаблоны проектирования — это как словарь мудрости архитекторов: порождающие — о рождении объектов, структурные — о построении систем, поведенческие — о взаимодействии."*  
> — **Эрих Гамма**, *«Design Patterns»*  

#### **Идиомы**

**Что такое простая фабрика (Simple Factory)?**  
> *"Простая фабрика — это как конвейер: одно место для создания объектов, чтобы не разбрасывать 'new' по всему коду."*  
> — **Эрих Гамма**, *«Design Patterns»*  

#### **Порождающие паттерны**

**Фабричный метод (Factory Method)**  
> *"Фабричный метод — это как завещание: родитель объявляет, что нужно создать, но оставляет детям право решать как."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Абстрактная фабрика (Abstract Factory)**  
> *"Абстрактная фабрика — это королевская династия: каждая фабрика знает, как создать своё семейство объектов."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Строитель (Builder)**  
> *"Строитель — это архитектор и прораб в одном лице: сначала план, потом пошаговая реализация."*  
> — **Джошуа Блох**, *«Effective Java»*  

**Паттерн Прототип (Prototype)**  
> *"Прототип — это клонирование вместо рождения: иногда проще сделать копию, чем создавать с нуля."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Одиночка (Singleton)**  
> *"Singleton — это фараон: только один, всемогущий, но его глобальность может стать проклятием."*  
> — **Роберт Мартин**, *«Clean Code»*  

#### **Структурные паттерны**

**Паттерн Адаптер (Adapter)**  
> *"Адаптер — это переводчик между старым и новым кодом: делает несовместимое совместимым."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Мост (Bridge)**  
> *"Мост — это разделение абстракции и реализации: как дорога и мост через реку — меняются независимо."*  
> — **Джон Влиссидес**, *«Pattern Hatching»*  

**Паттерн Компоновщик (Composite)**  
> *"Компоновщик — это матрёшка: и целое, и части работают одинаково."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Декоратор (Decorator)**  
> *"Декоратор — это новогодняя ёлка: можно бесконечно добавлять украшения, не меняя саму ёлку."*  
> — **Ральф Джонсон**, соавтор GoF  

**Паттерн Фасад (Facade)**  
> *"Фасад — это личный помощник: скрывает сложность системы за простым интерфейсом."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Приспособленец (Flyweight)**  
> *"Flyweight — это общая память: как деревья в лесу, которые делятся корневой системой."*  
> — **Мартин Фаулер**, *«Refactoring»*  

**Паттерн Заместитель (Proxy)**  
> *"Proxy — это телохранитель объекта: контролирует доступ, как секьюрити у знаменитости."*  
> — **Эрих Гамма**, *«Design Patterns»*  

#### **Поведенческие паттерны**

**Паттерн Цепочка ответственности (Chain of Responsibility)**  
> *"Цепочка — это эстафета: запрос передаётся, пока кто-то не возьмёт ответственность."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Команда (Command)**  
> *"Команда — это окаменевшее действие: можно хранить, отменять и повторять."*  
> — **Бертран Мейер**, *«Object-Oriented Software Construction»*  

**Паттерн Итератор (Iterator)**  
> *"Итератор — это путеводитель по коллекции: не важно, что внутри — массив или джунгли."*  
> — **Александр Степанов**, создатель STL  

**Паттерн Посредник (Mediator)**  
> *"Посредник — это диспетчер аэропорта: объекты больше не общаются напрямую, только через центр."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Хранитель (Memento)**  
> *"Memento — это машина времени: можно сохранить состояние и вернуться к нему."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Наблюдатель (Observer)**  
> *"Наблюдатель — это система оповещения: 'Произошло событие — всем сообщить!'"*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Посетитель (Visitor)**  
> *"Посетитель — это турист: ходит по структуре данных и выполняет операции."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Стратегия (Strategy)**  
> *"Стратегия — это набор инструментов: меняй алгоритмы, как перчатки."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Состояние (State)**  
> *"Состояние — это хамелеон: объект меняет поведение в зависимости от внутреннего состояния."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Паттерн Шаблонный метод (Template Method)**  
> *"Шаблонный метод — это кулинарный рецепт: базовый класс определяет шаги, наследники — детали."*  
> — **Эрих Гамма**, *«Design Patterns»*  

#### **Принципы**

**Что такое Dependency Injection?**  
> *"DI — это как заказ в ресторане: не готовь сам, пусть принесут готовое."*  
> — **Мартин Фаулер**, *«Inversion of Control Containers»*  

**Что такое 'адаптер функции'?**  
> *"Адаптер функции — это переходник: делает неудобный интерфейс удобным."*  
> — **Скотт Мейерс**, *«Effective STL»*  

**Что такое 'прокси-объект'?**  
> *"Прокси — это тень объекта: выглядит так же, но контролирует доступ."*  
> — **Эрих Гамма**, *«Design Patterns»*  

**Как реализовать паттерн 'Фабрика' без if?**  
> *"Фабрика без if — это как автомат по продаже напитков: нажал кнопку — получил нужный объект."*  
> — **Александреску**, *«Modern C++ Design»*  

**Как реализовать паттерн 'Наблюдатель' без STL?**  
> *"Наблюдатель — это голубиная почта: объекты регистрируются и получают сообщения."*  
> — **Джон Лакос**, *«Large-Scale C++»*