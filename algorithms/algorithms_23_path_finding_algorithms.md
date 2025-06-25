#### **1. Введение в алгоритмы поиска пути**
Алгоритмы поиска пути используются для нахождения маршрутов между точками в графах. Выбор алгоритма зависит от:
- **Типа графа** (взвешенный/невзвешенный, ориентированный/неориентированный).
- **Критериев оптимальности** (кратчайший путь, минимальное время, ресурсы).
- **Ограничений** (память, процессорное время).

#### **2. BFS (поиск в ширину)**
**Когда использовать**:
- Кратчайший путь в **невзвешенном графе** (минимальное количество рёбер).
- Поиск ближайших соседей.

**Реализация (C++)**:
```cpp
#include <queue>
#include <unordered_map>
#include <vector>

std::vector<int> BFS(const std::vector<std::vector<int>>& graph, int start, int target) {
    std::queue<int> q;
    std::unordered_map<int, int> parent;
    std::vector<int> path;
    q.push(start);
    parent[start] = -1;

    while (!q.empty()) {
        int current = q.front();
        q.pop();
        if (current == target) {
            // Восстановление пути
            for (int at = target; at != -1; at = parent[at]) {
                path.push_back(at);
            }
            std::reverse(path.begin(), path.end());
            return path;
        }
        for (int neighbor : graph[current]) {
            if (parent.find(neighbor) == parent.end()) {
                parent[neighbor] = current;
                q.push(neighbor);
            }
        }
    }
    return {}; // Путь не найден
}
```

**Теория**:
- BFS исследует граф уровень за уровнем.
- Гарантирует нахождение кратчайшего пути в невзвешенных графах.
- Использует очередь (FIFO) для обработки вершин.

**Сложность**:
- Время: `O(V + E)`
- Память: `O(V)`

#### **3. DFS (поиск в глубину)**
**Когда использовать**:
- Поиск **любого пути** (не обязательно кратчайшего).
- Проверка связности, топологическая сортировка.

**Реализация (C++)**:
```cpp
#include <stack>
#include <unordered_set>
#include <vector>

bool DFS(const std::vector<std::vector<int>>& graph, int start, int target) {
    std::stack<int> s;
    std::unordered_set<int> visited;
    s.push(start);

    while (!s.empty()) {
        int current = s.top();
        s.pop();
        if (current == target) return true;
        if (visited.find(current) == visited.end()) {
            visited.insert(current);
            for (int neighbor : graph[current]) {
                s.push(neighbor);
            }
        }
    }
    return false;
}
```

**Теория**:
- DFS исследует граф, углубляясь в одну ветвь до конца.
- Может зациклиться на графах с циклами без отметки посещённых вершин.
- Использует стек (LIFO) для обработки вершин.

**Сложность**:
- Время: `O(V + E)`
- Память: `O(V)` (глубина стека)

#### **4. Алгоритм Дейкстры**
**Когда использовать**:
- Кратчайший путь во **взвешенном графе** (рёбра ≥ 0).

**Реализация (C++)**:
```cpp
#include <queue>
#include <vector>
#include <limits>

std::vector<int> Dijkstra(const std::vector<std::vector<std::pair<int, int>>>& graph, int start, int target) {
    std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>, std::greater<>> pq;
    std::vector<int> dist(graph.size(), std::numeric_limits<int>::max());
    std::vector<int> parent(graph.size(), -1);
    dist[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        auto [current_dist, u] = pq.top();
        pq.pop();
        if (u == target) break;
        if (current_dist > dist[u]) continue;
        for (const auto& [v, weight] : graph[u]) {
            if (dist[v] > dist[u] + weight) {
                dist[v] = dist[u] + weight;
                parent[v] = u;
                pq.push({dist[v], v});
            }
        }
    }

    // Восстановление пути
    std::vector<int> path;
    for (int at = target; at != -1; at = parent[at]) {
        path.push_back(at);
    }
    std::reverse(path.begin(), path.end());
    return path;
}
```

**Теория**:
- Основан на жадном выборе вершины с минимальной текущей стоимостью.
- Использует приоритетную очередь для эффективного выбора вершин.

**Сложность**:
- Время: `O((V + E) log V)`
- Память: `O(V)`

#### **5. A* (A-star)**
**Когда использовать**:
- Кратчайший путь с **эвристикой** (например, GPS-навигация).

