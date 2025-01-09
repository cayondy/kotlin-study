# 8장 고차 함수 : 파라미터와 반환 값으로 람다 사용


### 8.1 고차 함수 정의

- 고차 함수 : 람다나 함수 참조를 인자로 넘길 수 있거나 람다나 함수 참조를 반환하는 함수


#### 8.1.1 함수 타입

- 함수 타입을 정의하려면 함수 타입의 파라미터를 ()괄호 안에 넣고 뒤에 -> 화살표를 추가하고 반환타입을 지정하면 된다.
  - (param1, param2 ...) -> returnType 형태
  
```kotlin
// 로컬 변수에 대입하면 타입 추론으로 인해 컴파일러가 함수 타입으로 추론
val sum = { x: Int, y: Int -> x + y }
val action = { println(42) }

// Int 파라미터를 2개 받아서 Int 타입을 반환
val sum: (Int, Int) -> Int { x, y -> x + y }

// 인자 X, 반환값 X
val action: () -> Unit = { println(42) }


```

- Unit 타입은 의미 있는 값을 반환하지 않을 때 사용하는 타입
- 보통의 함수를 정의할 때는 Unit 반환 타입을 생략 가능하지만, 함수 타입을 선언할 때는 반환 타입을 반드시 명시해야 하므로 Unit을 빼먹으면 안됨
- 변수 타입을 함수 타입으로 지정하면 함수 타입에 있는 파라미터로 부터 람다의 파라미터 타입을 유추할 수 있기 때문에 생략 가능
- 함수 타입에서도 반환 타입을 널이 될 수 있는 타입으로 지정이 가능


```kotlin
// 반환 타입이 널 가능
var canReturnNull: (Int, Int) -> Int { x, y => null }

// 함수 타입 전체가 널 가능
var funOrNull: ((Int, Int) -> Int)? = null
```


> **파라미터 이름과 함수 타입**<br>
> - 함수 타입에서 파라미터 이름 지정이 가능하다.
> - 파라미터 이름은 타입 검사 시에 무시
> - 함수 타입의 람다를 정의할 때 파라미터 이름이 함수 타입 선언 시의 이름과 일치하지 않아도 된다.
>   - 가독성을 위한 것, IDE에서 코드에 활용 가능
> 
```kotlin
fun performRequest(
      url: String,
       callback: (code: Int, content: String) -> Unit  // 파라미터에 이름 지정
) {
    /*...*/
}

fun main(args: Array<String>) {
    val url = "http://kotl.in"
    performRequest(url) { code, content -> /*...*/ }  // API에서 제공하는 이름을 람다에 사용
    performRequest(url) { code, page -> /*...*/ }     // 파라미터 이름이 일치하지 않아도 가능
}
```


#### 8.1.2 인자로 받은 함수 호출

- 인자로 받은 함수를 호출하는 구문은 일반 함수 호출 구문과 동일하다.
  - funName(param1, param2 ...)

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {  // 함수 타입인 파라미터 선언
    val result = operation(2, 3)    // 함수 타입 파라미터 호출
    println("The result is $result")
}

fun main(args: Array<String>) {
    twoAndThree { a, b -> a + b }
    twoAndThree { a, b -> a * b }
}
```

**술어 함수를 파라미터로 받는 filter 함수 정의**

```kotlin
/*
String(.): 수신객체 타입
predicate: 파라미터 이름
(Char) -> Boolean: 함수 타입인 파라미터
(Char): 함수 타입인 파라미터의 파라미터 타입
Boolean: 함수 타입인 파라미터의 리턴 타입
(:)String: filter 함수의 리턴 타입
*/
fun String.filter(predicate: (Char) -> Boolean): String {
    val sb = StringBuilder()
    for (index in 0 until length) {
        val element = get(index)
        if (predicate(element)) sb.append(element)  // predicate 파라미터로 전달받은 함수 호출
    }
    return sb.toString()
}

