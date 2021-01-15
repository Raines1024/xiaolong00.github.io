---
layout: post
title: 《Java异步编程实战》读书笔记
description: 《Java异步编程实战》读书笔记
category: blog
date: 2020-01-07 13:50:39
---

## Java异步编程实战 (Java核心技术系列)

- 使用Future确实可以获取异步任务的执行结果，但是获取其结果还是会阻塞调用线程的，并没有实现完全异步化处理，所以在JDK8中提供了CompletableFuture来弥补其缺点。CompletableFuture类允许以非阻塞方式和基于通知的方式处理结果，其通过设置回调函数方式，让主线程彻底解放出来，实现了实际意义上的异步处理。

- 在Java中实现异步编程最简单的方式是：每当有异步任务要执行时，使用Tread来创建一个线程来进行异步执行。

- Java中有两种方式来显式开启一个线程进行异步处理。第一种方式是实现java.lang.Runnable接口的run方法，然后传递Runnable接口的实现类作为创建Thread时的参数，启动线程

- Java中第二种开启线程进行异步执行的方式是实现Thread类，并重写run方法

- Java中线程是有Deamon与非Deamon之分的，默认情况下我们创建的都是非Deamon线程，线程属于什么类型与JVM退出条件有一定的关系。在Java中，当JVM进程内不存在非Deamon的线程时JVM就退出了。那么如何创建一个Deamon线程呢？其实将调用线程的setDaemon(boolean on)方法设置为true就可以了

- 显式使用Thread创建异步任务的两种方式，但是上述实现方式存在几个问题：  ·每当执行异步任务时，会直接创建一个Thread来执行异步任务，这在生产实践中是不建议使用的，因为线程创建与销毁是有开销的，并且没有限制线程的个数，如果使用不当可能会把系统线程用尽，从而造成错误。在生产环境中一般创建一个线程池，然后使用线程池中的线程来执行异步任务，线程池中的线程是可以被复用的，这可以大大减少线程创建与销毁开销；另外线程池可以有效限制创建的线程个数。  ·上面使用Thread执行的异步任务并没有返回值，如果我们想异步执行一个任务，并且需要在任务执行完毕后获取任务执行结果，则上面这个方式是满足不了的，这时候就需要用到JDK中的Future了。

- 线程池状态含义：  ·RUNNING：接收新任务并且处理阻塞队列里的任务。  ·SHUTDOWN：拒绝新任务但是处理阻塞队列里的任务。  ·STOP：拒绝新任务并且抛弃阻塞队列里的任务，同时中断正在处理的任务。  ·TIDYING：所有任务都执行完（包含阻塞队列里面任务），当前线程池活动线程为0，将要调用terminated方法。  ·TERMINATED：终止状态。terminated方法调用完成以后的状态

- 线程池同时提供了一些方法用来获取线程池的运行状态和线程池中的线程个数

- 线程池提供了可供使用者配置的参数：

- corePoolSize：线程池核心线程个数。

- workQueue：用于保存等待执行的任务的阻塞队列，比如基于数组的有界Array-BlockingQueue、基于链表的无界LinkedBlockingQueue、最多只有一个元素的同步队列SynchronousQueue、优先级队列PriorityBlockingQueue等。

- maximunPoolSize：线程池最大线程数量。

- threadFactory：创建线程的工厂类。

- defaultHandler：饱和策略，当队列满了并且线程个数达到maximunPoolSize后采取的策略，比如AbortPolicy（抛出异常）、CallerRunsPolicy（使用调用者所在线程来运行任务）、DiscardOldestPolicy（调用poll丢弃一个任务，执行当前任务）、DiscardPolicy（默默丢弃，不抛出异常）。

- keeyAliveTime：存活时间。如果当前线程池中的线程数量比核心线程数量要多，并且是闲置状态的话，这些闲置的线程能存活的最大时间。

