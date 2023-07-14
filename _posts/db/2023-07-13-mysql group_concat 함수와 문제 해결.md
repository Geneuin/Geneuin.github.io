---
title: "mysql group_concat 함수와 문제 해결"
date: 2023-07-13T00:19:00+09:00
categories: 
    - DB
share: false
---

# 페이지 목차
1. [mysql group_concat 함수와 문제 해결]({% post_url /db/2023-07-13-mysql group_concat 함수와 문제 해결 %})

## postgresql 백업
group_concat 함수란? 데이터에 list를 하나의 raw로 보여주는 mysql 함수 이다.

사용중 너무 데이터가 길어지면 짤리는 증상이 발생하였다. 
해결 방법은 다음과 같다.

1. mysql의 현재 group_concat_max_len 확인

```sql
  SHOW VARIABLES LIKE 'group_concat_max_len';
```
결과는 102400 default값이 존재한다. 해당 값을 늘려주면 해결 할 수 있다.


2. group_concat_max_len 값 늘리기
```sql
  SET @@session.group_concat_max_len = 1000000;
```

3. my.ini 파일 적용
group_concat_max_len = 1000000


1. mysql의 현재 group_concat_max_len 확인

```sql
  SHOW VARIABLES LIKE 'group_concat_max_len';
```
정상 적용 된것을 확인 할 수 있다.

- references:
- Written by: 이재봉 (jblee6110@gmail.com)
- reporting date: 2023-07-13