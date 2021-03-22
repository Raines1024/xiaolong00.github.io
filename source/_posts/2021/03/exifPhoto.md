---
title: Java读取文件exif属性
date: 2021-03-22 20:01:39
description: 获取无人机拍摄图像读取经纬度及拍摄时间
tags: [编程,过去]
category:
    - 200 工作类
    - 210 代码类
    - 111 Java

---

## EXIF介绍

EXIF(Exchangeable Image File format)是可交换图像文件的缩写，是专门为数码相机的照片设定的，可以记录数码照片的属性信息和拍摄数据。 EXIF可以附加于JPEG、TIFF、RIFF、RAW等文件之中，为其增加有关数码相机拍摄信息的内容和索引图或图像处理软件的版本信息。

## Java读取文件exif属性

1. 添加依赖

   ```xml
   <dependency>
     <groupId>com.drewnoakes</groupId>
     <artifactId>metadata-extractor</artifactId>
     <version>2.7.2</version>
   </dependency>
   ```

2. Java代码

   ```java
   import com.drew.imaging.ImageMetadataReader;
   import com.drew.imaging.ImageProcessingException;
   
   import java.io.File;
   import java.io.IOException;
   
   /**
    * 实现文件exif属性操作工具类
    */
   public class ExifUtil {
   
   
       public static String[] readExif(File file) throws ImageProcessingException, IOException {
           String[] array = new String[3];
           //如果你对图片的格式有限制，可以直接使用对应格式的Reader如：JPEGImageReader
           ImageMetadataReader.readMetadata(file)
                   .getDirectories().forEach(v ->
                   v.getTags().forEach(t -> {
                       System.out.println(t.getTagName() + " ： " + t.getDescription());
                       switch (t.getTagName()) {
                           //经度
                           case "GPS Longitude":
                               array[0] = t.getDescription();
                               break;
                           //纬度
                           case "GPS Latitude":
                               array[1] = t.getDescription();
                               break;
                           //拍摄时间
                           case "Date/Time Original":
                               array[2] = t.getDescription();
                           default:
                               break;
                       }
                   })
           );
           return array;
       }
   
   }
   ```

   