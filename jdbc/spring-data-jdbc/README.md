## spring-data-jdbc

JPA를 사용하는 것에 대한 부담을 느낄때 JPA의 컨셉을 활용하며 JDBC를 활용할 수 있도록 Spring에서 제공하는 라이브러리이다.
이는 JPA를 사용했을때의 아래와 같은 점에 대한 부담을 덜어줄 수 있다

1. JPA의 Lazy Loading이나 1차 캐시를 사용하지 않을 수 있다
2. 이로 인하여 엔티티를 사용할 경우, 1차 캐시나 더티 체킹 없이 바로 데이터를 저장할 수 있다

### Entity 생성
엔티티는 기존 Java의 VO와 동일하게 객체를 생성하되 Primary Key에 대하여 ID 애노테이션을 통하여 지정해준다
```java
public class Customers {
    @Id
    private long id;
    private String name;
    private int age;
    private String phoneNumber;
    private String email;
    private String address;
}
```

이후 Repository를 생성하는데 이는 spring-data-jpa와 생성방식이 매우 흡사하지만, extends 받는 인터페이스가 아래와 같이 다르다
- spring-data-jpa : extends JpaRepository
- spring-data-jdbc : extends CrudRepository

Generic 또한 spring-data-jpa와 동일하게 엔티티 객체와 Primary Key의 타입을 지정해주면 된다. 또한 별도의 @Repository 애노테이션 설정 없이 extends만으로도 Spring의 Application Context에 Bean으로 자동 등록되게 된다

### Repository 사용
spring-data-jpa와 마찬가지로 기본 CRUD 메서드를 제공하는 것 뿐만 아니라 메서드명을 통하여 쿼리 생성이 가능하다.
```java
public interface CustomerRepository extends CrudRepository<Customers, Long> {
    List<Customers> findByName(String name);

    List<Customers> findByAddress(String address);

    List<Customers> findByNameAndEmail(String name, String email);

    @Modifying
    @Query("UPDATE customers SET name = :name WHERE id = :id")
    boolean updateByName(@Param("id") Long id, @Param("name") String name);
}
```

위와 같이 엔티티의 필드명을 통하여 조건절 쿼리를 생성할수 있고 @Query 애노테이션을 통하여 실제 Native Query 설정도 가능하다.
수행되는 쿼리에 대한 로그를 확인하고 싶을 경우에는 아래와 같이 로깅레벨을 설정할 수 있다
```properties
logging.level.org.springframework.jdbc.core=TRACE
```