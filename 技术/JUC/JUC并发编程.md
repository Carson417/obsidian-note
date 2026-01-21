JUC是 `java.util.concurrent` 包的缩写，就是并发场景进行多线程编程的工具类

# 前置知识
## 1 进程、线程、协程
### 1.1 什么是进程
- os中，进程是基本的资源分配单位，OS通过进程管理计算机资源
	- 每个进程都有唯一的进程标识符（PID）
> 通俗的说，进程可看作正在执行的程序

### 1.2 什么是线程
- 线程是OS中的基本执行单元（能执行的最小代码块），是进程中的一个实体，是CPU调度和分配的基本单位
- 一个进程可以包含多个线程，每个线程都可以独立执行不同任务，但它们共享进程的资源
- 同一时刻，一个CPU核心只能运行一个线程
> 进程之间不共享全局变量，线程之间共享全局变量

### 1.3 什么是协程
- 可以在一个线程内部创建多个协程，这些协程之间可以共享同一个线程的资源
- 协程是在同一个进程内部运行的，***不需要OS介入***，可以在用户空间内实现协作式多任务处理。因此协程的创建和销毁开销很小
> Java19才支持协程（虚拟线程），或者第三方协程库quasar


---


## 2 并发 并行 串行
### 2.1 并发 Concurrent
- 在OS中，并发是在**同一时间段内**宏观上有多个程序同时运行，在单CPU系统中，每一时刻只能有一道程序执行，即微观上这些程序是分时的交替运行，只不过是给人的感觉是同时运行，那时因为分时交替的时间很短

### 2.2 并行 Parallel
- 在多核CPU系统中，这些**同一时刻**的程序可以分配到多个处理器上(CPU)，实现多任务并行执行，即利用每个处理器来处理一个可以并发执行的程序，这样多个程序可以同时执行

### 2.3 串行
- 单核CPU，同一时刻只能运行一个程序，如果存在多个程序，需要按照先后顺序执行，线程也是一次只能执行一个线程代码指令，其他线程需排队等候


---


## 3 CPU核心数和线程数的关系
> 线程是CPU调度的最小单位

同一时刻，一个CPU只能运行一个线程
但 Intel 引入超线程技术后，产生逻辑处理器的概念，**CPU核心数与线程数 1：2**

```java
// 获取当前CPU核心数 --- 是逻辑处理器数
Runtime.getRuntime().availableProcessors()
```

> [!合理设置线程数]
    > - 高并发，低耗时：少线程
    > - 低并发，高耗时：多线程
    > - 高并发高耗时：分析任务类型，增加排队，加大线程数


---


## 4 上下文切换 Context switch
- 提高并发并不意味着启动更多线程执行，更多线程意味着线程创建销毁开销加大，上下文切换频繁（线程调度），反而不能支持更高的TPS
> TPS：每秒事务数

### 4.1 时间片
一个CPU同时只能执行一项任务，让用户感觉这些任务在同时进行 ---时间片轮转
- ***时间片是CPU分配给各个任务（线程）的时间***

线程上下文指某一时间点CPU寄存器和PC的内容，CPU通过时间片分配算法循环执行任务（线程），时间片非常短，CPU不停的切换线程执行
- 单CPU这么频繁，多核CPU一定程度可以减少上下文切换


---


# 线程
## 1 创建线程
1. 通过继承 `Thread`，并重写 run 方法
2. 通过实现 `Runnable` 接口的 run 方法（推荐）
3. 使用 FutureTask 方式（实现 `Callable` 接口）

启动线程：`.start()`

### 1.1 Callable 接口
一般情况使用Runnable、Thread实现的线程都是无法返回结果的
- Callable 只能在 ExecutorService 的线程池中跑，有返回结果；也可以通过返回的 Future 对象查询执行状态
- Future 本身也是一种设计模式，用来取得异步任务的结果

只有一个 call 方法，并有一个返回V，是泛型。这里的 V 就是线程返回的结果
```java
public interface Callable<V> {
	V call() throws Exception;
}
```

> [!NOTE]
    > thread 的构造函数只能接收 runnbale 对象，不能接收 callable 对象
    > 所以需要 future ”中间人‘

***使用 FutureTask 类包装 Callable 对象，thread 构造函数可接收 future对象，因为FutureTask 实现了 Runnable 接口***
```java
FutureTask<Integer> future = new FutureTask<Integer>(()->{
	log.info();
	return 5;
})

new Thread(future).start();

try{
	// 调用 FutureTask对象的方法获取子线程执行结束后的返回值
	Integer value = future.get();
}
```

### 1.2 三种线程创建方式的区别
Java中类只支持单继承，如果一个类继承了 Thread 类，就无法在继承其他类；实现Callable接口的方式，可以获取到线程执行的返回值，是否执行完成等信息


## 2 线程原理
- 会调用 `native` 本地方法去执行C/C++的代码

![[Pasted image 20251126164241.png]]
<center>java线程创建调用关系图</center>

- 这个 `run()` 方法，就是我们重写的 `run()` 方法

- 继承Thread，重写Thread的run方法，启动过程：`thread.start()` -> 中间过程 -> `thread.run()`
- 实例化Thread，传递一个Runnable任务，启动过程：`thread.start()` -> 中间过程 -> `thread.run()` -> `runable.run()`

> 注意两个`thread.run()` 并不相同
> - 第一处的run我们已经重写，是真正的业务逻辑
> - 第二处的run是Thread类里默认的默认逻辑，会调用 `runable.run()`，业务逻辑在 `runable.run()` 里


## 3 线程的常用方法
### 3.1 start
`public void start()`
功能：启动一个新线程，JVM调用此线程的run方法
说明：start方法<u>只是让线程进入就绪</u>，里面的代码不一定立即运行（CPU未分配给它）。***每个线程对象的start方法只能调用一次***

