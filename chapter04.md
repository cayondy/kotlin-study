## 4장 클래스, 객체, 인터페이스


 코틀린 선언은 기본적으로 final 이며 public 이다.

### 4.1 클래스 계층 정의 

#### 4.1.1 코틀린 인터페이스

- 코틀린 인터페이스는 자바8 인터페이스와 비슷하다.
- 코틀린 인터페이스 안에는 추상메소드 뿐 아니라 구현이 있는 메소드도 정의할 수 있다. (자바 default 메소드와 비슷)
- 다만 인터페이스에 어떤 필드도 들어갈 수 없다.

``` kotlin
interface Clickable {
    fun click()
}
class Button : Clickable {
    override fun click() = println("I was clicked")
}

```
- 코틀린에서는 콜론(:)을 붙이고 클래스나 인터페이스 이름을 적어서 클래스 확장과 인터페이스 구현을 한다.


**override**

- 코틀린에서 오버라이드 하는 경우엔 꼭 override 변경자를 사용
- 상위 클래스에 있는 메소드와 이름이 겹치는 경우 컴파일이 안되기 때문
- override를 붙이거나 메소드 이름을 변경해서 해결


**default 구현**

- 메소드 시그니처 뒤에 메소드 본문을 추가하면 끝

  > 자바에서는 메소드 앞에 default를 명시

``` kotlin
class Button : Clickable, Focusable {
    override fun click() = println("I was clicked")

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}

interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}

interface Focusable {
    fun setFocus(b: Boolean) =
        println("I ${if (b) "got" else "lost"} focus.")

    fun showOff() = println("I'm focusable!")
}

```


**상속한 인터페이스의 메소드 구현**

- super<상위타입 이름>.메소드 형태로 구현할 멤버 메소드를 지정할 수 있다.
  
    - 자바에서는 상위타입이름.super.메소드 형태로 사용


> **자바에서 코틀린의 메소드가 있는 인터페이스 구현하기**
> 
> 코틀린은 자바 6과 호환되기 때문에 인터페이스의 디폴드 메소드를 지원하지 않는다.<br>
> 따라서 코틀린의 디폴트 메소드를 일반 인터페이스와 디폴트 메소드 구현이 정적 메소드로 들어있는 클래스를 조합해 구현한다.



#### 4.1.2 open, final, abstract 변경자: 기본적으로 final

- 어떤 클래스의 상속을 허용하려면 open 변경자를 붙여야 한다.
- 메소드에 final을 붙이면 오버라이드를 금지할 수 있다.
- override한 메소드는 기본적으로 open 되어 있다.
- override한 메소드 앞에 final을 붙이면 오버라이드를 금지할 수 있다.

>**열린 클래스와 스마트 캐스트**
>
>


