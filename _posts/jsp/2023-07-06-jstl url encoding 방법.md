---
title: "jstl url encoding 방법"
date: 2023-07-06T00:19:00+09:00
categories: 
    - JSP
share: false
---

# 페이지 목차
1. [jstl url encoding 방법]({% post_url /jsp/2023-07-06-jstl url encoding 방법 %})

## 2023-07-06-jstl url encoding 방법

- jsp를 사용중 jstl을 통해 url 인코딩 하는 방법은 다음과 같다.


- AJAX를 통한 페이지 전환 예시
```jsp
    <c:url value="/display.ajax" var="url">
        <c:param name="fileSvrPath" value="${file}"/>
    </c:url>
    <a href="${url}"></a>
```

- 결과

```jsp
<a href="/display.ajax?fileSvrPath=http%3a%2f%2fnaver.com%2find"></a>
```


- references:
- Written by: 이재봉 (jblee6110@gmail.com)
- reporting date: 2023-07-06
