---
title: "chap03.lambda"
date: 2018-12-15 17:04:30
categories: java8
tag: lambda
---

## summary
- lamdba expression은 익명 함수의 일종이다.
- functional interface는 하나의 abstract method만 정의하는 interface
- functional interface를 기대하는 곳에서 lambda expression 사용 가능
- lambda expression 전체가 functional interface의 instance로 취급됨
- 자주 사용하는 functional interface
    - Predicate<T>, Function<T, R>, Supplier<T>, Consumer<T>, BinaryOperator<T> 등
- 박싱 동작을 피할 수 있는 기본형 특화 interface
- execute around pattern + lambda = good
- lambda expression의 기대형식을 target type이라고 함
- Comparator, Predicate, Function과 같은 functional interface는 lambda expression을 조합할 수 있는 다양한 default method를 제공

---

1. lambda란?
    - 특징
        - anonymons
        - function : class에 종속된 method와 다름. but, parameter list, body, return type, exception 포함
        - 전달 : method 인수로 전달하거나 변수로 저장할 수 있음
        - 간결성

    - 미적분학 학계에서 유래
    - java8 이전에 없던 것이 아님. behavior parameterization을 간결하게 구현할 수 있음

    - lambda expression 구조
        - parameter list
        - ->
        - lambda body
        
        - ex) 
            ```java
            (Apple a1, Apple a2) -> a1.getWeight().compare(a2.getWeight())
            - parameter list -
                            - arrow -
                                                - body -
            ```
    - 유효한 lambda expression
        ```java
        (String s) -> s.length()    // return 이 함축되어 있음
        (Apple a) -> a.getWeight() > 150    // boolean 반환
        (int x, int y) -> {
            System.out.println("Result:");
            System.out.println(x+y);
        }   // 여러 행의 문장 포함 가능
        () -> 42    // paramter가 없으면 int 반환
        () -> {}    // parameter가 없으며 void 반환
        () -> "Seoul"   // parameter 없으며 String 반환
        () -> {return "Korea";} // 명시적 return
        (List<String> list) -> list.isEmpty()   // return boolean
        () -> new Apple(10)                     // 객체 생성
        (Apple a) -> {
            System.out.println(a.getWeight());
        }                                       // 객체에서 소비
        (String s) -> s.length()                // 객체에서 선택/추출
        (int a, int b) -> a * b                 // 두 값을 조합
        (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())    // 두 객체 비교
        ```

2. where, how
    - functional interface라는 문맥에서 사용
    1) functional interface?
        - 하나의 abstract method를 지정하는 interface
        - default method 많아도, abstract method만 하나면 됨.
        - lambda expression으로 functional interface의 추상 메서드 구현을 직접 전달할 수 있음. 즉, 전체 표현식을 functional interface의 instance로 취급. (기술적으로 따지면 functional interface를 concrete 구현한 클래스의 instance)
        - ex
            ```java
            public static void process(Runnable r){
                r.run();
            }
            process(() -> System.out.println("hello world, lambda expression"));

            Runnable r = new Runnable() {
                public void run() {
                    System.out.println("hello world, extends abstract interface")
                }
            }
            process(r);
            ```

    2) function descriptor
        - abstract method signature는 lambda expression의 signature를 가리킴
        - labmda expression의 signature를 서술하는 method를 function descriptor라고 부름
        
3. execute around pattern
    - 자원처리(자원 열고 - 처리하고 - 자원 닫는)는 설정과 정리과정이 비슷
    - 처리 코드를 설정/정리 코드가 둘러 싸고 있음
    ```java
    public static String processFIle() throws IOException {
        try (BufferedReader br 
                = new BufferedReader(new FileReader("data.txt"))){
            return br.readLine();
        }
    }
    ```
    1. behavior parameterization 적용
        - processFile의 동작을 parameterization
        - processFile method가 BufferedReader를 이용해서 다른 동작을 수행할 수 있도록 processFile method로 동작을 전달
        - ex) 한 번에 두 행을 읽으려면
            ```java
            String result = processFile(
                (BufferedReader br) -> br.readLine() + br.readLine()
            );
            ```
    2. functional interface 이용
        ```java
        @FunctionalInterface // 실제로 functional interface가 아니면 컴파일 에러 발생
        public interface BufferedReaderProcessor {
            String process(BufferedReader b) throws IOException;
        }

        // processFIle parameter를 변경
        public static String processFile(BufferedReaderProcessor p) throws IOException {
            ...
        }
        ```

    3. 동작 실행
        - processFile 코드 변경
        ```java
        public static String processFile(BufferedReaderProcessor p) throws IOException {
            try (BufferedReader br = 
                    new BufferedReader(new FileReader("data.txt"))) {
                return p.process(br);
            }
        }
        ```

    4. lambda
        ```java
        String oneLine = processFile((BufferedReader br) -> br.readLine());
        String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
        ```

