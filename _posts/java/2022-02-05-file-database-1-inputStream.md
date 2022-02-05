---
title: "파일 데이터베이스 - InputStream 이해하기"
excerpt: "file-database"
date: 2022-02-05
categories:
    - java
    - file-database
tags:
    - java
    - file-database
    - IO
---

# 실행 가능한 예제

본 포스팅에서 다루는 모든 코드는 [깃허브 링크][1]에서 확인 가능합니다.

# 개념부터 보겠습니다

파일 IO의 개념을 처음 접하면 굉장히 혼란스럽습니다. 여러 클래스들이 어떤 구조를 이루고 있는지, 어떤 것을 사용해야하는지 이해하기 참 어렵습니다.
하지만 알고보면 알아야할 개념이 많지 않습니다.  

우선, 자바에서 모든 입출력은 Stream(이하 스트림)을 통해 이루어집니다. 컴퓨터에 연결될 수 있는 입출력 장치는 무한히 많습니다. 
입출력 장치에 무관하게 일관성있게 파일 IO를 사용한 개발을 할 수 있도록 가상의 통로인 스트림이 제공됩니다.

스트림을 다루기 위한 여러 클래스들이 제공되며, 이러한 클래스들은 아래와 같이 3가지 분류로 구분될 수 있습니다.

- 입력 스트림, 출력 스트림
   - 입력 : *InputStream, *Reader
   - 출력 : *OutputStream, *Writer
- 바이트 스트림, 문자 스트림
   - 바이트 : *InputStream, *OutputStream
   - 문자 : *Reader, *Writer
- 기반 스트림, 보조 스트림
   - 기반 : File*
   - 보조 : Input*, Output*, BufferedInput*, BufferedOutput*, Data*

## 입력 스트림, 출력 스트림

하나의 파일에 대해서 IO를 할 때에도 입력과 출력 스트림을 각각 생성해야 합니다. 데이터의 입력과 출력이 하나의 스트림에서 동시에 일어날 수 없습니다.

## 바이트 스트림, 문자 스트림

그림, 동영상, 음악 등 대부분의 파일은 바이트 단위로 읽거나 쓰면 됩니다. 그런데 자바에서는 문자를 나타내는 Char형은 2바이트이기 때문에 1바이트만 읽고 쓰면 한글과 같은 문자는 꺠집니다.
따라서 문자를 위해 문자 스트림을 별도로 제공합니다.

## 기반 스트림, 보조 스트림

기반 스트림은 직접 데이터를 읽거나 씁니다. 보조 스트림은 기반 스트림에 부가 기능을 제공하는 역할을 합니다. 보조 스트림만 단독으로 사용하지는 않습니다. 

아래와 같은 코드에서 FileOutputStream만 기반 스트림이고, DataOutputStream, BufferedOutputStream은 부가 기능을 제공합니다.

```java
// 많이 길죠? 바로 아래에서 설명있습니다.
DataOutputStream dos = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("test.txt"));
```

## 데코레이터 패턴

아래 사진은 모두 입력 스트림에 대한 그림입니다. 출력 스트림도 매우 똑같습니다. 스트림 클래스는 매우 많은데요, 우선 자주 쓰이는 (그리고 제가 파일 데이터 베이스 관련 게시물에서 사용할) 클래스만 보겠습니다. 

![그림 1. InputStream 하위 클래스에서 사용되는 데코레이터 패턴](/assets/gliffy/java/inputStream.jpeg)
*그림 1. InputStream 하위 클래스에서 사용되는 데코레이터 패턴*

- 노란색 : 바이트 스트림 + 기반 스트림
- 초록색 : 바이트 스트림 + 보조 스트림
- 파란색 : 문자 스트림 + 기반 스트림

기반 스트림은 단독으로 쓰일 수도 있고, 보조 스트림 1..\*개로 감싸고 사용할 수도 있습니다.

아직 감이 잘 안올거에요. 바로 동작하는 코드로 보면 좀 더 쉬워집니다.

# 코드로 보겠습니다

[깃허브 링크의 tutorial package][2] 하위에 있는 클래스 별로 하나씩 설명하겠습니다.  

## tutorial.FileInputStreamTest

파일 객체를 하나 만들고, 기반 스트림 하나만 만들었습니다. 그리고 1와 'R'이라는 문자를 쓰고 읽었습니다.

```java
File file = FileUtil.getFile("test1.txt");
 try(FileInputStream fis = new FileInputStream(file);
    FileOutputStream fos = new FileOutputStream(file);){

    fos.write(1);
    fos.write('R');
    // 출력 스트림의 close() 메서드 안에서 flush() 메서드를 호출합니다.
    // 저장만하고 다시 읽지 않는다면 flush()를 호출하지 않아도 됩니다.
    fos.flush();

    System.out.println(fis.read());
    System.out.println((char)fis.read());
} catch (IOException e) {
    ExceptionUtil.ignore(e);
}
```

