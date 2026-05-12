# Famous Graph Questions Revision

This file contains common graph problems and their main patterns.

Most templates use C++ and `0`-based indexing.

---

## 1. Bipartite Graph

A graph is bipartite if we can color every node using two colors such that no adjacent nodes have the same color.

Common approaches:

- BFS coloring
- DFS coloring

Works for disconnected graphs too, so check every component.

Time complexity: `O(V + E)`

```cpp
bool isBipartite(int n, vector<vector<int>>& adj) {
    vector<int> color(n, -1); // -1 means uncolored, 0 and 1 are colors

    for (int start = 0; start < n; start++) {
        if (color[start] != -1) {
            continue;
        }

        queue<int> q;
        q.push(start);
        color[start] = 0;

        while (!q.empty()) {
            int node = q.front();
            q.pop();

            for (int neighbor : adj[node]) {
                if (color[neighbor] == -1) {
                    color[neighbor] = 1 - color[node];
                    q.push(neighbor);
                } else if (color[neighbor] == color[node]) {
                    return false;
                }
            }
        }
    }

    return true;
}
```

Key idea: If an edge connects two nodes of the same color, graph is not bipartite.

---

## 2. Shortest Path With K Stops

Famous problem: Cheapest Flights Within K Stops.

Given flights `{u, v, price}`, find minimum cost from `src` to `dst` using at most `k` stops.

Important:

- Stops are different from edges.
- At most `k` stops means at most `k + 1` edges.
- Normal Dijkstra can fail if we ignore number of stops.

Time complexity: usually `O(K * E)` or close to it.

```cpp
int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
    vector<vector<pair<int, int>>> adj(n);

    for (auto flight : flights) {
        int u = flight[0];
        int v = flight[1];
        int price = flight[2];
        adj[u].push_back({v, price});
    }

    // {stopsUsed, {node, cost}}
    queue<pair<int, pair<int, int>>> q;
    q.push({0, {src, 0}});

    vector<int> dist(n, 1e9);
    dist[src] = 0;

    while (!q.empty()) {
        auto [stops, info] = q.front();
        q.pop();

        int node = info.first;
        int cost = info.second;

        // Cannot use more than k + 1 edges
        if (stops > k) {
            continue;
        }

        for (auto [neighbor, price] : adj[node]) {
            if (cost + price < dist[neighbor]) {
                dist[neighbor] = cost + price;
                q.push({stops + 1, {neighbor, dist[neighbor]}});
            }
        }
    }

    return dist[dst] == 1e9 ? -1 : dist[dst];
}
```

Note: This BFS-style approach works because we process by number of stops. Another clean method is Bellman-Ford for `k + 1` relaxations.

---

### Shortest Path With K Stops Using Bellman-Ford

This version is often easier to reason about.

```cpp
int findCheapestPriceBellman(int n, vector<vector<int>>& flights, int src, int dst, int k) {
    const int INF = 1e9;
    vector<int> dist(n, INF);
    dist[src] = 0;

    // At most k stops means at most k + 1 edges
    for (int i = 0; i <= k; i++) {
        vector<int> temp = dist;

        for (auto flight : flights) {
            int u = flight[0];
            int v = flight[1];
            int price = flight[2];

            if (dist[u] != INF && dist[u] + price < temp[v]) {
                temp[v] = dist[u] + price;
            }
        }

        dist = temp;
    }

    return dist[dst] == INF ? -1 : dist[dst];
}
```

Key idea: Use a copy array so every round uses only paths with one more edge.

---

## 3. Number of Islands

Given a grid of `'1'` and `'0'`, count connected groups of land.

Pattern:

- Grid DFS/BFS
- Connected components

Time complexity: `O(n * m)`

```cpp
void dfsIsland(int r, int c, vector<vector<char>>& grid) {
    int n = grid.size();
    int m = grid[0].size();

    if (r < 0 || c < 0 || r >= n || c >= m || grid[r][c] == '0') {
        return;
    }

    grid[r][c] = '0'; // mark visited

    dfsIsland(r + 1, c, grid);
    dfsIsland(r - 1, c, grid);
    dfsIsland(r, c + 1, grid);
    dfsIsland(r, c - 1, grid);
}

int numIslands(vector<vector<char>>& grid) {
    int n = grid.size();
    int m = grid[0].size();
    int count = 0;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (grid[i][j] == '1') {
                count++;
                dfsIsland(i, j, grid);
            }
        }
    }

    return count;
}
```

---

## 4. Rotten Oranges

Given a grid:

