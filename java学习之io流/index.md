# Java学习之IO流






### File类的使用

- `java.io.File`类:**文件**和**文件目录路径**的抽象表示形式，与平台无关
- File能新建、删除、重命名文件和目录，但 File不能访问文件内容本身。如果需要访问文件内容本身，则需要使用输入/输出流。
- **想要在Java程序中表示一个真实存在的文件或目录，那么必须有一个File对象，但是Java程序中的一个File对象，可能没有一个真实存在的文件或目录。**
- File对象可以作为参数传递给流的构造器



```java
 * 1. File类的一个对象，代表一个文件或一个文件目录(俗称:文件夹)
 * 2. File类声明在java.io包下
 * 3.File类中涉及到关于文件或文件目录的创建、删除、重命名、修改时间、文件大小等方法，并未涉及到写入或读取文件内容的操作。如果需要读取或写入文件内容，必须使用Io流来完成。
 * 4.后续File类的对象通常会作为参数传递到流的构造器中，指明读取或写入的"终点"
```





#### 创建File类的实例:



**常用构造器：**

```bash
# public File(String pathname)
以pathname为路径创建File对象，可以是绝对路径或者相对路径，如果pathname是相对路径，则默认的当前路径在系统属性user.dir中存储。
    绝对路径:是一个固定的路径,从盘符开始
    相对路径:是相对于某个位置开始

# public File(String parent,String child)
以parent为父路径，child为子路径创建File对象。

# public File(File parent,String child)
根据一个父File对象和子文件路径创建File对象

```



**路径分隔符：**

- 路径中的每级目录之间用一个路径分隔符隔开
- 路径分隔符和系统有关:
- windows和DOS系统默认使用`\`来表示
- UNIX和URL使用`/`来表示
- Java程序支持跨平台运行，因此路径分隔符要慎用。
- 为了解决这个隐患，File类提供了一个常量:`public static final String separator`。根据操作系统，动态的提供分隔符。举例: 

```java
File file1 = new File("d:\\hello.txt");
File file2 = new File("d:" + File.separator + "hello.txt");
File file3 = new File("d:/hello.txt");
```



```java
/*
    1.如何创建File类的实例
        File(String filePath)
        File(String parentPath, String childPath)
        File(File parentPath, String childPath)

    2.相对路径、绝对路径

    3.路径分隔符

    * */
@Test
public void test(){
    //构造器1：
    //相对路径(相较于某个路径下，指明的路径:这里的IDEA中相对于当前module中)
    File file1 = new File("hello.txt");
    //绝对路径
    File file2 = new File("d:\\hello.txt");
    // 为了解决不同操作系统下路径分隔符的隐患，File类提供了一个常量:`public static final String separator`。根据操作系统，动态的提供分隔符
    File file3 = new File("d:" + File.separator + "hello.txt");

    //构造器2：
    File file4 = new File("d:\\Code", "JavaCode");

    //构造器3：
    File file5 = new File(file4, "1.txt");

}

```





#### 常用方法

**File类的获取功能**

```bash
public String getAbsolutePath():获取绝对路径
public String getPath():获取路径
public String getName():获取名称
public String getParent():获取上层文件目录路径。若无，返回null
public long length():获取文件长度（即:字节数）。不能获取目录的长度。
public long lastModified():获取最后一次的修改时间，毫秒值
# 以下两个适用于文件目录
public String[] list():获取指定目录下的所有文件或者文件目录的名称数组
public File[] listFiles():获取指定目录下的所有文件或者文件目录的File数组
```



**File类的重命名功能**

```bash
public boolean renameTo(File dest):把文件重命名为指定的文件路径
```



**File类的判断功能**

```bash
public boolean isDirectory():判断是否是文件目录
public boolean isFile():判断是否是文件
public boolean exists() :判断是否存在
public boolean canRead():判断是否可读
public boolean canWrite():判断是否可写
public boolean isHidden():判断是否隐藏
```



**File类的创建功能:**

```bash
public boolean createNewFile():创建文件。若文件存在，则不创建，返回false
public boolean mkdir() :创建文件目录。如果此文件目录存在，就不创建了。如果此文件目录的上层目录不存在，也不创建。
public boolean mkdirs():创建文件目录。如果上层文件目录不存在，一并创建

# 注意事项:如果你创建文件或者文件目录没有写盘符路径，那么，默认在项目路径下。
```



**File类的删除功能:**

```bash
public boolean delete():删除文件或者文件夹

