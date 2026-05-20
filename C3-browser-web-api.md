# C3 - 瀏覽器 / Web API

← [返回索引(README.md)](./README.md)

---

## 目錄

### HTTP 客戶端
- [XHR(XMLHttpRequest)🟡](#xhr)
- [Fetch API 🟢](#fetch)
- [Axios 🟢](#axios)
- [HTTP 客戶端對照(XHR / Fetch / Axios)🟡](#http-client-compare)

### 瀏覽器儲存
- [Cookies 🟡](#cookies)
- [Local Storage 🟢](#local-storage)
- [Session Storage 🟢](#session-storage)
- [三者對照(Cookies / Local / Session)🟡](#storage-compare)

### 二進位資料
- [Blob 🟡](#blob)

### 即時通訊
- [WebSocket(瀏覽器視角)🟡](#websocket)

---

## HTTP 客戶端

<a id="xhr"></a>
### XHR(XMLHttpRequest)🟡

**定義**:1999 年 Microsoft 在 IE 5 推出的 API,讓 JS 在不重新載入頁面的前提下發出 HTTP 請求——**AJAX 一詞的「X」就是 XHR**(`A`synchronous `J`avaScript `A`nd `X`HR)。

**為什麼用**:
- **歷史地位**:現代 Web 的奠基技術,所有 SPA、所有「不刷頁面就更新內容」的功能都從這裡開始
- **現代多被 fetch 取代**,但**舊系統 / 舊瀏覽器相容**仍會見到
- 上傳進度事件(`onprogress`)早期只有 XHR 有

**範例**:
```javascript
const xhr = new XMLHttpRequest();
xhr.open("GET", "/api/users/123");
xhr.onload = () => {
    if (xhr.status === 200) {
        const data = JSON.parse(xhr.responseText);
        console.log(data);
    }
};
xhr.onerror = () => console.error("Network error");
xhr.send();
```

**為什麼少用**:
- API 是 callback / event-based,**不是 Promise**(難跟 async/await 整合)
- 程式碼冗長(每次都要 `open` / `onload` / `send`)
- 沒有取消請求的標準方式(只有 `xhr.abort()`)
- fetch 出來後,新專案幾乎不會用 XHR

---

<a id="fetch"></a>
### Fetch API 🟢

**定義**:2015 年標準化的瀏覽器原生 HTTP 客戶端,**Promise-based**——取代 XHR 成為現代標準。

**為什麼用**:
- 瀏覽器原生(無需任何套件)
- Promise / async-await 整合自然
- 語法簡潔
- Node.js 18+ 也內建(全環境通用)

**範例**:
```javascript
// 基本 GET
const res = await fetch("/api/users/123");
const user = await res.json();

// POST + JSON
const res = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name: "Alice" })
});

// 取消請求
const ctrl = new AbortController();
fetch("/api/slow", { signal: ctrl.signal });
setTimeout(() => ctrl.abort(), 5000);   // 5 秒後取消
```

**容易踩雷的「fetch 不會自動做」**:
- ⚠ **HTTP 4xx / 5xx 不會 reject**——`res.ok` 才能判斷成功
- ⚠ **不自動帶 cookie**(同源也不會)——要 `credentials: 'include'`
- ⚠ **沒預設超時**——要自己用 `AbortController` + `setTimeout`
- ⚠ **不自動轉 JSON**——要 `await res.json()`

```javascript
// ❌ 常見 bug:不會偵測到 500
const res = await fetch("/api/foo");
const data = await res.json();   // Server 回 500 也會走到這裡

// ✅ 正確處理
const res = await fetch("/api/foo");
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();
```

---

<a id="axios"></a>
### Axios 🟢

**定義**:第三方 HTTP 客戶端套件,Promise-based,**主流選擇**(npm 月下載 ~5000 萬)。對比 fetch,**功能更豐富、開箱即用**。

**為什麼用**:
- **攔截器**(`interceptors.request` / `interceptors.response`)— 統一加 token、處理 401、log
- **HTTP 4xx / 5xx 自動 reject**(不用手動判斷)
- **自動 JSON 轉換**(send 自動 stringify、receive 自動 parse)
- **設定 baseURL / timeout**(fetch 沒有)
- **Node.js + Browser 通用 API**

**範例**:
```javascript
import axios from "axios";

// 全域設定
const api = axios.create({
    baseURL: "https://api.example.com",
    timeout: 5000,
    headers: { "Content-Type": "application/json" }
});

// 攔截器:統一加 Authorization header
api.interceptors.request.use(config => {
    config.headers.Authorization = `Bearer ${getToken()}`;
    return config;
});

// 攔截器:401 統一處理(跳登入頁)
api.interceptors.response.use(
    res => res,
    err => {
        if (err.response?.status === 401) location.href = "/login";
        return Promise.reject(err);
    }
);

// 使用
const { data } = await api.get("/users/123");
const { data: created } = await api.post("/users", { name: "Alice" });
```

**體積取捨**:axios 約 11KB(min+gzip),fetch 是 0KB(原生)——對 bundle size 敏感的場景值得考慮。

> 📌 **個人實戰偏好**:**目前主用**。

---

<a id="http-client-compare"></a>
### HTTP 客戶端對照(XHR / Fetch / Axios)🟡

**完整對照表**:

| 維度 | XHR | Fetch | Axios |
|---|---|---|---|
| 出生 | 1999 | 2015 | 2014 |
| API 風格 | Callback / Event | Promise | Promise |
| async/await | ❌ 需 Promisify | ✅ | ✅ |
| 攔截器 | ❌ | ❌(要自己包) | ✅ 內建 |
| 4xx/5xx 自動 reject | ❌ | ❌(`res.ok`) | ✅ |
| 自動 JSON | ❌ | ❌(`res.json()`) | ✅ |
| 預設帶 cookie | ✅ 同源 | ❌(需 `credentials`) | ✅ 同源 |
| 取消請求 | `abort()` | `AbortController` | `AbortController` / cancel token |
| 超時 | ❌ 需手寫 | ❌ 需 AbortController | ✅ `timeout` 設定 |
| 上傳進度 | ✅(`onprogress`) | ✅(較晚才有) | ✅ |
| 瀏覽器相容 | 全部 | 現代瀏覽器 | 全部(底層用 XHR) |
| Node.js | ❌ | ✅ Node 18+ | ✅ |
| 額外體積 | 0 | 0 | ~11KB(gzip) |
| 維護新專案推薦 | ❌(只在維護舊系統)| ✅ 體積敏感 | ✅ 功能優先 |

**選型建議**:
- **新專案 / 體積敏感** → fetch
- **新專案 / 功能優先(攔截器、自動 JSON、便利設定)** → axios
- **維護舊系統 / IE 相容** → XHR

---

## 瀏覽器儲存

<a id="cookies"></a>
### Cookies 🟡

**定義**:HTTP 規範定義的「**Server 在瀏覽器存的小段資料**」(< 4KB)——Server 透過 `Set-Cookie` header 設定,瀏覽器**自動**在後續同 origin 請求帶上 `Cookie` header。

**為什麼用**:
- **Session 管理**(傳統有狀態驗證)
- **使用者偏好**(語系、主題)
- **追蹤 / 分析**(GA、廣告)

**安全屬性**:

| 屬性 | 意思 | 為什麼重要 |
|---|---|---|
| `HttpOnly` | JS **無法**讀取(`document.cookie` 看不到) | 防 XSS 偷 session |
| `Secure` | 只在 HTTPS 上傳送 | 防中間人攔截 |
| `SameSite=Strict` | 跨站請求**完全不**帶 | 強防 CSRF |
| `SameSite=Lax` | 跨站 GET 帶,POST 不帶(現代瀏覽器**預設**) | 平衡安全與可用性 |
| `SameSite=None` | 跨站都帶(必須搭 `Secure`) | 跨域 SSO 必要 |
| `Max-Age` / `Expires` | 過期時間 | 不設則為 session cookie(關瀏覽器消失) |

**範例**:
```http
HTTP/1.1 200 OK
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
Set-Cookie: locale=zh-TW; Max-Age=2592000
```

**JS 操作 cookie 的限制**:
```javascript
document.cookie = "name=value; max-age=3600; path=/";   // 寫入(有 HttpOnly 的讀不到)
document.cookie;   // 讀全部(無法只讀某一個,要自己 parse)
```

**與 CSRF 的關聯**:cookie 自動帶上是雙面刃——惡意網站誘發瀏覽器發請求,**會自動帶你的 cookie**(這就是 CSRF)。詳見 [D2 CSRF](./D2-web-security.md#csrf)。

**Cookie 法規**:GDPR / ePrivacy 要求「非必要 cookie」需使用者同意(就是那些煩人的彈窗)。

---

<a id="local-storage"></a>
### Local Storage 🟢

**定義**:瀏覽器提供的 key-value 儲存(同 origin),容量 ~5–10MB,**永久性**(除非主動清除或使用者清快取)。同步 API、只能存字串。

**為什麼用**:
- 不需重複下載的資料(使用者偏好、編輯草稿、已讀清單)
- 比 cookie 大很多,且**不會自動帶到 Server**(省頻寬)

**範例**:
```javascript
// 寫入
localStorage.setItem("user", JSON.stringify({ name: "Alice", id: 123 }));

// 讀取
const user = JSON.parse(localStorage.getItem("user") ?? "null");

// 刪除
localStorage.removeItem("user");
localStorage.clear();   // 清全部

// 監聽其他分頁的變動
window.addEventListener("storage", e => {
    console.log(`${e.key} 從 ${e.oldValue} 變成 ${e.newValue}`);
});
```

**關鍵注意**:
- 只能存**字串**(物件要 `JSON.stringify` / `JSON.parse`)
- **同步 API**——大資料寫入會卡 UI thread
- **JS 可讀** → **XSS 風險**(JWT / 敏感資料**不**建議存)
- 永久存到使用者主動清,不像 session cookie 會自動消失

---

<a id="session-storage"></a>
### Session Storage 🟢

**定義**:API 與 `localStorage` **完全相同**,但**生命週期是「分頁級」**——關閉分頁就清空。不同分頁的 sessionStorage 互不共享,即使是同一個網站。

**為什麼用**:
- 臨時表單草稿(關了不需保留)
- 多步驟流程的中間狀態(註冊精靈、結帳流程)
- 頁面 reload 後仍要保留的暫時狀態(reload **不會**清,只有「關閉分頁」會清)

**範例**:
```javascript
sessionStorage.setItem("step", "2");
sessionStorage.setItem("formDraft", JSON.stringify(data));

// 關閉分頁 → 全部清空
// 開新分頁 → 全新的 sessionStorage(不共享)
```

**對 SPA 的特殊行為**:SPA 在同一個分頁內切換路由,sessionStorage **不會清**——除非關掉分頁或 navigate 到不同 origin。

---

<a id="storage-compare"></a>
### 三者對照(Cookies / Local / Session)🟡

**完整對照表**:

| 維度 | Cookies | Local Storage | Session Storage |
|---|---|---|---|
| 容量 | ~4KB | ~5–10MB | ~5–10MB |
| 生命週期 | `Max-Age` / 關瀏覽器(session cookie) | **永久**(直到主動清) | **關分頁清空** |
| 自動帶到 Server | ✅(每個同 origin 請求) | ❌ | ❌ |
| JS 可讀 | 預設可,`HttpOnly` 不可 | ✅ | ✅ |
| 跨分頁共享 | ✅(同 origin) | ✅(同 origin) | ❌ |
| API 風格 | `document.cookie`(難用) | `setItem/getItem` | 同 Local |
| 同步/非同步 | 同步 | 同步 | 同步 |
| XSS 風險 | `HttpOnly` 可防 | **高**(JS 全可讀) | **高** |
| CSRF 風險 | **高**(自動帶) | 無(JS 主動讀才用) | 無 |
| 適合存的東西 | session token(`HttpOnly`)、語系 | 使用者偏好、編輯草稿、cache | 表單草稿、流程狀態 |

**JWT 存哪的爭論**:

| 存放位置 | 優點 | 缺點 |
|---|---|---|
| **`HttpOnly` Cookie** | XSS 偷不走 | CSRF 風險(需 SameSite + CSRF token) |
| **Local Storage** | 簡單、跨網域(微前端) | **XSS 偷得走**(中了 XSS = token 全洩) |
| **記憶體變數** | 安全 | reload 就消失 |
| **混合**:Refresh Token 在 HttpOnly Cookie + Access Token 在記憶體 | 兩者長處 | 實作複雜 |

**業界共識(隨爭論演進)**:
- 純 SPA + JWT → 多數仍存 Local Storage(便利優先)
- 安全敏感(銀行、醫療) → `HttpOnly` Cookie + CSRF token
- 現代 OWASP 推薦 → 走 cookie 路線

---

## 二進位資料

<a id="blob"></a>
### Blob 🟡

**定義**:**B**inary **L**arge **OB**ject——瀏覽器代表「**任意二進位資料**」的物件。檔案上傳、圖片下載、PDF 生成都會用到。

**為什麼用**:
- 處理檔案 / 二進位內容的**統一抽象**
- 與 `<input type="file">` 上傳的 `File` 共通(`File extends Blob`)
- 可以建出 URL 讓 `<img>` / `<a>` 引用(`URL.createObjectURL`)

**核心 API**:
```javascript
// 從字串建 Blob
const text = new Blob(["Hello, World!"], { type: "text/plain" });

// 從二進位建
const png = new Blob([uint8Array], { type: "image/png" });

// 取出內容
await text.text();          // "Hello, World!"
await text.arrayBuffer();   // ArrayBuffer
text.size;                  // 13
text.type;                  // "text/plain"
```

**典型場景 — 檔案上傳**:
```javascript
const file = document.querySelector("input[type=file]").files[0];   // file 是 File / Blob

const form = new FormData();
form.append("upload", file);

await fetch("/api/upload", { method: "POST", body: form });
```

**典型場景 — 動態下載**(產生 PDF / CSV 在前端):
```javascript
const csv = new Blob(["name,age\nAlice,30"], { type: "text/csv" });
const url = URL.createObjectURL(csv);

const a = document.createElement("a");
a.href = url;
a.download = "data.csv";
a.click();

URL.revokeObjectURL(url);   // 用完釋放(避免記憶體洩漏)
```

**Blob 家族關係**:

| | 是 | 用途 |
|---|---|---|
| `Blob` | 二進位資料的基礎抽象 | 通用 |
| `File` | `extends Blob` + `name` + `lastModified` | `<input type="file">` 取得 |
| `FormData` | 包裝多個欄位 + 檔案 | 上傳 multipart |
| `ArrayBuffer` | 原生二進位緩衝(更底層) | 大量處理位元組 |
| `Uint8Array` | 視 ArrayBuffer 為 byte 陣列 | 操作位元組 |

**與後端 Java `Blob` 對照**:詳見 [B4 Blob / Lob](./B4-persistence.md#blob-lob)——**都叫 Blob 但是不同層次的東西**:後端是 DB 大型物件型別,前端是瀏覽器二進位抽象。

---

## 即時通訊

<a id="websocket"></a>
### WebSocket(瀏覽器視角)🟡

**主場詳見** [D3 WebSocket](./D3-networking.md#websocket)——含協定握手、frame 結構、子協定(STOMP / MQTT / GraphQL Subscription)、Spring `@MessageMapping` / Quarkus `@ServerEndpoint` 整合、Sticky session 等議題。本節僅補瀏覽器端 API 與生態。

**瀏覽器原生 API**:
```javascript
const ws = new WebSocket("wss://example.com/chat");

ws.onopen = () => ws.send("hello");
ws.onmessage = (e) => console.log(e.data);
ws.onclose = () => console.log("closed");
ws.onerror = (e) => console.error(e);

// 主動關閉
ws.close(1000, "bye");
```

**常用前端 wrapper**:

| 套件 | 用途 | 配套後端 |
| --- | --- | --- |
| **socket.io-client** | 與 socket.io server 配套,自動 fallback Long Polling | Node.js(socket.io) |
| **STOMP.js / @stomp/stompjs** | STOMP 子協定 client | Spring `@MessageMapping` |
| **graphql-ws** | GraphQL Subscription 標準 | Apollo Server / Hasura |
| **mqtt.js** | MQTT over WebSocket | IoT broker(EMQX / Mosquitto) |

**與 [Fetch](#fetch) / [Axios](#axios) 關係**:WebSocket 是**獨立協定**,**不走 HTTP client**——但 handshake 階段是 HTTP,**可以共用 Cookie / Authorization**(同源 + `credentials: 'include'`),所以登入後的身份能無痛延續到 WS 連線。

**踩雷**:
- **不能在握手後改 header**:第一次連線時 header 帶什麼就什麼,中途 token 過期沒法更新——常見對策是過期前主動 `close()` 重連
- **瀏覽器限制 6 個並發**(對應同網域,**和 HTTP/1.1 同公平池**):大量分頁可能撞上限
- **Heroku / 部分 PaaS 預設 30 秒閒置斷線**:需自己定期 send ping 或調整 platform 設定

---

← [返回索引(README.md)](./README.md)