fun main(args: Array<String>) {
    println("ab1c".filter { it in 'a'..'z' })  // 람다를 predicate 파라미터로 전달
}
```


#### 8.1.3 자바에서 코틀린 함수 타입 사용

- 컴파일된 코드 안에서 함수타입은 일반 인터페이스로 바뀐다.
- 즉, 함수 타입의 변수는 FunctionN 인터페이스를 구현하는 객체를 저장한다.
  - Function0<R> : 인자가 없는 함수, Function1<p1, R> : 인자가 하나인 함수 ...
  - 인자의 갯수에 따라 FunctionN 인터페이스를 구현하는 클래스의 인스턴스를 저장
- 각 인터페이스에는 invoke 메소드 정의가 있다
  - invoke 호출 -> 함수 실행
  - invoke 메소드 본문 -> 람다의 본문


 같은 함수의 구현을 비교해보면,
 
```kotlin
/* 코틀린 */
val sum: (Int, Int) -> Int = { a, b -> a + b }

```

```java
/* 자바 */
import kotlin.jvm.functions.Function2;

Function2<Integer, Integer, Integer> sum = (a, b) -> a + b;
System.out.println(sum.invoke(3, 5)); // 출력: 8


```

- 함수 타입을 사용하는 코틀린 함수를 자바에서 호출하는 경우
  
  - 자바8

  ```kotlin
  /* 코틀린 선언 */
  fun processTheAnswer(f: (Int) -> Int) {
      println(f(42))
  }
  
  /* 자바 호출 */
  //processTheAnswer(number -> number + 1);
  // = 43
  ```

  - 자바8 이전 버전

  ```java
  public class ProcessTheAnswerAnonymous {
      public static void main(String[] args) {
          processTheAnswer(
              new Function1<Integer, Integer>() {  // 코틀린 함수 타입 사용
                  @Override
                  public Integer invoke(Integer number) {
                      System.out.println(number);
                      return number + 1;
                  }
              });
      }
  }
  ```

- 자바에서 코틀린 표준 라이브러리가 제공하는 람다를 인자로 받는 확장 함수를 호출할 수 있다.
- 하지만 수신 객체를 확장함수의 첫 번째 인자로 명시적으로 넘겨야 한다.
- (String) -> Unit 처럼 반환 타입이 Unit인 함수 타입의 파라미터 위치에 void를 리턴하는 자바 람다를 넘길 수는 없다.

```java
List<String> strings = new ArrayList();
strings.add("42");
// 코틀린 표준 라이브러리에서 가져온 함수를 자바에서 호출
CollectionsKt.forEach(strings, s -> {  // strings는 확장 함수의 수신 객체
  System.out.println(s);
  return Unit.INSTANCE;  // Unit 타입의 값을 반환
});
```


#### 8.1.4 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터

- 파라미터를 함수 타입으로 선언할 때도 디폴트 값을 정할 수 있다.
- joinString을 호출할 때마다 람다를 전달하면 기본 동작으로도 충분한 대부분의 경우 함수 호출을 더 불편하게 만든다.

* joinToString 예제

```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        transform: (T) -> String = { it.toString() }  // 함수 타입 파라미터를 선언하면서 람다를 디폴트 값으로 지정 => 지금 정의된 람다는 기본동작과 동일
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(transform(element))  // transform 파라미터로 받은 함수를 호출
    }

    result.append(postfix)
    return result.toString()
}

fun main(args: Array<String>) {
    val letters = listOf("Alpha", "Beta")
    println(letters.joinToString())  // 디폴트 변환 함수
    println(letters.joinToString { it.toLowerCase() }) // 람다를 인자로 전달
    println(letters.joinToString(separator = "! ", postfix = "! ",
           transform = { it.toUpperCase() }))  // 람다를 포함한 여러 인자를 전달
}

