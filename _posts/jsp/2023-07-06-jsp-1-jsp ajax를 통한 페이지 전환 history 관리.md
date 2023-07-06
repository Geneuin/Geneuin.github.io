---
title: "ajax를 통한 페이지 전환 history 관리"
date: 2023-07-06T22:11:00+09:00
categories: 
    - jsp
share: false
---

# 페이지 목차
2. [jsp ajax를 통한 페이지 전환 history 관리]({% post_url /jsp/2023-07-06-jsp-1-jsp ajax를 통한 페이지 전환 history 관리 %})

## jsp ajax를 통한 페이지 전환 history 관리

-vue나 react와 같은 SPA(SINGLE PAGE APPLICATION) 이 아닌 SSR(SERVER SIDE RENDERING)인경우, 공통부분을 매 패이지 전환시, 불러와야하는 불편함이 존재한다. 하여 ajax를 통하여 content 부분만 변경해 주는 경우가 있다.
- 

- AJAX를 통한 페이지 전환 예시
```jsp
 jQuery.ajax({
            url : "/test.do"
            , type : "POST"
            , data : data
            , dataType: "HTML"
            , success:function(result) {
                $('#grid_main_view').children().remove();
                $("#grid_main_view").html(result);

            }
            , error: function (jqXHR){
                alert(jqXHR.responseText);
            }
        });
```
위에서 중요한 부분은 dataType을 html로 return 받아 content 부분에 바인딩 시켜주는 방식이다. 허나 위와 같은 방식에는 단점이 있다. history back 즉 뒤로가기 시, history가 없어 재 기능을 하지 못한다.

하여 아래와 같이 강제로 history를 부여해줄 필요가 있다.

```jsp
 jQuery.ajax({
            url : "/test.do"
            , type : "POST"
            , data : data
            , dataType: "HTML"
            , success:function(result) {
                // url history 등록  history.pushState(parameter,title, url);
                history.pushState({list:result},'', '/test.do##');
                $('#grid_main_view').children().remove();
                $("#grid_main_view").html(result);

            }
            , error: function (jqXHR){
                alert(jqXHR.responseText);
            }
        });
```
위와같이, history.pushState를 통하여 history를 등록하면, history가 추가되고, popstate event를 통해 historyBack시 catch가 가능해 진다. 
history에 ##을 붙인 이유는 아래 코드와 같이 history가 존재하는지 체크해 주기 위함이다. popstate 이벤트는 아래와 같다.


```jsp
$(window).on('popstate', function(event) {
            if(location.hash){
                var data = history.state;
                if(data){
                    $('#grid_main_view').children().remove();
                    $("#grid_main_view").html(data.data);
                }
            }
        });
```

위와 같이 작업시, 뒤로가기 기능이 가능해진다. 그러나 새로고침이나 다른페이지의 이동하여 뒤로가기 등 해결 못하는 많은 문제점이 존재한다. 위와 같은 소스를 수정하여 모두 처리할 수는 있겠지만 소스코드가 복잡해지고 지저분해진다.
그러니 위와같은 구조에서는 가능한
###  SPA 를 사용하자


- references:
- Written by: 이재봉 (jblee6110@gmail.com)
- reporting date: 2023-07-06
