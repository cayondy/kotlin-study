# 3장 함수 정의와 호출



## 3.1 코틀린에서 컬렉션 만들기

```kotlin
// 집합
val set = hashSetOf(1, 7, 53)
```
```kotlin
// 리스트와 맵
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```
- 여기서 to는 키워드가 아닌 일반 함수이다.
```console
>>> println(set.javaClass) 
class java.util.HashSet
>>> println(list.javaClass)
class java.util.ArrayList
>>> println(map.javaClass)
class java.util.HashMap
```
- 코틀린은 표준 자바 컬렉션을 활용한다.

## 3.2 함수를 호출하기 쉽게 만들기

```kotlin
// 리스트 출력 기본
val list = listOf(1, 2, 3)
println(list)  
>>>[1, 2, 3]
```
```kotlin
// 리스트 3.1 joinToString() 함수의 초기 구현
fun <T> joinToString(
        collection: Collection<T>,
        separator: String,
        prefix: String,
        postfix: String
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)    
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
```kotlin
// joinToString 함수를 이용한 리스트 출력 변형
val list = listOf(1, 2, 3)
println(joinToString(list, "; ", "(", ")"))
>>>(1; 2; 3)
```


### 3.2.1 이름 붙인 인자

```kotlin
joinToString(collection, " ", " ", ".")
```
- 위 형태로 호출 할 때 전달한 인자 값이 어떤 값인지 알기 어려움
  
```java
/* Java */
joinToString(collection, /* separator */ " ",  /* prefix */ " ",
    /* postfix */ ".");
```

```kotlin
joinToString(collection, separator = " ", prefix = " ", postfix = ".")
```
- 코틀린으로 작성한 함수는 함수에 전달하는 인자의 이름을 명시할 수 있음
- 대신 인자를 하나라도 명시한다면 모든 인자를 명시하기 (혼돈 방지)

  > IntelliJ 에서 함수가 파라미터 이름을 변경한 경우,
  > 대상 함수를 호출할 때 붙인 인자의 이름을 추적 가능!
  > 대신 Refactor > Rename or Change Signiture 으로 이름을 변경해야 함
  - 자바로 작성한 함수는 코틀린 컴파일러가 위 내용을 인식하지 못함
  - 자바8 이후 추가된 선택적 특징(함수 파라미터의 정보를 넣는 것)인데 코틀린은 JDK 6 와 호환되기 때문

### 3.2.2 디폴트 파라미터 값

```kotlin
// 리스트 3.2 디폴트 파라미터 값을 사용해 joinToString() 정의하기
fun <T> joinToString(
        collection: Collection<T>,
        separator: String = ", ", 
        prefix: String = "",      
        postfix: String = ""      
): String
```
- 디폴트 파라미터를 지정함으로서 모든 인자를 쓰거나 생략 가능하다.

```kotlin
joinToString(list, ", ", "", "")
joinToString(list)
joinToString(list, "; ")
```
- 호출할 때 함수를 선언할 때와 같은 순서로 인자를 지정해야 한다.
- 일부를 생략할 경우, 뒷부분의 인자가 생략된다.

```kotlin
joinToString(list, suffix = ";", prefix = "# ")
>>># 1, 2, 3;
```
- 지정하고 싶은 인자의 이름을 붙여서 순서와 관계없이 지정할 수 있음

  > - 디폴트 값과 자바
  > 자바에는 디폴트 값이라는 개념이 없기 때문에 코틀린 함수를 자바에서 호출하는 경우,
  > 코틀린 함수가 디폴트 파라미터 값을 제공하더라도 모든 인자를 명시해야 함
  >
  > 코틀린 함수를 자바에서 자주 호출해야 한다면 @JvmOverLoads 어노테이션을 함수에 추가하면 코틀린 컴파일러가 마지막 파라미터부터 하나씩 생략하여 오버로딩한 자바 메소드를 추가해줌!
  > ```java
  > String joinToString(Collection<T> collection, String separator, String prefix, String postfix);
  > 
  > String joinToString(Collection<T> collection, String separator,String prefix);
  >
  > String joinToString(Collection<T> collection, String separator);
  >
  > String joinToString(Collection<T> collection);
  > ```


### 3.2.3 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

- 자바에서는 모든 코드를 클래스의 메소드로 작성하지만 코틀린에서는 함수를 직접 소스 파일의 최상위 수준, 다른 클래스의 밖에 위치 시킬 수 있다.

- 최상위 프로퍼티
  - 프로퍼티도 파일의 최상위 수준에 놓을 수 있음
  ```kotlin
  
  var opCount = 0  
  fun performOperation() {
      opCount++  
      / ...
  }
  fun reportOperationCount() {
      println("Operation performed $opCount times")  
  }
  ```
  - 최상위 프로퍼티를 활용해 상수를 추가할 수 있음
  ```kotlin
  const val UNIX_LINE_SEPARATOR = "\n"
  // public static final String  UNIX_LINE_SEPARATOR = "\n";
  ```
  - const는 원시 타입과 string 타입의 프로퍼티만 지정이 가능


## 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

기존 자바 API를 재작성하지 않고 코틀린이 제공하는 기능을 사용하기 위함

1. 확장 함수
- 확장 함수를 만들기 위해서는 추가하려는 함수 이름 앞에 그 함수가 확장할 클래스 이름을 덧붙이면 된다.
- 클래스 이름을 수신 객체 타입(receiver type), 확장 함수가 호툴되는 대상이 되는 값을 수신 객체(receiver object)

```kotlin
package strings

