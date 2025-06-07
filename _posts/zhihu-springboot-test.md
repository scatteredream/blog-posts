---
name: rpcboottest
title: 全链路压测优化GC参数
date: 2025-05-31
tags: 
categories: zhihu
---



在实际工作中，经常会需要进行在[全链路压测](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=全链路压测&zhida_source=entity)，优化 [GC参数](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=GC参数&zhida_source=entity)，优化 [JVM 内存分配](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=JVM+内存分配&zhida_source=entity)。

当知道 1 次 RPC 请求和 Http 请求需要的堆内存大小后，你可以精确地计算：指定的并发量之下，系统需申请多少堆内存。同时结合 JVM [新生代堆大小](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=新生代堆大小&zhida_source=entity)，就能推算出 1 分钟发生多少次 GC，这个 [GC频率](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=GC频率&zhida_source=entity)是否过于频繁？从而针对性的优化。

我们希望 1 次 Rpc、Http 请求申请堆内存足够少，这样可减少 GC 导致的系统停顿，提高系统性能，单机可以支撑更高的并发量。

1次 Http 请求，申请多少堆内存？

1 次 RPC 请求，申请多少堆内存？

如果不亲自实验，无法得出结论。

## **1. 实验思路**

### **关键动作**

1. 创建[SpringBoot](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=SpringBoot&zhida_source=entity)新应用(版本`2.5.4`)。
2. 新增 Post 接口，供 [JMeter](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=JMeter&zhida_source=entity) 调用。
3. JMeter（开源压测工具）新建测试计划。每个线程执行2000 次Http接口调用，共10 个线程，总调用 20000 次。
4. SpringBoot 打印 GC 详细日志，记录GC 前后，新生代申请了多少内存。

Jmeter 调用 20000 次 Http 接口以后，通过手动 GC 的方式触发 GC，通过 GC 详细日志计算压测期间新生代堆内存增长量。（对象基本分配在新生代）

## **2. SpringBoot 声明 Http 接口**

如下代码声明了一个 Post接口 create；创建了 Get 接口，用于触发GC。

```java
@Slf4j
@RestController
public class TestController {

   private AtomicLong count = new AtomicLong(0);

   @ResponseBody
   @RequestMapping(value = "create", method = RequestMethod.POST)
   public String create(@RequestBody Order order) {
      //log.warn("收到提单请求 cnt{}:{}", count.getAndIncrement(), order);
      return "ok";
   }

   @ResponseBody
   @RequestMapping(value = "gc", method = RequestMethod.GET)
   public String gc() {
      System.gc();
      return "ok";
   }
}
```

## **3. JMeter 新建测试计划**

### **3.1 新增线程组**

新建线程组，选择 10 个线程，每个线程循环 2000次。



![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-5d9d9fa448616508fa7a9c56f813e893_1440w.webp)



![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-ae94445a6a75a61e820a52e65df1dc54_1440w.webp)

### **3.2 新建 Http 默认值**

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-64ba1f2c3a1eca7d2fa8a8bc63f210ad_1440w.webp)

### **3.3 新建请求头**

由于请求体是 JSON，所以新增请求头 Content-Type



![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-7510127e1f61c1f513e287c5907dec04_1440w.webp)

### **3.4 新建 Http 请求**

请求中指定 Url 和 请求体



![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-d6686ab8e643e3965a9090179e126511_1440w.webp)

## **4. 实验过程**

### **4.1 启动 SpringBoot应用**

堆内存大小 4G，其中新生代内存 2G。`SurivivorRadio=8`，即每个 Surivivor 占比新生代 1/10。

指定GC日志位置：`-Xloggc:/Users/testUser/log/gc.log`

