---
title: Java解压zip压缩包并返回文件列表
date: 2021-03-22 20:01:39
description: 获取图片压缩包内无人机拍摄图像
tags: [编程,代码,过去]
category:
    - 200 工作类
    - 210 代码类
    - 211 后端代码

---

## 解压zip压缩包并返回解压之后所得到的文件List

### 使用Java util包（推荐使用）

使用Java util包中打包的文件使用Apache的工具类解压失败，该工具类能正常解压Java压缩的zip包及手动压缩的zip包，故推荐使用该工具类。

```java
import java.io.*;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;

public class UnzipUtil {

    private static final int BUFFER_LEN = 8192;

    public static void main(String[] args) {
        String zipFilePath = "/Users/raines/Desktop/my/test_upload.zip";
        String unzipDestPath = "/Users/raines/Desktop/my/";

        List<File> files = null;
        try {
            files = unZipFile(new File(zipFilePath), new File(unzipDestPath));
        } catch (IOException e) {
            e.printStackTrace();
        }
        for (File file : files) {
            System.out.println(file.getPath());
        }

        System.out.println(files.size());
    }

    /**
     * Unzip the file by keyword.
     *
     * @param zipFile The ZIP file.
     * @param destDir The destination directory.
     * @return the unzipped files
     * @throws IOException if unzip unsuccessfully
     */
    public static List<File> unZipFile(final File zipFile, final File destDir)
            throws IOException {
        if (zipFile == null || destDir == null) return null;
        List<File> files = new ArrayList<>();
        ZipFile zip = new ZipFile(zipFile);
        Enumeration<?> entries = zip.entries();
        try {
            while (entries.hasMoreElements()) {
                ZipEntry entry = ((ZipEntry) entries.nextElement());
                String entryName = entry.getName().replace("\\", "/");
                if (entryName.contains("../")) {
                    continue;
                }
                if (!unzipChildFile(destDir, files, zip, entry, entryName)) return files;
            }
        } finally {
            zip.close();
        }
        return files;
    }


    private static boolean unzipChildFile(final File destDir, final List<File> files, final ZipFile zip, final ZipEntry entry, final String name) throws IOException {
        File file = new File(destDir, name);
        files.add(file);
        if (entry.isDirectory()) {
            return createOrExistsDir(file);
        } else {
            if (!createOrExistsFile(file)) return false;
            InputStream in = null;
            OutputStream out = null;
            try {
                in = new BufferedInputStream(zip.getInputStream(entry));
                out = new BufferedOutputStream(new FileOutputStream(file));
                byte buffer[] = new byte[BUFFER_LEN];
                int len;
                while ((len = in.read(buffer)) != -1) {
                    out.write(buffer, 0, len);
                }
            } finally {
                if (in != null) {
                    in.close();
                }
                if (out != null) {
                    out.close();
                }
            }
        }
        return true;
    }


    public static boolean createOrExistsDir(final File file) {
        return file != null && (file.exists() ? file.isDirectory() : file.mkdirs());
    }

    public static boolean createOrExistsFile(final File file) {
        if (file == null) return false;
        if (file.exists()) return file.isFile();
        if (!createOrExistsDir(file.getParentFile())) return false;
        try {
            return file.createNewFile();
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        }
    }


}
```



### 使用apache依赖

1. 添加依赖

   ```xml
   <dependency>
     <groupId>org.apache.ant</groupId>
     <artifactId>ant</artifactId>
     <version>1.9.7</version>
   </dependency>
   ```

2. Java代码

   ```java
   import org.apache.tools.zip.ZipEntry;
   import org.apache.tools.zip.ZipFile;
   
   import java.io.File;
   import java.io.FileOutputStream;
   import java.io.IOException;
   import java.io.InputStream;
   import java.util.ArrayList;
   import java.util.Enumeration;
   import java.util.List;
   
   public class FileUtil {
   
       static String path = "/Users/raines/Desktop/my/test_upload.zip";
   
       public static void main(String[] args){
           List<File> fileList = UnZipFile(path);
           System.out.println(fileList);
       }
   
       /**
        * 解压zip压缩包并返回解压之后所得到的文件List
        */
       public static List<File> UnZipFile(String zipPath) {
           File file = new File(zipPath);
           //设置 压缩包所在的目录下与压缩包同名文件夹 为 解压后的文件所在的目录
           String unZipPath = zipPath.substring(0, zipPath.lastIndexOf("."));
           ZipFile zipFile = null;
           List<File> fileList = new ArrayList<File>();
           try {
               //设置编码格式
               //zipFile = new ZipFile(file,"GBK");
               zipFile = new ZipFile(file);
           } catch (IOException e1) {
               e1.printStackTrace();
           }
           if (zipFile == null){
               throw new RuntimeException("zipFile exception:zipFile is null");
           }
           Enumeration<ZipEntry> e = zipFile.getEntries();
           while (e.hasMoreElements()) {
               ZipEntry zipEntry = e.nextElement();
               if (zipEntry.isDirectory()) {
                   String name = zipEntry.getName();
                   name = name.substring(0, name.length() - 1);
                   File f = new File(unZipPath + File.separator + name);
                   f.mkdirs();
               } else {
                   File f = new File(unZipPath + File.separator + zipEntry.getName());
                   fileList.add(f);
                   f.getParentFile().mkdirs();
                   try {
                       f.createNewFile();
                       InputStream is = zipFile.getInputStream(zipEntry);
                       FileOutputStream fos = new FileOutputStream(f);
                       int length = 0;
                       byte[] b = new byte[1024];
                       while ((length = is.read(b, 0, 1024)) != -1) {
                           fos.write(b, 0, length);
                       }
                       is.close();
                       fos.close();
                   } catch (IOException e1) {
                       e1.printStackTrace();
                   }
               }
           }
           try {
               zipFile.close();
           } catch (IOException e1) {
               e1.printStackTrace();
           }
           file.delete();//解压完以后将压缩包删除
           return fileList;  //返回解压后所得到的文件list
       }
   
       /**
        * 删除该文件夹以及子目录和子目录文件
        */
       public static void delFolder(String folderPath) {
           try {
               delAllFile(folderPath); //删除完里面所有内容
               File path = new File(folderPath);
               path.delete(); //删除空文件夹
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   
       /**
        * 删除文件夹内所有文件和子目录
        */
       public static boolean delAllFile(String path) {
           boolean flag = false;
           File file = new File(path);
           if (!file.exists()) {
               return flag;
           }
           if (!file.isDirectory()) {
               return flag;
           }
           String[] tempList = file.list();
           File temp = null;
           for (int i = 0; i < tempList.length; i++) {
               if (path.endsWith(File.separator)) {
                   temp = new File(path + tempList[i]);
               } else {
                   temp = new File(path + File.separator + tempList[i]);
               }
               if (temp.isFile()) {
                   temp.delete();
               }
               if (temp.isDirectory()) {
                   delAllFile(path + File.separator + tempList[i]);//先删除文件夹里面的文件
                   delFolder(path + File.separator + tempList[i]);//再删除空文件夹
                   flag = true;
               }
           }
           return flag;
       }
   
   
   }
   ```

   