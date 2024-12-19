## 6장 코틀린 타입 시스템


### 6.1 널 가능성

#### 6.1.1 널이 될 수 있는 타입

- 코틀린에서는 널이 될 수 있는 타입을 뒤에 물음표(?)를 붙여 정의할 수 있다.
- ex) fun strLenSafe(s: String?) = ...

#### 6.1.2 타입의 의미

- 자바에서는 null이 들어있는 경우 사용할 수 없는 연산이 많지 않다.
  - @Nullable 이나 @NotNull과 같은 어노테이션을 활용할 수도 있지만 모든 코드에 어노테이션을 적용하긴 쉽지않다.


#### 6.1.3 안전한 호출 연산자: ?.

- ?.는 null 검사와 메소드 호출을 한 번의 연산으로 수행한다.
- 호출하려는 값이 null이 아니라면 일반 메소드처럼 동작하고, null이라면 호출을 무시하고 null이 반환된다.
  - 안전한 호출의 결과 타입은 널이 될 수 있는 타입이 된다.


```kotlin
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.toUpperCase()
    println(allCaps)
}

fun main(args: Array<String>) {
    printAllCaps("abc")
    printAllCaps(null)
}

```

- 메소드 뿐 아니라 프로퍼티를 읽고 쓸 때도 사용 가능하다.
- 여러번 연쇄적으로 사용 가능하다.

  
```kotlin
class Address(val streetAddress: String, val zipCode: Int,
              val city: String, val country: String)

class Company(val name: String, val address: Address?)

class Person(val name: String, val company: Company?)

fun Person.countryName(): String {
   val country = this.company?.address?.country
   return if (country != null) country else "Unknown"
}

fun main(args: Array<String>) {
    val person = Person("Dmitry", null)
    println(person.countryName())
}

```



#### 6.1.4 엘비스 연산자: ?:

- null 대신 사용할 디폴트 값을 지정할 때 사용 가능하다.
- 이항 연산자로 좌항을 계산한 값이 널인지 검사한다.
- 좌항이 널이면 우항 값을 결과로 한다.
  - 코틀린에서는 return이나 throw 등의 연산도 식이므로 우항에 넣을 수 있음


``` kotlin
// 안전한 호출 연산자와 함께 사용해 객체가 널인 경우에 대비한 값을 지정함
fun strLenSafe(s: String?): Int = s?.length ?: 0 

fun main(args: Array<String>) {
    println(strLenSafe("abc"))
    println(strLenSafe(null))
}

```

#### 6.1.5 안전한 캐스트: as?

- 자바의 타입 캐스트와 마찬가지로 as로 지정한 타입으로 바꿀 수 없으면 ClassCastException이 발생한다.
  - is로 미리 as로 변환 가능한 지 검사할 수는 있다.
- as? 연산자는 지정한 타입으로 캐스트하고, 변환할 수 없으면 null을 반환한다.
  - equals를 구현할 때 유용하다.


``` kotlin
class Person(val firstName: String, val lastName: String) {
   override fun equals(o: Any?): Boolean {
      val otherPerson = o as? Person ?: return false // 타입이 일치하지 않으면 false 반환

      return otherPerson.firstName == firstName && // 안전한 캐스트 후에는 스마트 캐스트 된다.
             otherPerson.lastName == lastName
   }

   override fun hashCode(): Int =
      firstName.hashCode() * 37 + lastName.hashCode()
}

fun main(args: Array<String>) {
    val p1 = Person("Dmitry", "Jemerov")
    val p2 = Person("Dmitry", "Jemerov")
    println(p1 == p2) // equals 호출
    println(p1.equals(42))
}

```


#### 6.1.6 넘 아님 단언: !!

- 널 아님 단언은 널이 될 수 있는 인자를 널이 될 수 없는 타입으로 변환한다.

``` kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!! // 예외는 이 위치를 가리킨다.
    println(sNotNull.length)
}

fun main(args: Array<String>) {
    ignoreNulls(null)
}


```
  - s가 널이면 예외를 던짐


#### 6.1.7 let 함수

- 자신의 수신객체를 인자로 받은 람다에게 넘긴다.
  
* 안전한 호출과 함께 사용하면
- let을 활용하면 널이 될 수 있는 값을 널이 아닌 값만 인자로 받는 함수에 넘길 수 있다.
- null인 경우는 아무 일도 일어나지 않는다.

```kotlin
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main(args: Array<String>) {
    var email: String? = "yole@example.com"
    email?.let { sendEmailTo(it) } // sendEmailTo에게 it을 넘긴다.
    email = null
    email?.let { sendEmailTo(it) } // email = null이기 때문에 sendEmailTo 를 호출하지 않음
}

```


#### 6.1.8 나중에 초기화할 프로퍼티

- lateinit 변경자를 붙여서 사용한다.
- 나중에 초기화할 프로퍼티는 항상 **var**여야 한다! (val은 final 로 선언되니까)
- 널이 될 수 없는 프로퍼티가 생성자 안에서 널이 아닌 값으로 초기화할 방법이 없는 경우에 사용하면 편하다.
   - 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화 해야한다.

