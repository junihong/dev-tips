# Cors 이해
## CORS(Cross-Origin Resource Sharing, 교차 출처 리소스 공유)
- [ ] 실행 중인 웹 애플리케이션이 다른 Origin의 선택한 자원에 접근할 수 있는 권한을 HTTP 헤더를 통하여 부여하도록 브라우져에 알려주는 방법
- [ ] 웹 애플리케이션 리소스가 자신의 출처와 다를 때 브라우저는 요청 헤더에 Origin 필드에 요청 출처를 함께 담아 교차 출처 HTTP 요청을 실행한다
- [ ] 출처를 비교하는 로직은 브라우져 스펙 기준으로 처리되며, 브라우져는 클라이언트의 요청 헤더와 서버의 응답헤더를 비교하여 응답을 결정
- [ ] 두개의 출처를 비교하는 방법은 URL 의 구성요소 중 Protocol, Host, Port가 동일한지 확인한다

## Cors 요청의 종류
### Simple Request
- [ ] Simple Request는 Preflight 과정 없이 바로 서버에 본 요청을 보낸 후, 서버가 응답의 헤더에 Access-Control-Allow-Origin과 같은 값을 전송하면 브라우져가 CORS 정책 위반여부를 판단하는 방식
#### 제약사항
- [ ] GET, POST, HEAD 중에 한가지 Http Method를 사용해야 한다
- [ ] 헤더의 경우 Accept, Accept-Language, Content-Language, Content-Type, DRP, Downlink, Save-Data, Viewport-Width Width만 가능하고 이외의 Custom Header는 허용되지 않는다
- [ ] Content-Type은 application_x-www-form-urlencoded, multipart_form-data, text/plain만 가능하다
### Preflight Request
- [ ] 브라우져는 요청을 한번에 보내지 않고, Preflight 요청과 본 요청으로 나누어 서버에 전달하는데 이때 브라우져가 보내는 예비요청을 Preflight라고 하며, 이 예비 요청의 메소드에는 OPTIONS가 사용된다
- [ ] 예비요청의 역할은 본 요청을 보내기 전에 브라우져 스스로 안전한 요청인지 확인하는 것으로 요청 사양이 Simple Request에 해당하지 않을 경우 브라우져가 Preflight Request를 실행한다

## Cors 해결 방법 - 서버에서 Access-Control-Allow-* 헤더 세팅
- [ ] Access-Control-Allow-Origin : 헤더에 작성된 출처만 브라우져가 리소스를 접근할 수 있도록 허용
- [ ] Access-Control-Allow-Method : Preflight Request에 대한 응답으로 실제 요청중에 사용할 수 있는 메서를 나타낸다
- [ ] Access-Control-Allow-Headers : Preflight Request에 대한 응답으로 실제 요청중에 사용할 수 있는 헤더 필드 이름을 나타낸다
- [ ] Access-Control-Allow-Credential : 실제 요청에 쿠키나 인증 등의 인증정보가 포함될 수 있음을 나타낸다
- [ ] Access-Control-Max-Age : Preflight 요청 결과를 캐시할 수 있는 시간을 나타내는 것으로 해당 시간동안은 Preflight 요청을 다시 하지 않게 된다

### Spring-Security 사용
```java
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests().anyRequest().authenticated();
        http.cors().configurationSource(corsConfigurationSource());

        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.addAllowedOrigin("*");
        configuration.addAllowedHeader("*");
        configuration.addAllowedMethod("*");
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);

        return source;
    }
}

```

#### CorsConfigurer
- [ ] Spring Security 필터 체인에 CorsFilter를 추가한다
- [ ] corsFilter라는 이름의 Bean이 제공되면 해당 CorsFilter가 사용횐다
- [ ] corsFilter라는 이름의 Bean이 없고 CorsConfigurationSource빈이 정의된 경우 해당 CorsConfiguration이 사용된다
- [ ] CorsConfigurationSource빈이 정의되어 있지 않은 경우 Spring MVC가 클래스 경로에 있으면 HandlerMappingIntrospector가 사용된다

#### CorsFilter
- [ ] Cors Preflight 요청을 처리하고 Cors 요청을 가로채고, 제공된 CorsConfigurationSource를 통해 일치된 정책에 따라 Cors 응답 헤더와 같은 응답을 업데이트 하기 위한 필터
- [ ] Spring MVC Java 구성과 Spring MVC XML 네임스페이스에서 Cors를 구성하는 대