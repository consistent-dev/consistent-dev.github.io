---
title: "파일 데이터베이스 - IO Util 만들기"
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

[파일 데이터베이스 - InputStream 이해하기][2]에서 익숙해진 스트림 클래스들을 실전에서 사용할 수 있게 도와주는 Util을 만들어보겠습니다.  

파일 데이터베이스를 만드는 것을 대부분 애플리케이션에서 발생하는 데이터를 저장하기 위함입니다. 즉, 메모리에 있는 데이터를 파일로 IO 해야합니다.
따라서 DataInputStream, DataOutputStream을 주로 다루게 됩니다. 그리고 당장은 다루지 않겠지만 RandomAccessFile을 사용해야하는 경우도 있습니다.
이 2가지 경우를 모두 커버하기 위해서 DataInput/DataOutput interface를 사용한 util을 만들어보겠습니다.

```java
public class DataInputStream extends FilterInputStream implements DataInput {...}
```

```java
public class DataOutputStream extends FilterOutputStream implements DataOutput { ... }
```

```java
public class RandomAccessFile implements DataOutput, DataInput, Closeable {...}
```

# 코드로 보겠습니다. 

DataInputStream/DataOuputStream을 사용해서 메모리에 있는 데이터를 파일로 IO 해야합니다. 자료를 쓸 때 사용한 메서드와 같은 자료형의 메서드로 읽어야하는 특성 때문에 input, output을 위한 util을 함께 이해해야 합니다. 

## 생성자

먼저 DataInputX라는 클래스를 만들었습니다. DataInputStream, RandomAccessFile 모두를 참조하기 위해 DataInput 인터페이스 타입의 필드를 추가합니다.
RandomAccessFile를 사용한 기능은 추후 별도의 포스팅으로 다루겠습니다.

```java
public class DataInputX {

    private int offset;
    private DataInput inner; 

    public DataInputX(DataInputStream in){
        this.inner = in;
    }

    public DataInputX(BufferedInputStream in){
        this.inner = new DataInputStream(in);
    }
    // TODO
//    public DataInputX(RandomAccessFile in){
//        this.inner = in;
//    }
}
```

비슷하게 DataOutputX라는 클래스를 만들었습니다. DataOutputStream, RandomAccessFile 모두를 참조하기 위해 DataOutput 인터페이스 타입의 필드를 추가합니다.

```java
public class DataOutputX {
    private int written;
    private DataOutput outter; 

    public DataOutputX(DataOutputStream out){
        this.outter = out;
    }

    public DataOutputX(BufferedOutputStream out){
        this.outter = new DataOutputStream(out);
    }
    // TODO
//    public DataInputX(RandomAccessFile in){
//        this.inner = in;
//    }
```

## 1byte 쓰고 읽어보기

현재 DataOutputX.outter는 항상 DataOutputStream의 인스터스만 참조할 수 있습니다. 그리고 DataInputX.inner는 DataInputStream의 인스턴스만 참조합니다.  

1byte를 write하기 위해서 int v를 받아서 (byte)v로 형변환을 해서 writeByte()했습니다.
int형 4바이트 중 하위 1바이트만 write합니다. 
몇 바이트를 write했는지 추적하기 위해 written을 1 증가시킵니다.

{: .notice--info}
**Info Notice:** writeByte(int v) v.s. writeByte(byte v)  
byte 타입이 아닌 int 타입의 값을 전달받은 것은 큰 이유는 없습니다. writeByte(0); 이라고 자주 호출하게 될텐데 그 때마다 wrtieByte((byte)0);라고 형변환을 하기 번거로워서 그렇습니다. 편하신 타입으로 사용하시면 됩니다.

```java
/**
* OutputStream에 1바이트를 씁니다.
*/
public DataOutputX writeByte(int v){
    this.written += 1;
    try{
        this.outter.writeByte((byte)v);
    } catch (IOException e) {
        throw new DataIOException(e);
    }
    return this;
}
```

1byte를 읽습니다. 몇 바이트를 읽었는지 추적하기 위해 offset을 1 증가시킵니다.

