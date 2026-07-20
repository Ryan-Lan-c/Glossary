# F2 - AI Agent 與 LLM 應用

← [返回索引(README.md)](./README.md)

---

## 為什麼有這一篇?

**LLM 與 AI Agent 是 2024-2026 工程界最大趨勢之一**——從 ChatGPT 引爆到 Agentic AI、Browser Agent、MCP 標準化,影響範圍從 ML 工程師擴散到所有後端工程師(包括 Java)。

本章收錄這個領域的核心詞彙,讓你在會議、PR、跨團隊討論時跟得上,且**從 Java 角度**了解如何整合。

> 此章節為**逐步擴充**設計,新詞遇到再加入。

---

## 目錄

### LLM 基礎概念
- [LLM(Large Language Model)🟢](#llm)
- [Token / Tokenization 🟢](#token)
- [Prompt / Prompt Engineering 🟢](#prompt)
- [Context Window 🟡](#context-window)
- [Temperature / Top-p 🟡](#temperature)
- [主流 LLM 模型 🟢](#models)

### 深度學習函式庫(DL Frameworks for Java)
- [TensorFlow / TensorFlow Java 🟡](#tensorflow)
- [DeepLearning4J(DL4J)🔴](#dl4j)
- [ONNX Runtime for Java 🟡](#onnx)
- [DJL(Deep Java Library)🟡](#djl)

### AI Agent 概念
- [AI Agent 🟡](#ai-agent)
- [Tool Use / Function Calling 🟡](#function-calling)
- [Agentic AI / 自主代理 🟡](#agentic-ai)
- [典型架構:ReAct / Plan-and-Execute 🔴](#agent-architecture)

### 具體 Agent 框架與工具
- [Hermes Agent 🟡](#hermes-agent)
- [Browser Agent(AI 驅動瀏覽器)🟡](#browser-agent)
- [LLM-ready 網頁擷取(Crawl4AI / Firecrawl)🟡](#web-extraction)
- [LangChain / LlamaIndex 🟡](#langchain)
- [LSP(Language Server Protocol)🟡](#language-server-protocol)
- [MCP(Model Context Protocol)🔴](#mcp)
- [ACP(Agent Communication Protocol)🔴](#acp)

### Java 端整合
- [Spring AI 🟡](#spring-ai)
- [LangChain4j / Quarkus LangChain4j 🟡](#langchain4j)

### 進階模式
- [RAG(Retrieval-Augmented Generation)🟡](#rag)
- [微調(Fine-tuning)vs RAG 🔴](#finetune-vs-rag)

---

## LLM 基礎概念

<a id="llm"></a>
### LLM(Large Language Model)🟢

**定義**:**大型語言模型**,基於 Transformer 架構、用海量文字訓練出來的神經網路。能做**生成、對話、翻譯、摘要、程式碼撰寫、推理**等。

**核心能力**:
- 文字生成 / 對話
- 程式碼撰寫(Copilot、Cursor、Claude Code)
- 推理(數學、邏輯)
- 指令遵循(回答問題 / 做任務)
- 工具使用(下面會講)

**Java 工程師為什麼要懂**:
- 越來越多後端 API **整合 LLM**(智能客服、文件問答、自動分類)
- LLM 可能成為**新一層 middleware**(類似當年 ORM 普及)
- 至少要懂如何**呼叫 LLM API、處理流式回應、管理成本**

---

<a id="token"></a>
### Token / Tokenization 🟢

**定義**:LLM 處理的**最小單位**——不是「字元」也不是「單詞」,而是**模型自己學出來的子詞單位**(BPE、WordPiece、SentencePiece 等演算法切出)。

**範例**(英文):
```
"unbelievable" → ["un", "believ", "able"]   # 3 tokens
"Hello, world!" → ["Hello", ",", " world", "!"]   # 4 tokens
```

**中文**:大致 1 漢字 ≈ 1~2 tokens(視 tokenizer)。

**為什麼工程師要懂**:
- **計費單位**:OpenAI / Anthropic 都按 token 計費(input / output 不同價),寫 prompt 要意識成本
- **長度限制**:Context Window 是 token 數,不是字元數
- **效能**:處理 token 數影響回應時間

**估算工具**:OpenAI tiktoken、Anthropic 自家 tokenizer、Hugging Face tokenizers Java 綁定。

---

<a id="prompt"></a>
### Prompt / Prompt Engineering 🟢

**Prompt**:給 LLM 的**輸入文字**,可以是問題、指令、範例、格式要求。

**Prompt Engineering**:**設計 prompt 讓模型輸出想要的結果**——既是技術也是經驗。

**核心技巧**:

| 技巧 | 範例 |
| --- | --- |
| **Zero-shot** | 直接問:「翻譯這段成英文:你好」 |
| **Few-shot** | 給範例:「`good→好`、`bad→壞`、`great→?`」 |
| **Chain-of-Thought (CoT)** | 「請逐步思考」讓模型展開推理 |
| **Role-playing** | 「你是資深 Java 架構師,請評估...」 |
| **Output Format** | 「以 JSON 回應,欄位:`{name, age}`」 |

**System Prompt vs User Prompt**:
- **System**:規則 / 角色(「你是客服機器人,只能回答產品問題」)
- **User**:實際對話內容
- **Assistant**:模型過往回應(多輪對話的上下文)

**規範**:Java 後端整合 LLM 時,**System prompt 應寫進 code 或設定檔**,避免硬編碼字串到處散落。

---

<a id="context-window"></a>
### Context Window 🟡

**定義**:LLM **單次能處理的 token 上限**(input + output 加起來)。

**主流模型**(2026 年參考):

| 模型 | Context Window |
| --- | --- |
| GPT-4 Turbo | 128K |
| Claude 3.5/4 Sonnet | 200K |
| Gemini 1.5 Pro | 1M~2M |
| Llama 3 | 128K |
| Hermes(Nous Research)| 視底層模型(常 32K~128K) |

**為什麼重要**:
- 太短的 Context → 長文件處理需要拆分(chunking)
- 大 Context → 一次塞整本書,但**注意力分散**(中間部分易被忽略,「lost in the middle」現象)
- **計費**:超過某些門檻可能跳到貴的 tier

**RAG 為何存在**:即使 1M context 也存不下整個公司知識庫,**動態檢索**才是規模化解法(見下面 RAG)。

---

<a id="temperature"></a>
### Temperature / Top-p 🟡

控制 LLM 輸出的**隨機性 / 創意度**:

| 參數 | 作用 | 範圍 |
| --- | --- | --- |
| **Temperature** | 越高越隨機 / 創意,越低越保守 / 重複 | 0.0 ~ 2.0(常用 0~1) |
| **Top-p**(nucleus sampling)| 只從累積機率前 p 的 token 中選 | 0.0 ~ 1.0 |

**何時調**:
- **Temperature = 0**:程式碼、SQL、JSON 抽取——要**確定性**(同 input 同 output)
- **Temperature = 0.7~1.0**:創意寫作、發想
- **二擇一**:通常**只調一個**(常用 Temperature),Top-p 較少改

**Java 工程師注意**:**業務邏輯類用 0**,別讓客服回應同個問題每次答案不一樣。

---

<a id="models"></a>
### 主流 LLM 模型 🟢

| 類別 | 模型(廠商) | 特色 |
| --- | --- | --- |
| **閉源 SaaS** | GPT-4 / GPT-4o / o1 / o3(OpenAI) | 通用最強,生態最大 |
| | Claude 3.5/4(Anthropic) | 程式碼、長文、推理強;Computer Use 首發 |
| | Gemini 1.5/2(Google) | 巨大 context、多模態 |
| **開源** | Llama 3.x(Meta) | 開源最廣泛 |
| | **Hermes**(Nous Research) | Llama 微調,**function calling 強** |
| | Mistral / Mixtral | 歐洲開源,效率好 |
| | Qwen(阿里) | 中文 / 多語言強 |
| | DeepSeek | 推理 / 程式碼強 |
| **本地部署** | Ollama / LM Studio / vLLM | 跑開源模型在本機 / 自家機房 |

**選型考量**:
- 資料**敏感度**(送 OpenAI 還是自架)
- **成本**(自架硬體 vs API 費用)
- **能力需求**(複雜推理 → GPT-4 / Claude;簡單分類 → 小模型即可)
- **延遲**(SaaS 100~500ms,本地視 GPU)

---

## 深度學習函式庫(DL Frameworks for Java)

Java 後端整合 ML 模型有兩條路:**(A) 開 Python 微服務**,Java 透過 REST/gRPC 呼叫;**(B) Java 直接跑模型**(本節)。何時選 B:延遲敏感、無法部 Python、想簡化 stack。

<a id="tensorflow"></a>
### TensorFlow / TensorFlow Java 🟡

**TensorFlow**:Google 開源的深度學習框架(2015 至今),曾是 ML 框架霸主,目前與 PyTorch 並列(企業 / 部署偏 TF,研究偏 PyTorch)。

**TensorFlow Java**:官方 Java 綁定,通常**用於推論**(載入訓練好的 SavedModel 跑預測),少用於訓練。

```java
try (SavedModelBundle model = SavedModelBundle.load("/path/to/model", "serve")) {
    Tensor input = TFloat32.tensorOf(...);
    Tensor output = model.session().runner()
        .feed("input", input)
        .fetch("output")
        .run().get(0);
}
```

**注意**:TensorFlow Java 底層走 JNI,**Native binary 與 OS / GPU 綁定**(部署要小心)。**新專案建議改用 ONNX Runtime / DJL**(下面)。

---

<a id="dl4j"></a>
### DeepLearning4J(DL4J)🔴

**定義**:**Java 原生**的深度學習框架(Skymind 開源,後捐給 Eclipse Foundation)——可在 Java 中**訓練 + 推論**神經網路,底層用 **nd4j**(見 15 章)做張量運算。

**特色**:
- ✅ 純 Java 生態,**不需要 Python**
- ✅ 與 Spring / 企業 Java 整合好
- ✅ 支援 GPU(透過 CUDA)
- ❌ 模型生態不如 PyTorch / TensorFlow 豐富
- ❌ 社群活躍度下降(主流仍是 Python)

**何時用**:純 Java 環境、無法接觸 Python、自有訓練流程。

---

<a id="onnx"></a>
### ONNX Runtime for Java 🟡

**ONNX**(Open Neural Network Exchange):**跨框架的模型格式標準**——TensorFlow / PyTorch 訓練的模型可以匯出成 `.onnx`,任何支援 ONNX 的 runtime 都能載入推論。

**為什麼革命性**:
- 訓練用 PyTorch(研究員主場),**部署用任何語言**(Java / C# / C++ / JS)
- **Vendor-neutral**——換訓練框架不用重寫部署 code
- ONNX Runtime 由 **Microsoft 主導**,社群活躍

**Java 整合**:
```java
try (OrtEnvironment env = OrtEnvironment.getEnvironment();
     OrtSession session = env.createSession("/path/to/model.onnx")) {
    OnnxTensor input = OnnxTensor.createTensor(env, inputData);
    Map<String, OnnxTensor> inputs = Map.of("input", input);
    OrtSession.Result result = session.run(inputs);
    // ... 取出 output
}
```

**規範建議**:**模型部署優先選 ONNX Runtime**——比 TensorFlow Java 輕量、跨平台好、效能好。

---

<a id="djl"></a>
### DJL(Deep Java Library)🟡

**定義**:**AWS 開源**的 Java 深度學習統一 API——同一份 Java code,底層可切換 PyTorch / TensorFlow / MXNet / ONNX 引擎。

**特色**:
- ✅ **單一 API**,跑多種模型格式
- ✅ Hugging Face 模型整合(載 BERT、Llama 等)
- ✅ 文件與範例豐富
- ✅ 適合「載 Python 訓練的模型在 Java 推論」場景

**範例**(載 BERT 做 embedding):
```java
Criteria<String, float[]> criteria = Criteria.builder()
    .setTypes(String.class, float[].class)
    .optModelUrls("djl://ai.djl.huggingface.pytorch/sentence-transformers/all-MiniLM-L6-v2")
    .build();

try (ZooModel<String, float[]> model = criteria.loadModel();
     Predictor<String, float[]> predictor = model.newPredictor()) {
    float[] embedding = predictor.predict("Hello world");
}
```

**規範建議**:Java 端做 ML 推論的**現代首選**——比 TensorFlow Java 簡單、比手動載 ONNX 友善。

---

## AI Agent 概念

<a id="ai-agent"></a>
### AI Agent 🟡

**定義**:**能自主決策與執行任務的 AI 系統**,通常結構是 **LLM + Tools + Loop**:
1. LLM 分析任務
2. 決定要用哪個工具(查 DB、發 email、開檔)
3. 看到工具回應後,決定下一步
4. 反覆直到任務完成

**對比 Chatbot**:
- **Chatbot**:被動回應,單輪 / 多輪對話,**不執行外部動作**
- **Agent**:主動執行任務,**呼叫 API、改 DB、操作系統**

**簡化心智模型**:
```
Agent = LLM + Memory + Tools + Loop(Plan → Act → Observe → Plan ...)
```

---

<a id="function-calling"></a>
### Tool Use / Function Calling 🟡

**定義**:LLM **主動呼叫外部工具(函數)**的能力——你給 LLM 一份「工具清單」(JSON schema 描述),它根據對話需要產生「我要呼叫 `get_weather(city='Tokyo')`」的指令,你執行後把結果回給它。

**OpenAI 範例**(簡化):
```json
{
  "tools": [{
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "取得指定城市天氣",
      "parameters": {
        "type": "object",
        "properties": {
          "city": { "type": "string" }
        },
        "required": ["city"]
      }
    }
  }],
  "messages": [{ "role": "user", "content": "東京天氣如何?" }]
}
```

LLM 回:`{"tool_call": {"name": "get_weather", "arguments": {"city": "Tokyo"}}}`
你執行 → 結果回給 LLM → LLM 整合成自然語言回覆。

**Anthropic Tool Use** 與 **OpenAI Function Calling** 概念相同,API 略不同。

**為什麼是 Agent 基石**:沒有 Tool Use,LLM 只能輸出文字;有了 Tool Use,LLM 能**真的做事**——Agent 由此而生。

---

<a id="agentic-ai"></a>
### Agentic AI / 自主代理 🟡

**定義**:強調 AI **自主性**的詞,代表「**能多步驟完成複雜任務**而非單次問答」的 AI 系統。

**Agentic 程度**(光譜):
- 🌱 **Workflow with LLM**:固定流程中嵌入 LLM(分類、摘要)— **最確定性**
- 🌿 **Single-step Agent**:LLM 決定一次工具呼叫
- 🌳 **Multi-step Agent**:LLM 自主迴圈(本節定義的 Agent)
- 🌲 **Multi-agent System**:多個 Agent 協作(planner + executor + critic)

**現況**:**Agentic 是 2025 年的關鍵詞**,但「真正可靠的 multi-step agent」仍在早期——容易迴圈、跑偏、燒 token。生產系統多半從 workflow 開始,逐步增加 agentic 程度。

---

<a id="agent-architecture"></a>
### 典型架構:ReAct / Plan-and-Execute 🔴

**ReAct**(Reasoning + Acting):**最普及的 agent 架構**(2022 提出)——每步驟交替「**思考**」(Thought)與「**行動**」(Action),觀察結果後再思考下一步。

```
Thought: 我需要查使用者的訂單狀態
Action: query_db("SELECT status FROM orders WHERE user_id = 123")
Observation: status = "PAID"
Thought: 已付款,要查物流
Action: call_logistics_api(order_id)
Observation: shipped
Thought: 我可以回覆使用者了
Final Answer: 您的訂單已出貨
```

**Plan-and-Execute**:先做完整計畫(plan),再執行——適合**步驟多但可預期**的任務。

**現代趨勢**:**動態混合**——複雜任務先 plan,執行中遇到意外切回 ReAct。

---

## 具體 Agent 框架與工具

<a id="hermes-agent"></a>
### Hermes Agent 🟡

**定義**:**Nous Research** 開源的 Agent 框架(github.com/nousresearch/hermes-agent),基於他們的 **Hermes 系列 LLM**(Llama 微調),專長 **function calling 與工具使用**。

**Nous Research 是誰**:知名的開源 LLM 微調團隊,Hermes 系列(Hermes 2 / 3 / 4)在 function calling、role-play、reasoning 表現亮眼,屬於開源界一線。

**特色**:
- ✅ 開源,**可本地部署**(資料不外流)
- ✅ Function calling 訓練深入(比通用模型更可靠)
- ✅ 適合**自架 Agent 服務**而不依賴 OpenAI / Anthropic
- 🟡 仍需自己包應用層(prompt、工具、UI)

**台灣 cg.com.tw 的 HermesAgent**:推測是台灣公司基於同類概念(可能用 Nous Hermes 或自家模型)做的**企業級 Agent 產品**(具體功能待你補充)。

**何時考慮用**:
- 公司資料**不能送雲端 LLM**(金融、政府)
- 需要**深度 function calling** 的 agentic 應用
- 自己有 GPU 機房 / 願意租雲 GPU

---

<a id="browser-agent"></a>
### Browser Agent(AI 驅動瀏覽器)🟡

**定義**:**LLM 控制瀏覽器**完成複雜任務——看畫面截圖、決定點哪、填什麼、按下一步,**像人類一樣操作網頁**。

**主流 Browser Agent**:

| 名稱 | 廠商 | 特色 |
| --- | --- | --- |
| **Computer Use** | Anthropic(2024-10 首發) | 不只瀏覽器,**控制整個桌面**;Claude 3.5+ 內建能力 |
| **Operator** | OpenAI(2025-01) | 雲端瀏覽器代理,SaaS 服務 |
| **Browser Use** | 開源 Python(github.com/browser-use) | 開源,可整合任何 LLM |
| **OmniParser** | Microsoft 開源 | 把 UI 截圖轉成結構化元素描述 |
| **AutoGPT / BabyAGI**(舊) | 早期 agent 嘗試 | 概念先驅,實用度有限 |

**運作流程**(簡化):
1. 截圖網頁
2. LLM 分析「這頁面有什麼可點的」
3. 產出指令:「點 (x=200, y=300) 的『登入』按鈕」
4. Selenium / Playwright 執行
5. 看新畫面 → 重複

**對比 Selenium / Playwright**:
- Selenium / Playwright:**人寫測試腳本**,確定性高
- Browser Agent:**LLM 自主決定**,適合腳本寫不出來的場景(隨機 UI、未知網站)

**Java 端整合**:目前主流仍是 Python 生態(Browser Use、Playwright Python)。Java 工程師**比較常做的事**是**建構後端供 Browser Agent 呼叫的 API**,而非自己跑 Browser Agent。

**注意**:Browser Agent 仍**不穩定且燒 token**,且**安全議題嚴重**(讓 LLM 控制瀏覽器 = 它能登入、轉帳、刪檔)。**沙箱與審計必備**。

---

<a id="web-extraction"></a>
### LLM-ready 網頁擷取(Crawl4AI / Firecrawl)🟡

**定義**:把網頁自動轉成**乾淨、結構化、LLM 可直接吃的 Markdown**,當作 [RAG](#rag) / Agent 的「資料進料端」。解決傳統爬蟲(BeautifulSoup / Scrapy)的兩大痛點——抓回來的是**滿是雜訊的原始 HTML**、且遇到 JavaScript 動態網站會抓到**空殼**。

**為什麼用**:
- 內建**無頭瀏覽器**(headless browser),能執行 JS、等動態內容載入
- 自動過濾導覽列 / 廣告 / script,輸出**正文 Markdown**,省掉 RAG 前最麻煩的清洗步驟
- 正好補上 [RAG](#rag) 流程裡「Document Loader(讀網頁)+ Chunking」這一段

**Crawl4AI vs Firecrawl 對照**:

| 面向 | Crawl4AI | Firecrawl |
| --- | --- | --- |
| 型態 | **開源自架**(Apache 2.0,73k+ ★) | 商用 **SaaS API**(核心開源,主打雲端) |
| 費用 | 免費 | 按用量計費(有免費額度) |
| 底層 | Playwright(Chromium / Firefox / WebKit) | 託管式爬取基礎設施 |
| 特色 | 完整瀏覽器控制、stealth 反偵測、adaptive crawling、**可配本地 LLM(Ollama)零 API key** | 開箱即用、`/scrape`・`/crawl` API 極簡、零維運 |
| 取捨 | 要自架 / 資料不外流 / 要客製 → 選它 | 要快速整合 / 不想維運爬蟲 → 選它 |

**和 [Browser Agent](#browser-agent) 的區隔**(別混淆):
- **Browser Agent** = LLM 自主「**操作**」網頁(點按、填表、完成任務)
- **網頁擷取** = 把網頁「**擷取**」成資料餵給 LLM,本身**不做決策**

**範例(Crawl4AI)**:
```python
import asyncio
from crawl4ai import AsyncWebCrawler

async def main():
    async with AsyncWebCrawler() as crawler:
        result = await crawler.arun(url="https://example.com")
        print(result.markdown)   # 乾淨、可直接餵 LLM 的 Markdown

asyncio.run(main())
```

**Java 端**:兩者皆 Python 生態。Java 工程師常做的是**呼叫其 REST API**(Crawl4AI 可 Docker 自架成 FastAPI 服務、Firecrawl 直接打雲端 API),把回傳的 Markdown 接進 [Spring AI](#spring-ai) / [LangChain4j](#langchain4j) 的 RAG pipeline。

---

<a id="langchain"></a>
### LangChain / LlamaIndex 🟡

**LangChain**:Python / JS 主流 LLM 應用框架——把 prompt、模型、tool、memory、output parsing 等串成鏈(chain)。Agent / RAG 都基於它建構。

**LlamaIndex**:**專注 RAG** 的框架,處理「**外部知識 + LLM**」場景特別好。

**對比**:
- LangChain:**通用**,適合各種 LLM workflow,功能多到爆
- LlamaIndex:**RAG 專門**,文件處理、向量索引、查詢路由更深

**Java 工程師會遇到**:多數 ML / Data Science 同事用這兩個寫 prototype,你可能需要**接他們的 Python 服務**或**遷移到 Java**(用 LangChain4j,下面)。

---

<a id="language-server-protocol"></a>
### LSP(Language Server Protocol)🟡

> ⚠ 別跟 [A1 SOLID 的 LSP(Liskov Substitution Principle,里氏替換)](./A1-code-quality.md#lsp) 搞混——同縮寫、完全不同概念。

**定義**:Microsoft 2016 提出的協定,把「編輯器 / IDE」和「語言智慧」(autocomplete、go-to-definition、找引用、即時診斷、rename)**解耦**。編輯器當 client、語言端當 server(language server),兩者用 **JSON-RPC** 溝通(走 stdio / socket)。

**為什麼重要(N×M → N+M)**:
- 沒 LSP 前:N 個編輯器 × M 種語言 = **N×M** 份語言智慧實作
- 有 LSP 後:一種語言只要寫**一個 language server**,所有支援 LSP 的編輯器都能用 → **N+M**
- 所以今天 VS Code / Neovim / JetBrains 能共用同一個 `rust-analyzer`、`gopls`、`typescript-language-server`

**範例(概念流)**:
```
編輯器(client) --textDocument/completion-->  Language Server
編輯器(client) <--補全清單 / 診斷 / 定義位置--  Language Server
```

**和 MCP 的關係**:[MCP](#mcp) 之於 LLM Tool Use,正如 LSP 之於 IDE——同樣是「**一套標準協定取代 N×M 各自整合**」的思路,MCP 也走 JSON-RPC、也分 client / server。理解 LSP 就懂 MCP 的設計動機。

---

<a id="mcp"></a>
### MCP(Model Context Protocol)🔴

**定義**:**Anthropic 在 2024-11 提出的 LLM 與工具 / 資料源之間的標準化協定**——把 Tool Use 從「每家 LLM 各搞一套」變成「**一套標準,任何 LLM 通用**」。

**類比**:
- **[LSP](#language-server-protocol)**(Language Server Protocol)之於 IDE
- **OpenTelemetry** 之於觀測性
- **MCP** 之於 LLM Tool Use

**架構**:
```
LLM Client(Claude / Cursor / 你的 App)
   ↕ MCP 協定(JSON-RPC over stdio / SSE)
MCP Server(由工具提供方實作:GitHub、Slack、DB、檔案系統 ...)
```

**為什麼重要**:
- 你的工具實作一次 MCP server,**所有支援 MCP 的 LLM client 都能用**
- 解決 Tool Use 碎片化問題(OpenAI / Anthropic / Google 各搞各的)
- **2025 年快速崛起**,各大 IDE / AI 工具陸續支援

**Java 工程師會遇到**:
- 公司想把內部系統(Jira、Confluence、自家 API)接上 LLM → **寫 MCP server**(有官方 Java SDK)
- 用 IDE(Cursor / Claude Desktop)時遇到 `.mcp/` 配置檔

**現況**:**新但快速普及**,不確定會否成為長期標準,但目前看 Anthropic 推動有力 + 社群跟進熱烈。

> → MCP 解決的是「**模型 ↔ 工具**」通訊;Agent ↔ Agent 之間的協作協定見 [ACP](#acp)。

---

<a id="acp"></a>
### ACP(Agent Communication Protocol)🔴

**定義**:**Agent ↔ Agent 之間的通訊協定**——當多個 AI Agent 需要協作(委派任務、共享狀態、轉發訊息)時,定義它們如何「對話」的標準。對應於 [MCP](#mcp)(**模型 ↔ 工具**)的另一個層次。

**為什麼用**:
- 多 Agent 系統(MAS, Multi-Agent System)中,Agent 之間需要**結構化通訊**,不能各說各話
- **互通性**:不同廠商 / 不同框架的 Agent 能對話(例如 Claude Agent 跟 OpenAI Agent 協作)
- **可觀測性**:標準化訊息能被 log / monitor / route

**概念來源**:**FIPA-ACL**(Foundation for Intelligent Physical Agents - Agent Communication Language)是 1990s-2000s 的學術標準,定義了 `inform` / `request` / `propose` / `accept-proposal` 等 **performative**(語用行為)。現代 AI Agent 時代的 ACP 多以**自然語言 + 結構化欄位**為主,沒有單一定論的協定,各家(Microsoft、Google、Anthropic、開源社群)都在演化。

**典型欄位**(綜合各家方案):

| 欄位 | 用途 |
|---|---|
| `from` / `to` | 發送方 / 接收方 Agent ID |
| `performative` | 行為類型(`request` / `inform` / `query` / `propose`) |
| `content` | 訊息主體(自然語言 / JSON) |
| `conversation_id` | 對話 ID,串接多輪訊息 |
| `reply_to` | 對應哪一條訊息的回覆 |
| `language` / `ontology` | 訊息使用的語言 / 領域知識 |

**MCP vs ACP 對照**:

| 維度 | MCP(Model Context Protocol) | ACP(Agent Communication Protocol) |
|---|---|---|
| 通訊雙方 | **模型 ↔ 工具 / 資源** | **Agent ↔ Agent** |
| 主要功能 | 讓 LLM 能呼叫外部工具(tool use)、讀外部資源 | 讓多個 Agent 協作(委派、協商、共享) |
| 標準化程度 | Anthropic 主推、有開放規範 | 多家併存,**無單一主流標準** |
| 典型場景 | Claude 連到本地檔案系統 / GitHub / Slack | Agent A 委派 Agent B 寫 code,Agent C 審 review |
| 概念來源 | 2024 Anthropic 提出 | FIPA-ACL(1990s 學術)+ 現代 LLM Agent 演化 |

**現實狀態(2026)**:
- **MCP** 在 LLM tool use 領域已是事實標準(Anthropic + 多家採用)
- **ACP** 還在早期演化:**A2A**(Agent2Agent,Google 2025 提的)、**Agent Protocol**(LangChain 等社群推的)、**IBM ACP**(IBM 自家版)等多個方案並存
- 目前多 Agent 系統多採用「框架內部協定」(LangGraph、AutoGen、CrewAI 各有),**跨框架互通仍在發展中**

**選型現實**:
- **單一 Agent 系統,需 tool use** → 用 MCP
- **多 Agent 系統,同框架** → 用框架內建協定(LangGraph state、AutoGen ConversableAgent)
- **跨框架 Agent 互通** → 觀望 A2A / Agent Protocol 等標準成熟

---

## Java 端整合

<a id="spring-ai"></a>
### Spring AI 🟡

**定義**:**Spring 官方**的 LLM 整合框架(2024 起,屬 Spring 主項目)——統一不同 LLM 廠商的 API,用 Spring 慣例寫 LLM 應用。

**支援的 provider**:OpenAI、Anthropic、Azure OpenAI、Google Vertex AI、Ollama、Hugging Face、Amazon Bedrock、Mistral 等。

**範例**(基本對話):
```java
@RestController
@RequiredArgsConstructor
public class ChatController {
    private final ChatClient chatClient;

    @PostMapping("/chat")
    public String chat(@RequestBody String userInput) {
        return chatClient.prompt()
            .user(userInput)
            .call()
            .content();
    }
}
```

```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      chat:
        options:
          model: gpt-4o
          temperature: 0.7
```

**功能**:
- Chat Client(同步 / 流式)
- Embedding Client(向量化)
- Function Calling
- RAG 工具(VectorStore 抽象,可接 PgVector / Milvus / Pinecone / Chroma)
- Image 生成
- Spring 1.0 起支援 **MCP**

**規範建議**:Spring Boot 專案**首選 Spring AI**,別自己包 OkHttp 打 OpenAI API。

---

<a id="langchain4j"></a>
### LangChain4j / Quarkus LangChain4j 🟡

**LangChain4j**:LangChain 的 **Java 移植**(社群開源,非官方),API 風格與 Python 對齊,**Quarkus 與 Spring 都能用**。

**特色**:
- ✅ 大量 LLM provider 整合(類似 Spring AI)
- ✅ 內建 RAG 工具、向量資料庫、文件 loader
- ✅ Agent / Tool Use 支援好
- ✅ 與 Quarkus 整合特別深(`quarkus-langchain4j` extension)

**Quarkus 範例**(用 declarative AI service):
```java
@RegisterAiService
public interface CustomerSupportAgent {
    @SystemMessage("你是友善的客服機器人,只回答產品相關問題")
    String chat(@UserMessage String userInput);
}

// 注入後直接用
@Inject CustomerSupportAgent agent;
String reply = agent.chat("你們有提供退貨服務嗎?");
```

**Spring AI vs LangChain4j 怎麼選**:
- **Spring Boot 專案** → **Spring AI**(官方、整合好、未來性強)
- **Quarkus 專案** → **LangChain4j**(Quarkus 官方推、Native 友善)
- 兩者都熟 → 看團隊偏好

---

## 進階模式

<a id="rag"></a>
### RAG(Retrieval-Augmented Generation)🟡

**定義**:**「外部知識 + LLM」的問答模式**——LLM 不知道你公司內部知識,但你可以**先檢索相關文件,再讓 LLM 基於檢索結果回答**。

**流程**:
```
1. 使用者問題 → embedding 模型 → query 向量
2. query 向量 → 向量資料庫(見 15 章) → top-K 相關文件
3. top-K 文件 + 原問題 → LLM
4. LLM 整合資訊 → 回答
```

**為什麼需要**:
- LLM 訓練資料**有截止日期**(2024-04 等),不知道之後的事
- LLM **不知道公司內部資料**(Confluence、產品文件)
- 把所有資料塞 Context 不可能(成本 + lost in the middle)
- RAG = **動態檢索 + 動態生成**,規模化解法

**核心元件**:
- **Embedding 模型**(把文字轉向量)— 見 15 章
- **向量資料庫**(Milvus / Pinecone / Qdrant / Elasticsearch dense_vector / pgvector)— 見 15 章
- **LLM**(本章)
- **Document Loader**(讀 PDF / Word / 網頁)
- **Chunking 策略**(切文件成可索引的片段)

**Java 整合**:Spring AI / LangChain4j 都內建 RAG pipeline,連 Pinecone / Milvus / pgvector 都有 starter。

**規範建議**:RAG 設計時**先問**:文件多大?查詢頻率?準確度要多高?——再決定 chunking 策略、embedding 模型、retrieval 方式(向量 / hybrid / re-rank)。

---

<a id="finetune-vs-rag"></a>
### 微調(Fine-tuning)vs RAG 🔴

**Fine-tuning**:用你的資料**繼續訓練 LLM**,讓它「**內化**」這些知識(權重變了)。
**RAG**:LLM 不變,**查找後塞進 prompt** 讓它臨時看到資訊。

**對照表**:

| 面向 | Fine-tuning | RAG |
| --- | --- | --- |
| **改變什麼** | 模型權重 | 不改模型,只改 prompt |
| **訓練成本** | 高(需 GPU、需資料、需 ML 知識) | 無(不訓練) |
| **更新成本** | 每次更新都要重訓 | **只要更新文件即可**(成本低) |
| **適合學什麼** | 風格、格式、領域語言 | **事實知識**(會變動的、量大的) |
| **可解釋性** | 黑盒(為什麼這樣回?不知道) | **可追溯**(從哪份文件來) |
| **幻覺風險** | 高 | **較低**(有來源,可比對) |

**選擇**:
- 想要 LLM「**用特定語氣 / 格式回答**」 → Fine-tuning
- 想要 LLM「**知道公司資料**」 → **RAG(99% 場景的答案)**
- 兩者**可同時用**:fine-tune 風格 + RAG 注入知識

**規範建議**:**先試 RAG,別輕易 fine-tune**——成本大、維護痛、能用 RAG 解的問題九成不需要 fine-tune。

---

← [返回索引(README.md)](./README.md)