```

- 널이 될 수 있는 함수 타입을 사용할 수도 있다.
- 널이 될 수 있는 타입으로 함수를 받으면 그 함수를 직접 호출할 수 없다.

* 널이 될 수 있는 함수 타입 파라미터를 사용한 joinToString 예제

```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        transform: ((T) -> String)? = null  // 널이 될 수 있는 함수 타입 파라미터 선언
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        val str = transform?.invoke(element)  // transform이 널일 수 있으므로 직접 호출 불가 -> ?. 안전 호출 사용
            ?: element.toString()  // 엘비스 연산자로 람다를 인자로 받지 않은 경우 처리
        result.append(str)
    }

    result.append(postfix)
    return result.toString()
}

fun main(args: Array<String>) {
    val letters = listOf("Alpha", "Beta")
    println(letters.joinToString())
    println(letters.joinToString { it.toLowerCase() })
    println(letters.joinToString(separator = "! ", postfix = "! ",
           transform = { it.toUpperCase() }))
}

```

#### 8.1.5 함수를 함수에서 반환

- 다른 함수를 반환하는 함수를 정의하려면 함수의 반환 타입으로 함수 타입을 지정해야 한다.
- 함수를 반환하려면 return 식에 람다나 멤버 참조나 함수 타입의 값을 계산하는 식을 넣으면 된다.

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(
        delivery: Delivery): (Order) -> Double {  // 함수를 반환하는 함수 선언
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }

    return { order -> 1.2 * order.itemCount }  // 함수에서 람다 반환
}

fun main(args: Array<String>) {
    val calculator =  // 반환받은 함수를 저장할 변수(함수 타입)
        getShippingCostCalculator(Delivery.EXPEDITED)
    println("Shipping costs ${calculator(Order(3))}")  // 반환받은 함수를 호출
}
```


- 아래 예제에서 getPredicate 메소드는 filter 함수에게 인자로 넘길 수 있는 함수를 반환한다.
- 일반 타입의 값을 함수가 반환하는 것처럼 함수 타입을 사용해서 함수에서 함수를 반환할 수 있다.

```kotlin
data class Person(
        val firstName: String,
        val lastName: String,
        val phoneNumber: String?
)

class ContactListFilters {
    var prefix: String = ""
    var onlyWithPhoneNumber: Boolean = false

    fun getPredicate(): (Person) -> Boolean {  // 함수를 반환하는 함수 정의
        val startsWithPrefix = { p: Person ->
            p.firstName.startsWith(prefix) || p.lastName.startsWith(prefix)
        }
        if (!onlyWithPhoneNumber) {
            return startsWithPrefix  // 함수 타입의 변수를 반환
        }
        return { startsWithPrefix(it)
                    && it.phoneNumber != null }  // 람다를 반환
    }
}

fun main(args: Array<String>) {
    val contacts = listOf(Person("Dmitry", "Jemerov", "123-4567"),
                          Person("Svetlana", "Isakova", null))
    val contactListFilters = ContactListFilters()
    with (contactListFilters) {
        prefix = "Dm"
        onlyWithPhoneNumber = true
    }
    println(contacts.filter(
        contactListFilters.getPredicate()))  // getPredicate이 반환한 함수를 filter의 인자로 넘김
}

```


#### 8.1.6 람다를 활용한 중복 제거

- 람다를 사용하면 간결하고 쉽게 코드 중복 제거가 가능하다.

* average 함수를 활용한 윈도우 사용자의 평균 방문시간 출력

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/", 22.0, OS.MAC),
    SiteVisit("/login", 12.0, OS.WINDOWS),
    SiteVisit("/signup", 8.0, OS.IOS),
    SiteVisit("/", 16.3, OS.ANDROID)
)

val averageWindowsDuration = log
    .filter { it.os == OS.WINDOWS }
    .map(SiteVisit::duration)
    .average()

fun main(args: Array<String>) {
    println(averageWindowsDuration)
}

```

* 맥 사용자 통계 출력 + OS 필터링 중복 제거

```kotlin
/* val log 정의까지는 동일 */
fun List<SiteVisit>.averageDurationFor(os: OS) =  // 중복 코드를 별도 함수로 추출
        filter { it.os == os }.map(SiteVisit::duration).average()

