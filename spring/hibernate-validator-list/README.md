## @RequestBody가 다건의 리스트일 경우에 대한 Validator
우선 RequestBody를 통하여 전달받은 객체를 생성한다

```java
public class UserDto {

    @NotEmpty(message = "User name cannot be empty")
    private String name;
    private int age;
    private String gender;
    private String phoneNumber;
    private String address;
}
```

생성된 객체에 위와 같이 jakarta.validation.constraints 에서 지원하는 @NotEmpty 애노테이션을 활용하여 name 필드에 대한 Validation을 설정한다
이후 Controller에서 @Valid 애노테이션을 추가한 후에 @RequestBody에 대한 @NotEmpty Valid및 List 객체 내 UserDto에 대한 @Valid 애노테이션을 추가해준다

```java
@Validated
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping("/addAll")
    public ResponseEntity<?> addAll(@RequestBody @NotEmpty(message = "Input user cannot be empty.")List<@Valid UserDto> userDtoList) {
        List<User> resultList = userService.createAll(userDtoList);
        return new ResponseEntity<>(resultList, HttpStatus.OK);
    }
}
```
위와 같이 설정할 경우 @RequestBody에 대한 @NotEmpty Valid를 통하여 리스트 객체에 대한 Empty 체크 Validation이 가능하고, List내 UserDto에 대하여 @valid를 통한 name 필드 Validation이 가능하다. 하지만 테스트 결과 UserDto 앞에 @Valid 애노테이션을 제외해도 UserDto 객체 내의 name 필드에 대한 Validation은 동일하게 가능한 것으로 보인다. 아마도 컨트롤러 자체에 설정된 @Validated 애노테이션을 통하여 가능한 것이 아닐까 예상해본다.

## ExceptionHandler를 통한 Exception handling
만일 Validation에 실패하여 Exception이 발생하게 되면 ConstraintViolationException이 발생하여 Thrown되게 된다. ExceptionHandler를 통하여 이 Exception에 대한 공통 에러 처리를 수행할 수 있게 된다.
```java
@RestControllerAdvice
public class ConstraintViolationExceptionHandler {

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<String> handle(ConstraintViolationException constraintViolationException) {
        Set<ConstraintViolation<?>> constraintViolations = constraintViolationException.getConstraintViolations();
        StringBuilder sb = new StringBuilder();
        if (!constraintViolations.isEmpty()) {
            for (ConstraintViolation<?> violation : constraintViolations) {
                sb.append(violation.getMessage()).append("\n");
            }
        }

        return new ResponseEntity<>(sb.toString(), HttpStatus.BAD_REQUEST);
    }
}
```

### 참고자료
[https://www.baeldung.com/spring-validate-list-controller](https://www.baeldung.com/spring-validate-list-controller)