# 删除注意事项:
Java中的删除不走'回收站'。
要删除一个文件目录，请注意该文件目录内不能包含文件或者文件目录
```



代码：

```java
   @Test
    public void test1(){
        File file1 = new File("hello.txt");
        File file2 = new File("d:\\io\\1.txt");

        System.out.println(file1.getAbsolutePath());
        System.out.println(file1.getName()); //如果只是在内存中，没有创建，获取的值是null
        System.out.println(file1.getParent()); // 相对路径一般是获取不到父级目录的
        System.out.println(file1.getAbsoluteFile());
        System.out.println(file1.length());
        System.out.println(file1.lastModified());
        System.out.println("***********");

        System.out.println(file2.getAbsolutePath()); // d:\io
        System.out.println(file2.getName());  // d:\io
        System.out.println(file2.getParent());  // d:\io
        System.out.println(file2.getAbsoluteFile()); // d:\io\1.txt
        System.out.println(file2.length());
        System.out.println(new Date(file2.lastModified()));
        System.out.println("***********");

        // # 适用于文件目录
        File file3 = new File("E:\\Code");
        //获取文件名
        String[] list = file3.list();
        for (String s: list){
            System.out.println(s);
        }
        //绝对路径的形式显示出来
        File[] list2 = file3.listFiles();
        for (File f: list2){
            System.out.println(f);
        }


        //重命名
//        public boolean renameTo(File dest):把文件重命名为指定的文件路径
        //要想返回true，需保证：file4在硬盘中存在，且file5不能存在
        File file4 = new File("d:\\io\\2.txt");
        File file5 = new File("d:\\io\\222222.txt");
        //file4命名为file5(file4就没了，file5就存在了)
        boolean renameTo = file4.renameTo(file5);
        System.out.println(renameTo);




//        public boolean isDirectory():判断是否是文件目录
        System.out.println(file3.isDirectory());
//        public boolean isFile():判断是否是文件
        System.out.println(file3.isFile());
//        public boolean exists() :判断是否存在
        System.out.println(file3.exists());
//        public boolean canRead():判断是否可读
        System.out.println(file3.canRead());
//        public boolean canWrite():判断是否可写
        System.out.println(file3.canWrite());
//        public boolean isHidden():判断是否隐藏
        System.out.println(file3.isHidden());


    }


    /*
   真正在硬盘上创建、删除

    1.File类的创建功能:
public boolean createNewFile():创建文件。若文件存在，则不创建，返回false
public boolean mkdir() :创建文件目录。如果此文件目录存在，就不创建了。如果此文件目录的上层目录不存在，也不创建。
public boolean mkdirs():创建文件目录。如果上层文件目录不存在，一并创建

# 注意事项:如果你创建文件或者文件目录没有写盘符路径，那么，默认在项目路径下。



  2.File类的删除功能:

public boolean delete():删除文件或者文件夹

# 删除注意事项:
Java中的删除不走'回收站'。
要删除一个文件目录，请注意该文件目录内不能包含文件或者文件目录



     */
    @Test
    public void test2() throws IOException {
        //文件创建和删除
        File file = new File("d:\\io\\2.txt");
        if(!file.exists()){ //如果不存在就创建
            file.createNewFile();
            System.out.println("创建ok");
        }else{ //如果存在，就删除
            file.delete();
            System.out.println("delete ok");
        }

        //目录的创建和删除
        File pathName = new File("d:\\io\\xx\\yy");
        if(!pathName.exists()){ //如果不存在就创建
            boolean mk = pathName.mkdir();
            //boolean mk = pathName.mkdirs();  //如果上级目录不存在就递归创建目录
            if(mk){
                System.out.println("目录创建ok");
            }

        }else{ //如果存在，就删除
            pathName.delete(); // 注意该文件目录内不能包含文件或者文件目录
            System.out.println("目录delete ok");
        }

    }
