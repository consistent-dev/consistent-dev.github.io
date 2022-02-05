---
title: "파일 데이터베이스 - 객체 읽고 쓰기"
excerpt: "file-database"
date: 2022-02-05
categories:
    - java
    - file-database
tags:
    - java
    - IO
    - file-database
---

# 실행 가능한 예제

본 포스팅에서 다루는 모든 코드는 [깃허브 링크][1]에서 확인 가능합니다.

# 개념부터 보겠습니다

[파일 데이터베이스 - IO Util 만들기][2]에서 만든 Util을 사용해서 데이터를 저장하고 읽어보겠습니다. 단, 성능과 저장공간의 효율성을 위해서 ObjectInputStream, ObjectOutputSteam을 사용하지 않고 DataInputStream, DataOutputStream을 사용하겠습니다.  

# 코드로 보겠습니다. 

## 모든 데이터의 추상 클래스

애플리케이션에서 다뤄야하는 데이터들에 공통적인 필드가 있다고 가정하겠습니다. 데이터 설계 부분은 예시이기 때문에 눈으로만 보시면 됩니다.

```java
public interface Pack {

    PackType getPackType();
    DataOutputX write(DataOutputX out);
    Pack read(DataInputX in);
}

public enum PackType {
    LOGGING
}
```

```java

public abstract class AbstractPack implements Pack {

    protected long time;
    protected long id;

    protected AbstractPack(long time, long id){
        this.time = time;
        this.id = id;
    }

    public abstract PackType getPackType();

    @Override
    public DataOutputX write(DataOutputX out) {
        byte version = 10;
        out.writeByte(version);
        out.writeDecimal(id);
        out.writeLong(time);
        return out;
    }

    @Override
    public Pack read(DataInputX in) {
        byte version = in.readByte();
        this.id = in.readDecimal();
        this.time = in.readLong();
        return this;
    }
}
```

모든 데이터들의 추상 클래스인 AbstractPack 클래스를 만들었습니다. 모든 데이터가 time과 id 값을 갖도록 하기 위함입니다.  

write()와 read()에서는 바이트 순서에 따라서 쓰고 읽으면 됩니다. [파일 데이터베이스 - IO Util 만들기][2]에서 만들어둔 Util을 대칭적으로 사용하면 됩니다.
1 byte의 version을 넣어둔 것은 혹시 나중에 바이트 순서가 바뀌어야할 수도 있기 때문에 여유 공간 만들어둔 것입니다. 당장은 쓰이지 않습니다.

## 실제 데이터 클래스 

애플리케이션에서 발생하는 로깅 정보를 파일로 저장하는 상황을 가정해보았습니다. 위에서 다룬 내용과 크게 다르지 않습니다.

```java

public class LoggingPack extends AbstractPack {

    private String category; // 로깅 정보 분류를 위한 카테고리 
    private long line; // 몇 번째 라인인지. 추후 indexing에 사용됩니다. 
    private String content; // 내용
    private Map<String, String> tags = new HashMap<>(); // 로깅 정보를 특정할 수 있는 값 1. 추후 search에 사용됩니다.
    private Map<String, String> fields; // 로깅 정보를 특정할 수 있는 값 2

    public LoggingPack(long time, long id, String category, long line, String content) {
        super(time, id);
        this.category = category;
        this.line = line;
        this.content = content;
    }

    @Override
    public PackType getPackType() {
        return PackType.LOGGING;
    }

    @Override
    public DataOutputX write(DataOutputX out) {
        super.write(out);
        byte version = 0;
        out.writeByte(version);
        out.writeText(this.content);
        out.writeLong(this.line);
        out.writeMap(this.tags);
        out.writeMapNullable(this.fields);
        return out;
    }

    @Override
    public Pack read(DataInputX in) {
        super.read(in);
        byte version = in.readByte();
        this.category = in.readText();
        this.line = in.readLong();
        this.tags = in.readMap();
        this.fields = in.readMapNullable();
        return this;
    }
```

## 테스트

```java
public static void main(String[] args) {
    LoggingDB loggingDB = LoggingDB.getInstance();
    try(DataOutputStream dos = new DataOutputStream(new BufferedOutputStream(new FileOutputStream(loggingDB.getRoot())));
        DataInputStream dis = new DataInputStream(new BufferedInputStream(new FileInputStream(loggingDB.getRoot())));
    ){
        DataOutputX dout = new DataOutputX(dos);
        DataInputX din = new DataInputX(dis);
        for(int i=0; i< 10; i++){
            LoggingPack p = new LoggingPack(DateUtil.now(), i, "testCategory" + i, 100 + i,"testContent");
            p.addTag("testKey", "testValue");
            if(i %2 == 0){
                p.addField("testFieldKey", "testFieldValue");
            }
            p.write(dout).flush();
        }

        for(int i=0; i< 10; i++) {
            LoggingPack readPack = (LoggingPack) new LoggingPack().read(din);
            System.out.println(readPack);
        }
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

```java
// 실행 결과
time=1644032977125, pcode=0, category=testContent, line=100, content=null, tags={testKey=testValue}, fields={testFieldKey=testFieldValue}
time=1644032977126, pcode=1, category=testContent, line=101, content=null, tags={testKey=testValue}
time=1644032977126, pcode=2, category=testContent, line=102, content=null, tags={testKey=testValue}, fields={testFieldKey=testFieldValue}
time=1644032977126, pcode=3, category=testContent, line=103, content=null, tags={testKey=testValue}
time=1644032977126, pcode=4, category=testContent, line=104, content=null, tags={testKey=testValue}, fields={testFieldKey=testFieldValue}
time=1644032977126, pcode=5, category=testContent, line=105, content=null, tags={testKey=testValue}
time=1644032977126, pcode=6, category=testContent, line=106, content=null, tags={testKey=testValue}, fields={testFieldKey=testFieldValue}
time=1644032977127, pcode=7, category=testContent, line=107, content=null, tags={testKey=testValue}
time=1644032977127, pcode=8, category=testContent, line=108, content=null, tags={testKey=testValue}, fields={testFieldKey=testFieldValue}
time=1644032977127, pcode=9, category=testContent, line=109, content=null, tags={testKey=testValue}
```

# 마치며

메모리에 있는 객체의 필드를 하나씩 바이트 순서에 맞추어서 저장하고 읽는 과정을 크게 어렵지 않습니다. Util을 잘 만들어두고 사용하면 되니까요.
이제 객체를 파일 데이터베이스에 쓰고 읽는 것이 가능해졌으니, 비즈니스 로직만 고민하면 되겠습니다.   

이 기능을 활용할 수 있는 사례가 생각나시면 댓글창에서 공유 부탁드립니다.

[1]: https://github.com/consistent-dev/archive/tree/main/file-database
[2]: https://consistent-dev.github.io/java/file-database-1-inputStream/