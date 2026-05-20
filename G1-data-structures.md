# G1 - 資料結構(Data Structures)

← [返回索引(README.md)](./README.md)

---

## 為什麼有這一篇?

資料結構是**語言無關的 CS 基礎**——同一個 Trie,IK Analyzer 用 Java、jieba 用 Python、HanLP 用 Java + DAT,底層邏輯一致;HashMap 為什麼要在 Java 8 樹化、TreeMap 為何選 Red-Black、RDBMS 為何用 B+Tree 不用 BST,理解結構**才能解釋系統為何這樣設計**。本章與 [G2 演算法](./G2-algorithms.md) 配套——**G1 是名詞(結構),G2 是動詞(動作)**。

> 此章節為**逐步擴充**設計,新詞遇到再加入(未來預期會涵蓋:AVL、Segment Tree / Fenwick Tree 等)。

---

## 目錄

### 樹結構
- [Trie(字首樹)🟡](#trie)
- [Binary Tree(二元樹)🟢](#binary-tree)
- [BST(二元搜尋樹)🟢](#bst)
- [Red-Black Tree(紅黑樹)🟡](#red-black-tree)
- [B-Tree / B+Tree 🟡](#b-tree)

### 堆 / 優先佇列
- [Heap / Priority Queue 🟡](#heap)

### Hash 結構
- [Hash Table & 衝突處理 🟡](#hash-table)
- [Bloom Filter 🟡](#bloom-filter)

### 圖結構
- [Graph & 表示法 🟡](#graph)

### 線性結構
- [Linked List 🟢](#linked-list)
- [Stack / Queue / Deque 🟢](#stack-queue-deque)

### 系統實戰結構
- [Skip List 🟡](#skip-list)
- [LSM-Tree 🟡](#lsm-tree)

---

## 樹結構

<a id="trie"></a>
### Trie(字首樹)🟡

**定義**:每個節點代表一個字元的樹,從 root 沿邊走到 leaf 構成完整字串。同一前綴的字串**共享路徑**,因此**前綴查找 O(L)**(L = 字串長度),與字典大小**無關**。

**為什麼用 Trie 而不是 HashMap**:

| 操作 | HashMap | Trie |
| --- | --- | --- |
| 完整字串查找 | O(L) | O(L) |
| **前綴查找**(自動補全) | ❌ 不支援(需全表掃) | ✅ O(L + 結果數) |
| **找最長共同前綴** | ❌ | ✅ |
| **字典序遍歷** | ❌(hash 無序) | ✅(中序走訪) |
| 空間效率 | 緊湊 | 較鬆(每節點 26+ 指標) |

只要場景**涉及前綴**,Trie 幾乎是唯一答案。

**結構示意**(字典含 `cat`, `car`, `card`, `dog`):

```
        root
       /    \
      c      d
      |      |
      a      o
     / \     |
    t   r    g  ← end
        |
        d  ← end
```

走訪 `c → a → r → d` 即查到 `card`;走到 `c → a → r` 則同時是 `car` 與 `card` 的共同前綴。

**簡易 Java 實作**:

```java
public class Trie {
    static class Node {
        Map<Character, Node> children = new HashMap<>();
        boolean isEnd;
    }
    private final Node root = new Node();

    public void insert(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            cur = cur.children.computeIfAbsent(c, k -> new Node());
        }
        cur.isEnd = true;
    }

    public boolean search(String word) {
        Node n = traverse(word);
        return n != null && n.isEnd;
    }

    public boolean startsWith(String prefix) {
        return traverse(prefix) != null;
    }

    private Node traverse(String s) {
        Node cur = root;
        for (char c : s.toCharArray()) {
            cur = cur.children.get(c);
            if (cur == null) return null;
        }
        return cur;
    }
}
```

**主要變體**:

| 變體 | 特性 | 典型應用 |
| --- | --- | --- |
| **Standard Trie** | 每節點存一個字元 | 教學、小型字典 |
| **Compressed Trie / Radix Tree** | 只有一個子節點時合併路徑 | etcd、Linux kernel routing、IP 路由表 |
| **DAT(Double-Array Trie)** | 用兩個 int 陣列模擬 Trie,**極省空間、超高速** | HanLP、ICU、生產級切詞器 |
| **Suffix Trie / Suffix Array** | 字串**所有後綴**建 Trie | 子字串搜尋、生物序列比對 |
| **Aho-Corasick(AC 自動機)** | Trie + 失敗連結,**一次掃描比對多 pattern** | 敏感詞過濾、IK Analyzer 切詞核心(詳見 [G2 Aho-Corasick](./G2-algorithms.md#aho-corasick)) |

**Glossary 中的 Trie 應用**:
- [IK Analyzer](./F1-search-text.md#ik-analyzer) — `DictSegment` 用 Trie 建主詞典,切詞時走 AC 自動機
- [jieba-analysis](./F1-search-text.md#jieba) — 字典載入後建 prefix dict(本質是 Trie)
- [HanLP](./F1-search-text.md#hanlp) — 主詞典用 **DAT**,記憶體與速度都最佳
- URL Router(Spring `RequestMappingHandlerMapping` 內部用 Trie 變體做路徑匹配)
- 自動補全(input box 打字推薦下拉)

**陷阱**:
- **記憶體放大**:每節點都帶子節點 map / array,純 Standard Trie 在中文場景(Unicode 65,536 個 code point)**爆炸性肥大**——實務多用 DAT 或 HashMap 子節點
- **不適合稀疏字典**:只有 100 個字串就用 Standard Trie,空間反而**比 HashMap 浪費**
- **不取代 B-Tree**:Trie 適合「字串 key + 前綴」場景;RDBMS 索引仍以 B+Tree 為主(範圍查詢、磁碟 I/O 友善)

**現代趨勢**:大廠 NLP、搜尋、敏感詞引擎**幾乎都是 Trie + AC 自動機**,改進方向是 DAT / Compressed Radix 把空間進一步壓縮。

---

<a id="binary-tree"></a>
### Binary Tree(二元樹)🟢

**定義**:每節點**最多兩個子節點**(左、右)的樹結構。是所有進階樹結構的基礎——BST、Heap、AVL、Red-Black、B-Tree、AST(抽象語法樹)都是二元樹的變體或推廣。

**重要變體**:

| 變體 | 定義 |
| --- | --- |
| **滿二元樹(Full)** | 每節點要嘛 0 個子,要嘛 2 個子 |
| **完全二元樹(Complete)** | 除最後一層外都填滿,最後一層從左到右填——Heap 的形狀 |
| **平衡二元樹(Balanced)** | 左右子樹高度差有限——保證 O(log N) 操作 |
| **退化(Degenerate)** | 變成 linked list(每節點只一個子)——最壞 BST 插入排序資料的下場 |

**四種走訪**(基於遞迴定義):

| 走訪 | 順序 | 用途 |
| --- | --- | --- |
| **前序(Pre-order)** | 根 → 左 → 右 | 複製樹、序列化 |
| **中序(In-order)** | 左 → 根 → 右 | **BST 中序 = 排序輸出** |
| **後序(Post-order)** | 左 → 右 → 根 | 刪除樹、計算大小 |
| **層序(Level-order)** | 一層一層 BFS | 印 tree、計算高度 |

**Java 實作(以中序走訪為例)**:
```java
record TreeNode(int val, TreeNode left, TreeNode right) {}

void inorder(TreeNode node, List<Integer> result) {
    if (node == null) return;
    inorder(node.left(), result);
    result.add(node.val());
    inorder(node.right(), result);
}
```

**為什麼是基礎**:理解 Binary Tree 才能順理解 [BST](#bst)、[Red-Black Tree](#red-black-tree)、[Heap](#heap)、[B-Tree](#b-tree)——這些都是「**多塞東西進 binary tree 節點 / 多加平衡規則**」的結果。

---

<a id="bst"></a>
### BST(二元搜尋樹)🟢

**定義**:對每個節點,**左子樹所有節點值 < 根 < 右子樹所有節點值** 的二元樹。中序走訪即排序輸出。

**操作複雜度**:

| 操作 | 平均 | **最壞** |
| --- | --- | --- |
| Search | O(log N) | **O(N)** |
| Insert | O(log N) | **O(N)** |
| Delete | O(log N) | **O(N)** |

**致命缺陷**:**最壞退化成 linked list**——對排序資料連續插入(`1, 2, 3, 4, ...`),樹高 = N。

```
1, 2, 3, 4, 5 依序插入 → 全部往右掛
1
 \
  2
   \
    3
     \
      4
       \
        5
```

**修正方式 = 自平衡**:
- [AVL Tree](#avl-tree)(嚴格平衡,操作慢)
- [Red-Black Tree](#red-black-tree)(寬鬆平衡,實務首選)

**為什麼還要學 BST**:不是 BST 本身好用(實務沒人用裸 BST),而是 **AVL / RB / B-Tree 的動機都建立在「BST 退化問題」上**。沒懂 BST 就沒辦法理解為何要平衡。

---

<a id="red-black-tree"></a>
### Red-Black Tree(紅黑樹)🟡

**定義**:**自平衡 BST**,用「節點塗紅或黑 + 5 條規則」維持樹高近似平衡——所有操作最壞 **O(log N)**。

**5 條規則**:
1. 每節點非紅即黑
2. **根節點是黑**
3. **NULL 葉節點視為黑**
4. **紅節點的子節點必為黑**(不能有兩個連續紅)
5. **任一節點到所有葉的路徑,黑節點數量相同**(black-height)

→ 規則 4 + 5 確保「最長路徑 ≤ 2 × 最短路徑」,樹高 O(log N)。

**操作**:Insert / Delete 後可能違規 → 用 **旋轉 + 變色** 修復。

**哪裡見過**(極廣):

| 系統 / 語言 | 用途 |
| --- | --- |
| **Java `TreeMap` / `TreeSet`** | 排序 Map / Set |
| **C++ `std::map` / `std::set`** | 同上 |
| **Linux CFS scheduler** | 進程時間調度(找最該執行的進程) |
| **Linux kernel epoll** | 監聽 fd 的紅黑樹 |
| **Java 8+ HashMap** | 衝突鏈 > 8 時樹化(防 hash collision attack) |

**Red-Black vs AVL Tree**:

| 維度 | AVL | Red-Black |
| --- | --- | --- |
| 平衡嚴格度 | **嚴格**(左右高度差 ≤ 1) | 寬鬆(最長 ≤ 2× 最短) |
| 樹高上限 | 1.44 log N | 2 log N |
| 查找速度 | **稍快**(樹較矮) | 略慢 |
| 插入/刪除 | **較慢**(旋轉次數多) | **較快** |
| 實務首選 | 唯讀為主 | **混合讀寫(主流)** |

實務系統幾乎都選 Red-Black——AVL 太嚴格,旋轉成本超過查找的優勢。

---

<a id="b-tree"></a>
### B-Tree / B+Tree 🟡

**定義**:**每節點存多個 key 與多個子節點指標** 的平衡多叉樹——不是「Binary Tree」(別被名字騙)。專為**磁碟 I/O 友善**設計。

**為什麼出現**:
- BST / Red-Black 每節點只存 1 個 key,讀取 N 個 key 需要 log N 次磁碟存取
- 磁碟一次 I/O 讀 4KB ~ 16KB **(一個 page)**,只用 8 bytes 太浪費
- B-Tree 把**一個 page 塞 100+ 個 key**,樹高大幅降低(百萬筆只需 3~4 層)

**B-Tree 結構**(M-ary,每節點最多 M 個子):

```
            [30, 70]
           /   |    \
       [10,20] [40,50,60] [80,90,100]
```

**B+Tree(B-Tree 的改良,實務主流)**:
1. **所有資料在 leaf**,內部節點只當索引
2. **Leaf 之間用 linked list 串起來** → **超高效範圍查詢**

```
                [30, 70]                ← 內部節點(只是索引)
               /   |    \
        [10,20] [40,50,60] [80,90,100]  ← leaf(實際資料)
            ↓——————→————————→            ← linked list 串接
```

**B+Tree 為何贏**:`WHERE age BETWEEN 20 AND 30` 只要找到 20 後走 leaf chain 即可,**不需要回根**。

**哪裡用**(幾乎所有 RDBMS 與檔案系統):

| 系統 | 結構 |
| --- | --- |
| **MySQL InnoDB** | B+Tree(clustered index) |
| **PostgreSQL** | B+Tree(預設索引型別) |
| **Oracle / SQL Server** | B+Tree |
| **MongoDB** | B-Tree |
| **ext4 / NTFS / HFS+** | B-Tree / B+Tree 變體 |
| **CouchDB** | append-only B+Tree |

**對照 [LSM-Tree](#lsm-tree)**:B+Tree 讀快寫慢(隨機 I/O);LSM 寫快讀稍慢——選哪個看 workload。

**與 Glossary 關聯**:[B4 持久層](./B4-persistence.md) 的 JPA / Hibernate 查詢效能,根本上取決於資料庫的 B+Tree 索引——理解 B+Tree 才能解釋為何複合索引「最左前綴原則」、為何 `LIKE 'abc%'` 走索引但 `LIKE '%abc'` 不走。

---

## 堆 / 優先佇列

<a id="heap"></a>
### Heap / Priority Queue 🟡

**定義**:**滿足堆性質的完全二元樹**——Max Heap 父 ≥ 子,Min Heap 父 ≤ 子。**動態取最值** O(log N),比每次排序整個陣列高效。

**注意**:「Heap」這個資料結構**和記憶體「heap」(JVM 堆積、malloc heap)是完全不同的概念**——後者只是借用「一堆東西」的字面意義。

**為何用陣列存**(不用節點指標):

```
索引:  0  1  2  3  4  5  6
值:   [9, 7, 8, 3, 5, 6, 2]

對應的樹:
        9          ← index 0
       / \
      7   8        ← index 1, 2
     / \ / \
    3  5 6  2      ← index 3, 4, 5, 6

公式: 父 = (i-1)/2,左子 = 2i+1,右子 = 2i+2
```

完全二元樹 + 陣列表示 = **零指標開銷、cache-friendly**。

**操作**(Max Heap):

| 操作 | 步驟 | 複雜度 |
| --- | --- | --- |
| **Peek** | 直接讀 index 0 | O(1) |
| **Insert** | 加到末尾 → 向上比較交換(swim) | O(log N) |
| **Extract-Max** | 取 index 0 → 末尾搬到 0 → 向下交換(sink) | O(log N) |
| **Build Heap** | 從中間往前 sink | **O(N)**(不是 O(N log N)) |

**經典應用**:
- **Top-K 問題**:維持大小為 K 的 min heap,O(N log K)
- **[Dijkstra 最短路](./G2-algorithms.md#dijkstra)**:取「目前距離最近的未訪節點」
- **Merge K sorted lists**:每條 list 的頭塞 min heap
- **任務排程器**:取「最早該執行的任務」
- **作業系統 Process Scheduler**(部分實作)

**Java 範例**:
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();          // 預設 min heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

minHeap.offer(3); minHeap.offer(1); minHeap.offer(2);
minHeap.poll();   // 1
minHeap.peek();   // 2
```

**注意**:`PriorityQueue` 走訪順序**不是排序順序**(只能保證 poll 的順序對)——iterator 印出來是亂的。

---

## Hash 結構

<a id="hash-table"></a>
### Hash Table & 衝突處理 🟡

**定義**:用 hash function 把 key 映射到陣列 index,**平均 O(1)** 查找/插入/刪除。

**致命議題:衝突(Collision)**——不同 key 算出同 index 怎麼辦?

**兩大流派**:

### Chaining(分離鏈)
每個 index 掛一條 linked list,撞 hash 的 key 都串上去。

```
index 0: → [k1,v1] → [k7,v7] → [k13,v13]
index 1: → [k2,v2]
index 2: (空)
index 3: → [k4,v4] → [k9,v9]
```

**優點**:實作簡單、刪除直觀、不怕 load factor > 1
**缺點**:cache 不友善(指標跳)

### Open Addressing(開放定址)
撞了就找下一個空位,**所有 key 都在陣列裡**:
- **Linear Probing**:i+1, i+2, i+3...(連續往後找,可能 primary clustering)
- **Quadratic Probing**:i+1², i+2², i+3²
- **Double Hashing**:用第二個 hash 算 step

**優點**:cache 友善、無額外指標
**缺點**:刪除困難(要用 tombstone 標記)、load factor 須 < 1

**Java HashMap 內部**(必懂):

| 項目 | 細節 |
| --- | --- |
| 衝突解法 | **Chaining** |
| **Java 8 樹化** | 同 bucket 鏈長 > **8** 時升級成 [Red-Black Tree](#red-black-tree),小於 **6** 時降回 list |
| 為何樹化 | 防止**惡意撞 hash 攻擊**(讓所有 key 落同 bucket,O(N) DoS) |
| Load Factor | 預設 **0.75**——超過就 **rehash**(陣列翻倍 + 重新分配) |
| 初始 capacity | 預設 **16**,必為 2 的次方(用位元運算取 index) |

**HashMap 內部結構**:
```
table: [Node[]] 陣列
Node = { hash, key, value, next }     ← linked list 節點
TreeNode extends Node { ... }         ← Java 8 樹化後的紅黑樹節點
```

**經典坑**:
- **HashMap 不是執行緒安全**——並發改動 → 死迴圈(Java 7 環狀鏈)或資料錯(Java 8+),要用 `ConcurrentHashMap`
- **修改作為 key 的物件 hashCode** → key 永遠找不到(物件還在 map 裡,但去新 bucket 查空的)
- **沒 override `equals`/`hashCode`** → 永遠不相等

**對照其他語言**:
- C++ `std::unordered_map`:chaining
- Python `dict`:**open addressing + random probing**(防 hash flooding)
- Go `map`:chaining + bucket-based

---

<a id="bloom-filter"></a>
### Bloom Filter 🟡

**定義**:**機率資料結構**——用 `k` 個 hash function + bit array 判斷「**X 一定不在集合**」或「**X 可能在集合**」。

**核心承諾**:

| | 結果 |
| --- | --- |
| 說「**不在**」 | **一定不在**(零 false negative) |
| 說「**在**」 | **可能在**,也可能誤報(有 false positive) |

**為什麼用**:超省空間——10 億筆字串只需幾 MB(換成 HashSet 要幾十 GB)。

**運作示意**:

```
插入 "apple":  hash1=3, hash2=7, hash3=11  → bit[3]=bit[7]=bit[11]=1
插入 "banana": hash1=2, hash2=7, hash3=15  → bit[2]=bit[7]=bit[15]=1

查 "apple":   bit[3]&bit[7]&bit[11]=1     → 可能在
查 "cherry":  hash1=5, hash2=9, hash3=13
              bit[5]=0                    → 一定不在
查 "grape":   hash1=2, hash2=11, hash3=15
              三個 bit 都是 1            → 「可能在」(其實沒插入過 = false positive)
```

**大小估算**:給定 n 個元素、目標誤報率 p:

```
m = -(n × ln p) / (ln 2)²
k = (m/n) × ln 2
```

例:n=10^6,p=1%,需要 m ≈ 9.6M bits = 1.2MB(對比 HashSet 數十 MB)。

**哪裡用**:
- **Cassandra / HBase / RocksDB / LevelDB**:讀請求先查 Bloom Filter,**不在就直接回**,不用查磁碟 SSTable(讀放大救星)
- **Chrome / Safari**:危險 URL 預先過濾
- **CRLite**(詳見 [D1 CRL / OCSP](./D1-security-jwt.md#crl-ocsp))
- **CDN**:edge 判斷「這個 key 我沒 cache 過,別問了」
- **垃圾郵件過濾、爬蟲去重**

**侷限**:
- **不能刪除**(刪 bit 會破壞別人)——衍生 **Counting Bloom Filter**(每位置存計數)
- **不能列舉內容**(只能查)

**親戚**:
- **Cuckoo Filter**:支援刪除、空間更省、查更快
- **HyperLogLog**:估算「**集合有多少 unique 元素**」(只算 cardinality,不查成員)
- **Count-Min Sketch**:估算「**X 出現幾次**」(頻率)

---

## 圖結構

<a id="graph"></a>
### Graph & 表示法 🟡

**定義**:**頂點(Vertex / Node)** + **邊(Edge)** 的集合。可說是最通用的資料結構——樹是無環連通圖,linked list 是線性圖。

**分類**:

| 維度 | 類別 | 例子 |
| --- | --- | --- |
| 方向 | **有向**(Directed)/ **無向**(Undirected) | Twitter follow 有向、Facebook friend 無向 |
| 權重 | **加權**(Weighted)/ **無權** | 路網有距離 / 社交網路無 |
| 環 | **有環**(Cyclic)/ **無環**(Acyclic) | 課程相依 = **DAG**(有向無環圖) |
| 連通 | **連通**(Connected)/ **不連通** | 多個獨立群組 |

**兩種表示法**:

### Adjacency List(鄰接表)
每節點存「相鄰節點 list」。

```
0 → [1, 2]
1 → [0, 3]
2 → [0, 3]
3 → [1, 2]
```

- **空間 O(V + E)**——稀疏圖(現實多數)首選
- **查鄰居 O(degree)**
- 查「u 和 v 是否相鄰」要 O(degree(u))

```java
Map<Integer, List<Integer>> graph = new HashMap<>();
graph.computeIfAbsent(0, k -> new ArrayList<>()).add(1);
```

### Adjacency Matrix(鄰接矩陣)
V × V 二維陣列,`matrix[u][v] = 1` 代表有邊。

```
     0  1  2  3
  0 [0, 1, 1, 0]
  1 [1, 0, 0, 1]
  2 [1, 0, 0, 1]
  3 [0, 1, 1, 0]
```

- **空間 O(V²)**——稠密圖(邊接近 V²)較划算
- **查相鄰 O(1)**
- 浪費空間於稀疏圖

**如何選**:

| 場景 | 用 |
| --- | --- |
| 一般圖(稀疏,V 大) | Adjacency List |
| 稠密圖、頻繁查相鄰 | Adjacency Matrix |
| 邊很少、需要存邊本身屬性 | **Edge List**(只存 `(u, v, w)` 列表) |

**走訪(BFS / DFS)詳見 [G2 BFS / DFS](./G2-algorithms.md#bfs-dfs)**——這裡只談「結構長怎樣」,走法是演算法層次的事。

**現實系統**:
- 社交網路(LinkedIn 知名 graph DB Neo4j)
- 知識圖譜(Wikidata、Google Knowledge Graph)
- 路網(Google Maps)
- Spring Bean 相依、Maven 相依、Build 系統(全是 DAG → 走 [Topological Sort](./G2-algorithms.md#topo-sort))

---

## 線性結構

<a id="linked-list"></a>
### Linked List 🟢

**定義**:每節點存 `data + next 指標` 的鏈式結構,記憶體不連續。

**變體**:

| 變體 | 結構 | 用途 |
| --- | --- | --- |
| **單向(Singly)** | `data → next` | 簡單佇列 |
| **雙向(Doubly)** | `prev ← data → next` | 兩端操作、[LRU Cache](./G2-algorithms.md#lru-lfu) |
| **循環(Circular)** | 尾接首 | Round-robin、ring buffer |

**對比 Array(關鍵)**:

| 操作 | Array(ArrayList) | LinkedList |
| --- | --- | --- |
| **隨機讀取** `get(i)` | **O(1)** | O(N) |
| **頭部插入** | O(N)(整列搬) | **O(1)** |
| **中間插入** | O(N) | O(1)(但**找位置 O(N)**) |
| **尾部插入** | O(1) 攤銷 | O(1)(維護 tail 指標) |
| **記憶體** | 連續、**cache 友善** | 不連續、cache miss 頻繁 |

**工程現實**:**多數情況 `ArrayList` 比 `LinkedList` 快**——cache locality 的優勢勝過理論複雜度。Java 工程師應預設 ArrayList,**只有「頻繁兩端操作」**(用 `ArrayDeque` 也行)才考慮 LinkedList。

**典型應用**(linked list 真正不可替代之處):
- **[HashMap 衝突鏈](#hash-table)**(Java 8 樹化前)
- **[LRU Cache](./G2-algorithms.md#lru-lfu)**(HashMap + DoublyLinkedList)
- **Stream 處理**(Java `Stream` 內部某些操作)
- **Undo / Redo 鏈**

---

<a id="stack-queue-deque"></a>
### Stack / Queue / Deque 🟢

三種**受限存取**的線性結構——只能從固定位置進出。

| 結構 | 規則 | 縮寫 |
| --- | --- | --- |
| **Stack** | 後進先出 | **LIFO** |
| **Queue** | 先進先出 | **FIFO** |
| **Deque**(Double-Ended Queue) | 兩端都可進出 | 通吃 |

**用途**:

| 結構 | 經典用途 |
| --- | --- |
| Stack | **函式呼叫(call stack)**、[DFS](./G2-algorithms.md#bfs-dfs)、運算式求值、Undo、瀏覽器歷史 |
| Queue | [BFS](./G2-algorithms.md#bfs-dfs)、任務排程、訊息佇列、CPU scheduling |
| Deque | [LRU Cache](./G2-algorithms.md#lru-lfu)、Sliding Window 最值、Round-Robin |

**Java 介面對照(容易混淆)**:

| 類別 | 線程安全 | 用途 | 現代是否推薦 |
| --- | --- | --- | --- |
| `java.util.Stack` | ✅(synchronized) | Stack | **❌(legacy,改用 ArrayDeque)** |
| `Deque<E>` interface | 看實作 | Stack 或 Queue | ✅ |
| **`ArrayDeque<E>`** | ❌ | Stack / Queue 通用 | **✅ 預設選這個** |
| `LinkedList<E>` | ❌ | Stack / Queue / Deque | ✅(實作 Deque,但效能略遜 ArrayDeque) |
| `Queue<E>` interface | 看實作 | Queue | ✅ |
| `BlockingQueue<E>` | ✅ | 生產者-消費者、Thread Pool | ✅ |
| `ConcurrentLinkedQueue` | ✅(無鎖) | 高並發 Queue | ✅ |
| `PriorityQueue` | ❌ | 詳見 [Heap](#heap) | ✅ |

**用 Stack 還是 ArrayDeque?** 官方文件直接寫:**"A more complete and consistent set of LIFO stack operations is provided by the `Deque` interface and its implementations, which should be used in preference to this class."** 不要用 `Stack`。

**ArrayDeque vs LinkedList 當 Queue**:
- ArrayDeque:陣列實作,cache 友善,**快 2~3 倍**
- LinkedList:節點實作,記憶體碎片,慢

**結論**:Java 99% 場景**直接用 `ArrayDeque`**,不論你需要 Stack 還是 Queue。

---

## 系統實戰結構

<a id="skip-list"></a>
### Skip List 🟡

**定義**:**多層 linked list**——底層全部節點,上層每隔幾個取一個,**用高層稀疏指標跳躍快速查找**。期望 O(log N) 查/插/刪。

**結構示意**(找 17):

```
Level 3: 1 ─────────────────────── 25
Level 2: 1 ────── 9 ─────── 17 ─── 25
Level 1: 1 ─ 4 ─ 9 ── 12 ── 17 ─── 25
Level 0: 1 ─ 4 ─ 9 ── 12 ── 17 ─── 25 ─ 30  ← 完整 linked list
```

從 Level 3 開始,過頭就下層、不過頭就右——log N 步跳到目標。

**為什麼用 Skip List 而不是 [Red-Black Tree](#red-black-tree)**:

| 維度 | Red-Black Tree | Skip List |
| --- | --- | --- |
| 平均時間複雜度 | O(log N) | O(log N) |
| 實作難度 | **複雜**(旋轉 / 變色 / 刪除特例) | **簡單**(隨機決定層數,linked list 操作) |
| **並發友善** | 困難(rebalance 需大範圍鎖) | **容易**(局部修改、無需 rebalance) |
| 範圍查詢 | 中序遍歷 | **底層走 linked list,自然支援** |
| 記憶體 | 每節點 2 指標 + 顏色 | 平均 2 個 next 指標 |

**哪裡用**(超關鍵):

| 系統 | 用途 |
| --- | --- |
| **Redis ZSET(sorted set)** | 核心結構——`ZRANGE`、`ZRANGEBYSCORE` 都用 Skip List |
| **LevelDB / RocksDB MemTable** | 寫入緩衝,排序儲存 |
| **Java `ConcurrentSkipListMap` / `ConcurrentSkipListSet`** | 並發排序 Map/Set |
| HBase MemStore | 儲存 sorted KV |

**為什麼 Redis 不用 RB Tree**:Antirez(Redis 作者)的官方理由——**Skip List 實作簡單、性能相當、並發更友善、範圍查詢更自然**。對 Redis 這種 single-threaded 系統,簡單性 = 可維護性。

---

<a id="lsm-tree"></a>
### LSM-Tree 🟡

**定義**:**Log-Structured Merge-Tree**——把所有寫入變成 **append-only**,後台批次合併排序。**寫入優化**,犧牲讀放大換取極致寫入吞吐。

**為什麼出現**:
- B+Tree 寫入需 **random I/O**(找到 leaf → 改 page → 寫回磁碟)
- LSM 把寫入變成**順序追加**(SSD 也愛、傳統磁碟更愛)
- 適合**寫多讀少 / 寫遠多於讀** 的場景

**結構分層**:

```
寫路徑:
  Client write
      ↓
  WAL (Write-Ahead Log,先落磁碟保資料安全)
      ↓
  MemTable (記憶體,通常用 Skip List 保持排序)
      ↓ 達到閾值
  Immutable MemTable
      ↓ flush 到磁碟
  SSTable Level 0 (Sorted String Table,不可變)
      ↓ Compaction
  SSTable Level 1 / 2 / 3 ...

讀路徑:
  查 MemTable → 沒有就查 Immutable MemTable → 還沒就由新到舊查各 Level SSTable
  (用 Bloom Filter 加速「這 SSTable 一定沒有」的判斷)
```

**Compaction(關鍵)**:後台合併多個 SSTable,**回收已被覆蓋 / 刪除的資料**,維持讀效能。常見策略:
- **Leveled**(LevelDB / RocksDB 預設)— 讀放大小、寫放大大
- **Tiered**(Cassandra 預設)— 寫放大小、讀放大大
- **Universal** / **FIFO** — 特殊場景

**B+Tree vs LSM-Tree**:

| 維度 | B+Tree | LSM-Tree |
| --- | --- | --- |
| 寫入 | random I/O,慢 | **順序 append,極快** |
| 讀取 | 直接定位 leaf | **可能查多個 SSTable(讀放大)** |
| 空間放大 | 低 | 中(舊資料未 compact 前) |
| 寫放大 | 低 | **高**(Compaction 重寫) |
| 適合 | OLTP、讀為主 | **OLAP、寫多讀少、時序資料** |

**哪裡用**(現代 NoSQL 主流):

| 系統 | 備註 |
| --- | --- |
| **Cassandra** | 經典 LSM 實作 |
| **RocksDB / LevelDB** | Facebook / Google KV store,**被超多系統當底層**(TiDB、CockroachDB、Kafka Streams) |
| **HBase** | Hadoop 生態 |
| **InfluxDB** | 時序資料庫(寫多) |
| **TiDB / CockroachDB** | 分散式 SQL(底層 RocksDB) |
| **ScyllaDB** | C++ 重寫 Cassandra |

**踩坑**:
- **Compaction 偶發大延遲**——後台 flush 突然吃 CPU/IO,p99 延遲爆衝
- **空間放大**——刪掉的 key 直到 compact 才真正釋放磁碟
- 不適合**讀頻繁、改頻繁** 的 OLTP——MySQL 還是首選

**與 Glossary 關聯**:[B4 持久層](./B4-persistence.md) 選資料庫時這是關鍵考量——traditional RDBMS(B+Tree)vs NoSQL / 時序(LSM)的選擇本質上是「**讀寫比 + 一致性需求**」的取捨。

---

← [返回索引(README.md)](./README.md)