### 3.2 run
`public void run()`
功能：线程启动后调用该方法
说明：如果在构造Thread对象时传递了Runnable参数，则线程启动后会调用Runnable中的run方法，否则<u>默认不执行任何操作</u>。但可以<u>创建Thread的子类</u>对象，来覆盖默认行为

### 3.3 setName 和 getName
给线程取名与获取
线程的默认名称：主线程是***main***，子线程是***Thread-索引***

### 3.4 currentThread
获取当前线程对象，代码在哪个线程中执行

### 3.5 sleep
让当前线程休眠多少毫秒再继续执行，更多是用来测试
- `Thread.sleep(0)` ：让OS立刻重新执行一次CPU竞争
- 其他线程可以用 `interrupt` 方法打断正在睡眠的线程，这是 sleep 会抛出 InterruptedException
- 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep，其底层还是 sleep 方法
```JAVA
TimeUnit.SECONDS.sleep(1);
```
- 我们随便写个小项目，执行结束就停止了，但是像tomcat这样的服务器并不能停止，我们的项目想一直循环，很简单加一个死循环，但是这会导致CPU使用率巨高，那么我们加上个 sleep，CPU就降下来了
	- SpringBoot内嵌的tomcat就会创建一个持续的阻塞线程

### 3.6 yield（native）
***提示***线程调度器让出当前线程对CPU的使用
> 并不能保证线程一定让出CPU，只是提示，告诉调度器当前线程愿意让出CPU
- `Thread.yield()` ：暂停当前正在执行的线程对象（及放弃当前拥有的CPU资源），并执行其他线程
- yield 做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会   ---目的是让相同优先级的线程间能适当的轮转执行
	- 但实际中，无法保证达到让步的目的，因让步的线程还可能被调度器选中

### 3.7 getPriority 和 setPriority
- set 的参数  int 类型，常用 1，5，10
- 优先级是1~10的整数，越大越有几率被CPU调度

### 3.8 interrupt
`public void interrupt()`：中断整个线程，异常处理机制
`public static boolean interrupted()` ：判断当前线程是否被打断，清除打断标记（撤销打断）
`public boolean isInterrupted()` ：判断当前线程是否被打断，不清除打断标记

#### 3.8.1 线程打断
- 实例方法 interrupt() 仅仅是***设置线程的中断状态为 true，不会停止线程***
- isInterrupted() 通过检查中断标志位，判断当前线程是否被中断
- 静态方法 interrupted() 判断线程是否被中断，并清除当前中断状态
	- 做了2件事
		1. 返回当前线程的中断状态
		2. 将当前线程的中断状态设置为 false
```java
while(true){
	if(Thread.interrupted()){
		break;
	}
}
```

之前说过，死循环CPU占用率会很高，那么sleep，如果在睡眠时中断，就会捕捉到异常，正常来说继续执行就会退出，但是实际上并未退出
- 走到catch说明睡眠时进行中断，***InterruptedException，会清除中断标记***
- 所以要再加一下中断标记
```java
try{
	Thread.sleep(1000);
} catch(InterruptedException e){
	e.printStackTrace();
	Thread.currentThread().interrupt();
}
```

### 3.9 join
子线程join，主线程会等子线程结束再执行主线程
```java
class test{
	static int value = 1；
	main(){
		Thread t1 = new Thread(()->{
			value = 10;
		});
	}
	t1.start();
	sout("主线程" + value);
}
```
输出的值是1，因为开启线程后，主线程会直接执行下行代码，不会等子线程

```java
class test{
	static int value = 1；
	main(){
		Thread t1 = new Thread(()->{
			value = 10;
		});
	}
	t1.start();
	t1.join();
	// 输出10
	sout("主线程" + value);
}
```
 
### 3.10 isAlive
`public final native boolean isAlive()` 线程是否存活
```java
class test{
	static int value = 1；
	main(){
		Thread t1 = new Thread(()->{
			value = 10;
		});
	}
	t1.start();
	// true
	sout(t1.isAlive());
	t1.join();
	// false
	sout(t1.isAlive());
	sout("主线程" + value);
}
```

### 3.11 setDaemon(boolean on)
将此线程标记为守护线程或用户线程

#### 3.11.1 守护线程
默认情况我们创建的线程都是用户线程，进程需要等所有的线程执行完毕后，进程才会结束
- <u>当所有的用户线程退出后，守护线程会立马结束</u>

通过 `Thread.isDaemon()` 判断线程是用户线程还是守护线程

应用：
1. 垃圾回收器线程
2. tomcat用来接收外部请求的线程

### 3.12 getState()
获取线程状态
Java中线程状态是用6个enum：NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED

NEW：初始状态，线程被构建，但还没有调用 start 方法
RUNNABLE：运行状态，Java线程将OS中的就绪和运行状态统称运行中
BLOCKED：阻塞，线程阻塞于锁
WAITING：等待状态，当前线程需要其他线程通知（notify或notifyAll）
TIMED_WAITING：超时等待，可指定等待时间自己返回
TERMINATED：终止：当前线程已执行完毕

![[Pasted image 20251215135243.png]]


## 4 线程状态间转换
### 4.1 Blocked 进入 Runnable
必须要线程获得 monitor 锁
但是如果想进入其他状态那就比较特殊，因为它没有超时机制，不会主动进入
![[Pasted image 20251215140323.png]]

### 4.2 Waiting 进入 Runnable
只有[执行了 `LockSupport.unpark()` / `join` 的线程运行结束 / 被中断]时才可以进入 Runnable
![[Pasted image 20251215140624.png]]

如果通过其他线程调用 notify() 或 notifyAll() 唤醒它，它会直接进入Blocked 状态
- 为什么不是直接进入Runnbale？
	- 因为唤醒 waiting 线程的线程如果调用 notify() 或 notifyAll()，要求必须首先持有该 monitor 锁，这就是我们说的 wait()、notify() 必须在synchronized块中
	- 所以 waiting 状态的线程被唤醒时拿不到该锁，就会进入 Blocked 状态，直到执行了 notify() 或 notifyAll()  的唤醒它的线程执行完毕并释放 monitor锁，才可能轮到它去抢夺这把锁，如果能抢到，就从 blocked 到 Runnable

