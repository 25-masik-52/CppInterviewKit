#### **1. Определение**  
- **Кратчайший путь** — это путь между двумя вершинами графа с минимальным суммарным весом рёбер (для взвешенных графов) или минимальным количеством рёбер (для невзвешенных).  

#### **2. Основные понятия**  
- **Невзвешенный граф:** рёбра не имеют весов, кратчайший путь определяется количеством рёбер.  
- **Взвешенный граф:** рёбра имеют веса, кратчайший путь определяется суммой весов.  

#### **3. Реализация в C++**  

**Для невзвешенного графа:**  
Используется **BFS (поиск в ширину)**.  
```cpp
#include <queue>
#include <vector>
#include <algorithm>

std::vector<int> BFS_ShortestPath(const std::vector<std::vector<int>>& graph, int start, int target) {
    std::queue<int> q;
    std::vector<bool> visited(graph.size(), false);
    std::vector<int> parent(graph.size(), -1);

    q.push(start);
    visited[start] = true;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        if (u == target) {
            std::vector<int> path;
            for (int v = target; v != -1; v = parent[v]) {
                path.push_back(v);
            }
            std::reverse(path.begin(), path.end());
            return path;
        }

        for (int v : graph[u]) {
            if (!visited[v]) {
                visited[v] = true;
                parent[v] = u;
                q.push(v);
            }
        }
    }
    return {};  // Путь не найден
}
```

**Для взвешенного графа:**  
- **Алгоритм Дейкстры** (если веса ≥ 0).  
- **Алгоритм Беллмана-Форда** (если есть отрицательные веса).  

**Пример Дейкстры:**  
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

    if (dist[target] == std::numeric_limits<int>::max()) return {};

    std::vector<int> path;
    for (int v = target; v != -1; v = parent[v]) {
        path.push_back(v);
    }
    std::reverse(path.begin(), path.end());
    return path;
}
```

#### **4. Применение и примеры**  
- **BFS:**  
  - Социальные сети (поиск минимальных связей между пользователями).  
  - Лабиринты (поиск выхода за минимальное число шагов).  
- **Дейкстра:**  
  - GPS-навигация (поиск оптимального маршрута).  
  - Сетевые алгоритмы (маршрутизация данных).  
- **Беллман-Форд:**  
  - Финансовые модели (учёт отрицательных весов, например, убытков).  

#### **5. Плюсы и минусы**  
| Алгоритм          | Плюсы                                     | Минусы                                      |
|-------------------|-------------------------------------------|---------------------------------------------|
| **BFS**           | Прост в реализации, линейная сложность.   | Не работает с весами.                       |
| **Дейкстра**      | Оптимален для весов ≥ 0, быстрый.         | Не работает с отрицательными весами.        |
| **Беллман-Форд**  | Работает с отрицательными весами.         | Высокая сложность (`O(V*E)`).               |

#### **6. Сравнение с другими подходами**  
- **BFS vs Дейкстра:**  
  - BFS подходит только для невзвешенных графов, Дейкстра — для взвешенных (с весами ≥ 0).  
- **Дейкстра vs Беллман-Форд:**  
  - Дейкстра быстрее, но не поддерживает отрицательные веса.  

#### **7. Best Practices**  
- Для больших графов используйте оптимизированные структуры данных (например, `std::priority_queue` для Дейкстры).  
- Проверяйте граф на отрицательные циклы перед применением Беллмана-Форда.  

**Идеальный ответ:**
_Для невзвешенных графов используйте **BFS** (сложность `O(V + E)`), для взвешенных с весами ≥ 0 — **Дейкстру** (`O((V + E) log V)`), а для отрицательных весов — **Беллмана-Форда** (`O(V * E)`). Примеры кода на C++ включают восстановление пути и обработку особых случаев (например, отсутствие пути)._