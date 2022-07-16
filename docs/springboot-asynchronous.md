# springboot 异步

asynchronous



Springboot@Async 异步方法

**1. 异步调用**

异步调用就是在不阻塞主线程的情况下执行高耗时方法

**2.常规异步**

通过开启新线程实现

**3.在Springboot中启用异步方法**

需要4个注解

1. @EnableAsync 开启异步
2. @Component 注册异步组件
3. @Async 标注异步方法
4. @Autowired 注入异步组件

**4.进行一次异步调用**

1. 首先在一个Config类上标注开启异步
2. 然后创建一个异步的组件类，就跟Service，Controller 一样一样的，用Component标注，Service也行
3. 在类内创建一个异步方法，打上Async 标记。这个方法必须是实例方法。
4. 然后就跟注入Service一样一样的了。

**5.异步事务**

在Async 方法上标注@Transactional是没用的。
在Async 方法调用的Service上标注@Transactional 有效。

**6.异步方法的内部调用**

异步方法不支持内部调用，也就是异步方法不能写在需要调用他的类的内部。
比如Class A 有a，b，c。b有Async标注。此时a对b的异步调用是失效的。

**7.为什么异步方法必须是实例方法**

因为static方法不能被Override。因为@Async 异步方法的实现原理是通过注入一个代理类到Bean中，这个代理继承这个Bean，需要覆写异步方法并执行。
然后这个东西，会被Spring放到自己维护的一个队列中。等待线程池读取并执行。





`@EnableAsync` 注解启用了 Spring 异步方法执行功能，在 [Spring Framework API](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableAsync.html) 中有详细介绍。

`@EnableAsync` 默认启动流程：
 1 搜索关联的线程池定义：上下文中唯一的 [`TaskExecutor`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/TaskExecutor.html) 实例，或一个名为 `taskExecutor` 的 `java.util.concurrent.Executor` 实例；
 2 如果以上都没找到，则会使用 [`SimpleAsyncTaskExecutor`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/SimpleAsyncTaskExecutor.html) 处理异步方法调用。

注意：具有 `void` 返回类型的带注释方法不能将任何异常发送回调用者，默认情况下此类未捕获异常只会被记录日志。