### 4.3 Timed Waiting 进入 Runnable
同样的，在 timed waiting 中执行 notify 和 notifyAll，会先进入Blocked状态，然后抢锁成功后，再回到 Runnable状态
![[Pasted image 20251215141933.png|400]]

但是 timed waiting 存在超时机制，如果超时时间到了，系统会自动拿到锁，或者当 [join 的线程执行结束 / 调用了 `LockSupport.unpark()` / 被中断]等情况都会直接进入 Runnble 状态

### 4.4 总结
- 线程的状态是按箭头方向走的， new 不可以直接 runnable，要先经历 runnable
- 生命周期不可逆，一旦进入Runnable 就不能回到new，一终止就不可能有任何状态变化
- 一个线程只能有一次 new 和 terminated 状态，处于中间状态才可相互转换


## 5 线程池
如果并发数量很多，而且每个线程都是很短时间就执行结束，频繁创建销毁效率低
- 线程池：就是一个容纳多个线程的容器，其中的线程可以反复使用

- 线程池的优势：主要是<u>控制运行的线程数量</u>，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务执行
- 特点：线程复用、控制最大并发数、管理线程
1. 降低资源消耗：重复利用已创建的线程降低消耗
2. 提高响应速度：任务到达时，可以不用等待线程创建就能立即执行
3. 提高线程的可管理性：统一的分配、调优、监控

```java
ExecutorService es = Executors.newSingleThreadExecutor();
for(){
	es.execute(new Runnable(){
		@Override
		public void run(){
			...
		}
	})
}
es.shutdown();
es.awaitTermination(1, TimeUnit.DAYS);
```

线程池5种状态：RUNNING, SHUTDOWN, STOP, TIDYING, TERMINATED

### 5.1 内置线程池使用
java中线程池顶级接口是 `java.util.concurrent.Executor`，但严格说 Executor并不是一个线程池，而只是一个执行线程的工具
真正的线程池接口是 `java.util.concurrent.ExecutorService`
在 `java.util.concurrent.Executors` 线程工厂类里提供了一些静态工厂，生成一些常见的线程池

- newFixedThreadPool：创建一个固定长度的线程池，当达到线程池最大数量时，线程池规模不变
- newCachedThreadPool：创建一个可缓存的线程池，如果规模超出处理需求，将回收空线程；当需求增加，会增加线程数；线程池规模无限制
- newSingleThreadPoolExecutor：创建一个单线程的Executor，确保任务对了，串行执行
- newScheduledThreadPool：创建一个固定长度的线程池，可以定时的调度

前3个执行都是使用 `execute` 方法，`execute(task)；`
- task必须是要实现Runnable接口
	- ***execute()方法只接受 Runnable，不接受 Callable***
第4个执行是 `schedule` 方法，`schedule(task, 5, TimeUnit.SECONDS);`
- task可以是 Runnable或 Callable
- 第二个参数是延迟

### 5.2 线程池关闭
写在 finally 里，`threadPool.shutdown()`
- 不会立马停止正在执行的线程，会等待***所有任务***执行完后再关闭

`threadPool.shutdownNow()`
- 不会立马停止正在执行的线程，会等待***正在执行的线程***执行完后再关闭 
	- 阻塞队列中的任务不会执行

`threadPool.isTerminated();`
- 判断线程池是否真正的终止，代表线程已经执行完毕

 `threadPool.awaitTermination();`
- 等待线程池关闭，等待线程池中所有线程执行完

### 5.3 execute 和 submit
1. execute 只能接收 Runnable；submit 能接收Runnable 和 Callable
2. execute 返回值void，submit有返回值
3. execute 会在子线程中抛出异常，在主线程捕捉不到；submit不会抛出异常，而是会暂时存起来，直到future.get 才抛出，可以在主线程捕捉

<u>为什么 Execute 也能执行带返回值的线程 --- FutureTask</u>

### 5.4 线程池参数 & 原理
前三个都是new 了 `ThreadPoolExecutor`，只不过其参数不同，那么我们就可以不用内置的，自己填自己想要的参数，创建自己的线程池

- corePoolSize：核心线程池数量
- maximumPoolSize：最大线程数量
	- ***最大线程数量包含了核心线程数量***
- keepAliveTime：非核心线程的空闲状态的存活时间
- unit：时间单位
- workQueue：工作队列（阻塞队列）
- threadFactory：线程工厂（创建线程）
- handler：拒绝策略
	- AbortPolicy：丢弃并抛出异常
	- DiscardPolicy：丢弃但不抛出异常
	- DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务
	- CallerRunsPolicy：由调用线程处理该任务

![[Pasted image 20251216100138.png]]
- 注意先执行10，再执行20-30，最后11-20，注意顺序！

> [!创建优先级与执行优先级]
    > 创建优先级：核心 -> 阻塞队列 -> 最大线程
    > 执行优先级：核心 -> 最大线程 -> 阻塞队列

### 5.5 tomcat线程池和jdk线程池的区别
```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(  
        10,  
        200,  
        30,  
        TimeUnit.SECONDS,  
        new LinkedBlockingQueue<>(500)  
);
```
- jdk new一个线程池后，里面是没有线程的
- tomcat 抄了jdk，但是在构造函数的最后加了个 prestartAllCoreThreads();

### 5.6 源码
#### 5.6.1 线程池如何创建线程
addWorker方法作用：创建线程，并start
线程池构造出来，里面是没有线程的

提交一个任务，线程池就要构造一个线程，只要当前线程池现有的线程个数小于设置的核心线程数，就会addworker
> 也就是线程池核心线程数设置为10，假入开始提交了3个任务，线程池创建了3个线程执行，假设提交第4个任务时，任务1完成，线程1空闲，此时任务4不会用线程1，而是因为3 < 10，创建线程4来执行任务4

