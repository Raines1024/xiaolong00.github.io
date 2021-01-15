---
layout: post
title: Java自定义注解
description: Java自定义注解基础知识
category: blog
date: 2020-01-07 13:50:39
---

## Java自定义注解理论  

### 介绍
注解相当于一种标记，在程序中加了注解就等于为程序打上了某种标记，没加，则等于没有某种标记，以后，javac编译器，开发工具和其他程序可以用反射来了解你的类及各种元素上有无何种标记，看你有什么标记，就去干相应的事。标记可以加在包，类，字段，方法，方法的参数以及局部变量上。  

### 元注解
Java中提供了四种元注解，专门负责注解其他的注解，分别如下:    
- @Retention元注解，表示需要在什么级别保存该注释信息（生命周期）。可选的RetentionPoicy参数包括：  
RetentionPolicy.SOURCE: 停留在java源文件，编译器被丢掉   
RetentionPolicy.CLASS：停留在class文件中，但会被VM丢弃（默认）  
RetentionPolicy.RUNTIME：内存中的字节码，VM将在运行时也保留注解，因此可以通过反射机制读取注解的信息   
- @Target元注解，默认值为任何元素，表示该注解用于什么地方。可用的ElementType参数包括   
ElementType.CONSTRUCTOR: 构造器声明  
ElementType.FIELD: 成员变量、对象、属性（包括enum实例）  
ElementType.LOCAL_VARIABLE: 局部变量声明   
ElementType.METHOD: 方法声明   
ElementType.PACKAGE: 包声明   
ElementType.PARAMETER: 参数声明   
ElementType.TYPE: 类、接口（包括注解类型)或enum声明   
- @Documented注解将注解包含在JavaDoc中   
- @Inheried注解允许子类继承父类中的注解   

### 注解种类
java的自定义注解可以分为三类：没有任何元素的注解，有一个元素的注解和有多个元素的注解。  
1. Marker注解  
这类注解没有任何元素，此类注解仅仅是一个标示。  
2. 单值注解  
只接受单值类型，数据成员使用单词value指定。指定成员的语法与声明方法类似。  
但是如果数据成员不使用value定义，新定义如下所示：  

```
    public @interface Good{
       String description();
    }
```
现在，需要使用下面的注解方式

```
    @Good(description="this good")
```
注意：数据成员使用默认名称value时候，我们只指定了目标字符串，而省略了成员名称，这次我们需要显示拼写出数据成员的名称description，如果不这么做，编译器将会在编译过程中产生错误。  
3. 多值注解   

### 设置默认值  
java允许为任何数据成员指定默认值，这可以使用default关键字来完成。  
当使用默认值注解的时候，target成员可以不指定，除非想为target设置不同的值。  

### 注解的定义规则   
定义一个注解还是很简单的，需要遵照以下几个规则就可以了：  
（1）注解声明以@interface开设，随后是注解的名称。  
（2）为了创建注解的参数，需要使用参数的类型声明方法：  
方法声明不应包含任何参数；  
方法声明不应包含任何throws子句；  
方法的返回类型应该为：基本类型，字符串，类，枚举，上述类型的数组。  

## Java自定义注解实战

