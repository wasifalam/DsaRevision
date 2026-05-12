# Graph DSA Revision Notes

This file is ordered for revision:

1. DFS
2. BFS
3. Cycle detection
4. Topological sort and Kahn's algorithm
5. Shortest path algorithms
6. Spanning tree / Minimum spanning tree
7. DSU

Most code templates use `0`-based indexing and C++.

---

## Graph Representation

```cpp
int n, m; // n = number of nodes, m = number of edges
vector<vector<int>> adj(n);

for (int i = 0; i < m; i++) {
    int u, v;
    cin >> u >> v;

    // For undirected graph
    adj[u].push_back(v);
    adj[v].push_back(u);

    // For directed graph, use only:
    // adj[u].push_back(v);
}
```

For weighted graphs:

```cpp
vector<vector<pair<int, int>>> adj(n); // {neighbor, weight}

int u, v, wt;
cin >> u >> v >> wt;
adj[u].push_back({v, wt});
```

---

## 1. DFS

DFS means Depth First Search. It goes as deep as possible before backtracking.

Common uses:

- Traversing graph/components
- Cycle detection
- Topological sort
- Connected components
- Tree problems

Time complexity: `O(V + E)`

```cpp
void dfs(int node, vector<vector<int>>& adj, vector<int>& visited) {
    visited[node] = 1;

    // Visit all unvisited neighbors
    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            dfs(neighbor, adj, visited);
        }
    }
}

int main() {
    int n;
    vector<vector<int>> adj(n);
    vector<int> visited(n, 0);

    // For disconnected graph, run DFS from every unvisited node
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(i, adj, visited);
        }
    }
}
```

---

## 2. BFS

BFS means Breadth First Search. It visits nodes level by level.

Common uses:

- Shortest path in unweighted graph
- Level order traversal
- Bipartite checking
- Connected components

Time complexity: `O(V + E)`

```cpp
void bfs(int start, vector<vector<int>>& adj, vector<int>& visited) {
    queue<int> q;

    visited[start] = 1;
    q.push(start);

    while (!q.empty()) {
        int node = q.front();
        q.pop();

        for (int neighbor : adj[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = 1;
                q.push(neighbor);
            }
        }
    }
}
```

### Shortest Path in Unweighted Graph Using BFS

```cpp
vector<int> shortestPathUnweighted(int n, vector<vector<int>>& adj, int src) {
    vector<int> dist(n, -1);
    queue<int> q;

    dist[src] = 0;
    q.push(src);

    while (!q.empty()) {
        int node = q.front();
        q.pop();

        for (int neighbor : adj[node]) {
            if (dist[neighbor] == -1) {
                dist[neighbor] = dist[node] + 1;
                q.push(neighbor);
            }
        }
    }

    return dist;
}
```

---

## 3. Cycle Detection

Cycle detection is different for undirected and directed graphs.

---

### Cycle Detection in Undirected Graph Using DFS

Idea: If a visited neighbor is not the parent, then a cycle exists.

```cpp
bool dfsCycleUndirected(int node, int parent, vector<vector<int>>& adj, vector<int>& visited) {
    visited[node] = 1;

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            if (dfsCycleUndirected(neighbor, node, adj, visited)) {
                return true;
            }
        } else if (neighbor != parent) {
            // Visited neighbor which is not parent means cycle
            return true;
        }
    }

    return false;
}

bool hasCycleUndirected(int n, vector<vector<int>>& adj) {
    vector<int> visited(n, 0);

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            if (dfsCycleUndirected(i, -1, adj, visited)) {
                return true;
            }
        }
    }

    return false;
}
```

---

### Cycle Detection in Undirected Graph Using BFS

Idea: Store `{node, parent}` in queue. If a visited neighbor is not parent, cycle exists.

```cpp
bool bfsCycleUndirected(int start, vector<vector<int>>& adj, vector<int>& visited) {
    queue<pair<int, int>> q; // {node, parent}

    visited[start] = 1;
    q.push({start, -1});

    while (!q.empty()) {
        auto [node, parent] = q.front();
        q.pop();

        for (int neighbor : adj[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = 1;
                q.push({neighbor, node});
            } else if (neighbor != parent) {
                return true;
            }
        }
    }

    return false;
}
```

---

### Cycle Detection in Directed Graph Using DFS

Idea: Use two arrays:

- `visited[node]`: node has been visited
- `pathVisited[node]`: node is in the current DFS path

If we reach a node already in current path, cycle exists.

