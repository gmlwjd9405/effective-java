# 아이템 26. 로 타입은 사용하지 말라 
클래스와 인터페이스 선언에 타입 매개변수(type patameter)가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네렉 인터페이스**라 한다. 이 둘을 통틀어 **제네릭 타입**(generic type)이라고 한다.

## 로(raw) 타입
- 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.
  - ex) `List<E>`의 로 타입은 `List`이다.
- 타입 선언에서 제네릭 타입 정보가 전부 지워진 것 처럼 동작한다.

> 로 타입을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 앓게된다.
### 로 타입의 안좋은 예시
``` java
private final Collection stamps = ...;
```

``` java
// 실수로 동전을 넣는다.
stamps.add(new Coin(...)); // "unchecked call" 경고를 내뱉는다.
```
컬렉션에서 이 동전을 다시 꺼내기 전에는 오류를 알아차리지 못한다.
``` java
for (Iterator i = stamps.iterator(); i.hasNext();) {
    Stamp stamp = (Stamp) i.next(); // ClassCastException을 던진다.
    stamp.cancel();
}
```
오류는 가능한 발생 즉시, 이상적으로는 컴파일할 때 발견하는 것이 좋다. 런타임때에 발견한다면 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 상당히 떨어져 있을 가능성이 커진다.

### 개선 - 매개변수화된 컬렉션 타입
``` java
private final Collection<Stamp> stamps = ...;
```
이렇게 선언하면 컴퍼일러는 stamps에는 `Stamp`의 인스턴스만 넣아야 함을 인지하게 된다. 따라서 아무런 경고 없이 컴파일된다면 의도대로 동작할 것임을 보장한다.

컴파일러는 컬렉션에서 원소를 꺼내는 모든 곳에서 보이지 않는 형변환을 추가하여 절대 실패하지 않음을 보장한다. 

### 로 타입의 존재 이유
- **호환성** 때문이다.
  - 자바가 제네릭을 받아들일 때 까지 거의 10년이 걸린 탓에 제네렉 없이 짠 코드가 이미 세상을 뒤덮어 버렸다.
  - 마이그레이션 호환성을 위해 제네릭 구현에는 소거 방식을 사용하기로 했다.

## `List`와 `List<Object>`
`List`같은 로 타입은 사용해서는 안되지만,  `List<Object>`처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다.
- `List`: 제네릭 타입과는 아무런 상관이 없다.
- `List<Object>`: 제네릭 타입을 이용하여 모든 타입을 허용한다는 것은 컴파일러에게 전달한다.

> `List<Object>` 같은 매개변수화 타입을 사용할 때와 달리 `List` 같은로 타입을 사용하면 타입 안정성을 잃게된다.

### 예시

## 비한정적 와일드카드 타입(unbounded wildcard type)
제네릭 타입을 쓰고 싶지만 실데 타입 매개변수가 무엇인지 신경쓰고 싶지 않는다면 물음표(?)를 사용하자.
- ex) `Set<E>`의 비 한정적 와일드카드 타입은 `Set<?>`

``` java
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

### 비한정적 와일드 카드(`Set<?>`)와 로 타입(`Set`)의 차이
- **비한정적 와일드카드 타입**은 <u>안전</u>하고, **로 타입**은 <u>그렇지 않다</u>.
  - 로 타입 컬렉션에는 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽다.
  - 반면 `Collection<?>`에는 (null 외에는) 어떤 원소도 넣을 수 없다.
    - 다른 원소를 넣으려 하면 컴파일 시 에러가 발생한다.

즉, 비한정적 와일드카드 타입은 컬렉션 타입 불변식을 훼손하지 못하게 막았다. 구체적으로는 (null 이외의) 어떤 원소도 `Collection<?>`에 넣지 못하게 했으며 컬렉션에서 꺼낼 수 있는 객체 타입도 전혀 알 수 없게 했다.
  - 이러한 제약을 받아들일 수 없다면 제네릭 메서드나 한정적 와일드카드 타입을 사용하면 된다.