> 线程启动有start方法，但是没有明确的结束方法，线程执行结束后就消失了
> 线程池的作用就是，把这10个线程保留下来；如果不保留，任务1来了创建执行线程1，完事消失，来任务2再创建执行线程1，那线程池就形同虚设

所以线程1在执行完任务1后要***保活*** ---***利用阻塞队列***
> 实现上来说就是，先执行自己的任务，执行完自己的任务就会看阻塞队列，如果阻塞队列不空，就拿出一个任务执行；<u>如果阻塞队列为空，那线程会阻塞在阻塞队列 </u>---一边在等待任务，一边保活

当提交第11个，线程池的线程数已经有10个了，不再小于核心线程数，<u>任务会直接放到阻塞队列里</u>，如果有多个空闲线程，最终是哪个执行了任务要看阻塞队列的实现

假设阻塞队列已经满了且核心线程都在执行，又来了一个新的任务；此时会addWorker，又会创建线程执行任务 ---线程池其实是不公平的，后来的可能先执行
> 能再开多少个线程和配置的线程池最大线程数有关，如200，减去10个核心线程

如果超过最大线程数，再来任务，就会拒绝

#### 5.6.2 线程池拒绝策略
线程池的拒绝策略触发条件：
- 当前线程数 = `maximumPoolSize`（最大线程数）
- 工作队列已满
- 有新任务提交

##### 5.6.2.1 默认策略：AbortPolicy
抛一个异常，抛给 `execute` 方法

##### 5.6.2.2 CallerRunsPolicy
主线程会执行 runnable 中的 run 方法
正常来说 runnable 是由线程池里的某个线程执行的
但是线程池满了，线程池拒绝这个任务，所以就由当前调用这个线程的方法来执行（如main）来执行 run 方法

##### 5.6.2.3 DiscardOldestPolicy
把阻塞队列里第一个任务拿出阻塞队列，再把新的任务提交到线程池里（此时阻塞队列空一个，放进去，在末尾）

##### 5.6.2.4 DiscardPolicy
空方法，什么都不做

#### 5.6.3 线程池淘汰策略
***并不是最先创建的10个线程就是核心线程，比如最多时有200个线程，看淘汰后最后剩的那10个才是核心线程***

addWorker 会 new 一个 Worker，其构造器里会 new 一个 thread，(Worker 本身也是一个 Runnable) ，`newThread(this)`，这个 thread 对象 runnable 的对象是这个 worker， 所以线程会先执行 worker 里的 run 方法，这个 run 会调用 `runWorker` 方法

runWorker方法，先执行第一个任务，也就是我们提交的任务，这时候会调用 `task.run()`，***这个就是我们自己提交进来的 runnable 的重写的 run 方法，就是我们自己的逻辑***

执行完第一个任务，就会不断去阻塞队列里获取其他任务执行，写在 `getTask()` 里

`while (task != null || (task = getTask()) != null) {` 如果退出这个while循环，就会执行 finally 中的 `processWorkerExit` 方法，线程就消亡了，***所以控制了 getTask 什么时候返回 null，就控制了线程什么时候消亡***

##### 5.6.3.1 正常执行完毕
假设核心线程数是10，现在线程池有12个线程，任务都执行完，现在要淘汰两个

```java
private Runnable getTask() {  
    boolean timedOut = false; // Did the last poll() time out?  
  
    for (;;) {  
        int c = ctl.get();  
  
        // Check if queue empty only if necessary.  
        if (runStateAtLeast(c, SHUTDOWN) && 
           (runStateAtLeast(c, STOP)||workQueue.isEmpty())) {  
            decrementWorkerCount();  
            return null;        
        }  
  
        int wc = workerCountOf(c);  
  
        // Are workers subject to culling?  
        boolean timed = allowCoreThreadTimeOut || 
                        wc > corePoolSize;  
  
        if ((wc > maximumPoolSize || (timed && timedOut))  
            && (wc > 1 || workQueue.isEmpty())) {  
            if (compareAndDecrementWorkerCount(c))  
                return null;  
            continue;        
        }  
  
        try {  
            Runnable r = timed ?  
                workQueue.poll(keepAliveTime,                                                   TimeUnit.NANOSECONDS) :  
                workQueue.take();  
            if (r != null)  
                return r;  
            timedOut = true;  
        } catch (InterruptedException retry) {  
            timedOut = false;  
        }  
    }  
}
```

