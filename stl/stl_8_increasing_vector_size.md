#### **1. Определение**  
`std::vector` — это контейнер с динамическим массивом, который автоматически увеличивает свою ёмкость при добавлении элементов, когда текущий размер (`size`) достигает текущей ёмкости (`capacity`). Процесс увеличения включает перевыделение памяти и копирование/перемещение существующих элементов.

#### **2. Основные понятия**  
- **`size`**: Текущее количество элементов в векторе.  
- **`capacity`**: Максимальное количество элементов, которое вектор может хранить без перевыделения памяти.  
- **Перевыделение (reallocation)**: Процесс выделения нового блока памяти, копирования/перемещения элементов и освобождения старой памяти.  

#### **3. Механизм увеличения размера**  
##### **1. Стратегия роста**  
- **Экспоненциальное увеличение**:  
  При перевыделении ёмкость увеличивается в **1.5–2 раза** (зависит от реализации стандартной библиотеки).  
  **Пример**:  
  - Исходная `capacity = 4` → после перевыделения `capacity = 8`.  
  - Следующее перевыделение: `capacity = 16`.  
##### **2. Алгоритм перевыделения**  
1. **Проверка ёмкости**:  
   Перед добавлением элемента проверяется условие `size == capacity`.  
2. **Выделение нового блока памяти**:  
   - Размер нового блока: `new_capacity = old_capacity * growth_factor`.  
3. **Перенос элементов**:  
   - Элементы копируются или перемещаются (если тип поддерживает `std::move`) в новый блок.  
4. **Освобождение старой памяти**:  
   - Исходный блок памяти освобождается.  
##### **3. Пример перевыделения**  
```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec;
    for (int i = 0; i < 10; ++i) {
        vec.push_back(i);
        std::cout << "Size: " << vec.size() 
                  << ", Capacity: " << vec.capacity() << "\n";
    }
}
```
**Вывод**:  
```
Size: 1, Capacity: 1
Size: 2, Capacity: 2
Size: 3, Capacity: 4
Size: 4, Capacity: 4
Size: 5, Capacity: 8
... 
Size: 9, Capacity: 16
```
**Пояснение**:  
Ёмкость увеличивается в 2 раза при каждом перевыделении.

#### **4. Почему экспоненциальный рост?**  
- **Амортизированная сложность O(1)**:  
  Средняя стоимость добавления элемента остаётся постоянной, несмотря на периодические перевыделения.  
- **Минимизация перевыделений**:  
  Увеличение ёмкости в геометрической прогрессии сокращает количество дорогостоящих операций копирования.  

**Формула амортизированной стоимости**:  
Для `n` операций `push_back` общая сложность — `O(n)`, что даёт `O(1)` на операцию.  

#### **5. Управление ёмкостью вручную**  
Методы для оптимизации:  
- **`reserve(n)`**: Резервирует память для `n` элементов, избегая перевыделений.  
- **`shrink_to_fit()`**: Уменьшает `capacity` до `size` (не гарантируется стандартом).  

**Пример**:  
```cpp
std::vector<int> vec;
vec.reserve(100); // Избегаем перевыделений при добавлении до 100 элементов.
```

#### **6. Особенности и подводные камни**  
- **Инвалидация итераторов**:  
  После перевыделения все итераторы, указатели и ссылки на элементы становятся недействительными.  
- **Стоимость копирования**:  
  Для типов с "тяжёлым" копированием (например, `std::string`) перевыделение может быть дорогим.  
- **Исключения**:  
  Если конструктор копирования/перемещения выбрасывает исключение во время перевыделения, вектор остаётся в согласованном состоянии.  

**Пример с перемещением**:  
```cpp
std::vector<std::string> vec;
vec.push_back("data"); // Перемещение вместо копирования, если возможно.
```

#### 7. Сравнение с другими контейнерами  

| **Критерий**           | **`std::vector`**                         | **`std::list`**                                    |
| ---------------------- | ----------------------------------------- | -------------------------------------------------- |
| **Увеличение размера** | Требует перевыделения и копирования.      | Нет перевыделения (элементы выделяются по одному). |
| **Производительность** | Быстрый доступ, но дорогое перевыделение. | Медленный доступ, но дешёвые вставки/удаления.     |

#### **8. Best Practices**  
1. **Используйте `reserve()`**:  
   Если известен будущий размер, резервируйте память заранее.  
2. **Избегайте частых перевыделений**:  
   Для часто изменяемых контейнеров рассмотрите `std::deque`.  
3. **Оптимизация для сложных типов**:  
   Используйте семантику перемещения (`std::move`).  

**Пример оптимизации**:  
```cpp
std::vector<ExpensiveType> vec;
vec.reserve(1000); // Резервируем память для 1000 элементов.
for (int i = 0; i < 1000; ++i) {
    vec.push_back(ExpensiveType()); // Избегаем перевыделений.
}
```

**Идеальный ответ:**
_"Увеличение размера `std::vector` происходит при исчерпании ёмкости (`size == capacity`). Алгоритм:_
_1. Выделяется новый блок памяти (обычно в **1.5–2 раза** больше)._
_2. Элементы копируются/перемещаются в новый блок._
_3. Старая память освобождается._
_**Результат**:_
_- Амортизированная сложность `push_back` — O(1)._
_- Итераторы/указатели инвалидируются._
_Для оптимизации используйте `reserve()` и перемещение."_