# Disjoint Set Union Revision

Disjoint Set Union, also called DSU or Union-Find, is used to manage groups of connected elements.

It supports two main operations:

- `findParent(x)`: tells which group/component `x` belongs to.
- `union(u, v)`: joins the groups of `u` and `v`.

DSU is very useful when we need to repeatedly connect nodes and check whether two nodes are already connected.

---

## 1. Basic Idea

Suppose every node starts in its own component.

```text
0   1   2   3   4
```

Initially:

```text
parent[0] = 0
parent[1] = 1
parent[2] = 2
parent[3] = 3
parent[4] = 4
```

If we union `1` and `2`, they become part of the same component.

If we union `2` and `3`, then `1`, `2`, and `3` become part of the same component.

DSU lets us check this quickly.

---

## 2. Why Parent Array?

Each component has one representative, also called parent or ultimate parent.

Example:

```text
1 -> 2 -> 3
```

Here, if `3` is the ultimate parent:

```text
findParent(1) = 3
findParent(2) = 3
findParent(3) = 3
```

If two nodes have the same ultimate parent, they are in the same component.

---

## 3. Naive DSU

This version works, but can become slow if the tree becomes long.

```cpp
class DSU {
public:
    vector<int> parent;

    DSU(int n) {
        parent.resize(n);

        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    int findParent(int node) {
        if (parent[node] == node) {
            return node;
        }

        return findParent(parent[node]);
    }

    void unionSet(int u, int v) {
        int pu = findParent(u);
        int pv = findParent(v);

        if (pu == pv) {
            return;
        }

        parent[pv] = pu;
    }
};
```

Problem: In worst case, `findParent` can take `O(n)`.

---

## 4. Path Compression

Path compression makes every node on the path directly point to the ultimate parent.

Before path compression:

```text
1 -> 2 -> 3 -> 4
```

After `findParent(1)`:

```text
1 -> 4
2 -> 4
3 -> 4
4 -> 4
```

Code:

```cpp
int findParent(int node) {
    if (parent[node] == node) {
        return node;
    }

    // Path compression
    return parent[node] = findParent(parent[node]);
}
```

This makes future queries much faster.

---

## 5. Union By Rank

Rank roughly means height of the tree.

Idea:

