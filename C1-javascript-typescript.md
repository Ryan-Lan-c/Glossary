# C1 - JavaScript / TypeScript

← [返回索引(README.md)](./README.md)

---

## 目錄

- [JavaScript(JS)🟢](#javascript)
- [TypeScript(TS)🟡](#typescript)
- [Prototype(原型鏈)🟡](#prototype)
- [Deep Clone(深拷貝)🟡](#deep-clone)
- [`eval()`(字串執行,危險)🟡](#eval)
- [Optional Chaining(`?.`)🟢](#optional-chaining)
- [Nullish Coalescing(`??`)🟢](#nullish-coalescing)
- [JSX 🟡](#jsx)

---

<a id="javascript"></a>
### JavaScript(JS)🟢

**定義**:Brendan Eich 1995 年於 Netscape 開發,標準化為 **ECMAScript**(ES)。動態類型、原型導向、單執行緒 + Event Loop 的腳本語言;原本只能在瀏覽器跑,現在 Node.js / Deno / Bun 等 runtime 也能在 Server 端跑。

**為什麼用**:
- **唯一**能在瀏覽器原生執行的語言(WebAssembly 另一回事)
- 全端開發:前端 + Node.js 後端共用語言
- 龐大生態(npm 套件數量遠超其他語言)

**版本演進**:
- ES5(2009):多數舊瀏覽器的最低支援基準
- **ES6 / ES2015**:革命性更新(`let` / `const` / arrow function / class / module / Promise)
- 之後每年一版:`async/await`(ES2017)、`?.` / `??`(ES2020)、`Object.hasOwn`(ES2022)…

**與 Java 命名相似但本質無關**:
- Java:靜態型別、class-based、編譯型、JVM 執行
- JavaScript:動態型別、prototype-based、解釋型、瀏覽器 / Node 執行
- 名字像只是當年的行銷策略

**範例**:
```javascript
// 動態型別
let x = 1;
x = "hello";        // 沒問題

// 函式為一等公民
const add = (a, b) => a + b;
[1, 2, 3].map(add.bind(null, 10));   // [11, 12, 13]

// 單執行緒非同步(Promise / async-await)
async function fetchUser(id) {
    const res = await fetch(`/users/${id}`);
    return await res.json();
}
```

---

<a id="typescript"></a>
### TypeScript(TS)🟡

**定義**:Microsoft 2012 年開源,**JavaScript 的 superset**(JS 是合法 TS,TS 加上靜態型別系統)—— `tsc` 編譯器把 TS 編譯成 JS 後執行。

**為什麼用**:
- **靜態型別**抓出 JS 動態型別容易犯的錯
- 大型專案的可維護性、refactor 安全性
- 編輯器智慧提示更準確
- 漸進式採用(`.js` 與 `.ts` 可混用、`any` 逃生口)

**核心概念**:

| 概念 | 說明 |
|---|---|
| `type` vs `interface` | 多數場景可互換;`interface` 可宣告合併、`type` 可組合(union / intersection) |
| 泛型 `<T>` | 跟 Java 類似 |
| `unknown` vs `any` | `any` 是逃生口(放棄檢查);`unknown` 強迫你 narrow 型別後才能用 |
| `never` | 不可能達到的狀態(switch 的 exhaustive check) |
| `strict` mode | `tsconfig.json` 開啟一系列嚴格檢查(強烈建議開) |

**範例**:
```typescript
type User = { id: number; name: string; email?: string };

interface Repository<T> {
    findById(id: number): T | null;
}

class UserRepo implements Repository<User> {
    findById(id: number): User | null { return null; }
}

// type narrow
function handle(x: unknown) {
    if (typeof x === "string") {
        x.toUpperCase();    // 此時 x 被 narrow 成 string,可以用
    }
}
```

**與 Java 開發者類比**:類似但更靈活——Java 型別檢查在 compile time + runtime;TS 只在 compile time(runtime 仍是 JS,沒型別)。

**範圍**:絕大多數新前端 / Node.js 後端專案都用 TS。

---

<a id="prototype"></a>
### Prototype(原型鏈)🟡

**定義**:JS 的 OO 機制——每個物件都有一個「**隱藏的 prototype 連結**」(`__proto__`),指向「父物件」;當你存取一個屬性,JS 沿著 prototype 鏈往上找,直到 `Object.prototype`(終點)或回 `undefined`。

**為什麼用**:
- 這是 JS **唯一**的繼承機制(ES6 `class` 是語法糖,底層仍是 prototype)
- 理解 prototype 才看得懂 JS 框架的「物件擴充」與「方法共享」原理
- Debug「為什麼這個方法不存在?」要查 prototype 鏈

**核心區分(`__proto__` vs `prototype`)**:
- `obj.__proto__` — **物件**的 prototype 連結(指向「父物件」)
- `Func.prototype` — **建構函式**的 prototype 物件(`new Func()` 出來的物件,其 `__proto__` 會指向這裡)

**範例**:
```javascript
// ES6 class(底層仍是 prototype)
class Animal {
    constructor(name) { this.name = name; }
    eat() { console.log(`${this.name} eats`); }
}
class Dog extends Animal {
    bark() { console.log(`${this.name} barks`); }
}

const d = new Dog("Rex");
d.bark();   // "Rex barks"
d.eat();    // "Rex eats"   ← 沿 prototype 鏈找到 Animal.eat

// 等同的舊寫法(直接操作 prototype)
function Animal(name) { this.name = name; }
Animal.prototype.eat = function() { console.log(`${this.name} eats`); };

function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.bark = function() { console.log(`${this.name} barks`); };
```

**與 Java class-based 繼承對比**:

| | JS Prototype | Java Class |
|---|---|---|
| 結構 | **動態**(runtime 可改) | **靜態**(編譯期定) |
| 共享機制 | 共用同一個 prototype 物件 | class 模板 → 每個 instance 獨立 |
| 風險 | 改 prototype 影響所有實例 | 修改 class 需重編譯 |

**反模式**:**修改 built-in prototype**(`Array.prototype.myMethod = ...`)會污染所有陣列、可能跟未來 ECMAScript 標準衝突——禁止這麼做。

---

<a id="deep-clone"></a>
### Deep Clone(深拷貝)🟡

**定義**:
- **淺拷貝(shallow copy)**:只複製第一層,內部物件還指向同一參考
- **深拷貝(deep copy)**:遞迴複製所有層級,完全獨立,改一邊不影響另一邊

**為什麼用**:JS 的物件 / 陣列是**參考傳遞**,直接 `=` 只是複製參考(改一個影響全部)。需要「修改副本不影響原物件」的場景就要深拷貝(state 管理、Redux immutable update、避免 side effect)。

**主流做法**:

| 做法 | 範例 | 限制 |
|---|---|---|
| **`structuredClone()`** | `structuredClone(obj)` | **現代瀏覽器 / Node 17+ 原生**,推薦首選 |
| `JSON.parse(JSON.stringify())` | 一行解決 | **不支援**:`Date` / `Map` / `Set` / `function` / `undefined` / 循環參考 — 會丟資訊 |
| Lodash `_.cloneDeep` | `_.cloneDeep(obj)` | 第三方依賴,但功能完整、瀏覽器相容 |
| 手寫遞迴 | 自己寫 | 邊界情況多,不推薦 |

**範例**:
```javascript
const original = {
    user: { name: "Alice", joined: new Date() },
    tags: ["admin", "vip"]
};

// 淺拷貝(陷阱)
const shallow = { ...original };
shallow.user.name = "Bob";
console.log(original.user.name);   // "Bob" ← 被改到了!

// 深拷貝(現代首選)
const deep = structuredClone(original);
deep.user.name = "Charlie";
console.log(original.user.name);   // "Bob" ← 原本不變

// JSON 法的限制
const broken = JSON.parse(JSON.stringify(original));
console.log(broken.user.joined instanceof Date);   // false!Date 變成 string
```

**與 Java 對比**:Java 有 `Cloneable` 介面 + `clone()`,常見問題類似(預設淺拷貝)。常用替代:Apache Commons `SerializationUtils.clone(obj)` 或 Jackson 序列化來回。

---

<a id="eval"></a>
### `eval()`(字串執行,危險)🟡

**定義**:JavaScript 內建全域函式,**把字串當 JS 程式碼執行**並回傳最後一個表達式的值。**極度強大也極度危險**——多數情境**有更安全的替代**,實務應幾乎不用。

**基本範例**:
```javascript
eval("2 + 3");                    // 5
eval("const x = 10; x * x;");     // 100

// 可存取 eval 呼叫處的區域變數(scope)
function demo() {
    const secret = 42;
    return eval("secret + 1");    // 43
}
```

**為什麼說「**evil**」**(Douglas Crockford 的 *"eval is evil"*):

1 / **安全漏洞(XSS 的主要管道之一)**:
```javascript
// 想像 userInput 來自網址 query
const userInput = "alert('hacked'); fetch('/api/transfer-money', {...})";
eval(userInput);   // 攻擊者的 JS 在你的頁面執行,等同你的身份
```
若 `userInput` 來自 URL、localStorage、第三方資料,**`eval` 等於把 code execution 權限交出去**。

2 / **效能差**:
- JS 引擎(V8)依賴**靜態分析**做優化(inline、escape analysis)
- `eval` 在 runtime 才產生 code,**整個函式所在的作用域都會被去優化**(deopt)
- 對 hot path 殺傷力大

3 / **可讀性 / 維護性差**:
- IDE 無法跳轉、refactor 工具看不見 `eval` 內的識別符
- ESLint 預設規則 `no-eval` 直接禁

4 / **strict mode 行為不一致**:
```javascript
"use strict";
eval("var x = 10");
console.log(x);  // ReferenceError:strict mode 下 eval 不會污染外層 scope
```

**典型該避免的場景與替代方案**:

| 場景 | ❌ 用 eval | ✅ 替代 |
| --- | --- | --- |
| **解析 JSON 字串** | `eval("(" + jsonStr + ")")` | `JSON.parse(jsonStr)` |
| **動態屬性存取** | `eval("obj." + key)` | `obj[key]`(bracket notation) |
| **算數字表達式**(計算機) | `eval(userExpression)` | 用 [expr-eval](https://www.npmjs.com/package/expr-eval)、[mathjs](https://mathjs.org/) 等沙箱化的 expression parser |
| **動態建構函式** | `eval("function f() {...}")` | `new Function("a", "b", "return a + b")`(也危險,但有獨立 scope) |
| **動態載入程式** | `eval(loadedCode)` | **動態 import**(`import("./module.js")`) |
| **template 渲染** | `eval` 處理變數 | **模板引擎**(`mustache`、`ejs`、`handlebars`)或框架 template |

**`new Function()` vs `eval()`**:`new Function` 也是動態執行字串,但**有獨立的全域 scope**(不像 eval 能存取呼叫處 locals)——略**安全一點**,但仍危險,且仍會觸發 CSP `unsafe-eval` 限制。

**CSP 對 eval 的封鎖**(對應 [D2 XSS / CSP](./D2-web-security.md#xss)):
```
Content-Security-Policy: script-src 'self'    ← 不含 'unsafe-eval'
```
這個 header 一設,**所有 `eval()` 與 `new Function()` 都會被瀏覽器擋掉**——現代防 XSS 最有效手段之一。

**極少數合理使用情境**(真的非常少):
- **REPL / Online Editor**:需要執行使用者輸入的 code(本來就是設計目的,要在 sandbox / Worker 內隔離)
- **嚴格受信任、無使用者輸入**的內部工具
- **某些古老的 JS 元學程式設計需求**(現代有更好替代)

**Java 對應的危險原語**(類比):
- **`Runtime.exec(userInput)`** — 命令注入
- **JavaScript Engine in Java**(Nashorn 已棄用、GraalJS):`engine.eval(userInput)` 在 Java 內跑 JS,**同樣是 RCE 入口**
- **Reflection** + 字串方法名(若 method name 來自不可信輸入,同樣是 code execution)

**反模式總結**:
- ❌ 用 eval 解析 JSON——歷史包袱,**現代一律 `JSON.parse`**
- ❌ 用 eval 做動態 key 存取——`obj[key]` 即可
- ❌ 在生產 code 出現 `eval(somethingFromUser)`——立即 code review block
- ✅ ESLint 加上 `"no-eval": "error"`
- ✅ CSP 移除 `unsafe-eval`,讓瀏覽器替你把關

---

<a id="optional-chaining"></a>
### Optional Chaining(`?.`)🟢

**定義**:ES2020 引入的**安全鏈式存取**語法——`obj?.prop` 在 `obj` 為 `null` / `undefined` 時短路回傳 `undefined`,不會拋 `TypeError`。

**為什麼用**:處理「**可能不存在**」的物件鏈,取代落落長的 `&&` 防禦寫法。

**用法**:
```javascript
const user = { profile: { name: "Alice" } };

// 物件屬性
user?.profile?.name        // "Alice"
user?.address?.city        // undefined(不會錯)

// 陣列存取
arr?.[0]                   // 安全取第一個

// 函式呼叫(`onClick` 可能未定義)
btn.onClick?.()            // 有就呼叫,沒有就跳過

// 結合
config?.endpoints?.api?.url
```

**舊寫法對比**:
```javascript
// ❌ 舊寫法(冗長)
const name = user && user.profile && user.profile.name;

// ✅ Optional Chaining
const name = user?.profile?.name;
```

**注意陷阱**:`?.` **不會**「自動建立中間物件」——以下會錯:
```javascript
const obj = {};
obj?.a?.b = 1;   // SyntaxError(賦值左側不可用 ?.)
```

**與 Java `Optional` 對照**:Java `Optional.of(user).map(u -> u.getProfile()).map(p -> p.getName())` 是同等概念但語法重得多。JS 的 `?.` 是語言層支援,簡潔很多。

---

<a id="nullish-coalescing"></a>
### Nullish Coalescing(`??`)🟢

**定義**:ES2020 引入的「**只有 `null` 或 `undefined` 時走右邊**」運算子。跟 `||` 的關鍵差異:不會被 `0` / `""` / `false` 觸發。

**為什麼用**:`||` 的「falsy 檢查」常導致誤判——當 `0` / `""` / `false` 是合法值時,`||` 會錯誤地走 fallback。`??` 只在 `null` / `undefined` 才走 fallback,**語意更精準**。

**`??` vs `||` 深入對照**:

| 表達式 | `\|\|` 結果 | `??` 結果 | 為什麼差? |
|---|---|---|---|
| `0 \|\| 100` | `100` | `0` | `0` 是 falsy → `\|\|` fallback;但 `0` 不是 nullish |
| `"" \|\| "預設"` | `"預設"` | `""` | `""` 是 falsy → fallback;但 `""` 不是 nullish |
| `false \|\| true` | `true` | `false` | `false` 是 falsy → fallback;但 `false` 不是 nullish |
| `null ?? "預設"` | `"預設"` | `"預設"` | null 在兩者皆 fallback |
| `undefined ?? "預設"` | `"預設"` | `"預設"` | undefined 在兩者皆 fallback |
| `NaN ?? 0` | `0` | `NaN` | `NaN` 是 falsy → `\|\|` fallback;但 `NaN` 不是 nullish |

**典型誤用案例**:
```javascript
// ❌ Bug:當 stockCount 是 0(沒貨),會誤顯示「庫存不明」
const display = stockCount || "庫存不明";

// ✅ 正確:0 = 沒貨會顯示 0,只有真的沒值才顯示「庫存不明」
const display = stockCount ?? "庫存不明";
```

**`??=` 賦值版**(ES2021):
```javascript
let config = { timeout: undefined };
config.timeout ??= 5000;       // timeout 為 nullish 才賦值
// config.timeout === 5000
```

**搭配 `?.` 常用組合**:
```javascript
const lang = user?.preferences?.language ?? "zh-TW";
```

**與 Java 對照**:`??` 對應 Java 的 `Optional.orElse(default)`(Java 沒有區分「null 與假值」的需求,所以不需要這個運算子)。

---

<a id="jsx"></a>
### JSX 🟡

**定義**:**JavaScript 的語法擴充**,允許在 JS 裡寫類似 HTML 的標記。React 用 JSX 為主,Vue 也支援(但 Vue 主流仍用 template)。**JSX 不是 HTML**,而是會被 Babel / TS 編譯成 `React.createElement(...)` 函式呼叫的「**語法糖**」。

**為什麼用**:
- 把「結構(HTML)」與「邏輯(JS)」放在同一份檔案,**元件即模組**
- 編輯器、工具鏈成熟(語法檢查、auto-complete、refactor)
- TSX(TypeScript + JSX)讓元件 props 也能型別檢查

**範例**:
```jsx
// JSX 寫法
function Greeting({ name }) {
    return <h1 className="greeting">Hello, {name}!</h1>;
}

// 編譯後(等價)
function Greeting({ name }) {
    return React.createElement(
        "h1",
        { className: "greeting" },
        "Hello, ", name, "!"
    );
}
```

**與 HTML 的關鍵差異**:

| HTML | JSX | 原因 |
|---|---|---|
| `class="x"` | `className="x"` | `class` 是 JS 保留字 |
| `for="x"` | `htmlFor="x"` | `for` 是 JS 保留字 |
| 屬性 lowercase | 屬性 **camelCase**(`onclick` → `onClick`) | JS 慣例 |
| 可多 root | **必須單一 root**(或用 `<>...</>` Fragment) | 函式回傳值單一 |
| HTML 註解 `<!-- -->` | JS 註解 `{/* */}` | JSX 是 JS |

**Fragment**(避免多餘 wrapper div):
```jsx
return (
    <>
        <h1>Title</h1>
        <p>Content</p>
    </>
);
```

**Vue 的 JSX**:Vue 也支援(在 `.vue` 檔的 `<script setup lang="jsx">`),但**主流 Vue 仍用 template**,JSX 多用於需要動態渲染的場景。

---

← [返回索引(README.md)](./README.md)
