## @DynamicInsert, @DynamicUpdate 애노테이션
JPA 활용시 기본전략은 모든 엔티티의 필드에 대하여 전체 INSERT, 및 전체 UPDATE가 수행되게 된다. Hibernate에서 지원하는 DynamicInsert와 DynamicUpdate 애노테이션이 어떻게 동작하는 지에 대하여 테스트를 진행해보았다.

먼저 DynamicInsert 애노테이션에 대한 테스트를 수행하였다.

### @DynamicInsert
기본적으로 엔티티에 설정된 값의 존재 여부와 관계없이 JPA에서 Insert 쿼리에 대한 Flush시에는 모든 필드에 대한 INSERT 쿼리가 수행되는 것을 확인할 수 있다.
예를 들어 아래 엔티티에서 특정 필드에 대한 값은 세팅하지 않고, DynamicInsert 애노테이션이 존재하는 경우와 하지 않는 경우에 대하여 테스트를 수행해 보았다.
```java
@Entity
@Data
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String gender;
    private String phoneNumber;
    private String location;

    @Builder
    public Employee(String name, String gender, String phoneNumber, String location) {
        this.name = name;
        this.gender = gender;
        this.phoneNumber = phoneNumber;
        this.location = location;
    }

    public void update(EmployeeRequestDto requestDto) {
        this.name = requestDto.getName();
        this.gender = requestDto.getGender();
        this.phoneNumber = requestDto.getPhoneNumber();
        this.location = requestDto.getLocation();
    }
}
```

먼저 @DynamicInsert 애노테이션 없이 location 필드를 제외한 나머지 필드의 값만 설정한 채 INSERT를 수행해보았다.
#### @DynamicInsert 애노테이션 없이 수행하였을 경우
- RequestBody : location 필드에 대한 값은 설정하지 않음
```json
{
    "name": "Test01",
    "gender": "MAIL",
    "phoneNumber": "010-0000-0000"
}
```

#### INSERT 수행 결과 Hibernate에서 수행되는 쿼리 결과
```sql
insert 
into
    employee
    (gender, location, name, phone_number, id) 
values
    (?, ?, ?, ?, default)
```
위와 같이 설정하지 않은 필드(location)까지 모든 필드에 대하여 INSERT 쿼리가 수행되는 것을 확인할 수 있다.
다음은 @DynamicInsert를 설정하고 동일한 테스트를 수행했을때 쿼리 결과이다
```sql
insert 
into
    employee
    (gender, name, phone_number, id) 
values
    (?, ?, ?, default)

```
위 결과에서 알 수 있듯이 설정하지 않은 필드(location)을 제외한 INSERT문에 수행되는 것을 확인할 수 있다.

다음은 DynamicUpdate 애노테이션에 대한 테스트를 수행해 보았다.
### @DynamicUpdate
먼저 findById를 통하여 1차 캐시에 적재된 엔티티 데이터에 대하여 특정 필드에 대하여 변경한 업데이트 API를 호출하고 1차 캐시의 엔티티에 대하여 모두 요청 Body 필드값을 설정한 후에 애노테이션 존재 여부에 따른 차이를 확인해보았다.
즉 엔티티내 총 4개 필드 중 2개 필드에 대한 변경사항에 대해서 업데이트 API를 호출하되 1차 캐시에 있는 모든 필드에 대하여 Setter 메서드를 통하여 필드 세팅을 수행하였을때의 결과이다.
- 최초 INSERT 수행한 엔티티에 대한 RequestBody
```json
{
    "name": "Test01",
    "gender": "MAIL",
    "phoneNumber": "010-0000-0000",
    "location": "seoul"
}
```

- 업데이트를 수행한 RequestBody
```json
{
    "name": "Test02",
    "gender": "FEMALE",
    "phoneNumber": "busan"
}
```

- 1차 캐시 엔티티에 대하여 모든 필드에 대하여 Setter 메서드를 통한 데이터 변경 수행
```java
public EmployeeResponseDto update(Long id, EmployeeRequestDto requestDto) {
    Employee employee = employeeRepository.findById(id).orElseThrow(() -> new RuntimeException("ERROR"));
    employee.setName(requestDto.getName());
    employee.setGender(requestDto.getGender());
    employee.setPhoneNumber(requestDto.getPhoneNumber());
    employee.setLocation(requestDto.getLocation());
}
```

