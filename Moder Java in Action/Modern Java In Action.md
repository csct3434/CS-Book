### 함수형 프로그래밍

- 함수형 인터페이스 : 정확히 하나의 추상 메서드를 지정하는 인터페이스
    - 디폴트 메서드가 여러 개 있더라도 추상 메서드가 오직 하나면 함수형 인터페이스
- 함수 디스크립터 : 함수형 인터페이스의 추상 메서드의 시그니처
    - java.util.function 패키지로 다양한 함수형 인터페이스를 제공
- 동작 파라미터화 : 전략패턴

### 형식 추론

- 람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않음
- 람다가 사용되는 context를 이용해서 람다의 형식(target type, 특정 context에서 기대되는 람다 표현식의 형식)을 추론함
    - ex) 메서드의 선언을 확인하여 함수형 인터페이스의 추상 메서드가 묘사하는 함수 디스크립터를 인자로 전달된 람다식이 만족하는지 확인

### 람다 캡처링

- 람다 표현식의 바디에서 자유변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 참조하는 것
- 제약사항
    - 인스턴스 변수와 정적 변수는 자유롭게 캡처 가능
    - 지역 변수는 **effectivly final variable** 이어야 한다. 즉 명시적으로 final  키워드가 붙거나 final 처럼 변경없이 사용해야 한다는 것이다
- *어렵다. 보충 필요 (p.112)*

### 메소드 참조

- 하나의 메서드를 참조하는 람다를 간단히 표현할 수 있는 문법
- 메소드 참조 유형
    1. 정적 메서드 참조
        
        ```java
        ToIntFunction<String> stringToInt =
        	(String s) -> Integer.parseInt(s);
        
        ToIntFunction<String> stringToInt = Integer::parseInt;
        ```
        
    2. 다양한 형식의 인스턴스 메서드 참조
        
        ```java
        BiPredicate<List<String>, String> contains = 
        	(list, element) -> list.contains(element);
        
        BiPredicate<List<String>, String> contains = List::contains;
        ```
        
    3. 기존 객체의 인스턴스 메서드 참조
        
        ```java
        Predicate<String> startsWithNumber =
        	(String string) -> this.startsWithNumber(string);
        
        Predciate<String> startsWithNumber = this::startsWithNumber;
        ```
        
- *형 추론이 체계적으로 어떻게 되는건지 명확하게 파악하기가 힘드네. 특히 2번.*

### 활용 예시

```java
void sort(Comparator<? super E> c)

// Before
public class AppleComparator implements Comparator<Apple> {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
}
inventory.sort(new AppleComparator());

// After
inventory.sort(comparing(Apple::getWeight));
```

### 함수형 인터페이스의 디폴트 메서드를 활용한 람다 표현식의 조합

- Comparator 조합
    
    ```java
    inventory.sort(comparing(Apple::getWeight)
    	.reversed() // 무게를 내림차순으로 정렬
    	.thenComparing(Apple::getCountry)); // 무게가 같다면 국가별로 정렬
    ```
    
- Predicate 조합
    
    ```java
    Predicate<Apple> redApple = apple -> apple.getColor().equals("RED");
    
    Predicate<Apple> notRedApple = redApple.negate();
    
    Predicate<Apple> redAndHeavyApple = 
    	redApple.and(apple -> apple.getWeight() > 150);
    
    Predicate<Apple> redAndHeavyAppleOrGreen =
    	redApple.and(apple -> apple.getWeight() > 150)
    					.or(apple -> apple.getColor().equals("GREEN");
    ```
    
- Function 조합
    
    ```java
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    
    Function<Integer, Integer> h = f.andThen(g); // h = g(f(x))
    
    Function<Integer, Integer> i = f.compose(g); // i = f(g(x))
    ```
