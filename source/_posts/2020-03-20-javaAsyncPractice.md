---
layout: post
title: 多线程及异步编程的前世今生
description: 多线程与异步系列（一）
category: blog
date: 2020-01-07 13:50:39
---

## 显式使用线程和线程池实现异步编程

### Java中显式开启一个线程进行异步处理的两种方式  

#### 理论基础
- 在Java中，Java虚拟机允许应用程序同时运行多个执行线程，所以我们可在main函数内开启一个线程来异步执行任务A，而main函数所在线程执行B，即可大大缩短整个任务处理耗时。  
- Java中有两类线程：  
User Thread(用户线程)、Daemon Thread(守护线程)

- run()是多线程程序的一个约定，所有的多线程代码都在其中执行。  
- 在启动多线程时，首先需要通过Thread(Runnable target)构造出线程对象，再调用start()运行多线程。  
- 所有的多线程代码都是通过Thread的start()来运行。  
- 线程调用 join() 后，main线程必须等待该线程终止后，main线程才能继续进行；若不调用join()，main线程执行完会退出，等待非守护线程结束后JVM退出  
- 并发问题的小疑惑  
既然CPU同一时间只能执行一个线程，为什么存在并发问题  
CPU的时间是按时间片分的，而不是一个时间点，并发问题是由于CPU线程切换导致的。  
现在假设有一段代码如下：

```
if(i == 1) {
    i++;　　//断点1
    system.out.print(i);       
}　//断点2
```
有两个线程A，B同时执行这一段代码，假设A线程先被CPU调度，然而A线程在断点1处，时间片到期了，此时A线程的代码并没有执行完，但是CPU此时会调度B线程，并不会管A线程是不是执行完了这一段代码。  
再接着假设B线程现在执行完了这一段代码(当然也可能没有执行完)，CPU 现在就又会调度A线程，并且从A线程的断点1处继续执行(注意不是重新执行，CPU切换的时候保存了线程的上下文)   
总结一下：CPU切换线程并不会管你线程是否将代码执行完，而是和分给线程的时间片是否到期有关，时间片到期了就会切换线程，并发也就由此产生了。  

#### 方式
1. 实现java.lang.Runnable接口的run方法，然后传递Runnable接口的实现类作为创建Thread时的参数，启动线程。  

```
public class AsyncThreadExample{
    public static void A(){
        try{
            Thread.sleep(2000);
        }catch(InterruptedException e){
            e.printStackTrace();
        }
        System.out.println("----deSomethingA--");
    }

    public static void B(){
        try{
            Thread.sleep(2000);
        }catch(InterruptedException e){
            e.printStackTrance();
        }
        System.out.println("---doSomethingB--");
    }

    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        
        //方式1启动线程执行A
        Thread thread = new Thread( () -> {
            try{
                A();
            }catch(Exception e){
                e.printStackTrace();
            }
        },"threadA");
        thread.start();

        //main线程继续执行B
        B();

        //同步等待线程threadA运行结束
        thread.join();
        
        //打印运行时间
        System.out.println(System.currentTimeMillis() - start);
    }
}
```
<span style="color: red">*</span>注意：    
- 如果去掉同步等待threadA运行结束的代码，则main线程执行完毕后先行死亡。留下threadA线程继续执行（JVM发现有非守护线程在运行就不会停止），等threadA执行完毕后JVM关闭。  

2. 开启线程进行异步执行的方式是实现Thread类，并重写run方法（创建了Thread的匿名类的实现，并重写了run方法，然后启动了线程执行。）  

```
Thread thread = new Thread("threadA") {
    @Override
    public void run() {
        try {
            doSomethingA();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
};
thread.start();
```
<span style="color: red">*</span>总结：  
- Runnable相较于Thread的优势   
1.适合拥有多个相同程序代码的线程去处理同一资源  
2.可以避免Java中的单继承限制  
3.代码可以被多个线程共享  
4.代码和数据实现独立  
5.增加程序的健壮性  
6.线程池只能放入实现Runnable或Callable的类，不能直接放入继承Thread的类  

#### Java三种创建线程的方法
https://blog.csdn.net/u012894692/article/details/80215140