- Attach smaller-rank tree under bigger-rank tree.
- If both ranks are equal, attach one under the other and increase rank.

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
        if (parent[node] == node) {
            return node;
        }

        // Path compression
        return parent[node] = findParent(parent[node]);
    }

    bool unionByRank(int u, int v) {
        int pu = findParent(u);
        int pv = findParent(v);

        if (pu == pv) {
            return false; // already in same component
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

## 6. Union By Size

Size means number of nodes in the component.

Idea:

- Attach smaller component under larger component.
- Update size of the larger component.

Many people prefer union by size because it is easier to understand.

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
```

---

## 7. Time Complexity

With path compression and union by rank/size:

```text
findParent: almost O(1)
union: almost O(1)
```

Exact complexity is:

```text
O(alpha(n))
```

`alpha(n)` is inverse Ackermann function. For practical constraints, it behaves like constant time.

---

## 8. When to Use DSU

Use DSU when the problem has words like:

- connected components
- merge groups
- same group
- redundant connection
- cycle in undirected graph
- minimum spanning tree
- accounts merge
- provinces
- number of islands 2
- dynamic connectivity

DSU is generally useful when edges are added and we need to know whether two nodes are already connected.

---

## 9. Check If Two Nodes Are Connected

```cpp
bool isConnected(int u, int v, DSU& dsu) {
    return dsu.findParent(u) == dsu.findParent(v);
}
```

If ultimate parents are same, both nodes are in same component.

---

## 10. Count Connected Components

```cpp
int countComponents(int n, vector<vector<int>>& edges) {
    DSU dsu(n);

    for (auto edge : edges) {
        dsu.unionBySize(edge[0], edge[1]);
    }

    int components = 0;

    for (int i = 0; i < n; i++) {
        if (dsu.findParent(i) == i) {
            components++;
        }
    }

    return components;
}
```

Important: Count only nodes whose parent is themselves.

---

## 11. Cycle Detection in Undirected Graph

If an edge connects two nodes that already belong to the same component, then that edge creates a cycle.

```cpp
bool hasCycleUndirected(int n, vector<vector<int>>& edges) {
    DSU dsu(n);

    for (auto edge : edges) {
        int u = edge[0];
        int v = edge[1];

        if (!dsu.unionBySize(u, v)) {
            return true;
        }
    }

    return false;
}
```

Note: DSU cycle detection is for undirected graphs.

---

## 12. Redundant Connection

Given edges of an undirected graph, return the edge that creates a cycle.

```cpp
vector<int> findRedundantConnection(vector<vector<int>>& edges) {
    int n = edges.size();
    DSU dsu(n + 1); // nodes are usually 1-based in this problem

    for (auto edge : edges) {
        int u = edge[0];
        int v = edge[1];

        if (!dsu.unionBySize(u, v)) {
            return edge;
        }
    }

    return {};
}
```

---

## 13. Number of Provinces

Given an adjacency matrix, count how many connected components exist.

```cpp
int findCircleNum(vector<vector<int>>& isConnected) {
    int n = isConnected.size();
    DSU dsu(n);

    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            if (isConnected[i][j] == 1) {
                dsu.unionBySize(i, j);
            }
        }
    }

    int provinces = 0;

    for (int i = 0; i < n; i++) {
        if (dsu.findParent(i) == i) {
            provinces++;
        }
    }

    return provinces;
}
```

---

## 14. Kruskal's Algorithm Using DSU

Kruskal's algorithm finds Minimum Spanning Tree.

Steps:

1. Sort edges by weight.
2. Pick smallest edge if it does not create a cycle.
3. Use DSU to check cycle.

```cpp
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

    // If edgesUsed != n - 1, graph was disconnected
    return mstWeight;
}
```

---

## 15. Accounts Merge

If two accounts share an email, they belong to the same person.

Pattern:

- Map each email to account index.
- Union accounts that share the same email.
- Group emails by ultimate parent.

```cpp
vector<vector<string>> accountsMerge(vector<vector<string>>& accounts) {
    int n = accounts.size();
    DSU dsu(n);
    unordered_map<string, int> emailToIndex;

    for (int i = 0; i < n; i++) {
        for (int j = 1; j < accounts[i].size(); j++) {
            string email = accounts[i][j];

            if (emailToIndex.find(email) == emailToIndex.end()) {
                emailToIndex[email] = i;
            } else {
                dsu.unionBySize(i, emailToIndex[email]);
            }
        }
    }

    vector<vector<string>> mergedEmails(n);

    for (auto [email, index] : emailToIndex) {
        int parent = dsu.findParent(index);
        mergedEmails[parent].push_back(email);
    }

    vector<vector<string>> answer;

    for (int i = 0; i < n; i++) {
        if (mergedEmails[i].empty()) {
            continue;
        }

        sort(mergedEmails[i].begin(), mergedEmails[i].end());

        vector<string> account;
        account.push_back(accounts[i][0]);

        for (string email : mergedEmails[i]) {
            account.push_back(email);
        }

        answer.push_back(account);
    }

    return answer;
}
```

---

## 16. Number of Operations to Make Network Connected

To connect `n` nodes, we need at least `n - 1` edges.

If edges are enough, answer is:

```text
number of components - 1
```

```cpp
int makeConnected(int n, vector<vector<int>>& connections) {
    if (connections.size() < n - 1) {
        return -1;
    }

    DSU dsu(n);

    for (auto edge : connections) {
        dsu.unionBySize(edge[0], edge[1]);
    }

    int components = 0;

    for (int i = 0; i < n; i++) {
        if (dsu.findParent(i) == i) {
            components++;
        }
    }

    return components - 1;
}
```

---

## 17. Number of Islands II

Land is added one by one to a water grid. After every operation, return number of islands.

Pattern:

- Dynamic connectivity
- DSU on grid

Convert cell `(r, c)` into node number:

```text
node = r * cols + c
```

```cpp
vector<int> numOfIslands(int rows, int cols, vector<vector<int>>& operators) {
    DSU dsu(rows * cols);
    vector<vector<int>> visited(rows, vector<int>(cols, 0));
    vector<int> answer;
    int count = 0;

    int dr[] = {-1, 0, 1, 0};
    int dc[] = {0, 1, 0, -1};

    for (auto op : operators) {
        int r = op[0];
        int c = op[1];

        if (visited[r][c]) {
            answer.push_back(count);
            continue;
        }

        visited[r][c] = 1;
        count++;

        int node = r * cols + c;

        for (int i = 0; i < 4; i++) {
            int nr = r + dr[i];
            int nc = c + dc[i];

            if (nr >= 0 && nc >= 0 && nr < rows && nc < cols && visited[nr][nc]) {
                int adjNode = nr * cols + nc;

                if (dsu.findParent(node) != dsu.findParent(adjNode)) {
                    dsu.unionBySize(node, adjNode);
                    count--;
                }
            }
        }

        answer.push_back(count);
    }

    return answer;
}
```

---

## 18. Making a Large Island

Given a grid of `0` and `1`, change at most one `0` to `1` and find the largest island size.

Pattern:

- DSU on grid
- Store component sizes
- For each `0`, check unique neighboring components

```cpp
int largestIsland(vector<vector<int>>& grid) {
    int n = grid.size();
    DSU dsu(n * n);

    int dr[] = {-1, 0, 1, 0};
    int dc[] = {0, 1, 0, -1};

    // First, connect all existing land cells
    for (int r = 0; r < n; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == 0) {
                continue;
            }

            int node = r * n + c;

            for (int i = 0; i < 4; i++) {
                int nr = r + dr[i];
                int nc = c + dc[i];

                if (nr >= 0 && nc >= 0 && nr < n && nc < n && grid[nr][nc] == 1) {
                    int adjNode = nr * n + nc;
                    dsu.unionBySize(node, adjNode);
                }
            }
        }
    }

    int answer = 0;

    // Try converting every 0 into 1
    for (int r = 0; r < n; r++) {
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == 1) {
                continue;
            }

            set<int> uniqueParents;

            for (int i = 0; i < 4; i++) {
                int nr = r + dr[i];
                int nc = c + dc[i];

                if (nr >= 0 && nc >= 0 && nr < n && nc < n && grid[nr][nc] == 1) {
                    int adjNode = nr * n + nc;
                    uniqueParents.insert(dsu.findParent(adjNode));
                }
            }

            int sizeAfterChange = 1;

            for (int parent : uniqueParents) {
                sizeAfterChange += dsu.size[parent];
            }

            answer = max(answer, sizeAfterChange);
        }
    }

    // Case where grid already has all 1s
    for (int cell = 0; cell < n * n; cell++) {
        answer = max(answer, dsu.size[dsu.findParent(cell)]);
    }

    return answer;
}
```

This problem requires DSU with `size`, not only rank.

---

## 19. DSU Template to Memorize

This is the main template worth remembering.

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
        if (node == parent[node]) {
            return node;
        }

        return parent[node] = findParent(parent[node]);
    }

    bool unionBySize(int u, int v) {
        int pu = findParent(u);
        int pv = findParent(v);

        if (pu == pv) {
            return false;
        }

        if (size[pu] < size[pv]) {
            swap(pu, pv);
        }

        parent[pv] = pu;
        size[pu] += size[pv];
        return true;
    }
};
```

---

## 20. Common Mistakes

- Forgetting path compression in `findParent`.
- Mixing union by rank and union by size in the same DSU.
- Counting components without calling `findParent`.
- Using DSU cycle detection for directed graphs.
- Forgetting that some problems use `1`-based node labels.
- In grid DSU, using wrong formula for cell number.
- In island problems, double-counting the same component; use `set` for unique parents.
- In Kruskal, forgetting to sort edges by weight.

---

## Quick Revision Table

| Task | DSU Use |
|---|---|
| Check if two nodes are connected | Compare ultimate parents |
| Merge two components | `unionBySize` or `unionByRank` |
| Count components | Count nodes where `findParent(i) == i` |
| Detect cycle in undirected graph | Edge connects nodes with same parent |
| Kruskal MST | Add edge only if union succeeds |
| Accounts merge | Union accounts sharing email |
| Number of islands II | Union newly added land with land neighbors |
| Largest island | Combine unique neighboring components |