```java
/**
* InputStream에서 1바이트를 읽습니다.
*/
public byte readByte(){
    this.offset += 1;
    try{
        return this.inner.readByte();
    } catch (IOException e) {
        throw new DataIOException(e);
    }
}
```

## 4byte를 쓰고 읽어보기

1byte와 비슷하기 때문에 설명은 생략합니다.

```java
public DataOutputX writeInt(int v) {
    this.written += 4;
    try {
        this.outter.write(toBytes(v));
    } catch (IOException e) {
        throw new DataIOException(e);
    }
    return this;
}
```

```java
public int readInt(){
    this.offset += 4;
    try{
        return this.inner.readInt();
    } catch (IOException e) {
        throw new DataIOException(e);
    }
}
```

## 2byte를 쓰고 읽어보기

outter.writeShort()메소드가 있기는 하지만, 바이트 연산에 익숙해지기 위해 shift 연산을 사용해보겠습니다.

shift 연산에는 크게 3가지 종류가 있습니다.  

- \<\< : 제일 오른쪽 공간의 값 = 0
- \>\> : 제일 왼쪽 공간의 값 = 양수 ? 1 : 0
- \>\>\> : 제일 왼쪽 공간의 값 = 0

16진수 0xFF는 1바이트 1111 1111을 의미합니다. (& 0xFF)는 해당 바이트만 추출한다는 뜻입니다.

- v                  = 0000 0000 0011 1100
- v \>\>\> 8            = 0000 0000 0000 0011
- 0xFF               = 0000 0000 0000 1111
- ((v \>\>\> 8) & 0xFF) = 0000 0000 0000 0011
- ((v \>\>\> 0) & 0xFF) = 0000 0000 0000 1100

```java
public DataOutputX writeShort(int v) {
    this.written += 2;
    try {
        this.outter.write(toBytes((short) v));
    } catch (IOException e) {
        throw new DataIOException(e);
    }
    return this;
}

public static byte[] toBytes(short v){
    byte[] buf = new byte[2];
    buf[0] = (byte) ((v >>> 8) & 0xFF);
    buf[1] = (byte) ((v >>> 0) & 0xFF);
    return buf;
}
```

읽을 때는 readShort()로 2바이트만 읽으면 됩니다.

```java
public short readShort(){
    this.offset += 2;
    try{
        return this.inner.readShort();
    } catch (IOException e) {
        throw new DataIOException(e);
    }
}
```

## String 타입 읽고 쓰기

다른 자료형은 위에서 본 내요에서 크게 벗어나지 않습니다. 하지만 String 타입은 조금 다릅니다. 우선 기본적으로 제공되는 writeChars(String s)를 보면 for loop을 사용해서 성능에 제한이 생길 수 있습니다.

```java
public class DataOutputStream extends FilterOutputStream implements DataOutput {
    public final void writeChars(String s) throws IOException {
        int len = s.length();
        for (int i = 0 ; i < len ; i++) {
            int v = s.charAt(i);
            out.write((v >>> 8) & 0xFF);
            out.write((v >>> 0) & 0xFF);
        }
        incCount(len * 2);
    }
}
```

```java
public class RandomAccessFile implements DataOutput, DataInput, Closeable {
    public final void writeChars(String s) throws IOException {
        int clen = s.length();
        int blen = 2*clen;
        byte[] b = new byte[blen];
        char[] c = new char[clen];
        s.getChars(0, clen, c, 0);
        for (int i = 0, j = 0; i < clen; i++) {
            b[j++] = (byte)(c[i] >>> 8);
            b[j++] = (byte)(c[i] >>> 0);
        }
        writeBytes(b, 0, blen);
    }
}
```

우선 쉽게 해결하기 위해서는 다음과 같이 String을 write할 수 있습니다.

```java
public DataOutputX writeText(String s) {
    if (s == null) {
        writeByte((byte) 0);
    } else {
        write(s.getBytes(StandardCharsets.UTF_8));
    }
    return this;
}

public DataOutputX writeByte(int v){
    this.written += 1;
    try{
        this.outter.writeByte((byte)v);
    } catch (IOException e) {
        throw new DataIOException(e);
    }
    return this;
}

public DataOutputX write(byte[] b) {
    this.written += b.length;
    try {
        this.outter.write(b);
    } catch (IOException e) {
        throw new DataIOException(e);
    }
    return this;
}
```

