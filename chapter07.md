# 7. 연산자 오버로딩과 기타 관례


- 관례 : 어떤 언어의 기능과 미리 정해진 이름의 함수를 연결해주는 기법



### 7.1 산술 연산자 오버로딩


 자바는 원시타입에서만 산술연산자를 사용할 수 있고, String에 대해서는 + 사용이 가능하지만 코틀린은 더 다양한 타입에 사용이 가능하다.

#### 7.1.1 이항 산술 연산 오버로딩


- 정해진 함수 이름 앞에 operator 키워드를 붙여서 선언하면 기호로 해당 함수를 호출할 수 있다.
- 직접 정의한 함수가 호출되도 연산자 우선순위는 만족한다.(예 - 곱셈은 덧셈보다 먼저 수행됨)
  
  - 오버로딩 가능한 이항 산술 연산자<br>
  
    | 식 | 함수 이름 |
    |---|---|
    | a * b | times | 
    | a / b | div |
    | a % b | mod (1.1 부터는 rem) |
    | a + b | plus |
    | a - b | minus | 
  
```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

fun main(args: Array<String>) {
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)
    println(p1 + p2)  // 함수 plus가 호출됨
}
```

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.plus(other: Point): Point {  // 확장 함수로도 선언 가능
    return Point(x + other.x, y + other.y)
}

fun main(args: Array<String>) {
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)
    println(p1 + p2)
}
```



> **연산자 함수와 자바**<br><br>
> 자바를 코틀린에서 호출하는 경우에는 operator 변경자를 사용하거나 연산자에 따로 표시를 할 수 없다.<br>
> 따라서 이름과 파라미터 갯수만 문제가 되는데 자바 클래스에 원하는 기능이 있다면 코틀린에서 관례에 맞는 이름을 확장 함수로 선언하고 연산은 자바 메소드에 위임한다.



- 두 피연산자의 타입이 같지 않아도 연산이 가능하다.
    - 하지만 교환법칙이 성립하진 않음을 유의
    - p * 1.5 는 Point.times 로 선언하지만 1.5 * p 로 사용하려면 Double.times로 선언해야한다.
 

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.times(scale: Double): Point {
    return Point((x * scale).toInt(), (y * scale).toInt())
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(p * 1.5)
}

```


- 연산자의 반환 타입이 꼭 피연산자 중 하나와 일치하지 않아도 가능


```kotlin
operator fun Char.times(count: Int): String {
    return toString().repeat(count)
}

fun main(args: Array<String>) {
    println('a' * 3)
}


```



#### 7.1.2 복합 대입 연산자 오버로딩


- 코틀린은 plus 연산자를 오버로딩하면 관련 연산자인 += 도 자동으로 함께 지원한다.
- +=, -= 등의 연산자를 복합 대입 연산자라 한다.


```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}

fun main(args: Array<String>) {
    var point = Point(1, 2)
    point += Point(3, 4)
    println(point)
}

```


- 객체를 다른 참조로 바꾸기 보다 참조하는 객체 내부 상태를 변경하고 싶을 때: 반환 타입이 Unit 인 plusAssign 함수를 정의하면 코틀린은 해당 함수를 사용한다.
  - 대신 plus 와 plusAssign을 혼용하면 오류가 난다.

```kotlin


fun main(args: Array<String>) {
    var point = Point(1, 2)
    point += Point(3, 4)   // 내부적으로는 MutableCollection.plusAssign 가 호출 됨
    println(point)
}

```

#### 7.1.3 단항 연산자 오버로딩


- 이항 연산자와 사용 방법 같음

  | 식 | 함수 이름 |
  |---|---|
  | +a | unaryPlus | 
  | -a | unaryMinus |
  |  !a| not |
  | ++a, a++| inc |
  |  --a, a-- | dec | 
  

```kotlin

operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(-p)
}


```


### 7.2 비교 연산자 오버로딩