4. functional interface 사용
    - 다양한 lambda expression을 사용하려면 공통의 function descriptor를 기술하는 functional interface 집합이 필요하다.
    - java API에 Comparable, Runnable, Callable 등 다양한 functional interface 포함
    - 기본형 특화
        - Generic Parameter에는 참조형만 사용 가능(Consumer<T>의 T)
        - 기본형 -> 참조형 변환 : 박싱
        - 참조형 -> 기본형 변환 : 언박싱
        - 자동변환 오토박싱
        - 이런 과정은 비용 소모
        - autoboxing을 피하기 위해 특별한 버전의 functioncal interface 제공
        - ex)
            - IntPredicate는 1000을 boxing하지 않음
            - Predicate<Integer>는 1000을 Integer 객체로 boxing 함
    - 대표적인 functional interface list
        | functional interface | function descriptor | Primitive interface |
        | --- | --- | --- |
        | Predicate<T>        | T -> boolean        | IntPredicate, LongPrdicate, ... |
        | Consumer<T>         | T -> void           | IntConsumer, LongConsumer, ... |
        | Function<T, R>      | T -> R              | IntFunction<R>, IntToDoubleFunction, ToIntFunction<T>, ... |
        | Supplier<T>         | () -> T             | BooleanSupplier, IntSupplier, ... |
        | UnaryOperator<T>    | T -> T              | IntUnaryOperator, ... |
        | BinaryOperator<T>   | (T, T) -> T         | IntBinaryOperator, ... |
        | BiPredicate<L, R>   | (L, R) -> boolean   | 
        | BiConsumer<T, U>    | (T, U) -> void      | ObjIntConsumer<T>, ... |
        | BiFunction<T, U, R> | (T, U) -> R         | ToIntBiFunction<T, U>, ... |
    
    - exception, lambda, functional interface
        - functional interface는 확인된 예외를 던지는 동작을 허용하지 않는다? 무슨 뜻이야, 말이 어려워
        - 확인된 예외 = checked Exception
        - 예외를 던지는 lambda expression을 만들려면
            1. checked exception을 선언하는 functional interface를 정의하거나
            2. lambda를 try/catch block으로 감싸야 한다.
        - ex)
            ```java
            @FunctionalInterface
            public interface BufferedReaderProcessor {
                String process(BufferedReader b) throws IOException;
            }
            BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
            ```
        - but, Function<T, R> 형식의 functional interface를 기대하는 API를 사용하고 있으므로, 직접 functional interface를 만들기는 상황. 따라서, try/catch 사용
        - ex)
            ```java
            Function<BufferedReader, String> f =
                (BufferedReader b) -> {
                    try {
                        return b.readLine();
                    } catch(IOException e){
                        throw new RuntimeException(e);
                    }
                };
            ```

5. 형식 검사, 형식 추론, 제약
    - lambda expression 자체에는 어떤 functional interface를 구현하는지 정보가 포함되어 있지 않다.
    -> lambda의 실제 형식을 파악해야 함.
    1) 형식 검사
        - lambda가 사용되는 context를 파악
        - context에서 기대되는 lambda expression의 형식을 target type 이라고 한다.
        - ex
            ```java
            List<Apple> havierThan150g =
                filter(inventory, (Apple a) -> a.getWeight() > 150);
            // 1. filter method 선언을 확인 = Predicate<Apple>
            // 2. Predicate<Apple>은 test라는 abstract method 하나를 정의하는 functional interface
            // 3. test의 function descriptor = boolean test(Apple a)
            // 4. lambda의 signature와 일치
            ```
    2) 같은 lambda, 다른 functional interface
        - 같은 lambda expression이라도 호환되는 abstract method를 가진 다른 functional interface로 사용될 수 있다.
        - void 호환 규칙
            - lambda body에 일반 표현식이 있으면, void를 반환하는 function descriptor와 호환된다.
            - ex)
                ```java
                Consumer<String> b = s -> list.add(s);
                // Consumer context는 T -> void
                // list.add(s)는 boolean을 반환하지만 유효함.
                ```
    3) 형식 추론
        - target type을 이용해서 function descriptor를 알 수 있으므로 lambda의 signature도 추론할 수 있다.
        -> 생략 가능
            ```java
            List<Apple> greenApples = 
                filter(inventory, a -> "green".equals(a.getColor()));
            // parameter a의 형식을 명시적으로 지정하지 않음.
            ```
    4) local variable 사용
        - lambda expression에서는 익명 함수가 하는 것처럼 free variable(parameter로 넘겨진 variable이 아닌 외부에서 정의된 variable)을 활용할 수 있다 = capturing lambda라고 함.
            ```java
            int portNumber = 1337;
            Runnable r = () -> System.out.println(portNumber);
            ```
        - 이를 위해
            1. local variable은 final로 선언되어 있거나
            2. 실질적으로 final로 선언된 variable과 똑같이 사용되어야 함.
        
        - 지역 클래스에서 외부클래스의 인스턴스멤버와 static메법에 모두 접근 가능하지만, 지역변수는 final이 붙은 상수만 접근할 수 있는 이유는?
            - method가 수행을 마치고 지역변수가 소멸된 시점에도, 지역클래스의 인스턴스가 소멸된 지역변수를 참조하려는 경우가 발생할 수 있음
            - final로 지정된 지역변수는 JVM constant pool에서 따로 관리됨.
            - String도 constant pool에서 관리됨.
        
        - 지역 변수의 제약
            - 인스턴스변수는 힙에 저장, 지역변수는 스택에 위치.
            - 람다에서 지역 변수에 바로 접근할 수 있다는 가정 하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. -> 여기까지는 이해됨
            - 따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다. -> "final로 선언된 variable과 똑같이 사용되어야 한다" 와 관련된 내용인듯
                ```java
                int port = 1337;
                Runnable r = () -> System.out.println(port);
                //  port = 31337;   컴파일 에러
                ```

