---
title: "TOMCAT -1- Catalina.out"
date: 2021-10-01T11:26:00+09:00
categories: 
    - TOMCAT
share: false
---

# 페이지 목차
1. [TOMCAT Catalina.out 의 문제]({% post_url /tomcat/2021-10-01-tomcat-1-catalina.out 문제 %})

## Tomcat catalina.out의 사이즈 문제

---
- 프로잭트 운영중, 알수 없는 문제로 인해, backend가 작동하지 않는 경우가 발생하였다. 문제를 찾아보니, 
catalina.out이 가득 차버려서 로그를 쌓지 못해, 서버가 작동하지 않는 것 이였다.

그래서 해결방안으로 2가지를 생각하고 찾아봐서 변경하였다. 2가지 방안에 대해 제안한다.

1. catalina.out을 쌓지 않기.

우선 tomcat>bin>catalina.sh를 열어서 다음과 같은 LOGGING_MANAGER 를 찾는다.

```sh
 touch "$CATALINA_OUT"
  if [ "$1" = "-security" ] ; then
    if [ $have_tty -eq 1 ]; then
      echo "Using Security Manager"
    fi
    shift
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
      -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
      -classpath "\"$CLASSPATH\"" \
      -Djava.security.manager \
      -Djava.security.policy=="\"$CATALINA_BASE/conf/catalina.policy\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"

  else
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
      -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
      -classpath "\"$CLASSPATH\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"

  fi
```
 catalina.out을 사용하지 않기 위해서, 
touch "$CATALINA_OUT"  -->  #touch "$CATALINA_OUT"
"$CATALINA_OUT" 2>&1 "&"  -->  >> /dev/null 2>&1 &
로 변경 시켜주면 된다.


2. catalina.out을 날짜별로 쌓기


tomcat>bin>catalina.sh를 열어서 다음과 같은 LOGGING_MANAGER 를 찾는다.

```sh
 touch "$CATALINA_OUT"
  if [ "$1" = "-security" ] ; then
    if [ $have_tty -eq 1 ]; then
      echo "Using Security Manager"
    fi
    shift
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
      -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
      -classpath "\"$CLASSPATH\"" \
      -Djava.security.manager \
      -Djava.security.policy=="\"$CATALINA_BASE/conf/catalina.policy\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"

  else
    eval $_NOHUP "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER "$JAVA_OPTS" "$CATALINA_OPTS" \
      -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
      -classpath "\"$CLASSPATH\"" \
      -Dcatalina.base="\"$CATALINA_BASE\"" \
      -Dcatalina.home="\"$CATALINA_HOME\"" \
      -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
      org.apache.catalina.startup.Bootstrap "$@" start \
      >> "$CATALINA_OUT" 2>&1 "&"

  fi
```
 catalina.out을 날짜별로 쌓기 위해서, 
touch "$CATALINA_OUT"  -->  #touch "$CATALINA_OUT"
"$CATALINA_OUT" 2>&1 "&"  -->  "$CATALINA_OUT".$(date '+%Y-%m-%d') 2>&1
로 변경 시켜주면 된다.


- references:
- Written by: 이재봉 (jblee6110@gmail.com)
- reporting date: 2021-10-01