```text
 java -server 
 -Xmx4g -Xms4g -XX:SurvivorRatio=8 -Xmn2g
 -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=1g -XX:MaxDirectMemorySize=1g 
 -XX:+HeapDumpOnOutOfMemoryError -XX:+PrintGCCause -XX:+PrintGCDetails 
 -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution 
 -XX:+UnlockDiagnosticVMOptions -XX:ParGCCardsPerStrideChunk=32768 
 -XX:+PrintCommandLineFlags -XX:+UseConcMarkSweepGC -XX:+UseParNewGC 
 -XX:ParallelCMSThreads=6 -XX:+CMSClassUnloadingEnabled 
 -XX:+UseCMSCompactAtFullCollection -XX:+CMSParallelInitialMarkEnabled 
 -XX:+CMSParallelRemarkEnabled -XX:+CMSScavengeBeforeRemark -XX:+PrintHeapAtGC 
 -XX:CMSFullGCsBeforeCompaction=1 -XX:CMSInitiatingOccupancyFraction=70 
 -XX:+UseCMSInitiatingOccupancyOnly -XX:+PrintReferenceGC  
 -XX:+ParallelRefProcEnabled -XX:ReservedCodeCacheSize=256M 
 -Xloggc:/Users/testUser/log/gc.log
 -jar target/activiti-0.0.1-SNAPSHOT.jar
```

### **4.2 多次手动 GC**

由于JVM 启动过程中，需要加载大量对象，所以我们在压测之前先手动 GC，清理一下存量对象。

```text
curl http://localhost:8080/gc
```

### **4.3 Jmeter 启动压测**

执行 JMeter压测计划，每次执行会调用 20000 次，

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-01e19093ebe98a61a006882fd7ab05d2_1440w.webp)

### **4.4 GC 日志解读**

GC 以后，新生代 Eden 区已使用内存为 0。GC 前 Eden区大小就是 20000 次 Http 调用所申请的内存总和！



![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-184e3d1df97081f9ae9b757c14d7b333_1440w.webp)

## **5. 实验结果**

SpringBoot 在处理 Http 请求时，即使请求体相对较小，**平均每次 Http 调用仍会申请约 34 K 的堆内存**。这一点显得尤为突出，因为请求体仅包含 50 个字符，远远未达到 1K 大小。

然而，每次 Http 请求所消耗的内存却依然高达 34K。这可能是由于在 SpringBoot 的内部处理流程中需要创建多个对象，这些对象的总内存占用显著高于请求体本身。

```text
{"userId": 32898493, "productId":39043, "detail": ""}
```

在调整 Http 请求后，如将 detail 字段设置为 1200 个字符时，每次 Http 调用平均占用堆内存为 36K。两次实验结果间的差异为 2K，这与 1200 个字符占用的内存大小基本持平（需考虑一定的误差）。

**这表明 SpringBoot 内部未进行多次请求体拷贝。**

### **5.1 添加日志打印**

```text
log.warn("收到提单请求 cnt{}:{}", count.getAndIncrement(), order);
```

在打印请求日志后，单次 Http 请求的平均内存使用量达到了 56 KB，比之前增加了整整 20 KB。

然而，当我移除 detail 字段后，单次请求的内存使用骤降至 35.7 KB。

这表明，当日志量较小时，打印日志对内存占用的影响较小。但随着日志大小的增加，内存占用显著上升，这可能触发更频繁地GC，最终导致系统性能明显下降。

因此，建议各位严格控制单条日志的大小，以优化内存使用和系统性能。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-3602e22b4df0e789f597d9d5400defb9_1440w.webp)

**6. 真实的数据**

根据以上实验结果可得出结论：单次 HTTP 请求消耗约34KB内存，这并不意味着所有SpringBoot应用的内存消耗都是如此。由于实验所用的代码相对简单，因此34KB可能是内存消耗的最小值。

举例来说，在我司的线上环境中，**单次RPC请求的内存消耗在 0.5MB 到 1MB 之间**，内存占用量相对较大。

这是因为复杂的基础架构、复杂的业务逻辑、复杂的流程、多次下游调用、多次SQL 调用、多次缓存调用、日志打印等等 均需要消耗大量的内存！

在此之前，我一直对新生代 Eden 区高达 5G 的情况下，仍每分钟进行 1-2 次 [young GC](https://zhida.zhihu.com/search?content_id=693946600&content_type=Answer&match_order=1&q=young+GC&zhida_source=entity) 感到困惑。

经过粗略计算后发现，如果每次请求消耗 0.5M 内存，当单台服务器每秒并发度达到 500 次时，每分钟需要分配的内存高达 15G。因此，至少需要进行3次 young GC 才能满足需求。