6. method reference
    ```java
    inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
    inventory.sort(comparing(Apple:getWeight));
    ```
    - 특정 method만을 호출하는 lambda의 축약형
    - ex)
        |lambda                                 | method reference |
        |---|---|
        |(Apple a) -> a.getWeight()                 |Apple::getWeight|
        |() -> Thread.currentThread().dumpStack()   |Thread.currentThread()::dumpStack|
        |(str, i) -> str.substring(i)               |String::substring|
        |(String s) -> System.out.println(s)        |System.out::println|

    - 유형
        1. **정적** 메서드 레퍼런스 : Integer의 parseInt = Integer::parseInt
        2. 다양한 형식의 **인스턴스** 메서드 레퍼런스 : (String s) -> s.length() = String::length
        3. **기존 객체의 인스턴스** 메서드 레퍼런스 : Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고, Transaction 객체에는 getValue 메서드가 있다면 () -> expensiveTransaction.getValue() = expensiveTransaction::getValue

        - 생성자, 배열 생성자, super
        - List의 sort method는 Comparator를 기대하고, Comparator는 (T, T) -> int 함수 디스크립터를 가짐.
            ```java
            List<String> str = Arrays.asList("a", "b", "A", "B");
            str.sort((s1, s2) -> s1.compareToIgnoreCase(s2));
            str.sort(String::compareToIgnoreCase);
            ```
    - 생성자 reference
        - ClassName::new
            ```java
            Supplier<Apple> c1 = () -> new Apple();
            Supplier<Apple> c1 = Apple::new;
            Apple a1 = c1.get();

            Function<Integer, Apple> c2 = (weight) -> new Apple(weight);
            Function<Integer, Apple> c2 = Apple::new;
            Apple a2 = c2.apple(110);

            List<Integer> weights = Arrays.asList(7, 3, 4, 10);
            public static List<Apple> map(List<Integer> list,
                    Function<Integer, Apple> f){
                List<Apple> result = new ArrayList<>();
                for(Integer i : list){
                    result.add(f.apply(i));
                }
                return result;
            }
            List<Apple> apples = map(weight, (Integer i) -> new Apple(i));
            List<Apple> apples = map(weight, Apple::new);
            
            ```
7. lambda, method reference usecase
    ```java
    // 1. code
    // sort method의 signature
    void sort(Comparator<? super E> c)

    public class AppleComparator implements Comparator<Apple> {
        public int compare(Apple a1, Apple a2){
            return a1.getWeight().compareTo(a2.getWeight());
        }
    }
    inventory.sort(new AppleComparator());

    // 2. anonymous class
    inventory.sort(new Comparator<Apple>() {
        public int compare(Apple a1, Apple a2){
            return a1.getWeight().compareTo(a2.getWeight());
        }
    });

    // 3. lambda expression
    inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
    // lambda parameter type 추론
    inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
    // Comparator는 Comparable 키를 추출해서 Comparator 객체로 만드는 Function 함수를 인수로 받는 정적 메서드 comparing을 포함한다.
    Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
    inventory.sort(comparing((a) -> a.getWeight()));
    
    // 4. method reference
    inventory.sort(comparing(Apple::getWeight));
    ```

8. combinate lambda expression
    1) Comparator
        ```java
        Inventory.sort(comparing(Apple::getWeight)
                    .reversed()                             // 내림차순 정렬
                    .thenComparing(Apple::getCountry));     // 첫번째 비교에서 같으면, 두번째 비교 실행
        ```
    2) Predicate
        ```java
        // redApple 결과를 반전시킨 객체를 만듦
        Predicate<Apple> notRedApple = redApple.negate();
        // a && b || c
        Predicate<Apple> redAndHeadvyAppleOrGreen =
            redApple.and(a -> a.getWeight() > 150)
                    .or(a -> "green".equals(a.getColor()));
        ```
    3) Function
        ```java
        Function<Integer, Integer> f = x -> x + 1;
        Function<Integer, Integer> g = x -> x * 2;
        Function<Integer, Integer> h1 = f.andThen(g);   // g(f(x))
        int result = h1.apply(1);                       // 4
        Function<Integer, Integer> h2 = f.compose(g);   // f(g(x))
        int result = h2.apply(1);                       // 3
        ```
