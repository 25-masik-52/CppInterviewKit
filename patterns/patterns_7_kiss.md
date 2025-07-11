#### 1. **Определение**  
**KISS (Keep It Simple, Stupid)** — принцип разработки, утверждающий, что системы должны быть максимально простыми, но не примитивными.  

#### 2. **Примеры применения**  
**Плохо (сложно):**  
```cpp  
bool isPrime(int n) {  
    return (n % 2 != 0 || n == 2) && (n % 3 != 0 || n == 3);  
}  
```  
**Хорошо (просто):**  
```cpp  
bool isPrime(int n) {  
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; ++i) 
        if (n % i == 0) return false;
    return true;
}
```  

#### 3. **Плюсы KISS**  
- **Лучшая читаемость**: Код понятен даже новичкам.  
- **Лёгкость поддержки**: Меньше багов, проще вносить изменения.  
- **Упрощённое тестирование**: Меньше состояний и зависимостей.  

#### 4. **Когда использовать?**  
- В **большинстве случаев** (95% кода).  
- Особенно в **командной разработке**.  

#### 5. **Когда можно нарушить?**  
- **Критические участки**: Низкоуровневые оптимизации.  
- **Специфичные алгоритмы**: Например, машинное обучение.  

#### 6. **Как применять?**  
1. Разбивайте сложные функции на простые шаги.  
2. Избегайте избыточных абстракций.  
3. Используйте понятные имена переменных.

#### 7. **KISS vs DRY**  
- **KISS**: Упрощайте, даже если придётся повториться.  
- **DRY**: Устраняйте дублирование, но не ценой сложности.  
**Баланс**: В сомнительных случаях выбирайте KISS.

**Идеальный ответ:**  
*"KISS — это принцип разработки, требующий сохранять код максимально простым и понятным. Он улучшает читаемость, упрощает поддержку и снижает количество ошибок. Например, вместо хитроумной однострочной проверки на простое число лучше написать понятный цикл. Хотя KISS можно нарушить для оптимизации критических участков, в большинстве случаев простота должна быть приоритетом. Главное правило: пишите код так, чтобы его мог понять junior-разработчик через полгода."*  