```









### IO流原理及流的分类

- l/O是Input/Outpqt的缩写，I/O技术是非常实用的技术，用于**处理设备之间的数据传输**。如读/写文件，网络通讯等。
- Java程序中，对于数据的输入/输出操作以“**流(stream)**”的方式进行。
- java.io包下提供了各种“流”类和接口，用以获取不同种类的数据，并通过**标准的方法**输入或输出数据。



- 输入input: 读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存)中。
- 输出output: 将程序（内存)数据输出到磁盘、光盘等存储设备中。
- 我们要站在程序的角度



#### 流的分类

- 按操作**数据单位**不同分为:**字节流**(8 bit -- bytes)，**字符流**(16 bit --- char)

- 按数据流的**流向**不同分为: 输入流(数据--->程序)，输出流(程序-->数据)

- 按流的**角色**的不同分为:节点流(直接作用在文件上的)，处理流(这个流的对象作为外面的流的参数，包了一层，加快了传输速度)

  | （抽象基类） |    字节流    | 字符流 |
  | :----------: | :----------: | :----: |
  |    输入流    | InputStream  | Reader |
  |    输出流    | OutputStream | Writer |

1. Java的IO流共涉及40多个类，实 际上非常规则，都是从如上4个抽象基类派生的。
2. 由这四个类派生出来的子类名称都是以其父类名作为子类名后缀（比如后缀带Stream的都是`字节`，后缀带`Reader`或`Writer`的都是`字节`）。

![IO流体系](http://image.xpshuai.cn/JavaIO%E6%B5%81%E4%BD%93%E7%B3%BB.png)

> 深色的为重点

















### 节点流(或文件流)

#### 字符流

- FileReader
- FileWriter

字符流不能读取图片等字节数据



代码：

```java
/*
字符流：
FileReader
FileWriter


Note:
1. read()的理解:返回读入的一个字符。如果达到文件末尾，返回-1
2．异常的处理:为了保证流资源一定可以执行关闭操作。需要使用try-catch-finally处理
3. 读入的文件一定要存在

     */
    @Test
    public void test(){
        //如果是在main方法中，则相对路径是相对于当前工程
        //【功能】：读取文件内容到程序中，并输出到控制台

        FileReader fr = null;
        try{
            //1.实例化File类的对象，指明要操作的文件
            File file = new File("d:\\io\\1.txt");
            //2.提供具体的流
            fr = new FileReader(file);
            //3.数据的读入
            //read():返回读入的一个字符，如果达到文件末尾，返回-1
            //方式1：
//        int data = fr.read();  // 返回-1表示读完了
//        while (data != -1){
//            System.out.println((char) data);
//            data = fr.read(); //继续向后读
//        }
            //方式2:
            int data;
            while ((data = fr.read()) != -1){
                System.out.println((char) data);
            }

        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            //4.流的关闭
            try{
                if(fr != null){
                    fr.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }


/*
FileReader(char[] cbuf)读入数据
对read()操作升级: 使用read的重载方法

 */
    @Test
    public void test1(){
        FileReader fr = null;
        try{
            //1.实例化File类的对象，指明要操作的文件
            File file = new File("d:\\io\\1.txt");
            //2.FileReader的实例化，提供具体的流
            fr = new FileReader(file);
            //3.数据的读入操作（难点）：
            char[] cbuf = new char[5]; //每次读入cbuf数组长度个字符的个数，如果达到文件末尾则返回-1
            int len;
            while ((len = fr.read(cbuf)) != -1){
                //方式1：
                for (int i = 0; i < len; i++) {
                    System.out.print(cbuf[i]);
                }
                //方式2：
//                String str = new String(cbuf, 0,len); //从0开始每次取len个
//                System.out.println(str);

            }

        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            //4.流资源的关闭
            try{
                if(fr != null){
                    fr.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }


    /*
    FileWriter(): 从内存中写出数据到硬盘的文件中

    1．输出操作，对应的FiLe是可以不存在的。
        如果不存在，在输出的过程中，会自动创建此文件。
        如果存在，默认参数append:false会对原有文件的覆盖; true则在原有文件基础上添加
        即: 如果流使用的构造器是:FiLewriter(fiLe,false)/FiLewriter(fiLe):对原有文件的覆盖
            如果流使用的构造器是:FiLewriter(file,true):不会对原有文件覆盖，而是在原有文件基础上追加内容
     */
    @Test
    public void test2(){
        FileWriter fw = null;
        try{
            // 1.提供File类的对象，指明写出到的文件
            File file = new File("d:\\io\\11.txt");
            //2.提供FileWriter的对象，用于数据的写出
            fw = new FileWriter(file, false);
            //3.写出的操作
            fw.write("hhh hhh hdaldk\n");
            fw.write("hhh 222 hdaldk\n");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                //4.流资源的关闭
                fw.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }

    }


/*
使用FileReader和FileWriter实现文本文件的复制

 */
    @Test
    public void test3() {
        FileReader fr = null;
        FileWriter fw = null;
        try {
            // 1.创建File类的对象，指明读入和写出的文件
            File srcFile = new File("d:\\io\\1.txt");
            File dstFile = new File("d:\\io\\111.txt");

//        2.创建输入流和输出流的操作
            fr = new FileReader(srcFile);
            fw = new FileWriter(dstFile);

//        3.数据的读入和写出操作
            char[] cbuf = new char[5];
            int len; //每次读入cbuf数组长度个字符的个数
            while ((len = fr.read(cbuf)) != -1) {  //
                fw.write(cbuf, 0, len); //每次写出len个字符
            }
//        4.关闭流资源
            }catch(Exception e){
                e.printStackTrace();
            }finally{
                try {
                    if(fr != null){
                        fr.close();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    try {
                        if(fw != null){
                            fw.close();
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }

        }


```







#### 字节流

- FileInputStream
- FileOutputStream



```java
    /*
字节流：
FileInputStream
FileOutputStream

结论：FileInputStream
对于文本文件，还是要用字符流
对于图片等，还是要用字节流

 */
    @Test
    public void test4(){
        FileInputStream fis = null;
        try{
            //1.造文件
            File file = new File("d:\\io\\1.txt");
            //2.造流
            fis = new FileInputStream(file);
            //3.读数据
            byte[] buffer = new byte[5];
            int len; //记录每次读取的字节的个数
            while ((len = fis.read(buffer)) != -1){
                String str = new String(buffer,0, len);
                System.out.println(str);  // 这样中文可能乱码，因为uft-8中文一个字是占3个字节
            }


//        File file = new File("d:\\io\\1.txt");

        }catch (Exception e){
            System.out.println(e.getMessage());
        }finally {
            //4.关闭资源
            try{
                fis.close();
            }catch (Exception e){
                e.printStackTrace();
            }

        }


    }

/*
 FileOutputStream
 实现对图片的复制操作
 如果只是相对文本文件进行复制操作，也可以使用字节流
 */
    @Test
    public void test5(){
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try{
            //1.造文件
            File srcFile = new File("d:\\io\\1.png");
            File dstFile = new File("d:\\io\\2.png");

            //2.造流
            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(dstFile);

            //3.复制的过程
            byte[] buffer = new byte[10];
            int len; //记录每次读入的字节的个数
            while ((len = fis.read(buffer)) != -1){
                fos.write(buffer, 0, len); // 每次写10个字节
            }
            System.out.println("复制ok.");

        }catch (Exception e){
            System.out.println(e.getMessage());
        }finally {
            //4.关闭资源
            try{
                if(fis != null){
                    fis.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }

            try{
                if(fos != null){
                    fos.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }

        }


    }


    //为了方便指定路径下文件的复制(封装为一个函数)
    public void copyFile(String srcPath, String dstPath){
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try{
            //1.造文件
            File srcFile = new File(srcPath);
            File dstFile = new File(dstPath);

            //2.造流
            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(dstFile);

            //3.复制的过程
            byte[] buffer = new byte[1024];  // 并不是越大越好(太大了占内存大了，找个适中的)
            int len; //记录每次读入的字节的个数
            while ((len = fis.read(buffer)) != -1){
                fos.write(buffer, 0, len); // 每次写10个字节
            }
            System.out.println("复制ok.");

        }catch (Exception e){
            System.out.println(e.getMessage());
        }finally {
            //4.关闭资源
            try{
                if(fis != null){
                    fis.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }

            try{
                if(fos != null){
                    fos.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }

        }

    }


    @Test
    public void testCopy(){
        long start = System.currentTimeMillis();

        //可以直接调用我们封装好的复制文件的函数：
        copyFile("d:\\io\\xiaodi.mp4", "d:\\io\\study.mp4");   // 后面再学对于视频可以用缓冲流

        long end = System.currentTimeMillis();

        System.out.println(end - start);

    }

```







### 处理流

#### 1.缓冲流(使用的更多)

> 开发中用缓冲流多，一般不用前面的字节流(FileInputStreamFile、OutputStream)



- BufferedInputStream
- BufferedOutputStream
- BufferedReader
- BufferedWriter



**缓冲流作用：**提供流的读取、写入的速度

----> 原因：提高读写速度的原因:内部提供了一个缓冲区

**处理流,** 就是“套接”在已有的流的基础上。



**缓冲流与节点流：**

```
BufferedInputStream
BufferedOutputStream
```

```java
/*
BufferedInputStream
BufferedOutputStream


缓冲流作用：提供流的读取、写入的速度
----> 原因：提高读写速度的原因:内部提供了一个缓冲区
 */
    @Test
    public void test6(){
        //以非文本复制文件为例
        bufferCopyFile("d:\\io\\xiaodi.mp4", "d:\\io\\xiaodi2.mp4");


    }
    public void bufferCopyFile(String srcPath, String dstPath){
        //缓冲流与字节流
        FileInputStream fis = null;
        FileOutputStream fos = null;
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;
        try{
            //1.造文件
            File srcFile = new File(srcPath);
            File dstFile = new File(dstPath);

            //2.造流(处理流不能直接作用域文件上): 先造里层的流, 再造外层的流
            //2.1造节点流
            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(dstFile);

            //2.2造缓冲流
            bis = new BufferedInputStream(fis);
            bos = new BufferedOutputStream(fos);


            //3.复制的过程: 读取，写入
            byte[] buffer = new byte[10];  // 并不是越大越好(太大了占内存大了，找个适中的)
            int len; //记录每次读入的字节的个数
            while ((len = bis.read(buffer)) != -1){  // 用bis来read
                bos.write(buffer, 0, len); // 用bos来write，每次写10个字节
            }
            System.out.println("复制ok.");

        }catch (Exception e){
            System.out.println(e.getMessage());
        }finally {
            //4.关闭资源（要求:先关外层的流(处理流), 在关里层的流）
            // 说明：在关闭外层流的同时，内层流也会自动关闭(我们可以省略不写了就)
            try{
                if(bis != null){
                    bis.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
            try{
                if(bos != null){
                    bos.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }


        }

    }

```



**缓冲流与字符流：**（）

```
BufferedReader
BufferedWriter
```

```java
//以文本文件复制为例
// BufferedReader、BufferedWriter
bufferCopyTxtFile("d:\\io\\1.txt", "d:\\io\\11111.txt");

    

public void bufferCopyTxtFile(String srcPath, String dstPath){
        //缓冲流与字符流

        BufferedReader br = null;
        BufferedWriter bw = null;
        try{
            //1.造文件、造流(熟练之后可以这样写在一起)
            br = new BufferedReader(new FileReader(new File(srcPath)));
            bw = new BufferedWriter(new FileWriter(new File(dstPath)));

            //2.读取，写入(字符流)
            //方式1：
//            char[] cbuf = new char[1024];
//            int len; //记录每次读入的个数
//            while ((len = br.read(cbuf)) != -1){  // 用bis来read
//                bw.write(cbuf, 0, len);
////                bw.flush();
//            }

            //方式2：
            String data;
            while((data = br.readLine()) != null){ //一次读一行
                //方法1：
                bw.write(data + "\n"); //data中不包含换行符
                //方法2：
                bw.write(data ); //data中不包含换行符
                bw.newLine(); //提供换行操作
            }



            System.out.println("复制ok.");

        }catch (Exception e){
            System.out.println(e.getMessage());
        }finally {
            //4.关闭资源（要求:先关外层的流(处理流), 在关里层的流）
            // 说明：在关闭外层流的同时，内层流也会自动关闭(我们可以省略不写了就)
            try{
                if(br != null){
                    br.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
            try{
                if(bw != null){
                    bw.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }


        }

    }

```



![几个IO流小汇总](http://image.xpshuai.cn/%E5%87%A0%E4%B8%AAIO%E6%B5%81%E5%B0%8F%E6%B1%87%E6%80%BB.png)



**练习1**：实现图片加密操作

```java
//提示:
//图片加密：
FileInputStream fis = new FileInputStream(srcFile);
FileOutputStream fos = new FileOutputStream(dstFile);

byte[] buffer = new byte[1024];  
int len; //记录每次读入的字节的个数

while ((len = fis.read(buffer)) != -1){ 
    for(int i; i<len; i++){
        buffer[i] = (byte)(buffer[i]^5); //每位字节做了异或的运算
        fos.write(buffer, 0, len);
    }
}


//图片解密：
FileInputStream fis = new FileInputStream("加密后的文件");
FileOutputStream fos = new FileOutputStream(dstFile);

byte[] buffer = new byte[1024];  
int len; //记录每次读入的字节的个数

while ((len = fis.read(buffer)) != -1){ 
    for(int i; i<len; i++){
        buffer[i] = (byte)(buffer[i]^5); //每位字节做了异或的运算，还原
        fos.write(buffer, 0, len);
    }
}


```

**练习1*2**：获取文本上**每个字符**出现的次数

```java
//提示:遍历文本的每一个字符;字符及出现的次数保存在Map中(key是字符,value是次数);将Map中数据写入文件


package cn.xpshuai.java3;

import org.junit.Test;

import java.io.*;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;


/**
 *
 * 思路：
 * 1.遍历文本每一个字符
 * 2.字符出现的次数存在Map中
 *
 * Map<Character,Integer> map = new HashMap<Character,Integer>();
 * map.put('a',18);
 * map.put('你',2);
 *
 * 3.把map中的数据写入文件
 *
 */
public class WordCount {
    /*
    说明：如果使用单元测试，文件相对路径为当前module
          如果使用main()测试，文件相对路径为当前工程
     */
    @Test
    public void testWordCount() {
        FileReader fr = null;
        BufferedWriter bw = null;
        try {
            //1.创建Map集合
            Map<Character, Integer> map = new HashMap<Character, Integer>();

            //2.遍历每一个字符,每一个字符出现的次数放到map中
            fr = new FileReader("dbcp.txt");
            int c = 0;
            while ((c = fr.read()) != -1) {
                //int 还原 char
                char ch = (char) c;
                // 判断char是否在map中第一次出现
                if (map.get(ch) == null) {
                    map.put(ch, 1);
                } else {
                    map.put(ch, map.get(ch) + 1);
                }
            }

            //3.把map中数据存在文件count.txt
            //3.1 创建Writer
            bw = new BufferedWriter(new FileWriter("wordcount.txt"));

            //3.2 遍历map,再写入数据
            Set<Map.Entry<Character, Integer>> entrySet = map.entrySet();
            for (Map.Entry<Character, Integer> entry : entrySet) {
                switch (entry.getKey()) {
                    case ' ':
                        bw.write("空格=" + entry.getValue());
                        break;
                    case '\t'://\t表示tab 键字符
                        bw.write("tab键=" + entry.getValue());
                        break;
                    case '\r'://
                        bw.write("回车=" + entry.getValue());
                        break;
                    case '\n'://
                        bw.write("换行=" + entry.getValue());
                        break;
                    default:
                        bw.write(entry.getKey() + "=" + entry.getValue());
                        break;
                }
                bw.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.关流
            if (fr != null) {
                try {
                    fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if (bw != null) {
                try {
                    bw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }

    }
}

    
```







#### 2.转换流

> 转换流提供了在字节流和字符流之间的转换
>
> 属于字符流
>
> 作用：提供字节流与字符流之间的转换

**Java API提供了两个转换流:**

- **InputStreamReader**: 将lnputStream转换为Reader
- **outputStreamWriter**:将Writer转换为OutputStream

字节流中的数据都是字符时，转成字符流操作更高效。
很多时候我们使用转换流来处理文件乱码问题。实现编码和解码的功能。

![Java转换流](http://image.xpshuai.cn/Java%E8%BD%AC%E6%8D%A2%E6%B5%81.png)





```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.io.*;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-30 11:24
 * @功能：处理流2：转换流的使用
 *
 * 1.转换流：属于字符流
 * - InputStreamReader: 将字节的输入流InputStream转换为字符的输入流Reader
 * - outputStreamWriter:将字符的输出流Writer转换为字节的输出流OutputStream
 *
 * 2.解码：字节、字节数组 --> 字符数组、字符串
 *   编码：字符数组、字符串 --> 字节、字节数组
 *
 *
 * 3.字符集
 *
 *
 *
 */
public class InputStreamReaderTest {
    /*
    输入流
    InputStreamReader
     */
    @Test
    public void test() throws IOException {  //仍然需要try-catch，这里我偷懒一下
        FileInputStream fis = new FileInputStream(new File("d:\\io\\1.txt"));
        InputStreamReader isr = new InputStreamReader(fis, "UTF-8"); //默认使用系统的字符集

        char[] cbuf = new char[5];
        int len;
        while ((len = isr.read(cbuf)) != -1){
            String str = new String(cbuf, 0, len);
            System.out.println(str);
        }
        isr.close();

    }

    /*
    输出流：outputStreamWriter
    综合使用InputStreamReader和outputStreamWriter，读取，换个字符集，再写出去
     */
    @Test
    public void test1() throws IOException {
        //1.造文件、造流
        FileInputStream fis = new FileInputStream(new File("d:\\io\\1.txt"));
        FileOutputStream fos = new FileOutputStream(new File("d:\\io\\1_gbk.txt"));

        InputStreamReader isr = new InputStreamReader(fis, "UTF-8"); //默认使用系统的字符集
        OutputStreamWriter osw = new OutputStreamWriter(fos, "gbk");

        //2.读取写入
        char[] cbuf = new char[5];
        int len;
        while ((len = isr.read(cbuf)) != -1){
            osw.write(cbuf, 0, len);
        }

        //3.关闭
        isr.close();
        osw.close();
    }

}

```



##### 字符集

**编码表的由来**

```bash
计算机只能识别二进制数据，早期由来是电信号。为了方便应用计算机，让它可以识别各个国家的文字。就将各个国家的文字用数字来表示，并一―对应，形成一张表。这就是编码表。
```



**常见的编码表**

- **ASCII**: 美国标准信息交换码。
  用一个字节的7位可以表示。
- **ISO8859-1**: 拉丁码表/欧洲码表
  用一个字节的8位表示。
- **GB2312**: 中国的中文编码表。最多两个字节编码所有字符
- **GBK**: 中国的中文编码表升级，融合了更多的中文文字符号。最多两个字节编码
- **Unicode**: 国际标准码，融合了目前人类使用的所有字符。为每个字符分配唯一的字符码。所有的文字都用两个字节来表示。
- **UTF-8**:变长的编码方式，可用1-4个字节来表示一个字符。



- Unicode不完美，这里就有三个问题，一个是，我们已经知道，英文字母只用一个字节表示就够了，第二个问题是如何才能区别Unicode利和IIASCIl?计算机怎么知道两个字节表示一个符号，而不是分别表示两个符号呢?第三个，如果和IGBK等双字节编码方式一样，用最高位是1或O表示两个字节和一个字节，就少了很多值无法用于表示字符，不够表示所有字符。Unicode在很长一段时间内无法推广，直到互联网的出现。
- 面向传输的众多UTF (UCS Transfer Format〉标准出现了，顾名思义，**UTF-8就是每次8个位传输数据，而UTF-16就是每次16个位**。这是为传输而设计的编码，并使编码无国界，这样就可以显示全世界上所有文化的字符了。
- **Unicode只是定义了一个庞大的、全球通用的字符集，并为每个字符规定了唯一确定的编号，具体存储成什么样的字节流，取决于字符编码方案**。推荐的Unicode编码是UTF-8和UTF-16。

- ANSI编码, 通常指的是平台的默认编码,例如英文操作系统中是ISO-8859-1，中文系统是GBK
- **Unicode字符集**只是定义了字符的集合和唯一编号，**Unicode编码**，则是对UTF-8、UCS-2/UTF-16等具体编码方案的**统称**而已，并不是具体的编码方案。







### 标准输入、输出流(了解即可)

- System.in和System.out分别代表了系统标准的输入和输出设备

- 默认输入设备是:键盘，输出设备是:显示器

- System.in的类型是InputStream

- System.out的类型是PrintStream，其是OutputStream的子类FilterOutputStream 的子类

- 重定向:通过System类的setIn，setOut方法对默认设备进行改变。

  ```java
  public static void setln(InputStream in)
  public static void setOut(PrintStream out)
  ```

  

```java
    /*
    标准输入、输出流
    1.
    System.in:标准的输入流，默认从键盘输出
    System.out:标准的输出流，默认从控制台输出

    2.
    System类的setIn(InputStream is) / setOut(PrintStream ps)方式重新指定输入和输出的流

     */
    @Test
    public void test(){
        // 练习：从键盘输入字符串，要求将读取到的整行字符串转成大写输出。然后继续进行输入操作，直至当输入“e”或者“exit”时，退出程序。
//       方法1： 可以用Scanner来做, 调用next()返回一个字符串
//       方法2：System.in --> 转换流 --> BufferedReader的readLine()

        BufferedReader br = null;
        try{
            InputStreamReader isr = new InputStreamReader(System.in);
             br = new BufferedReader(isr);

            while (true){
                System.out.println("请输入字符串：");
                String data = br.readLine();
                if("e".equalsIgnoreCase(data) || "exit".equalsIgnoreCase(data)){
                    System.out.println("程序结束");
                }

                String upperCase = data.toUpperCase();
                System.out.println(upperCase);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if(br != null){
                try{
                    br.close();
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
    }

```





### 打印流(了解即可)

- 实现将**基本数据类型**的数据格式转化为**字符串**输出
- 打印流:**PrintStream**和**PrintWriter**
- 提供了一系列重载的print()和printIln()方法，用于多种数据类型的输
- PrintStream和PrintWriter的输出不会抛出IOException异常
- PrintStream和PrintWriter有自动flush功能
- PrintStream打印的所有字符都使用平台的默认字符编码转换为字节。在需要写入字符而不是写入字节的情况下，应该使用 PrintWriter类。
- System.out返回的是PrintStream的实例

```
提供了一些列重载的print()和println()
```



### 数据流(了解即可)

> ​    作用: 用于读取或写出基本数据类型的变量或字符串

- 为了方便地操作Java语言的基本数据类型和String的数据，可以使用数据流。
- 数据流有两个类:(用于读取和写出基本数据类型、String类的数据)
- **DatalnputStream**和 **DataOutputStream**
- 分别“套接”在InputStream和 OutputStream子类的流上
- DatalnputStream中的方法
  boolean readBoolean()
  byte readByte()
  char readChar(
  float readFloat()
  double readDouble()
  short readShort()
  long readLong()
  int readInt()
  String readUTF(
  void readFully(byte[ b)
- DataOutputStream中的方法
- 将上述的方法的read改为相应的write即可。



```java
    /*
    数据流
    1.
    DataInputStream
    DataOutputStream

    2.
    作用:用于读取或写出基本数据类型的变量或字符串, 基本数据类型持久换
    
    3.Note:
    读取不同类型的数据要与当初写入文件时，保存的数据的顺序一致

     */
    @Test
    public void test2() throws IOException {
        // 练习：将内存中的字符串、基本数据类型的变量写出到文件中。|
        DataOutputStream dos = new DataOutputStream(new FileOutputStream("1.txt"));
        dos.writeUTF("小小");
        dos.flush();  //刷新操作，将内存中数据写入
        dos.writeInt(18);
        dos.flush();
        dos.close(); //这个文件也要用数据流读

        //将文件中存储的基本数据类型和字符串读取到内存中，保存在变量中
        DataInputStream dos2 = new DataInputStream(new FileInputStream("1.txt"));
        String name = dos2.readUTF();
        int age = dos2.readInt();
        dos2.close(); 
        
    }


```





### 对象流(重点)

> ​    作用: 用于读取或写出对象,  对象持久换



- **ObjectInputStream**和**ObjectOutputSteam**
- 用于存储和读取**基本数据类型**数据或**对象**的处理流。它的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。
- **序列化:** 用ObjectOutputStream类保存基木类型数据或对象的机制
- **反序列化**: 用ObjectInputStream类读取基木类型数据或对象的机
- ObjectOutputStream和IObjectlnputStream不**能序列化static利transient修饰的**成员变量





#### 对象的序列化

- **对象序列化机制**允许**把内存中的Java对象转换成平台无关的二进制流**，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。当其它程序获取了这种二进制流，就可以恢复成原来的Java对象
- 序列化的好处在于可将任何实现了Serializable接口的对象转化为**字节数据**，使其在保存和传输时可被还原
- 序列化是 **RMI(**Remote Method Invoke -远程方法调用)过程的参数和返回值都必须实现的机制，而 RMI是JavaEE的基础。因此序列化机制是JavaEE平台的基础
- 如果需要让某个对象支持序列化机制，则必须让对象所属的类及其属性是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一。否则，会抛出NotSerializableException异常
- Serializable
- Externalizable





**要想序列化自定义类，需要满足哪些要求？**

自定义需要满足如下要求，方可序列化

1. 需要实现接口：Serializable

2. 当前类提供一个全局常量：serialVersionUID

`static final long serialVersionUID = 446768135468782L;`

3. 除了当前自定义类需要实现Serializable接口之外，还必须保证其内部所有属性也是可序列化的(但是默认情况下，基本数据类型是可序列化的)，内部如果有其他自定义类，就需要让其他类也实现Serializable接口
   
   



代码：

```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.io.*;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-30 14:47
 * @功能：对象流
 * 1.
 * ObjectInputStream
 * ObjectOutputSteam
 *
 *2.要想序列化自定义类，需要满足哪些要求？
 *  * Person需要满足如下要求，方可序列化
 *  * 1.需要实现接口：Serializable
 *  * 2.当前类提供一个全局常量：serialVersionUID
 *  * static final long serialVersionUID = 446768135468782L;
 *  3.除了当前Person类需要实现Serializable接口之外，还必须保证其内部所有属性也是可序列化的
 *   (但是默认情况下，基本数据类型是可序列化的)内部如果有其他自定义类，就需要让其他类也实现Serializable接口
 *
 *
 * 4.序列化机制：
 * 对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从
 * 而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到
 * 另一个网络节点。当其它程序获取了这种二进制流，就可以恢复成原来的Java对象
 *
 *
 *
 */
public class ObjectStream {
    /*
    序列化过程：将内存中的Java对象保存到磁盘中或通过网络传输出去
    使用ObjectOutputStream实现
     */
    @Test
    public void test(){
        ObjectOutputStream oos = null;
        try {
            //1.
            oos = new ObjectOutputStream(new FileOutputStream("d:\\io\\233.txt")); //这个文件不是让你打开的，想看就用反序列化
            //2.
            oos.writeObject(new String("我爱天安门"));
            oos.flush();

            //也可以序列化自定义类
            //
            oos.writeObject(new Person("Tom", 18));
            oos.flush();

            oos.writeObject(new Person("Tom2", 18, new Account(1000)));
            oos.flush();

        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(oos != null){
                try {
                    //3.
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /*
    反序列化：将磁盘文件中的对象或网络中的序列化的东西，还原为内存中的Java对象
    使用 ObjectInputStream
     */
    @Test
    public void test1(){
        ObjectInputStream ois = null;
        try {
            //1.
            ois = new ObjectInputStream(new FileInputStream("d:\\io\\233.txt")); // 通过反序列化来查看前面保存的文件内容
            //2.
            Object obj = ois.readObject();
            String str = (String)obj;

            //看一下自定义类反序列化后
            Person p = (Person)ois.readObject();
            System.out.println(p.toString());


        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if(ois != null){
                try {
                    //3.
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }


    }



    @Test
    public void test2(){

    }


    @Test
    public void test3(){

    }
}

```

```java
package cn.xpshuai.java1;

import java.io.Serializable;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-30 17:25
 * @功能： 自定义的可被序列化的类
 *
 * Person需要满足如下要求，方可序列化
 * 1.需要实现接口：Serializable
 * 2.当前类提供一个全局常量：serialVersionUID
 * static final long serialVersionUID = 446768135468782L;
 *
 * 3.除了当前Person类需要实现Serializable接口之外，还必须保证其内部所有属性也是可序列化的
 *   (但是默认情况下，基本数据类型是可序列化的)内部如果有其他自定义类，就需要让其他类也实现Serializable接口
 *
 * 补充：
 * ObjectOutputStream和IObjectlnputStream不**能序列化static利transient修饰的**成员变量
 *
 *
 */
public class Person implements Serializable {
    static final long serialVersionUID = 446768135468782L;

    private String name;
    private int age;
    private Account acc;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Person(String name, int age, Account acc) {
        this.name = name;
        this.age = age;
        this.acc = acc;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
   }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", acc=" + acc +
                '}';
    }
}


class Account implements Serializable{
    static final long serialVersionUID = 44676813666782L;

    private double balance;

    public Account() {
    }

    public Account(double balance) {
        this.balance = balance;
    }

    public double getBalance() {
        return balance;
    }

    public void setBalance(double balance) {
        this.balance = balance;
    }

    @Override
    public String toString() {
        return "Account{" +
                "balance=" + balance +
                '}';
    }
}

```



**serialVersionUID的理解：**

- 凡是实现Serializable接口的类都有一个表示序列化版本标识符的静态变量:private 
- **static final long serialVersionUID**;
- `serialVersionuID`用来表明类的不同版本间的兼容性。简言之，其目的是以序列化对里进行版本控制，有关各版本反序列化时是否兼容。
- 如果类没有显示定义这个静态常量，它的值是Java运行时环境根据类的内部细节自动生成的。若类的实例变量做了修改，seriaVersionUID可能发生变化。故**建议，显式声明**。
- 简单来说，Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版木一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与木地相应实体类的serialVersionUID进行比较，**如果相同就认为是一致的，可以进行反序列化**，否则就会出现序列化版本不一致的异常(InvalidCastException)。











### 随机存取文件流(了解即可)

**RandomAccessFile类**

- `RandomAccessFile`声明在`java.io`包下，但直接继承于`java.lang.Object`类。并且它实现了Datalnput、DataOutput这两个接口，也就意味着这个类既**可以读也可以写**。
- RandomAccessFile类支持“随机访问”的方式，程序**可以直接跳到文件的任意地方**来读、写文件
- 支持只访问文件的部分内容
- 可以向**已存在**的文件后**追加**内容
- RandomAccessFile对象包含一个记录指针，用以标示当前读写处的位置。RandomAccessFile 类对象可以自由移动记录指针:
- `long getFilePointer()`:获取文件记录指针的当前位置
- `void seek(long pos)`:将文件记录指针定位到pos位置



**构造器**

```bash
public RandomAccessFile(File file, String mode)

public RandomAccessFile(String name, string made)
```



创建RandomAccessFile类实例需要指定一个**mode参数**，该参数指定RandomAccessFile的访问模式:

```bash
r:以只读方式打开
rw:打开以便读取和写入
rwd:打开以便读取和写入:同步文件内容的更新
rws:打开以便读取和写入;同步文件内容和元薮据的更新
```

- 如果模式为只读`r`。则不会创建文件，而是会去读取一个已经存在的文件,如果读取的文件不存在则会出现异常。

- 如果模式为`rw读写`。如果文件不存在则会去创建文件，如果存在则不会创建。





```java
package cn.xpshuai.java1;

import org.junit.Test;

import java.io.File;
import java.io.IOException;
import java.io.RandomAccessFile;

/**
 * @author: 剑胆琴心
 * @create: 2021-01-30 17:46
 * @功能：随机存取文件流
 *
 * 1.直接继承java.lang.Object类
 *   实现了DataInput和DataOutput接口
 *      RandomAccessFile，既可以输入也可以输出(但是复制的时候还是需要造两个对象哦)
 *
 *
 *2.如果RandomAccessFile作为输出李璐，写出到的文件如果不存在，则在执行过程中自动创建
 *  如果写出的文件存在，则会对原有文件从头覆盖
 *
 *
 * 3.通过相关操作，实现RandomAccessFile对文件内容实现插入式的效果
 *
 */
public class RandomStream {
    @Test
    public void test() throws IOException {
//        1.造输入流、输出流
        RandomAccessFile raf1 = new RandomAccessFile(new File("d:\\io\\1.txt"), "r");
        RandomAccessFile raf2 = new RandomAccessFile(new File("d:\\io\\1111111.txt"), "rw");

//        2.读写过程
        byte[] buffer = new byte[1024];
        int len;
        while ((len = raf1.read()) != -1){
            raf2.write(buffer, 0, len);
        }

//        3.关闭
        raf1.close();
        raf2.close();

    }


    /*
    测试 对文本内容的覆盖

     */
    @Test
    public void test2() throws IOException {
        RandomAccessFile raf2 = new RandomAccessFile(new File("d:\\io\\hello1.txt"), "rw");
        raf2.write("xxx".getBytes());
        raf2.close();

    }


    /*
实现数据的插入：seek(long pos) -->将文件记录定位到指针位置

 */
    @Test
    public void test3() throws IOException {
        RandomAccessFile raf2 = new RandomAccessFile(new File("d:\\io\\hello1.txt"), "rw");
        raf2.seek(3); //指针调到角标为3的位置(第四个字符)
        raf2.write("xxx".getBytes()); // write 直接覆盖了第四个位置原有字符

        //实现指定位置的插入数据的追加效果(不覆盖原有位置的字符)



        raf2.close();

    }

    /*
实现数据的插入：seek(long pos) -->将文件记录定位到指针位置
实现指定位置的插入数据的追加效果(不覆盖原有位置的字符)
*/
    @Test
    public void test4() throws IOException {
        RandomAccessFile raf2 = new RandomAccessFile(new File("d:\\io\\hello1.txt"), "rw");
        raf2.seek(3); //指针调到角标为3的位置(第四个字符)
        //保存指针3后面的数据
        StringBuilder builder = new StringBuilder((int)new File("d:\\io\1.txt").length());
        byte[] buffer = new byte[1024];
        int len;
        while ((len = raf2.read()) != -1){
            builder.append(new String(buffer,0,len));
        }
        //调回指针
        raf2.seek(3);
        raf2.write("XXX".getBytes());
        //将StringBuilder中的数据写回去
        raf2.write(builder.toString().getBytes());

        raf2.close();

    }
}

```







### NIO.2中Path、Paths、Files类的使用(了解即可)

#### Java NIO

Java NIO (New lO，Non-Blocking lO)是从Java 1.4版木开始引入的一套新的IOAPI，可以替代标准的Java lO API。NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同，**NIO支持面向缓冲区的**(IO是面向流的)、基于通道的IO操作。**NIO将以更加高效的方式进行文件的读写操作**。

Java API中提供了两套NIO，一套是针对标准输入输出NIO，另一套就是网络编程NIO.

```bash
|-----java.nio.channels.Channel
	|-----FileChannel:处理本地文件
	|-----SocketChannel:TCP网络编程的客户端的Channel
	|-----ServerSocketChannel:TCP网络编程的服务器端的Channel		
    |-----DatagramChannel:UDP网络编程中发送端和接收端的Channel
```







#### NIO.2

随着JDK7的发布，Java对NIO进行了极大的扩展，增强了对文件处理和文件系统特性的支持，以至于我们称他们为NIO.2。因为NIO提供的一些功能，NIO已经成为文件处理中越来越重要的部分。





**Path、Paths和Files核心API:**

- **早期**的Java只提供了一个**File类**来访问文件系统，但File类的功能比较有限，提供的方法性能也不高。而且，**大多数方法在出错时仅返回失败，并不会提供异常信息**。

- **NIO.2**为了弥补这种不足，引入了Path接口，代表一个平台无关的平台路径,描述了目录结构中文件的位置。**Path可以看成是File类的升级版木，实际引用的资源也可以不存在**。

- 在以前IO操作都是这样写的:

  ```java
  import java.io.File;
  File file = new File("index.html");
  ```

  

- 但在Java7中，我们可以这样写:

  ```java
  import java.nio.file.Path;
  import java.nio.file.Paths;
  Path path = Paths.get(""index.html");
  ```

- 同时，NIO.2在java.nio.file包下还提供了Files、Paths工具类，Files包含了大量静态的工具方法来操作文件: Paths则包含了两个返回Path的静态工厂方法。

- Paths类提供的静态 get()方法用来获取Path对象:

  ```java
  static Path get(String first, string ... more):     //用于将多个字符串串连成路径
  static Path get(URl uri):      //返回指定uri对应的Path路径
  ```

  



**Files类：**

![JavaFiles类1](http://image.xpshuai.cn/JavaFiles%E7%B1%BB1.png)

![JavaFiles类2](http://image.xpshuai.cn/JavaFiles%E7%B1%BB2.png)







**题外话:**

 **IDEA导入jar包：**复制到目录下，右击--> add as Library  ， 然后就可以用了(里面放的class文件，可以对它进行反编译)







---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/java%E5%AD%A6%E4%B9%A0%E4%B9%8Bio%E6%B5%81/  