그런데 파일 데이터베이스도 데이터페이스이기 때문에 TEXT, BLOB 데이터에 대해서 최대 길이에 따라서 타입을 구분을 할 수 있어야 합니다.

최대 길이는 MYSQL BLOB Maximum Length에 따르겠습니다.
 
-  TINYBLOB  : maximum length of 255 bytes
-  BLOB      : maximum length of 65,535 bytes
-  MEDIUMBLOB: maximum length of 16,777,215 bytes
-  LONGBLOB  : maximum length of 4,294,967,295 bytes

- unSignedByte  = 1바이트 = 8비트,  2^8 = 256
- unSignedShort = 2바이트 = 16비트, 2^16 = 65536

-  value 길이에 따른 데이터 프로토콜
    1. 253이하   : \[value 길이(1~253) 1바이트\]\[value\]
    2. 65535이하 : \[254 1바이트\]\[value 길이 2바이트\]\[value\]
    3. 65535초과 : \[255 1바이트\]\[value 길이 4바이트\]\[value\]

위에서 만든 규칙에 따라서 데이터의 길이를 저장하고, 데이터를 저장하는 방법을 사용합니다.  

```java
    private static final int TINYBLOB_MAX = 253; // TINYBLOB의 최대 길이를 255로 사용하면, 첫 바이트만 읽어서 데이터의 길이인지 PREFIX인지 구분하지 못한다.
    private static final int BLOB_MAX = 65535;
    private static final int EMPTY_VALUE_PREFIX = 0;
    private static final int BLOB_PREFIX = 254; // byte[]의 길이가 TINYBLOB_MAX를 넘는다는 것을 나타내기 위한 값
    private static final int MEDIUMBLOB_PREFIX = 255; // byte[]의 길이가 BLOB_MAX를 넘는다는 것을 나타내기 위한 값

    public DataOutputX writeBlob(byte[] value) {
        if (value == null || value.length == 0) {
            writeByte((byte) EMPTY_VALUE_PREFIX);
        } else {
            int len = value.length;
            if (len <= TINYBLOB_MAX) {
                writeByte((byte) len);
                write(value);
            } else if (len <= BLOB_MAX) {
                byte[] buff = new byte[3];
                buff[0] = (byte) BLOB_PREFIX;
                write(toBytes(buff, 1, (short) len));
                write(value);
            } else {
                byte[] buff = new byte[5];
                buff[0] = (byte) MEDIUMBLOB_PREFIX;
                write(toBytes(buff, 1, len));
                write(value);
            }
        }
        return this;
    }
```

```java
public byte[] readBlob() {
    int baselen = readUnsignedByte();
    switch (baselen) {
        case BLOB_PREFIX: {
            int len = readUnsignedShort();
            byte[] buffer = read(len);
            return buffer;
        }
        case MEDIUMBLOB_PREFIX: {
            int len = this.readInt();
            byte[] buffer = read(len);
            return buffer;
        }
        case EMPTY_VALUE_PREFIX: {
            return new byte[0];
        }
        default:
            byte[] buffer = read(baselen);
            return buffer;
    }
}
```


# 마치며

DataInputStream, DataOutputStream을 한단계 추상화해둔 Util을 사용하면 좀 더 직관적으로 개발을 할 수 있습니다.
코드의 양이 많지만 자료형의 크기와 사이즈에 따라 바이트를 다루는 방법이 크게 다르지 않습니다.
다음 포스팅에서는 실제로 데이터 객체를 생성한 뒤 read, write 해보는 시간을 가지겠습니다.  

이 기능을 활용할 수 있는 사례가 생각나시면 댓글창에서 공유 부탁드립니다.

[1]: https://github.com/consistent-dev/archive/tree/main/file-database
[2]: https://consistent-dev.github.io/java/file-database-1-inputStream/