@DynamicUpdate 애노테이션을 설정하지 않고 업데이트를 수행한 후 JPA를 통하여 수행된 쿼리 결과이다.
- @DynamicUpdate 애노테이션을 설정하지 않고 JPA를 통하여 수행한 쿼리 결과
```sql
update
        employee 
    set
        gender=?,
        location=?,
        name=?,
        phone_number=? 
    where
        id=?
```
위와 같이 업데이트 RequestBody에 포함되지 않은 필드이지만 1차 캐시 데이터에 대한 모든 필드의 Setter 설정을 수행하였기 때문에 전체 필드에 대한 업데이트 문이 수행되는 것을 확인할 수 있다. 다음은 @DynamicUpdate 애노테이션을 설정하고 동일한 테스트를 수행한 결과이다.
- @DynamicUpdat 애노테이션을 설정하고 JPA를 통하여 수행한 쿼리 결과
```sql
    update
        employee 
    set
        gender=?,
        location=?,
        name=?,
        phone_number=? 
    where
        id=?
```
앞서 @DynamicUpdate 애노테이션을 설정하지 않은 쿼리와 동일하게 모든 필드에 대한 업데이트 쿼리가 수행되는 것을 확인할 수 있다. 이 결과로 보았을때 1차 캐시 데이터에 대한 Setter 메서드를 통한 데이터 변경 여부에 따라 모든 필드에 대한 업데이트가 수행되는 것을 확인할 수 있다.

다음은 실제 Setter를 통하여 1차 캐시 데이터에 대한 데이터 세팅을 변경되지 않은 필드에 대해서는 수행하지 않았을 경우에 대한 @DynamicUpdate 애노테이션을 설정하였을 경우와 하지 않았을 경우에 대한 차이를 테스트해 보았다.
- location 필드에 대한 세팅은 제외
```java
public EmployeeResponseDto update(Long id, EmployeeRequestDto requestDto) {
        Employee employee = employeeRepository.findById(id).orElseThrow(() -> new RuntimeException("EX001"));
        employee.setName(requestDto.getName());
        employee.setGender(requestDto.getGender());
        employee.setPhoneNumber(requestDto.getPhoneNumber());
//        employee.setLocation(requestDto.getLocation());
}
```

- @DynamicUpdate 애노테이션을 설정하지 않고 location 필드 값 세팅은 제외한 업데이트를 수행한 결과
```sql
update
        employee 
    set
        gender=?,
        location=?,
        name=?,
        phone_number=? 
    where
        id=?
```

- @DynamicUpdate 애노테이션을 설정하고 location 필드 값 세팅은 제외한 업데이트를 수행한 결과
```sql
update
        employee 
    set
        gender=?,
        name=?,
        phone_number=? 
    where
        id=?
```

@DynamicUpdate를 설정한 경우와 하지 않은 경우에 대한 차이를 확인할 수 있다. 애노테이션을 설정하지 않게 되면 1차캐시에 대한 location 필드 값 변경을 하지 않아도 location필드에 대한 업데이트 쿼리가 수행되지만, @DynamicUpdate 애노테이션을 설정하게 되면 해당 필드에 대해서는 업데이트 쿼리에서 제외하고 업데이트 쿼리가 수행되는 것을 확인할 수 있다.

결론적으로 1차 캐시 데이터를 변경하였을 경우에는 @DynamicUpdate 애노테이션의 존재 여부와 관계없이 업데이트 시에 변경하지 않는 데이터 필드에 대한 업데이트 쿼리가 실행되지만 1차 캐시 데이터를 변경하지 않은 경우에 대해서는 @DynamicUpdate 애노테이션의 존재 유무에 따라 변경하지 않은 필드에 대한 업데이트 쿼리에 포함 여부 애노테이션 존재 여부를 통하여 결정된다는 것을 확인할 수 있다.