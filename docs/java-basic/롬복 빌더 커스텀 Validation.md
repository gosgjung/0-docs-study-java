
[Add Custom Validations with Lombok Builders](https://trguduru.github.io/2019/06/add-custom-validations-with-lombok-builders/) 의 예제중 일부를 발췌해보면 아래와 같다.

```java
@Getter
@EqualsAndHashCode
@ToString
@Builder(builderClassName = "Builder",buildMethodName = "build")
public class Customer {
    private long id;
    private String name;

    static class Builder {
        Customer build() {
            if (id < 0) {
                throw new RuntimeException("Invaid id");
            }
            if (Objects.isNull(name)) {
                throw new RuntimeException("name is null");
            }
            return new Customer(id, name);
        }
    }
}
```

<br>