> workQueue.poll(）如果队列是空的，超时阻塞
> workQueue.take(）如果队列是空的，无限阻塞

这12个线程都会来获取线程池里的线程个数 `workerCountOf` ，各自获取的都是12 > 10，timed = true，此时 `timedOut` 是false， `compareAndDecrementWorkerCount` 方法不会执行，因为 timed 为 true，所以都会执行 `workQueue.poll()`，timeOut 设置为 true

第二次进入 for 循环，可执行 `compareAndDecrementWorkerCount` 方法，CAS，12个线程同时CAS，其中只有1个能成功，成功的线程可返回 null，淘汰；剩下失败的11个，同理；10个时，timed 为false，`workQueue.take()`，无限阻塞

##### 5.6.3.2 线程异常
假设我们淘汰两个后，现在就剩10个线程了，然后，这时候来了个任务，t3线程执行，过程中抛出异常 ---***线程会消亡，但是消亡前会创建一个新线程***

为什么这么做？和线程的一个机制 `UncaughtExceptionHandler` 有关，他需要异常在出错时抛出来，所以对线程池而言，如果其中的线程在执行中出现异常却不抛出，那就会导致这个机制失效

示例，非源码
```java
ThreadPoolExecutor executor1 = new ThreadPoolExecutor(  
        10,  
        200,  
        30,  
        TimeUnit.SECONDS,  
        new ArrayBlockingQueue<>(500),  
        new ThreadFactory() {  
            @Override  
            public Thread newThread(Runnable r) {  
                Thread thread = new Thread(r);  
                thread.setUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() { 
                    @Override  
                    public void uncaughtException(Thread t, Throwable e) {  
                        System.out.println("出错了");  
                    }  
                });  
                return thread;  
            }  
        }  
);  
  
executor1.execute(new Runnable() {  
    @Override  
    public void run() {  
        throw new NullPointerException();  
    }  
});
```

##### 5.6.3.3 线程池关闭
`executor.shutdown()` 和 `executor.shutdownNow()`

> [!关闭线程]
    > 线程有stop方法， 但是一般不用，优雅的关闭线程：***中断***

shutdown会把线程池状态改成`SHUTDOWN`，shutdownNow会把改成`STOP`
改完线程池状态后，都会中断线程
中断线程，拿t1举例，只是给t1发了一个信号，t1是否会停掉，要看t1自己的逻辑

```java
if (runStateAtLeast(c, SHUTDOWN)  
    && (runStateAtLeast(c, STOP) || workQueue.isEmpty())) {  
    decrementWorkerCount();  
    return null;}
```

 假设阻塞队列正好是空的，10个线程正在被阻塞(workQueue.take())，怎么关闭？
 中断它们，它们都会抛出 InterrupttedException

总结：<u>正在执行任务的线程，会先执行完任务，在取下一个任务时，判断退出；没有任务在阻塞的线程，通过中断导致退出</u>


---


## 6 线程安全
### 6.1 什么是线程安全
多线程下并发同时对共享数据进行独写，会造成数据混乱 = 线程不安全
 当多线程并发访问临界资源时，如果破坏其原子性、可见性、有序性，可能会造成数据不一致

- 临界资源：共享资源（同一对象）同时读写，一次仅允许一个线程使用

#### 6.1.1 原子性
单一不可分割的操作
提供互斥访问，同一时刻只能有一个线程对数据进行操作（Atomic、CAS算法、synchronized、Lock）

只有第一个是原子性操作，像第二个，分拿到i的值，把i赋给j
```java
i = 0;
j = i;
i++;
i = j +1;
```

***Java只保证了基本数据类型的变量和赋值操作是原子性的***
在32位的JDK环境下，对64位数据的读取不是原子性操作，如long，double

要想在多线程环境下保证原子性，可以通过锁、synchronized来确保
<u>volatile无法保证复合操作的原子性</u>

#### 6.1.2 可见性
可见性是指当多个线程访问同一个线程时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值

CPU在执行代码时，为了减少变量访问的时间消耗，可能将代码中访问的变量的值缓存到该CPU缓存区中，因此相应的代码再次访问该变量时，相应的值可能从CPU缓存中而不是主内存中读取。同样的，代码对这些被缓存过的变量的值的修改也可能仅是被写入CPU缓存区，而没有写入主内存。由于每个CPU都有自己的缓存区，因此一个CPU缓存区中的内容对于其他CPU而言是不可见的

程序会一直执行，说明always不可见
```java
static boolean always = true;

main(){
	// 线程1  
	new Thread(() -> {  
	    while(always){  
	  
	    }  
	}).start();  
  
	Thread.sleep(2000);  
  
	// 线程2  
	always = false;
}
```

- 解决：
	- 或者不用synchronized，用个sout就行，因为sout里也有synchronized
```java
new Thread(() -> {  
    while(always){  
        synchronized (always){  
  
        }  
    }  
}).start();
```

对于可见性，java提供了volatile关键字保证可见性。***当一个共享变量被volatile修饰时，他会保证修改的值会立即更新到主存***，当有其他线程需要读取时，它会去内存中读取新值。而普通的共享变量不能保证可见性，因为普通共享变量被修改后，什么时候写入主存是不确定的，当其他线程去读取时，此时的内存中可能还是原来的旧值。另外，***通过synchronized和Lock也能够保证同一时刻只有一个线程获取锁然后执行同步代码***，并在释放锁之前会将对变量的修改刷新到主存中

##### JMM内存模型
![[Pasted image 20251223141554.png]]

左测蓝底部分是 工作内存(缓存区) --- 每个线程
 
#### 6.1.3 有序性（指令重排）
有序性最终表述的现象是，CPU是否按照既定代码顺序依次执行指令，<u>编译器和CPU为了提高指令的执行效率可能会进行指令重排序</u>，这使得代码的实际执行方式可能不是按照我们认为的方式进行
> 我们的代码会编译成一个字节码文件(.class)，二进制指令会交给JVM执行

在单线程的情况下只要保证最终执行结果正确即可，as-if-serial
- 上面代码最终执行结果是i=1，flag=true；在不影响这个结果下，2可能比1先执行，4可能比3先执行 
```java
int i = 0; //语句1
boolean flag = false; //语句2
i = 1; //语句3
flag = true; //语句4
```


### 6.2 如何解决线程不安全问题
***synchronized 会把并行改为串行***，性能上不去

#### 6.2.1 破坏临界资源
##### 6.2.1.1 只读
`final`

##### 6.2.1.2 局部变量
***每个线程的局部变量会存在栈帧中***，会在每个线程的栈帧内存中被创建多份，因此不存在共享
![[Pasted image 20251223173827.png|350]]

##### 6.2.1.3 ThreadLocal
###### 6.2.1.3.1 ThreadLocal是什么
ThreadLocal 是线程本地变量，如果创建了一个ThreadLocal 变量，那么访问这个变量的每个线程都会有这个变量的一个本地拷贝，多个线程操作这个变量的时候，实际是操作自己本地内存里面的变量

![[Pasted image 20251223175015.png|450]]

***ThreadLocal是整个线程的全局变量，不是整个程序的全局变量***

```java
static ThreadLocal<User> userThreadLocal = new ThreadLocal<>();  
  
public static void main(String[] args) throws InterruptedException {  
      
    Thread thread1 = new Thread(() -> {  
        userThreadLocal.set(new User());  
        User user = userThreadLocal.get();  
    });  
    thread1.start();
```

- ThreadLocal是java中提供的线程本地存储机制，可以利用该机制将缓存数据存在某个线程内部，该线程可以在任意时刻、任意方法中获取缓存的数据
- ThreadLocal底层是通过ThreadLocalMap实现的，***每个Thread对象（注意不是
ThreadLocal对象）中都存在一个ThreadLocalMap***，***Map的key为ThreadLocal对象，value为需要缓存的值***

![[Pasted image 20251225103915.png|400]]

###### 6.2.1.3.2 ThreadLocal内存泄漏
在***线程池中使用ThreadLocal会造成内存泄漏***，因为当ThreadLocal对象使用完后，应该把设置的key，value，也就是entry对象进行回收，但线程池中的线程不会回收，而线程对象是通过强引用指向ThreadLocalMap，ThreadLocalMap也是通过强引用指向Entry对象，线程不被回收，Entry对象也就不会回收，从而出现内存泄漏
- 解决办法：使用了ThreadLocal对象后，<u>手动调用ThreadLocal的remove方法</u>

> GC不会回收强引用

![[Pasted image 20251225111035.png]]

###### 6.2.1.3.3 Inheritable ThreadLocal
 `Inheritable ThreadLocal` 是 ThreadLocal 子类

ThreadLocal中的数据是绑定在线程上的，主线程和子线程是两个数据，数据隔离
所以s的值是null
```java
public static void main(String[] args) throws InterruptedException {  
  
    ThreadLocal<String> threadLocal = new ThreadLocal<>();  
    threadLocal.set("test");  
  
    Thread thread2 = new Thread(()->{  
        String s = threadLocal.get();  
        System.out.println(s);  // null  
    });  
    thread2.start();
```

- 用 `Inheritable ThreadLocal` 可以避免这个问题
- 但是也有问题，发现并没有输出teset111
	- 当改变了InheritableThreadLocal 的值，输出的值是一样的。
	- 因为在使用线程池时，***核心线程在首次使用被创建时能正确复制父线程的上下文，但之后已复制上下文的核心线程不会回收的情况下，线程池中的线程上下文将不会再改变***！
	- 可以用阿里开源的 `TransmittableThreadLocal`
```java
InheritableThreadLocal<String> threadLocal1 = new InheritableThreadLocal<>();  
threadLocal1.set("test");

ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(5);  
  
for (int i = 0; i < 10; i++) {  
    if (i == 5) {  
        threadLocal.set("test111");  
    }  
    executor.execute(() -> {  
        String s = threadLocal.get();  
        System.out.println(s);  
    });  
}
```

##### 6.2.1.4 volatile
两个特性：保证
可见性、禁止指令重排

```java
static volatile Boolean always1 = true;

// 线程1  
new Thread(() -> {  
    while (always) {  
        synchronized (always1) {  
  
        }  
    }  
}).start();  
  
Thread.sleep(2000);  
  
// 线程2  
always1 = false;
```


---


# JMM内存模型
## 1 介绍
Java多线程内存模型

![[Pasted image 20251225150610.png]]
<center>多核并发缓存架构</center>

JMM内存模型就是根据多核并发缓存模型设计的
- Java线程内存模型时标准化的，屏蔽掉了底层不同计算机的区别
![[Pasted image 20251225151041.png|400]]

- 我们发现一直不会输出success，说明修改的是线程内各自的副本
	- 想输出succcess，加上volatile关键字
```java
static boolean initFlag = false;  
  
public static void main(String[] args) throws InterruptedException {  
  
    new Thread(()->{  
        System.out.println("waiting data");  
        while(!initFlag){  
  
        }  
        System.out.println("success");  
    }).start();  
  
    Thread.sleep(2000);  
  
    new Thread(()->{  
        System.out.println("prepare data");  
        initFlag = true;  
        System.out.println("prepare end");  
    }).start();
```


## 2 JMM数据原子操作
read（读取）：从主内存读取数据
load（装载）：将主内存读取到的数据写入工作内存
use（使用）：从工作内存读取数据来计算
assign（赋值）：将计算好的值重新赋值到工作内存中
store（存储）：将工作内存数据写入主内存
write（写入）：将store过去的变量值赋值给主内存中的变量
lock（锁定）：将主内存变量加锁，标识为线程独占状态
unlock（解锁）：将主内存变量解锁，解锁后其他线程可以锁定该变量

- 不加 volatile
![[Pasted image 20251225161909.png]]

## 3 volatile底层原理 ---可见性
- 缓存一致性协议（MESI）：多个CPU从主内存读取同一个数据到各自的高速缓存，当其中某个CPU修改了缓存里的数据，该数据会***马上同步回主存***，其他CPU通过***总线嗅探机制***可以感知到数据的变化从而将自己缓存里的数据失效

- 缓存加锁：缓存锁的核心机制是基于缓存一致性协议来实现的，一个处理器的缓存回写到内存会导致其他处理器的缓存无效，Intel64处理器使用MESI实现缓存一致性协议

- Volatile缓存可见性实现原理
	- 通过***汇编lock前缀指令***，它会<u>锁定这块内存区域的缓存（缓存行锁定），并回写到主内存</u>
- Intel64对lok指令的解释
1. 会将当前处理器缓存行的数据立即写回到系统内存
2. 这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效(MESI)
3. 提供内存屏障功能，使lock前后指令不能重排序


---


# 并发编程
## 1 三大特性
并发编程三大特性：**可见性、有序性、原子性**

- volatile 保证可见性和有序性，但不保证原子性，原子性需要synchronized锁

- 指令重排序：在不影响单线程程序执行结果的前提下，计算机为了最大限度的发挥机器性能，会对机器指令重排序优化
	- 重排序会遵守 `as-if-serial` 与 `happens-before` 原则                  
![[Pasted image 20251225171334.png|500]]

## 2 有序性
### 2.1 as-if-serial
不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。为了遵守这个语义，编译器和处理器<u>不会对存在数据依赖关系的操作做重排序</u>，因为这种重排序会改变执行结果

### 2.2 happens-before
判断数据是否存在竞争、线程是否够安全的依据


## 3 可见性
### 3.1 单例模式DCL导致的可见性问题
双重检测锁DCL对象半初始化问题

```java
public class DoubleCheckLockSingleton {  
  
    private static DoubleCheckLockSingleton instance = null;  
  
    private DoubleCheckLockSingleton() {  
  
    }  
  
    // DCL  
    public static DoubleCheckLockSingleton getInstance() {  
        if (instance == null) {  
            synchronized (DoubleCheckLockSingleton.class) {  
        //多线程排队，确保第一个返回instance后，其他线程再判断不是null  
                if (instance == null) {  
                    instance = new DoubleCheckLockSingleton();  
                }  
            }  
        }  
        return instance;  
    }  
  
    public static void main(String[] args) {  
        DoubleCheckLockSingleton instance = DoubleCheckLockSingleton.getInstance();  
    }  
  
}
```

- 但是有bug，会导致对象的半初始化

- `private static DoubleCheckLockSingleton instance = null;` 要加volatile

- `instance = new DoubleCheckLockSingleton();` 这步编译后其中有两个指令
1. `<init>` 创建对象时的指令赋值
2. `putstatic` 赋值静态变量，即把new的对象赋值给instance
- 但是这两步可能会指令重排，因为没有违反两个原则
	- 重排后，对象还没有init就先put了，半初始化状态


#### 3.1.1 补充知识---对象的创建
![[Pasted image 20251226102304.png]]

##### 3.1.1.1 类加载检查
虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程
> new指令对应到语言层面上讲是，new关键词、对象克隆、对象序列化等

##### 3.1.1.2 分配内存
在类加载检查通过后，接下来虚拟机将新生对象分配内存。对象所需内存的大小在类加载完成后即可完全确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来

这个步骤有两个问题：
1. 如何划分内存
2. 在并发情况下，可能出现正在给对象A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存

划分内存的方法：
- 指针碰撞（Bump the Pointer）（默认）
	- 如果java堆中内存是绝对规整的，所用用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离
- 空闲列表（free list）
	- 如果java堆中内存不是规整的，已使用和空闲内存交错，虚拟机就需维护一个列表，记录哪些内存是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录

解决并发问题的方法：
- CAS（compare and swap）
	- 虚拟机采用 [CAS + 失败重试] 的方式保证更新操作的原子性来对分配内存空间的动作进行同步处理
- 本地线程分配缓冲（Thread Local Allocation Buffer，TLAB）
	- 把内存分配的动作按照线程划分在不同的空间中进行，即每个线程在java堆中预先分配一小块内存，通过 `-XX:+/-UseTLAB` 参数来设定虚拟机是否使用TLAB（默认开启）

##### 3.1.1.3 初始化零值
内存分配后完成后，虚拟机需要将分配的内存空间都初始化为零值（不包括对象头），如果使用TLAB，这一工作过程也可以提前至TLAB分配时进行。这一步操作保证了<u>对象的实例字段在java代码中可以不赋初始值就直接使用</u>，程序能访问到这些字段的数据类型所对应的零值

##### 3.1.1.4 设置对象头
初始化零值之后，虚拟机要对对象进行必要的设置，如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息，这些都存放在对象的对象头 `Object Header` 之中

在Hotspot虚拟机中，对象在内存中存储的布局可以分为3个区域：对象头（Header)、实例数据（Instance)、对齐填充（Padding）
- 对象头包括：
1. 用于存储对象自身的运行时数据
2. 类型指针，即对象指向它的元数据的指针，虚拟机通过这个指针确定这个对象是哪个类的实例