- 프로퍼티를 초기화 하기 전에 접근하면 "lateinit property *** has not been initialized" 예외를 발생시킨다.
    - NPE보다 명확하다.


```kotlin
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    private lateinit var myService: MyService

    @Before fun setUp() {
        myService = MyService()
    }

    @Test fun testAction() {
        Assert.assertEquals("foo",
            myService.performAction())
    }
}

```

     
#### 6.1.9 널이 될 수 있는 타입 확장

- 널이 될 수 있는 타입에 대한 확장 함수를 정의해서 null 값을 다루는 도구로 사용할 수 있다.
  - String? 타입의 수신 객체에 호출 가능한 메소드
    
    ***isNullOrEmpty**
    : 명시적으로 null을 검사해서 null인 경우 true를 반환, 아닌 경우 isEmpty를 호출
    
    ***isNullOrBlank** 
    : 명시적으로 null을 검사해서 null인 경우 true를 반환, 아닌 경우 isBlank를 호출

```kotlin
fun verifyUserInput(input: String?) {
    if (input.isNullOrBlank()) {  // 안전한 호출 안해도됨
        println("Please fill in the required fields")
    }
}

fun main(args: Array<String>) {
    verifyUserInput(" ")
    verifyUserInput(null)
}

```

***자바와 다른 점**

: 자바에서는 수신 객체가 널이라면 NPE가 발생하기 때문에 메소드 안의 this는 항상 널이 아니지만
코틀린에서는 널이 될 수 있는 String 을 확장한 함수 안에서 this가 널이 될 수 있다.


```kotlin
fun String?.isNullOrBlank(): Boolean =  // 널이 될 수 있는 String 을 확장
  this == null || this.isBlank()  // 두번째 this에는 스마트캐스트가 적용된다. (?)
```


#### 6.1.10 타입 파리미터의 널 가능성

- 코틀린에서는 함수나 클래스의 모든 타입 파라미터는 널이 될 수 있다.
- 타입 파라미터 이름 뒤에 ? 가 없어도 널이 될 수 있는 타입이 된다.
  - 타입 파라미터를 Any? 로 추론하기 때문


```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode())  // t는 널이 될 수 있으므로 안전한 호출 사용
}

fun main(args: Array<String>) {
    printHashCode(null)
}

```


#### 6.1.11 널 가능성과 자바

- 코틀린은 자바 상호운용성을 강조하는 언어이지만 자바의 타입 시스템은 널 가능성을 지원하지 않는다.
- 하지만 자바에도 어노테이션으로 정보를 줄 수 있다.
    - @Nullable String 은 코틀린에서 String? 과 같고 @NotNull String 은 String 과 같다.

* 어노테이션이 없는 경우는??
  
  ***플랫폼 타입**

  : 플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다.
  
  - 말 그대로 널이 될 수 있는 타입으로 처리하거나 널이 될 수 없는 타입으로도 처리 할 수 있다.
  - 플랫폼 타입 연산의 모든 책임을 개발자에게 넘긴다.
    - 필요에 따라 널 가능성이 있는 지 없는지 검사하고 처리해야 한다는 뜻


  ```kotlin
  fun yellAtSafe(person: Person) {
      println((person.name ?: "Anyone").toUpperCase() + "!!!") // 엘비스 연산자로 안전한 호출
  }
  
  fun main(args: Array<String>) {
      yellAtSafe(Person(null))
  }

  ```
  - 코틀린 컴파일러는 public 함수의 널이 아닌 타입인 파라미터와 수신 객체에 널 검사를 추가해준다.
  - 이런 파라미터 값 검사는 파라미터 사용 시점이 아닌 **함수 호출 시점**에 이뤄지므로 예외 발생시 원인 파악하기 편리하다.
 
  - 오류를 피하려면 자바 메소드 문서를 보고 널을 반환하는 지에 따라 널 검사를 추가해야한다.



  > **코틀린이 플랫폼 타입을 도입한 이유**
  >
  > ArrayList<String> 을 ArrayList<String?>? 처럼 다루면 검사에 많은 비용이 들기 때문에 자바 타입을 가져온 경우 가져온 프로그래머에게 그 타입을 처리할 책임을 부여하는 방법을 선택했다.<br>
  > 코틀린에서 플랫폼 타입을 선언할 수는 없지만 코틀린 컴파일러에서 타입뒤에 !를 붙여 표기하면 널 가능성에 대한 정보가 없다는 의미로 플랫폼 타입임을 알 수 있다.


**상속**

- 코틀린에서 자바 메소드를 오버라이드 하는 경우 그 메소드의 널 타입을 어떻게 선언할 지 결정해야 한다.
    - 널이 될 수 있는 타입, 없는 타입 모두 가능하다.
- 코틀린에서는 널이 될 수 없는 타입으로 선언한 모든 파라미터에 대해 널이 아님을 검사하는 단언문을 만들어준다.