```cpp
bool dfsCycleDirected(int node, vector<vector<int>>& adj, vector<int>& visited, vector<int>& pathVisited) {
    visited[node] = 1;
    pathVisited[node] = 1;

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            if (dfsCycleDirected(neighbor, adj, visited, pathVisited)) {
                return true;
            }
        } else if (pathVisited[neighbor]) {
            // Back edge found
            return true;
        }
    }

    pathVisited[node] = 0; // remove from current path while backtracking
    return false;
}

bool hasCycleDirected(int n, vector<vector<int>>& adj) {
    vector<int> visited(n, 0), pathVisited(n, 0);

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            if (dfsCycleDirected(i, adj, visited, pathVisited)) {
                return true;
            }
        }
    }

    return false;
}
```

---

### Cycle Detection in Directed Graph Using Kahn's Algorithm

Idea: If topological sort does not include all nodes, graph has a cycle.

```cpp
bool hasCycleDirectedKahn(int n, vector<vector<int>>& adj) {
    vector<int> indegree(n, 0);

    for (int u = 0; u < n; u++) {
        for (int v : adj[u]) {
            indegree[v]++;
        }
    }

    queue<int> q;
    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) {
            q.push(i);
        }
    }

    int count = 0;

    while (!q.empty()) {
        int node = q.front();
        q.pop();
        count++;

        for (int neighbor : adj[node]) {
            indegree[neighbor]--;
            if (indegree[neighbor] == 0) {
                q.push(neighbor);
            }
        }
    }

    return count != n;
}
```

---

## 4. Topological Sort

Topological sort works only for Directed Acyclic Graphs, also called DAGs.

If there is an edge `u -> v`, then `u` must appear before `v` in topological order.

Common uses:

- Course schedule
- Task dependency order
- Build systems
- DAG shortest path

---

### Topological Sort Using DFS

Idea: Push node after visiting all its neighbors. Reverse the final list.

```cpp
void dfsTopo(int node, vector<vector<int>>& adj, vector<int>& visited, vector<int>& topo) {
    visited[node] = 1;

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            dfsTopo(neighbor, adj, visited, topo);
        }
    }

    // Add after all dependencies are processed
    topo.push_back(node);
}

vector<int> topoSortDFS(int n, vector<vector<int>>& adj) {
    vector<int> visited(n, 0);
    vector<int> topo;

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfsTopo(i, adj, visited, topo);
        }
    }

    reverse(topo.begin(), topo.end());
    return topo;
}
```

---

## 5. Kahn's Algorithm

Kahn's algorithm is BFS-based topological sorting.

Idea:

1. Calculate indegree of every node.
2. Push all nodes with indegree `0` into queue.
3. Remove nodes one by one and reduce indegree of their neighbors.
4. If neighbor indegree becomes `0`, push it into queue.

Time complexity: `O(V + E)`

```cpp
vector<int> kahnTopoSort(int n, vector<vector<int>>& adj) {
    vector<int> indegree(n, 0);

    for (int u = 0; u < n; u++) {
        for (int v : adj[u]) {
            indegree[v]++;
        }
    }

    queue<int> q;
    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) {
            q.push(i);
        }
    }

    vector<int> topo;

    while (!q.empty()) {
        int node = q.front();
        q.pop();
        topo.push_back(node);

        for (int neighbor : adj[node]) {
            indegree[neighbor]--;

            if (indegree[neighbor] == 0) {
                q.push(neighbor);
            }
        }
    }

    // If topo.size() < n, graph contains a cycle
    return topo;
}
```

---

## 6. Shortest Path Algorithms

Use different shortest path algorithms depending on graph type.

| Algorithm | Works With | Negative Weights | Detects Negative Cycle | Complexity |
|---|---|---:|---:|---|
| BFS | Unweighted graph | No | No | `O(V + E)` |
| Dijkstra | Weighted graph | No | No | `O((V + E) log V)` |
| Bellman-Ford | Weighted graph | Yes | Yes | `O(V * E)` |
| Floyd-Warshall | All-pairs shortest path | Yes | Yes | `O(V^3)` |
| DAG shortest path | Directed acyclic graph | Yes | No cycle in DAG | `O(V + E)` |

---

### Dijkstra Algorithm

Dijkstra finds shortest path from one source to all nodes in a weighted graph.

Important:

- Works only when edge weights are non-negative.
- Uses min-heap / priority queue.

