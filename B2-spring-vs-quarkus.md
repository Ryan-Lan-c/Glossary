# B2 - 框架生態:Spring Boot vs Quarkus

← [返回索引(README.md)](./README.md)

---

## 為什麼放在一起講?

Spring Boot 與 Quarkus 在概念上**高度重疊**(都做 IoC、DI、AOP、配置管理),但底層哲學不同:
- **Spring Boot**:基於 Spring Framework,大量在**執行期**透過 reflection / proxy 處理。
- **Quarkus**:基於 Jakarta EE / MicroProfile,把能在**編譯期**處理的都在編譯期處理(Build-time augmentation),啟動極快、記憶體小,適合 GraalVM Native Image。

> 對 Quarkus 的「編譯期魔法」與 Native Image 細節有興趣,請看 [B7-quarkus.md](./B7-quarkus.md)。

---

## 目錄

- [IoC(控制反轉)🟢](#ioc)
- [DI(依賴注入)🟢](#di)
- [Bean 🟢](#bean)
- [Bean Scope(作用域)🟡](#bean-scope)
- [Component Scan(元件掃描)🟢](#component-scan)
- [Configuration / @Bean 工廠 🟡](#configuration)
- [Profile / Config Profile 🟢](#profile)
- [AutoConfiguration / Quarkus Extension 🟡](#autoconfiguration)
- [@ConditionalOnMissingBean 🟡](#conditional)
- [Spring MVC 🟢](#spring-mvc)
- [`@RestController`(獨立介紹)🟢](#rest-controller)
- [HTTP Endpoint:`@RestController` vs `@Path` 🟢](#http-endpoint)
- [API Versioning(API 版本管理)🟡](#api-versioning)
- [@RestControllerAdvice / `@ServerExceptionMapper` 🟡](#exception-handler)
- [@Transactional 🟡](#transactional)
- [@Auditable(自訂 annotation)🟡](#auditable)
- [核心對照表](#summary-table)

---

<a id="ioc"></a>
### IoC(Inversion of Control,控制反轉)🟢

**定義**:傳統程式由你 `new` 物件、控制流程;**IoC 是把「物件建立與生命週期」的控制權交給 Container**。你只宣告「我需要什麼」,Container 幫你組裝好。

**為什麼用**:
- 解耦:你不再依賴具體實作,只依賴抽象
- 可測試:測試時注入 mock 即可
- 集中管理:單例、初始化、銷毀都在 Container 控制

**Spring vs Quarkus**:兩者都是 IoC Container,Spring 用自己的 `ApplicationContext`,Quarkus 用 **CDI(Contexts and Dependency Injection)**——Jakarta EE 的標準。

---

<a id="di"></a>
### DI(Dependency Injection,依賴注入)🟢

**定義**:IoC 的具體實作手段——**把依賴從外部傳入物件**,而非物件內部 `new`。

**三種注入方式**:
```java
// 1. Constructor Injection(推薦,Lombok 一行搞定)
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository repo;
}

// 2. Setter Injection(罕用)
@Autowired public void setRepo(OrderRepository repo) { this.repo = repo; }

// 3. Field Injection(不推薦,難測試、無法用 final)
@Autowired private OrderRepository repo;
```

**Quarkus 寫法**(CDI):
```java
@ApplicationScoped
public class OrderService {
    @Inject OrderRepository repo;       // 等同 @Autowired
    // 或 constructor injection(推薦)
    public OrderService(OrderRepository repo) { this.repo = repo; }
}
```

**最佳實踐**:**永遠用 Constructor Injection**——不可變、可單元測試、編譯期檢查依賴。

---

<a id="bean"></a>
### Bean 🟢

**定義**:由 Container 管理生命週期的物件。

| 概念 | Spring | Quarkus(CDI) |
| --- | --- | --- |
| 標記為元件 | `@Component` / `@Service` / `@Repository` | `@ApplicationScoped` / `@Singleton` |
| 工廠方法 | `@Bean`(在 `@Configuration` 內) | `@Produces` |
| 注入 | `@Autowired`(可省略) | `@Inject` |

**範例**:
```java
// Spring
@Service public class FooService {}

// Quarkus
@ApplicationScoped public class FooService {}
```

---

<a id="bean-scope"></a>
### Bean Scope(作用域)🟡

| Scope | Spring | Quarkus | 說明 |
| --- | --- | --- | --- |
| 單例 | `@Scope("singleton")`(預設) | `@ApplicationScoped` / `@Singleton` | 整個應用一個 |
| 每次新建 | `@Scope("prototype")` | `@Dependent`(預設) | 每次注入都產生新實例 |
| 每請求一個 | `@RequestScope` | `@RequestScoped` | HTTP 請求生命週期 |
| 每 Session 一個 | `@SessionScope` | `@SessionScoped` | Session 生命週期 |

**注意 Quarkus 的差異**:Quarkus 的 `@ApplicationScoped` 是**懶載入 + Proxy**,`@Singleton` 是**啟動時即建立 + 無 Proxy**(更快但無法被代理,不能用某些功能)。

---

<a id="component-scan"></a>
### Component Scan(元件掃描)🟢

**定義**:Container 啟動時掃描指定 package,自動把標記為 Bean 的類別註冊。

**Spring**:`@SpringBootApplication` 預設掃描 main class 所在的 package 與其子 package。
**Quarkus**:預設掃描整個 application,**不需設定**,且許多事情在編譯期就完成。

---

<a id="configuration"></a>
### @Configuration / @Bean 工廠 🟡

**定義**:當你需要為**第三方類別**產生 Bean(無法直接加 `@Component`)時用這個。

**Spring**:
```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory cf) {
        var t = new RedisTemplate<String, Object>();
        t.setConnectionFactory(cf);
        return t;
    }
}
```

**Quarkus**:
```java
public class RedisProducer {
    @Produces @ApplicationScoped
    public RedisClient redisClient() { return RedisClient.create("..."); }
}
```

---

<a id="profile"></a>
### Profile / Config Profile 🟢

**定義**:依環境(dev / staging / prod)載入不同 Bean 與設定。

**Spring**:
```java
@Profile("real") @Component class RealCacheAdapter implements CachePort {}
@Profile("null") @Component class NullCacheAdapter implements CachePort {}
```
啟動時:`--spring.profiles.active=real`

**Quarkus**:
```java
@IfBuildProfile("prod") @ApplicationScoped class RealCacheAdapter {}
```
配置:`%prod.quarkus.redis.hosts=...`(profile 寫在 key 前)

**規範要求**:預設(無 Profile)要能用 Null Adapter 啟動,不需任何外部連線。

---

<a id="autoconfiguration"></a>
### AutoConfiguration / Quarkus Extension 🟡

**Spring Boot AutoConfiguration**:在 classpath 偵測到 `H2` jar 就自動配 H2 DataSource。透過 `META-INF/spring/...AutoConfiguration.imports` 註冊,大量使用 `@ConditionalOn*` annotation。

**Quarkus Extension**:類似概念但**在編譯期完成**——加上 `quarkus-redis-client` 依賴,build 時就把整合程式碼織入。所以 Quarkus 啟動快(很多事 build 時做完了)、Native 友善。

---

<a id="conditional"></a>
### @ConditionalOnMissingBean 🟡

**定義**:當 Container 內**沒有**某個 Bean 時,才建立這個。常用於 fallback / 預設實作。

**Spring**:
```java
@ConditionalOnMissingBean(CachePort.class)
@Component
public class NullCacheAdapter implements CachePort { ... }
```

**Quarkus**:用 `@DefaultBean`
```java
@DefaultBean
@ApplicationScoped
public class NullCacheAdapter implements CachePort { ... }
```

**規範用途**:Null Adapter 用此 annotation 確保「Real Adapter 存在時自動讓位」。

---

<a id="spring-mvc"></a>
### Spring MVC 🟢

**定義**:Spring 提供的 Web 框架,基於 **Servlet API**。核心是 **Front Controller 模式**——所有 HTTP request 由 `DispatcherServlet` 統一接收,再分發給對應的 Controller。

**為什麼用**:
- 把 Web 處理流程標準化(URL → Controller 方法 → View 渲染 / JSON 序列化)
- 自動處理 request 解析、response 序列化、異常處理
- 提供 `@Controller` / `@RestController` / `@RequestMapping` 等 annotation 化 API,不再寫 raw Servlet

**處理流程**:
```mermaid
flowchart LR
    Req[Request] --> DS[DispatcherServlet]
    DS --> HM[HandlerMapping<br/>找對應 Controller]
    HM --> HA[HandlerAdapter<br/>調用 Controller]
    HA --> Ctrl[Controller method]
    Ctrl --> VR{ViewResolver<br/>?}
    VR -->|@Controller| View[View 渲染 HTML]
    VR -->|@RestController| Json[直接序列化 JSON]
    View --> Resp[Response]
    Json --> Resp
```

**Spring Boot 預設整合**:`spring-boot-starter-web` 已含 Spring MVC + 內嵌 Tomcat,開箱即用。
```java
@SpringBootApplication
public class App {
    public static void main(String[] args) { SpringApplication.run(App.class, args); }
}

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() { return "Hello"; }
}
```

**Spring MVC vs Spring WebFlux**:

| | Spring MVC | Spring WebFlux |
|---|---|---|
| 模型 | Servlet(thread-per-request) | Reactive(Netty / 非阻塞) |
| 回傳型別 | `Object` / `ResponseEntity` | `Mono<T>` / `Flux<T>` |
| 阻塞 IO | 可以(thread 阻塞) | 不可以(會卡 event loop) |
| 適用 | 大多數企業應用、IO 不重 | 高並發、IO-heavy(gateway、stream) |
| 學習曲線 | 平 | 陡(要懂 Reactive Streams) |

**MVC 命名來源**:借用經典 [MVC 設計模式](./A2-design-patterns.md)——**M**odel(資料 Entity / DTO)/ **V**iew(渲染層 JSP / Thymeleaf / 直接序列化)/ **C**ontroller(處理 request、選 View)。

**與 `@RestController` 關係**:`@RestController` = `@Controller` + `@ResponseBody`,跳過 ViewResolver 直接序列化回應(REST API 用)。

---

<a id="rest-controller"></a>
### `@RestController`(獨立介紹)🟢

**定義**:Spring 4 引入的 annotation,等同 `@Controller` + `@ResponseBody` 的合體:
- `@Controller` — 把類別標記為 Spring MVC 的處理器,Bean 會被掃描註冊
- `@ResponseBody` — 方法回傳值**直接序列化成 HTTP body**(JSON / XML),而不是當成 view name 去找 template

**為什麼用**:
- **少寫**:每個方法不需重複加 `@ResponseBody`
- **語意明確**:看到 `@RestController` 就知道這個類別是 REST API,不是 server-side rendering
- **預設回 JSON**:Spring Boot 自動配 Jackson,POJO / Record 直接回傳即可序列化

**該加在哪一層**:
- **API Layer**(Clean Architecture 的最外層)
- **不要**加在 Application Layer 的 Use Case、Domain Layer 的 Domain Service、Infrastructure Layer 的 Adapter
- 一個 Controller 的職責**只有三件事**:接收請求 → 委派 Use Case → 包裝回應。**業務邏輯不寫在這裡**

**標準寫法**:
```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
@Validated
public class OrderController {
    private final OrderUseCase orderUseCase;        // 注入 Use Case 介面
    private final OrderApiMapper mapper;            // Request/Response ↔ DTO

    @GetMapping("/{id}")
    public ApiResponse<OrderResponse> get(@PathVariable @NotBlank String id) {
        var dto = orderUseCase.findById(id);
        return ApiResponse.ok(mapper.toResponse(dto));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ApiResponse<OrderResponse> create(@Valid @RequestBody CreateOrderRequest req) {
        var dto = orderUseCase.create(mapper.toCommand(req));
        return ApiResponse.ok(mapper.toResponse(dto));
    }
}
```

**配對使用的 annotation**:

| Annotation | 用途 |
| --- | --- |
| `@RequestMapping("/path")` | 類別 / 方法層級的 URL 前綴 |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@PatchMapping` / `@DeleteMapping` | HTTP method + 路徑 |
| `@PathVariable` | 路徑變數(`/orders/{id}` 的 `{id}`) |
| `@RequestParam` | Query string |
| `@RequestBody` | 請求 body 反序列化為物件 |
| `@RequestHeader` | HTTP header |
| `@Valid` | 觸發 Jakarta Bean Validation |
| `@ResponseStatus` | 自訂 HTTP 狀態碼 |

**反例**:
```java
// ❌ 業務邏輯寫在 Controller
@RestController
public class OrderController {
    private final OrderJpaRepository repo;       // ❌ 直接接 JPA
    private final PaymentClient client;

    @PostMapping("/orders/{id}/pay")
    public OrderResponse pay(@PathVariable Long id) {
        var order = repo.findById(id).orElseThrow();
        if (order.getStatus() != PENDING) throw new IllegalStateException();   // ❌ 業務規則
        var result = client.charge(order.getAmount());                          // ❌ 整合邏輯
        order.setStatus(PAID);
        return new OrderResponse(repo.save(order));
    }
}

// ✅ 把業務邏輯下放到 Use Case
@RestController
@RequiredArgsConstructor
public class OrderController {
    private final PayOrderUseCase payOrderUseCase;
    private final OrderApiMapper mapper;

    @PostMapping("/orders/{id}/pay")
    public ApiResponse<OrderResponse> pay(@PathVariable String id) {
        var dto = payOrderUseCase.execute(new PayOrderCommand(OrderId.of(id)));
        return ApiResponse.ok(mapper.toResponse(dto));
    }
}
```

**和 `@Controller` 何時用哪個**:
- 寫 REST API、回 JSON / XML → **`@RestController`**
- 寫傳統 server-side rendering(回 Thymeleaf / JSP view)→ `@Controller`
- 規範下的後端服務基本都用 `@RestController`

---

<a id="http-endpoint"></a>
### HTTP Endpoint:`@RestController` vs `@Path` 🟢

**Spring Boot**:
```java
@RestController
@RequestMapping("/orders")
@RequiredArgsConstructor
public class OrderController {
    private final OrderUseCase useCase;

    @GetMapping("/{id}")
    public OrderResponse get(@PathVariable Long id) {
        return useCase.findById(id);
    }

    @PostMapping
    public OrderResponse create(@Valid @RequestBody CreateOrderRequest req) {
        return useCase.create(req.toCommand());
    }
}
```

**Quarkus(JAX-RS)**:
```java
@Path("/orders")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class OrderResource {
    @Inject OrderUseCase useCase;

    @GET @Path("/{id}")
    public OrderResponse get(@PathParam("id") Long id) {
        return useCase.findById(id);
    }

    @POST
    public OrderResponse create(@Valid CreateOrderRequest req) {
        return useCase.create(req.toCommand());
    }
}
```

**對照表**:

| Spring | Quarkus(JAX-RS) |
| --- | --- |
| `@RestController` | `@Path` + `@Produces/Consumes` |
| `@GetMapping("/x")` | `@GET` + `@Path("/x")` |
| `@PathVariable` | `@PathParam` |
| `@RequestParam` | `@QueryParam` |
| `@RequestBody` | (參數無需 annotation) |
| `@RequestHeader` | `@HeaderParam` |

---

<a id="api-versioning"></a>
### API Versioning(API 版本管理)🟡

**為什麼需要**:API 一旦有外部 client(行動 App、合作夥伴、第三方),**改 API 等於毀約**——舊 client 不會立刻升級,你必須讓新舊版同時並存,慢慢 deprecate 舊的。

**三種主流策略**:

#### ① URI Versioning(URL 路徑放版本)— 最常見

```
GET /api/v1/orders/123
GET /api/v2/orders/123
```

**Spring 範例**:
```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/orders")
public class OrderControllerV2 { ... }
```

**優缺**:
- ✅ 直觀,瀏覽器 / curl / Postman 都看得到版本
- ✅ Routing 容易(load balancer / API Gateway 可依 path 分流)
- ❌ 嚴格來說違反 REST 哲學(同一資源應同一 URI)
- ❌ 大版號才換,小變更不適合

**用在**:Twitter、Facebook、GitHub、絕大多數公開 API。**最務實的選擇**。

#### ② Header Versioning(自訂 header)

```
GET /api/orders/123
X-API-Version: 2
```

**Spring 範例**(`headers` 屬性篩選):
```java
@GetMapping(value = "/orders/{id}", headers = "X-API-Version=1")
public OrderV1Response getV1(@PathVariable Long id) { ... }

@GetMapping(value = "/orders/{id}", headers = "X-API-Version=2")
public OrderV2Response getV2(@PathVariable Long id) { ... }
```

**優缺**:
- ✅ URI 乾淨,符合 REST
- ❌ **瀏覽器不易測試**(無法直接打網址)
- ❌ 文件不直觀

#### ③ Media Type Versioning(Content Negotiation)— REST 純粹派

```
GET /api/orders/123
Accept: application/vnd.myapp.v2+json
```

**Spring 範例**(`produces` 屬性):
```java
@GetMapping(value = "/orders/{id}", produces = "application/vnd.myapp.v1+json")
public OrderV1Response getV1(@PathVariable Long id) { ... }

@GetMapping(value = "/orders/{id}", produces = "application/vnd.myapp.v2+json")
public OrderV2Response getV2(@PathVariable Long id) { ... }
```

**優缺**:
- ✅ 最 RESTful,版本是「資源的表達形式」
- ❌ 學習曲線陡(很多前端不熟 vendor media type)
- ❌ debug 不直觀

#### 對照表

| 策略 | 範例 | 優點 | 缺點 | 推薦 |
| --- | --- | --- | --- | --- |
| URI Versioning | `/api/v2/orders` | 直觀、易 routing | 違反 REST 純粹 | ✅ 多數情境首選 |
| Header Versioning | `X-API-Version: 2` | URI 乾淨 | 難測試、難文件 | 內部 API |
| Media Type | `Accept: vnd.myapp.v2+json` | 最 RESTful | 太冷門 | 純粹 REST 信仰者 |

#### 進化策略(不只技術,還是流程)

- **Semantic Versioning**:major(破壞)/ minor(向下相容新增)/ patch(bug fix)。**只有 major 才需新版本號**。
- **Deprecation Policy**:寫進文件「v1 將於 6 個月後停用」,並用 `Deprecation` / `Sunset` HTTP header 通知。
- **向下相容原則**:**新增欄位**(client 不認得會忽略)= 不破壞;**改名 / 刪欄位 / 改 enum** = 破壞,需新版號。
- **Avoid Versioning When Possible**:能用「新增欄位 / 寬鬆 schema」解決就別開新版號——版本越多維護越痛。

#### Quarkus 範例

```java
@Path("/api/v1/orders")
public class OrderResourceV1 { ... }

@Path("/api/v2/orders")
public class OrderResourceV2 { ... }
```

JAX-RS 也支援 `@Produces("application/vnd.myapp.v2+json")` 與 `@HeaderParam` 篩選。

---

<a id="exception-handler"></a>
### @RestControllerAdvice / `@ServerExceptionMapper` 🟡

**Spring**:全域錯誤處理。
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ApiResponse<?>> on(BusinessException e) {
        return ResponseEntity.badRequest().body(ApiResponse.fail(e.code(), e.getMessage()));
    }
}
```

**Quarkus**:用 `@ServerExceptionMapper` 或實作 `ExceptionMapper<E>`。
```java
public class GlobalExceptionHandler {
    @ServerExceptionMapper
    public RestResponse<ApiResponse<?>> on(BusinessException e) {
        return RestResponse.status(BAD_REQUEST, ApiResponse.fail(e.code(), e.getMessage()));
    }
}
```

---

<a id="transactional"></a>
### @Transactional 🟡

**定義**:把方法包成資料庫交易,進入時 begin,正常結束 commit,丟出 RuntimeException 時 rollback。

**Spring**:`@Transactional`(來自 `org.springframework.transaction.annotation`)
**Quarkus**:`@Transactional`(來自 `jakarta.transaction`,標準 Jakarta EE)

**常見坑**:
- 預設只在丟出 **RuntimeException** 時 rollback,**Checked Exception 不會 rollback**(要加 `rollbackFor = Exception.class`)
- 同類別內 `this.foo()` 呼叫**不會經過 Proxy**,因此 `@Transactional` 失效
- 不要把 `@Transactional` 加在 `private` method 上(Proxy 看不到)

---

<a id="auditable"></a>
### @Auditable(自訂 annotation)🟡

**定義**:**這不是框架內建**,而是專案自訂的 marker annotation,搭配 AOP 切面實現「標記要記錄審計日誌的方法」。

**範例**:
```java
@Target(METHOD)
@Retention(RUNTIME)
public @interface Auditable {
    String action();
}

// AOP 切面攔截
@Aspect @Component
public class AuditAspect {
    @After("@annotation(auditable)")
    public void log(JoinPoint jp, Auditable auditable) {
        auditLogPort.write(auditable.action(), jp.getArgs());
    }
}
```

**為什麼這樣設計**:讓「要審計什麼」由業務層決定(在方法上標 annotation),記錄行為集中在切面,**業務程式不用碰 log**。

---

<a id="summary-table"></a>
## 核心對照表

| 概念 | Spring Boot | Quarkus | 備註 |
| --- | --- | --- | --- |
| Container | Spring `ApplicationContext` | CDI(ArC 實作) | |
| 標記 Bean | `@Component` `@Service` | `@ApplicationScoped` | |
| 注入 | `@Autowired` | `@Inject` | 兩者都建議用 constructor |
| 工廠方法 | `@Bean` in `@Configuration` | `@Produces` | |
| 條件 fallback | `@ConditionalOnMissingBean` | `@DefaultBean` | |
| 環境切換 | Spring Profile(`@Profile`) | Build/Run Profile(`@IfBuildProfile`) | Quarkus 有 build-time profile |
| 環境設定 | `application-{profile}.yml` | `%{profile}.key=value` | |
| HTTP 框架 | Spring MVC / WebFlux | JAX-RS(RESTEasy Reactive) | |
| 全域錯誤處理 | `@RestControllerAdvice` | `@ServerExceptionMapper` / `ExceptionMapper` | |
| 交易 | `@Transactional`(Spring) | `@Transactional`(Jakarta) | |
| 持久層 | Spring Data JPA / JdbcTemplate | Hibernate ORM with Panache | |
| 啟動時間 | 1~10 秒 | 1 秒以內 / Native < 50ms | Quarkus 主打優勢 |
| 記憶體用量 | 200MB+ | 100MB / Native 20MB | |
| 主要場景 | 企業級應用、生態完整 | 雲原生、Serverless、Kubernetes | |

---

← [返回索引(README.md)](./README.md)
