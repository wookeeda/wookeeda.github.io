---
title: "chap02.동작 파라미터화 코드 전달하기"
date: 2018-12-10 00:03:30
categories: java8
---

- behavior parameterization을 이용하면 변화하는 요구사항에 효과적으로 대처할 수 있다.
    1. 리스트의 모든 요소에 '어떤 동작'을 수행할 수 있음
    2. 리스트 관련 작업을 끝낸 다음에 '어떤 다른 동작'을 수행할 수 있음
    3. 에러가 발생하면 '정해진 어떤 다른 동작'을 수행할 수 있음

    ```
    ex)
    goAndBuy : 슈퍼마켓 가서 식료품을 사와라.
    ->
    go(Man::buy) : go method에 원하는 동작을 인수로 전달할 수 있음.

## 변화에 대응하는 코드 구현하기
1. 녹색 사과 필터링
    ```java
    public static List<Apple> filterGreenApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>();
        for(Apple a : invetory){
            if("green".equals(a.getColor())){
                result.add(a);
            }
        }
        return result;
    }
    ```
2. 색을 파라미터화
    ```java
    public static List<Apple> filterApplesByColor(List<Apple> inventory, String color) {
        List<Apple> result = new ArrayList<>();
        for(Apple a : invetory){
            if(color.equals(a.getColor())){
                result.add(a);
            }
        }
        return result;
    }

    List<Apple> greenApples = filterApplesByColor(inventory, "green");
    List<Apple> greenApples = filterApplesByColor(inventory, "red");

    public static List<Apple> filterApplesByColor(List<Apple> inventory, int weight) {
        List<Apple> result = new ArrayList<>();
        for(Apple a : invetory){
            if(a.getWeight() > weight){
                result.add(a);
            }
        }
        return result;
    }
    ```
    - 대부분 중복 -> DRY(don't repeat yourself)
    - 어떤 것을 기준으로 필터링할지 가리키는 플래그 : **절대 이 방법을 사용하지 말 것**
3. 가능한 모든 속성으로 필터링
    ```java
    public static List<Apple> filterApplesByColor(List<Apple> inventory, String color, int weight, boolean flag) {
        List<Apple> result = new ArrayList<>();
        for(Apple a : invetory){
            if( (flag && a.getColor().equals(color))
                || (!flag && a.getWeight() > weight) ){
                result.add(a);
            }
        }
        return result;
    }
    List<Apple> greenApples = filterApplesByColor(inventory, "green", 0, true);
    List<Apple> greenApples = filterApplesByColor(inventory, "", 150, false);
    ```
    - true / false는 뭘 의미하는 것인가.
4. behavior parameterization
    - predicate : 사과의 어떤 속성에 기초해서 불린값을 반환
    - 선택 조건을 결정하는 인터페이스를 정의한다.
    ```java
    public interface ApplePredicate {
        boolean test(Apple apple);
    }
    public class AppleHeavyWeightPredicate implements ApplePredicate {
        public boolean test(Apple apple){
            return apple.getWeight() > 150;
        }
    }
    public class AppleGreenColorPredicate implements ApplePredicate {
        public boolean test(Apple apple){
            return "green".equals(apple.getColor());
        }
    }
    ```
    - ApplePredicate : 알고리즘 패밀리
    - AppleHeavyWeightPredicate / AppleGreenColorPredicate : 전략
    ```java
    public static List<Apple> filterApplesByColor(List<Apple> inventory, ApplePredicate p) {
        List<Apple> result = new ArrayList<>();
        for(Apple a : invetory){
            // predicate 객체로 사과 검사 조건을 캡슐화
            if(p.test(apple)){
                result.add(apple);
            }
        }
        return result;
    }
    public class AppleRedAndHeavyPredicate implements ApplePredicate {
        public boolean test(Apple apple){
            return "red".equals(apple.getColor()) 
                    && apple.getWeight() > 150;
        }
    }
    List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
    ```
    - 컬렉션 탐색 로직과
    - 각 항목에 적용할 동작을 분리

5. anonymous class : 복잡한 과정 간소화
    - anonymous class : 클래스의 선언과 인스턴스화를 동시에 수행
    ```java
    List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPApplePredicate(){
        public boolean test(Apple apple){
            return "red".equals(apple.getColor());
        }
    });
    ```
    - but, 여전히 많은 공간을 차지한다.
    - 코드의 장황함은 나쁜 특성 : verbosity
    - 장황한 코드는 구현하고 유지보수하는 데 시간이 오래 걸릴 뿐 아니라
    - **읽는 즐거움을 빼앗는 요소로**, 개발자로부터 외면받는다.

6. lambda
    ```java
    List<Apple> redAndHeavyApples = filterApples(inventory, (Apple a) -> "red".equals(a.getColor()));
    ```

7. list 형식으로 추상화
    - apple 이외의 다양한 물건에서 필터링이 작동하도록 추상화
    ```java
    public interface Predicate<T> {
        boolean test(T t);
    }
    public static <T> List<T> filter(List<T> list, Predicate<T> p){
        List<T> result = new ArrayList<>();
        for(T e : list){
            if(p.test(e)){
                result.add(e);
            }
        }
        return result;
    }
    List<Apple> redAndHeavyApples = filterApples(inventory, (Apple a) -> "red".equals(a.getColor()));
    List<String> evenNumbers = filterApples(inventory, (Integer i) -> i % 2 == 0);
    ```

## example
1. Comparator
    1. 과거
        ```java
        Collections.sort(inventory, new Comparator<Apple>(){
            public int compare(Apple a1, Apple a2){
                return a1.getWeight().compareTo(a2.getWeight());
        })
        ```
    2. java8
        ```java
        public interface Comparator<T> {
            public int compare(T o1, T o2);
        }
        inventory.sort(new Comparator<Apple>(){
            public int compare(Apple a1, Apple a2){
                return a1.getWeight().compareTo(a2.getWeight());
            }
        })
        // Lambda
        inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
        ```
2. Runnable
    ```java
    Thread t = new Thread(new Runnable(){
        public void run() {
            System.out.println("Hello world");
        }
    });
    // Lambda
    Thread t = new Thread(() -> System.out.println("Hello world"));
    ```
3. GUI event
    ```java
    button.setOnAction((ActionEvent event) -> label.setText("sent"));
    ```
    
## summary
1. behavior parameterization에서는 method 내부적으로 다양한 동작을 수행할 수 있도록 코드를 method parameter로 전달한다.
2. 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있다.
3. anonymous class로 어느정도 코드를 깔끔하게 할 수 있지만, Lambda가 짱이다.