定制 `@EnableAsync` 启动行为：
 1 实现 [`AsyncConfigurer`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/AsyncConfigurer.html) 接口
 2 实现 [`getAsyncExecutor()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/AsyncConfigurer.html#getAsyncExecutor--) 方法自定义 `java.util.concurrent.Executor`
 3 实现 [`getAsyncUncaughtExceptionHandler()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/AsyncConfigurer.html#getAsyncUncaughtExceptionHandler--) 方法自定义 [`AsyncUncaughtExceptionHandler`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html)



 Config

需要一个注解 @EnableAsync 开启 @Async 的功能，SpringBoot 可以放在 Application 上，也可以放其他配置文件上

```java
@EnableAsync
@SpringBootApplication
public class Application { }
```

@Async 配置有两个，一个是执行的线程池，一个是异常处理

执行的线程池默认情况下找唯一的 org.springframework.core.task.TaskExecutor，或者一个 Bean 的 Name 为 taskExecutor 的 java.util.concurrent.Executor 作为执行任务的线程池。如果都没有的话，会创建 SimpleAsyncTaskExecutor 线程池来处理异步方法调用，当然 @Async 注解支持一个 String 参数，来指定一个 Bean 的 Name，类型是 Executor 或 TaskExecutor，表示使用这个指定的线程池来执行这个异步任务

异常处理，@Async 标记的方法只能是 void 或者 Future 返回值，在无返回值的异步调用中，异步处理抛出异常，默认是**SimpleAsyncUncaughtExceptionHandler** 的 **handleUncaughtException()** 会捕获指定异常，只是简单的输出了错误日志(一般需要自定义配置异常处理)，原有任务还会继续运行，直到结束(具有 void 返回类型的方法不能将任何异常发送回调用者，默认情况下此类未捕获异常只会输出错误日志)，而在有返回值的异步调用中，异步处理抛出了异常，会直接返回主线程处理，异步任务结束执行，主线程也会被异步方法中的异常中断结束执行

**@Async有两个使用的限制**

1. 它必须仅适用于 public 方法

2. 在同一个类中调用异步方法将无法正常工作(self-invocation)

   

**配置类**

```java
/**
 * AsyncConfig
 *
 * @author wliduo[i@dolyw.com]
 * @date 2020/5/19 17:58
 */
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    /**
     * logger
     */
    private static final Logger logger = LoggerFactory.getLogger(AsyncConfig.class);

    /**
     * 这里不实现了，使用 ThreadPoolConfig 里的线程池即可
     *
     * @param
     * @return java.util.concurrent.Executor
     * @throws
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/19 18:00
     */
    /*@Override
    public Executor getAsyncExecutor() {
        return null;
    }*/

    /**
     * 只能捕获无返回值的异步方法，有返回值的被主线程处理
     *
     * @param
     * @return org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler
     * @throws
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/20 10:16
     */
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new CustomAsyncExceptionHandler();
    }

    /***
     * 处理异步方法中未捕获的异常
     *
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/19 19:16
     */
    class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {

        @Override
        public void handleUncaughtException(Throwable throwable, Method method, Object... obj) {
            logger.info("Exception message - {}", throwable.getMessage());
            logger.info("Method name - {}", method.getName());
            logger.info("Parameter values - {}", Arrays.toString(obj));
            if (throwable instanceof Exception) {
                Exception exception = (Exception) throwable;
                logger.info("exception:{}", exception.getMessage());
            }
            throwable.printStackTrace();
        }
    }
}
```



**线程池**

```java
/**
 * 线程池配置
 *
 * @author wliduo
 * @date 2019/2/15 14:36
 */
@Configuration
public class ThreadPoolConfig {

    /**
     * logger
     */
    private final static Logger logger = LoggerFactory.getLogger(ThreadPoolConfig.class);

    @Value("${asyncThreadPool.corePoolSize}")
    private int corePoolSize;

    @Value("${asyncThreadPool.maxPoolSize}")
    private int maxPoolSize;

    @Value("${asyncThreadPool.queueCapacity}")
    private int queueCapacity;

    @Value("${asyncThreadPool.keepAliveSeconds}")
    private int keepAliveSeconds;

    @Value("${asyncThreadPool.awaitTerminationSeconds}")
    private int awaitTerminationSeconds;

    @Value("${asyncThreadPool.threadNamePrefix}")
    private String threadNamePrefix;

    /**
     * 线程池配置
     * @param
     * @return java.util.concurrent.Executor
     * @author wliduo
     * @date 2019/2/15 14:44
     */
    @Bean(name = "threadPoolTaskExecutor")
    public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
        logger.info("---------- 线程池开始加载 ----------");
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        // 核心线程池大小
        threadPoolTaskExecutor.setCorePoolSize(corePoolSize);
        // 最大线程数
        threadPoolTaskExecutor.setMaxPoolSize(maxPoolSize);
        // 队列容量
        threadPoolTaskExecutor.setQueueCapacity(keepAliveSeconds);
        // 活跃时间
        threadPoolTaskExecutor.setKeepAliveSeconds(queueCapacity);
        // 主线程等待子线程执行时间
        threadPoolTaskExecutor.setAwaitTerminationSeconds(awaitTerminationSeconds);
        // 线程名字前缀
        threadPoolTaskExecutor.setThreadNamePrefix(threadNamePrefix);
        // RejectedExecutionHandler:当pool已经达到max-size的时候，如何处理新任务
        // CallerRunsPolicy:不在新线程中执行任务，而是由调用者所在的线程来执行
        threadPoolTaskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // 初始化
        threadPoolTaskExecutor.initialize();
        logger.info("---------- 线程池加载完成 ----------");
        return threadPoolTaskExecutor;
    }
}
```



**异步方法调用**

```java
/**
 * SmsUtil
 *
 * @author wliduo[i@dolyw.com]
 * @date 2020/5/20 10:50
 */
@Component
public class SmsUtil {

    private static final Logger logger = LoggerFactory.getLogger(SmsUtil.class);

    /**
     * 异步发送短信
     *
     * @param phone
     * @param code
     * @return void
     * @throws
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/20 10:53
     */
    @Async
    public void sendCode(String phone, String code) {
        logger.info("开始发送验证码...");
        // 模拟调用接口发验证码的耗时
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        logger.info("发送成功: {}", phone);
    }
    
}
```

```java
/**
 * AsyncServiceImpl
 *
 * @author wliduo[i@dolyw.com]
 * @date 2020/5/19 14:24
 */
@Service
public class AsyncServiceImpl implements AsyncService {

    /**
     * logger
     */
    private final static Logger logger = LoggerFactory.getLogger(AsyncServiceImpl.class);

    @Override
    @Async("threadPoolTaskExecutor")
    public void task1() throws Exception {
        logger.info("task1开始执行");
        Thread.sleep(3000);
        logger.info("task1执行结束");
        throw new RuntimeException("出现异常");
    }

    @Override
    @Async("threadPoolTaskExecutor")
    public Future<String> task2() throws Exception {
        logger.info("task2开始执行");
        Thread.sleep(3000);
        logger.info("task2执行结束");
        throw new RuntimeException("出现异常");
        // return new AsyncResult<String>("task2 success");
    }

    @Override
    @Async("threadPoolTaskExecutor")
    public Future<String> task3() throws Exception {
        logger.info("task3开始执行");
        Thread.sleep(3000);
        logger.info("task3执行结束");
        return new AsyncResult<String>("task3 success");
    }

    @Override
    @Async("threadPoolTaskExecutor")
    public Future<String> task4() throws Exception {
        logger.info("task4开始执行");
        Thread.sleep(3000);
        logger.info("task4执行结束");
        return new AsyncResult<String>("task4 success");
    }
}
```

```java
/**
 * AsyncController
 *
 * @author wliduo[i@dolyw.com]
 * @date 2020/5/19 14:46
 */
@RestController
@RequestMapping("/async")
public class AsyncController {

    /**
     * logger
     */
    private final static Logger logger = LoggerFactory.getLogger(AsyncController.class);

    @Autowired
    private AsyncService asyncService;

    @Autowired
    private SmsUtil smsUtil;

    /**
     * 可以看到无返回值异步方法出现异常，主线程还是继续执行完成
     *
     * @param
     * @return void
     * @throws
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/20 9:53
     */
    @GetMapping("/run1")
    public String run1() throws Exception {
        asyncService.task1();
        logger.info("run1开始执行");
        Thread.sleep(5000);
        logger.info("run1执行完成");
        return "run1 success";
    }

    /**
     * 可以看到有返回值异步方法出现异常，异常抛给主线程处理，导致主线程也被中断执行
     *
     * @param
     * @return java.lang.String
     * @throws
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/20 10:15
     */
    @GetMapping("/run2")
    public String run2() throws Exception {
        Future<String> future = asyncService.task2();
        // get()方法阻塞主线程，直到执行完成
        String result = future.get();
        logger.info("run2开始执行");
        Thread.sleep(5000);
        logger.info("run2执行完成");
        return result;
    }

    /**
     * 多个异步执行
     *
     * @param
     * @return java.lang.String
     * @throws
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/20 10:26
     */
    @GetMapping("/run3")
    public String run3() throws Exception {
        logger.info("run3开始执行");
        long start = System.currentTimeMillis();
        Future<String> future3 = asyncService.task3();
        Future<String> future4 = asyncService.task4();
        // 这样与下面是一样的
        logger.info(future3.get());
        logger.info(future4.get());
        // 先判断是否执行完成
        boolean run3Done = Boolean.FALSE;
        while (true) {
            if (future3.isDone() && future4.isDone()) {
                // 执行完成
                run3Done = Boolean.TRUE;
                break;
            }
            if (future3.isCancelled() || future4.isCancelled()) {
                // 取消情况
                break;
            }
        }
        if (run3Done) {
            logger.info(future3.get());
            logger.info(future4.get());
        } else {
            // 其他异常情况
        }
        long end = System.currentTimeMillis();
        logger.info("run3执行完成，执行时间: {}", end - start);
        return "run3 success";
    }

    /**
     * 工具类异步
     *
     * @param
     * @return java.lang.String
     * @throws
     * @author wliduo[i@dolyw.com]
     * @date 2020/5/20 10:59
     */
    @GetMapping("/sms")
    public String sms() throws Exception {
        logger.info("run1开始执行");
        smsUtil.sendCode("15912347896", "135333");
        logger.info("run1执行完成");
        return "send sms success";
    }
}
```









## 异步回调



若任务本身之间不存在依赖关系，可以并发执行的话，同步调用在执行效率方面就比较差，可以考虑通过异步调用的方式来并发执行。

在Spring Boot中，我们只需要通过使用`@Async`注解就能简单的将原来的同步函数变为异步函数，如下：

```java
@Slf4j
@Component
public class AsyncTasks {

    public static Random random = new Random();

    @Async
    public void doTaskOne() throws Exception {
        log.info("开始做任务一");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        log.info("完成任务一，耗时：" + (end - start) + "毫秒");
    }

    @Async
    public void doTaskTwo() throws Exception {
        log.info("开始做任务二");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        log.info("完成任务二，耗时：" + (end - start) + "毫秒");
    }

    @Async
    public void doTaskThree() throws Exception {
        log.info("开始做任务三");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        log.info("完成任务三，耗时：" + (end - start) + "毫秒");
    }

}
```



```java
@Slf4j
@SpringBootTest
public class Chapter75ApplicationTests {

    @Autowired
    private AsyncTasks asyncTasks;

    @Test
    public void test() throws Exception {
        asyncTasks.doTaskOne();
        asyncTasks.doTaskTwo();
        asyncTasks.doTaskThree();
    }

}
```



为了让@Async注解能够生效，还需要在Spring Boot的主程序中配置@EnableAsync，如下所示：

```java
@EnableAsync
@SpringBootApplication
public class Chapter75Application {
    public static void main(String[] args) {
        SpringApplication.run(Chapter75Application.class, args);
    }
}
```

此时可以反复执行单元测试，您可能会遇到各种不同的结果，比如：

- 没有任何任务相关的输出
- 有部分任务相关的输出
- 乱序的任务相关的输出
- 原因是目前`doTaskOne`、`doTaskTwo`、`doTaskThree`三个函数的时候已经是异步执行了。主程序在异步调用之后，主程序并不会理会这三个函数是否执行完成了，由于没有其他需要执行的内容，所以程序就自动结束了，导致了不完整或是没有输出任务相关内容的情况。

> **注：`@Async`所修饰的函数不要定义为static类型，这样异步调用不会生效**



为了让`doTaskOne`、`doTaskTwo`、`doTaskThree`能正常结束，假设我们需要统计一下三个任务并发执行共耗时多少，这就需要等到上述三个函数都完成调动之后记录时间，并计算结果。

那么我们如何判断上述三个异步调用是否已经执行完成呢？我们需要使用`CompletableFuture<T>`来返回异步调用的结果，就像如下方式改造`doTaskOne`函数：

```java
@Async
public CompletableFuture<String> doTaskOne() throws Exception {
    log.info("开始做任务一");
    long start = System.currentTimeMillis();
    Thread.sleep(random.nextInt(10000));
    long end = System.currentTimeMillis();
    log.info("完成任务一，耗时：" + (end - start) + "毫秒");
    return CompletableFuture.completedFuture("任务一完成");
}
```

按照如上方式改造一下其他两个异步函数之后，下面我们改造一下测试用例，让测试在等待完成三个异步调用之后来做一些其他事情。

```java
@Test
public void test() throws Exception {
    long start = System.currentTimeMillis();

    CompletableFuture<String> task1 = asyncTasks.doTaskOne();
    CompletableFuture<String> task2 = asyncTasks.doTaskTwo();
    CompletableFuture<String> task3 = asyncTasks.doTaskThree();

    CompletableFuture.allOf(task1, task2, task3).join();

    long end = System.currentTimeMillis();

    log.info("任务全部完成，总耗时：" + (end - start) + "毫秒");
}
```

看看我们做了哪些改变：

- 在测试用例一开始记录开始时间
- 在调用三个异步函数的时候，返回`CompletableFuture<String>`类型的结果对象
- 通过`CompletableFuture.allOf(task1, task2, task3).join()`实现三个异步任务都结束之前的阻塞效果
- 三个任务都完成之后，根据结束时间 - 开始时间，计算出三个任务并发执行的总耗时。

执行一下上述的单元测试，可以看到如下结果：

```stylus
2021-09-11 23:33:38.842  INFO 95891 --- [         task-3] com.didispace.chapter75.AsyncTasks       : 开始做任务三
2021-09-11 23:33:38.842  INFO 95891 --- [         task-2] com.didispace.chapter75.AsyncTasks       : 开始做任务二
2021-09-11 23:33:38.842  INFO 95891 --- [         task-1] com.didispace.chapter75.AsyncTasks       : 开始做任务一
2021-09-11 23:33:45.155  INFO 95891 --- [         task-2] com.didispace.chapter75.AsyncTasks       : 完成任务二，耗时：6312毫秒
2021-09-11 23:33:47.308  INFO 95891 --- [         task-3] com.didispace.chapter75.AsyncTasks       : 完成任务三，耗时：8465毫秒
2021-09-11 23:33:47.403  INFO 95891 --- [         task-1] com.didispace.chapter75.AsyncTasks       : 完成任务一，耗时：8560毫秒
2021-09-11 23:33:47.404  INFO 95891 --- [           main] c.d.chapter75.Chapter75ApplicationTests  : 任务全部完成，总耗时：8590毫秒
```

可以看到，通过异步调用，让任务一、二、三并发执行，有效的减少了程序的总运行时间。