- ThreadPoolExecutor(int corePoolSize,//核心线程个数                    int maximumPoolSize,//最大线程个数                    long keepAliveTime,//非核心不活跃线程最大存活时间                    TimeUnit unit,//keepAliveTime的单位                    BlockingQueue<Runnable> workQueue,//阻塞队列类型                    ThreadFactory threadFactory,//线程池创建工厂                    RejectedExecutionHandler handler)//拒绝策略

- 当我们需要创建自己的线程池时，就可以显式地新建一个该实例出来。

- 方法public void execute(Runnable command)提交任务到线程池的逻辑：

- /(1) 如果任务为null，则抛出NPE异常     if (command == null)         throw new NullPointerException();

- //（2）获取当前线程池的状态+线程个数变量的组合值     int c = ctl.get();

- //（3）当前线程池线程个数是否小于corePoolSize,小于则开启新线程运行     if (workerCountOf(c) < corePoolSize) {         if (addWorker(command, true))             return;         c = ctl.get();     }

- //（4）如果线程池处于RUNNING状态，则添加任务到阻塞队列     if (isRunning(c) && workQueue.offer(command)) {          //（4.1）二次检查         int recheck = ctl.get();         //（4.2）如果当前线程池状态不是RUNNING则从队列删除任务，并执行拒绝策略         if (! isRunning(recheck) && remove(command))             reject(command);          //（4.3）如果当前线程池线程为空，则添加一个线程         else if (workerCountOf(recheck) == 0)             addWorker(null, false);     }

- //（5）如果队列满了，则新增线程，新增失败则执行拒绝策略     else if (!addWorker(command, false))         reject(command);

- 代码3是指如果当前线程池线程个数小于corePoolSize，则会在调用方法addWorker新增一个核心线程执行该任务。

- 如果当前线程池线程个数大于等于corePoolSize则执行代码4，如果当前线程池处于RUNNING状态则添加当前任务到任务队列。这里需要判断线程池状态是因为线程池有可能已经处于非RUNNING状态，而非RUNNING状态下是抛弃新任务的。

- 如果任务添加任务队列成功，则执行代码4.2对线程池状态进行二次校验，这是因为添加任务到任务队列后，执行代码4.2前线程池的状态有可能已经变化了，如果当前线程池状态不是RUNNING则把任务从任务队列移除，移除后执行拒绝策略；如果二次校验通过，则执行代码4.3重新判断当前线程池里面是否还有线程，如果没有则新增一个线程。

- 如果代码4添加任务失败，则说明任务队列满了，那么执行代码5尝试调用addWorker方法新开启线程来执行该任务；如果当前线程池的线程个数大于maximumPoolSize则addWorker返回false，执行配置的拒绝策略。

- public Future<?>submit(Runnable task)方法提交任务的逻辑：

- // 6 NPE判断     if (task == null) throw new NullPointerException();

- // 7 包装任务为FutureTask     RunnableFuture<Void> ftask = newTaskFor(task, null);

- // 8 投递到线程池执行     execute(ftask);

- // 9 返回ftask     return ftask;

- 代码7调用newTaskFor方法对我们提交的Runnable类型任务进行包装，包装为RunnableFuture类型任务，然后提交RunnableFuture任务到线程池后返回ftask对象。

- 线程池是通过池化少量线程来提供线程复用的，当调用线程向线程池中投递大量任务后，线程池可能就处于饱和状态了。所谓饱和状态是指当前线程池队列已经满了，并且线程池中的线程已经达到了最大线程个数。当线程池处于饱和状态时，再向线程池投递任务，而对于投递的任务如何处理，是由线程池拒绝策略决定的。

- 线程池中提供了RejectedExecutionHandler接口，用来提供对线程池拒绝策略的抽象

- AbortPolicy策略

- 该拒绝策略执行时会直接向调用线程抛出RejectedExecutionException异常，并丢失提交的任务r。

- CallerRunsPolicy策略

- 该拒绝策略执行时，如果线程池没有被关闭，则会直接使用调用线程执行提交的任务r，否则默默丢弃该任务。

- DiscardPolicy策略

- 该拒绝策略执行时，什么都不做，默默丢弃提交的任务。

- DiscardOldestPolicy策略

- 该拒绝策略首先会丢弃线程池队列里面最老的任务，然后把当前任务r提交到线程池

- 虽然线程池方式提供了线程复用可以获取任务返回值，但是获取返回值时还是需要阻塞调用线程的，所以我们在下一章会讲解JDK提供的CompletableFuture来解决这个问题

- V get()throws InterruptedException，ExecutionException：等待异步计算任务完成，并返回结果

- V get(long timeout，TimeUnit unit)throws InterruptedException，ExecutionException，TimeoutException：相比get()方法多了超时时间，当线程调用了该方法后，在任务结果没有计算出来前调用线程不会一直被阻塞，而是会在等待timeout个unit单位的时间后抛出TimeoutException异常后返回。添加超时时间避免了调用线程死等的情况，让调用线程可以及时释放。

- boolean isDone()：如果计算任务已经完成则返回true，否则返回false。需要注意的是，任务完成是指任务正常完成了、由于抛出异常而完成了或者任务被取消了

- boolean cancel(boolean mayInterruptIfRunning)：尝试取消任务的执行；如果当前任务已经完成或者任务已经被取消了，则尝试取消任务会失败；如果任务还没被执行时调用了取消任务，则任务将永远不会被执行；如果任务已经开始运行了，这时候取消任务，则参数mayInterruptIfRunning将决定是否要将正在执行任务的线程中断，如果为true则标识要中断，否则标识不中断；当调用取消任务后，再调用isDone()方法，后者会返回true，随后调用isCancelled()方法也会一直返回true；如果任务不能被取消，比如任务完成后已经被取消了，则该方法会返回false。

- boolean isCancelled()：如果任务在执行完毕前被取消了，则该方法返回true，否则返回false。

- FutureTask代表了一个可被取消的异步计算任务，该类实现了Future接口，比如提供了启动和取消任务、查询任务是否完成、获取计算结果的接口。

- FutureTask任务的结果只有当任务完成后才能获取，并且只能通过get系列方法获取，当结果还没出来时，线程调用get系列方法会被阻塞。另外，一旦任务被执行完成，任务将不能重启，除非运行时使用了runAndReset方法。FutureTask中的任务可以是Callable类型，也可以是Runnable类型（因为FutureTask实现了Runnable接口），FutureTask类型的任务可以被提交到线程池执行。

- 当我们创建一个FutureTask时，其任务状态初始化为NEW，当我们把任务提交到线程或者线程池后，会有一个线程来执行该FutureTask任务，具体是调用其run方法来执行任务。在任务执行过程中，我们可以在其他线程调用FutureTask的get()方法来等待获取结果，如果当前任务还在执行，则调用get的线程会被阻塞然后放入FutureTask内的阻塞链表队列；多个线程可以同时调用get方法，这些线程可能都会被阻塞并放到阻塞链表队列中。当任务执行完毕后会把结果或者异常信息设置到outcome变量，然后会移除和唤醒FutureTask内阻塞链表队列中的线程节点，进而这些由于调用FutureTask的get方法而被阻塞的线程就会被激活。

- CompletableFuture是一个可以通过编程方式显式地设置计算结果和状态以便让任务结束的Future，并且其可以作为一个CompletionStage（计算阶段），当它的计算完成时可以触发一个函数或者行为；当多个线程企图调用同一个CompletableFuture的complete、cancel方式时只有一个线程会成功。

- CompletableFuture功能强大的原因之一是其可以让两个或者多个Completable-Future进行运算来产生结果