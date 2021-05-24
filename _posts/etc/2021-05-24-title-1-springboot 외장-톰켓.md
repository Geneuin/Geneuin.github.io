---
title: "SPRINGBOOT 외장 톰캣 빌드와, OPENJDK11"
date: 2021-04-30T00:13:00+09:00
categories: 
    - TOMCAT
share: false
---


# springboot에서 외장 톰캣빌드?

안녕하세요. springboot을 통해 프로젝트 개발 완료 후, 서버에 올리는 데 문제가 생겼습니다. tomcat에서 springboot을 통해 만든 war 파일을 빌드 못 해주는 문제가 생겼습니다. 무엇일까?? 우선 springboot 프로젝트는 기본적으로 packaging이 jar로 설정되어있습니다.  하여 먼저 war를 추출하기 위해,
pom.xml 아래에 다음과 같이 추가해줍니다. packaging을 변경해 줍니다.


```xml
	<packaging>war</packaging>

    <!-- marked the embedded servlet container as provided --> 
    <dependency> 
        <groupId>org.springframework.boot</groupId> 
        <artifactId>spring-boot-starter-tomcat</artifactId> 
        <scope>provided</scope> 
    </dependency>
```
다음으로 main class 수정이 필요합니다. Springboot은 기본적으로 내장된 jar 파일을 이용하기 때문에, war 파일로 배포하기 위해서는 SpringBootServletInitialize 상속받을 필요하였습니다. 하여 기존 메인 소스를 다음과 같이 변경합니다.


```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(Application.class);
	}
}
```


*** SpringBootServletInitializer 사용 이유 ***
Spring 웹 애플리케이션을 외부 Tomcat에서 동작하도록 하기 위해서는 web.xml (Deployment Descriptor, DD)에 애플리케이션 컨텍스트를 등록해야만 한다. 이는, Apache Tomcat(Servlet Container)이 구동될 때 /WEB-INF 디렉토리에 존재하는 web.xml을 읽어 웹 애플리케이션을 구성하기 때문이다. 
하지만 Servlet 3.0 스펙으로 업데이트되면서 web.xml이 없어도 동작이 가능해졌다. 이는, web.xml 설정을 WebApplicationInitializer인터페이스를 구현하여 대신할 수 있게 됐고, 프로그래밍적으로 ServletContext에 Spring IoC 컨테이너(AnnotationConfigWebApplicationContext)를 생성하여 추가할 수 있도록 변경됐기 때문이다.
이와 비슷한 맥락에서, web.xml이 없는 SpringBoot 웹 애플리케이션을 외부 Tomcat에서 동작하도록 하기 위해서는 WebApplicationInitializer 인터페이스를 구현한 SpringBootServletInitializer를 상속을 받는 것이 필요했던 것이다.

#참조: https://medium.com/@SlackBeck/spring-boot-%EC%9B%B9-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%9D%84-war%EB%A1%9C-%EB%B0%B0%ED%8F%AC%ED%95%A0-%EB%95%8C-%EC%99%9C-springbootservletinitializer%EB%A5%BC-%EC%83%81%EC%86%8D%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94%EA%B1%B8%EA%B9%8C-a07b6fdfbbde 



이렇게까지 한 후 war 파일을 추출하고, tomcat의 webapps에 넣으면 서버가 실행됩니다.


그러나... 저희가 이번 프로젝트에 사용했던 openJDK11은 tomcat에서 바로 빌드되지 않았습니다. ㅜㅜ 이곳저곳 찾아보고 검색을 해봐도 크게 해답을 찾을 수 없었습니다. 하여 이것저것 시도 끝에 빌드를 시켰습니다. 방법은 생각보다 간단했습니다. tomcat JAVA_HOME 위치를 openjdk 11로 변경해주는 것으로 해결하였습니다.
tomcat -> bin -> catalina.sh 의 맨 위에 'JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' 라인을 추가하였습니다.

```log

24-May-2021 22:33:21.063 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/9.0.31
24-May-2021 22:33:21.072 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Feb 5 2020 19:32:12 UTC
24-May-2021 22:33:21.072 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 9.0.31.0
24-May-2021 22:33:21.073 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
24-May-2021 22:33:21.073 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            5.4.0-1048-aws
24-May-2021 22:33:21.074 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
24-May-2021 22:33:21.074 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/lib/jvm/java-11-openjdk-amd64
24-May-2021 22:33:21.079 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           11.0.11+9-Ubuntu-0ubuntu2.18.04

```
log를 통해 jvm 버전이 변경된 것을 확인하였고, 무사히 배포 완료하였습니다. 간단한 부분인데 많이 돌아온 것 같네요. 이상으로 포스팅을 마칩니다. 감사합니다.
