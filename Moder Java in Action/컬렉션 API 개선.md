# 컬렉션 API 개선

- 자바 9에서 제공되는 of 팩토리 메소드는 불변 컬렉션을 반환한다
    - List.of
    - Set.of
    - Map.of
    - Map.ofEntries
- 자바 8에서는 List, Set 인터페이스에 다음과 같은 디폴트 메서드를 추가했다
    - `removeIf` : Predicate를 만족하는 요소를 제거한다
        
        ```java
        for(Transaction transaction : transactions) {
        	if(Character.isDigit(transaction.getReferenceCode.charAt(0))) {
        		transactions.remove(transaction);
        	}
        }
        ```
        
        - for-each 루프는 내부적으로 Iterator 객체를 사용하므로 두 개의 개별 객체가 컬렉션을 관리한다
        - 이러한 상황에서 컬렉션에 remove를 호출하면 컬렉션의 상태와 반복자의 상태가 서로 동기화되지 않는 문제가 발생하므로 ConcurrentModificationException을 일으킨다
        
        ```java
        for(Iterator<Transaction iterator = transactions.iterator(); iterator.hasNext(); ) {
        	Transaction transaction = iterator.next();
        	if(Character.isDigit(transaction.getReferenceCode.charAt(0))) {
        		iterator.remove();
        	}
        }
        ```
        
        - Iterator 객체를 명시적으로 사용하고 그 객체의 `remove` 메서드를 호출함으로써 위 문제를 해결할 수 있다
        - 하지만 코드가 조금 복잡해졌는데 이러한 코드 패턴을 자바 8의 `removeIf` 메서드로 바꿀 수 있다.
        
        ```java
        transactions.removeIf(transaction -> 
        	Character.isDigit(transaction.getReferenceCode().charAt(0)));
        ```
        
        - removeIf 메서드를 활용하면 코드가 단순해질 뿐 아니라 버그도 예방할 수 있다.
    - `replaceAll` : UnaryOperator 함수를 이용해 리스트의 요소를 바꾼다
        
        ```java
        referenceCodes.stream()
        	.map(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1))
        	.collect(Collectors.toList())
        	.forEach(System.out::println);
        ```
        
        - 스트림 API를 사용하면 새 문자열 컬렉션을 만들기 때문에 기존 컬렉션을 바꾸는 것이 불가능하다
        
        ```java
        for (ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNext(); ) {
        	String code = iterator.next();
        	iterator.set(Character.toUppercase(code.charAt(0)) + code.substring(1));
        }
        ```
        
        - `ListIterator` 객체의 `set` 메서드를 사용하여 기존 컬렉션의 요소를 변경하는 방법이 가능하다.
        - 하지만 코드가 복잡해지고 기존의 `Iterator` 객체와 혼용하면서 쉽게 문제가 발생한다.
        
        ```java
        referenceCodes.replaceAll(code -> 
        	Character.toUpperCase(code.charAt(0)) + code.substring(1));
        ```
        
        - 자바 8의 `replaceAll` 메서드를 사용하면 이를 간단하게 구현할 수 있다
- Map 처리
    - `forEach` 메서드
        
        ```java
        for(Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
        	String friend = entry.getKey();
        	Integer age = entry.getValue();
        	System.out.println(friend + " is " + age + " years old");
        }
        ```
        
        - 기존의 Map.Entry의 반복자를 이용해 키와 값을 확인하는 작업은 아주 번거로운 작업이다
        
        ```java
        ageOfFriends.forEach((friend, age) ->
        	System.out.println(friend + " is " + age + " years old"));
        ```
        
        - `BiConsumer` 를 인수로 받는 `forEach` 메서드를 이용하면 코드를 더 간단하게 구현할 수 있다.
    - 정렬 메서드
        - 스트림에 `Entry.comparingByKey`, `Entry.comparingByValue` 메서드를 통해 맵의 항목을 키 또는 값을 기준으로 정렬할 수 있다.
        
        ```java
        favouriteMovies.entrySet()
        	.stream()
        	.sorted(Entry.comparingByKey())
        	.forEachOrdered(System.out.println);
        ```
        
    - `getOrDefault` 메서드
        - 기존에 찾으려는 키가 존재하지 않으면 널이 반환되므로 NullPointerException을 방지하려면 요청 겨로가가 널인지 확인해야 한다. 기본값을 반환하는 방식으로 이 문제를 해결할 수 있다.
        - `System.out.println(favouriteMovies.getOrDefault(”Olivia”, “Matrix”));`
            - 첫번째 인수로 키를, 두번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두 번째 인수로 받은 기본값을 반환한다.
            - 키가 존재하더라도 값이 널인 상황에서는 `getOrDefault`가 `null`을 반환한다
    - 계산 패턴
        - `computeIfAbsent` : 제공된 키에 해당하는 값이 없거나 널이면, 키를 이용해 새 값을 계산하여 맵에 추가한다
            
            ```java
            Map<String, byte[]> dataToHash = new HashMap<>();
            MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
            
            lines.forEach(line ->
            	dataToHash.computeIfAbsent(line, line -> 
            		messageDigest.digest(line.getBytes(StandardCharsets.UTF_8));
            ```
            
        - `computeIfPresent` : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다
            
            ```java
            String friend = "Raphael";
            List<String> movies = friendsToMovies.get(firend);
            if(movies == null) {
            	movies = new ArrayList<>();
            	friendsToMovies.put(friend, movies);
            }
            movies.add("Star Wars");
            ```
            
            - 위와 같은 패턴의 코드를 `computeIfPresent` 메서드로 간단하게 구현이 가능하다
            
            ```java
            friendsToMovies.computeIfAbsent("Raphael", friend -> new ArrayList<>())
            								.add("Star Wars");
            ```
            
        - `compute` : 제공된 키로 새 값을 계산하고 맵에 저장한다
    - 제거 패턴
        - `favouriteMovies.remove(key, value)`
            - `key`에 대응하는 값이 `value` 인 경우에만 해당 항목을 맵에서 제거한다.
        - `favouriteMovies.removeIf(entry -> entry.getValue() < 10);`
            - Predicate를 인수로 받아 일치하는 맵의 항목을 맵에서 제거한다.
    - 교체 패턴
        - `replaceAll` 메서드 : BiFunction을 적용한 결과로 각 항목의 값을 교체한다.
            - `favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());`
    - 병합(merge) 패턴 : 생략
- concurrentHashMap : 생략

## 정리

- 자바 9는 적은 원소를 포함하며 바꿀 수 없는 리스트, 집합, 맵을 쉽게 만들 수 있도록 List.of, Set.of, Map.of, Map.ofEntries 등의 컬렉션 팩토리를 지원한다. 이들 컬렉션 팩토리가 반환한 객체는 만들어진 다음 바꿀 수 없다.
- List 인터페이스는 removeIf, replaceAll, sort 세 가지 디폴트 메서드를 지원한다.
- Set 인터페이스는 removeIf 디폴트 메서드를 지원한다.
- Map 인터페이스는 자주 사용하는 패턴과 버그를 방지할 수 있도록 다양한 디폴트 메서드를 지원한다.
- ConcurrentHashMap은 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안전성도 제공한다.
