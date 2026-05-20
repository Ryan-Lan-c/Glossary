# B9 - 排程與批次處理(Scheduling & Batch Processing)

← [返回索引(README.md)](./README.md)

---

> 本章涵蓋三個關聯但獨立的關注點:
> - **排程**(Scheduling)— **什麼時候**跑某個任務
> - **批次處理**(Batch Processing)— 怎麼處理**大量資料**的批次任務
> - **非同步執行**(Async Execution)— 怎麼讓任務**在背景跑**、不阻塞主執行緒

## 目錄

### 排程(什麼時候跑)
- [`@Scheduled` Annotation(Spring & Quarkus)🟢](#scheduled)
- [Quartz 🟡](#quartz)

### 批次處理(怎麼跑大量資料)
- [Spring Batch 🔴](#spring-batch)
- [Spring Cloud Task 🟡](#spring-cloud-task)

### 非同步執行(背景跑)
- [`@Async` 🟡](#async)
- [Thread Pool / Executor 🟡](#thread-pool)

### 三者分工
- [排程 vs 批次 vs 非同步 🟡](#three-compare)

---

## 排程

<a id="scheduled"></a>
### `@Scheduled` Annotation(Spring & Quarkus)🟢

**定義**:Spring 與 Quarkus 都提供 `@Scheduled` annotation,讓你**用 declarative 方式**在方法上標記「這個方法要被定期執行」,框架自動處理排程細節。

**為什麼用**:
- 不必寫 `Thread.sleep()` 或自己管理排程
- 不必引入 [Quartz](#quartz) 這種重量級框架(80% 場景夠用)
- 與 IoC / DI 整合(`@Scheduled` 方法所在的 Bean 仍能 DI)

**Spring 範例**:
```java
@SpringBootApplication
@EnableScheduling   // ⚠ 要記得開啟
public class App { }

@Component
public class ReportJob {

    // cron expression(每天 03:00)
    @Scheduled(cron = "0 0 3 * * ?")
    public void generateDailyReport() { /* ... */ }

    // 固定間隔(每 5 秒,從**前次完成**算)
    @Scheduled(fixedDelay = 5000)
    public void poll() { /* ... */ }

    // 固定頻率(每 5 秒,從**前次開始**算)
    @Scheduled(fixedRate = 5000)
    public void heartbeat() { /* ... */ }
}
```

**Quarkus 範例**(同樣的 annotation 名稱,但是 `io.quarkus.scheduler.Scheduled`):
```java
@ApplicationScoped
public class ReportJob {

    @Scheduled(cron = "0 0 3 * * ?")
    void generateDailyReport() { /* ... */ }

    @Scheduled(every = "5s")
    void poll() { /* ... */ }
}
```

**`fixedDelay` vs `fixedRate` 的差異**:

| | `fixedDelay` | `fixedRate` |
|---|---|---|
| 計時起點 | **前次方法結束** | **前次方法開始** |
| 任務久時 | 自然延後 | **可能重疊**(下一次該開始時前次還沒結束)|
| 適用 | poll、cleanup | heartbeat、metrics |

**限制**(超出時請改用 [Quartz](#quartz)):
- 沒有 **持久化排程**(Server 重啟,排程歷程消失)
- 沒有 **叢集協調**(多台 Server 都會跑同一個排程,造成重複)
- 沒有 **動態調整**(改 cron 要重新部署)
- 沒有 **失敗重試 / 歷程記錄**

---

<a id="quartz"></a>
### Quartz 🟡

**定義**:Java 生態最成熟的**企業級排程框架**(自 2001 年,現由 Terracotta / Software AG 維護)。提供持久化排程、叢集協調、複雜觸發規則、JobStore 等 [`@Scheduled`](#scheduled) 沒有的能力。

**為什麼用**(都是 `@Scheduled` 做不到的):
- **JobStore**:把排程資訊存進 DB(`QRTZ_*` 系列表),重啟不消失
- **叢集模式**:多台 Server 共用同一個 JobStore,**只有一台執行該次任務**(無重複)
- **複雜 Trigger**(CronTrigger / SimpleTrigger / CalendarIntervalTrigger / DailyTimeIntervalTrigger)
- **失敗處理**(misfire instruction、retry policy)
- **動態管理**:程式可以增 / 刪 / 改 Job

**核心概念**:

| 概念 | 角色 |
|---|---|
| **Job** | 要執行的任務(實作 `org.quartz.Job` 介面)|
| **JobDetail** | Job 的設定與識別(name、group、parameters)|
| **Trigger** | 何時觸發(CronTrigger / SimpleTrigger / ...)|
| **Scheduler** | 排程器本身,管理所有 Job + Trigger |
| **JobStore** | 排程資料儲存(RAM-based / **JDBC-based**)|
| **Calendar** | 排除特定日期(假日、週末)|

**Spring Boot 整合**:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

```java
public class ReportJob implements Job {
    @Override
    public void execute(JobExecutionContext context) {
        // 業務邏輯
    }
}

@Configuration
public class QuartzConfig {
    @Bean
    public JobDetail reportJobDetail() {
        return JobBuilder.newJob(ReportJob.class)
            .withIdentity("reportJob")
            .storeDurably()
            .build();
    }

    @Bean
    public Trigger reportTrigger(JobDetail reportJobDetail) {
        return TriggerBuilder.newTrigger()
            .forJob(reportJobDetail)
            .withSchedule(CronScheduleBuilder.cronSchedule("0 0 3 * * ?"))
            .build();
    }
}
```

**`@Scheduled` vs Quartz 對照**:

| 維度 | `@Scheduled`(Spring / Quarkus) | Quartz |
|---|---|---|
| 啟用方式 | annotation | XML / `JobDetail` Builder |
| 持久化(重啟保留) | ❌ | ✅ JobStore JDBC |
| 叢集協調 | ❌ | ✅(多 server 同一 Job 只跑一次) |
| 動態增/刪 Job | ❌ | ✅ |
| Trigger 種類 | cron / fixedRate / fixedDelay | 6 種 + 自訂 |
| 學習成本 | 平 | 中 |
| 適用 | 80% 一般場景 | 多 server、需可靠執行、複雜規則 |

**選型口訣**:**單機簡單排程 → `@Scheduled`;多機 / 需持久化 / 需動態管理 → Quartz**。

---

## 批次處理

<a id="spring-batch"></a>
### Spring Batch 🔴

**定義**:Spring 官方的**批次處理框架**——處理「**大量資料的離線任務**」(ETL、月底結帳、報表生成、資料遷移、千萬筆資料清理)。核心抽象:**Job 由多個 Step 組成,每個 Step 是 Read → Process → Write 的流程**。

**為什麼用**:
- 處理超大資料量(**Chunk-oriented processing**,一次處理一批,避免 OOM)
- **可重啟**(failure 後從中斷處繼續,不必從頭)
- **歷程追蹤**(每次 Job 執行存進 DB 的 `BATCH_*` 表)
- 並行處理(同步多 Step / Parallel Step / Partitioning)
- 與 [Quartz](#quartz) 搭配:Quartz 排程觸發 → 跑 Spring Batch Job

**核心概念**:

| 概念 | 角色 |
|---|---|
| **Job** | 批次任務最大單位(例如「每月結帳」)|
| **Step** | Job 的一個步驟(例如「讀帳單 → 計算費用 → 寫入帳單表」)|
| **ItemReader** | 讀資料(file / DB / queue / API)|
| **ItemProcessor** | 處理 / 轉換 / 過濾(可選)|
| **ItemWriter** | 寫資料(file / DB / queue)|
| **Chunk** | 一批的大小(讀 N 筆 → 處理 → 寫一次,而非逐筆寫)|
| **JobRepository** | 執行歷程儲存(DB 中的 `BATCH_*` 表)|
| **JobLauncher** | 觸發 Job 執行的入口 |

**範例**(讀 CSV → 處理 → 寫入 DB):
```java
@Configuration
@EnableBatchProcessing
public class BillingJobConfig {

    @Bean
    public Job billingJob(JobRepository repo, Step calcStep) {
        return new JobBuilder("billingJob", repo)
            .start(calcStep)
            .build();
    }

    @Bean
    public Step calcStep(JobRepository repo, PlatformTransactionManager tx,
                         ItemReader<Bill> reader,
                         ItemProcessor<Bill, Invoice> processor,
                         ItemWriter<Invoice> writer) {
        return new StepBuilder("calcStep", repo)
            .<Bill, Invoice>chunk(1000, tx)        // 一批 1000 筆
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
    }
}
```

**Chunk-oriented vs Tasklet-oriented**:
- **Chunk**:讀 N 筆 → 處理 → 寫 → commit,重複(大量資料的標準作法)
- **Tasklet**:一個 Step 就執行一個自訂操作(適合 cleanup、archive 等「一次性」步驟)

**與 Quartz 的常見組合**:
```
Quartz Scheduler ──(每天 03:00 觸發)──> JobLauncher.run(billingJob)
                                              ↓
                                         Spring Batch 跑 Job
```

**反模式**:**只有幾筆資料就上 Spring Batch** —— 殺雞用牛刀,簡單寫個 [`@Scheduled`](#scheduled) + Repository 操作就好。Spring Batch 的價值在「**幾十萬 ~ 千萬筆**」級別。

---

<a id="spring-cloud-task"></a>
### Spring Cloud Task 🟡

**定義**:Spring 官方的「**短生命週期(short-lived)微服務批次**」框架——適合「**跑一次、結束、釋放資源**」的場景(Kubernetes Job、Cloud Run、Lambda 風格)。

**為什麼用**(對比 Spring Batch):
- **[Spring Batch](#spring-batch)**:**長時間執行的大型批次**(Server 常駐,Job 啟動時還在跑)
- **Spring Cloud Task**:**短時間、一次性的任務**(任務跑完就退出,釋放資源,適合 cloud-native)
- 整合 K8s Job、Spring Cloud Data Flow,**容器化 / 排程化部署**自然
- 記錄執行歷程(`TASK_EXECUTION` 表),類似 Spring Batch 但更輕量

**範例**:
```java
@SpringBootApplication
@EnableTask
public class DataMigrationApp {

    @Bean
    public ApplicationRunner runner() {
        return args -> {
            // 跑一次性任務 — 完成後 JVM 退出
            log.info("Running migration...");
            // ...
        };
    }
}
```

**Spring Batch vs Spring Cloud Task 對照**:

| 維度 | Spring Batch | Spring Cloud Task |
|---|---|---|
| 設計目的 | 大型批次處理(Read-Process-Write) | 短生命週期任務(跑完即停) |
| 執行模型 | 長時間執行 | 短時間執行 |
| 適合部署 | 常駐 Server | K8s Job / Lambda / Cloud Run |
| 提供結構 | Job / Step / Reader-Processor-Writer | 一個 ApplicationRunner / Job |
| 歷程記錄 | `BATCH_*` 表(JobRepository)| `TASK_EXECUTION` 表 |
| 跟 Spring Batch 關係 | — | **可包裝 Spring Batch Job 變成 Task** |
| 適用 | 千萬筆資料、可重啟、複雜流程 | ETL one-shot、容器化批次 |

**常見組合**:**Spring Cloud Task 包裝 Spring Batch Job → 部署成 K8s CronJob**,享受兩者優點。

---

## 非同步執行

<a id="async"></a>
### `@Async` 🟡

**定義**:Spring 提供的 annotation,標記一個方法**在背景執行緒中執行**——呼叫方不會等待,可以立刻繼續。

**為什麼用**:
- 邏輯上「**不需要等結果**」的事(寄 email、寫 log、推送通知)
- 簡單的 fire-and-forget 模式,不必自己管 thread
- 與 IoC / DI 整合
- 比手動 `new Thread().start()` 乾淨

**範例**:
```java
@SpringBootApplication
@EnableAsync   // ⚠ 要記得開啟
public class App { }

@Service
public class NotificationService {

    @Async
    public void sendEmail(String to, String body) {
        // 在背景執行(立刻 return 給呼叫方)
    }

    @Async
    public CompletableFuture<Report> generate(String userId) {
        // 想要結果回傳就回 Future / CompletableFuture
        var report = doHeavyWork(userId);
        return CompletableFuture.completedFuture(report);
    }
}
```

**注意陷阱**:
- **`@Async` 內部呼叫無效**:同一個 Bean 內部呼叫自己的 `@Async` 方法**不會走 proxy**(會直接呼叫,沒有非同步效果)。要透過注入自己或拆出另一個 Bean
- **沒指定 Executor 用預設 `SimpleAsyncTaskExecutor`**:每次都建新 thread,**production 一定要指定 Thread Pool**(見下方)
- **異常處理**:`@Async` 方法的 exception 不會丟回呼叫方,需要 `AsyncUncaughtExceptionHandler`

**指定 Executor**:
```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean(name = "notifExecutor")
    public Executor notifExecutor() {
        var ex = new ThreadPoolTaskExecutor();
        ex.setCorePoolSize(5);
        ex.setMaxPoolSize(20);
        ex.setQueueCapacity(100);
        ex.setThreadNamePrefix("notif-");
        return ex;
    }
}

@Async("notifExecutor")   // 指定用哪個 Executor
public void sendEmail(/* ... */) { /* ... */ }
```

---

<a id="thread-pool"></a>
### Thread Pool / Executor 🟡

**定義**:Java 並行的**基礎抽象**(`java.util.concurrent`)——預先建好一群 thread 待命,任務丟進 queue,thread 從 queue 取任務執行。避免每次 `new Thread()` 的開銷,以及失控建立 thread 造成 OOM。

**為什麼用**:
- 限制併發數(避免無限建 thread 把 OS 壓垮)
- 重用 thread(減少建立/銷毀成本)
- 提供 backpressure(queue 滿了會 reject / block)
- 是上層 [`@Async`](#async)、[Spring Batch](#spring-batch) parallel、Reactive 的基礎

**核心 API**(`ExecutorService`):

| 工廠 | 行為 | 適用 |
|---|---|---|
| `Executors.newFixedThreadPool(n)` | 固定 n 個 thread | 大多數場景 |
| `Executors.newSingleThreadExecutor()` | 單一 thread,任務序列化 | 需順序保證 |
| `Executors.newCachedThreadPool()` | thread 數動態,閒置 60s 回收 | 短任務、burst pattern |
| `Executors.newScheduledThreadPool(n)` | 排程版 | 替代 `Timer` |

**範例**:
```java
ExecutorService pool = Executors.newFixedThreadPool(10);

Future<String> f = pool.submit(() -> "result");
String result = f.get();      // 阻塞等結果

pool.shutdown();              // 不再接新任務(等已提交的跑完)
pool.shutdownNow();           // 嘗試中斷已在跑的
```

**直接用 `ThreadPoolExecutor`(更可控)**:
```java
var executor = new ThreadPoolExecutor(
    5,                                          // core pool size
    20,                                         // max pool size
    60, TimeUnit.SECONDS,                       // 閒置 thread 存活時間
    new LinkedBlockingQueue<>(100),             // 工作 queue
    new ThreadPoolExecutor.CallerRunsPolicy()   // 飽和策略
);
```

**飽和策略**(Queue 滿且達 max 時):

| 策略 | 行為 |
|---|---|
| `AbortPolicy`(預設) | 拋 `RejectedExecutionException` |
| `CallerRunsPolicy` | **呼叫者 thread 自己跑**(隱含 backpressure)|
| `DiscardPolicy` | 默默丟棄 |
| `DiscardOldestPolicy` | 丟棄 queue 最舊的,再 enqueue 新的 |

**反模式**:
- `Executors.newCachedThreadPool()` + 大量任務 → **無上限建 thread**,OOM 風險(推薦用 `ThreadPoolExecutor` 直接控制)
- 沒呼叫 `shutdown()` → JVM 不會退出(non-daemon thread)
- 在 web request thread 中用全域 pool 跑長任務 → 互相影響(應分開 pool)

**與虛擬執行緒(Java 21)的關係**:Java 21 引入 **Virtual Threads(Project Loom)**,讓「為 IO 用 Thread Pool」變得不必要——`Executors.newVirtualThreadPerTaskExecutor()` 可開萬計 virtual threads。CPU-bound 任務仍需傳統 platform thread pool。

---

## 三者分工

<a id="three-compare"></a>
### 排程 vs 批次 vs 非同步 🟡

**核心差異一句話**:
- **排程** = 解決「**什麼時候跑**」
- **批次** = 解決「**怎麼跑大量資料**」
- **非同步** = 解決「**怎麼讓任務在背景跑、不阻塞**」

**完整對照表**:

| 維度 | 排程(Scheduling)| 批次(Batch)| 非同步(Async)|
|---|---|---|---|
| 核心問題 | When | What & How | Where(背景) |
| 觸發方式 | 時間驅動(cron / interval)| 事件 / 手動 / 排程觸發 | API 呼叫立即觸發 |
| 執行時長 | 短(觸發完即結束)| 長(數分鐘至數小時)| 中短(背景任務)|
| 資料量 | 不關心 | 大(千萬筆)| 不關心 |
| 持久化 | Quartz 有,`@Scheduled` 無 | ✅ Spring Batch JobRepository | 通常無(訊息佇列才有)|
| 主要工具 | [`@Scheduled`](#scheduled)、[Quartz](#quartz) | [Spring Batch](#spring-batch)、[Spring Cloud Task](#spring-cloud-task) | [`@Async`](#async)、[Thread Pool](#thread-pool)、`CompletableFuture` |
| 典型場景 | 每天清快取、定時 polling | 月底結帳、ETL、報表 | 寄 email、寫 log |

**常見組合**:
1. **排程 + 批次**:Quartz 排程觸發 → Spring Batch Job 執行(經典 ETL 模式)
2. **批次 + 非同步**:Spring Batch 內 Step parallel → 每個 partition 用 ThreadPool 並行
3. **排程 + 非同步**:`@Scheduled` 觸發後 `@Async` 在背景跑(避免阻塞 scheduler thread)

**選型決策樹**:
```
要定時跑?─ Yes ─ 多機/需持久化?─ Yes ─> Quartz
                                 └ No ──> @Scheduled

要處理大量資料?─ Yes ─ 短生命週期/容器化?─ Yes ─> Spring Cloud Task
                                            └ No ──> Spring Batch

只是不想阻塞?─ Yes ─ 簡單 fire-and-forget?─ Yes ─> @Async
                                            └ No ──> 直接用 ThreadPoolExecutor
```

---

← [返回索引(README.md)](./README.md)
