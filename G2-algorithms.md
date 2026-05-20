# G2 - 演算法(Algorithms)

← [返回索引(README.md)](./README.md)

---

## 為什麼有這一篇?

演算法是「**在資料結構上做事的方法**」——同一個 [Graph](./G1-data-structures.md#graph) 可以走 BFS 找最短路、走 DFS 找環、走 Dijkstra 找加權最短路。本章與 [G1 資料結構](./G1-data-structures.md) 配套——**G1 是名詞,G2 是動詞**。

本章只收**工程實務真的會用 / 系統設計題真的會考** 的演算法,不重複競賽刷題大全。

> 此章節為**逐步擴充**設計,新詞遇到再加入(未來預期會涵蓋:AVL/Segment Tree 操作、Bellman-Ford、Floyd-Warshall、KMP 自動機進階變形等)。

---

## 目錄

### 排序與搜尋
- [排序演算法總覽 🟡](#sorting)
- [Binary Search 🟢](#binary-search)

### 字串
- [KMP 🟡](#kmp)
- [Aho-Corasick(AC 自動機)🟡](#aho-corasick)
- [Edit Distance / Levenshtein 🟡](#edit-distance)

### 圖論演算法
- [BFS / DFS 🟢](#bfs-dfs)
- [Dijkstra 🟡](#dijkstra)
- [Topological Sort 🟡](#topo-sort)
- [Union-Find / Disjoint Set 🟡](#union-find)

### Dynamic Programming
- [DP 入門 🟡](#dp)

### 系統實戰演算法
- [LRU / LFU Cache 🟡](#lru-lfu)
- [Consistent Hashing 🟡](#consistent-hash)
- [Rate Limiting Algorithms 🟡](#rate-limiting)

---

## 排序與搜尋

<a id="sorting"></a>
### 排序演算法總覽 🟡

**完整對照表**:

| 演算法 | 平均 | 最壞 | 最佳 | 空間 | **穩定** | In-place | 備註 |
| --- | --- | --- | --- | --- | :---: | :---: | --- |
| Bubble Sort | O(N²) | O(N²) | O(N) | O(1) | ✅ | ✅ | 教學用,實務從不選 |
| Selection Sort | O(N²) | O(N²) | O(N²) | O(1) | ❌ | ✅ | 比較次數固定 |
| Insertion Sort | O(N²) | O(N²) | O(N) | O(1) | ✅ | ✅ | **小資料(< 10)反而最快** |
| **Merge Sort** | O(N log N) | O(N log N) | O(N log N) | O(N) | ✅ | ❌ | 分治經典,穩定 |
| **Quick Sort** | O(N log N) | O(N²) | O(N log N) | O(log N) | ❌ | ✅ | 工程常用,常數小 |
| **Heap Sort** | O(N log N) | O(N log N) | O(N log N) | O(1) | ❌ | ✅ | 用 [Heap](./G1-data-structures.md#heap) |
| **Tim Sort** | O(N log N) | O(N log N) | O(N) | O(N) | ✅ | ❌ | **Java / Python / JS 真實用的排序** |
| Radix Sort | O(N×K) | O(N×K) | O(N×K) | O(N+K) | ✅ | ❌ | 非比較排序,K 是位數 |
| Counting Sort | O(N+K) | O(N+K) | O(N+K) | O(N+K) | ✅ | ❌ | 適合值域小 |

**「穩定」是什麼**:相等的 key 排序前後相對順序不變。對「排了名字再排年齡」這種**多 key 排序**很關鍵。

**工程現實——你寫的 `sort()` 實際在用什麼**:

| 語言 / API | 演算法 |
| --- | --- |
| **Java `Arrays.sort(int[])`** | Dual-Pivot Quicksort |
| **Java `Arrays.sort(Object[])`** | **Tim Sort**(改良 Merge Sort) |
| **Java `Collections.sort(List)`** | Tim Sort |
| **Java `Stream.sorted()`** | Tim Sort |
| **Python `sorted()`** / `list.sort()` | Tim Sort(原創) |
| **JavaScript V8 `Array.sort`** | **Tim Sort**(2018 起;之前是 QuickSort) |
| **C++ `std::sort`** | IntroSort(Quick + Heap + Insertion) |

**Tim Sort 核心思想**:
1. 偵測**已排序的子序列(run)**——大資料常有局部有序
2. 短 run 用 Insertion Sort
3. 長 run 用 Merge Sort 合併
4. **最佳 O(N)**(全已排序時)

**何時手寫排序**:基本上**從不**(除了演算法教學或非常特殊需求)。內建排序已經被工程化到極致——你寫的版本九成九比較慢。

---

<a id="binary-search"></a>
### Binary Search 🟢

**定義**:在**排序陣列**上 O(log N) 找特定值——每次比較把搜尋區間切一半。

**經典寫法**:

```java
// 找 target 的 index,沒找到回 -1
int binarySearch(int[] arr, int target) {
    int l = 0, r = arr.length - 1;
    while (l <= r) {
        int mid = l + (r - l) / 2;        // ⚠ 避免整數溢位
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) l = mid + 1;
        else r = mid - 1;
    }
    return -1;
}
```

**三大邊界陷阱**(寫錯整段都壞):

| 陷阱 | 錯誤 | 正解 |
| --- | --- | --- |
| **整數溢位** | `mid = (l + r) / 2` 當 l+r > Integer.MAX_VALUE 會負數 | `mid = l + (r - l) / 2` |
| **無窮迴圈** | `r = mid` 但 `l <= r` 不更新時死循環 | 想清楚 `l = mid+1` 或 `r = mid-1` |
| **off-by-one** | `< target` 還是 `<= target` | 看是找「第一個 >= 」還是「最後一個 <= 」 |

**三大變形**(實戰常見):

| 問題 | 條件 |
| --- | --- |
| **找第一個 >= x**(lower_bound) | `arr[mid] < x` → `l = mid+1`,否則 `r = mid`,最後 `l` 即答案 |
| **找最後一個 <= x**(upper_bound - 1) | 對稱 |
| **無限陣列找 x** | 先指數倍擴大邊界 `r = 1, 2, 4, 8 ...` 找到第一個 `> x` 的位置,再二分 |

**Java 工具**:`Arrays.binarySearch(arr, key)` 和 `Collections.binarySearch(list, key)`——**但返回值對「找不到」不是 -1**,而是 `-(insertion_point + 1)`,要小心。

**何時用**:**資料是排序的**(或可排序)。對 RDBMS 索引、有序 list、需要找 boundary 的場景都適用。**對 hash 結構不需要——直接 O(1) 查就好**。

---

## 字串

<a id="kmp"></a>
### KMP(Knuth–Morris–Pratt)🟡

**定義**:**在 text 中找 pattern 的字串匹配演算法**,O(N + M)。N = text 長度,M = pattern 長度。

**為什麼出現**:朴素比對(`String.indexOf` 內部簡化版)最壞 O(N × M)——`text = "AAAAA...AAAB"`、`pattern = "AAAAB"` 時每次失敗要從頭來。

**核心想法**:**失敗時,pattern 自己往右滑多遠,不需要 text 回退**。預先計算 `next[]` 陣列告訴你:「pattern 的前 i 個字元,最長相同的前綴後綴是多長」——這就是不用重比的部分。

**示意**:
```
text:    A B C A B C A B D
pattern: A B C A B D
                    ↑ 比到這裡失敗
                    
next 表告訴我們:pattern 前 5 個字元 "ABCAB" 的最長相同前後綴是 "AB"(長 2)
所以 pattern 可以直接往右滑,從 next[5]=2 接著比,不用 text 回退
```

**Java 簡易實作**:
```java
public static int kmp(String text, String pattern) {
    int[] next = buildNext(pattern);
    int i = 0, j = 0;
    while (i < text.length() && j < pattern.length()) {
        if (j == -1 || text.charAt(i) == pattern.charAt(j)) {
            i++; j++;
        } else {
            j = next[j];        // pattern 滑動,text 不回退
        }
    }
    return j == pattern.length() ? i - j : -1;
}
```

**對比朴素比對**:
- 朴素 `String.indexOf`(Java 內部)— 短 pattern 通常夠快,且**JIT 高度優化**
- KMP — pattern 內部有重複時(`"AAAB"`、`"ABABABAB"`)優勢大,但**實務 Java 字串多用簡單比對**

**現代地位**:
- **單模式匹配**:工程中很少手寫 KMP(`indexOf` 夠)
- **多模式匹配**:KMP 不適合 → 用 [Aho-Corasick](#aho-corasick)
- 主要**教學價值** + 理解搜尋引擎、編譯器的 lexer

**親戚**:**Boyer-Moore**(實務常見,Linux `grep` 用)、**Rabin-Karp**(用滾動 hash,適合多 pattern 同長度)

---

<a id="aho-corasick"></a>
### Aho-Corasick(AC 自動機)🟡

**定義**:**[Trie](./G1-data-structures.md#trie) + 失敗連結(fail link)**——**一次掃描 text 比對多個 pattern**。複雜度 O(N + M + Z),N = text 長度、M = 所有 pattern 總長、Z = 匹配次數。

**為什麼出現**:
- [KMP](#kmp) 只能比 1 個 pattern
- 比 M 個 pattern 用 KMP 要跑 M 次 → O(M × N)
- AC 自動機把所有 pattern 建 Trie,**一次掃描全搞定**

**核心結構**:Trie + 每節點加一個 fail 指標(指到「**當前匹配前綴的最長 proper suffix 對應的 Trie 節點**」)——和 KMP 的 next 思想一致,只是泛化到樹上。

**運作示意**(pattern: `he`, `she`, `his`, `hers`):

```
            root
           / | \
          h  s  ...
         / \  \
        e   i  h
        |   |  |
        r   s  e  ← "she" 終點
        |
        s  ← "hers" 終點
```

掃 text `"ushers"`:
- 從 root 開始,`u` 沒匹配 → 停 root
- `s` → 走到 `s` 節點
- `h` → 走到 `sh` 節點
- `e` → 走到 `she` 節點 → **發現 "she" 命中**
- 此時 fail link 指到 `he` 節點 → **再發現 "he" 命中**
- `r` → 跟著 fail/Trie 走到 `her` 節點
- `s` → 走到 `hers` 節點 → **發現 "hers" 命中**

**哪裡用**(超廣):

| 場景 | 系統 |
| --- | --- |
| 中文切詞 | **[IK Analyzer](./F1-search-text.md#ik-analyzer)** 主詞典匹配 |
| 敏感詞過濾 | DFA-based 詞庫(微博、彈幕系統) |
| 防火牆 / IDS | Snort、Suricata 多 signature 比對 |
| Antivirus | 多病毒特徵碼掃描 |
| `grep -F multi-pattern` | GNU grep `-f` 多 pattern |
| 廣告關鍵字匹配 | 一段文字命中哪些關鍵字 |

**與 [G1 Trie](./G1-data-structures.md#trie) 關係**:AC 自動機 = Trie + 失敗連結。Trie 是「結構」,AC 是「在這結構上做多 pattern 匹配的演算法」——**G1/G2 分工的典型範例**。

---

<a id="edit-distance"></a>
### Edit Distance / Levenshtein 🟡

**定義**:把字串 A 變成字串 B,**最少需要幾次編輯操作**(insert / delete / replace,各算 1 次)。又稱 Levenshtein 距離。

**範例**:`kitten` → `sitting`,距離 = 3
- `kitten` → `sitten`(replace k → s)
- `sitten` → `sittin`(replace e → i)
- `sittin` → `sitting`(insert g)

**經典 DP**(2D 表格,O(M × N)):

```
       ""  s  i  t  t  i  n  g
   "" [ 0  1  2  3  4  5  6  7]
   k  [ 1  1  2  3  4  5  6  7]
   i  [ 2  2  1  2  3  4  5  6]
   t  [ 3  3  2  1  2  3  4  5]
   t  [ 4  4  3  2  1  2  3  4]
   e  [ 5  5  4  3  2  2  3  4]
   n  [ 6  6  5  4  3  3  2  3]

dp[i][j] = 把 A[0..i] 變成 B[0..j] 的最少操作數
  - 若 A[i] == B[j]:dp[i][j] = dp[i-1][j-1]
  - 否則:           dp[i][j] = 1 + min(dp[i-1][j],     // delete
                                       dp[i][j-1],     // insert  
                                       dp[i-1][j-1])   // replace
```

右下角 `dp[6][7] = 3` 即答案。

**Java 實作骨架**:
```java
int editDistance(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i-1) == b.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1];
            } else {
                dp[i][j] = 1 + Math.min(dp[i-1][j-1],
                                Math.min(dp[i-1][j], dp[i][j-1]));
            }
        }
    }
    return dp[m][n];
}
```

**空間優化**:只需保留前一列 → 用**滾動陣列**降到 O(min(M, N))。

**應用**:
- **拼字檢查**(找與輸入距離最小的詞典詞)
- **模糊搜尋 / Fuzzy Match**(Elasticsearch `fuzziness` 參數即 edit distance)
- **DNA 序列比對**(生物資訊)
- **OCR 後處理**(辨識結果與字典比對)
- **diff 工具**(`git diff` 內部變體)
- **F1 搜尋相關性**——配合 [TF-IDF](./F1-search-text.md#tf-idf) / [BM25](./F1-search-text.md#bm25) 提升模糊命中

**變體**:
- **Damerau-Levenshtein**:多支援「相鄰字元交換」(`th` ↔ `ht`)
- **Hamming Distance**:**等長** 字串,只算 replace
- **Jaro-Winkler**:對前綴匹配給高分,適合人名比對

**現代趨勢**:純 edit distance 對「**語意相似**」無感(`car` 和 `automobile` 距離很遠但語意相同)——現代搜尋越來越倚賴 [Embedding / 向量相似度](./F1-search-text.md#word-embedding)。

---

## 圖論演算法

<a id="bfs-dfs"></a>
### BFS / DFS 🟢

**[Graph](./G1-data-structures.md#graph) 的兩大基本走訪演算法**。

| 維度 | BFS(Breadth-First Search) | DFS(Depth-First Search) |
| --- | --- | --- |
| 資料結構 | **[Queue](./G1-data-structures.md#stack-queue-deque)** | **[Stack](./G1-data-structures.md#stack-queue-deque)**(顯式)/ 遞迴 |
| 探索順序 | **一層一層**外擴 | **一條路走到底,再回溯** |
| 找最短路徑 | ✅(**無權圖**) | ❌ |
| 空間 | O(W),W = 最寬一層的節點數 | O(H),H = 樹高 / 遞迴深度 |
| 適合 | 最近距離、層序、最短路 | 連通性、判環、拓樸排序、走迷宮 |

**BFS Java 實作**:
```java
void bfs(Map<Integer, List<Integer>> graph, int start) {
    Queue<Integer> queue = new ArrayDeque<>();
    Set<Integer> visited = new HashSet<>();
    queue.offer(start);
    visited.add(start);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        // 處理 node
        for (int next : graph.getOrDefault(node, List.of())) {
            if (visited.add(next)) {
                queue.offer(next);
            }
        }
    }
}
```

**DFS Java 實作(遞迴)**:
```java
void dfs(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
    if (!visited.add(node)) return;
    // 處理 node
    for (int next : graph.getOrDefault(node, List.of())) {
        dfs(graph, next, visited);
    }
}
```

**注意**:遞迴 DFS 深度過大會 **stack overflow**——大圖請用 **顯式 Stack 改寫**。

**經典應用對照**:

| 問題 | 演算法 |
| --- | --- |
| 無權圖最短路徑 | **BFS** |
| 最少幾步(社交網路 N 度好友) | **BFS** |
| 樹的層序遍歷 | **BFS** |
| 判斷有向圖有環 | **DFS**(三色標記法) |
| [Topological Sort](#topo-sort) | DFS(後序)或 BFS(Kahn) |
| 強連通分量(Tarjan / Kosaraju) | DFS |
| 走迷宮、迷宮所有解 | DFS |
| Flood Fill(填色) | BFS 或 DFS 都行 |

**陷阱**:**圖有環時必須記 visited**——否則無窮迴圈。樹則不需(無環本性)。

---

<a id="dijkstra"></a>
### Dijkstra 🟡

**定義**:**單源最短路演算法**——給定起點,找到圖中所有節點的最短距離。要求**邊權非負**。複雜度 O((V + E) log V) 用 priority queue 實作。

**核心想法**:**貪婪 + 鬆弛**——每次取「**目前距離最近的未訪節點**」,更新它鄰居的距離。

**演算法步驟**:
```
1. dist[start] = 0,其他 = ∞
2. 用 min-heap 存 (dist, node),先 push (0, start)
3. while heap 非空:
     取出最小 (d, u)
     若 u 已訪 → 跳過
     標記 u 已訪
     for u 的每個鄰居 v(邊權 w):
       若 dist[u] + w < dist[v]:
         dist[v] = dist[u] + w
         push (dist[v], v) to heap
```

**Java 實作**:
```java
int[] dijkstra(int n, List<int[]>[] graph, int start) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[start] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    pq.offer(new int[]{0, start});
    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int d = cur[0], u = cur[1];
        if (d > dist[u]) continue;        // 已被更新過,跳過
        for (int[] edge : graph[u]) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.offer(new int[]{dist[v], v});
            }
        }
    }
    return dist;
}
```

**使用 [Heap](./G1-data-structures.md#heap) 加速**:沒有 heap 版本 O(V²);有 heap O((V+E) log V),稀疏圖大幅快。

**限制 & 親戚**:

| 演算法 | 處理 |
| --- | --- |
| **Dijkstra** | 非負權,單源 |
| **Bellman-Ford** | **可負權**,可偵測負環,O(V × E) |
| **Floyd-Warshall** | **全源最短路**(每對節點),O(V³),DP |
| **A* (A-star)** | Dijkstra + 啟發式(heuristic),目標導向(遊戲 / 導航首選) |
| **Johnson's** | 全源最短路,稀疏圖 O(V² log V + VE) |

**應用**:
- **GPS / 地圖導航**(實際多用 A*)
- **網路路由**(OSPF 用 Dijkstra)
- **遊戲尋路**(A* 為主,Dijkstra 為底)
- **依賴解析**(找最短依賴鏈)

---

<a id="topo-sort"></a>
### Topological Sort 🟡

**定義**:**有向無環圖(DAG)** 的線性排序,使得**每條邊 u → v 都滿足 u 出現在 v 之前**。

**範例**(課程相依):
```
微積分 → 線性代數 → 機器學習
資料結構 → 演算法 → 機器學習

一個合法 topo 順序:
微積分 → 線性代數 → 資料結構 → 演算法 → 機器學習
(或:資料結構 → 微積分 → 線性代數 → 演算法 → 機器學習 ...)
```

**兩種演算法**:

### Kahn's Algorithm(BFS-based)
```
1. 計算每個節點的入度(in-degree)
2. 把入度 = 0 的節點塞 queue
3. while queue 非空:
     取出 u,加到結果
     for u 的每個鄰居 v:
       v 入度 -1
       若 v 入度 = 0,塞 queue
4. 若結果長度 < V,代表有環
```

```java
List<Integer> topoSort(int n, List<List<Integer>> graph) {
    int[] indegree = new int[n];
    for (int u = 0; u < n; u++)
        for (int v : graph.get(u)) indegree[v]++;
    
    Queue<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < n; i++) if (indegree[i] == 0) q.offer(i);
    
    List<Integer> result = new ArrayList<>();
    while (!q.isEmpty()) {
        int u = q.poll();
        result.add(u);
        for (int v : graph.get(u)) {
            if (--indegree[v] == 0) q.offer(v);
        }
    }
    return result.size() == n ? result : List.of();   // 空 list 代表有環
}
```

### DFS-based
**後序走訪 + 反向** = topological order。

**應用**(全在「**依賴順序**」的地方):

| 場景 | 系統 |
| --- | --- |
| **Build 系統** | Make / Maven / Gradle(編譯順序) |
| **套件相依** | npm / pip / Gradle 解析 |
| **Spring Bean 初始化順序** | DI container 解析 |
| **K8s Operator reconcile** | 資源建立順序 |
| **工作流 / DAG-based scheduler** | Airflow、Prefect、Luigi |
| **編譯器** | 表達式求值順序、type checking |
| **課程排序、任務排程** | 教學常見 |
| **判斷有向圖是否有環** | Kahn's 沒走完 = 有環 |

---

<a id="union-find"></a>
### Union-Find / Disjoint Set Union(DSU)🟡

**定義**:管理一組**不相交集合**的資料結構/演算法,主要操作:
- **`find(x)`**:x 屬於哪個集合(回傳代表元)
- **`union(x, y)`**:合併 x 和 y 所在的集合

加上兩個關鍵優化後,**幾乎是常數時間**(均攤 O(α(N)),α 是 Inverse Ackermann,實務 < 5)。

**兩大優化**(沒做這兩個 union-find 沒意義):

### Path Compression(路徑壓縮)
`find` 時把整條路徑上的節點直接掛到根。

```
原本:  x → a → b → root
find(x) 後: x → root
            a → root
            b → root
```

### Union by Rank / Size
合併時**矮的接到高的下面**,維持樹高。

**Java 實作**(經典 < 30 行):

```java
public class UnionFind {
    private int[] parent;
    private int[] rank;
    
    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    
    public int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);   // path compression
        return parent[x];
    }
    
    public boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;        // 已在同集合
        if (rank[rx] < rank[ry]) { parent[rx] = ry; }
        else if (rank[rx] > rank[ry]) { parent[ry] = rx; }
        else { parent[ry] = rx; rank[rx]++; }
        return true;
    }
}
```

**經典應用**:

| 場景 | 用法 |
| --- | --- |
| **Kruskal 最小生成樹** | 邊按權重排序,union 不在同集合的 |
| **動態連通性** | 加邊後問「u、v 是否連通?」 |
| **社交網路朋友圈計算** | 連通分量計數 |
| **Image segmentation** | 相鄰像素 union |
| **網路 partition 偵測** | 失聯節點群聚分析 |
| **HDU/LeetCode 經典題型** | 群體合併、等價類 |

**為什麼不用 BFS/DFS 算連通分量**:可以,但**動態加邊**時 union-find 每次 O(α(N)),BFS/DFS 要重跑 O(V+E)。

---

## Dynamic Programming

<a id="dp"></a>
### Dynamic Programming(DP)入門 🟡

**定義**:**把複雜問題拆成有重疊子問題的小問題,記下小問題結果以避免重算**。本質是「**遞迴 + 備忘錄**」的工程化思維。

**識別 DP 問題的三大訊號**:

1. **求最佳化**(max / min / count / 是否可行)
2. **重疊子問題**(同樣的子問題會被算很多次)
3. **最佳子結構**(大問題最佳解 = 小問題最佳解的組合)

**兩種寫法**:

### Top-Down(記憶化遞迴)
直覺、好寫,但有遞迴開銷。

```java
Map<Integer, Long> memo = new HashMap<>();

long fib(int n) {
    if (n < 2) return n;
    if (memo.containsKey(n)) return memo.get(n);
    long result = fib(n-1) + fib(n-2);
    memo.put(n, result);
    return result;
}
```

### Bottom-Up(Tabulation)
從小到大填表,**無遞迴開銷、可滾動陣列省空間**。

```java
long fib(int n) {
    if (n < 2) return n;
    long a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        long c = a + b;
        a = b;
        b = c;
    }
    return b;
}
```

**經典題型分類**:

| 類別 | 代表題目 | 狀態定義 |
| --- | --- | --- |
| **線性 DP** | 爬樓梯、LIS(最長遞增子序列)、LCS(最長公共子序列)、打家劫舍 | `dp[i]` |
| **區間 DP** | 矩陣鏈乘法、回文分割、戳氣球 | `dp[i][j]` |
| **揹包 DP** | 0/1 揹包、完全揹包、多重揹包、零錢兌換 | `dp[i][weight]` |
| **狀態壓縮 DP** | 旅行商問題(TSP)、棋盤覆蓋 | 用 bitmask 表示子集 |
| **樹形 DP** | 樹上 LIS、樹的直徑 | 在樹上遞迴 |
| **數位 DP** | 區間內滿足某條件的數字數量 | 按位 + 限制條件 |

**工程實務遇到 DP 的場景**(不只面試):
- **[Edit Distance](#edit-distance)**(拼字檢查、diff)
- **路徑規劃 / 資源配置最佳化**
- **機器學習 Viterbi**(HMM 解碼)
- **編譯器最佳化**(最佳暫存器分配的 DAG)
- **DP on 序列**:時間序列預測、文字 segmentation

**DP vs 貪婪 vs 分治**:

| 方法 | 適用 | 例子 |
| --- | --- | --- |
| **DP** | 有最佳子結構 + 重疊子問題 | LCS、揹包 |
| **貪婪** | 局部最佳 = 全域最佳 | Dijkstra、Huffman |
| **分治** | 無重疊子問題(無重複子問題) | Merge Sort、Quick Sort |

---

## 系統實戰演算法

<a id="lru-lfu"></a>
### LRU / LFU Cache 🟡

**Cache 滿了要丟誰?** 兩種主流策略:

| 策略 | 全名 | 規則 |
| --- | --- | --- |
| **LRU** | Least Recently Used | 丟「**最久沒用過的**」 |
| **LFU** | Least Frequently Used | 丟「**用最少次的**」 |

### LRU 經典實作

**[HashMap](./G1-data-structures.md#hash-table) + [Doubly Linked List](./G1-data-structures.md#linked-list)**,**O(1) get/put**:
- HashMap:`key → 節點`,O(1) 查找
- DoublyLinkedList:維持「**最近用的在頭、最久沒用的在尾**」
- `get(key)`:從 map 找節點,把節點移到頭
- `put(key, value)`:新節點插頭;滿了則刪尾

```java
public class LRUCache {
    static class Node {
        int key, value;
        Node prev, next;
        Node(int k, int v) { key = k; value = v; }
    }
    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0);    // dummy
    private final Node tail = new Node(0, 0);    // dummy
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        Node node = map.get(key);
        if (node == null) return -1;
        moveToHead(node);
        return node.value;
    }
    
    public void put(int key, int value) {
        Node node = map.get(key);
        if (node != null) {
            node.value = value;
            moveToHead(node);
            return;
        }
        node = new Node(key, value);
        map.put(key, node);
        addToHead(node);
        if (map.size() > capacity) {
            Node evict = tail.prev;
            removeNode(evict);
            map.remove(evict.key);
        }
    }
    
    private void moveToHead(Node node) { removeNode(node); addToHead(node); }
    private void addToHead(Node node)  { node.next = head.next; node.prev = head; head.next.prev = node; head.next = node; }
    private void removeNode(Node node) { node.prev.next = node.next; node.next.prev = node.prev; }
}
```

**Java 內建簡寫**:`LinkedHashMap` override `removeEldestEntry`:
```java
new LinkedHashMap<K, V>(16, 0.75f, true) {     // accessOrder = true
    protected boolean removeEldestEntry(Map.Entry e) {
        return size() > CAPACITY;
    }
};
```

### LFU
比 LRU 複雜——需追蹤每個 key 的頻率計數,**找最低頻率的**。常見實作:**HashMap + (頻率 → DoublyLinkedList) 的 map**。一般人手寫不出來。

### 工程實務

**手寫 LRU 主要是面試**——實務直接用:

| 工具 | 預設策略 | 備註 |
| --- | --- | --- |
| **Guava `Cache`** | LRU(可選 LFU) | Google,被 Caffeine 取代 |
| **Caffeine** | **W-TinyLFU**(2016 提出,**近代頂尖**) | 命中率超越 LRU/LFU |
| **Redis maxmemory-policy** | 多選:`allkeys-lru` / `volatile-lru` / `allkeys-lfu` / 純 TTL / random | 預設 `noeviction` |
| **Linux Page Cache** | 近似 LRU(雙列表) | 真實 OS 級別 LRU |
| **HTTP Browser Cache** | LRU + TTL | 結合過期策略 |
| **CDN edge cache** | LRU / LFU / 自家演算法 | Cloudflare / Akamai 各家不同 |

**W-TinyLFU 為何贏**:用 Count-Min Sketch 估頻率,搭配 LRU 視窗——**對「短期熱、長期不熱」的 key 處理特別好**(LRU 會留太多、LFU 會誤殺新進熱門)。

---

<a id="consistent-hash"></a>
### Consistent Hashing(一致性 Hash)🟡

**定義**:把 key 與 node 都 hash 到**同一個環**上(0 ~ 2³² - 1),key 順時針找到**第一個 node** 即歸屬。**加減 node 只影響 1/N 的 key**。

**為什麼出現**:傳統 `hash(key) % N` 改 N 時(加減 node)**幾乎所有 key 重新映射**——cache 大量 miss、雪崩。

**環狀示意**:

```
                    Node A (hash=100)
                    ●
              ●—————————●
       Node D       
     (hash=900)        ●  Node B (hash=300)
              ●
                    ●—————●
                    Node C (hash=600)

key "foo" hash=200 → 順時針找到 Node B
key "bar" hash=700 → 順時針找到 Node D
key "baz" hash=50  → 順時針找到 Node A

加入 Node E (hash=500):
  原本 hash 在 400~600 的 key 從 Node C 重新映射到 Node E
  其他 key 完全不變 → 只影響 1/N 的 key
```

**致命問題:資料不均**——hash 結果可能讓某些 node 區段大、有些小,負載不均。

**解法:Virtual Nodes(虛擬節點)**:
- 每個實體 node 對應多個虛擬節點(例如 150 個),hash 在環上多點分佈
- 大數法則拉平負載
- 同時也讓「**新增 node 從各 node 平均分擔**」,不是只搶一個鄰居

```
原本:Node A 一個點
虛擬節點:Node A 化身 A#1, A#2, ..., A#150,各佔不同 hash 區間
```

**典型應用**:

| 系統 | 用途 |
| --- | --- |
| **Cassandra** | 資料 partition |
| **DynamoDB** | 同上 |
| **Riak** | 同上 |
| **Memcached client(ketama)** | 分散式 cache 分片 |
| **CDN** | edge node 選擇(地理 + hash) |
| **Discord 訊息分片** | 用戶分配到 voice server |
| **Redis Cluster** | **不是純 consistent hash**,是 **slot-based**(16384 slots),概念類似但實作不同 |

**對照「**簡單取模 `hash % N`**」**:

| | hash % N | Consistent Hash |
| --- | --- | --- |
| 加 1 個 node | 幾乎所有 key 重新映射 | **只 1/N 的 key 移動** |
| 實作複雜度 | 低 | 中(尤其要 virtual nodes) |
| 適合 | 固定節點數 | **動態擴縮容** |

**衍生**:
- **Rendezvous Hashing(HRW)**:每 key 算所有 node 的 hash,挑最高分——數學更乾淨、無需 virtual nodes,**Google 內部多用**
- **Jump Consistent Hash**(Google 2014):極省記憶體、極快,但只能加在尾巴

---

<a id="rate-limiting"></a>
### Rate Limiting Algorithms 🟡

**目的**:**限制單位時間內的呼叫次數**——保護下游、避免被打死、計費。

**四種主流演算法**:

### 1. Fixed Window Counter(固定視窗)
每分鐘 0 秒歸零計數,計到上限就拒絕。

**問題:邊界突刺**——0:59 衝 100 次 + 1:00 又衝 100 次 = 1 秒內 200 次。

### 2. Sliding Window Log(滑動視窗日誌)
記錄每次請求的時間戳,當前時間往前 1 分鐘內的 log 數 = 計數。

**優點:精準**;**缺點:記憶體爆**(每請求一筆記錄)。

### 3. Sliding Window Counter(滑動視窗計數)
混合 Fixed Window 與 Sliding——當前 window 的計數 + 上一 window 計數 × (剩餘比例)。**省空間 + 平滑**。

### 4. Token Bucket(令牌桶)— **最常見**
- 桶內存 token,容量上限 N
- 每秒補 M 個 token(`refill rate`)
- 每次請求消耗 1 個 token,沒 token 就拒絕
- **允許短暫突發**(桶滿時可一次消耗 N 個)

```
桶容量 N=100,每秒補 M=10:
  正常情況:每秒最多通過 10 個
  突發情況:長期沒人用桶滿 100,瞬間可消耗 100 個再回到限速
```

### 5. Leaky Bucket(漏桶)
請求進桶,**桶以固定速率漏出**(處理請求)。桶滿就拒絕。

**Token Bucket 與 Leaky Bucket 的根本差異**:

| | Token Bucket | Leaky Bucket |
| --- | --- | --- |
| **允許突發** | ✅(桶滿時) | ❌(固定速率) |
| **輸出平滑** | 否 | **是**(嚴格定速) |
| 適合 | 對突發友善的 API | **需要平滑流量輸出**(防下游被瞬間打爆) |

**對照表**:

| 演算法 | 平滑度 | 允許突發 | 空間 | 實作難度 |
| --- | --- | --- | --- | --- |
| Fixed Window | ❌(邊界突刺) | 視位置 | O(1) | 最簡單 |
| Sliding Window Log | ✅ | ❌ | **O(請求數)** | 中 |
| Sliding Window Counter | 中 | ❌ | O(2) | 中 |
| **Token Bucket** | 中 | **✅** | O(1) | 簡單 |
| **Leaky Bucket** | ✅✅ | ❌ | O(桶容量) | 簡單 |

**分散式限流**(跨節點):
- **Redis** `INCR + EXPIRE`(Fixed Window)、Redis Lua(Sliding Window / Token Bucket)
- **Sentinel**(阿里巴巴,Java)
- **Bucket4j**(Java,搭配 Hazelcast / Redis 做分散式)
- **API Gateway 內建**:[APISIX](./D3-networking.md#apisix)、[Kong](./D3-networking.md#kong) 的 `limit-count` / `rate-limiting` plugin

**Java 生態實踐**:詳見 [B6 Rate Limiter](./B6-resilience.md#rate-limiter) — Resilience4j 配置範例與分散式限流選擇。本條目專注**演算法層次**,B6 專注**韌性模式 + 框架配置**。

**現代趨勢**:**Adaptive Rate Limiting**——根據後端負載動態調整速率(Netflix concurrency-limits、AWS Adaptive Retry),不再用固定數字。

---

← [返回索引(README.md)](./README.md)