## 线程池

### 向线程池投递一个Callable类型的异步任务，并且获取其执行结果

#### 解读
- Runtime.getRuntime().availableProcessors()  
获取当前物理机的CPU核数  
- new ThreadPoolExecutor(AVALIABLE_PROCESSORS,  //设置线程池核心线程个数为当前物理机的CPU核数  
              AVALIABLE_PROCESSORS * 2,  //最大线程个数为当前物理机CPU核数的2倍   
              1,  //当线程数大于内核数时，这是多余的空闲线程在终止之前等待新任务的最长时间。    
              TimeUnit.MINUTES,   //时间单位  
              new LinkedBlockingQueue<>(5),  //设置线程池阻塞队列的大小为5  
              new NamedThreadFactory("ASYNC-POOL"),  //使用了命名的线程创建工厂，以便排查问题时可以方便追溯是哪个相关业务。   
              new ThreadPoolExecutor.CallerRunsPolicy());  //将线程池的拒绝策略设置为CallerRunsPolicy，即当线程池任务饱和，执行拒绝策略时不会丢弃新的任务，而是会使用调用线程来执行   

```
import java.util.concurrent.*;

public class AsyncThreadPoolExample3 {

    public static String doSomethingA() {
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
        return "A Task Done";
    }

    // 0自定义线程池
    private final static int AVALIABLE_PROCESSORS = Runtime.getRuntime().availableProcessors();
    //设置线程池核心线程个数为当前物理机的CPU核数，最大线程个数为当前物理机CPU核数的2倍；设置线程池阻塞队列的大小为5；
    //需要注意的是，我们将线程池的拒绝策略设置为CallerRunsPolicy，即当线程池任务饱和，执行拒绝策略时不会丢弃新的任务，而是会使用调用线程来执行；
    // 另外我们使用了命名的线程创建工厂，以便排查问题时可以方便追溯是哪个相关业务。
    private final static ThreadPoolExecutor POOL_EXECUTOR = new ThreadPoolExecutor(AVALIABLE_PROCESSORS,
            AVALIABLE_PROCESSORS * 2, 1, TimeUnit.MINUTES, new LinkedBlockingQueue<>(5),
            new NamedThreadFactory("ASYNC-POOL"), new ThreadPoolExecutor.CallerRunsPolicy());

    public static void main(String[] args) throws InterruptedException, ExecutionException {
		// 1.开启异步单元执行任务A
		//使用lambda表达式将Callable类型的任务提交到线程池，提交后会马上返回一个Future对象
        Future<?> resultA = POOL_EXECUTOR.submit(() -> doSomethingA());
        // 2.同步等待执行结果
		//调用get()方法阻塞等待异步任务的执行结果
        System.out.println(resultA.get());
    }
    
}
```

#### 总结
- 自定义拒绝策略  

- 在 Spring 中统一管理线程池  
1. 基础回顾   
@bean 和 @component 的理解  
Spring帮助我们管理Bean分为两个部分，一个是注册Bean，一个装配Bean。   
完成这两个动作有三种方式，一种是使用自动配置的方式、一种是使用 JavaConfig 的方式，一种就是使用XML配置的方式。
在自动配置的方式中，使用 @Component 去告诉 Spring ，我是一个 bean，你要来管理我，然后使用 @Resource(@AutoWired) 注解去装配Bean(所谓装配，就是管理对象直接的协作关系)。然后在 JavaConfig 中，@Configuration 其实就是告诉spring，spring容器要怎么配置（怎么去注册bean，怎么去处理bean之间的关系（装配））。那么就很好理解了，@Bean 的意思就是，我要获取这个bean的时候，你spring要按照这种方式去帮我获取到这个bean。  
用@Bean注解的方法：会实例化、配置并初始化一个新的对象，这个对象会由spring IOC 容器管理。例如：  

```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }

}
```
生成对象的名字：默认情况下用 @Bean 注解的方法名作为对象的名字。但是可以用 name 属性定义对象的名字.    

@component 和 @Configuration 的区别：   
在 @Component 注解的类中不能定义 类内依赖的 @Bean注解的方法。@Configuration 可以。例如：    