##### 3.1.1.5 执行 `<init>` 方法
执行 `<init>` 方法，即对<u>象按照程序员的意向进行初始化</u>。即属性赋值和执行构造方法

#### 3.1.2  指令重排与内存屏障
为什么加个volatile就能避免问题？

Java规范定义的内存屏障
- <u>store相当于写，load相当于读</u>
![[Pasted image 20251226114304.png]]

- Java该规定volatile需要实现的内存屏障
```java
StoreStore屏障
a = 1;  // volatile写，a为volatile变量
StoreLoad屏障

b = a;  // volatile读
LoadLoad屏障
LoadStore屏障
```

- 不同CPU硬件对于JVM的内存屏障规范实现指令不一样
	- Intel
		- Ifence：是一种 load Barrier 读屏障
		- sfence：是一种 store Barrier 写屏障
		- mfence：全能型屏障
- JVM底层简化了内存屏障硬件指令的实现
	- lock前缀：<u>lock指令不是一种内存屏障，但它能完成类似内存屏障的功能</u>


---


## 4 原子类
### 4.1 概述
- 不可分割
- 一个操作是不可中断的，即使多线程下也可保证
- `java.util.concurrent.atomic`

原子变量可以把竞争范围缩小到变量级别，这是可获得的最细粒度

#### 4.1.1 基本数据类型

