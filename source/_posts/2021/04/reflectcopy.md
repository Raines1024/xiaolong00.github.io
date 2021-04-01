---
title: Java利用反射实现Spring中BeanUtils类的copyProperties方法
date: 2021-04-01 19:01:39
description: 利用反射机制对JavaBean的属性进行简单的复制值操作
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java
---

## 背景

实际使用`org.springframework.beans.BeanUtils#copyProperties(java.lang.Object, java.lang.Object)`方法时，总是有各种不适用的地方，比如说我不想复制为空的值，或者我想把`String`类型的值转换后赋值给`Date`类型的值，那么我们就可以自己通过反射来实现此功能，而且可以根据自身的业务情况自定义功能。

## 反射复制代码

建议使用`copyFiled3`方法，不过还是有很多问题的：

1. 不能给final对象赋值，如果实体类中有final属性会抛异常
2. 字段相同，类型不相同无法赋值：比如String类型的时间串无法给Date赋值

```java
import lombok.Data;
import org.springframework.util.Assert;

import java.lang.reflect.Field;
import java.util.*;

public class BeanUtil {

 	  //分别利用不用方法执行对象复制
    public static void main(String[] args) {

        User1 u1 = new User1();
        u1.setUserId(1);
        u1.setUserName("raines");
        u1.setDate(new Date());
        List list = new ArrayList();
        list.add("oooooooo");
        u1.setList(list);

        long start, end;

        User2 u3 = new User2();
        start = System.currentTimeMillis();
        copyFiled(u1, u3);
        end = System.currentTimeMillis();
        System.out.println("用时" + (end - start) + "毫秒");

        User2 u4 = new User2();
        start = System.currentTimeMillis();
        copyFiled2(u1, u4);
        end = System.currentTimeMillis();
        System.out.println("用时" + (end - start) + "毫秒");


        User2 u5 = new User2();
        start = System.currentTimeMillis();
        copyFiled3(u1, u5);
        end = System.currentTimeMillis();
        System.out.println("用时" + (end - start) + "毫秒");


        User2 u2 = new User2();
        start = System.currentTimeMillis();
        org.springframework.beans.BeanUtils.copyProperties(u1, u2);
        end = System.currentTimeMillis();
        System.out.println("BeanUtils.copyProperties用时" + (end - start) + "毫秒");
    }

    /**
     * 利用两个数组迭代复制
     */
    public static void copyFiled(Object source, Object target) {

        Assert.notNull(source, "Source must not be null");
        Assert.notNull(target, "Target must not be null");

        Field[] sourceFields = source.getClass().getDeclaredFields();
        Field[] targetFields = target.getClass().getDeclaredFields();

        Object value;
        int count = 0;
        try {
            for (Field field : sourceFields) {
                for (Field field2: targetFields) {
                    count++;
                    if (field.getName().equals(field2.getName()) && field.getType().equals(field2.getType())) {

                        field.setAccessible(true);
                        value = field.get(source);
                        if (null == value || "" == value) {
                            continue;
                        }
                        field2.setAccessible(true);
                        field2.set(target, value);
                    }
                }
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        System.out.println("迭代次数: " + count);
    }

    /**
     * 利用一个数组和一个链表迭代复制
     */
    public static void copyFiled2(Object source, Object target) {

        Assert.notNull(source, "Source must not be null");
        Assert.notNull(target, "Target must not be null");

        Field[] sourceFields = source.getClass().getDeclaredFields();
        Field[] targetFields = target.getClass().getDeclaredFields();

        List<Field> link = new LinkedList<>(Arrays.asList(targetFields));
        Object value;
        int count = 0;
        Field field2;
        int len;
        try {
            for (Field field : sourceFields) {
                len = link.size();
                for (int i = 0; i < len; i++) {
                    count++;
                    field2 = link.get(i);
                    if (field.getName().equals(field2.getName()) && field.getType().equals(field2.getType())) {

                        field.setAccessible(true);
                        value = field.get(source);
                        if (null == value || "" == value) {
                            continue;
                        }
                        field2.setAccessible(true);
                        field2.set(target, value);
                        link.remove(i);
                        break;
                    }
                }
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        System.out.println("迭代次数: " + count);
    }

    /**
     * 利用一个数组和一个HashMap迭代复制
     */
    public static void copyFiled3(Object source, Object target) {

        Assert.notNull(source, "Source must not be null");
        Assert.notNull(target, "Target must not be null");

        Field[] sourceFields = source.getClass().getDeclaredFields();
        Field[] targetFields = target.getClass().getDeclaredFields();

        Map<String, Field> fieldMap = new HashMap<>(targetFields.length);
        for (Field targetField : targetFields) {
            fieldMap.put(targetField.getName(), targetField);
        }
        Object value;
        int count = 0;
        Field field2;
        try {
            for (Field field : sourceFields) {
                count++;
                field2 = fieldMap.get(field.getName());
                if (field.getType().equals(field2.getType())) {

                    field.setAccessible(true);
                    value = field.get(source);
                    if (null == value || "" == value) {
                        continue;
                    }
                    field2.setAccessible(true);
                    field2.set(target, value);
                }
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        System.out.println("迭代次数: " + count);
    }
}

@Data
class User1 {
    private Date date;
    private Integer userId;
    private Date date;
    private String userName;
    private List list;
}
@Data
class User2 {
    private Integer userId;
    private String userName;
    private Date date;
    private List list;
}
```