```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }

}
```

2. 创建 Bean   

```
/**
 * 所有的线程池类型都从此类获取
 */
@Component
public class AllThreadPool {

    @Bean(name = "rejectThreadPool")
    public ThreadPoolExecutor getRejectExecutor(){
        /**cpu核心线程数*/
        //8 16
        //左移
        //1000   10000
        int coreNum=Runtime.getRuntime().availableProcessors();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                coreNum<<1,
                20,
                120L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(50000),
                new NamedThreadFactory("ASYNC-Pool"),
                new RejectedTaskPolicyWithReport("ASYNC-Pool"));
        return executor;
    }
    
}
```

3. 依赖注入使用   

```
@Resource
private ThreadPoolExecutor rejectThreadPool;
```

## 期间收获  

### 如何正确的终止一个线程  
- 我们可以采用设置一个条件变量的方式，run方法中的while循环会不断的检测flag的值，在想要结束线程的地方将flag的值设置为false就可以啦！注意这里要将flag设置成 volatile 的，因为 volatile 可以保证数据的有效性，如果不设置话，可能会造成子线程多执行一次的错误，例如子线程将flag读到自己线程栈中，flag的值为true，此时子线程的交出执行权，操作系统将执行权交给了主线程，主线程执行flag=false；的操作，希望子线程不要再执行了，但是这一改变子线程是不能看到的，所以子线程还会再向下执行一次，然后重新读取flag的值的时候才会终止。   

```
public class Main {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();
        myThread.start();
        Thread.sleep(3000);
        myThread.flag = false;
        System.out.println("主线程结束！");
    }
}

class MyThread extends Thread {
    public volatile boolean flag = true;

    public void run() {
        while (flag) {
            System.out.println("线程正在执行");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("线程已终止");
    }
}
```
- 当然你也可以将对flag所有操作都封装在synchronized关键字修饰的方法中，实现互斥访问，也可以达到相同的效果。  

### stop方法为什么不安全
其实stop方法天生就不安全，因为它在终止一个线程时会强制中断线程的执行，不管run方法是否执行完了，并且还会释放这个线程所持有的所有的锁对象。这一现象会被其它因为请求锁而阻塞的线程看到，使他们继续向下执行。这就会造成数据的不一致，我们拿银行转账作为例子，从A账户向B账户转账500元，这一过程分为三步，第一步是从A账户中减去500元，假如到这时线程就被stop了，那么这个线程就会释放它所取得锁，然后其他的线程继续执行，这样A账户就莫名其妙的少了500元而B账户也没有收到钱。这就是stop方法的不安全性。  

### main函数是主线程吗
1. 线程的概念：  
线程是程序最基本的运行单位，而进程不能运行，所以能运行的，是进程中的线程。
2. 线程是如何创建起来的：
进程仅仅是一个容器，包含了线程运行中所需要的数据结构等信息。一个进程创建时，操作系统会创建一个线程，这就是主线程，而其他的从线程，却要主线程的代码来创建，也就是由程序员来创建。  
当一个程序启动时，就有一个进程被操作系统（OS）创建，与此同时一个线程也立刻运行，该线程通常叫做程序的主线程（Main Thread），因为它是程序开始时就执行的，如果你需要再创建线程，那么创建的线程就是这个主线程的子线程。每个进程至少都有一个主线程，在Winform中，应该就是创建GUI的线程。  主线程的重要性体现在两方面：1.是产生其他子线程的线程；2.通常它必须最后完成执行比如执行各种关闭动作。  
3. 究竟main函数是进程还是线程  
因为它们都是以main()做为入口开始运行的。　是一个线程,同时还是一个进程。在现在的操作系统中，都是多线程的。但是它执行的时候对外来说就是一个独立的进程。这个进程中，可以包含多个线程，也可以只包含一个线程。当用c写一段程序的话，就是在操作系统中起一个进程它包含一个线程。而当用java等开发一个多线程的程序的话，它在操作系统中起了一个进程，但它可以包含多个同时运行的线程。  