| 类型                                                            | 具体类                                                                                                                                   |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `Atomic*` 基本类型原子类                                             | AtomicInteger<br>AtomicLong<br>AtomicBoolean                                                                                          |
| `Atomic*Array`  数组类型原子类<br><br><br>`Atomic*Reference` 引用类型原子类 | AtomicIntegerArray<br>AtomicLongArray<br>AtomicReferenceArray<br>AtomicReference<br>AtomicStampedReference<br>AtomicMarkableReference |
| `Atomic*FieldUpdater` 升级类型原子类                                 | AtomicIntegerfieldUpdater<br>AtomicLongfieldUpdater<br>AtomicReferencefieldUpdater                                                    |
| `Adder` 累加器                                                   | LongAdder<br>DoubleAdder                                                                                                              |
| `Accumulator` 积累器                                             | LongAccumulator<br>DoubleAccumulator                                                                                                  |

```java
static AtomicInteger a = new AtomicInteger(100);
```

#### 4.1.2 AtomicInteger
```java
AtomicInteger ai = new AtomicInteger(1);

ai.get(); // 获取值,1
ai.addAndGet(5); // 增加指定值并且获取，6
// 比较并且设置，1.预期值，2.新值，会将预期值与当前比较，同则设置新值
// 因为第一个参数就是ai本身，所以肯定是返回true
ai.compareAndSet(ai.get(),10); 
ai.getAndIncrement(); // 获取并且递增
ai.incrementAndget(); // 递增并且获取
ai.set(); // 设置值，会保证可见性
ai.lazySet(); // 懒设置，不会保证可见性
```

