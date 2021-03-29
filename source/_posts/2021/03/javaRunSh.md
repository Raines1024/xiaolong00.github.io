---
title: Java调用Linux系统命令
date: 2021-03-25 20:01:39
description: 调用Linux系统命令并返回执行结果
tags: [编程,代码,过去]
category:
    - 200 工作类
    - 210 代码类
    - 211 后端代码

---

## 背景

之前解压zip包的时候，想过调用系统命令来完成解压、获取文件等操作，因为懒也拖了好几天今日才看，不过这只是保底的方案，在系统中使用不利于扩展和维护。

##简介

### Java方法

java调用操作系统命令主要使用到Process以及ProcessBuilder这两个类。

Process：JDK1.5之前调用系统命令广泛使用的一个抽象类，一般都是通过Runtime.exec()和ProcessBuilder.start()来间接创建其实例，但是其功能相对较少。

ProcessBuilder：JDK1.5之后出现的一个final类，有两个带参数的构造方法，可以通过构造方法来直接创建其对象，为进程提供了更多功能(可获取/设置系统的环境变量、设置命令执行路径)，但是其不是同步的。

### sh命令

**sh命令**是shell命令语言解释器，执行命令从标准输入读取或从一个文件中读取。通过用户输入命令，和内核进行沟通！Bourne Again Shell （即bash）是自由软件基金会（GNU）开发的一个Shell，它是Linux系统中一个默认的Shell。Bash不但与Bourne Shell兼容，还继承了C Shell、Korn Shell等优点。

- 语法 

  ```sh
  bash [options] [file]
  ```

- 选项 

  ```
  -c string：命令从-c后的字符串读取。
  -i：实现脚本交互。
  -n：进行shell脚本的语法检查。
  -x：实现shell脚本逐条语句的跟踪。
  ```

## 代码

```java
public class Main {

    public static void main(String[] args) throws IOException {
        //调用linux含有管道的复杂shell
        shell("ps -ef | grep java | grep -v grep | wc -l");
        //调用linux简单的shell语句
        shell("ls");
    }

    /**
     * 使用sh -c执行Linux系统命令
     * @param execution 要执行的shell
     * @throws IOException
     */
    public static void shell(String execution) throws IOException {
        Process process = null;
        BufferedReader input = null;
        try {
            ProcessBuilder builder = new ProcessBuilder("sh", "-c", execution);
            //设置在/Users/raines目录下执行命令
            builder.directory(new File("/Users/raines"));
            builder.redirectErrorStream(true);
            process = builder.start();
            input = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String str = null;
            while ((str = input.readLine()) != null) {
                //打印返回执行结果
                System.out.println(str);
            }
            //Map<String, String> map = builder.environment();           //获得进程的环境
        } catch (Exception e) {
            //异常操作
            e.printStackTrace();
        } finally {
            //销毁process（process.destroy()）以及关闭流
            if (input != null) input.close();
            if (process != null) process.destroy();
        }
    }

}

```