```java
// 실행 결과 
1
R
```

당연히 쓴대로 읽어집니다.  

중요한 점은 test1.txt라는 파일에 아래와 같이 바이트로 쓰여져서 사람이 제대로 읽을 수 없는 형식이라는 점입니다.

![그림 2. InputStream으로 write하면 바이트로 저장되어 사람이 읽을 수 없다](/assets/gliffy/java/inputStream_tutorial_0.png)  
*그림 2. InputStream으로 write하면 바이트로 저장되어 사람이 읽을 수 없다*

{: .notice--info}
**Info Notice:** append 모드  
매번 실행할 때마다 파일에 적힌 값이 사라졌다가, 새롭게 써집니다. 만약 파일에 계속 값을 추가하고 싶으면 outputStream을 append 모드로 열어야 합니다. FileOutputStream fos = new FileOutputStream(file,true)

{: .notice--info}
**Info Notice:** flush  
flush()는 출력 스트림에 있는 데이터를 모두 비우는 역할을 수행합니다. close()가 호출될 때 flush()가 자동으로 호출됩니다. 위 예시에서는 flush()는 하지 않으면, read()를 하기 전에 출력 스트림에 있는 데이터가 파일로 보내지지 않아서 read()에서 에러가 발생합니다.

## tutorial.FileReaderTest

파일 객체를 하나 만들고, 문자 스트림 하나만 만들었습니다. 그리고 '1'와 'R'이라는 문자를 쓰고 읽었습니다.

Reader는 문자 스트림이기 때문에 숫자 1이 아닌 문자 '1'을 사용했습니다.  

```java
File file = FileUtil.getFile("test1.txt");
try(FileReader fileReader = new FileReader(file);
    FileWriter fileWriter = new FileWriter(file);){

    fileWriter.write('1');
    fileWriter.write('R');
    fileWriter.flush();

    System.out.println((char)fileReader.read());
    System.out.println((char)fileReader.read());
} catch (IOException e) {
    ExceptionUtil.ignore(e);
}
```

```java
// 실행 결과 
1
R
```

![그림 3. Reader로 write하면 문자로 저장되어 사람이 읽을 수 있다](/assets/gliffy/java/inputStream_tutorial_1.png)  
*그림 3. Reader로 write하면 문자로 저장되어 사람이 읽을 수 있다*


## tutorial.BufferedFileInputStreamTest

파일 객체를 하나 만들고, 기반 스트림 하나만 만들었습니다. 그리고 BufferedInputStream이라는 보조 스트림으로 감쌌습니다.  

BufferedInputStream은 8192바이트 크기의 배열(Buffer)을 가지고 있습니다. 8K 정보를 한 번에 읽고 쓸 수 있습니다. 만약 Buffer가 없다면 loop을 돌면서 한 바이트씩 읽고 써야해서 느립니다.  

```java
File file = FileUtil.getFile("test1.txt");
File copy = FileUtil.getFile("copy.txt");

try(FileInputStream fis = new FileInputStream(file);
    FileInputStream cfis = new FileInputStream(copy);

    BufferedInputStream bfis = new BufferedInputStream(new FileInputStream(file));
    FileOutputStream fos = new FileOutputStream(file, true);
    FileOutputStream cfos = new FileOutputStream(copy);
    BufferedOutputStream bcfos = new BufferedOutputStream(new FileOutputStream(copy));){

    // 임의의 데이터를 저장합니다.
    for(int j=0; j<1_000_000; j++){
        fos.write(j);
    }
    fos.flush();

    int i = 0;

    // 한 바이트씩 읽을 때 시간이 얼마나 걸리는지 확인합니다.
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    while((i = fis.read()) != -1){
        cfos.write(i);
    }
    System.out.println(stopWatch.stop());

    i = 0;
    // 버퍼를 사용해서 읽을 때 얼마나 시간이 걸리는지 확인합니다.
    stopWatch.start();
    while((i = bfis.read()) != -1){
        bcfos.write(i);
    }
    System.out.println(stopWatch.stop());
} catch (IOException e) {
    ExceptionUtil.ignore(e);
}
```

```java
// 실행 결과 
2247 // 버퍼 사용하지 않았을 때(ms)
32   // 버퍼 사용했을 때 (ms)
```

이처럼 보조 스트림은 기반 스트림에 부가적인 기능을 제공합니다.  


