---
title: "filter와 interceptor의 차이"
date: 2023-07-07T00:21:00+09:00
categories: 
    - spring
share: false
---

# 페이지 목차
1. [filter와 interceptor의 차이]({% post_url /jsp/2023-07-07-filter와 interceptor의 차이 %})

## 2023-07-07-filter와 interceptor의 차이

- Spring은 공통으로 여러 작업을 처리할 수 있도록 기능들을 지원한다. 그중 filter와 interceptor를 확인하고 그 차이를 비교해보았다.


- filter 란?
필터란 디스패처서블릿에 요청이 전달되기 전/후에 작업을 할 수 있는 기능이다. 그러므로 spring 범위 밖에서 처리가 된다. 스프링컨테이너가 아닌 웹(톰캣, 제우스 등) 컨테이너에서 처리하는것을 말한다.
 ![1-1](/images/spring/filterContext.png)


 -filter의 구성 
 필터는 Filter인터페이스를 implements 하여 사용이 가능하다.  다음 예제는 Filter를 구체화하는 소스 코드 예제 이다.

```java
package javax.servlet;

import java.io.IOException;

public interface Filter {
    default void init(FilterConfig filterConfig) throws ServletException {
    }

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    default void destroy() {
    }
}
```
init : filter 객체를 생성하기위해 처음 1번 실행되는 methods이다.

destory : filter를 반환하기위한 methods 이다.

doFilter : url 요청이 디스패처서블릿으로 전달되기 전 실행하는 methods 이다. param 중 FilterChain의 do.Filter를 통해 다음 대상으로 전달된다.


-interceptor 이란?
디스패처서블릿이 컨트롤러에 요청이 전달되기 전/후에 작업을 할 수 있는 기능이다. spring 범위 내에서 처리가 된다.
 ![1-2](/images/spring/spriongContext.png)

-interceptor 의 구성
 인터셉터는 HandlerInterceptor인터페이스를 implements 하여 사용이 가능하다. 

```java
package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.lang.Nullable;

public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
}

```
preHandle : preHandle는 컨트롤러에 요청이 전달되기 전 실행된다.

postHandle : postHandle는 컨트롤러에 요청이 전달된 후 실행된다. 작업중 controller 단에서 예외가 발생하면 실행되지 않는다.

afterCompletion : afterCompletion는 모든 작업이 완료된 후 실행된다. 작업중 controller단에서 예외가 발생해도 실행된다.

### 필자는 filter는 보안작업 (크로스사이트스크립트 등),spring과 분리 필요한 기능, 모든 요청 로깅의 기능으로 사용 하고 interceptor은 사용자 권한 작업, API 호출의 로깅 작업등 을 토대로 사용하고있다.

- references:
- Written by: 이재봉 (jblee6110@gmail.com)
- reporting date: 2023-07-07