### 利用java反射和java自定义注解验证数据的完整性  
JDK1.5及以后版本引入的java自定义注解，可以应用到反射中，比如自己写个小框架。如实现实体类某些属性不自动赋值，或者验证某个对象属性完整性等，下面具体说说使用注解对实体数据进行非空校验的过程。  
1. 首先自定义非空注解NotEmpty    

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface NotEmpty {
}
```  
2. 其次定义一个实体类Person，并在部分属性上面加上注解@NotEmpty    

```
public class Person {
    @NotEmpty
    private Integer id;
    @NotEmpty
    private int age;
    @NotEmpty
    private String name;
    @NotEmpty
    private String sex;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```   
3. 编写测试类Main  

```
import java.lang.reflect.Field;

/**
 * 测试类
 */
public class Main {
    public static void main(String[] args) {
        try {
            Person person = new Person();
            person.setId(1);
            person.setAge(50);
            person.setName("张三");
            person.setAddress("北京");
//            person.setSex("male");
            validateParam(person);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 验证数据完整性
     *
     * @param person
     * @throws Exception
     */
    public static void validateParam(Person person) throws Exception{
        Class<?> personClass = (Class<?>)person.getClass();
        //获取该类所有属性
        Field[] field = personClass.getDeclaredFields();

        for (int i = 0; i < field.length; i++) {
            Field f= field[i];
            f.setAccessible(true);
//            System.out.println(f.isAnnotationPresent(NotEmpty.class)+";"+f.getName());
            if (f.getAnnotation(NotEmpty.class)!= null) {
                //获取属性值
                Object attrValue=f.get(person);
                if(attrValue==null||attrValue.toString().trim().equals("")){
//                    throw new RuntimeException(f.getName()+"属性值不能为空");
                    System.out.println(f.getName()+"属性值不能为空");
                }
                continue;
            }
        }
    }
}
```

### 类使用多值注解并获取属性  
1. 首先写一个自定义注解@MyAnnotation   

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.METHOD, ElementType.TYPE })
public @interface MyAnnotation {
    // 为注解添加属性
    String color();

    String value() default "我是XXX"; // 为属性提供默认值

    int[] array() default { 1, 2, 3 };

    Gender gender() default Gender.MAN; // 添加一个枚举

    // 添加枚举属性
    MetaAnnotation metaAnnotation() default @MetaAnnotation(birthday = "我的出生日期为1995-8-8");
}
```
2. 写一个枚举类Gender，模拟注解中添加枚举属性  

```
public enum Gender {
    MAN {
        public String getName() {
            return "男";
        }
    },
    WOMEN {
        public String getName() {
            return "女";
        }
    }; // 后面记得有“;”
    public abstract String getName();
}
```
3. 写一个注解类MetaAnnotation，模拟注解中添加注解属性    

```
public @interface MetaAnnotation {
    String birthday();
}
```
4. 最后写注解测试类AnnotationTest  

```
//调用注解并赋值
@MyAnnotation(metaAnnotation = @MetaAnnotation(birthday = "我的出生日期为1995-8-88"), color = "red", array = {23, 26})
public class AnnotationTest {
    public static void main(String[] args) {
        // 检查类AnnotationTest是否含有@MyAnnotation注解
        if (AnnotationTest.class.isAnnotationPresent(MyAnnotation.class)) {
            // 若存在就获取注解
            MyAnnotation annotation = (MyAnnotation) AnnotationTest.class.getAnnotation(MyAnnotation.class);
            System.out.println(annotation);
            // 获取注解属性
            System.out.println(annotation.color());
            System.out.println(annotation.value());
            // 数组
            int[] arrs = annotation.array();
            for (int arr : arrs) {
                System.out.println(arr);
            }
            // 枚举
            Gender gender = annotation.gender();
            System.out.println("性别为：" + gender);
            // 获取注解属性
            MetaAnnotation meta = annotation.metaAnnotation();
            System.out.println(meta.birthday());
        }
    }
}
```

## Java8可重复注解的理解与应用

### 参考：  
基于 隔叶黄莺 的博文修改，重新实现代码  
https://yanbin.blog/java8-repeatable-annotations/


### 可重复注解实战
1. 定义 @Log 的容器注解 @Logs   

```
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)//注解会在class中存在，运行时可通过反射获取
@Documented//文档生成时，该注解将被包含在javadoc中，可去掉
@Target(ElementType.METHOD)//目标是方法
public @interface Logs {
    Log[] value();
}
```

2. @Repeatable 是 Java8 开始提供的，现在由它来告诉 Java @Logs 是 @Log 的容器注解，不需要用嵌套的方式来使用它们了，可以在一个类型重复使用同一个注解。   

```
@Retention(RetentionPolicy.RUNTIME)//注解会在class中存在，运行时可通过反射获取
@Target(ElementType.METHOD)//目标是方法
@Documented//文档生成时，该注解将被包含在javadoc中，可去掉
@Repeatable(Logs.class)//这行建立了@Log和@Logs的关系
public @interface Log {

    String value() default "";
    /**
     * 模块名字
     */
    String modelName() default "";

    /**
     * 操作类型
     */
    String option() default "";

}
```

3. 反射获得注解内容   
Java8 为了避免在使用重复注解时的编码与反射时的尴尬，引入了一个新的反射注解的 API getAnnotationByType(Class<A> annotationClass), 该 API 的参数可接受 Log.class, 并且返回一个 @Log 的数组。   
<span style="color: red">*</span> 解释 method.getAnnotationsByType(Log.class) 的执行过程:  
如果能找到 @Repeatable 关联的容器注解类 @Logs, 就获得 @Logs 的所有 value(类型为 @Log) 值组成的数组；  
如果未有关联的容器注解类，则返回 @Log 本身组成的数组(只有一个元素), 此时和 method.getAnnotation(Log.class) 是一样的。  
因此，在 Java 8 中对于可重复注解应该调用 getAnnotationsByType(Class<A> annotationClass) 来反射得到，如果是不可重复注解建议还是调用原来的 getAnnotation(Class<A> annotationClass), 因为没必要使用 getAnnotationsByType(Class<A> annotationClass) 获得一个空的或 1 个元素的数组。  

最终 Java 8 推荐我们反射重复注解的途径就是下面那样:  

```
import java.lang.reflect.Method;

public class RepeatTest {

    @Log(modelName = "demo", option = "test", value = "ds")
    @Log(modelName = "demo2", option = "test", value = "ds")
    private static boolean demo() {
        System.out.println("demo Method.");
        return true;
    }

    public static void main(String[] args) {
        RepeatTest repeatTest = new RepeatTest();
        Class<?> repeatTestClass = (Class<?>) repeatTest.getClass();
        //获取该类所有属性
        Method[] methods = repeatTestClass.getDeclaredMethods();
        for (int i = 0; i < methods.length; i++) {
            Method method = methods[i];
            Log[] logs = method.getAnnotationsByType(Log.class);
            for (Log log : logs){
                if (log == null) continue;
                System.out.println(log.modelName());
            }
        }
    }

}
```

#### 注意
实际上在字节码中 Java8 前后对重复注解的内部实现也确实是一样的，@Repeatable 就是个语法糖而已。



















