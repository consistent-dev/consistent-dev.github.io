---
title: "런타임에 설정값 변경하고 적용하기"
excerpt: "runtime-configuration"
date: 2022-02-02
categories:
  - java
  - runtime-configuration
tags:
  - java
  - runtime-configuration
---

# 실행 가능한 예제

본 포스팅에서 다루는 모든 코드는 [깃허브 링크][2]에서 확인 가능합니다.

# 결과부터 보겠습니다.

```java
import conf.Configure;
import util.ThreadUtil;

public class App {

    public static void main(String[] args) {
        for(int i=0; i<10; i++){
            ThreadUtil.sleep(1000);
            System.out.println(Configure.getInstance().debug_test);
            System.out.println(Configure.getInstance().user_name);
        }
    }
}

```

```
// 실행 결과
true
hwanseok
false      // 설정 파일에서 값을 바꾸었더니, 코드 수정 없이 런타임에 값이 변경된다.
hwanseok
false
hwanseok
```

애플리케이션에서 사용하기 위한 여러 설정값들을 my-server.conf라는 파일에 아래와 같이 저장해두었습니다. 

```
// cat my-server.conf
debug_test=true        
user_name=${user.name}
```

애플리케이션이 실행되는 도중에 my-server.conf 파일에서 debug_test의 값을 false로 변경하면, 코드 수정없이 런타임에 서버의 동작을 제어할 수 있게 됩니다.

{: .notice--info}
**Info Notice:**  
"${user.name}"은 시스템 프로퍼티의 값을 Value로 사용하기 위한 기능으로, [문자열에서 파라미터 사용하기][1]에서 소개한 기능을 사용합니다.


# 적용 사례

```java
if(Configure.getInstance().debug_test){
    // debug logging
}
```

배포하기 전에 미리 위와 같이 설정값이 true로 변경되었을 때 로깅을 하도록 세팅해둘 수 있습니다. 중요도가 높은 기능, 에러가 자주 발생하는 기능에서 로깅 옵션을 설정해두면 이미 배포된 기능에서 에러가 발생했을 때 신속하게 원인 파악이 가능해집니다.

# 알고리즘

1. 일정 시간마다 설정 파일을 읽어서 2번 작업을 수행하는 Thread를 생성합니다.
2. 시스템 프로퍼티에서 지정된 설정 파일의 경로를 읽어서  Properties 객체로 로딩합니다.
3. 1번 Thread를 싱글톤으로 구성하고, 원하는 곳에서 Configure.getInstance()와 같이 불러와서 사용합니다.

## Thread 생성

```java
public abstract class ServerConfig extends Thread{

    private String fileName = "server.conf";
    private boolean running = true;
    private long lastChecked = DateUtil.now();
    private String confPath = ".";
    private long lastLoaded = -1;
    public Properties property = new Properties();
    protected ServerConfig(String fileName){
        this.fileName = fileName;
    }

    @Override
    public void run() {
        while(running){
            reload(false);
            ThreadUtil.sleep(3000);
        }
    }
}
```

```java
public class Configure extends ServerConfig {
    private static Configure instance = null;

    public boolean debug_test = false;
    public String user_name = "";

    public final static synchronized Configure getInstance(){
        if(instance == null){
            instance = new Configure();
            instance.setDaemon(true);
            instance.setName(ThreadUtil.getName(instance));
            instance.reload(true);
            instance.start();
        }
        return instance;
    }

    private Configure(){
        super("my-server.conf");
    }

    @Override
    protected void apply() {
        this.debug_test = getBoolean("debug_test", false);
        this.user_name = getValue("user_name", "default_user_name");
    }
}
```

## 파일 읽어서 Properties로 로딩

```java
public abstract class ServerConfig extends Thread{

    private String fileName = "server.conf";
    private boolean running = true;
    private long lastChecked = DateUtil.now();
    private String confPath = ".";
    private long lastLoaded = -1;
    public Properties property = new Properties();
    protected ServerConfig(String fileName){
        this.fileName = fileName;
    }

    @Override
    public void run() {
        while(running){
            reload(false);
            ThreadUtil.sleep(3000);
        }
    }

    public synchronized boolean reload(boolean force){
        // 3초마다 한 번씩 reload
        long now = DateUtil.now();
        if(force == false && now < this.lastChecked + 3000){
            return false;
        }
        this.lastChecked = now;

        // conf 파일을 읽는다.
        File file = getPropertyFile();

        // 수정되지 않았으면 reload하지 않는다.
        if(file.lastModified() == this.lastLoaded){
            return false;
        }
        this.lastLoaded = file.lastModified();

        // 파일 읽어서 properties에 저장
        Properties temp = new Properties();
        if(file.canRead()){
            FileInputStream in = null;
            try {
                in = new FileInputStream(file);
                temp.load(in);
            }catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                FileUtil.close(in);
            }
        }
        this.property = ConfigValueUtil.replaceSysProp(temp);
        apply();
        return true;
    }

    protected abstract void apply();
}
```

# 런타입 설정 값 사용하기

```java
public class App {

    public static void main(String[] args) {
        if(Configure.getInstance().debug_test){
            // debug logging
        }
    }
}
```

# 마치며

런타입에 에러가 발생했을 때 그 순간의 데이터를 바로 확인할 수 있다면 원인 파악을 가장 빨리 할 수 있습니다. 애플리케이션의 고가용성을 지키기 위해 문제가 발생했을 때만 로깅을 찍는 코드를 넣어두면 효과적으로 대응할 수 있습니다.  

이 기능을 활용할 수 있는 사례가 생각나시면 댓글창에서 공유 부탁드립니다.

[1]: ../string-with-parameter
[2]: https://github.com/consistent-dev/archive/tree/main/runtime-configuration