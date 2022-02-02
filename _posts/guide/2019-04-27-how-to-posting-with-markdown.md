---
title: "Markdown을 사용한 Post 작성 가이드"
excerpt: ""
date: 2019-04-27T15:34:30-04:00
categories:
  - Guide
tags:
  - Markdown
  - minimal-mistakes 
teaser: /assets/images/teaser/guide.jpg
toc : true
toc_label: "=== Contents ==="
toc_icon: "heart"  # corresponding Font Awesome icon name (without fa prefix)
toc_sticky: true
classes: wide
---

## NOTICE

```
**Notice:**  
기본적인 Notice
{: .notice}

**Primary Notice:**  
중요한 Notice
{: .notice--primary}

**Info Notice:**  
정보 Notice
{: .notice--info}

**Warning Notice:**  
경고 Notice
{: .notice--warning}

**Danger Notice:**  
Danger Notice
{: .notice--danger}

**Success Notice:**  
Success Notice
{: .notice--success}
```
**Changes in Service:**  
기본적인 Notice
{: .notice}

**Primary Notice:**  
중요한 Notice 
{: .notice--primary}

**Info Notice:**  
정보 Notice
{: .notice--info}

**Warning Notice:**  
경고 Notice
{: .notice--warning}

**Danger Notice:**  
Danger Notice
{: .notice--danger}

**Success Notice:**  
Success Notice
{: .notice--success}



Main Title : 이 포스트는 mmistakes의 minimal-mistakes 
===========

Sub Title : jekyll theme에 최적화 되어있습니다. 
-----------

## Main Title
```
Main Title    // YAML Front Matter 바로 아래에
===========
```

## Sub Title
```
Sub Title     // Maint title의 아래에
-----------
```
## YAML front matter 
```yml
---
title: "Markdown을 사용한 Post 작성 가이드"
excerpt: "MD부터 YAML front Matter까지"
date: 2019-04-27T15:34:30-04:00
categories:
  - Guide
tags:
  - Markdown
toc : true
toc_label: "=== Contents ==="
toc_icon: "heart"  # corresponding Font Awesome icon name (without fa prefix)
toc_sticky: true
classes: wide
---
```


## H1~H6
```
# 제목 1
## 제목 2
### 제목 3
#### 제목 4 : https://mmistakes.github.io/minimal-mistakes/
##### 제목 5
###### 제목 6
```
# 제목 1
## 제목 2
### 제목 3
#### 제목 4 : https://mmistakes.github.io/minimal-mistakes/
##### 제목 5
###### 제목 6

## 평서문
```
평서문은 그냥 적습니다.
```
평서문은 그냥 적습니다.

## 강조
```
1. 평서문 중간에 `강조`를 싶을 때 
3. *기울이기* 또는 _기울이기_
4. **Bold** 또는 __Bold__
1. ~~취소~~
1. <u>밑줄</u>
1. -, *, +를 사용한 순서가 없는 list
```
1. 평서문 중간에 `강조`를 싶을 때 
3. *기울이기* 또는 _기울이기_
4. **Bold** 또는 __Bold__
1. ~~취소~~
1. <u>밑줄</u>
1. -, *, +를 사용한 순서가 없는 list

## 하이퍼링크
```
- 하이퍼 링크를 작성하는 방법
  - [GOOGLE](https://google.com)
  - [NAVER](https://naver.com "링크 설명(title)을 작성하세요.")
  - [GitHub1][1] "본문을 복잡하지 않게 하기 위함"
  - [GitHub2][2]
  - [GitHub3][3]
  - [GitHub4][4]
  - 구글 홈페이지: https://google.com
  - 네이버 홈페이지: <https://naver.com>

[1]: https://github.com
[2]: https://github.com
[3]: https://github.com
[4]: https://github.com
```
- 하이퍼 링크를 작성하는 방법
  - [GOOGLE](https://google.com)
  - [NAVER](https://naver.com "링크 설명(title)을 작성하세요.")
  - [GitHub1][1] "본문을 복잡하지 않게 하기 위함"
  - [GitHub2][2]
  - [GitHub3][3]
  - [GitHub4][4]
  - 구글 홈페이지: https://google.com
  - 네이버 홈페이지: <https://naver.com>

[1]: https://github.com
[2]: https://github.com
[3]: https://github.com
[4]: https://github.com

## 이미지
```
* 이미지를 등록하는 방법 : 개행을 하지 않으면 되도록 정렬되서 출력됨.
[![imagename1](/assets/images/500x300.jpg)](https://www.naver.com)
![imagename2](/assets/images/500x300.jpg)
```

[![imagename1](/assets/images/500x300.jpg)](https://www.naver.com)
![imagename2](/assets/images/500x300.jpg)


## 코드 블록
```
- 코드를 등록하는 방법
  // 포멧이 깨지는 것을 방지하기 위해 '로 작성함. 실제로는 ' 대신 `를 사용할 것
  '''Code-type
  Code
  '''

```
  -bash
  
  ```bash
  $ vim ./~zshrc
  ```
  -python
  
  ```python
  s = "Python syntax highlighting"
  print s
  ```
  -c++
 
  ```cpp
  int main (void){ return 0;}
  ```


## Table
```
- 표 그리기
| 왼쪽 정렬 | 가운데 정렬 | 오른쪽 정렬 |
|:--------|:--------:|--------:|
| 내용 11 | 내용 12 | 내용 13 |
| 내용 21 | 내용 22 | 내용 23 |

|  <center>Header1</center> |  <center>Header2</center> |  <center>Header3</center> |
|:--------|:--------:|--------:|
|**cell 1x1** | <center>cell 1x2 </center> |*cell 1x3* |
|**cell 2x1** | <center>cell 2x2 </center> |*cell 2x3* |
|**cell 3x1** | <center>cell 3x2 </center> |*cell 3x3* |
```

| 왼쪽 정렬 | 가운데 정렬 | 오른쪽 정렬 |
|:--------|:--------:|--------:|
| 내용 11 | 내용 12 | 내용 13 |
| 내용 21 | 내용 22 | 내용 23 |

|  <center>Header1</center> |  <center>Header2</center> |  <center>Header3</center> |
|:--------|:--------:|--------:|
|**cell 1x1** | <center>cell 1x2 </center> |*cell 1x3* |
|**cell 2x1** | <center>cell 2x2 </center> |*cell 2x3* |
|**cell 3x1** | <center>cell 3x2 </center> |*cell 3x3* |

## 인용문
```
>인용문
>>인용문
```
>인용문
>>인용문

## 구분선
```
구분선
---
```
구분선
---

## 개행
```
문장의 끝에서 띄어쓰기 두번이면 줄바꿈 됩니다.  
br tag로 줄바꿈 <br> 됩니다.
```
문장의 끝에서 띄어쓰기 두번이면 줄바꿈 됩니다.  
br tag로 줄바꿈 <br> 됩니다.