## 现在使用的代码

现在使用主要是为解决修改实体类时Spring的`copyProperties`方法会把null值和空字符串赋值给相应的字段，然后顺带把时间类型转换一下。

```java
    public static void copyProperties(Object source, Object target,String format) {

        Assert.notNull(source, "Source must not be null");
        Assert.notNull(target, "Target must not be null");

        Field[] sourceFields = source.getClass().getDeclaredFields();
        Field[] targetFields = target.getClass().getDeclaredFields();

        HashMap<String, Field> fieldMap = new HashMap<>(targetFields.length);
        for (Field targetField : targetFields) {
            //跳过final字段
            if (Modifier.toString(targetField.getModifiers()).contains("final")){
                continue;
            }
            fieldMap.put(targetField.getName(), targetField);
        }
        Object value;
        Field field2;
        try {
            for (Field field : sourceFields) {
                if (Modifier.toString(field.getModifiers()).contains("final")){
                    continue;
                }
                field2 = fieldMap.get(field.getName());
                if (field.getType().equals(field2.getType())) {

                    field.setAccessible(true);
                    value = field.get(source);
                    if (null == value || "" == value) {
                        continue;
                    }
                    field2.setAccessible(true);
                    field2.set(target, value);
                }else if (field.getType().equals(Date.class) && field2.getType().equals(String.class)){
                    field.setAccessible(true);
                    value = field.get(source);
                    if (null == value || "" == value) {
                        continue;
                    }
                    field2.setAccessible(true);
                    field2.set(target, $.getDateString(((Date) value).getTime(),format));
                }else if (field.getType().equals(String.class) && field2.getType().equals(Date.class)){
                    field.setAccessible(true);
                    value = field.get(source);
                    if (null == value || "" == value) {
                        continue;
                    }
                    field2.setAccessible(true);
                    field2.set(target, $.getDateForString((String) value,format));
                }
            }
        } catch (ParseException e) {
            log.error("日期转换异常",e);
            log.error("源:"+JSON.toJSONString(source)+"目的:"+JSON.toJSONString(target));
        }catch (IllegalAccessException e){
            log.error("反射异常",e);
        }
    }

```

## 拓展：反射给final属性赋值（不建议使用）

### 方案

使用Java反射，通过 `Field#setAccessible(true)` 将 private 修饰的字段变为 accessible；再将 final 修饰符去掉；最后再设置新值即可。当然，如果涉及到 Java 内联优化，则会失效。具体见示例代码：

```java
public class ChangeStaticFinalFieldSample {

    static void changeStaticFinal(Field field, Object newValue) throws Exception {
        field.setAccessible(true); // 如果field为private,则需要使用该方法使其可被访问

        Field modifersField = Field.class.getDeclaredField("modifiers");
        modifersField.setAccessible(true);
        // 把指定的field中的final修饰符去掉
        modifersField.setInt(field, field.getModifiers() & ~Modifier.FINAL);

        field.set(null, newValue); // 为指定field设置新值
    }

    public static void main(String[] args) throws Exception {
        Sample.print();

        Field canChangeField = Sample.class.getDeclaredField("CAN_CHANGE");
        Field cannotChangeField = Sample.class.getDeclaredField("CANNOT_CHANGE");
        Field strField = Sample.class.getDeclaredField("STR");
        changeStaticFinal(canChangeField, 2);
        changeStaticFinal(cannotChangeField, 3);
        changeStaticFinal(strField, "success");

        Sample.print();
    }
}

class Sample {
    private static final int CAN_CHANGE = new Integer(1); // 未内联优化
    private static final int CANNOT_CHANGE = 1; // 内联优化
    private static final String STR = "str";


    public static void print() {
        System.out.println("CAN_CHANGE = " + CAN_CHANGE);
        System.out.println("CANNOT_CHANGE = " + CANNOT_CHANGE);
        System.out.println("STR = " + STR);
        System.out.println("------------------------");
    }
}
```

`CAN_CHANGE` 和 `CANNOT_CHANGE` 字段同属于 final 修饰符修饰的常量字段，但是由于 `CANNOT_CHANGE` 常量在 Java 编译过程中使用了内联优化，其值在编译阶段就被编译为常量值 `1`，故而使用内联优化的 final 字段更改其值是无效的； 而 `CAN_CHANGE` 字段未被内联优化，故而能通过 Java 反射对其值进行修改。

如果把`private static final String STR = "str";`修改为`private static final String STR = new String("str");`，则能通过 Java 反射对其值进行修改。

























