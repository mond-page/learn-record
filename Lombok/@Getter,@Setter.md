## @Getter, @Setter - [공식문서](https://projectlombok.org/features/GetterSetter)

Getter, Setter를 자동으로 만들어준다.

Access Level을 따로 명시하지 않는 한 public이다. (`AcessLevel - PUBLIC, MODULE, PROTECTED, PACKAGE, PRIVATE, NONE`)

- Getter와 Setter 기본 사용 예제

    ```java
    @Getter
    @Setter
    public class Student {
        private String name;
        private int age;
    }
    ```

    ```java
    // Decompile
    public class Student {
        private String name;
        private int age;
    
        public Student() {
        }
    
        public String getName() {
            return this.name;
        }
    
        public int getAge() {
            return this.age;
        }
    
        public void setName(final String name) {
            this.name = name;
        }
    
        public void setAge(final int age) {
            this.age = age;
        }
    }
    ```

- 특정 필드에 대해 접근제어자를 변경하고 싶을 때

    ```java
    @Setter
    @Getter
    public class Account {
    
        private String accountNumber;
    
        @Getter(AccessLevel.PRIVATE)
        private String password;
        
    }
    ```

    ```java
    public class Account {
        private String accountNumber;
        private String password;
    
        public Account() {
        }
    
        public void setAccountNumber(final String accountNumber) {
            this.accountNumber = accountNumber;
        }
    
        public void setPassword(final String password) {
            this.password = password;
        }
    
        public String getAccountNumber() {
            return this.accountNumber;
        }
    
        private String getPassword() {
            return this.password;
        }
    }
    ```

- 해당 필드를 Lazy 시키고 싶을 때 `lazy = true` 를 사용 ( Decompiler로 확인해보면 Double checked Locking 방식을 사용하고 있다 )

    ```java
    @Getter
    @Setter
    public class Plan {
    
        private String title;
        private LocalDate startDate;
        private LocalDate endDate;
    
        @Getter(lazy = true)
        private final String precautions = "이 여행상품은 환불이 불가능한 여행상품입니다.";
    }
    ```

    ```java
    // Decompiler
    public class Plan {
        private String title;
        private LocalDate startDate;
        private LocalDate endDate;
        private final AtomicReference<Object> precautions = new AtomicReference();
    
        public Plan() {
        }
    
        public String getTitle() {
            return this.title;
        }
    
        public LocalDate getStartDate() {
            return this.startDate;
        }
    
        public LocalDate getEndDate() {
            return this.endDate;
        }
    
        public void setTitle(final String title) {
            this.title = title;
        }
    
        public void setStartDate(final LocalDate startDate) {
            this.startDate = startDate;
        }
    
        public void setEndDate(final LocalDate endDate) {
            this.endDate = endDate;
        }
    
        public String getPrecautions() {
            Object value = this.precautions.get();
            if (value == null) {
                synchronized(this.precautions) {
                    value = this.precautions.get();
                    if (value == null) {
                        String actualValue = "이 여행상품은 환불이 불가능한 여행상품입니다.";
                        value = "이 여행상품은 환불이 불가능한 여행상품입니다." == null ? this.precautions : "이 여행상품은 환불이 불가능한 여행상품입니다.";
                        this.precautions.set(value);
                    }
                }
            }
    
            return (String)(value == this.precautions ? null : value);
        }
    }
    ```


- Setter의 onMethod를 사용하면 해당 Annotation이 붙은 메소드가 생성된다.

    ```java
    public class Marking {
    
        @Setter(onMethod_ = {@Marker})
        private String answer;
        
    }
    ```

    ```java
    public class Marking {
        private String answer;
    
        public Marking() {
        }
    
        @Marker
        public void setAnswer(final String answer) {
            this.answer = answer;
        }
    }
    ```

- Stter의 onParam을 사용하면 해당 Annotation이 파라미터에 붙은 메소드가 생성된다.

    ```java
    public class Marking {
    
        @Setter(onMethod_ = {@Marker}, onParam_ = {@ParameterMarker})
        private String answer;
    
    }
    ```

    ```java
    public class Marking {
        private String answer;
    
        public Marking() {
        }
    
        @Marker
        public void setAnswer(@ParameterMarker final String answer) {
            this.answer = answer;
        }
    }
    ```