- `0` = empty
- `1` = fresh orange
- `2` = rotten orange

Each minute, rotten oranges rot adjacent fresh oranges. Find minimum time to rot all oranges.

Pattern:

- Multi-source BFS

Time complexity: `O(n * m)`

```cpp
int orangesRotting(vector<vector<int>>& grid) {
    int n = grid.size();
    int m = grid[0].size();

    queue<pair<pair<int, int>, int>> q; // {{row, col}, time}
    int fresh = 0;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (grid[i][j] == 2) {
                q.push({{i, j}, 0});
            } else if (grid[i][j] == 1) {
                fresh++;
            }
        }
    }

    int time = 0;
    int dr[] = {-1, 0, 1, 0};
    int dc[] = {0, 1, 0, -1};

    while (!q.empty()) {
        auto [cell, t] = q.front();
        q.pop();

        int r = cell.first;
        int c = cell.second;
        time = max(time, t);

        for (int i = 0; i < 4; i++) {
            int nr = r + dr[i];
            int nc = c + dc[i];

            if (nr >= 0 && nc >= 0 && nr < n && nc < m && grid[nr][nc] == 1) {
                grid[nr][nc] = 2;
                fresh--;
                q.push({{nr, nc}, t + 1});
            }
        }
    }

    return fresh == 0 ? time : -1;
}
```

---

## 5. Flood Fill

Change the color of a connected component in a grid.

Pattern:

- DFS/BFS on grid

```cpp
void dfsFill(int r, int c, vector<vector<int>>& image, int oldColor, int newColor) {
    int n = image.size();
    int m = image[0].size();

    if (r < 0 || c < 0 || r >= n || c >= m || image[r][c] != oldColor) {
        return;
    }

    image[r][c] = newColor;

    dfsFill(r + 1, c, image, oldColor, newColor);
    dfsFill(r - 1, c, image, oldColor, newColor);
    dfsFill(r, c + 1, image, oldColor, newColor);
    dfsFill(r, c - 1, image, oldColor, newColor);
}

vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
    int oldColor = image[sr][sc];

    if (oldColor == color) {
        return image;
    }

    dfsFill(sr, sc, image, oldColor, color);
    return image;
}
```

---

## 6. Course Schedule

Given prerequisites, check whether all courses can be completed.

Pattern:

- Directed graph cycle detection
- Kahn's algorithm

If graph has cycle, impossible to complete all courses.

```cpp
bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> adj(numCourses);
    vector<int> indegree(numCourses, 0);

    for (auto p : prerequisites) {
        int course = p[0];
        int prereq = p[1];

        // prereq must come before course
        adj[prereq].push_back(course);
        indegree[course]++;
    }

    queue<int> q;
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) {
            q.push(i);
        }
    }

    int completed = 0;

    while (!q.empty()) {
        int node = q.front();
        q.pop();
        completed++;

        for (int neighbor : adj[node]) {
            indegree[neighbor]--;
            if (indegree[neighbor] == 0) {
                q.push(neighbor);
            }
        }
    }

    return completed == numCourses;
}
```

---

## 7. Course Schedule II

Return one valid order of courses.

Pattern:

- Topological sort using Kahn's algorithm

```cpp
vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> adj(numCourses);
    vector<int> indegree(numCourses, 0);

    for (auto p : prerequisites) {
        int course = p[0];
        int prereq = p[1];
        adj[prereq].push_back(course);
        indegree[course]++;
    }

    queue<int> q;
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) {
            q.push(i);
        }
    }

    vector<int> order;

    while (!q.empty()) {
        int node = q.front();
        q.pop();
        order.push_back(node);

        for (int neighbor : adj[node]) {
            indegree[neighbor]--;
            if (indegree[neighbor] == 0) {
                q.push(neighbor);
            }
        }
    }

    if (order.size() != numCourses) {
        return {}; // cycle exists
    }

    return order;
}
```

---

## 8. Eventual Safe States

A node is safe if every path starting from it eventually reaches a terminal node.

Pattern:

- DFS cycle detection
- Or reverse graph + Kahn

DFS states:

- `0` = unvisited
- `1` = currently visiting
- `2` = safe

```cpp
bool dfsSafe(int node, vector<vector<int>>& graph, vector<int>& state) {
    state[node] = 1;

    for (int neighbor : graph[node]) {
        if (state[neighbor] == 1) {
            return false; // cycle
        }

        if (state[neighbor] == 0 && !dfsSafe(neighbor, graph, state)) {
            return false;
        }
    }

    state[node] = 2;
    return true;
}

vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> state(n, 0);
    vector<int> safe;

    for (int i = 0; i < n; i++) {
        if (state[i] == 2 || dfsSafe(i, graph, state)) {
            safe.push_back(i);
        }
    }

    return safe;
}
```

