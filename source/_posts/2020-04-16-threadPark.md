---
layout: post
title: 线程阻塞与唤醒
description: 多线程与异步系列外传
category: blog
---

## 线程阻塞与唤醒
### 方式一（@Deprecated）
早期JAVA采用suspend()、resume()对线程进行阻塞与唤醒，但这种方式产生死锁的风险很大，因为线程被挂起以后不会释放锁，可能与其他线程、主线程产生死锁，suspend 它会一直保持对锁的占有，一直到其他的线程调用resume方法，它才能继续向下执行。   


- 演示 suspend 方法为什么被弃用  
1).独占：因为 suspend 在调用过程中不会释放所占用的锁，所以如果使用不当会造成对公共对象的独占，使得其他线程无法访问公共对象，严重的话造成死锁。  
假如有A，B两个线程，A线程在获得某个锁之后被suspend阻塞，这时A不能继续执行，线程B在获得相同的锁之后才能调用resume方法将A唤醒，但是此时的锁被A占有，B不能继续执行，也就不能及时的唤醒A，此时A，B两个线程都不能继续向下执行而形成了死锁。这就是suspend被弃用的原因。  
如果把 System.out.println("I'm OK."); 注掉，则会输出 suspend complete? ；否则因为println函数，suspend挂起后的线程没有释放锁，导致其他线程在调用println函数时出现阻塞

```
public class ThreadSuspendTest {
    public static void main(String[] args) {
        Thread a = new MyThread();
        a.start();
        try {
            Thread.currentThread().sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        a.suspend();
        System.out.println("suspend complete?");
        a.resume();
        System.err.println("--");
    }

    static class MyThread extends Thread {
        public void run() {
            while (true) {
                System.out.println("I'm OK.");
            }
        }
    }
}
```
2).不同步：容易造成因线程暂停而导致的数据不同步  

### 方式二
- wait、notify形式通过一个object作为信号，object的wait()方法是锁门的动作，notify()、notifyAll()是开门的动作，某一线程一旦关上门后其他线程都将阻塞，直到别的线程打开门。notify()准许阻塞的一个线程通过，notifyAll()允许所有线程通过。   
演示代码：主线程分别启动两个线程，随后通知子线程暂停等待，再逐个唤醒后线程抛异常退出。

```
public class ObjectWaitTest {
    public static Object waitObject = new Object();

    public static void notifyAllThread() {
        System.out.println("notifyAllThread");
        synchronized (waitObject) {
            waitObject.notifyAll();
        }
    }

    public static void notifyThread() {
        System.out.println("notifyThread");
        synchronized (waitObject) {
            waitObject.notify();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        MyThread tm1 = new MyThread(waitObject);
        tm1.setName("tm1");
        tm1.start();
        MyThread tm2 = new MyThread(waitObject);
        tm2.setName("tm2");
        tm2.start();
        Thread.currentThread().sleep(1000);
        tm1.suspendThread();
        tm2.suspendThread();
        Thread.currentThread().sleep(1000);
        notifyThread();
        Thread.currentThread().sleep(1000);
        notifyThread();
    }

}

class MyThread extends Thread {
    public Object waitObject = null;
    private boolean isStop = false;

    public MyThread(Object waitObject) {
        this.waitObject = waitObject;
    }

    public void run() {
        while (true) {
            synchronized (waitObject) {
                if (isStop) {
                    System.out.println(Thread.currentThread().getName() + " is stop");
                    try {
                        waitObject.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + " is resume");
                    System.out.println(Thread.currentThread().getName() + " will  exit");
                    throw new RuntimeException(Thread.currentThread().getName() + " exit");
                }
            }
        }
    }

    public void suspendThread() {
        this.isStop = true;
    }
}
```

- wait、notify使用要点：  
1、对象操作都需要加同步synchronized；  
2、线程需要阻塞的地方调用对象的wait方法；  
存在的不足：面向对象的阻塞是阻塞当前线程，而唤醒的是随机的一个线程或者所有线程，偏重线程间的通信；同时某一线程在被另一线程notify之前必须要保证此线程已经执行到wait等待点，错过notify则可能永远都在等待。  

### 方式三
LockSupport 提供的 park 和 unPark 方法，提供避免死锁和竞态条件，很好地代替 suspend 和 resume 组合。  
park 与 unPark 方法控制的颗粒度更加细小，能准确决定线程在某个点停止，进而避免死锁的产生。  
park 与 unPark 引入了许可机制，许可逻辑为：  
①park 将许可在等于0的时候阻塞，等于1的时候返回并将许可减为0；  
②unPark 尝试唤醒线程，许可加1。根据这两个逻辑，对于同一条线程，park 与 unPark 先后操作的顺序似乎并不影响程序正确地执行，假如先执行 unPark 操作，许可则为1，之后再执行park操作，此时因为许可等于1直接返回往下执行，并不执行阻塞操作。  
park 与 unPark 组合真正解耦了线程之间的同步，不再需要另外的对象变量存储状态，并且也不需要考虑同步锁，wait与notify要保证必须有锁才能执行，而且执行notify操作释放锁后还要将当前线程扔进该对象锁的等待队列，LockSupport则完全不用考虑对象、锁、等待队列等问题。  

```
public class ThreadParkTest {
    public static void main(String[] args) throws Exception {
        Thread t = new Thread(() -> {
            System.out.println("start");
            LockSupport.park(); //一直wait
            System.out.println("continue");
        });
        t.start();

        Thread.sleep(1000);
        LockSupport.unpark(t); //指定t线程解除wait态
    }
}
```

### 总结
suspend()、resume()已经被deprecated，不建议使用。wait、notify需要对对象加同步，性能有折扣。LockSupport则完全不用考虑对象、锁、等待队列。
