### **@NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor - [공식문서](https://projectlombok.org/features/constructor)**

생성자를 만들 때 사용하고 Static 생성자를 만들고 싶으면 `staticName` 필드를 활용해주면 된다.

- `NoArgsConstructor` - 파라미터가 없는 기본 생성자를 만들어준다.
    - 생성자가 아예 없는 경우엔 컴파일러가 자동으로 생성해주어 `@NoArgsConstructor` 는 단독으로 쓰여도 무의미하다.

    ```java
    @NoArgsConstructor
    public class NoArgumentConstructorExample {
        
        private int number;
    }
    ```

    ```java
    public class NoArgumentConstructorExample {
        private int number;
    
        public NoArgumentConstructorExample() {
        }
    }
    ```


- `RequiredArgsConstructor` - 필수 변수값인 `@Nonnull` 과 초기화 되어 있지 않는 `final` 키워드를 가진 변수들에 대해서 생성자를 만들어준다.
    - Nonnull이면 안되는 변수들은 null일 경우 NullPointerException 예외를 발생시킨다.
    - 필수 변수값들이 아닌 경우
        1. final 변수가 아닌 경우
        2. 초기화가 된 final 변수
        3. Static 변수
        4. Nonnull이 아닌 변수들

    ```java
    @RequiredArgsConstructor(staticName = "of")
    public class RequiredArgsConstructorExample {
    
        @NonNull
        private String id;
    
        @NonNull
        private String password;
    
        private String email;
        private final Boolean agreeInformationUsing;
        private static int code;
        private final Boolean vipUser = Boolean.TRUE;
    }
    ```

    ```java
    public class RequiredArgsConstructorExample {
        @NonNull
        private String id;
        @NonNull
        private String password;
        private String email;
        private final Boolean agreeInformationUsing;
        private static int code;
        private final Boolean vipUser;
    
        public RequiredArgsConstructorExample(@NonNull final String id, @NonNull final String password, final Boolean agreeInformationUsing) {
            this.vipUser = Boolean.TRUE;
            if (id == null) {
                throw new NullPointerException("id is marked non-null but is null");
            } else if (password == null) {
                throw new NullPointerException("password is marked non-null but is null");
            } else {
                this.id = id;
                this.password = password;
                this.agreeInformationUsing = agreeInformationUsing;
            }
        }
    
    		public static RequiredArgsConstructorExample of(@NonNull final String id, @NonNull final String password, final Boolean agreeInformationUsing) {
            return new RequiredArgsConstructorExample(id, password, agreeInformationUsing);
        }
    }
    ```

- `AllArgsConstructor` - static 변수와 초기화된 final 변수를 제외한 모든 변수들이 포함된 생성자를 만들어준다.
