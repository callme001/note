## IO

组成结构图：
字符流
![字符流](http://images.cnitblog.com/blog/399666/201308/19180501-592781219ca446bc8f100d9147759011.jpg)

字节流
![](http://images.cnitblog.com/blog/399666/201308/19180440-050dbb463e4e4919ad8931623598ed15.jpg)
### 1. Java字符流和字节流的区别

Java字符流主要是针对处理一些**字符串**流的工具包，字节流则是针对**二进制数据**流处理的工具包。字节流也可以处理字符串，只不过字符流更加便捷一些。
还有一个区别就是字节流不使用缓冲区的，字符流使用缓冲区，如下图：
![](http://images.51cto.com/files/uploadimg/20090731/162655699.jpg)

### 2. 如何阅读上面的图

从第一点已经直到了字符流和字节流的区别，但是上面的工具包都是“对称”的，Reader对应Writer,InputStream对应OutputStream。而且工具包的命名十分规范，也助于我们记忆。

#### Reader和Writer

以Reader结尾的表示处理的是操作读取字符流，Writer则是写字符流。

举个栗子：OutputStreamWriter和InputStreamReader

```java
@Test
public static void testStreamWriter() throws IOException {
    //写入字符串到文件
    OutputStreamWriter outputStreamWriter = new FileWriter("/home/jincarry/test1");
    outputStreamWriter.write("test OutputStreamWriter\nline 2");
    outputStreamWriter.close();

    //从文件读字符串
    InputStreamReader inputStreamReader = new FileReader("/home/jincarry/test1");
    BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

    char[] chars = new char[100];
    bufferedReader.read(chars);
    bufferedReader.close();
    inputStreamReader.close();
    System.out.println(new String(chars));
}
```

再来一个栗子：	PrintWriter

```java
@Test
public static void testPrintWriter(){
    try {
        PrintWriter printWriter = new PrintWriter(new File("/home/jincarry/test1.txt"));
        printWriter.write(new String("hello word\n2 line"));
        printWriter.write("\n line 3");
        printWriter.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```
用法都是类似的。当我们调用close方法的时候会自己把缓冲区的数据写入到文件，不必手动flush

#### InputStream和OutputStream

InputStream结尾表示的是操作读取的字节流，OutputStream结尾的表示操作的是写字节流。
下面展示了从网络读取Stream流
```java
@Test
public static void testSocketIO() throws IOException {

    newSocket();

    Socket socket = new Socket("127.0.0.1",3389);
    InputStream inputStream = socket.getInputStream();
    StringBuilder stringBuilder = new StringBuilder();
    while (true){
        int temp = inputStream.read();
        if(temp == -1)
            break;
        stringBuilder.append((char) temp);
    }

    inputStream.close();
    socket.close();
    System.out.println(stringBuilder.toString());
}
//建立一个本地socket用于测试
public static void newSocket(){

    Thread thread = new Thread(new Runnable() {
        public void run() {

            while (true){
                ServerSocket ss = null;
                try {
                    ss = new ServerSocket(3389);
                    Socket socket = ss.accept();
                    OutputStream os = socket.getOutputStream();
                    os.write("Welcome!! hello socket".getBytes());
                    os.close();
                    socket.close();
                    ss.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    });

    thread.start();
}
```
解压一个压缩文件并把第一个文件重新压缩
```java
@Test
public static void testZipStream() throws IOException {
    File file = new File("/home/jincarry/test.zip");

    ZipFile zipFile = new ZipFile(file);
    ZipEntry zipEntry = zipFile.entries().nextElement();

    InputStream inputStream = zipFile.getInputStream(zipEntry); //读取第一个文件的流

    ZipOutputStream zipOutputStream = new ZipOutputStream(new FileOutputStream("/home/jincarry/new.zip"));
    zipOutputStream.putNextEntry(new ZipEntry(zipEntry.getName()));

    while (true){
        int temp = inputStream.read();
        if (temp == -1)
            break;
        zipOutputStream.write((char)temp);
    }

    inputStream.close();
    zipOutputStream.close();
}
```
