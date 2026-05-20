# F1 - 搜尋與文字處理(Search & Text Processing)

← [返回索引(README.md)](./README.md)

---

## 為什麼有這一篇?

中文搜尋是 Java 後端常見但**專業性高**的場景——切詞、相關性計分、向量化、向量搜尋,每一塊都有獨立的工具與術語。本章收錄相關詞彙,方便你在 Elasticsearch 配置、搜尋功能討論、上下游系統對接時快速查找。

> 此章節為**逐步擴充**設計,新詞遇到再加入。

---

## 目錄

### 中文切詞工具
- [IK Analyzer 🟡](#ik-analyzer)
- [jieba-analysis 🟡](#jieba)
- [HanLP 🟡](#hanlp)

### 相關性計分
- [TF-IDF 🟡](#tf-idf)
- [BM25 🟡](#bm25)
- [Cosine Similarity 🟡](#cosine)

### 向量化(Embedding)
- [Word Embedding 🔴](#word-embedding)
- [Word2Vec 🔴](#word2vec)
- [BERT Embeddings / Sentence-BERT 🔴](#bert)
- [nd4j 🔴](#nd4j)

### 向量搜尋
- [kNN / ANN / Dense Vector Search 🟡](#knn-ann)
- [向量資料庫 🟡](#vector-db)

### Elasticsearch 進階
- [Analyzer / Tokenizer 🟡](#analyzer)
- [Custom Scoring 🔴](#custom-scoring)

### 企業資料平台
- [Microsoft Fabric 🔴](#microsoft-fabric)

---

## 中文切詞工具

中文沒有空白做天然分詞(英文有),因此**搜尋之前必須先切詞**。`"我愛吃蘋果"` → `["我", "愛", "吃", "蘋果"]`。切詞的選擇直接影響搜尋品質。

> **底層資料結構**:三家切詞器(IK / jieba / HanLP)的字典查找都建構在 **Trie / DAT(雙陣列 Trie)/ [Aho-Corasick](./G2-algorithms.md#aho-corasick)** 之上——Trie 主場詳見 [G1 Trie(字首樹)](./G1-data-structures.md#trie),AC 自動機詳見 [G2 Aho-Corasick](./G2-algorithms.md#aho-corasick)。

<a id="ik-analyzer"></a>
### IK Analyzer 🟡

**定義**:**Elasticsearch 中文切詞 plugin 的事實標準**。原為 Lucene 中文分詞器,後出 Elasticsearch / OpenSearch 版。

**兩種模式**:

| 模式 | 說明 | 範例(中華人民共和國) |
| --- | --- | --- |
| **`ik_smart`** | 智能切分,**粗粒度** | `[中華人民共和國]` |
| **`ik_max_word`** | 最細粒度切分,**所有可能組合** | `[中華人民共和國, 中華人民, 中華, 華人, 人民共和國, 人民, 共和國, 共和, 國]` |

**怎麼選**:
- `ik_max_word` 用於 **index**(建索引時細切,提高召回率)
- `ik_smart` 用於 **search**(查詢時粗切,提高準確率)

**Elasticsearch 範例**:
```json
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

**自訂詞典**:支援 `IKAnalyzer.cfg.xml` 加入專業詞、產品名、人名,或熱更新詞典(從遠端 URL 拉)。

---

<a id="jieba"></a>
### jieba-analysis 🟡

**定義**:**結巴中文分詞**,Python 的 jieba 是中文 NLP 圈最知名的切詞工具,**jieba-analysis 是它的 Java port**(huaban/jieba-analysis)。

**三種模式**:
- **精確模式**(預設):最精準切詞
- **全模式**:把所有可能切出的詞都列出
- **搜尋引擎模式**:精確基礎上對長詞再切

**Java 範例**:
```java
JiebaSegmenter segmenter = new JiebaSegmenter();
List<SegToken> tokens = segmenter.process("結巴中文分詞", SegMode.SEARCH);
// [結巴, 中文, 分詞, 結巴中文分詞]
```

**對比 IK**:
| | IK Analyzer | jieba-analysis |
| --- | --- | --- |
| Elasticsearch 整合 | ✅ 原生 plugin | 🟡 需自寫 plugin 或在應用層處理 |
| 切詞品質 | 良好 | **公認略勝**(尤其新詞識別) |
| 詞典更新活躍度 | 中 | 中(Java port 不如 Python 版活躍) |
| 對短句 / 口語效果 | 好 | **更好** |

**何時選 jieba**:Elasticsearch 之外的場景(後端應用層直接切詞、爬蟲處理、文字分析),或對切詞品質很挑剔。

---

<a id="hanlp"></a>
### HanLP 🟡

**定義**:**功能最完整**的 Java 中文 NLP 框架(何晗,大連理工大學起源),不只切詞——還有**詞性標註、命名實體識別(NER)、依存句法、語義相似度、文字分類、摘要、翻譯**等。

**特性**:
- ✅ 中文 NLP 一站式
- ✅ 詞典與模型完整
- ✅ 有 Elasticsearch plugin(`hanlp-analyzer`)
- ❌ **較重**(完整版下載大、初始化慢)
- ❌ 對單純切詞而言過度設計

**何時選 HanLP**:不只切詞,還需要 NER(抓人名 / 地名 / 公司名)、句法分析、語義分析等深度 NLP 功能。

**選擇建議**:
- 純 Elasticsearch 中文搜尋 → **IK**
- 應用層切詞、品質要求高 → **jieba-analysis**
- 完整中文 NLP → **HanLP**

---

## 相關性計分

「使用者打 `蘋果手機`,文件庫返回 100 筆,**哪一筆排第一**?」這就是相關性計分。

<a id="tf-idf"></a>
### TF-IDF 🟡

**定義**:**Term Frequency × Inverse Document Frequency**——衡量「**這個詞對這份文件有多重要**」。

- **TF**(詞頻):該詞在這份文件中出現幾次?越多越重要
- **IDF**(逆文件頻率):該詞在**整個文件庫**中越**罕見**,越能辨識文件——「的」「是」每篇都有,IDF 低、貢獻小;「量子糾纏」少見,IDF 高、貢獻大
- **TF-IDF = TF × IDF**

**為什麼用**:能自動降低常用詞權重(stop words 之外更精細)、突出特徵詞。

**現況**:Lucene **2.0 之前的預設算法**,**已被 BM25 取代**。但仍是文字向量化(`TfidfVectorizer`)、文件分類常用基礎。

---

<a id="bm25"></a>
### BM25 🟡

**定義**:**Best Matching 25**——TF-IDF 的改良版,**Lucene / Elasticsearch 從 5.0 起的預設相關性算法**。

**比 TF-IDF 強在**:
1. **TF 飽和**:詞出現 100 次不該比 10 次重要 10 倍——BM25 引入飽和函數,讓 TF 上限化
2. **文件長度標準化**:長文件會「順便」匹配很多詞,BM25 對長文件懲罰
3. **可調參數** `k1`(控制 TF 飽和)與 `b`(控制長度懲罰)

**Elasticsearch 內**:
```json
"settings": {
  "similarity": {
    "my_bm25": {
      "type": "BM25",
      "k1": 1.2,    // 預設
      "b": 0.75     // 預設
    }
  }
}
```

**何時調整**:商品名、人名等短文件用較低 `b`(0.3~0.5),減少長度懲罰;部落格、長文章用預設或更高。

---

<a id="cosine"></a>
### Cosine Similarity 🟡

**定義**:衡量**兩個向量的夾角餘弦**,值域 `[-1, 1]`,**1 = 完全相同方向、0 = 正交、-1 = 完全相反**。

**為什麼用於文字**:
- 把文件 / 句子表達為向量(TF-IDF 向量、Word Embedding 向量、BERT 向量)
- 兩段文字「相關性」≈ 兩個向量的餘弦相似度

**公式**:
```
cos(A, B) = (A · B) / (|A| × |B|)
        = Σ(Ai × Bi) / sqrt(ΣAi²) × sqrt(ΣBi²)
```

**Java 計算**:用 nd4j、Apache Commons Math、或自己寫 for 迴圈(維度小)。

**對比 Euclidean Distance**:歐氏距離受向量「**長度**」影響,Cosine **只看方向**——對 embedding 場景**通常 Cosine 更合適**(語意相似但 norm 不同的詞向量)。

---

## 向量化(Embedding)

「電腦看不懂中文」——把文字轉成**數字向量**才能算數學。Embedding = 把詞 / 句子 / 文件「**表達為向量**」的技術。

<a id="word-embedding"></a>
### Word Embedding 🔴

**定義**:把每個**詞**對應到一個**dense vector**(通常 100~1000 維),且**語意相似的詞向量距離近**。

**經典性質**:
```
vec("國王") - vec("男人") + vec("女人") ≈ vec("女王")
```
向量空間中存在語意關係,可做加減法。

**對比 One-hot**:傳統 one-hot 編碼維度等於詞表大小(萬、十萬),且**任兩個詞之間距離都一樣**(沒有語意);Word Embedding 維度小(幾百)、有語意結構。

**主流方法**:Word2Vec、GloVe、FastText、BERT(下面)。

---

<a id="word2vec"></a>
### Word2Vec 🔴

**定義**:Mikolov(Google)2013 提出,**第一個普及的 Word Embedding 方法**。

**兩種訓練架構**:
- **CBOW**(Continuous Bag-of-Words):用上下文預測中間詞
- **Skip-gram**:用中間詞預測上下文(對少見詞效果好)

**Java 工具**:**DeepLearning4J(DL4J)** 內建 Word2Vec 訓練 API:
```java
Word2Vec vec = new Word2Vec.Builder()
    .minWordFrequency(5)
    .layerSize(100)
    .iterate(new BasicLineIterator("corpus.txt"))
    .tokenizerFactory(new DefaultTokenizerFactory())
    .build();
vec.fit();
```

**用途**:訓練自家領域(法律、醫療、電商)詞向量,做相關詞推薦、文字分類、搜尋擴展。

**現況**:**已被 BERT / Sentence-BERT 取代**(更精準,有上下文),但 Word2Vec 仍因**輕量、可解釋**在許多生產系統中存活。

---

<a id="bert"></a>
### BERT Embeddings / Sentence-BERT 🔴

**BERT**(Bidirectional Encoder Representations from Transformers,Google 2018):基於 Transformer 的**上下文相關**語言模型——**同一個詞在不同句子有不同 embedding**。

**對比 Word2Vec**:
- Word2Vec:`apple` 永遠是同一個向量
- BERT:`I ate an apple` 的 `apple` ≠ `Apple released iPhone` 的 `apple`(BERT 區分得出來)

**Sentence-BERT**(SBERT):BERT 變體,專門產生**整句向量**(BERT 原本對單句語意 embedding 不直接好用)。**搜尋向量化主流選擇**。

**中文 BERT 模型**:
- `bert-base-chinese`(Google 官方中文)
- `hfl/chinese-bert-wwm`(全詞遮蔽,效果更好)
- `BAAI/bge-large-zh`(智源,中文 embedding 排行榜頂端)

**Java 工具**:
- **DJL**(Deep Java Library, AWS 開源)——載入 PyTorch / TF / ONNX 模型,Java 推論
- **ONNX Runtime for Java**——把 BERT 轉 ONNX 後在 Java 推論
- 多數團隊用 **Python 服務 + REST/gRPC**:Java 後端把文字送 Python 服務,Python 回 embedding,Java 拿向量做後續

---

<a id="nd4j"></a>
### nd4j 🔴

**定義**:**N-Dimensional Arrays for Java**——Java 的張量(tensor)/ 矩陣運算函式庫,概念類似 Python NumPy。**DeepLearning4J(DL4J)的底層**。

**主要應用**:
- **深度學習**(主用途):DL4J 訓練 / 推論神經網路的數學引擎
- **向量相似度計算**(你的場景):算 cosine similarity、批次點積、向量正規化
- **NLP**:詞向量、句向量的批次運算(數百萬向量計算 cosine 相似度時必須有效率工具)
- **金融建模、科學計算**:蒙地卡羅、線性代數

**搜尋場景的具體用法**:
```java
// 假設已有 query 向量與文件向量集
INDArray queryVector = Nd4j.create(new float[]{0.1f, 0.5f, 0.3f, ...});  // [768]
INDArray docMatrix = Nd4j.create(...);  // [N, 768] N 篇文件

// 批次算 cosine similarity(一行,GPU 加速)
INDArray normalizedQuery = queryVector.div(queryVector.norm2Number().floatValue());
INDArray normalizedDocs = docMatrix.divColumnVector(docMatrix.norm2(1));
INDArray similarities = normalizedDocs.mmul(normalizedQuery);  // [N]
```

**為什麼不直接 for 迴圈**:
- nd4j 底層用 SIMD / OpenBLAS / CUDA(GPU)— 比純 Java 快 10~100 倍
- 處理百萬級向量時差距巨大

**現況**:Java 生態中**做大量數值運算的事實標準**。但若團隊有 Python 服務,通常 embedding 計算放 Python(NumPy / PyTorch),Java 端只接結果——這時 nd4j 就用不上。

---

## 向量搜尋

<a id="knn-ann"></a>
### kNN / ANN / Dense Vector Search 🟡

**kNN**(k-Nearest Neighbors):給一個 query 向量,找出**距離最近的 K 個向量**。是向量搜尋的核心問題。

**精確 kNN 的問題**:每次查詢要與**所有向量**計算距離,百萬級資料下慢得無法接受。

**ANN**(Approximate Nearest Neighbors):**犧牲一點精準度換取速度**——99% 找到正確的 top-K,但快 100~1000 倍。

**主流 ANN 演算法**:
- **HNSW**(Hierarchical Navigable Small World):**目前最普及**,Elasticsearch / Milvus / Qdrant 都用
- **IVF**(Inverted File Index):FAISS 主推
- **PQ**(Product Quantization):壓縮向量,節省記憶體
- **LSH**(Locality-Sensitive Hashing):較古典,效果普通

**Elasticsearch 8+ 範例**:
```json
PUT /products
{
  "mappings": {
    "properties": {
      "embedding": {
        "type": "dense_vector",
        "dims": 768,
        "index": true,
        "similarity": "cosine"
      }
    }
  }
}

GET /products/_search
{
  "knn": {
    "field": "embedding",
    "query_vector": [0.1, 0.5, 0.3, ...],
    "k": 10,
    "num_candidates": 100
  }
}
```

**Hybrid Search**:**BM25(關鍵字)+ 向量搜尋(語意)合併分數**——現代搜尋最佳實踐,Elasticsearch / OpenSearch 已內建支援。

---

<a id="vector-db"></a>
### 向量資料庫 🟡

**定義**:專門儲存與查詢**高維向量**的資料庫,**核心能力是 ANN 搜尋**。

| 產品 | 特色 |
| --- | --- |
| **Milvus** | 開源 + 雲端服務(Zilliz),功能最完整,**中文社群活躍** |
| **Pinecone** | **完全 SaaS**,易用、規模化好,商業 |
| **Qdrant** | 開源 Rust 寫的,**輕量高效** |
| **Weaviate** | 開源,內建模組整合(自動向量化) |
| **Chroma** | 輕量、Python 友善,LLM RAG 場景常用 |
| **Elasticsearch / OpenSearch** | **加上向量搜尋的全文檢索引擎**,適合「既有 ES 想加向量」場景 |
| **PostgreSQL + pgvector** | DB 直接內建,小規模場景實用 |

**何時專用向量 DB,何時用 Elasticsearch**:
- 純向量、極高維、極高量(數十億級向量)→ 專用向量 DB
- 已有 Elasticsearch 全文搜尋,要加語意搜尋 → ES dense_vector 即可
- LLM RAG(Retrieval-Augmented Generation)→ Pinecone / Chroma / Qdrant 主流

---

## Elasticsearch 進階

<a id="analyzer"></a>
### Analyzer / Tokenizer 🟡

**Analyzer 三段式結構**:

```
原始文字 → Character Filter → Tokenizer → Token Filter → terms(進倒排索引)
```

| 階段 | 做什麼 | 範例 |
| --- | --- | --- |
| **Character Filter** | 字元級處理 | HTML 移除、字元映射(`&` → ` and `) |
| **Tokenizer** | 切詞(**核心**) | `whitespace` / `standard` / `ik_max_word` / `keyword` |
| **Token Filter** | Token 級處理 | 轉小寫(`lowercase`)、停用詞(`stop`)、同義詞(`synonym`)、stemming |

**自訂 Analyzer 範例**(中文 + 同義詞 + 停用詞):
```json
"settings": {
  "analysis": {
    "analyzer": {
      "my_chinese_analyzer": {
        "type": "custom",
        "tokenizer": "ik_max_word",
        "filter": ["lowercase", "my_stop", "my_synonym"]
      }
    },
    "filter": {
      "my_stop": {
        "type": "stop",
        "stopwords": ["的", "了", "和", "是"]
      },
      "my_synonym": {
        "type": "synonym",
        "synonyms": ["手機,行動電話", "電腦,計算機,PC"]
      }
    }
  }
}
```

**測試 Analyzer**:
```json
GET /_analyze
{
  "analyzer": "my_chinese_analyzer",
  "text": "我想買一台新的蘋果手機"
}
```

---

<a id="custom-scoring"></a>
### Custom Scoring 🔴

**為什麼自訂計分**:預設 BM25 純看關鍵字匹配,但業務常需要結合**其他因子**:

- 商品**銷量**、**評分**(賣得好的應該排前面)
- 文章**新鮮度**(新文加分)
- 使用者**個人化**(他常看的類別加分)
- **地理位置**(距離近的商家排前面)

**Elasticsearch 三種自訂計分**:

#### `function_score` query
最直觀,把 BM25 分數**乘上 / 加上**自訂函數:
```json
{
  "function_score": {
    "query": { "match": { "title": "蘋果手機" } },
    "functions": [
      { "field_value_factor": { "field": "sales", "modifier": "log1p" } },
      { "gauss": { "created_at": { "scale": "30d", "decay": 0.5 } } }
    ],
    "score_mode": "multiply",
    "boost_mode": "multiply"
  }
}
```

#### `script_score` query
直接寫 Painless script:
```json
{
  "script_score": {
    "query": { "match_all": {} },
    "script": {
      "source": "cosineSimilarity(params.query_vector, 'embedding') + 1.0",
      "params": { "query_vector": [0.1, 0.5, ...] }
    }
  }
}
```
**注意**:script_score 慢,大資料集不適用,通常配 `rescore`。

#### `rescore`
**先用 BM25 找 top N,再對 N 筆做精細計分**——平衡速度與品質。
```json
"query": { "match": { "title": "蘋果" } },
"rescore": {
  "window_size": 100,
  "query": {
    "rescore_query": { "script_score": { ... } },
    "query_weight": 0.7,
    "rescore_query_weight": 1.3
  }
}
```

**何時用哪個**:
- 簡單加權(銷量、新鮮度)→ `function_score`
- 向量相似度 / 複雜公式 → `script_score` + `rescore`(避免全索引算)
- ML 排序 → 學習排序(LTR plugin)

---

## 企業資料平台

<a id="microsoft-fabric"></a>
### Microsoft Fabric 🔴

**定義**:Microsoft 在 2023 年推出的**雲原生資料分析整合平台**,**把 Power BI、Azure Synapse、Azure Data Factory、Azure Data Explorer 等多個分散產品整合進單一 SaaS**——「**All-in-One Data Platform**」定位,對標 Databricks、Snowflake、GCP BigQuery + Looker。

**為什麼出現**:過去 Microsoft Azure 上做資料分析要拼裝多個服務:
- 資料攝取:**Azure Data Factory**
- 資料倉儲:**Azure Synapse Analytics**
- 即時分析:**Azure Data Explorer**(Kusto)
- 視覺化:**Power BI**
- ML:**Azure Machine Learning**
- Lakehouse:第三方 Databricks

**Fabric 把這些塞進一個介面、一個權限、一套儲存**——對企業用戶降低整合成本與學習負擔。

**核心架構:OneLake**(Fabric 的關鍵差異化):
- **「資料的 OneDrive」** — 整個 Fabric 共用一個邏輯資料湖,所有工作負載讀寫同一份資料
- 底層儲存 **Delta Lake / Parquet** 格式(開源)
- 跨 Workspace、跨團隊**不必再複製資料**(對比傳統:每個系統一份 copy)
- 支援 **Shortcut**:外部資料來源(S3、ADLS Gen2、Google Storage)以「捷徑」形式接入,**不搬資料**

**Fabric 的主要工作負載**(Workloads):

| Workload | 對應傳統產品 | 用途 |
| --- | --- | --- |
| **Data Factory** | Azure Data Factory | **資料攝取 / ETL**(無 / 低 code pipeline) |
| **Data Engineering** | Synapse Spark | **Spark notebook**、Lakehouse |
| **Data Warehouse** | Synapse SQL | **T-SQL 企業資料倉儲**(可寫可讀) |
| **Real-Time Analytics** | Azure Data Explorer | **KQL 即時分析**(IoT、log、event stream) |
| **Data Science** | Azure ML | **MLflow notebook**、模型訓練與部署 |
| **Power BI** | Power BI | **視覺化 / 儀表板** |
| **Data Activator** | (新) | 規則觸發告警 / 行動(類似 Reverse ETL) |

**Lakehouse vs Warehouse**(Fabric 內兩種模式):

| | Lakehouse | Warehouse |
| --- | --- | --- |
| 引擎 | **Spark + SQL endpoint(唯讀)** | **T-SQL 完整(讀寫)** |
| 適合 | 半 / 非結構化、ML、批次 | BI、結構化、低延遲查詢 |
| Schema | Schema-on-read 靈活 | Schema-on-write 嚴格 |
| 儲存 | Delta Lake(在 OneLake) | Parquet + Delta(在 OneLake) |

**對照其他資料平台**:

| 平台 | 廠商 | 定位特色 |
| --- | --- | --- |
| **Microsoft Fabric** | Microsoft | **All-in-One SaaS**,整合 Power BI、Synapse、ADF;**OneLake 統一儲存** |
| **Databricks** | Databricks | **Lakehouse 始祖**,Spark 強、Delta Lake 母公司、ML 強;跨雲(AWS / Azure / GCP) |
| **Snowflake** | Snowflake | **Cloud Data Warehouse** 老牌,SQL 體驗極佳,儲存 / 運算分離;跨雲 |
| **GCP BigQuery** | Google Cloud | Serverless 資料倉儲,SQL 體驗好,GA4 預設整合 |
| **AWS Redshift / Athena / Glue** | AWS | 拼裝式;**Lake Formation** 是其 OneLake 對應 |
| **Cloudera Data Platform** | Cloudera | 內部部署 Hadoop / Spark 老牌 |

**Java 工程師會遇到的場景**:
- **後端應用是 OLTP**(交易資料庫),**Fabric 是 OLAP**(分析端)
- 把資料從應用 DB 同步到 Fabric 的方式:
    - **CDC**(Change Data Capture):Debezium → Event Hubs / Kafka → Fabric Data Factory
    - **批次匯出**:Spring Batch / Airflow → 寫 Parquet 到 OneLake
    - **Direct Query**:Power BI 直連 PostgreSQL / MySQL(資料量小可行)
- **與 [F2 AI Agent](./F2-ai-agent.md) 的連結**:Fabric 的 **AI Skills / Copilot** 內建 LLM-on-data,可對企業資料庫做自然語言查詢——後端應用日後可能要提供 **語意層**(semantic layer)給這類 AI agent 使用

**搭配 Microsoft 生態**:
- 與 **Azure AD / Entra ID**([D1 Active Directory](./D1-security-jwt.md#ad))整合做存取控管
- 與 **Azure DevOps**、**GitHub** 整合做 CI/CD
- 與 **Power Platform**(Power Apps / Power Automate)無縫接

**現況與成熟度**(2024-2026):
- **GA**(2024)後快速演進,但**部分功能仍在 Preview**
- 大型企業若已重度使用 Microsoft 365 + Power BI,**轉 Fabric 阻力最低**
- **計價以 Capacity Unit(CU)為單位**(F2 ~ F2048),共享算力給所有工作負載——理解 CU 規模是成本控管關鍵
- 對工程文化偏 Open Source / Spark 的團隊,**Databricks 仍是強競爭者**

**何時選 Fabric**:
- ✅ 已大量使用 Power BI 與 Azure 生態
- ✅ 希望單一平台、單一帳單、單一權限管理
- ✅ 分析需求廣(BI + Spark + 即時 + ML)
- ❌ 需要跨雲 / Open Source 偏好 → Databricks
- ❌ 純 SQL 資料倉儲、極致查詢效能 → Snowflake / BigQuery

---

← [返回索引(README.md)](./README.md)
