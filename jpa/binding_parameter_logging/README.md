## Hibernate 6에서 바인딩 파라미터 로깅 변경사항
spring boot application.properties로 설정 가능한 바인딩 파라미터 로깅 key가 아래와 같이 변경되었다
- As-IS
```properties
logging.level.org.hibernate.type.descriptor.sql=TRACE
```

- To-BE
```properties
logging.level.org.hibernate.orm.jdbc.bind=trace
```