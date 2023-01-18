---
title: "Preflight 란?"
date: 2023-01-18T00:00:00+09:00
categories:
  - VUE
share: false
---

## Preflight 란

!! Preflight는 CORS가 정상적으로 통과 가능한지 api 송출전에 미리 통신 체크를 진행함.
실제 요청을 보내도 안전한지 판단하기 위해 preflight 요청을 먼저 보냄
Preflight Request는 actual 요청 전에 인증 헤더를 전송하여 서버의 허용 여부를 미리 체크하는 테스트 요청.
이 요청으로 트래픽이 증가할 수 있는데 서버의 헤더 설정으로 캐쉬가 가능.

## preflight 가 backend의 intercepter에 걸려 error 발생시 해결방안
preflight 요청은 OPTIONS 메서드를 사용하며 "Access-Control-Request-*" 형태의 헤더로 전송.
intercepter 부에 OPTIONS일 경우 통과시킴.
if (HttpMethod.OPTIONS.matches(request.getMethod())) {
            return true;
        }


## 마치며

해당 부분때매 통신 고생을 하지 않기를 바라며 이만 마침

---

- references:
  - [http://wiki.gurubee.net/display/SWDEV/CORS+%28Cross-Origin+Resource+Sharing%29)
  - [https://klyhyeon.tistory.com/270)
- Written by: 이재봉 (jblee6110@gmail.com)
- reporting date: 2023-01-18