```cpp
vector<int> dijkstra(int n, vector<vector<pair<int, int>>>& adj, int src) {
    const int INF = 1e9;
    vector<int> dist(n, INF);

    // {distance, node}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

    dist[src] = 0;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [currentDist, node] = pq.top();
        pq.pop();

        // Ignore outdated heap entries
        if (currentDist > dist[node]) {
            continue;
        }

        for (auto [neighbor, weight] : adj[node]) {
            if (dist[node] + weight < dist[neighbor]) {
                dist[neighbor] = dist[node] + weight;
                pq.push({dist[neighbor], neighbor});
            }
        }
    }

    return dist;
}
```

---

### Bellman-Ford Algorithm

Bellman-Ford finds shortest path from one source to all nodes.

Important:

- Works with negative weights.
- Can detect negative weight cycles.
- Relax all edges `n - 1` times.

```cpp
struct Edge {
    int u, v, wt;
};

vector<int> bellmanFord(int n, vector<Edge>& edges, int src) {
    const int INF = 1e9;
    vector<int> dist(n, INF);
    dist[src] = 0;

    // Relax all edges n - 1 times
    for (int i = 0; i < n - 1; i++) {
        for (auto edge : edges) {
            if (dist[edge.u] != INF && dist[edge.u] + edge.wt < dist[edge.v]) {
                dist[edge.v] = dist[edge.u] + edge.wt;
            }
        }
    }

    // Check for negative cycle
    for (auto edge : edges) {
        if (dist[edge.u] != INF && dist[edge.u] + edge.wt < dist[edge.v]) {
            cout << "Negative cycle detected\n";
            break;
        }
    }

    return dist;
}
```

---

### Floyd-Warshall Algorithm

Floyd-Warshall finds shortest distance between every pair of nodes.

Important:

- Good when `n` is small.
- Works with negative weights.
- Can detect negative cycles.
- Uses adjacency matrix.

Time complexity: `O(V^3)`

```cpp
void floydWarshall(vector<vector<int>>& dist) {
    int n = dist.size();

    for (int via = 0; via < n; via++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][via] == 1e9 || dist[via][j] == 1e9) {
                    continue;
                }

                // Try path i -> via -> j
                dist[i][j] = min(dist[i][j], dist[i][via] + dist[via][j]);
            }
        }
    }

    // If dist[i][i] < 0 for any i, there is a negative cycle
}
```

Matrix initialization:

```cpp
const int INF = 1e9;
vector<vector<int>> dist(n, vector<int>(n, INF));

for (int i = 0; i < n; i++) {
    dist[i][i] = 0;
}

for (int i = 0; i < m; i++) {
    int u, v, wt;
    cin >> u >> v >> wt;
    dist[u][v] = wt;
}
```

---

### Shortest Path in DAG

For a weighted DAG, shortest path can be found using topological sort.

Idea:

1. Find topological order.
2. Set `dist[src] = 0`.
3. Process nodes in topological order and relax outgoing edges.

```cpp
vector<int> shortestPathDAG(int n, vector<vector<pair<int, int>>>& adj, int src) {
    const int INF = 1e9;
    vector<vector<int>> simpleAdj(n);

    for (int u = 0; u < n; u++) {
        for (auto [v, wt] : adj[u]) {
            simpleAdj[u].push_back(v);
        }
    }

    vector<int> topo = kahnTopoSort(n, simpleAdj);
    vector<int> dist(n, INF);
    dist[src] = 0;

    for (int node : topo) {
        if (dist[node] == INF) {
            continue;
        }

        for (auto [neighbor, weight] : adj[node]) {
            if (dist[node] + weight < dist[neighbor]) {
                dist[neighbor] = dist[node] + weight;
            }
        }
    }

    return dist;
}
```

---

## 7. Spanning Tree and Minimum Spanning Tree

A spanning tree connects all vertices of a connected undirected graph using exactly `n - 1` edges.

A Minimum Spanning Tree, or MST, is a spanning tree with minimum total edge weight.

Important:

- MST is for connected, undirected, weighted graphs.
- Common algorithms: Prim's and Kruskal's.

---

### Prim's Algorithm

Prim's algorithm grows the MST by always choosing the smallest edge going from MST set to outside.

Time complexity: `O((V + E) log V)`

```cpp
int primMST(int n, vector<vector<pair<int, int>>>& adj) {
    vector<int> visited(n, 0);

    // {edgeWeight, node}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

    pq.push({0, 0});
    int mstWeight = 0;

    while (!pq.empty()) {
        auto [weight, node] = pq.top();
        pq.pop();

        if (visited[node]) {
            continue;
        }

        visited[node] = 1;
        mstWeight += weight;

        for (auto [neighbor, edgeWeight] : adj[node]) {
            if (!visited[neighbor]) {
                pq.push({edgeWeight, neighbor});
            }
        }
    }

    return mstWeight;
}
```

