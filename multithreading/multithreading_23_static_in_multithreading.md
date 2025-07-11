#### **1. Определение**  
Ключевое слово `static` в C++ используется для создания переменных или функций с уникальными свойствами:  
- Сохранение значения между вызовами (локальные `static`).  
- Общий доступ для всех экземпляров класса (члены класса).  
- Ограничение видимости в пределах файла (глобальные `static`).  

В многопоточности `static`-переменные являются общими для всех потоков, что требует синхронизации.

#### **2. Основные понятия**  
- **Гонка данных (Data Race)**: Несинхронизированный доступ к общим `static`-переменным.  
- **Потокобезопасная инициализация**: Гарантируется для локальных `static`-переменных с C++11.  
- **Атомарные операции и мьютексы**: Инструменты для синхронизации.  

#### **3. Реализация в C++**  
**Примеры:**  
1. **Локальные `static`-переменные** (потокобезопасная инициализация, но не доступ):  
   ```cpp
   void foo() {
       static int counter = 0; // Инициализация безопасна, но инкремент требует синхронизации.
       ++counter; // Data Race!
   }
   ```  
   Решение:  
   ```cpp
   static std::atomic<int> counter{0}; // Атомарный доступ.
   ```

2. **Статические члены класса**:  
   ```cpp
   class Logger {
   public:
       static void log() {
           static std::mutex mtx;
           std::lock_guard<std::mutex> lock(mtx);
           static int calls = 0; // Защищённый доступ.
       }
   };
   ```

3. **Глобальные `static`-переменные**:  
   ```cpp
   namespace {
       static int shared = 0;
       static std::mutex mtx;
   }
   ```

#### **4. Применение и примеры**  
- **Singleton**: Ленивая инициализация через `static`-локальную переменную (потокобезопасна в C++11+).  
- **Счётчики/Кэши**: Общие ресурсы, требующие синхронизации.  
- **Логирование**: Статический мьютекс для защиты записи в файл.  

#### **5. Плюсы и минусы**  
| **Плюсы**                                 | **Минусы**                                          |
| ----------------------------------------- | --------------------------------------------------- |
| ✅ Общие ресурсы без глобальных переменных | ❌ Риск гонок данных без синхронизации               |
| ✅ Ленивая инициализация (C++11+)          | ❌ False Sharing для близко расположенных переменных |

#### **6. Сравнение с другими подходами**  
- **Глобальные переменные**: Аналогичны `static`, но видимы везде. `static` ограничивает область видимости.  
- **thread_local**: Альтернатива для переменных, уникальных для каждого потока.  

#### **7. Best Practices**  
- Используйте `std::atomic` или `std::mutex` для защиты `static`-переменных.  
- Для однократной инициализации предпочитайте `static`-локальные переменные (C++11+).  
- Избегайте `static` для часто изменяемых данных из-за накладных расходов на синхронизацию.  

**Идеальный ответ:**
*"`static` в многопоточности применяется для создания общих ресурсов (счётчиков, кэшей, Singleton), но требует синхронизации (мьютексы, атомарные операции) для избежания гонок данных. С C++11 инициализация локальных `static`-переменных потокобезопасна. Best practice: использовать `std::lock_guard` и `std::atomic` для защиты доступа."*