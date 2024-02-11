## Hibernate Validor annotation 종류
- NotBlank
- NotNull
- Size
- Email
- Pattern
- Min
- Max

### Hibernate-Validator를 활용하여 Dto의 필드에 대한 필드 검증 애노테이션 적용 예
```java
public class EmployeeRequestDto implements DefaultDto<Employee> {

    @NotBlank(message = "Name is not blank")
    @NotNull(message = "Name is not null")
    @Size(min = 1, max = 100, message = "Name must be of 1 to 100 characters")
    private String name;

    @NotBlank(message = "Email is not blank")
    private String email;

    @NotBlank(message = "PhoneNumber is mandatory")
    @Pattern(regexp = "^\\\\d{10}$", message = "Invalid phone number")
    private String phoneNumber;

    @Min(value = 1, message = "Age must be over 1")
    @Max(value = 100, message = "Age must smaller than 100")
    private Integer age;
    private String gender;
    private String grade;
    private String address;
}
```

### @ControllerAdvice를 활용한 ExceptionHandler 적용 예
```java
@RestControllerAdvice
public class MyExceptionHandler {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Map<String, String> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });

        return errors;
    }
}
```