---

### Kruskal's Algorithm

Kruskal's algorithm sorts all edges by weight and picks edges that do not form a cycle.

It uses DSU.

Time complexity: `O(E log E)`

```cpp
class DSU {
public:
    vector<int> parent, size;

    DSU(int n) {
        parent.resize(n);
        size.resize(n, 1);

        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    int findParent(int node) {
        if (parent[node] == node) {
            return node;
        }

        // Path compression
        return parent[node] = findParent(parent[node]);
    }

    bool unionBySize(int u, int v) {
        int pu = findParent(u);
        int pv = findParent(v);

        if (pu == pv) {
            return false; // already connected
        }

        if (size[pu] < size[pv]) {
            swap(pu, pv);
        }

        parent[pv] = pu;
        size[pu] += size[pv];
        return true;
    }
};

struct Edge {
    int u, v, wt;
};

int kruskalMST(int n, vector<Edge>& edges) {
    sort(edges.begin(), edges.end(), [](Edge a, Edge b) {
        return a.wt < b.wt;
    });

    DSU dsu(n);
    int mstWeight = 0;
    int edgesUsed = 0;

    for (auto edge : edges) {
        if (dsu.unionBySize(edge.u, edge.v)) {
            mstWeight += edge.wt;
            edgesUsed++;
        }
    }

    // If edgesUsed != n - 1, graph was not connected
    return mstWeight;
}
```

---

## 8. DSU

DSU means Disjoint Set Union. It is also called Union-Find.

Common uses:

- Kruskal's MST
- Detect cycle in undirected graph
- Number of connected components
- Dynamic connectivity

Optimizations:

- Path compression
- Union by size/rank

Almost constant time per operation: `O(alpha(n))`

```cpp
class DSU {
public:
    vector<int> parent, rank;

    DSU(int n) {
        parent.resize(n);
        rank.resize(n, 0);

        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    int findParent(int node) {
        if (node == parent[node]) {
            return node;
        }

        // Path compression
        return parent[node] = findParent(parent[node]);
    }

    bool unionByRank(int u, int v) {
        int pu = findParent(u);
        int pv = findParent(v);

        if (pu == pv) {
            return false;
        }

        if (rank[pu] < rank[pv]) {
            parent[pu] = pv;
        } else if (rank[pv] < rank[pu]) {
            parent[pv] = pu;
        } else {
            parent[pv] = pu;
            rank[pu]++;
        }

        return true;
    }
};
```

---

### Cycle Detection in Undirected Graph Using DSU

If two nodes of an edge already belong to the same component, adding that edge creates a cycle.

```cpp
bool hasCycleUndirectedDSU(int n, vector<pair<int, int>>& edges) {
    DSU dsu(n);

    for (auto [u, v] : edges) {
        if (!dsu.unionByRank(u, v)) {
            return true;
        }
    }

    return false;
}
```

---

## Quick Revision Table

| Topic | Main Data Structure | Best For |
|---|---|---|
| DFS | Recursion / stack | Deep traversal, cycle detection, topo sort |
| BFS | Queue | Level traversal, unweighted shortest path |
| Directed cycle | DFS path visited / Kahn | Detect cycle in directed graph |
| Undirected cycle | Parent tracking / DSU | Detect cycle in undirected graph |
| Topo sort DFS | Stack/list | Ordering DAG nodes |
| Kahn's algo | Queue + indegree | Topo sort and cycle detection |
| Dijkstra | Min heap | Non-negative weighted shortest path |
| Bellman-Ford | Edge list | Negative weight shortest path |
| Floyd-Warshall | Matrix | All-pairs shortest path |
| Prim | Min heap | MST from adjacency list |
| Kruskal | Sorting + DSU | MST from edge list |
| DSU | Parent + rank/size | Components, cycle detection, Kruskal |

---

## Common Mistakes

- Dijkstra does not work correctly with negative edge weights.
- Topological sort is valid only for directed acyclic graphs.
- For undirected cycle detection with DFS/BFS, always track parent.
- For directed cycle detection, parent tracking is not enough; use path visited.
- Floyd-Warshall needs proper `INF` handling before adding distances.
- MST algorithms are for undirected weighted graphs.
- In disconnected graphs, run DFS/BFS from every unvisited node.

