# Java-IO

- File：An abstract reprsentation of file and directory pathnames（既能代表特定的文件名称又能代表一个目录下一组文件的名称）
  - File file = new File("C:\\\\Users\\bug\\Desktop");
  - String files : file.list()读取目录下的文件名
  - 目录创建：if (!file.exists()) file.mkdirs();

- java 1.0 输入输出流
  - InputStream
    - ByteArrayInputStream 缓冲区
    - StringBufferInputStream 字符串
    - FileInputStream 字符串，表示文件名，文件等
    - PipedInputStream 管道化
    - FilterInputStream 作为装饰器接口
  - OutputStream
    - ByteArrayOutputStream
    - FileOutputStream
    - PipedOutputStream
    - FilterOutputStream

- java 1.1 Reader &  Writer
  - 面向字符
  - FileReader
  - FileWriter
  - FilterWriter
  - FilterReader
  - 凡是 readLine() use BufferedReader
  - 其他首选DataInputStream
  - PrintWriter 简化输入写入时文件的创建过程
- I/O流经典使用
  - 缓冲输入文件 BufferedReader(面向字符)

    ```
    public class BufferedReaderFile {
    public static String read(String filepath) {
    String s;
    StringBuffer sb = new StringBuffer();
    try {
    BufferedReader in = new BufferedReader(new FileReader(filepath));
    while ((s = in.readLine()) != null) {
    sb.append(s+"\n");
    }
    in.close();
    } catch (FileNotFoundException e) {
    e.printStackTrace();
    } catch (IOException e) {
    e.printStackTrace();
    }
    return sb.toString();
    }
    public static void main(String[] args) {
    String filepath="C:\\\\Users\\bug\\Desktop\\20151212UPDATE.SQL";
    System.out.println(read(filepath));
    }
    }
    ```

  - 格式化的内存输入

    ```
    public class DataInputStreamFile {
    public static void main(String[] args) {
    try {
    DataInputStream in = new DataInputStream(new ByteArrayInputStream(BufferedReaderFile.read(
    "C:\\\\Users\\bug\\Desktop\\20151212UPDATE.SQL").getBytes()));
    while (true) {
    System.out.println((char) in.readByte());
    }
    } catch (EOFException e) {
    System.err.println("end of stream");
    } catch (IOException e) {
    e.printStackTrace();
    }
    }
    }

    ```

  - 基本文件输出

    ```
    public class BaiscFileOutput {
    public static void main(String[] args) throws IOException {
    String fileout = "/Users/guilinwang/Downloads/out.md";
    String filein = "/Users/guilinwang/github/JAVA/thinking_in_java/reading_notes2.md";
    BufferedReader in = new BufferedReader(new StringReader(BufferedReaderFile.read(fileout)+BufferedReaderFile.read(filein)));
    PrintWriter out = new PrintWriter(fileout);//封装BufferedWriter和FielWriter处理
    String s;
    while ((s = in.readLine())!= null) {
    out.println(s);
    }
    out.close();
    }
    }
    ```