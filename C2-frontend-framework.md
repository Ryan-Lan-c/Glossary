# C2 - 前端框架與生態

← [返回索引(README.md)](./README.md)

---

## 目錄

### 主流框架
- [Vue 2 🟡](#vue2)
- [Vue 3 🟡](#vue3)
- [React 🟡](#react)
- [Angular 🟡](#angular)

### 狀態管理
- [Vuex 🟡](#vuex)
- [Pinia 🟡](#pinia)

### 元件通訊
- [Props 🟢](#props)
- [Emit(`$emit` / `defineEmits`)🟢](#emit)
- [Event Bus 🟡](#event-bus)

### 工具鏈
- [Vite 🟡](#vite)

### CSS / UI 框架
- [Tailwind CSS 🟡](#tailwind)
- [Bootstrap 🟢](#bootstrap)
- [Element Plus 🟡](#element-plus)
- [Quasar 🟡](#quasar)

---

## 主流框架

<a id="vue2"></a>
### Vue 2 🟡

**定義**:Evan You 2014 年發布漸進式前端框架,2016 年推出 Vue 2。**Options API** 為主、用 `Object.defineProperty` 實作響應式、配合 Vuex 為狀態管理——是 2017–2022 年中文社群最主流的 SPA 框架。

**為什麼用**:
- **學習曲線最平**(三大框架中)
- 中文資源最多
- 漸進式:可以從「在 HTML 加一個 `<script>`」一路擴展到完整 SPA
- 生態(Element UI、Vue Router、Vuex)成熟

**核心概念(Options API)**:
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <button @click="increment">{{ count }}</button>
  </div>
</template>

<script>
export default {
  data() {
    return { title: "Hello", count: 0 };
  },
  computed: {
    doubled() { return this.count * 2; }
  },
  methods: {
    increment() { this.count++; }
  },
  mounted() {
    console.log("元件掛載完成");
  }
};
</script>
```

**生命週期 hooks**:`beforeCreate` → `created` → `beforeMount` → `mounted` → `beforeUpdate` → `updated` → `beforeDestroy` → `destroyed`

**已知限制**(Vue 3 大多解決):
- 響應式底層 `Object.defineProperty` **不能偵測**新加屬性(要用 `Vue.set`)
- 不能偵測陣列 index 賦值(`arr[0] = x`)
- TypeScript 支援差(`vue-class-component` 是社群解決方案)

**EOL 狀態**:
- **2023-12-31 已 End of Life**(官方不再維護)
- LTS(NES)有付費延伸支援
- 大量舊專案還在跑(尤其後台管理、企業內部系統)

---

<a id="vue3"></a>
### Vue 3 🟡

**定義**:2020 年發布的下一代 Vue,**Composition API**(類似 React Hooks)、用 ES6 Proxy 實作響應式、TypeScript first-class、官方推薦狀態管理改為 Pinia——**現在 Vue 的預設選擇**。

**為什麼用**:
- 響應式底層改 Proxy,**可偵測新屬性、陣列 index 賦值**(Vue 2 的痛點解決)
- Composition API 讓邏輯**可重用**(類似 React Hooks 精神)
- TypeScript 支援大幅改善
- Tree-shaking 友善(bundle 較小)
- 新生態(Vite、Pinia)同步推進

**核心概念(Composition API + `<script setup>`)**:
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <button @click="increment">{{ count }}</button>
    <p>Doubled: {{ doubled }}</p>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

const title = "Hello";
const count = ref(0);                            // ref 包裝原始值
const doubled = computed(() => count.value * 2);

function increment() {
  count.value++;
}

onMounted(() => console.log("元件掛載完成"));
</script>
```

**Vue 2 vs Vue 3 對照**:

| 維度 | Vue 2 | Vue 3 |
|---|---|---|
| API 風格 | Options API | **Composition API**(Options API 仍可用) |
| 寫法 | `data() { return {...} }` | `ref()` / `reactive()` + `<script setup>` |
| 響應式底層 | `Object.defineProperty` | ES6 **Proxy** |
| 偵測新屬性 | ❌ 需 `Vue.set` | ✅ 自動 |
| 陣列 index 賦值 | ❌ 需 `Vue.set` | ✅ 自動 |
| TypeScript 支援 | 差(社群方案) | ✅ 內建一等公民 |
| Fragments(多 root) | ❌ 必須單一 root | ✅ |
| Teleport(渲染到別處) | ❌ | ✅ `<Teleport to="body">` |
| Suspense(非同步元件) | ❌ | ✅ 試驗性 |
| 生命週期名 | `beforeDestroy` | `onBeforeUnmount` |
| Bundle 大小 | 較大 | 較小(tree-shake 友善) |
| 官方狀態管理 | Vuex | **Pinia** |
| 構建工具 | Webpack(Vue CLI) | **Vite** |
| EOL 狀態 | **2023-12-31 已結束** | 主流維護中 |

> 📌 **個人實戰偏好**:一般遇到的專案都使用 **Composition API + `<script setup>`**。

---

<a id="react"></a>
### React 🟡

**定義**:Facebook(現 Meta)2013 年開源,**JSX + 函式組件 + Hooks** 為核心,**單向資料流 + Virtual DOM**。**全球用量最大**的前端框架。

**為什麼用**:
- 生態最大(npm 套件、教學資源、招募市場)
- **JSX** 讓「邏輯與結構」緊密結合
- Hooks(2018 年 React 16.8)使函式組件 + 邏輯重用變主流
- 元件化思維清晰(no template magic、no two-way binding)
- 配套生態完整(Next.js / React Router / Redux / Zustand / TanStack Query)

**核心概念(函式組件 + Hooks)**:
```jsx
import { useState, useEffect } from "react";

function Counter({ initial = 0 }) {
    const [count, setCount] = useState(initial);
    const doubled = count * 2;

    useEffect(() => {
        document.title = `Count: ${count}`;
    }, [count]);    // 依賴陣列,count 變才重跑

    return (
        <div>
            <button onClick={() => setCount(count + 1)}>
                {count}
            </button>
            <p>Doubled: {doubled}</p>
        </div>
    );
}
```

**常用 Hooks**:

| Hook | 用途 |
|---|---|
| `useState` | 元件 state |
| `useEffect` | 副作用(API call、訂閱、DOM 操作) |
| `useContext` | 跨層共享資料(避免 prop drilling) |
| `useMemo` | 記憶昂貴計算結果 |
| `useCallback` | 記憶函式參考(避免子元件 rerender) |
| `useReducer` | 複雜 state 邏輯(類 Redux 風格) |
| `useRef` | 持久化參考(取 DOM、存可變值) |

**設計哲學差異(vs Vue)**:
- React **自由度高、約定少**(state 管理、router、Form 都自己選)
- Vue **約定多**(template 語法、響應式、官方 router)

**生態**:
- **Next.js** — SSR / SSG / 全端框架(現代 React 預設)
- **React Native** — 跨平台 mobile App
- **TanStack Query** — Server state 管理(取代 Redux 用於 fetch)
- **Zustand / Redux** — Client state 管理

---

<a id="angular"></a>
### Angular 🟡

**定義**:Google 出品,2016 年發布 Angular 2(完全重寫,跟 AngularJS 1 是不同產品)。**完整框架**——routing / forms / HTTP / i18n 全內建,**TypeScript 預設且強制**,Decorator 風格。

**為什麼用**:
- **意見強烈、約定多**——團隊不需爭論「該用哪個 router、哪個狀態管理」
- TypeScript 一等公民(Angular 用 TS 開發、強制使用)
- **DI 容器(類似 Spring 風格)**,Java 後端轉前端會熟悉
- RxJS Observables 處理非同步資料流
- **大企業 / 大團隊偏好**(LOB 應用、銀行、電信)

**核心概念**:
```typescript
// Component
@Component({
    selector: "app-counter",
    template: `
        <button (click)="increment()">{{ count }}</button>
        <p>Doubled: {{ doubled }}</p>
    `
})
export class CounterComponent {
    count = 0;
    get doubled() { return this.count * 2; }

    constructor(private logger: LoggerService) {}    // ← DI

    increment() {
        this.count++;
        this.logger.log(`Count: ${this.count}`);
    }
}

// Service
@Injectable({ providedIn: "root" })
export class LoggerService {
    log(msg: string) { console.log(`[LOG] ${msg}`); }
}
```

**三大框架對照**:

| 維度 | React | Vue | Angular |
|---|---|---|---|
| 出生 | 2013 | 2014 | 2016(v2) |
| 出品 | Meta | Evan You(社群) | Google |
| 範式 | 函式組件 + Hooks | SFC + Composition API | Class + DI + Decorator |
| 模板 | JSX | template(也支援 JSX) | template |
| TypeScript | 支援(可選) | 支援(Vue 3 一等公民) | **強制** |
| 學習曲線 | 中(自由度高) | 平 | 陡(意見強) |
| Bundle 大小 | 小 | 小 | 大(完整框架) |
| 適用 | 各種規模 | 中小到大 | **大企業 / 大團隊** |
| 中文社群 | 大 | 最大 | 小 |
| Routing / Form / HTTP | 各自社群方案 | 官方方案 | **內建** |

**注意**:**AngularJS(Angular 1)** 跟 **Angular(2+)** 是完全不同的產品(架構、API 都重寫),AngularJS 已 EOL——現在說「Angular」一律指 2+。

> 📌 **個人實戰偏好**:**中信大電營**專案有用過。

---

## 狀態管理

<a id="vuex"></a>
### Vuex 🟡

**定義**:Vue 2 時代官方主流的**集中式狀態管理**,設計受 Redux 啟發——**State / Mutation / Action / Getter** 四件套。

**為什麼用**:
- 跨元件共享狀態(避免 prop drilling)
- 統一狀態變更入口(便於 debug、time-travel debugging via Vue DevTools)
- 大型應用的 state 邏輯可預測

**核心概念**:

| 概念 | 角色 | 範例 |
|---|---|---|
| **State** | 唯一真實資料來源 | `state: { count: 0 }` |
| **Mutation** | **同步**修改 state(唯一可改的途徑) | `mutations: { INC(s) { s.count++ } }` |
| **Action** | **非同步**操作,完成後 commit mutation | `actions: { async load({commit}) { ... commit('INC') } }` |
| **Getter** | 派生狀態(類 computed) | `getters: { doubled: s => s.count * 2 }` |

**範例**:
```javascript
import Vuex from "vuex";

const store = new Vuex.Store({
    state: { count: 0 },
    mutations: {
        INCREMENT(state) { state.count++; },
        ADD(state, payload) { state.count += payload; }
    },
    actions: {
        async fetchAndAdd({ commit }, n) {
            await api.log(n);
            commit("ADD", n);
        }
    },
    getters: {
        doubled: state => state.count * 2
    }
});

// 元件用
this.$store.commit("INCREMENT");
this.$store.dispatch("fetchAndAdd", 5);
this.$store.state.count;
this.$store.getters.doubled;
```

**Vue 3 之後**:
- **官方推薦改用 Pinia**(更簡潔、TS 友善、無 Mutation)
- Vuex 仍能用,但新專案幾乎都選 Pinia
- 舊專案升級 Vue 3 時,**狀態管理常一起遷移到 Pinia**

> 📌 **個人實戰偏好**:舊專案有用過,目前大多已升級改用 **Pinia**。

---

<a id="pinia"></a>
### Pinia 🟡

**定義**:Vue 3 的**官方推薦**狀態管理(取代 Vuex)。Composition API 風格、無 Mutation、TypeScript 友善。Vue 作者背書、官方文件直接推薦。

**為什麼用**:
- **API 簡潔**(沒 Mutation,直接改 state)
- **TypeScript 一等公民**(infer 完整,不用手動標型)
- **模組化**:每個 store 是獨立檔案,自動 code-splitting
- **DevTools 整合**(Vue DevTools 一鍵看 state、time-travel)
- 也支援 Vue 2(漸進升級路徑)

**範例**:
```typescript
// stores/counter.ts
import { defineStore } from "pinia";

export const useCounterStore = defineStore("counter", {
    state: () => ({ count: 0 }),
    getters: {
        doubled: state => state.count * 2
    },
    actions: {
        increment() { this.count++; },
        async fetchAndAdd(n: number) {
            await api.log(n);
            this.count += n;
        }
    }
});
```

```vue
<!-- 元件用(Composition API) -->
<script setup>
import { useCounterStore } from "@/stores/counter";

const counter = useCounterStore();
counter.count;             // 直接讀
counter.count++;           // 直接改(不用 commit!)
counter.increment();
counter.fetchAndAdd(5);
</script>
```

**Vuex vs Pinia 對照**:

| 維度 | Vuex | Pinia |
|---|---|---|
| 推出 | 2015(Vue 2 時代) | 2019,Vue 3 後成主流 |
| 核心概念 | State + Mutation + Action + Getter | State + Getter + Action(**無 Mutation**) |
| 修改 state | **必須** commit Mutation | **直接賦值** |
| Module 拆分 | 巢狀 modules,namespace 字串 | 每個 store 獨立檔案,自動隔離 |
| TypeScript 支援 | 差(需大量手動型別) | ✅ 完整 infer |
| Composition API 支援 | 差 | ✅ 原生 |
| Bundle 大小 | 較大 | 較小 |
| Vue 2 / Vue 3 | 各有版本 | 兩者皆支援 |
| Vue 官方推薦 | 過去 | **現在** |

> 📌 **個人實戰偏好**:**主用**。

---

## 元件通訊

<a id="props"></a>
### Props 🟢

**定義**:**父元件傳資料給子元件**的標準方式,Vue / React / Angular 共通概念,**單向資料流**(子元件不應改動 props)。

**為什麼用**:
- 元件的「**輸入介面**」——讓元件可重用(同個 `<UserCard>` 顯示不同 user)
- 強制單向資料流,可預測
- 配合 TypeScript 型別宣告,介面契約明確

**Vue 範例(Composition API + TS)**:
```vue
<script setup lang="ts">
interface Props {
    name: string;
    age?: number;          // 可選
    isAdmin?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
    age: 18,
    isAdmin: false
});
</script>

<template>
  <div>{{ props.name }} ({{ props.age }})</div>
</template>

<!-- 父元件用 -->
<UserCard name="Alice" :age="30" :is-admin="true" />
```

**React 範例(TS)**:
```tsx
interface Props {
    name: string;
    age?: number;
    isAdmin?: boolean;
}

function UserCard({ name, age = 18, isAdmin = false }: Props) {
    return <div>{name} ({age})</div>;
}

// 父元件用
<UserCard name="Alice" age={30} isAdmin={true} />
```

**為什麼不應直接改 props**:
- **Vue**:會被警告(props 是 readonly proxy),且資料變動方向會混亂
- **React**:props 在函式組件中本來就無法改(它是函式參數)

**正確做法**:子元件需要「修改」時,**透過 emit / callback 通知父元件**(父改完後再傳新 props 下來)。

---

<a id="emit"></a>
### Emit(`$emit` / `defineEmits`)🟢

**定義**:Vue 子元件向父元件「**發出事件**」的機制——子元件 `emit('event-name', payload)`,父元件用 `@event-name="handler"` 接收。React 沒有 `emit`,改用「傳一個 callback function 當 prop」。

**為什麼用**:
- 維持「**單向資料流**」(子不能直接改 props,只能通知父)
- 元件解耦(子元件不知道父元件是誰)
- 介面顯式(父元件能看出子元件會發哪些事件)

**Vue 2 vs Vue 3 寫法對照**:

```javascript
// Vue 2(Options API)
export default {
  methods: {
    onClick() {
      this.$emit("submit", { data: 123 });
    }
  }
};
```

```vue
<!-- Vue 3(Composition API + <script setup>) -->
<script setup>
const emit = defineEmits(["submit", "cancel"]);
// 或 TS 強型別
// const emit = defineEmits<{ submit: [payload: { data: number }]; cancel: [] }>();

function onClick() {
  emit("submit", { data: 123 });
}
</script>

<!-- 父元件 -->
<MyForm @submit="handleSubmit" @cancel="handleCancel" />
```

**React 對照(沒 emit,用 callback prop)**:
```jsx
// 子元件
function MyForm({ onSubmit, onCancel }) {
    return <button onClick={() => onSubmit({ data: 123 })}>送出</button>;
}

// 父元件
<MyForm onSubmit={handleSubmit} onCancel={handleCancel} />
```

**核心差異**:
- **Vue emit** = 用「事件名字串」+ `@event` 接收(類似 DOM event)
- **React callback prop** = 用「函式參考」直接傳遞(更 JS 原生)

兩者效果相同,但 React 開發者第一次接觸 Vue emit 會覺得「為什麼不直接傳 function?」—— 這是 Vue 對 DOM event 風格的延續。

---

<a id="event-bus"></a>
### Event Bus 🟡

**定義**:**跨元件通訊**(非父子關係)的舊式做法——一個全域物件,任何元件都能 `emit` 與 `on`,當作事件中繼站。Vue 2 內建可用,**Vue 3 官方移除**。

**為什麼用過**:
- 早期 Vue 2 解決「兄弟元件 / 跨層元件溝通」的常見手段
- 比 Vuex 輕量(不用學 mutation/action)
- 程式碼少時方便

**Vue 2 範例**:
```javascript
// event-bus.js
import Vue from "vue";
export const EventBus = new Vue();

// 元件 A:發出事件
EventBus.$emit("user-updated", { id: 1, name: "Alice" });

// 元件 B:接收
EventBus.$on("user-updated", payload => { /* ... */ });

// ⚠ 記得清理(避免記憶體洩漏)
beforeDestroy() {
  EventBus.$off("user-updated");
}
```

**為什麼 Vue 3 移除**:
- **資料流難追蹤**——任何元件都能發/收,debug 噩夢(誰改了 state?到處 grep)
- **記憶體洩漏陷阱**(忘記 `$off`)
- **更好的替代方案出現**:Pinia / Provide-Inject

**Vue 3 替代方案**:

| 方案 | 適用 |
|---|---|
| **Pinia store** | 跨元件共享 state(推薦) |
| **Provide / Inject** | 父元件向多層子元件傳資料 |
| **mitt 套件**(`npm i mitt`) | 真的需要 event bus 的場景 |
| **Composables**(`useXxx` hook) | 共享邏輯而非 state |

**反模式提醒**:**新專案不應用 Event Bus**——現代 Vue 3 不需要它。出現 `mitt` 通常是「從 Vue 2 升級時順便保留」。

> 📌 **個人實戰偏好**:**長榮專案**有用過。

---

## 工具鏈

<a id="vite"></a>
### Vite 🟡

**定義**:Vue 作者 Evan You 在 2020 年發布的**下一代前端構建工具**——開發伺服器使用**原生 ES Modules**(瀏覽器直接認的 `import`),**冷啟動極快**;生產構建用 Rollup。發音:**Vite**(法文「快」)= /vit/。

**為什麼用**:
- **冷啟動極快**(原生 ESM,不需把全部模組打包)
- **HMR 極快**(只重編輯到的檔)
- 開箱即用支援 Vue / React / Svelte / 純 TS
- `vite.config.js` 配置簡單(對比 Webpack)
- 已成 Vue / Svelte 預設,React 也常見(Next.js 例外,Next 用自家 turbopack)

**範例**:
```bash
# 建專案
npm create vite@latest my-app -- --template vue-ts

# 啟動 dev server
npm run dev   # 通常 < 1 秒

# 構建
npm run build
```

**Vite vs Webpack 對照**:

| 維度 | Webpack | Vite |
|---|---|---|
| 出生 | 2012 | 2020 |
| Dev server 啟動 | **慢**(全模組打包後才能跑) | **極快**(原生 ESM) |
| HMR 速度 | 受專案大小影響 | 幾乎不受 |
| 生產構建 | Webpack 自身 | **Rollup**(更好的 tree-shake) |
| 設定複雜度 | 高(`webpack.config.js` 常上百行) | 低 |
| 生態 / 老專案支援 | 完整 / 全部 | 仍在追趕 |
| 預設於 | (傳統)Vue CLI、Create React App | Vue 3、Svelte、Astro |
| 適用 | 舊專案維護 | 新專案 |

> 📌 **個人實戰偏好**:大部分專案有用。

---

## CSS / UI 框架

<a id="tailwind"></a>
### Tailwind CSS 🟡

**定義**:**Utility-first** 的 CSS 框架——不提供「Button」「Card」這種預製元件,而是提供大量「**單一用途的 class**」(`flex`、`pt-4`、`text-red-500`),你**直接在 HTML class 屬性組合**出設計。

**為什麼用**:
- **JIT 編譯**——只把實際用到的 class 編入 CSS,bundle 極小
- 設計系統強制一致(間距、顏色都是預定義 scale)
- 不用為 CSS 命名抓破頭(不再有 `card-header__title--active` 這種 BEM)
- 響應式 / hover / dark mode 都是 class 前綴(`md:flex` / `hover:bg-blue-500` / `dark:text-white`)
- 與框架無關(Vue / React / Angular / 純 HTML 都能用)

**範例**:
```html
<!-- 一個按鈕 -->
<button class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white font-bold rounded">
  Click me
</button>

<!-- 響應式(手機 1 欄、桌面 3 欄) -->
<div class="grid grid-cols-1 md:grid-cols-3 gap-4">
  <Card />
  <Card />
  <Card />
</div>
```

**設定**(`tailwind.config.js`):
```javascript
export default {
  content: ["./src/**/*.{vue,jsx,tsx,html}"],
  theme: {
    extend: {
      colors: { brand: "#1a73e8" }
    }
  }
};
```

**Tailwind vs Bootstrap 對照**:

| 維度 | Tailwind | Bootstrap |
|---|---|---|
| 哲學 | Utility-first(原子 class) | Component-first(預製元件) |
| 學習成本 | 中(要記 class) | 低(複製範例) |
| 設計自由度 | 高(無預設 look) | 低(預設 look 明顯) |
| HTML 長度 | **長**(class 多) | 短 |
| Bundle 大小 | **極小**(JIT 只打包用到的) | 中等 |
| 適合 | 客製設計、設計系統 | 快速原型、後台 |

**初學爭議**:「**HTML 變很長 / 醜**」是常見抱怨——但 component 化(Vue / React)後,長 class 只出現在元件內部,使用方乾淨。

> 📌 **個人實戰偏好**:個人 side project 預計使用。

---

<a id="bootstrap"></a>
### Bootstrap 🟢

**定義**:Twitter 出品(2011)、最老牌的 CSS 框架——**Component-first**,提供 Button / Modal / Nav 等**預製元件 class**,12-column Grid 響應式設計。**Bootstrap 5(2021)移除 jQuery 依賴**。

**為什麼用**:
- **快速搭原型 / MVP**(不用設計就有過得去的 look)
- 學習曲線最低(複製官方範例就能跑)
- 老開發者熟悉(2011 開始流行)
- **後台管理 / 內部系統**常見(不講究設計)

**範例**:
```html
<!-- 直接用預製元件 class -->
<button class="btn btn-primary btn-lg">Primary Button</button>

<div class="container">
  <div class="row">
    <div class="col-md-6">左半</div>
    <div class="col-md-6">右半</div>
  </div>
</div>

<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <a class="navbar-brand" href="#">Logo</a>
</nav>
```

**Bootstrap 4 → 5 重要變化**:
- 移除 jQuery 依賴(改用 vanilla JS + ES Module)
- Flexbox-based grid(取代浮動)
- 字體 Reboot(更現代)

**現代地位**:
- **新專案**:Tailwind / 設計系統興起,Bootstrap 使用率下降
- **後台 / 內部系統**:仍常見(快、夠用、團隊熟)
- **WordPress / Bootstrap 主題**:仍有龐大市場

---

<a id="element-plus"></a>
### Element Plus 🟡

**定義**:**餓了麼**開源、**Vue 3 最主流**的 UI 元件庫(Element UI 的 Vue 3 版本)。針對**企業中後台場景**(管理系統、Dashboard、表單、表格)設計,中文社群普及度極高。

**為什麼用**:
- 企業後台常用元件齊全(Table、Form、Pagination、Dialog、Tree、DatePicker)
- **中文文件、中文 Issue 支援**佳(餓了麼是中國公司)
- 設計風格商務、保守(適合 LOB 場景)
- 按需引入(tree-shaking,只打包用到的元件)
- 與 Pinia / Vue Router 整合順暢

**範例**:
```vue
<script setup>
import { ElButton, ElTable, ElTableColumn, ElDialog } from "element-plus";
import { ref } from "vue";

const tableData = ref([
    { name: "Alice", age: 30 },
    { name: "Bob", age: 25 }
]);
const dialogVisible = ref(false);
</script>

<template>
  <ElButton type="primary" @click="dialogVisible = true">開啟</ElButton>

  <ElTable :data="tableData">
    <ElTableColumn prop="name" label="姓名" />
    <ElTableColumn prop="age" label="年齡" />
  </ElTable>

  <ElDialog v-model="dialogVisible" title="提示">內容</ElDialog>
</template>
```

**對照其他 Vue 3 UI 庫**:

| | Element Plus | Ant Design Vue | Naive UI |
|---|---|---|---|
| 風格 | 商務、傳統 | 商務、Ant 系 | 現代、簡約 |
| 中文社群 | 最大 | 大(Ant 系) | 中 |
| 後台場景 | ✅ 主流 | ✅ 主流 | ✅ |
| TypeScript | 良好 | 良好 | **最佳**(全 TS 寫) |
| 上手難度 | 平 | 中 | 平 |

> 📌 **個人實戰偏好**:**中信專案**大多使用。

---

<a id="quasar"></a>
### Quasar 🟡

**定義**:**Vue 3 全方位框架**——不只是 UI 元件庫,而是**單一 codebase 部署多平台**(SPA / SSR / PWA / Mobile / Desktop)的完整框架。設計遵循 Material Design。

**為什麼用**:
- **「一份 code 跑遍所有平台」**——Web / iOS / Android(via Capacitor / Cordova)/ Desktop(via Electron)/ PWA
- 內建超完整元件(比 Element Plus 還多,因為要涵蓋 mobile)
- Quasar CLI 工具鏈強大(build target 切換、模擬器整合)
- Material Design 設計系統(Google 風格,跨平台一致)
- TypeScript 一等公民

**範例**:
```bash
# 建專案
npm i -g @quasar/cli
quasar create my-app

# 切換 build target
quasar dev               # SPA
quasar dev -m ssr        # SSR
quasar dev -m pwa        # PWA
quasar dev -m capacitor  # Mobile
quasar dev -m electron   # Desktop
```

```vue
<template>
  <q-page padding>
    <q-btn color="primary" label="Click" @click="onClick" />
    <q-table :rows="rows" :columns="columns" />
  </q-page>
</template>
```

**Element Plus vs Quasar 對照**:

| | Element Plus | Quasar |
|---|---|---|
| 定位 | UI 元件庫 | 完整框架(UI + build target) |
| 設計風格 | 商務 / 中後台 | Material Design |
| 多平台部署 | ❌(只 Web) | ✅ Web / Mobile / Desktop |
| CLI 工具 | ❌ | ✅ Quasar CLI |
| Bundle 大小 | 小 | 中(包很多) |
| 中文社群 | 大 | 中 |
| 適合 | 企業後台 | 跨平台應用、新創產品 |

> 📌 **個人實戰偏好**:**長榮專案**、**eap** 都是使用 Quasar。

---

← [返回索引(README.md)](./README.md)
