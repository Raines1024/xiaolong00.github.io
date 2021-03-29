---
title: Java异常日志打印那些事
date: 2021-03-29 20:01:39
description: 服务器日志不显示详细错误信息的解决方案
tags: [编程,过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java
---

## 背景

服务器上的shell启动脚本中，通过在启动命令后紧跟着这个命令:` > /dev/null 2>&1`，这个命令将标准输出和错误输出全部重定向到/dev/null中，也就是将产生的所有信息丢弃。
许多开发人员在catch代码块中还是习惯于写`e.printStackTrace();`来定位问题代码的具体位置，我们接下来再讨论，这是一个打印异常日志严重错误的使用方式。
这就产生了我们今天的问题：怎么正确输出异常日志呢？

## getStackTrace和printStackTrace的区别
在JAVA中收到程序报错，将堆栈信息打印出来是一个好习惯，但是在catch到exception之后，发现有两个方法都和堆栈信息有关，一个是getStackTrace，一个是printStackTrace，那么他们的区别是什么？
我们从源码的角度来分析：

1. getStackTrace()

   ```java
     public StackTraceElement[] getStackTrace() {
           return getOurStackTrace().clone();
       }
    
       private synchronized StackTraceElement[] getOurStackTrace() {
           // Initialize stack trace field with information from
           // backtrace if this is the first call to this method
           if (stackTrace == UNASSIGNED_STACK ||
               (stackTrace == null && backtrace != null) /* Out of protocol state */) {
               int depth = getStackTraceDepth();
               stackTrace = new StackTraceElement[depth];
               for (int i=0; i < depth; i++)
                   stackTrace[i] = getStackTraceElement(i);
           } else if (stackTrace == null) {
               return UNASSIGNED_STACK;
           }
           return stackTrace;
       }
   ```

2. printStackTrace()

   ```java
    public void printStackTrace() {
           printStackTrace(System.err);
       }
    
       /**
        * Prints this throwable and its backtrace to the specified print stream.
        *
        * @param s {@code PrintStream} to use for output
        */
       public void printStackTrace(PrintStream s) {
           printStackTrace(new WrappedPrintStream(s));
       }
    
       private void printStackTrace(PrintStreamOrWriter s) {
           // Guard against malicious overrides of Throwable.equals by
           // using a Set with identity equality semantics.
           Set<Throwable> dejaVu =
               Collections.newSetFromMap(new IdentityHashMap<Throwable, Boolean>());
           dejaVu.add(this);
    
           synchronized (s.lock()) {
               // Print our stack trace
               s.println(this);
               StackTraceElement[] trace = getOurStackTrace();
               for (StackTraceElement traceElement : trace)
                   s.println("\tat " + traceElement);
    
               // Print suppressed exceptions, if any
               for (Throwable se : getSuppressed())
                   se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION, "\t", dejaVu);
    
               // Print cause, if any
               Throwable ourCause = getCause();
               if (ourCause != null)
                   ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
           }
       }
   ```

getStackTrace()返回的是通过getOurStackTrace方法获取的StackTraceElement[]数组，而这个StackTraceElement是ERROR的每一个cause by的信息。

printStackTrace()返回的是一个void值，但是可以看到其方法内部将当前传入打印流锁住，然后同样通过getOurStackTrace方法获取的StackTraceElement[]数组，只不过printStackTrace()方法直接打印出来了。而getStackTrace()则是得到数组，使用者可以根据自己的需求去得到打印信息，相比printStackTrace()会更细一些。

这里值得一提的是slf4j的log方法，如果放入Exception对象，则会自动打印出其堆栈信息，不必要再专门去转流写入日志。

## 代码

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Main {

    protected final  static Logger log = LoggerFactory.getLogger(Main.class);

    public static void main(String[] args) {
        indexOutOfBounds();
        System.out.println("===============");
        nullPointer();
    }

    private static void nullPointer(){
        try {
            List list = null;
            list.get(0);
        } catch (Exception e) {
            log.error("xx报错",e);
            System.out.println(e.getMessage());
            System.out.println(e.toString());
            System.out.println(Arrays.toString(e.getStackTrace()));
        }
    }

    private static void indexOutOfBounds(){
        try {
            List list = new ArrayList();
            list.get(0);
        } catch (Exception e) {
            log.error("xx报错",e);
            System.out.println(e.getMessage());
            System.out.println(e.toString());
            System.out.println(Arrays.toString(e.getStackTrace()));
        }
    }

}
```









