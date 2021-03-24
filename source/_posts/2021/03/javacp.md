---
title: Java版复制文件
date: 2021-03-23 20:01:39
description: 通过代码复制文件到指定目录
tags: [编程,代码,过去]
category:
    - 200 工作类
    - 210 代码类
    - 111 Java

---

## 背景

无人机拍摄的图片经过解压后需要存储到nginx转发的图片文件夹下，思路是把本地文件转为MultipartFile类型，然后再存储到目标路径上。

## 代码

```java
import org.apache.commons.fileupload.FileItem;
import org.apache.commons.fileupload.FileItemFactory;
import org.apache.commons.fileupload.disk.DiskFileItemFactory;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;

public class FileUtil {

    /**
     * 示例
     */
    public static void main(String[] args) {
        //复制文件
        upload(getMulFileByPath("/Users/raines/Desktop/my/test_upload/test_upload/100.jpg"),"/Users/raines/Desktop/my/test_upload/test_upload/demo.jpg");
    }

    /**
     * 本地文件转为MultipartFile类型 
     * @param picPath 文件路径
     */
    public static MultipartFile getMulFileByPath(String picPath) {
        FileItem fileItem = createFileItem(picPath);
        MultipartFile mfile = new CommonsMultipartFile(fileItem);
        return mfile;
    }

    private static FileItem createFileItem(String filePath)
    {
        FileItemFactory factory = new DiskFileItemFactory(16, null);
        String textFieldName = "textField";
        int num = filePath.lastIndexOf(".");
        String extFile = filePath.substring(num);
        FileItem item = factory.createItem(textFieldName, "text/plain", true,
                "MyFileName" + extFile);
        File newfile = new File(filePath);
        long fileSize = newfile.length();
        int bytesRead = 0;
        byte[] buffer =new byte[(int) fileSize];
        try
        {
            FileInputStream fis = new FileInputStream(newfile);
            OutputStream os = item.getOutputStream();
            while ((bytesRead = fis.read(buffer, 0,  buffer.length))!= -1)
            {
                os.write(buffer, 0, bytesRead);
            }
            os.close();
            fis.close();
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
        return item;
    }
    
    /**
     * 保存文件
     * @param file 文件
     * @param pathName 文件路径
     * @return
     */
    public static boolean upload(MultipartFile file, String pathName) {

        File tempFile = new File(pathName);


        try {
            //判断文件父目录是否存在
            if (!tempFile.getParentFile().exists()) {
                tempFile.getParentFile().mkdirs();//创建父级文件路径
                tempFile.createNewFile();//创建文件
            }
            //保存文件
            file.transferTo(tempFile);
            return true;
        } catch (IllegalStateException e) {
            e.printStackTrace();
            return false;
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }

    }

}
```