// 수신 객체 타입: String , 수신 객체: this
fun String.lastChar(): Char = this.get(this.length - 1)

fun String.lastChar(): Char1 = get(length - 1)
```
- 수신 객체 타입은 확장이 정의될 클래스의 타입이며, 수신 객체는 그 클래스에 속한 인스턴스 객체이다.
- this 는 생략 가능

확장 함수 내부에서는 일반적인 인스턴스 메소드 처럼 수신 객체의 메소드나 프로퍼티를 바로 사용 가능하다.
- 하지만 확장 함수가 캡슐화를 깨는 것은 아님
  -> 확장 함수 안에서 클래스 내부에서만 사용 가능한 private, protected 멤버를 사용할 수 없음

### 3.3.1 임포트와 확장 함수

```kotlin
import strings.*

val c = "Kotlin".lastChar()
```
- 클래스 임포트와 동일한 구문 사용

```kotlin
import strings.lastChar as last

val c = "Kotlin".last()
```
- as 를 이용한 알리아스 지정이 가능

### 3.3.2 자바에서 확장 함수 호출
```java
// StringUtil.kt 파일에 정의한 확장 함수 호출
char c = StringUtilKt.lastChar("Java");
```

### 3.3.3 확장 힘수로 유틸리티 함수 정의

```kotlin
// 리스트 3.4 joinToString()를 확장으로 정의하기
fun <T> Collection<T>.joinToString(  
        separator: String = ", ",    
        prefix: String = "",        
        postfix: String = ""        
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) 
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```
- 컬렉션에 확장 함수로 선언
- 클래스의 멤버처럼 호출이 가능하다.
 
```kotlin
val list = listOf(1, 2, 3)
println(list.joinToString(separator = "; ", prefix = "(", postfix = ")"))
// (1; 2; 3)

val list1 = arrayListOf(1, 2, 3)
println(list.joinToString(" "))
// 1 2 3
```

- 확장 함수는 정적 메소트 호출에 대한 문법적인 편의로 더 구체적인 타입을 수신 객체 타입으로 지정이 가능하다.
```kotlin
// 문자열 컬렉션에 대해서만 호출 가능한 join 함수 정의
fun Collection<String>.join(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
) = joinToString(separator, prefix, postfix)

>>> println(listOf("one", "two", "eight").join(" "))
one two eight
```

### 3.3.4 확장 함수는 오버라이드할 수 없다

코틀린의 메소드 오버라이드는 일반적인 객체지향 메소드 오버라이드와 같다.

```kotlin
// 리스트 3.5 멤버 함수 오버라이드하기
open class View {
    open fun click() = println("View clicked")
}
class Button: View() { 
    override fun click() = println("Button clicked")
}
```
```kotlin
val view: View = Button()
view.click() 
// Button clicked
```
  > - Button 이 View 의 하위 타입이기 때문에 가능
  > 실행 시점에 객체 타입에 따라 동적으로 호출될 대상을 결정하는 방법을 동적 디스패치
  > 컴파일 시점에 알려진 변수 타입에 따라 정해진 메소드를 호출하는 방식은 정적 디스패치
  > 동적(dynamic): 실행 시점
  > 정적(static): 컴파일 시점
```kotlin
// 리스트 3.6 확장 함수는 오버라이드할 수 없다.
fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a button!")

```
```kotlin
val view: View = Button()
view.showOff() 
// I'm a view!
```
- 확장 함수는 클래스의 일부가 아님
- 확장 함수는 클래스 밖에 선언됨
- 확장 함수를 호출할 때 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 확장 함수가 호출된 지 결정
- 변수에 저장된 객체의 동적인 타입에 의해 결정되지 않음
  - view의 실제 타입은 Button이지만 수신 객체가 View 이므로 View의 확장 함수가 호출됨!

  > 어떤 클래스를 확장한 변수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 멤버 함수가 호출된다. (멤버 함수의 우선순위가 더 높음!)
  > 클래스에 확장 함수와 이름이 같은 멤버 함수를 새로 추가했다면 재컴파일한 순간부터 새로 추가한 멤버 함수를 사용한다.

### 3.3.5 확장 프로퍼티

확장 프로퍼티를 사용하여 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있다.
프로퍼티는 상태를 저장할 적절한 방법이 없기 때문에 실제로는 아무 상태도 가지지 못함.
하지만 문법적으로 더 짧은 코드를 작성할 수 있다.

```kotlin
// 3.7 확장 프로퍼티 선언하기
val String.lastChar: Char
    get() = get(length - 1)
```
- 기존 클래스의 인스턴스 객체에 필드가 추가 되는 것이 아니기 때문에 게터를 제공하지 못함
- 최소한 게터는 꼭 정의 해야함
- 초기화에서 계산한 값을 담을 장소도 없기 때문에 초기화 코드도 사용 못함

```kotlin
// 3.8 변경 가능한 확장 프로퍼티 선언하기
var StringBuilder.lastChar: Char
    // 프로퍼타 게터
    get() = get(length - 1)
    // 프로퍼티 세터 
    set(value: Char) {
        this.setCharAt(length - 1, value)  
    }
```
```kotlin
println("Kotlin".lastChar)
// n
val sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
println(sb)
// Kotlin!
```
- 확장 프로퍼티의 사용 방법은 멤버 프로퍼티를 사용하는 방법과 같음

```java
StringUtilKt.getLastChar("Java");
```
- 자바에서 사용하고 싶은 경우 게터나 세터를 명시적으로 호출해야 한다.


  





  