---

## 9. Number of Provinces

Given an adjacency matrix, count connected components.

Pattern:

- DFS/BFS
- DSU

```cpp
void dfsProvince(int node, vector<vector<int>>& isConnected, vector<int>& visited) {
    visited[node] = 1;
    int n = isConnected.size();

    for (int neighbor = 0; neighbor < n; neighbor++) {
        if (isConnected[node][neighbor] == 1 && !visited[neighbor]) {
            dfsProvince(neighbor, isConnected, visited);
        }
    }
}

int findCircleNum(vector<vector<int>>& isConnected) {
    int n = isConnected.size();
    vector<int> visited(n, 0);
    int provinces = 0;

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            provinces++;
            dfsProvince(i, isConnected, visited);
        }
    }

    return provinces;
}
```

---

## 10. Network Delay Time

Given directed weighted edges, find time for signal to reach all nodes from source.

Pattern:

- Dijkstra

```cpp
int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    vector<vector<pair<int, int>>> adj(n + 1);

    for (auto edge : times) {
        int u = edge[0];
        int v = edge[1];
        int wt = edge[2];
        adj[u].push_back({v, wt});
    }

    vector<int> dist(n + 1, 1e9);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;

    dist[k] = 0;
    pq.push({0, k});

    while (!pq.empty()) {
        auto [d, node] = pq.top();
        pq.pop();

        if (d > dist[node]) {
            continue;
        }

        for (auto [neighbor, wt] : adj[node]) {
            if (d + wt < dist[neighbor]) {
                dist[neighbor] = d + wt;
                pq.push({dist[neighbor], neighbor});
            }
        }
    }

    int answer = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == 1e9) {
            return -1;
        }
        answer = max(answer, dist[i]);
    }

    return answer;
}
```

---

## 11. Path With Minimum Effort

Given a height grid, effort of a path is the maximum absolute difference between adjacent cells. Find path with minimum effort.

Pattern:

- Dijkstra on grid

```cpp
int minimumEffortPath(vector<vector<int>>& heights) {
    int n = heights.size();
    int m = heights[0].size();

    vector<vector<int>> dist(n, vector<int>(m, 1e9));
    priority_queue<pair<int, pair<int, int>>,
                   vector<pair<int, pair<int, int>>>,
                   greater<pair<int, pair<int, int>>>> pq;

    dist[0][0] = 0;
    pq.push({0, {0, 0}});

    int dr[] = {-1, 0, 1, 0};
    int dc[] = {0, 1, 0, -1};

    while (!pq.empty()) {
        auto [effort, cell] = pq.top();
        pq.pop();

        int r = cell.first;
        int c = cell.second;

        if (r == n - 1 && c == m - 1) {
            return effort;
        }

        for (int i = 0; i < 4; i++) {
            int nr = r + dr[i];
            int nc = c + dc[i];

            if (nr >= 0 && nc >= 0 && nr < n && nc < m) {
                int newEffort = max(effort, abs(heights[r][c] - heights[nr][nc]));

                if (newEffort < dist[nr][nc]) {
                    dist[nr][nc] = newEffort;
                    pq.push({newEffort, {nr, nc}});
                }
            }
        }
    }

    return 0;
}
```

---

## 12. Word Ladder

Find minimum transformations from `beginWord` to `endWord`, changing one character at a time.

Pattern:

- BFS shortest path
- Each word is a node

```cpp
int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
    unordered_set<string> st(wordList.begin(), wordList.end());

    if (st.find(endWord) == st.end()) {
        return 0;
    }

    queue<pair<string, int>> q;
    q.push({beginWord, 1});
    st.erase(beginWord);

    while (!q.empty()) {
        auto [word, steps] = q.front();
        q.pop();

        if (word == endWord) {
            return steps;
        }

        for (int i = 0; i < word.size(); i++) {
            char original = word[i];

            for (char ch = 'a'; ch <= 'z'; ch++) {
                word[i] = ch;

                if (st.find(word) != st.end()) {
                    st.erase(word);
                    q.push({word, steps + 1});
                }
            }

            word[i] = original;
        }
    }

    return 0;
}
```

---

## 13. Alien Dictionary

Given sorted words from an alien language, find character order.

Pattern:

- Build directed graph
- Topological sort