#### 4.1.3 AtomicArray
- 和上面的区别就是多了个参数，下标
- 注意 aia 是arr 的一个***拷贝，两者不是同一个***
```java
static int[] arr = new int[]{1, 2, 3};  
static AtomicIntegerArray aia = new AtomicIntegerArray(arr);

arr[0] 和 aia.get(0) 值不一定相同
```

#### 4.1.4 AtomicFieldUpdator
把对象中的某个字段升级成原子类型
比如User对象中有个Integer age，就可以升级为AtomicInteger
```java
@Data  
public class User {  
    public String name;  
    public volatile Integer age;  
}

static AtomicIntegerFieldUpdater<User> aifuUser = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");
```

#### 4.1.5 Adder
如果是count++操作，可用 
```java
AtomicInteger count = new AtomicInteger();
count.addAndGet(1);
```

如果是jdk8，推荐使用 LongAdder对象，比AtomicLong性能好（<u>减少乐观锁的重试次数</u>）
- 本质是空间换时间
- 竞争激烈时，LongAdder把不同线程对应到不同的cell上进行修改，降低了冲突的概率，是多段锁的理念，提高了并发性
- LongAdder 适合的场景是统计求和计数的场景，而且其只提供了add方法

#### 4.1.6 Accumulator
可以做自定义的运算
```java
static LongAccumulator la = new LongAccumulator(Long::sum, 1);
```


---


## 5 锁
锁的分类：[[技术/JUC/锁.canvas]]

### 5.1 乐观锁、悲观锁
乐观锁和悲观锁不是锁的实现，而是锁的类型

#### 5.1.1 悲观锁
认为自己在使用数据时，一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改

- synchronized 关键字和 Lock的实现类都是悲观锁
- 适合写操作多的场景，先加锁可以保证写操作时数据正确
- 显式的锁定之后再操作同步资源

![[Pasted image 20251229162830.png]]

#### 5.1.2 乐观锁
认为自己在使用数据时不会有别的线程修改数据，不会添加锁，只是在更新数据时去判断之前有没有别的线程更新了这个数据
- 如果这个数据没有被更新，当前线程将自己修改的数据成功写入
- 如果这个数据已经被其他线程更新，则根据不同的实现方式执行不同的操作

乐观锁在java中是通过无锁编程来实现，最常采用是CAS算法（比较并替换），Java原子类中的递增操作就通过CAS自旋实现的

- 适合读操作多的场景，不加锁的特点能使其读操作的性能大幅提升
- 乐观锁直接去操作同步资源，是一种无锁算法

![[Pasted image 20251229163725.png]]

##### 5.1.2.1 乐观锁的实现 CAS
###### 5.1.2.1.1 没有CAS之前
多线程环境不使用原子类保证线程安全

###### 5.1.2.1.2 多线程环境
使用原子类保证线程安全（基本数据类型）

###### 5.1.2.1.3 CAS是什么
compare and swap的缩写，比较并交换
- 包含三个操作数：内存位置、预期原值、更新值
执行CAS操作时，将内存未知的值与预期原值比较：
- 如果相匹配，则处理器会自动将该位置值更新为新值
- 如果不匹配，处理器不做任何操作，多个线程同时执行CAS操作只有一个会成功
 
> CAS有3个操作数，位置内存值V，旧的预期值A，要修改的更新值B
> 当且仅当旧的预期值A和内存值V相同时，将内存值V改为B，否则什么都不做

CAS是JDK提供的非阻塞原子性操作，它通过硬件保证了 比较-更新的原子性

CAS是一条CPU的原子指令（cmpxchg指令），Unsafe提供的CAS方法（如compareAndSwapXXX）底层实现即为CPU指令cmpxchg
执行cmpxchg指令时，会判断当前系统是否为多核系统，如果是就给总线加锁，只有一个线程会给总线加锁成功，加锁成功会执行cas操作，也就是<u>CAS的原子性实际上是CPU实现的</u>，其实在这一点上还是有排他锁的，只是比起用synchronized，这里的排他时间要短的多

###### 5.1.2.1.4 CAS底层原理
Unsafe类使java拥有像c指针一样操作内存空间的能力
- 特点：
1. 不受jvm管理，无法被GC，需手动GC
2. Unsafe的不少方法中必须提供原始地址（内存地址）和被替换对象的地址，偏移量要自己计算，一旦有问题就会导致整个JVM实例崩溃
3. 直接操作内存，速度快

AtomicInteger类主要利用 [CAS + volatile + native] 方法来保证原子操作

> i++不安全，那 atomicInteger.getAndIncrement()

###### 5.1.2.1.5 引出来ABA问题




### 5.2 自旋锁（spinlock）
当一个线程在获取锁的时候，如果锁已经被其他线程获取，那该线程将循环等待，然后不断的判断锁是否能被成功获取，直到获取锁时才退出循环

自旋锁与互斥锁类似，都是为了解决对某项资源的互斥使用
无论是互斥锁还是自旋锁，任何时刻，最多只能有一个保持者

- 对于互斥锁，会让没有得到锁资源的线程进入Blocked状态，而后在争夺到锁资源后恢复为RUNNABLE状态，整个过程涉及到OS的用户态和内核态的切换
- 自旋锁不会引起调用者堵塞，如果自旋锁已经被别的执行单元执行，调用者就一直在那里看持有者是否已经释放了锁

自旋锁的实现基础上是CAS。CAS自旋锁属于乐观锁，乐观的认为程序中的并发情况不那么严重，所以让线程不断地尝试



  















































