**Реализация (C++)**:
```cpp
#include <queue>
#include <functional>
#include <vector>

std::vector<int> AStar(const std::vector<std::vector<std::pair<int, int>>>& graph, int start, int target,
                      std::function<int(int, int)> heuristic) {
    std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>, std::greater<>> pq;
    std::vector<int> g_score(graph.size(), std::numeric_limits<int>::max());
    std::vector<int> parent(graph.size(), -1);
    g_score[start] = 0;
    pq.push({heuristic(start, target), start});

    while (!pq.empty()) {
        auto [f_score, u] = pq.top();
        pq.pop();
        if (u == target) break;
        for (const auto& [v, weight] : graph[u]) {
            int tentative_g = g_score[u] + weight;
            if (tentative_g < g_score[v]) {
                parent[v] = u;
                g_score[v] = tentative_g;
                pq.push({g_score[v] + heuristic(v, target), v});
            }
        }
    }

    // Восстановление пути аналогично Дейкстре
    std::vector<int> path;
    for (int at = target; at != -1; at = parent[at]) {
        path.push_back(at);
    }
    std::reverse(path.begin(), path.end());
    return path;
}
```

**Теория**:
- Комбинирует стоимость пути от старта (`g(n)`) и эвристическую оценку до цели (`h(n)`).
- Оптимален при допустимой (не переоценивающей) эвристике.

**Сложность**:
- Время: `O((V + E) log V)`
- Память: `O(V)`

#### **6. Backtracking (поиск с возвратом)**
**Когда использовать**:
- Задачи с **ограничениями** (например, N-ферзей, лабиринты).

**Реализация (C++)**:
```cpp
#include <vector>

bool solveMaze(std::vector<std::vector<int>>& maze, int x, int y, std::vector<std::pair<int, int>>& path) {
    if (x < 0 || x >= maze.size() || y < 0 || y >= maze[0].size() || maze[x][y] == 0) {
        return false;
    }
    if (x == maze.size() - 1 && y == maze[0].size() - 1) {
        path.push_back({x, y});
        return true;
    }
    maze[x][y] = 0;  // Помечаем как посещённое
    path.push_back({x, y});
    if (solveMaze(maze, x + 1, y, path) || solveMaze(maze, x, y + 1, path)) {
        return true;
    }
    path.pop_back();
    return false;
}
```

**Теория**:
- Систематически перебирает все возможные варианты.
- Откатывается при нарушении ограничений.

**Сложность**:
- Время: `O(2^(V+E))` (в худшем случае)
- Память: `O(V)` (глубина рекурсии)

#### **7. Сравнение алгоритмов**

| Алгоритм         | Тип графа              | Оптимальность          | Сложность        | Память |
| ---------------- | ---------------------- | ---------------------- | ---------------- | ------ |
| **BFS**          | Невзвешенный           | Кратчайший (по рёбрам) | `O(V + E)`       | `O(V)` |
| **DFS**          | Невзвешенный           | Любой путь             | `O(V + E)`       | `O(V)` |
| **Дейкстра**     | Взвешенный (≥ 0)       | Кратчайший (по весу)   | `O((V+E) log V)` | `O(V)` |
| **A***           | Взвешенный + эвристика | Оптимальный            | `O((V+E) log V)` | `O(V)` |
| **Backtracking** | Ограничения            | Зависит от задачи      | Экспоненциальная | `O(V)` |

#### **8. Выбор алгоритма**
1. **Невзвешенный граф**:
   - BFS — для кратчайшего пути.
   - DFS — для проверки связности.
2. **Взвешенный граф**:
   - Дейкстра — если веса ≥ 0.
   - A* — если есть эвристика (например, геометрические расстояния).
3. **Ограниченные задачи**:
   - Backtracking — для перебора с отсечениями.

**Пример**:
```cpp
// Кратчайший путь в невзвешенном графе
auto path = BFS(graph, 0, 5);

// Кратчайший путь с весами
auto path = Dijkstra(weighted_graph, 0, 5);
```

**Идеальный ответ:**
_Для поиска пути используйте:_
_- **BFS** — кратчайший путь в невзвешенных графах._
_- **DFS** — любой путь или проверка связности._
_- **Дейкстру** — кратчайший путь во взвешенных графах (рёбра ≥ 0)._
_- **A*** — оптимизация Дейкстры с эвристикой._
_- **Backtracking** — задачи с ограничениями (например, лабиринты)._
_**Критерии выбора**:_
_- Тип графа (взвешенный/невзвешенный)._
_- Наличие эвристики._
_- Ограничения по времени/памяти._
_**Пример**:_
```cpp
std::vector<int> shortest_path = Dijkstra(graph, start, target);  // Для взвешенного графа
```