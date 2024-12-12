## 5장 람다로 프로그래밍




### 5.2 컬렉션 함수형 API

#### 5.2.1 필수적인 함수: filter와 map

**filter 함수**

: 컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 람다가 true를 반환하는 원소만 모은다.
- 원치않는 원소를 제거할 때 사용


```kotlin
 val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.filter { it.age > 30 })
```

**map 함수**

: 주어진 람다를 각 원소에 적용해 새 컬렉션을 만든다.

```kotlin
   val people = listOf(Person("Alice", 29), Person("Bob", 31))
    println(people.map { it.name })

    println(people.map(Person::name))
```

* 계산을 반복하지 않도록 주의!
```kotlin
 val people = listOf(Person("Alice", 29), Person("Bob", 31))
    people.filter { it.age == people.maxBy(Person::Age)!!.age }

    val maxAge = people.maxBy(Person::Age)!!.age
    people.filter  { it.age == maxAge }
```

* 맵에서 사용할 때  
  - filterKeys, mapKeys : 키를 걸러내거나 변환
  - filyerValues, mapValues : 값을 걸러내거나 변환
 
```kotlin
   val numbers = mapOf(0 to "zero", 1 to "one")
    println(numbers.mapValues { it.value.toUpperCase() })

```

> 고차함수<br>
> : 람다나 다른 함수를 인자로 받거나 함수를 반환하는 함수


### 5.2.2 all, any, count, find: 컬렉션에 술어 적용

: 컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단하는 연산

**find 함수** 
: 조건을 만족하는 첫 번째 원소를 반환

**all 함수**
: 모든 원소가 조건을 만족하면 true

**any 함수**
: 조건을 만족하는 원소가 하나라도 있으면 true

**count 함수**
: 조건을 만족하는 원소의 갯수

```kotlin
   val people = listOf(Person("Alice", 27), Person("Bob", 31))
    println(people.find(canBeInClub27))
    println(people.all(canBeInClub27))
    println(people.count(canBeInClub27))

   val list = listOf(1, 2, 3)
    println(!list.all { it == 3 })
    println(list.any { it != 3 })

```
- all이나 any 앞에 !로 부정하는 것보다 술어에 부정을 하는 것을 권장
- size 함수로 컬렉션의 크기를 구할 경우 filter를 만족하는 컬렉션이 생기기 때문에 비효율적
  - println(people.count(canBeInClub27).size)
 

### 5.2.3 groupBy: 리스트를 여러 그룹으로 이뤄진 맵으로 변경

**groupBy 함수**
: 컬렉션의 원소를 구분하는 특성이 키, 키값에 따른 각 그룹이 값인 맵을 결과로 반환

```kotlin
   val people = listOf(Person("Alice", 31),
            Person("Bob", 29), Person("Carol", 31))
    println(people.groupBy { it.age })

```


### 5.2.4 flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

**flatMap 함수**
: 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 반환

```kotlin
  val strings = listOf("abc", "def")
    println(strings.flatMap { it.toList() })
    
   val books = listOf(Book("Thursday Next", listOf("Jasper Fforde")),
                       Book("Mort", listOf("Terry Pratchett")),
                       Book("Good Omens", listOf("Terry Pratchett",
                                                 "Neil Gaiman")))
    println(books.flatMap { it.authors }.toSet())

```
* toList() : 문자열에 속한 모든 문자로 이뤄진 리스트 반환
* toSet() : 리스트의 중복을 제거한 집합 반환




## 5.3 지연 계산 컬렉션 연산

- 시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.

```kotlin
   people.map(Person::name).filter{ it.startsWith("A") }

```
- map과 filter는 리스트를 반환하므로 연쇄 호출이 2개의 리스트를 만든다.
- 시퀀스를 사용하면 중간 결과를 저장하는 컬렉션이 생기지 않는다.
- Sequence 안에는 iterator 메소드 하나 밖에 없다.


### 5.3.1 시퀀스 연산 실행: 중간 연산과 최종 연산

- 시퀀스 연산은 중간 연산과 최종 연산으로 나뉜다.
- 중간 연산의 시퀀스는 최초 시퀀스의 원소를 변환하는 방법을 안다
- 최종 연산은 결과를 반환한다.
- 결과는 중간 연산의 시퀀스에서 계산을 수행해서 얻을 수 있는 컬렉션이나 원소, 숫자 또는 객체다.


```kotlin
   listOf(1, 2, 3, 4).asSequence()
            .map { print("map($it) "); it * it }
            .filter { print("filter($it) "); it % 2 == 0 }

```
- 최종 연산이 없는 코드를 실행하면 아무 내용도 출력되지 않는다.


```kotlin
    listOf(1, 2, 3, 4).asSequence()
            .map { print("map($it) "); it * it }
            .filter { print("filter($it) "); it % 2 == 0 }
            .toList()
```
- 최종 연산을 호출하면 연기된 모든 계산이 수행된다.


> * 컬렉션에서의 map 과 filter는 첫 연산을 먼저 수행하고 새 시퀀스를 얻고 그 시퀀스에 대해 다시 연산을 수행하지만,<br>
> 시퀀스에서는 각 원소의 연산이 순차적으로 진행된다.


