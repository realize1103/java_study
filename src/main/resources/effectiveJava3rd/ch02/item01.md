##Consider static factory methods instead of constructors(생성자 대신 정적 팩터리 메서드를 고려하라)

---
### 요점
습관처럼 우리는 pulbic 생성자를 이용한 프로그래밍을 해왔지만약, 이 static factory method의 장단점을 알아보고제 상황에 맞 고려하여 사용해보자.
정적 팩터리 메서드를 대신 써라라는 말은 아니다.
---

### 장점 
#### 1. They have names.(이름을 가질 수 있다.)
 
 생성자에서 제공하는 파라미터가 반환하는 객체를 잘 설명하지 못하는 경우에, 잘 만든 이름을 가진 정적 팩토리 메서드를 사용하는 것이 더 쉽게 읽힌다. 예를들어, BigInteger(int, int,Random)보다 BigInteger.probablePrime이 더 읽기 쉽다.
 또한, 클래스는 단일 생성장자만 가질 수 있다. 동일한 시그니쳐를 가진 생성자를 생성할 수 없고, 매개변수 유형의 순서에 따라서 다른 두개의 생성자를 제공하는 나쁜 방식을 사용할 수는 있으나 좋은 생각이 아니다.
 위 같은 경우 실수로 잘못된 생성자를 호출할 수 있기 때문이다. 이럴경우에 유용하다.
 
 
```java
public class Foo {

    String name;
    String address;

    public Foo() {
    }

    public Foo(String name) {
        this.name = name;
    }
    /* 이렇게 사용할 수 없음.
    public Foo(String address) {
        this.name = name;
    }
    */

    public static Foo withName(String name) {
        return new Foo(name);
    }
    
    //같은 타입의 매개변수를 만들 수 없었지만, 정적 팩토리 메서드는 아래처럼 가능하다.
    public static Foo withAddress(String address) {
        Foo foo = new Foo();
        foo.address = address;
        return foo;
    }

    public static void main(String[] args) {
        //일반적으로 사용되는 public 생성
        Foo foo = new Foo("kenneth"); //한눈에 이해되기 어렵다.
        Foo foo1 = Foo.withName("kenneth"); //정적 팩토리 방식은 파라미터와 메서드 명을 토대로 이해되기 더 쉽다.

    }
} 
```
 
#### 2. They are not required to create a new object each time they’re invoked. (매번 객체를 만들 필요가 없다.)
This allows immutable classes (Item 17) to use preconstructed instances, or to cache instances as they’re constructed, and dispense them repeatedly to avoid creating unnecessary duplicate objects.  
불변 클래스인 경우나 매번 새로운 객체를 만들 필요가 없는 경우에 미리 만들어둔 인스턴스 혹은 캐시해둔 인스턴스를 반환할 수 있다.  
아래처럼 매번 새로운 Boolean 객체를 생성하지 않고 반환가능하다.
```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
} 
```
#### 3. They can return an object of any subtype of their return type.(반환 타입의 하위 타입 인스턴스를 반환 할 수도 있다.)
이러한 방식은 매우 유연성을 가진다. 그 중 한가지는 API가 그들의 클래스를 공개하지 않고 객체를 반환할 수 있다는 것이다.
JAVA 8이전에는 Interface는 static method를 가질수 없엇지만, 8 이후 가능해졌고, 9부터는 private static method도 허용되었다.
예를들어, Collections은 45개의 구현체의 인스턴스를 제공하지만 이는 모두 공개되지 않는다. 프로그래머가 API를 사용할때 알아야할 개념의 무게까지 줄일 수 있다.


```java
public class Foo {

    public static Foo getFoo(boolean flag) {
        return flag ? new Foo() : new BarFoo();
    }

    public static void main(String[] args) {
        //반환 타입이 Foo이지만 flag에 따라서 BarFoo 즉 하위 리턴 타입을 반환 할 수 있다.
        Foo foo = Foo.getFoo(false);
    }    
    static class BarFoo extends Foo{
    }

} 
```
#### 4. The class of the returned object can vary from call to call as a function of the input parameters(입력 매개 변수에 따라 반환되는 개체의 클래스는 호출마다 다를 수 있다.)
장점 3과 같은 이유로 반환되는 객체의 타입이 다를 수 잇다. 예를 들어 EnumSet 클래스는 public 생성자가 없고, 정적 팩토리만 제공하는데 리턴하는 갯수가 64개 이하이면 RegualrEnumSet을 65개 이상이면 JumboEnumSet을 반환한다.
이런한 객체타입은 숨겨져 있기 때문에 우리가 알 이유는 없다.
 
#### 5. The class of the returned object need not exist when the class containing the method is written.(반환할 객체의 클래스는 정적 팩터리 메서드를 작성하는 시점에는 존재하지 않아도 된다.)
이러한 유연한 정적 팩토리 메서드는 Java Database Connectivity API(JDBC)와 같은 service provider frameworks를 기초로 되어있다.
서비스 프로바이더 프레임워크는 제고아자가 서비스를 구현하는 시스템이고, 시스템은 클라이언트와 구현체를 분리하여 클라이언트가 구현할 수 있도록 만든다.

```java
public class Foo {

    public static Foo getFoo(boolean flag) {
        Foo foo = new Foo();
        //foo에 다른 Foo구현체의 인스턴스를 대입하도록 코드를 작성할 수 있다.
        // 이때 현재 getFoo메서드 작성시에 다른 class가 존재하지 않아도 된다.
        return foo;
    }

    public static void main(String[] args) {
        //반환 타입이 Foo이지만 flag에 따라서 BarFoo 즉 하위 리턴 타입을 반환 할 수 있다.
        Foo foo = Foo.getFoo(false);
    }    

} 
```
### 단점
#### 1. The main limitation of providing only static factory methods is that classes without public or protected constructors cannot be subclassed.(public 또는 protected 생성자없이 정적 팩터리 매서드만 제공하면 하위 클래스를 만들 수 없다.)
앞서 예로 들은 `java.util.Collections`처럼 편의성 구현체는 상송할 수 없다. 불변타입인 경우나 컴포지션을 장려하기 때문에 오히려 장점일 수 있다.
#### 2. Static factory methods is that they are hard for programmers to find.(정적 팩토리 메서드는 프로그래머가 찾기 어렵다.)
공통 명명 규칙을 준수하여 메소드에 대한 문서를 제공하는것이 좋다.