```cpp
string alienOrder(vector<string>& words) {
    unordered_map<char, vector<char>> adj;
    unordered_map<char, int> indegree;

    for (string word : words) {
        for (char ch : word) {
            indegree[ch] = 0;
        }
    }

    for (int i = 0; i + 1 < words.size(); i++) {
        string a = words[i];
        string b = words[i + 1];
        int len = min(a.size(), b.size());
        bool foundDifference = false;

        for (int j = 0; j < len; j++) {
            if (a[j] != b[j]) {
                adj[a[j]].push_back(b[j]);
                indegree[b[j]]++;
                foundDifference = true;
                break;
            }
        }

        // Invalid case: "abc" before "ab"
        if (!foundDifference && a.size() > b.size()) {
            return "";
        }
    }

    queue<char> q;
    for (auto [ch, deg] : indegree) {
        if (deg == 0) {
            q.push(ch);
        }
    }

    string order = "";

    while (!q.empty()) {
        char ch = q.front();
        q.pop();
        order += ch;

        for (char neighbor : adj[ch]) {
            indegree[neighbor]--;
            if (indegree[neighbor] == 0) {
                q.push(neighbor);
            }
        }
    }

    return order.size() == indegree.size() ? order : "";
}
```

---

## 14. Accounts Merge

Merge accounts that share common emails.

Pattern:

- DSU
- Map email to account index

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

    void unionBySize(int u, int v) {
        int pu = findParent(u);
        int pv = findParent(v);

        if (pu == pv) {
            return;
        }

        if (size[pu] < size[pv]) {
            swap(pu, pv);
        }

        parent[pv] = pu;
        size[pu] += size[pv];
    }
};

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

## 15. Number of Operations to Make Network Connected

Given `n` computers and connections, find minimum operations to connect all computers.

Pattern:

- DSU
- Connected components

Key rule: Need at least `n - 1` edges. If edges are fewer, impossible.

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

## 16. Minimum Spanning Tree Questions

Common problem names:

- Min cost to connect all points
- Connecting cities with minimum cost
- Minimum spanning tree weight

Pattern:

- Prim's algorithm when graph is easy to build as adjacency list.
- Kruskal's algorithm when edge list is already available.
- DSU helps avoid cycles in Kruskal.

---

### Min Cost to Connect All Points

Complete graph where edge cost is Manhattan distance.

Pattern:

- Prim's algorithm without building all edges explicitly

```cpp
int minCostConnectPoints(vector<vector<int>>& points) {
    int n = points.size();
    vector<int> minDist(n, 1e9);
    vector<int> visited(n, 0);

    minDist[0] = 0;
    int answer = 0;

    for (int edgeCount = 0; edgeCount < n; edgeCount++) {
        int node = -1;

        for (int i = 0; i < n; i++) {
            if (!visited[i] && (node == -1 || minDist[i] < minDist[node])) {
                node = i;
            }
        }

        visited[node] = 1;
        answer += minDist[node];

        for (int neighbor = 0; neighbor < n; neighbor++) {
            int dist = abs(points[node][0] - points[neighbor][0])
                     + abs(points[node][1] - points[neighbor][1]);

            if (!visited[neighbor] && dist < minDist[neighbor]) {
                minDist[neighbor] = dist;
            }
        }
    }

    return answer;
}
```

---

## 17. Surrounded Regions

Replace all `'O'` regions surrounded by `'X'`.

Pattern:

- Boundary DFS/BFS
- Mark safe cells first

```cpp
void dfsBoundary(int r, int c, vector<vector<char>>& board) {
    int n = board.size();
    int m = board[0].size();

    if (r < 0 || c < 0 || r >= n || c >= m || board[r][c] != 'O') {
        return;
    }

    board[r][c] = '#'; // safe cell connected to boundary

    dfsBoundary(r + 1, c, board);
    dfsBoundary(r - 1, c, board);
    dfsBoundary(r, c + 1, board);
    dfsBoundary(r, c - 1, board);
}

void solve(vector<vector<char>>& board) {
    int n = board.size();
    int m = board[0].size();

    for (int i = 0; i < n; i++) {
        dfsBoundary(i, 0, board);
        dfsBoundary(i, m - 1, board);
    }

    for (int j = 0; j < m; j++) {
        dfsBoundary(0, j, board);
        dfsBoundary(n - 1, j, board);
    }

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (board[i][j] == 'O') {
                board[i][j] = 'X';
            } else if (board[i][j] == '#') {
                board[i][j] = 'O';
            }
        }
    }
}
```

---

## 18. Pacific Atlantic Water Flow

Find cells from where water can flow to both oceans.

Pattern:

- Reverse DFS/BFS from boundaries
- Intersection of reachable cells

```cpp
void dfsOcean(int r, int c, vector<vector<int>>& heights, vector<vector<int>>& visited) {
    int n = heights.size();
    int m = heights[0].size();
    visited[r][c] = 1;

    int dr[] = {-1, 0, 1, 0};
    int dc[] = {0, 1, 0, -1};

    for (int i = 0; i < 4; i++) {
        int nr = r + dr[i];
        int nc = c + dc[i];

        if (nr >= 0 && nc >= 0 && nr < n && nc < m &&
            !visited[nr][nc] && heights[nr][nc] >= heights[r][c]) {
            dfsOcean(nr, nc, heights, visited);
        }
    }
}

vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
    int n = heights.size();
    int m = heights[0].size();

    vector<vector<int>> pacific(n, vector<int>(m, 0));
    vector<vector<int>> atlantic(n, vector<int>(m, 0));

    for (int i = 0; i < n; i++) {
        dfsOcean(i, 0, heights, pacific);
        dfsOcean(i, m - 1, heights, atlantic);
    }

    for (int j = 0; j < m; j++) {
        dfsOcean(0, j, heights, pacific);
        dfsOcean(n - 1, j, heights, atlantic);
    }

    vector<vector<int>> answer;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (pacific[i][j] && atlantic[i][j]) {
                answer.push_back({i, j});
            }
        }
    }

    return answer;
}
```

---

## 19. Clone Graph

Deep copy an undirected graph.

Pattern:

- DFS/BFS
- Map old node to copied node

```cpp
class Node {
public:
    int val;
    vector<Node*> neighbors;

    Node() {
        val = 0;
    }

    Node(int _val) {
        val = _val;
    }

    Node(int _val, vector<Node*> _neighbors) {
        val = _val;
        neighbors = _neighbors;
    }
};

Node* cloneGraph(Node* node) {
    if (node == NULL) {
        return NULL;
    }

    unordered_map<Node*, Node*> mp;
    queue<Node*> q;

    mp[node] = new Node(node->val);
    q.push(node);

    while (!q.empty()) {
        Node* current = q.front();
        q.pop();

        for (Node* neighbor : current->neighbors) {
            if (mp.find(neighbor) == mp.end()) {
                mp[neighbor] = new Node(neighbor->val);
                q.push(neighbor);
            }

            mp[current]->neighbors.push_back(mp[neighbor]);
        }
    }

    return mp[node];
}
```

---

## 20. Reconstruct Itinerary

Use all tickets exactly once and return lexicographically smallest route.

Pattern:

- Eulerian path
- DFS with min-heap or sorted adjacency

```cpp
void dfsItinerary(string airport, unordered_map<string, priority_queue<string, vector<string>, greater<string>>>& adj, vector<string>& route) {
    while (!adj[airport].empty()) {
        string nextAirport = adj[airport].top();
        adj[airport].pop();
        dfsItinerary(nextAirport, adj, route);
    }

    route.push_back(airport);
}

vector<string> findItinerary(vector<vector<string>>& tickets) {
    unordered_map<string, priority_queue<string, vector<string>, greater<string>>> adj;

    for (auto ticket : tickets) {
        adj[ticket[0]].push(ticket[1]);
    }

    vector<string> route;
    dfsItinerary("JFK", adj, route);
    reverse(route.begin(), route.end());

    return route;
}
```

---

## Quick Pattern Map

| Problem Type | Use This Pattern |
|---|---|
| Check two-color possible | Bipartite BFS/DFS |
| Minimum edges in unweighted graph | BFS |
| Weighted non-negative shortest path | Dijkstra |
| Shortest path with at most K stops | BFS by stops / Bellman-Ford K rounds |
| All-pairs shortest path | Floyd-Warshall |
| Course dependency possible | Kahn / directed cycle detection |
| Count connected groups | DFS/BFS/DSU |
| Grid infection/time spread | Multi-source BFS |
| Boundary-safe grid cells | Boundary DFS/BFS |
| Merge connected records | DSU |
| Minimum connection cost | MST: Prim/Kruskal |
| Lexicographically smallest route using all edges | Eulerian path DFS |

---

## Quick Checklist Before Solving

- Is the graph directed or undirected?
- Is it weighted or unweighted?
- Are weights negative?
- Do we need shortest path, connectivity, ordering, or cycle detection?
- Is the input a grid, edge list, adjacency list, or matrix?
- Does the graph have multiple components?
- Is the answer asking for count, path, order, minimum cost, or true/false?

