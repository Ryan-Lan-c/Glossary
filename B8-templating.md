# B8 - 模板引擎(JSP / Thymeleaf)

← [返回索引(README.md)](./README.md)

---

> 本章聚焦 **Java 後端的 HTML 模板引擎**——也就是「Server 端組 HTML 字串」的傳統方式。
> 現代前端(Vue / React / Angular)的模板系統,見 [C2 前端框架與生態](./C2-frontend-framework.md)。

## 目錄

- [JSP(JavaServer Pages)🟡](#jsp)
- [JSTL 🟢](#jstl)
- [Thymeleaf 🟡](#thymeleaf)
- [JSP vs Thymeleaf 對照 🟡](#jsp-vs-thymeleaf)
- [模板引擎 vs SPA 的取捨 🟡](#template-vs-spa)

---

<a id="jsp"></a>
### JSP(JavaServer Pages)🟡

**定義**:Java EE 傳統的 Server 端 HTML 模板技術——`.jsp` 檔內混合 HTML 與 Java code,Web Container(Tomcat / Jetty)會把它編譯成 **Servlet** 後執行。

**為什麼用**:
- **歷史地位**:1999 年問世,Java Web 開發的「初代官方解法」
- 跟 Servlet 同陣營,Java EE 標準的一部分
- 大量舊系統還在用(銀行、政府、ERP)
- **新系統幾乎不選 JSP**(原因見下方)

**核心語法**:
```jsp
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="c" uri="jakarta.tags.core" %>
<html><body>
  <h1>Hello, <%= request.getAttribute("user") %></h1>   <%-- Scriptlet 表達式 --%>
  <c:forEach var="item" items="${items}">                <%-- JSTL 標籤 --%>
    <p>${item.name}</p>                                  <%-- EL 表達式 --%>
  </c:forEach>
  <%
    int count = (int) request.getAttribute("count");     // 傳統 Scriptlet,現代避免
  %>
</body></html>
```

**為什麼新系統不選**:
- View 與業務邏輯混雜(Scriptlet 寫 Java 在 HTML 裡)
- 難以單元測試(View 沒 standalone 渲染)
- 不能在瀏覽器直接打開預覽(語法不是 valid HTML)
- Spring Boot 預設**不**支援 JSP(要切到外部 Tomcat 才能跑)
- 編輯器支援不如 Thymeleaf 友善

---

<a id="jstl"></a>
### JSTL(JSP Standard Tag Library)🟢

**定義**:JSP 的標準標籤庫——讓你用 `<c:if>` / `<c:forEach>` 取代 Scriptlet 的 Java code,提升可讀性。

**為什麼用**:
- 把流程控制從「混 Java code 的 Scriptlet」變成「**像 HTML 標籤**」
- 配合 EL(Expression Language)`${user.name}` 取資料,不用寫 `<%= %>`
- 是 JSP「**還能用得下去**」的關鍵——沒有 JSTL,JSP 會變成 Java + HTML 大雜燴

**核心標籤**:
```jsp
<%@ taglib prefix="c" uri="jakarta.tags.core" %>

<c:if test="${user.age >= 18}">
  <p>歡迎成年訪客</p>
</c:if>

<c:choose>
  <c:when test="${score >= 90}"><p>優秀</p></c:when>
  <c:when test="${score >= 60}"><p>合格</p></c:when>
  <c:otherwise><p>不及格</p></c:otherwise>
</c:choose>

<c:forEach var="item" items="${items}" varStatus="loop">
  <p>${loop.index}: ${item.name}</p>
</c:forEach>
```

**版本注意**:JSTL 1.2 用 `javax.servlet.jsp.jstl` namespace → Jakarta EE 9 後改成 `jakarta.servlet.jsp.jstl`,套件名與 URI 都變,升級時需全文替換。

---

<a id="thymeleaf"></a>
### Thymeleaf 🟡

**定義**:Spring Boot 推薦的現代模板引擎——`.html` 檔本身是**合法 HTML**,Thymeleaf 用 **`th:` 前綴屬性** 標記動態行為,所以**可以直接在瀏覽器打開原始檔預覽**(這叫 **Natural Template**)。

**為什麼用**:
- **自然模板**——Designer 與 Developer 共用同一檔(打開預覽顯示佔位文字,渲染後替換)
- Spring Boot `spring-boot-starter-thymeleaf` 一個依賴搞定
- **預設自動 escape 防 XSS**
- 與 Spring 系列(Spring Security `sec:authorize`、Form Binding、i18n MessageSource)深度整合
- **Email 模板**的事實標準(Spring 生態下)

**核心語法**:
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
  <h1 th:text="${title}">[Designer 預覽用文字,渲染時被替換]</h1>

  <ul>
    <li th:each="item : ${items}" th:text="${item.name}">範例</li>
  </ul>

  <p th:if="${user.admin}">管理員專屬區塊</p>

  <a th:href="@{/users/{id}(id=${user.id})}">看詳情</a>
</body>
</html>
```

**Layout Dialect**(共用 layout):
```html
<!-- layout.html -->
<html xmlns:th="..." xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
  <body>
    <header>共用 header</header>
    <main layout:fragment="content">頁面內容會被替換</main>
  </body>
</html>

<!-- about.html -->
<html xmlns:layout="..." layout:decorate="~{layout}">
  <body>
    <main layout:fragment="content"><p>About 頁內容</p></main>
  </body>
</html>
```

**與 Spring Security 整合**:
```html
<div sec:authorize="hasRole('ADMIN')">只有管理員看得到</div>
<span sec:authentication="name">使用者名稱</span>
```

---

<a id="jsp-vs-thymeleaf"></a>
### JSP vs Thymeleaf 對照 🟡

**核心差異一句話**:
- **JSP** = 老派、語法混亂、Java EE 標準
- **Thymeleaf** = 現代、語法乾淨、Spring Boot 預設

**對照表**:

| 維度 | JSP | Thymeleaf |
|---|---|---|
| 檔案副檔名 | `.jsp` | `.html` |
| 瀏覽器直接預覽 | ❌(Scriptlet 不是 valid HTML) | ✅(Natural Template) |
| Spring Boot 整合 | ⚠ 需切到外部 Tomcat、額外設定 | ✅ 預設 starter |
| 自動 escape XSS | 需 `<c:out>` 或 EL `${...}` | ✅ 預設 |
| Layout 共用 | `<jsp:include>`(較古老) | Layout Dialect / Fragments(現代) |
| 學習曲線 | 中(需懂 EL + JSTL + Scriptlet) | 平 |
| 編輯器支援 | 一般 | IntelliJ / VSCode 都有專屬支援 |
| 執行模型 | 編譯成 Servlet | 直接解析 HTML |
| 社群活躍度 | 維持(舊系統) | 活躍(新系統首選) |
| 適用 | 舊 Java EE 系統維護 | 新 Spring Boot 後端模板 |

**結論**:**新專案幾乎都選 Thymeleaf**;接觸 JSP 通常是為了維護舊系統。

---

<a id="template-vs-spa"></a>
### 模板引擎 vs SPA 的取捨 🟡

**定義**:兩種「Web 前端渲染」的根本路線——
- **SSR / 模板引擎**:Server 組好 HTML,瀏覽器拿到就顯示
- **SPA(Single Page Application)**:Server 給空 HTML 殼 + JS bundle,瀏覽器跑 JS、呼叫 API 動態渲染(主流前端框架見 [C2 前端框架與生態](./C2-frontend-framework.md))

**為什麼是個取捨**:不是技術好壞,而是「**團隊成本、SEO、互動性、適用場景**」綜合判斷。

**對照表**:

| 維度 | 模板引擎(SSR) | SPA |
|---|---|---|
| 初始載入速度 | 快(直接是 HTML) | 慢(等 JS 下載 + 執行) |
| 互動性 | 每次操作 reload | 流暢(像 Native App) |
| SEO | ✅ 對搜尋引擎友善 | ⚠ 需處理(SSR / prerender) |
| 後端工程量 | 大(Server 寫 View) | 小(Server 只寫 API) |
| 前端工程量 | 小 | 大(整個 UI 框架) |
| 團隊分工 | Server 工程師全包 | 前後端分離 |
| 適用 | 內部後台、表單系統、SEO 重的官網 | 高互動 App、社交、編輯器 |

**混合方案**:
- **Hydration**:Server 先 SSR 出 HTML,前端 JS 接管成 SPA(**Next.js / Nuxt** 的核心賣點)
- **Islands Architecture**:大部分靜態,只把互動區塊 hydrate(Astro / Qwik)

**現實常見組合**:
- 後台管理 / Dashboard → SPA(互動重)
- 公開官網 / 部落格 → SSR(SEO 重)
- B2B 系統 → 混合或 SSR(內部用,SEO 不重要)

---

← [返回索引(README.md)](./README.md)
