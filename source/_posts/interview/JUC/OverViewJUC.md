---

title: java并发编程知识点

tags: [Concurrent]

categories: [java面试]

index_img: 

banner_img: 

excerpt: 记录笔者准备的java面试学到的知识点

author: GENCO

date: 2025-1-3 13:06:00

---

## JAVA并发编程的基础篇知识:



### Excutor接口类:

#### **子接口**: 

+ [ExecutorService](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html)
+ [ScheduledExecutorService](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ScheduledExecutorService.html)

**实现类:**

+ [AbstractExecutorService](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/AbstractExecutorService.html)

+ [ForkJoinPool](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinPool.html)

+ [ScheduledThreadPoolExecutor](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ScheduledThreadPoolExecutor.html)
+  [ThreadPoolExecutor](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)



{% note primary %}

此接口不要求一定异步(在单独的线程中)执行,采用这个接口的任务也可以在调用任务的线程中直接,同步地去进行

{% endnote%}

{% fold-into @异步执行%}

```java
//不严格要求异步执行
public class LearningExecutor implements Executor {
    @Override
    public void execute(Runnable command) {
        new Thread(command).start();
    }
	/**
	*分别打印出来
	*	Main
	*	Thread-0
	*	Thread-1
	*/
    public static void main(String[] args) {
        LearningExecutor learningExecutor=new LearningExecutor();
        System.out.println(Thread.currentThread().getName()); 
        learningExecutor.execute(()->{
            System.out.println(Thread.currentThread().getName());
        });
        learningExecutor.execute(()->{
            System.out.println(Thread.currentThread().getName());
        });
    }
}

```

{% endfold %}

### 一个复合执行器代码的实现:

> 许多 `Executor` 实现对任务的调度方式和时间施加某种限制。下面的执行器将任务提交序列化到第二个执行器
>
> + **它的作用是将多个提交的任务按**顺序串行地执行

{% fold-into @实现的一个序列化提交任务到工作队列的代码%}

```java
public class SerialExecutor implements Executor {
    //提交的线程任务队列
    final Queue<Runnable> tasks=new ArrayDeque<>();
    //不允许改变的Executor
    final Executor executor;

    Runnable activie;

    public SerialExecutor(Executor executor) {
        this.executor = executor;
    }

    @Override
    public synchronized void execute(Runnable r){
        tasks.add(()->{
            try{
                r.run();
            }finally {
                scheduleNext();
            }
        });
        if(activie==null){
            scheduleNext();
        }
    }
    //检测到队列不为空的话,执行加锁线程将其添加到可运行的工作队列中
    protected synchronized void scheduleNext(){
        if((activie= tasks.poll())!=null){
            executor.execute(activie);
        }
    }
}

```

{% endfold %}  

#### 

---

#### 分析: 

{% note info %}
**下面是针对上方内容的一个分析**

{% endnote %}

{% fold-info @代码运行流程 %}

以下是代码的运行机制：

1. **初始化：**

   - `SerialExecutor` 包含一个任务队列 `tasks`，底层委托的 `executor`（用于实际执行任务），以及一个当前正在运行的任务 `active`。
   - 构造函数中传入一个底层 `Executor` 实例，任务会最终通过这个 `Executor` 执行。

2. **提交任务（`execute` 方法）：**

   - 每次调用

      

     ```
     execute
     ```

      

     方法时：

     - 新任务 `r` 会被包装为一个匿名任务，加入 `tasks` 队列。
     - 包装后的任务在运行时，会先执行原任务 `r.run()`，然后调用 `scheduleNext()` 来触发下一个任务。
     - 如果当前没有正在运行的任务（`active == null`），则调用 `scheduleNext()`，开始执行队列中的第一个任务。

3. **调度下一个任务（`scheduleNext` 方法）：**

   - 从 `tasks` 队列中取出下一个任务（`poll` 方法），将其设置为当前活跃任务 `active`。
   - 将这个任务提交给底层的 `executor`。
   - 如果队列为空，则 `active` 被置为 `null`，表示所有任务已完成。

{% endfold %}





