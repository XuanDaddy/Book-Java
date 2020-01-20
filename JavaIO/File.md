# 【一】File

### 编码

* 文本文件就是字节序列；
* 直接创建的文件与项目编码有关；
* 不同的编码，汉字对应的字节数不一样；

### File类

`java.io.File`类用于表示文件（目录）。

`File`类只用于表示文件（目录）的信息（名称、大小等），不能用于文件内容的访问。

### 常用Api

* 创建File

  File由目录及文件名称组成，创建File也是由文件的抽象（File）或文件路径构建；

  构造方法有6种方式，其中两种为`private`，外部不可调用；

  ```java
  //通过父目录及文件路径创建
  public File(File parent, String child) {
  }
  
  //通过文件完整路径创建
  public File(String pathname) {
  }
  
  //通过父路径及文件路径创建
  public File(String parent, String child) {
  }
  
  //通过URI创建
  public File(URI uri) {
  }
  ```

* 文件是否存在

  ```java
  public boolean exists() {
  }
  ```

* 创建目录或文件（真实创建）

  ```java
  //创建目录，不会级联创建
  public boolean mkdir() {
  }
  
  //级联创建目录
  public boolean mkdirs() {
  }
  
  //真实创建文件
  public boolean createNewFile() throws IOException {
  }
  
  //在默认临时文件目录中创建一个空文件，使用给定前缀和后缀生成其名称
  //与deleteOnExit对应
  public static File createTempFile(String prefix, String suffix){
  }
  ```

* 删除文件

  ```java
  //立即执行删除操作
  public boolean delete() {
  }
  
  //在虚拟机终止时，请求删除此抽象路径名表示的文件或目录。也就是说，程序运行deleteOnExit成功后，File并没有直接删除，而是在虚拟机正常运行结束后才会删除。
  //与createTempFile对应
  public void deleteOnExit() {
  }
  ```

  > 程序有个需求需要创建临时文件，这个临时文件可能作为存储使用，但是程序运行结束后，这个文件应该就被删除了。在哪里做删除操作呢，需要监控程序关闭吗，如果有很多地方可以中止程序，这个删除操作需要都放置一份吗？其实只要这么写,程序结束后文件就会被自动删除了：
  >
  > ```java
  > File tempFile = File.createTempFile("temp",".file");
  > tempFile.deleteOnExit();
  > 
  > System.out.println(tempFile.getAbsolutePath());
  > System.out.println(tempFile.getName());
  > ```
  >
  > 上述代码在操作`deleteOnExit`之后，依然可以调用`tempFile.getAbsolutePath()`和`tempFile.getName()`。

* 获取文件长度（字节数）

  ```java
  public long length() {
  }
  ```

* 判断文件或目录

  ```java
  //是否目录
  public boolean isDirectory() {
  }
  
  //是否文件
  public boolean isFile() {
  }
  ```

* 获取文件路径

  ```java
  //绝对路径
  public String getAbsolutePath() {
  }
  
  //文件抽象路径，如果构建的是相对路径，就是相对路径，如果是绝对路径，就是绝对路径
  public String getPath() {
  }
  
  //文件名称
  String getName() {
  }
  
  //父路径
  public String getParent() {
  }
  
  //父路径以File返回
  public File getParentFile() {
  }
  
  //是否绝对路径
  public boolean isAbsolute() {
  }
  ```

* 遍历、过滤目录

  只对当前目录有效，不包含子目录下的文件。

  * 过滤器

    ```java
    @FunctionalInterface
    public interface FileFilter {
    
        boolean accept(File pathname);
    }
    
    @FunctionalInterface
    public interface FilenameFilter {
        boolean accept(File dir, String name);
    }
    ```

  * 遍历方法

    ```java
    //目录下的所有文件（不包含子文件目录），返回文件（目录）名称
    public String[] list() {
    }
    
    //目录上所有文件，返回File数组
    public File[] listFiles() {
    }
    
    //添加过滤器，返回文件（目录）名称
    public String[] list(FilenameFilter filter) {
    }
    
    //添加过滤器，返回File数组
    public File[] listFiles(FileFilter filter) {
    }
    ```

  ### RandomAccessFile

  对文件内容的访问，既可以读文件，也可以写文件，并且支持随机访问文件，可以访问文件的任意位置。

  Java文件模型在硬盘上是以byte进行存储的，它是byte数据的集合。

  #### 文件模式

  文件中存在指针进行移动，打开文件时指针在开头`pointer = 0`。

  ```java
  public native long getFilePointer() throws IOException;
  ```

  * 只读

    ```java
    RandomAccessFile accessFile = new RandomAccessFile("test.file", "r");
    
    accessFile.read();//只读一个字节
    ```

  * 读写

    ```java
    RandomAccessFile accessFile = new RandomAccessFile("test.file", "rw");
    
    accessFile.write(int);//写文件：只写一个字节（后8位），同时指针指向下一个位置
    ```

  每次读写都是一个字节，不同一数据类型，都会封装读写多个字节。

  移动指针：

  ```java
  public void seek(long pos) throws IOException {
  }
  ```

  

  



