#### Java文件File类



File类概述

- java.io.File代表与平台无关的文件或目录。也就是说可以通过File类在Java程序中操作文件或目录。

- File类只能用来操作文件或目录（包括新建、删除、重命名文件和目录等操作），但不能用来访问文件中的内容。

- 如果需要访问文件中的内容，则需要使用输入/输出流。

  

**绝对路径和相对路径、路径分隔符**

- Windows中绝对路径以盘符开头；Linux中绝对路径以斜线/开头；
- 不以盘符和斜线/开头的路径就是相对路径。默认情况下，系统总是依据用户的工作路径来解释相对路径，这个路径由系统属性user.dir指定，通常也就是运行java虚拟机时所在的路径。
- Windows的路径分隔符使用反斜线\(**Linux中使用斜线/**)，而**Java程序中的反斜线表示转义字符**，所以如果需要在Windows的路径中包括反斜线，则应该使用两条反斜线\\，**如：C:\\adb\\text.txt。**
- Java程序支持将斜线/当成平台无关的路径分隔符，所以**Java程序中表示Windows系统的路径分隔符\\可以用斜线/代替。**



**File类的构造方法**

> public File(String pathname)：根据指定的路径(可以是绝对路径或相对路径)创建File对象。
>
> public File(String parent, String child)：根据指定的父文件夹和子文件或者子文件夹创建File对象。
>
> public File(File parent, String child)：根据指定的父文件夹对象和子文件或者子文件夹创建File对象。
>
> public File(URI uri)：根据URI表示的文件或目录的路径创建对象，如file:/F:/迅雷下载/test.txt。
>
> **备注: 创建好File对象后，只是封装了一个路径，和磁盘上是否有这个路径无关。**





**File类其他成员方法**

访问文件或目录的名称和路径的方法

> public String getName(): 返回File构造方法中传入路径表示的文件名或目录名（**如果是目录名则是最后一级子目录名**）。
>
> public String getPath()： 返回File构造方法中传入的路径名。
>
> public String getParent()：返回File构造方法中传入文件或目录路径的父目录，如果传入路径没有父目录，则返回null。
>
> public String getAbsolutePath()： 返回File对象表示的文件或目录的绝对路径名。
>
> public File getAbsoluteFile()： 返回一个通过此File对象表示的文件或目录的绝对路径名重新new出来的一个File对象。



文件或目录检测相关的方法

> public boolean exists()： 判断此File对象表示的文件或目录是否存在。
>
> public boolean canWrite()：判断此File对象表示的文件或目录是否可写。
>
> public boolean canRead()：判断此File对象表示的文件或目录是否可读。
>
> public boolean isFile()：判断此File对象是否是表示文件，而非目录。
>
> public boolean isDirectory()：判断此File对象是否是表示目录，而非文件。
>
> public boolean isAbsolute()：判断此File对象的构造方法传入的路径是否是绝对路径。





##### **File类创建功能**

- 说明:最终创建出来的是一个文件还是文件夹，不取决于路径名称,取决于调用的什么方法去创建。

- 创建文件 :createNewFile()。

- 创建文件夹:

  > mkdir()方法是创建文件夹，**如果父级路径不存在，则文件夹创建失败**。
  > mkdirs()方法是创建文件夹，**如果父级路径不存在，则自动创建父级路径，再创建子级路径。**

  

  示例：

  > ```java
  > package com.java.atiodemo;
  > 
  > import java.io.File;
  > import java.io.IOException;
  > 
  > public class NewFileDemo {
  > 
  >     public static void main(String[] args) throws IOException {
  > 
  >         /**
  >          * 1、创建文件实例
  >          * 使用相对路径
  >          * */
  >         File file = new File("./myNewFile");
  > 
  >         /**
  >          * 2、调用mkdirs方法创建文件夹
  >          * */
  >         boolean mkdirs = file.mkdirs();
  > 
  >         /**
  >          * 3、创建文件实例，在以上文件基础上创建
  >          * */
  >         File file1 = new File(file,"HelloWorld.text");
  > 
  >         /**
  >          * 4、调用createNewFile方法创建文件夹，
  >          * 注意:异常处理
  >          * */
  >         boolean newFile = file1.createNewFile();
  >         
  >     }
  > 
  > }
  > 
  > ```

  

##### **File类删除功能**

  

方法名称: delete() 既可以删除文件，也可以删除文件夹。

注意事项：

1. 删除的时候不走回收站，直接删除。

2. 不能删除非空文件夹。

> ```java
> public class DelFileDemo {
> 
>     public static void main(String[] args) {
>         String fileUrl = "./myNewFile";
>         String fileName = "HelloWorld.text";
>         delFile(fileUrl,fileName);
>     }
> 
>     public static void delFile(String path,String filename){
>         File file = new File(path + "/" + filename);
>         if (file.exists()&&file.isFile()){
>             boolean delete = file.delete();
>             System.out.println("文件删除成功");
>         }
>     }
> 
> }
> ```



> ```java
> public class DelDirDemo {
> 
>     public static void main(String[] args) throws IOException {
>         //删除夹文件调用
>         String fileUrl="./myNewFile";
>         delDir(fileUrl);
>     }
> 
>     public static void delDir(String path){ //删除文件夹
>         File dir=new File(path);
>         if(dir.exists()){
>             File[] tmp=dir.listFiles();
>             for (File file : tmp) {
>                 if (file.isDirectory()) {
>                     delDir(file.getAbsolutePath()); //递归删除
>                 } else {
>                     file.delete();
>                 }
>             }
>             boolean bool = dir.delete();
>             System.out.println(bool);
>         }
>     }
> }
> ```


**备注：要利用File类的delete()方法删除目录时，必须保证该目录下没有文件或者子目录，否则删除失败，要删除目录，必须利用递归删除该目录下的所有子目录和文件，然后再删该目录 。**





##### **File类文件复制功能**

1. 实现输出读取文件内容的示列(不推荐使用低效)；
2. 使用缓冲流来实现（一般）；
3. **使用管道的方式：高效（推荐使用这种方式)。**



备注：复制可多种类型，比如txt，xml，jpg，doc等多种格式。



1. 实现输出读取文件内容的示列(不推荐使用低效)

   > ```java
   > 
   > ```
   >
   > 

> ```java
> 
> ```
>
> 

