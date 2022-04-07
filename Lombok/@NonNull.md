## @NonNull - [공식문서](https://projectlombok.org/features/NonNull)

해당 애노테이션은 Recrod Component(Java 16), 메소드 파라미터, 생성자에서 사용가능하다

null check를 하여 nll일 경우 NullPointerException 발생시킨다.

- 메소드에서 사용하는 경우 메소드 최상단에 if 문을 넣어 null체크를 한다.

    ```java
    public class NonNullMethodTest {
    
        public void nonNullAnnotation(@NonNull String str) {
            System.out.println(str);
        }
    }
    ```

    ```java
    // Decompile
    public class NonNullMethodTest {
    
        public void nonNullAnnotation(@NonNull String str) {
            if (str == null) {
                throw new NullPointerException("str is marked non-null but is null");
            }
            System.out.println(str);
        }
    }
    ```


- 생성자에 사용하는 경우는 @Data, @Getter, @Setter 등 다른 애노테이션과 함께 쓰인다.

    ```java
    @Setter
    public class NonNullConstructorTest {
    
        @NonNull
        String str;
    
    }
    ```

    ```java
    // Decompile
    public class NonNullConstructorTest {
        @NonNull
        String str;
    
    		public NonNullConstructorTest() {
        }
    
        public void setStr(@NonNull final String str) {
            if (str == null) {
                throw new NullPointerException("str is marked non-null but is null");
            }
            this.str = str;
        }
    }
    ```

- Record(Java 16)에 사용하는 경우 기본 생성자가 없는 경우 기본생성자에서 체크를 한다.

    ```java
    public record NonNullRecordTest(@NonNull String str) {}
    ```

    ```java
    // Decompile
    public record NonNullRecordTest(@NonNull String str) {
        public NonNullRecordTest(@NonNull String str) {
            if (str == null) {
                throw new NullPointerException("str is marked non-null but is null");
            } else {
                this.str = str;
            }
        }
    
        @NonNull
        public String str() {
            return this.str;
        }
    }
    ```