**자바 스트림과 코틀린 시퀀스 비교**

| 특징 | 자바 스트림 | 코틀린 시퀀스 |
|---|---|---|
| 평가 방식	| 게으른 평가 (중간 연산)	| 게으른 평가| 
| 한 번 사용 가능 여부	| 일회용	| 여러 번 사용 가능| 
| 병렬 처리 |	기본 제공 (parallelStream())	| 별도의 코루틴 사용 필요| 
| 컬렉션 변환| 	새로운 컬렉션으로 반환	| toList() 등으로 변환 가능 |
| 동시성 지원| 	병렬 스트림	| 코루틴 기반 동시성 |



### 5.3.2 시퀀스 만들기

**generateSequence 함수**
: 이전의 원소를 인자로 받아 다음 원소를 계산한다.

```kotlin
fun main(args: Array<String>) {
    // 0부터 100까지 자연수의 합
    val naturalNumbers = generateSequence(0) { it + 1 }
    val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
    println(numbersTo100.sum())
}

```

```kotlin
// 상위 디렉토리에 hidden 속성의 디렉토리가 있는지 검사
fun File.isInsideHiddenDirectory() =
        generateSequence(this) { it.parentFile }.any { it.isHidden }

fun main(args: Array<String>) {
    val file = File("/Users/svtk/.HiddenDir/a.txt")
    println(file.isInsideHiddenDirectory())
}

```



## 5.4 자바 함수형 인터페이스 활용


### 5.4.1 자바 메소드에 람다를 인자로 전달

- 함수형 인터페이스를 인자로 원하는 자바 메소드에 코틀린 람다를 전달할 수 있다.

```kotlin
/* 자바 */
void postponeComputation(int delay, Runnable computation);

postponeComputation(1000) { println(42) }

```
- 컴파일러는 람다를 Runnable 인스턴스로 변환해준다.
  - Runnable 인스턴스는 Runnable을 구현한 무명 클래스와 인스턴스 를 뜻한다.
 

### 5.4.2 SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경

**SAM 생성자**
: 람다를 함수형 인터페이스의 인스턴스로 변환할 수 있게 컴파일러가 자동으로 생성한 함수
- (Single Abstract Method: 단일 추상 메소드)

- 컴파일러가 자동으로 람다를 함수형 인터페이스 무명 클래스로 바꾸지 못하는 경우 사용

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!") }
}

fun main(args: Array<String>) {
    createAllDoneRunnable().run()
}

```

- SAM 생성자의 이름은 함수형 인터페이스의 이름과 같다.
- 유일한 추상 메소드의 본문에 사용할 람다만을 인자로 받아서 함수형 인터페이스를 구현하는 클래스의 인스턴스를 반환한다.




## 5.5 수신 객체 지정 람다: with와 apply

- 코틀린의 람다에서는 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있다.

### 5.5.1 with 함수

**with 함수**
: 어떤 객체의 이름을 반복하지 않고도 그 객체에 대해 다양한 연산을 수행할 수 있게 한다.


```kotlin
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
         result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}

fun main(args: Array<String>) {
    println(alphabet())
}

```
- result가 여러번 반복됨


```kotlin
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {  // 메소드를 호출하려는 수신 객체를 지정
        for (letter in 'A'..'Z') {
            this.append(letter)  // "this"를 명시해서 앞에서 지정한 수신 객체의 메소드를 호출
        }
        append("\nNow I know the alphabet!")  // "this"를 생략하고 메소드를 호출
        this.toString()
    }
}

fun main(args: Array<String>) {
    println(alphabet())  // 람다에서 값을 반환
}
```

- with 함수는 파라미터가 2개인 함수이다.
  - (위에서는 첫 번째 인자가 stringBuilder, 두 번째 인자가 람다)
- 첫 번째 인자로 받은 객체를 두 번째 받은 인자(람다)의 수신 객체로 만든다.
- 람다 본문에서는 this로 수신 객체에 접근할 수 있다.
  - (this. 을 생략하고 프로퍼티나 메소드 이름만 사용해도 수신객체의 멤버에 접근할 수 있다.)
 

> **수신 객체 지정 람다와 확장 함수 비교**
>
> 확장함수 안에서 this는 그 함수가 확장하는 타입의 인스턴스를 가리킨다.<br>
> 그리고 this. 을 생략하고 프로퍼티나 메소드 이름만 사용해도 수신객체의 멤버에 접근할 수 있다.
>
> 어떤 의미에서는 확장함수흫 수신 객체 지정 함수라 할 수도 있다.
>
> | 일반 함수 | 일반 람다 | 
> |---|---|
> | 확장함수| 수신 객체 지정 람다	|


```kotlin
fun alphabet() = with(StringBuilder()) {  // StringBuilder 인스턴스 생성
    for (letter in 'A'..'Z') {
        append(letter)  // 람다에서 수신 객체 인스턴스 참조
    }
    append("\nNow I know the alphabet!")
    toString()
}

fun main(args: Array<String>) {
    println(alphabet())  // alphabet 함수가 결과를 바로 반환
}
```


### 5.5.2 apply 함수
**apply 함수**


  
