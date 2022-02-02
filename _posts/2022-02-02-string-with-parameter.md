---
title: "문자열에서 파라미터 사용하기"
excerpt: "string-with-parameter"
date: 2022-02-02
categories:
  - java
tags:
  - java
---

# 결과부터 보겠습니다.

```java
import util.ParamText;

import java.util.HashMap;
import java.util.Map;

public class App {

    public static void main(String[] args) {
        ParamText paramText = new ParamText("log-${category1}-${category2}.conf");
        String category = "testCategoryName";
        System.out.println(paramText.replaceAll(category));

        Map<Object, Object> arg = new HashMap<>();
        arg.put("category1", "testCategoryName_1");
        arg.put("category2", "testCategoryName_2");
        System.out.println(paramText.replaceAllMap(arg));
    }
}
```

```
// 실행 결과
logsink-testCategoryName-testCategoryName.conf
logsink-testCategoryName_1-testCategoryName_2.conf
```

"log-\${category1}-\${category2}.conf"이라는 값을 가지는 문자열이 있습니다. 이 문자열은 특이하게도 "${}"를 사용해서 파라미터를 지정한 문자열입니다.  

- ParamText#replaceAll()을 사용하면 해당 문자열에 포함된 모든 파라미터를 하나의 값으로 치환할 수 있습니다. 
- ParamText#replaceAllMap()을 사용하면 파라미터의 이름에 맞추어서 적절한 값을 치환할 수도 있습니다.  

# 적용 사례

하나의 데이터를 가공해서 여러 가지 데이터를 저장해야하는 경우가 있습니다. 파일의 이름은 똑같고 확장자만 다르게해서 저장되는 데이터의 성격을 구분하고 싶은 경우입니다.

- log-category.idx
- log-category.dat

이 경우 "log", "category", "log-category" 모두 파라미터로 대체될 수 있어보입니다. 다음과 같이 파일 이름을 지정할 때 파라미터를 사용하면, ParamText를 사용해서 불필요한 작업을 덜 수 있을 것 같습니다.

- "${type}-{$subType}.idx"
- "${type}-{$subType}.dat"
- "${fileName}.idx"
- "${fileName}.dat"

# 알고리즘

1. 평문이 들어오면 "${}"를 기준으로 토큰화해서 잘라둔다.
2. 파라미터를 치환할 때 StringBuffer로 다시 합친다. 


## 평문 토크닝

```java
public class ParamText {

    protected String startBrace;
    protected String endBrace;
    protected List<Object> tokenList = new ArrayList<>(); // 토큰 리스트. 평문은 String, 파라미터는 OBJECT 타입으로 add한다.
    protected List<String> paramList = new ArrayList<>(); // 파라미터 리스트

    public ParamText(String plainText){
        this(plainText, "${", "}");
    }

    private ParamText(String plainText, String startBrace, String endBrace){
        this.startBrace = startBrace;
        this.endBrace = endBrace;
        while(plainText.length() > 0){
            int pos = plainText.indexOf(startBrace);
            if(pos < 0){ // 파라미터가 없는 경우
                this.tokenList.add(plainText);
                return;
            }else if(pos > 0){ // 파라미터가 중간에 있는 경우
                this.tokenList.add(plainText.substring(0, pos));
                plainText = plainText.substring(pos);
            }else{ // 파라미터가 제일 앞에 있는 경우
                pos += startBrace.length();
                int nextPos = plainText.indexOf(endBrace, pos);
                if(nextPos < 0){
                    break;
                }
                String paramName = plainText.substring(pos, nextPos).trim();
                this.paramList.add(paramName);
                this.tokenList.add(new OBJECT(paramName));
                plainText = plainText.substring(nextPos + endBrace.length());
            }
        }
    }
}
```

## 파라미터 치환

```java
   /**
     * 모든 파라미터를 하나의 값으로 대체한다.
     *
     * @param arg 모든 파라미터를 대체할 값
     * @return
     */
    public String replaceAll(Object arg){
        StringBuffer buffer = new StringBuffer();
        for(Object o : tokenList){
            if(o instanceof OBJECT){ // 파라미터인 경우 대체
                buffer.append(arg);
            }else{
                buffer.append(o);
            }
        }
        return buffer.toString();
    }

    /**
     * 파라미터를 대체할 Map을 전달한다.
     *
     * @param arg 파라미터를 대체할 값
     * @return
     */
    public String replaceAllMap(Map<Object, ?> arg){
        StringBuffer buffer = new StringBuffer();
        for(Object o : tokenList){
            if(o instanceof OBJECT){ // 파라미터인 경우 대체
                OBJECT<String> p = (OBJECT<String>) o;
                if(arg.containsKey(p.value)){
                    buffer.append(arg.get(p.value));
                }else{
                    buffer.append(o);
                }
            }else{
                buffer.append(o);
            }
        }
        return buffer.toString();
    }
```

# 마치며

문자열에서 파라미터를 사용할 수 있는 기능은 익숙해지면 다방면에서 활용할 수 있을 것 같습니다. 이 기능을 활용할 수 있는 사례가 떠오르면 댓글창에서 공유하면 좋겠습니다.  