fun main(args: Array<String>) {
    println(log.averageDurationFor(OS.WINDOWS))
    println(log.averageDurationFor(OS.MAC))
}
```


* 모바일 디바이스 사용자 출력 (필터링 조건이 여러개: IOS, ANDROID)

```kotlin
/* val log 정의까지는 동일 */
val averageMobileDuration = log
    .filter { it.os in setOf(OS.IOS, OS.ANDROID) }
    .map(SiteVisit::duration)
    .average()

fun main(args: Array<String>) {
    println(averageMobileDuration)
}

```

* IOS 사용자의 /signup 페이지 평균 방문 시간 출력

```kotlin
/* val log 정의까지는 동일 */
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) =
        filter(predicate).map(SiteVisit::duration).average()

fun main(args: Array<String>) {
    println(log.averageDurationFor {
        it.os in setOf(OS.ANDROID, OS.IOS) })
    println(log.averageDurationFor {
        it.os == OS.IOS && it.path == "/signup" })
}

```


### 8.2 인라인 함수: 람다의 부가 비용 없애기

- 코틀린에서 람다를 함수 인자로 넘기는 구문은 if나 for와 같은 일반 문장과 비슷하다.
- 람다를 활용한 코드의 성능이 일반 자바 문장보다 효율적으로 작동하는 지 확인해볼 필요가 있다.
    - 코틀린에서 람다는 무명 클래스로 컴파일하지만 사용할 때마다 새로운 객체가 생성되는 것은 아니다.
    - 람다가 실질적으로 실행되는 시점에 무명 클래스 객체가 생긴다. (5장 설명)
 
- 람다의 실행 시점에 무명 클래스 생성에 따른 부가 비용이 든다. -> 부가 비용? 메모리 차지?
    - 일반 함수를 사용한 구현보다 덜 효율적


**inline 변경자**
- 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트 코드로 바꿔치기 해준다.
- 함수 앞에 inline 변경자를 붙여서 사용

#### 8.2.1 인라이닝이 작동하는 방식

- 함수를 inline으로 선언하면 함수를 호출하는 코드를 호출하는 바이트코드 대신 함수 본문을 번역한 바이트코드로 컴파일한다.


* 인라인 함수 정의
 
```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {

  lock.lock()
  try {
    return action()
  } finally {
    lock.unlock()
  }
}

val l = Lock()
synchronized(1) {
  // ...
}

```

* synchronized() 사용 예제

```kotlin
fun foo(l: Lock) {
  println("Before sync")

  synchronized(1) {  // 컴파일하면 synchronized 함수가 인라이닝 된다.
    println("Action")
  }

  println("After sync")
}

/* 컴파일 버전 */
fun __foo__(l: Lock) {
  println("Before sync")  // foo 함수 본문 

  l.lock()  // synchronized 함수 인라이닝 start
  try {
    println("Action")  // 람다 코드 본문이 인라이닝
  } finally {
    l.unlock()
  }  // synchronized 함수 인라이닝 end

  println("After sync")  // foo 함수 본문 
}


```
- synchronized 함수의 본문 뿐만 아니라 전달된 람다의 본문도 함께 인라이닝 된다.
- 람다의 본문에 의해 만들어지는 바이트코드는 synchronized 정의의 일부분으로 인식해 무명 클래스를 생성하지 않는다.
- 인라인 함수를 호출하면서 함수 타입의 변수를 넘기는 것도 가능하다.
- 한 인라인 함수를 두 곳에서 각각 다른 람다를 사용해서 호출하면 두 호출은 각각 인라이닝된다.

```kotlin
class LockOwner(val lock: Lock) {
  fun runUnderLock(body: () -> Unit) {
    synchronized(lock, body)  // 함수 타입의 변수
  }
}