## tutorial.DataFileInputStreamTest

파일 객체를 하나 만들고, 기반 스트림 하나만 만들었습니다. 그리고 DataInputStream이라는 보조 스트림으로 감쌌습니다.  

FileInputStream은 사람이 읽고 쓰는 텍스트 형식의 데이터를 다루었습니다. 반면에 DataInputStream, DataOutputStream은 메모리에 저장된 0,1 상태를 그대로 읽거나 씁니다. 따라서 자료형의 크기가 그대로 보존됩니다. 자료형을 그대로 읽고 쓰는 스틀미이기 떄문에 같은 정수라도 자료형에 따라 다르게 처리됩니다. 자료를 쓸 때 사용한 메서드와 같은 자료형의 메서드로 읽어야 합니다.  

{: .notice--info}
**Info Notice:** dataInputStream  
A data input stream lets an application read primitive Java data types from an underlying input stream in a machine-independent way. An application uses a data output stream to write data that can later be read by a data input stream.


```java
File file = FileUtil.getFile("test1.txt");
File copy = FileUtil.getFile("copy.txt");

try(DataInputStream dis = new DataInputStream(new FileInputStream(file));
    DataOutputStream dos = new DataOutputStream(new FileOutputStream(file));
    ){
    dos.write(1024+8); // 하위 8비트만 write 0010 0000 1000
    dos.writeInt(1024+8); // 32비트 모두 write 0010 0000 0000
    System.out.println(dis.read());
    System.out.println(dis.readInt());
} catch (IOException e) {
    ExceptionUtil.ignore(e);
}
```

```java
// 실행 결과 
8
1032
```

DataInputStream이 파일 데이터베이스를 만들 때 가장 중심이 되는 스트림입니다. int형 변수를 하나 write하더라도 4바이트를 모두 write할 수도 있고, 하위 1바이트만 write할 수도 있습니다. 가장 최적화된 데이터만 입출력을 하면 속도가 빨라집니다. 저장 공간이 덜 사용되는 것도 장점입니다.  

# BufferedInputStream vs ByteArrayInputStream

위 두 가지 클래스의 차이점은 제가 정말 헷갈려했던 부분입니다. 문서의 설명을 먼저 보겠습니다.  

## BufferedInputStream

{: .notice--info}
**Info Notice:** dataInputStream  
A data input stream lets an application read primitive Java data types from an underlying input stream in a machine-independent way. An application uses a data output stream to write data that can later be read by a data input stream.


```java
public
class DataInputStream extends FilterInputStream implements DataInput {
    /**
     * Creates a DataInputStream that uses the specified
     * underlying InputStream.
     *
     * @param  in   the specified input stream
     */
    public DataInputStream(InputStream in) {
        super(in);
    }
```

## ByteArrayInputStream

{: .notice--info}
**Info Notice:** ByteArrayInputStream  
A ByteArrayInputStream contains an internal buffer that contains bytes that may be read from the stream. An internal counter keeps track of the next byte to be supplied by the read method.

```java
public
class ByteArrayInputStream extends InputStream {

    /**
     * Creates a <code>ByteArrayInputStream</code>
     * so that it  uses <code>buf</code> as its
     * buffer array.
     * The buffer array is not copied.
     * The initial value of <code>pos</code>
     * is <code>0</code> and the initial value
     * of  <code>count</code> is the length of
     * <code>buf</code>.
     *
     * @param   buf   the input buffer.
     */
    public ByteArrayInputStream(byte buf[]) {
        this.buf = buf;
        this.pos = 0;
        this.count = buf.length;
    }
```

## 차이점

ByteArrayInputStream은 생성자가 byte[] 입니다. InputStream이 아닙니다. 즉, ByteArrayInputStream은 다른 파일 시스템과 IO를 전혀하지 않습니다. 메모리 안에서 byte[]에 값을 read/write하는 기능을 수행합니다.  

반면에 BufferedInputStream은 기반 스트림에 버퍼를 추가하는 역할을 수행합니다. 실제로 파일 시스템과 IO를 합니다.

# 마치며

InputStream에 대한 최소한의 이해가 되셨길 바랍니다. 이제 다른 스트림 클래스를 보더라도 좀 더 쉽게 이해할 수 있습니다. 다음 시간에는 DataInputStream을 사용해서 실전에서 사용할 수 있는 유틸을 만들어보겠습니다.  

이 기능을 활용할 수 있는 사례가 생각나시면 댓글창에서 공유 부탁드립니다.

[1]: https://github.com/consistent-dev/archive/tree/main/file-database
[2]: https://github.com/consistent-dev/archive/tree/main/file-database/src/main/java/tutorial