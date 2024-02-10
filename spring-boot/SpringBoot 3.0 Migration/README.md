# SpringBoot 3.0 Migration 작업

## pom.xml의 버전 업그레이드
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.5</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

## javax 패키지명 jakarta 패키지명으로 변경
```java
import jakarta.persistence.*;
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpSession;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
```

> 주의 사항
>
> import javax. 문자열을 찾아서 import jakarta.로 변경하지 말 것
>
> javax.sql.DataSource, javax.crypto.SecretKey 같은 클래스는 JDK에 포함되어 있는 것들이며 이들은 바뀌지 않았기 때문이다

## querydsl 디펜던시 정리할것
- 기본적으로 SpringBoot 버전 업그레이드 하면 querydsl도 스프링부트의 버전관리 대상이기 때문에 5.0.0으로 자동 버전관리 되는거 같음
- 실제 실행시에 jakarta 패키지로 인식이 안되는 경우가 있어, classFier를 통해 명시적으로 선언해줌
```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>5.0.0</version>
    <classifier>jakarta</classifier>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>5.0.0</version>
    <classifier>jakarta</classifier>
</dependency>
```
### querydsl 버전 업그레이드 되면서 apt-maven-plugin 플러그인을 명시적으로 선언해주지 않아도 됨
- 만약 pom.xml에 apt-maven-plugin 설정이 있으면 아래와 같은 에러가 발생할 수 있음
```text
[ERROR] Attempt to recreate a file for type
```
querydsl-apt jakarta 버전에서는 javax.annotation.processing.Processor를 서비스로 제공하고 있고, 자바 컴파일러가 이를 사용하여 Q-class를 target/generated-sources/annotation에 생성해준다
따라서 명시적으로 pom.xml에 플러그인을 설정하면서 이중으로 동작하기 때문에 발생하는 현상이다

### 관련 참고자료
- https://post.dooray.io/we-dooray/tech-insight-ko/back-end/4173/

## logback의 설정파일(logback.xml)내 표현식 관련 에러 발생
- 우선을 디폴트 logback 설정을 사용하도록 해당 xml 표현식 부분을 제거하면서 해결