/* 컴파일 버전 */
class LockOwner(val lock: Lock) {
  fun __runUnderLock__(body: () -> Unit) {
    l.lock() 

    try {
      body()  // synchronized를 호출하는 부분에서 람다를 알 수 없음 -> 인라이닝 되지 않음
    }
    finally {
      l.unlock()
    }
  }
}
```


#### 8.2.2 인라인 함수의 한계

- 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달 받은 람다를 본문에 사용하는 방식이 한정된다.
- 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 변수를 사용한다면 람다를 인라이닝할 수 없다.
- 인라인 함수의 본문에서 람다 식을 바로 호출하거나 람다 식을 인자로 전달받아 호출하는 경우에는 인라이닝이 가능하다.
  - 아니라면 "Illegal usage of inline parameter" 메세지와 함께 인라이닝을 금지
 

- 시퀀스 동작 메소드 중에 람다를 받아서 모든 시퀀스 원소에 그 람다를 적용한 새 시퀀스를 반환하는 함수가 많다.
- 인자로 받은 람다를 시퀀스 객체 생성자의 인자로 넘기는 경우가 있다.
* Sequence.map 을 정의

```kotlin

fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
    return TransformingSequence(this, transform)
}

```
- 이 map 함수는 transform 함수 값을 호출하지 않는 대신, TransformingSequence 클래스 생성자에 함수 값을 넘긴다.
- 여기서 transform은 TransformingSequence 생성자에서 프로퍼티로 저장되고 인라이닝이 아닌 무명클래스 인스턴스로 만들어진다.

- noinline 변경자를 사용해서 인라이닝을 금지할 수 있다.
```kotlin
inline fun foo (inlined: () -> Unit, noinline notInlined: () -> Unit) {
    //...
}

```


#### 8.2.3 컬렉션 연산 인라이닝


* 람다를 이용해 컬렉션 거르기
```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun main(args: Array<String>) {
    println(people.filter { it.age < 30 })
}
```

* 람다를 사용하지 않고 컬렉션을 직접 거르기
```kotlin
fun main(args: Array<String>) {
    //println(people.filter { it.age < 30 })
    val result = mutableListOf<Person>()
    for (person in people) {
        if (person.age < 30) result.add(person)
    }
    println(result)
}

