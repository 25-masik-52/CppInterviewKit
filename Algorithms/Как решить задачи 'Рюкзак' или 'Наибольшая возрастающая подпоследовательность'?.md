#### **1. Определение**
##### "Задача о рюкзаке (Knapsack)"
**Формулировка:**  
Дано `N` предметов с весами `w[i]` и стоимостями `v[i]`, а также рюкзак вместимостью `W`.
**Цель:** Выбрать подмножество предметов с максимальной суммой стоимости, не превышающее `W`.

##### **Решение 1: Классический DP (O(N⋅W))**
```cpp
#include <vector>
#include <algorithm>

int knapsack_01(const std::vector<int>& weights, const std::vector<int>& values, int W) {
    int n = weights.size();
    std::vector<std::vector<int>> dp(n + 1, std::vector<int>(W + 1, 0));

    for (int i = 1; i <= n; ++i) {
        for (int j = 0; j <= W; ++j) {
            if (weights[i-1] <= j) {
                dp[i][j] = std::max(dp[i-1][j], dp[i-1][j - weights[i-1]] + values[i-1]);
            } else {
                dp[i][j] = dp[i-1][j];
            }
        }
    }
    return dp[n][W];
}
```
**Оптимизация памяти (O(W)):**
```cpp
int knapsack_01_optimized(const std::vector<int>& weights, const std::vector<int>& values, int W) {
    std::vector<int> dp(W + 1, 0);
    for (int i = 0; i < weights.size(); ++i) {
        for (int j = W; j >= weights[i]; --j) {  // Обратный порядок!
            dp[j] = std::max(dp[j], dp[j - weights[i]] + values[i]);
        }
    }
    return dp[W];
}
```
**Ключевые моменты:**
- Используется **обратный порядок** в цикле по `j` для избежания перезаписи.
- Сложность: **Псевдополиномиальная** (зависит от `W`).

##### **Решение 2: Для больших `W` (Meet-in-the-Middle)**
Если `W` слишком велико (например, `1e9`), но `N` небольшое (до 40):
```cpp
int knapsack_large_W(const std::vector<int>& weights, const std::vector<int>& values, int W) {
    int n = weights.size();
    int half = n / 2;
    std::vector<std::pair<int, int>> left, right;

    // Генерация всех подмножеств для левой и правой половин
    for (int mask = 0; mask < (1 << half); ++mask) {
        int w = 0, v = 0;
        for (int i = 0; i < half; ++i) {
            if (mask & (1 << i)) {
                w += weights[i];
                v += values[i];
            }
        }
        left.emplace_back(w, v);
    }

    for (int mask = 0; mask < (1 << (n - half)); ++mask) {
        int w = 0, v = 0;
        for (int i = 0; i < (n - half); ++i) {
            if (mask & (1 << i)) {
                w += weights[half + i];
                v += values[half + i];
            }
        }
        right.emplace_back(w, v);
    }

    // Сортировка и two-pointers
    std::sort(right.begin(), right.end());
    int max_value = 0;
    for (const auto& [w, v] : left) {
        if (w > W) continue;
        auto it = std::upper_bound(right.begin(), right.end(), std::make_pair(W - w, INT_MAX));
        if (it != right.begin()) {
            max_value = std::max(max_value, v + (--it)->second);
        }
    }
    return max_value;
}
```
**Сложность:** `O(N⋅2^(N/2))`.

##### **Наибольшая возрастающая подпоследовательность (LIS)**
**Формулировка:**  
Найти длину самой длинной строго возрастающей подпоследовательности в массиве `nums`.

##### **Решение 1: Классический DP (O(N²))**
```cpp
#include <vector>
#include <algorithm>

int lengthOfLIS_DP(const std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> dp(n, 1);
    int max_len = 1;

    for (int i = 1; i < n; ++i) {
        for (int j = 0; j < i; ++j) {
            if (nums[j] < nums[i]) {
                dp[i] = std::max(dp[i], dp[j] + 1);
            }
        }
        max_len = std::max(max_len, dp[i]);
    }
    return max_len;
}
```
**Недостаток:** Медленно для больших `N` (например, `N > 1e4`).

##### **Решение 2: Оптимизация с бинарным поиском (O(N log N))**
```cpp
#include <vector>
#include <algorithm>

int lengthOfLIS_optimized(const std::vector<int>& nums) {
    std::vector<int> tails;
    for (int num : nums) {
        auto it = std::lower_bound(tails.begin(), tails.end(), num);
        if (it == tails.end()) {
            tails.push_back(num);
        } else {
            *it = num;
        }
    }
    return tails.size();
}
```
**Как это работает:**
- `tails[i]` — минимальный последний элемент подпоследовательности длины `i+1`.
- **Пример:**  
  Для `nums = [10, 9, 2, 5, 3, 7, 101, 18]`:  
  `tails` эволюционирует как:  
  `[10]` → `[9]` → `[2]` → `[2,5]` → `[2,3]` → `[2,3,7]` → `[2,3,7,101]` → `[2,3,7,18]`.  
  Итоговая длина — `4`.

##### **Дополнение: Восстановление самой LIS**
```cpp
std::vector<int> getLIS(const std::vector<int>& nums) {
    std::vector<int> tails, parent(nums.size(), -1), lis;
    for (int i = 0; i < nums.size(); ++i) {
        auto it = std::lower_bound(tails.begin(), tails.end(), nums[i]);
        int pos = it - tails.begin();
        if (it == tails.end()) {
            tails.push_back(nums[i]);
        } else {
            *it = nums[i];
        }
        parent[i] = (pos > 0) ? parent[pos - 1] : -1;
    }

    int idx = parent.size() - 1;
    while (idx >= 0) {
        lis.push_back(nums[idx]);
        idx = parent[idx];
    }
    std::reverse(lis.begin(), lis.end());
    return lis;
}
```

#### **2. Сравнение методов**

| Задача       | Лучший метод               | Сложность      | Когда использовать               |
|--------------|----------------------------|----------------|----------------------------------|
| **Рюкзак**   | DP (оптимизированный)      | `O(N⋅W)`       | Если `W` умеренное (до `1e6`).  |
|              | Meet-in-the-Middle         | `O(N⋅2^(N/2))` | Если `N` мало (≤ 40), а `W` огромное. |
| **LIS**      | Бинарный поиск             | `O(N log N)`   | Всегда предпочтительнее DP.      |

**Идеальный ответ:**
_- **Для задачи о рюкзаке:**_
    _- Используйте **оптимизированный DP** (`O(W)` памяти) для умеренных `W`._
    _- Для огромных `W` и малых `N` — **Meet-in-the-Middle**._
_- **Для LIS:**_
    _- Всегда применяйте **бинарный поиск** (`O(N log N)`)._
    _- Для восстановления подпоследовательности — модифицируйте алгоритм с сохранением `parent`._
_**Примеры вызовов:**_
```cpp
vector<int> weights = {2, 3, 4, 5}, values = {3, 4, 5, 6};
cout << knapsack_01_optimized(weights, values, 5);  // 7 (предметы 2 и 3)  

vector<int> nums = {10, 9, 2, 5, 3, 7, 101, 18};
cout << lengthOfLIS_optimized(nums);  // 4
```