#### 注意
main线程死亡后虚拟机并不会关闭，只有不存在非守护线程时虚拟机才会关闭。  

### 使用JMX连接JVM
- 什么是JMX？  
什么是JMX,Java Management Extensions，即Java管理扩展，是一个为应用程序、设备、系统等植入管理功能的框架。JMX可以跨越一系列异构操作系统平台、系统体系结构和网络传输协议，灵活的开发无缝集成的系统、网络和服务管理应用，详细内容可查看https://www.jianshu.com/p/8c5133cab858  
- JMX使用  
在安装JDK开发工具包后，在bin目录中有jmc.exe、jvisualvm.exe、jconsole.exe,这三个工具都可以提供可视化界面来监控我们的Java程序运行状况，既可以连接本地程序，也可以监控远程环境，使用起来很方便，这里以/Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home/bin/jvisualvm为例。  
- 本地环境  
在Java安装bin目录启动jvisualvm，就可以看到本地监控程序，有启动的IDEA，如果有其他依赖于Java平台运行的程序也都会展示。  
- 远程环境连接   
连接远程环境，需要在Java程序启动的时候添加以下参数  

```
-Dcom.sun.management.jmxremote.port=server_port 远程连接需要开放的端口

-Dcom.sun.management.jmxremote.ssl=false 禁止使用ssl连接

-Dcom.sun.management.jmxremote.authenticate=false 不使用安全认证

-Djava.rmi.server.hostname=server_ip 要连接的远程主机的IP
```
然后启动你的Java程序，可以添加在启动参数文件中，也可以用下面方式启动:

````
java -Dcom.sun.management.jmxremote.port=server_port -Dcom.sun.management.jmxremote.ssl=false  省略...  -jar ./your_jar
````
启动jvisualvm，添加JMX连接，可选不要求SSL连接。若未成功，检查程序是否成功启动，启动参数文件是否使用，防火墙是否开放端口  

### centOS7 查看防火墙状态
 一、防火墙的开启、关闭、禁用命令

（1）设置开机启用防火墙：systemctl enable firewalld.service

（2）设置开机禁用防火墙：systemctl disable firewalld.service

（3）启动防火墙：systemctl start firewalld

（4）关闭防火墙：systemctl stop firewalld

（5）检查防火墙状态：systemctl status firewalld 

二、使用firewall-cmd配置端口

（1）查看防火墙状态：firewall-cmd --state

（2）重新加载配置：firewall-cmd --reload

（3）查看开放的端口：firewall-cmd --list-ports

（4）开启防火墙端口：firewall-cmd --zone=public --add-port=9200/tcp --permanent

　　命令含义：

　　–zone #作用域

　　–add-port=9200/tcp #添加端口，格式为：端口/通讯协议

　　–permanent #永久生效，没有此参数重启后失效

　　注意：添加端口后，必须用命令firewall-cmd --reload重新加载一遍才会生效

（5）关闭防火墙端口：firewall-cmd --zone=public --remove-port=9200/tcp --permanent

### CentOS 7中ifconfig命令找不到了，怎么办？
1. 首先我们在终端中输入：ifconfig，如果输入“bash: ifconfig: 未找到命令”，表示该系统中没有该命令，那么我们就需要安装它。
   
输入：yum install ifconfig，会发现输出了如下错误信息：

没有可用软件包 ifconfig。

错误：无须任何处理  
2. 通过命令：yum search ifconfig，来搜索可用或者匹配的安装包程序。会输入如下信息：
```
   已加载插件：fastestmirror
   
   Loading mirror speeds from cached hostfile
   
    * base: mirrors.tuna.tsinghua.edu.cn
   
    * extras: mirrors.tuna.tsinghua.edu.cn
   
    * updates: mirrors.tuna.tsinghua.edu.cn
   
   =================================================================== 匹配：ifconfig ====================================================================
   
   net-tools.x86_64 : Basic networking tools
```
3. 上面的搜索结果匹配ifconfig的安装包是net-tools.x86_64，这时，可以通过：yum install -y net-tools.x86_64命令来安装ifconfig命令组件了。  
4. 到此我们安装成功  

## Java 与线程





















