```

- 코틀린의 filter 함수는 인라인 함수이다.
- 따라서 filter 함수의 바이트코드는 그 함수에 전달된 람다 본문의 바이트코드와 함께 호출한 위치에 들어간다.
- filter를 사용한 바이트코드와 뒷 예제의 차이가 바이트코드가 거의 없음
  - 컬렉션에 대한 안전한 연산과 성능을 제공 -> 안전!


* filter와 map을 연결 
```kotlin
fun main(args: Array<String>) {
    println(people.filter { it.age > 30 }
                  .map(Person::name))
}
```

- map도 인라인 함수이고 추가 객체나 클래스 생성은 없다.
- 하지만 필터링 하는 과정에서 중간리스트가 생기는 데 map은 중간리스트를 읽어서 사용한다.

- 처리할 원소가 많아지면 중간리스트 사용을 위한 부가 비용도 커진다.
   - 리스트 대신 asSequence를 사용하면 줄일 수 있음
   - 각 중간시퀀스는 람다를 필트에 저장하는 객체로 표현되고 최종 연산은 중간시퀀스의 여러 람다를 연쇄 호출한다.
   - 주의: 시퀀스는 람다를 인라이닝 하지 않음 -> 크기가 작은 컬렉션은 일반 컬렉션 연산의 성능이 나을 수 있음
 
#### 8.2.4 함수를 인라인으로 선언해야 하는 경우

- 람다를 인자로 받는 함수의 경우 인라이닝하면 이익이 많다.
  - 일반 함수는 JVM에서 가장 이익이 되는 방법으로 호출하기 때문에 꼭 성능 확인 후 적용이 필요!
- 함수 호출 비용이 줄고 람다 표현 클래스, 람다 인스턴스 객체를 생성할 필요가 없다.
- 인라이닝을 사용하면 일반 람다에서 사용할 수 없는 기능 사용이 가능하다.
   - 넌로컬(non-local) 반환 => 뒤에서 설명
- 주의: 인라이닝되는 본문의 양이 많은 경우, 바이트코드가 커질 수 있음 => 람다 인자와 무관한 부분을 비인라인 함수로 추출


#### 8.2.5 자원 관리를 위해 인라인된 람다 사용


- 자원: 파일, 락, 데이터베이스 트랜잭션 등...
- 자원 관리 패턴을 만들 때 try/finally 문을 사용해 try 직전 자원을 획득하고 finally에서 자원을 해제하는 형태
- 자바의 synchronized는 락 객체를 인자로 취하는데, 코틀린에는 Lock 인터페이스의 확장 함수인 withLock이 있다.

```kotlin
val l: Lock =
l.withLock {   // 락을 잠근 다음 주어진 동작 수행
   // 락에 의해 보호되는 자원을 사용
}
```
* 자바 try-with-resource(자바7 부터)
```java
static String readFirstLineFromFile(String path) throw IOException {
    try (BufferedReader br =
      new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}

```

- 코틀린 표준 라이브러리에는 자바 try-with-resource 와 동일한 기능을 하는 use 함수가 있다.

**use 함수**
- 닫을 수 있는 자원에 대한 확장 함수
- 람다 호출 후에 자원을 닫음, 예외 발생 시에도 닫음!
- 인라인 함수


* use 함수를 자원 관리에 활용
```kotlin
import java.io.BufferedReader
import java.io.FileReader
import java.io.File

fun readFirstLineFromFile(path: String): String {
    BufferedReader(FileReader(path)).use { br ->   // BufferedReader 객체를 만들고 use 함수 호출, 파일에 대한 연산을 실행할 람다를 전달
        return br.readLine()   // 자원(여기서는 파일)에서 맨처음 가져온 한 줄을 람다가 아닌 readFirstLineFromFile 에서 반환 => 넌로컬 반환
    }
}

```


### 8.3 고차 함수 안에서 흐름 제어

- 루프와 같은 명령형 코드를 람다로 변환하면서 생기는 return 문제를 해결하는 방법은?

#### 8.3.1 람다 안의 return문: 람다를 둘러싼 함수로부터 반환


-

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun lookForAlice(people: List<Person>) {
    for (person in people) {
        if (person.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")  // people에 Alice가 없으면 출력
}

fun main(args: Array<String>) {
    lookForAlice(people)
}
```
- 위 예제를 forEach로 구현해서 사용할 수도 있다.

**람다안에서의 return**
- 람다안에서 리턴을 사용하면 람다로부터만 반환되는 게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환된다.
- 자신을 둘러싼 블록보다 더 바깥 블록을 반환하게 만드는 return문을 **넌로컬 리턴**이라고 한다.
- 넌로컬 리턴은 람다를 인자로 받는 함수가 인라인 함수일 경우만 가능하다.


 
* forEach 사용
```kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("Alice", 29), Person("Bob", 31))

fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return
        }
    }
    println("Alice is not found")
}

fun main(args: Array<String>) {
    lookForAlice(people)
}

```


#### 8.3.2 람다로부터 반환: 레이블을 사용한 return

- 람다에서도 로컬 리턴이 가능
- 로컬 리턴은 for에서의 break와 동일한 기능
- 리턴을 구분하기 위해 @:레이블을 사용한다.
- 리턴으로 실행을 끝내고 싶은 함수앞에 레이블을 붙이고, 리턴 키워드 뒤에 레이블을 추가하면 된다.

* 레이블로 로컬 리턴 사용
```kotlin

fun lookForAlice(people: List<Person>) {
    people.forEach label@{  // 람다 앞에 레이블
        if (it.name == "Alice") return@label
    }
    println("Alice might be somewhere")  // 항상 출력
}


```

* 함수 이름을 return 레이블로 사용
```kotlin

fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") return@forEach
    }
    println("Alice might be somewhere")
}


```



#### 8.3.3 무명 함수: 기본적으로 로컬 return

* 무명 함수 안에서 return 사용
```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (person.name == "Alice") return
        println("${person.name} is not Alice")
    })
}

```
