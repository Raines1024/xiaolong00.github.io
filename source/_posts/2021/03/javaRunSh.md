---
title: Java调用Linux系统命令
date: 2021-03-25 20:01:39
description: 调用Linux系统命令并返回执行结果
tags: [编程,代码,过去]
category:
    - 200 工作类
    - 210 代码类
    - 111 Java

---

## 背景

之前解压zip包的时候，想过调用系统命令来完成解压、获取文件等操作，因为懒也拖了好几天今日才看，不过这只是保底的方案，在系统中使用不利于扩展和维护。

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

