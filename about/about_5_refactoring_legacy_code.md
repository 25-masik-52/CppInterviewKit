#### **Что хочет услышать HR/тим-лид:**
1. **Системный подход** – как вы анализируете, планируете и минимизируете риски.
2. **Практические методы** – конкретные техники и инструменты, которые вы используете.
3. **Безопасность изменений** – как избегаете регрессий в рабочем коде.
4. **Командное взаимодействие** – согласование с коллегами, документация.

#### **Шаблон ответа для C++ разработчика:**
*"При работе с legacy-кодом я придерживаюсь принципа «не сломать то, что работает», поэтому мой подход включает несколько этапов:*

1. **Анализ и оценка**
	- Изучаю архитектуру и зависимости:
		- Инструменты: **Doxygen**, **callgrind**, **CMake graphviz**.
		- Особое внимание — на «боль» команды (частые баги, узкие места).
	- Определяю scope рефакторинга: точечные правки или глобальная переработка.

2. **Тестовая страховка**
	- Если тестов нет — пишу **юнит-тесты (Google Test/Catch2)** для критичных функций.
	- Для сложных случаев — **интеграционные тесты с моками**.
	- Пример: перед рефакторингом класса-синглтона добавил тесты на его жизненный цикл.

3. **Постепенное улучшение**
	- Применяю **«шаг за шагом»**:
		- Сначала упрощаю (убираю дублирование, заменяем `#define` на `constexpr`).
		- Затем модернизирую (перевожу **C-style-код** на **RAII**, **умные указатели**).
	- Использую **clang-tidy** и **clang-format** для соблюдения стиля.

4. **Контроль последствий**
	- **Профилирую до/после (perf, Valgrind)** — проверяю, что не добавил накладных расходов.
	- Документирую изменения (в **Git** — осмысленные сообщения коммитов, в **Confluence** — схемы новых решений).

#### **Что ценится в ответе:**
✔ **Акцент на безопасности**: «Сначала тесты, потом изменения».
✔ **Примеры инструментов**: покажите, что вы знакомы с современным стеком.
✔ **Измеримый результат**: «Сократили технический долг на X%».

#### **Что избегать:**
❌ Упоминаний радикальных мер («Полностью переписал за 2 ночи»).
❌ Критики предыдущих разработчиков («Код был ужасен»).

#### **Пример из практики:**
*"В проекте на C++03 был класс с 2000 строк кода и «магическими» enum.
Я:*
1. *Разбил его на 5 smaller классов с четкими обязанностями.*
2. *Заменил enum на enum class + static_assert для валидации.*
3. *Добавил тесты, покрывающие edge-cases.*

*Результат: количество связанных с этим классом багов упало на 90%."*