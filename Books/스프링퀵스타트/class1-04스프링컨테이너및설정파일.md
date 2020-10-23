# 04 의존성 주입

# 4.1 의존성 관리
## 4.1.1 스프링의 의존성 관리 방법
- 스프링 프레임워크의 가장 중요한 특징: `객체의 생성과 의존관계`를 컨테이너가 자동으로 관리한다. <br />
이는 `스프링 IoC(제어의 역행)의 핵심 원리`이다. <br />

- 스프링은 IoC를 다음 두 가지 형태로 지원한다.

![그림4-1스프링의IoC](https://user-images.githubusercontent.com/40673012/96664698-565ab100-138e-11eb-96ec-495202e4d68f.png)

### Dependency Lookup
- 컨테이너가 애플리케이션 `운용에 필요한 객체를 생성`하고, <br />
클라이언트는 컨테이너가 생성한 객체를 검색(Lookup)하여 사용하는 방식이다.
- `실제 애플리케이션 개발 과정`에서는 사용하지 않으며, 대부분 Dependency Injection을 사용하여 개발한다.

### Dependency Injection
- `객체 사이의 의존관계`를 `스프링 설정 파`일에 등록된 정보를 바탕으로 `컨테이너`가 `자동`으로 처리한다.
    - 따라서 `의존성 설정`을 바꾸고 싶을 때 프로그램 코드를 수정하지 않고, 
    `스프링 설정 파일` 수정만으로 변경사항을 적용할 수 있어 `유지보수`가 향상된다.
- Dependency Injection은 `Setter 메소드`를 기반으로하는 `세터 인젝션`과 `생성자`를 기반으로 하는 `생성자 인젝션(Constructor Injection)`으로 나뉜다.

## 4.1.2 의존성 관계
- `의존성(Dependency)관계`란 `객체와 객체의 결합 관계`다.  <br />
하나의 객체에서 다른 객체의 변수나 메소드를 이용하려면 이용하려는 객체에 대한 `객체 생성`과 생성된 `객체의 레퍼런스 정보`가 필요하다. 

- 위 그림에서 `SamsungTV`는 `SonySpeaker의 메소드`를 이용해 기능을 수행한다. <br />
따라서 SamsungTV는 SonySpeaker 타입의 speaker 변수를 멤버변수로 가지고 있고, <br />
speaker 변수는 SonySpeaker 객체를 참조해야 한다. <br />
따라서 SamsungTV 클래스 어딘가에는 SonySpeaker 클래스에 대한 `객체 생성 코드`가 반드시 필요하다.

![그림4-2 TV 컴포넌트 클래스 다이어그램](https://user-images.githubusercontent.com/40673012/96669537-65def780-1398-11eb-9ae0-3f6c883cc42f.png)

```java
public class SamsungTV implements TV{
    private  SonySpeaker speaker;
    public SamsungTV() {
        System.out.println("===> SamsiungTV 객체 생성");
    }
    
    public void powerOn() {
        System.out.println("SumsungTV--- 전원 켠다.");
    }
    public void powerOff() {
        System.out.println("SumsungTV--- 전원 끊다.");
    }
    // 클라이언트가 volumeUp(),volumeDown() 중 어떤 메소드를 먼저 호출할지 모르기 떄문에
    // 두 메소드 모두에서 객체 생성
    public void volumeUp() {
        speaker = new SonySpeaker();
        speaker.volumeUp();
    }
    public void volumeDown() {
        speaker = new SonySpeaker();
        speaker.volumeDown();
    }
}
```
```java
public class SonySpeaker {
    public SonySpeaker() {
            System.out.println("===> SonySpeaker 객체 생성");
    }
    public void volumeUp() {
        System.out.println("SumsungTV--- 소리 올린다.");
    }
    public void volumeDown() {
        System.out.println("SumsungTV--- 소리 내린다.");
    }
}
```
- 위 프로그램의 문제점: <br />
1. SonySpeaker 객체가 쓸데 없이 두 개 `생성`된다. <br />
2. 운영 과정에서 SonySpeaker의 성능이 떨어져서 SonySpeaker를 AppleSpeaker 같은 다른 Speaker로 변경하고자 할때, volumeUp(). volumeDown() 두 개의 메소드를 모두 `수정`해야 한다.

-문제의 원인: 의존관계에 있는 Speaker 객체에 대한 객체 생성 코드를 직접 SamsungTV 소스에 명시했기 때문이다.

# 4.2 생성자 인젝션 이용하기
- `스프링 컨테이너`는 `XML 설정 파일`에 등록된 클래스를 찾아 `객체를 생성`할 때 기본적으로 `매개변수 없는 Default 생성자`를 호출한다. <br />
- 하지만 `매개변수`를 가지는 다른 `생성자`를 호출하도록 설정할 수도 있으며, 이 기능으로 `생성자 인젝션`(Constructor Injection)을 처리한다.

```java
public class SamsungTV implements TV{
    private  SonySpeaker speaker;
    
    public SamsungTV() {
        System.out.println("===> SamsiungTV(1) 객체 생성");
    }
    
    public SamsungTV(SonySpeaker speaker) {
        System.out.println("===> SamsiungTV(2) 객체 생성");
        this.speaker = speaker;
    }
    
    public void powerOn() {
        System.out.println("SumsungTV--- 전원 켠다.");
    }
    public void powerOff() {
        System.out.println("SumsungTV--- 전원 끊다.");
    }
    public void volumeUp() {
        speaker.volumeUp();
    }
    public void volumeDown() {
        speaker.volumeDown();
    }
}
```

```XML
<bean id="tv" class="ch4_2.SamsungTV">
    <constructor-arg ref="sony"></constructor-arg>
</bean>

<bean id="sony" class="ch4_2.SonySpeaker"></bean>
```

- `생성자 인젝션`을 위해서는 SamsungTV 클래스 `<bean> 등록 설정`에서 시작 태그와 종료 태기 사이에 `<constructor-arg> 엘리먼트`를 추가한다. <br />
그리고 생성자 인자로 전달한 `객체의 아이디`를 <constructor-arg> 엘리먼트에 `ref 속성`으로 참조한다. <br />

- 실행 결과에서 알 수 있는 두 가지 사실:
1. SamsungTV 클래스의 객체가 생성될 때, 기본 생성자가 아닌 `두 번째 생성자`가 사용됐다. 
2. 스프링 설정 파일에 SonySpeaker가 SamsungTV 클래스 밑에 등록되었지만 `먼저 생성`된다.

실행결과
---
```
===> SonySpeaker 객체 생성
===> SamsiungTV(2) 객체 생성
SumsungTV--- 전원 켠다.
SumsungTV--- 전원 켠다.
SumsungTV--- 소리 올린다.
SumsungTV--- 소리 내린다.
SumsungTV--- 전원 끊다.
```

- 1. 스프링 컨테이너는 기본적으로 `bean 등록된 순서`대로 객체를 생성하며, <br />
2. 모든 객체는 `기본 생성자 호출`을 원칙 원칙으로 한다. <br />
3. 하지만 생성자 인젝션으로 `의존성 주입`될 객체가 먼저 생성된다.

## 4.2.1 다중 변수 매핑
-  생성자 인젝션에서 초기화해야 할 멤버변수가 여러 개이면, <br />
스프링 설정 파일에 `<constructor-arg> 엘리먼트`를 `매개변수의 개수`만큼 추가해야 한다.
```java
public class SamsungTV implements TV{
    private  SonySpeaker speaker;
    private int price;
    
    public SamsungTV() {
        System.out.println("===> SamsiungTV(1) 객체 생성");
    }
    
    public SamsungTV(SonySpeaker speaker) {
        System.out.println("===> SamsiungTV(2) 객체 생성");
        this.speaker = speaker;
    }
    
    public SamsungTV(SonySpeaker speaker, int price) {
        System.out.println("===> SamsiungTV(3) 객체 생성");
        this.speaker = speaker;
        this.price = price;
    }
    
    public void powerOn() {
        System.out.println("SumsungTV--- 전원 켠다. <가격 : " + price + ")");
    }
    
    public void powerOff() {
        System.out.println("SumsungTV--- 전원 끊다.");
    }
    public void volumeUp() {
        speaker.volumeUp();
    }
    public void volumeDown() {
        speaker.volumeDown();
    }
}
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="tv" class="ch4_2.SamsungTV">
        <constructor-arg ref="sony"></constructor-arg>
        <constructor-arg value="2700000"></constructor-arg>
    </bean>
    
    <bean id="sony" class="ch4_2.SonySpeaker"></bean>
</beans>
```

실행결과
---
```
===> SonySpeaker 객체 생성
===> SamsiungTV(3) 객체 생성
SumsungTV--- 전원 켠다. <가격 : 2700000)
SumsungTV--- 소리 올린다.
SumsungTV--- 소리 내린다.
SumsungTV--- 전원 끊다.
```

- 인자로 전달될 데이터가 `<bean>으로 등록된 다른 객체`일 때는 `ref 속성`을 이용하여 해당 객체의 아이디나 이름을 참조하지만,  <br />
고정된 문자열이나 정수 같은 `기본형 데이터`일 때는 `value 속성`을 사용한다. <br />

- 생성자가 여러 개 오버로딩 되면 어떤 생성자를 호출할 지 분명하지 않으므로, `index 속성`을 지원한다. <br />
index 속성을 이용하면 어떤 값이 몇 번째 매개변수로 매핑되는지 지정할 수 있다. <br />
index는 0 부터 시작한다.

```xml
<bean id="tv" class="ch4_2.SamsungTV">
    <constructor-arg index="0" ref="sony"></constructor-arg>
    <constructor-arg index="1" value="2700000"></constructor-arg>
</bean>
```

## 4.2.2 의존관계 변경
- 지금까지 SamsungTV 객체가 SonySpeaker를 이용하여 동작했지만, `유지보수` 과정에서 다른 스피커로 `교체`하는 상황도 발생할 수 있다. <br />
`의존성 주입`으로 이런 상황을  효과적으로 처리할 수 있다.<br />

1. Speaker 인터페이스 추가
```java
public interface Speaker {
    void volumeUp();
    void volumeDown();
}
```

2. Speaker 인터페이스를 구현한 AppleSpeaker 추가
```java
public class AppleSpeaker implements Speaker{
    public AppleSpeaker() {
        System.out.println("===> AppleSpeaker 객체 생성");
    }

    @Override
    public void volumeUp() {
        System.out.println("AppleSpeaker---소리 울린다.");
    }

    @Override
    public void volumeDown() {
        System.out.println("AppleSpeaker---소리 내린다.");
    }
}
```

3. SamsungTV 클래스의 멤버변수와 매개변수 타입을 모두 Speaker로 수정
```java
public class SamsungTV implements TV{
    private Speaker speaker;
    private int price;
    
    public SamsungTV() {
        System.out.println("===> SamsiungTV(1) 객체 생성");
    }
    
    public SamsungTV(Speaker speaker) {
        System.out.println("===> SamsiungTV(2) 객체 생성");
        this.speaker = speaker;
    }
    
    public SamsungTV(Speaker speaker, int price) {
        System.out.println("===> SamsiungTV(3) 객체 생성");
        this.speaker = speaker;
        this.price = price;
    }
    
    public void powerOn() {
        System.out.println("SumsungTV--- 전원 켠다. <가격 : " + price + ")");
    }
    
    public void powerOff() {
        System.out.println("SumsungTV--- 전원 끊다.");
    }
    public void volumeUp() {
        speaker.volumeUp();
    }
    public void volumeDown() {
        speaker.volumeDown();
    }
}
```

4. 스프링 설정 파일에 AppleSpeaker <bean> 등록 및 <constructor-arg> 엘리먼트의 속성값을 apple로 지정
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="tv" class="ch4_2_2.SamsungTV">
        <constructor-arg index="0" ref="apple"></constructor-arg>
        <constructor-arg index="1" value="2700000"></constructor-arg>
    </bean>
    
    <bean id="apple" class="ch4_2_2.AppleSpeaker"></bean>
</beans>
```

- 결론: `스프링 설정 파일`만 적절히 관리하면 어떤 `자바 코드 변경 없이` <br />
동작하는 TV도 변경할 수 있고, <br />
TV가 사용하는 스피커도 변경할 수 있다.

# 4.3 Setter 인젝션 이용하기
- Setter 인젝션은 Setter 메소드를 호출하여 의존성 주입을 한다. (생성자 인젝션은 생성자로...) <br />
Setter 인젝션, 생성자 인젝션 모두 `멤버변수를 원하는 값으로 설정`하는 것이 목적이고, 결과 같아 무엇을 쓰든 상관없다. <br />
하지만 코딩 컨벤션에 따라 `한 가지로 통일`해서 사용하며, 대부분은 `Setter 인젝션`을 사용하며, <br />
Setter 메소드가 제공되지 않는 클래스에 대해서만 `생성자 인젝션`을 사용한다. 


## 4.3.1 Setter 인젝션 기본 
- SamsungTV 클래스에 Setter 메소드를 추가한다.
- Setter 메소드는 `스프링 컨테이너`가 `자동`으로 호출하며, <br />
`호출하는 시점`은 <bean> 객체 생성 직후이다.<br />
따라서 Setter 인젝션이 동작하려면 Setter 메소드뿐만 아니라 `기본 생성자`도 반드시 필요하다.

```java
public class SamsungTV implements TV{
    private Speaker speaker;
    private int price;
    
    public SamsungTV() {
        System.out.println("===> SamsiungTV(1) 객체 생성");
    }
    
    public void setSpeaker(Speaker speaker) {
        System.out.println("===> setSpeaker() 호출");
        this.speaker = speaker;
    }
    
    public void setPrice(int price) {
        System.out.println("===> setPrice() 호출");
        this.price = price;
    }
    
    public void powerOn() {
        System.out.println("SumsungTV--- 전원 켠다. <가격 : " + price + ")");
    }
    
    public void powerOff() {
        System.out.println("SumsungTV--- 전원 끊다.");
    }
    public void volumeUp() {
        speaker.volumeUp();
    }
    public void volumeDown() {
        speaker.volumeDown();
    }
}
```
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="tv" class="ch4_3_1.SamsungTV">
        <property name="speaker" ref="apple"></property>
        <property name="price" value="2700000"></property>
    </bean>
    
    <bean id="apple" class="ch4_3_1.AppleSpeaker"></bean>
</beans>
```

- Setter 인젝션을 이용하려면 스프링 설정 파일에 <constructor-arg> 엘리먼트 대신 <br />
1. `<property> 엘리먼트`를 사용해야 한다. <br />
2. `name` 속성값은 호출하고자 하는 메소드 이름으로, <br />
3.  `호출할 메소드 이름`은 변수 이름에서 첫 글자를 대문자로 바꾸고 앞에 `set`을 붙인 것이다.  <br />
4. Setter 메소드를 호출하면서 `다른 <bean> 객체`를 인자로 넘기려면 `ref 속성`을 사용하고, <br />
5. `기본형 데이터`를 넘기려면 `value 속성`을 사용한다.


|Setter 메소드 이름|name 속성값|
|--|--|
|setSpeaker()|name="speaker"|
|setAddressList()|name="addressList"|
|setBoardDAO()|name="boardDAO"|

## 4.3.2 p 네임스페이스 사용하기
- Setter 인젝션을 처리할 때 `p 네임스페이스`를 이용하면 더 효율적으로 의존성 주입을 처리할 수 있다.
- p 네임스페이스는 네임스페이스에 대한 별도의 schemaLocation이 없다. 따라서 네임스페이스만 적절히 선언하고 사용할 수 있다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd"
    xmlns:p="http://www.springframework.org/schema/p"  >

</beans>
```

- 참조형 변수에 참조한 객체 할당 <br />
`p:변수명-ref="참조할 객체의 이름이나 아이디"`

- 기본형이나 문자형 변수에 직접 값을 설정 <br />
`p:변수명="설정할 값"`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd"
    xmlns:p="http://www.springframework.org/schema/p"  >

    <bean id="tv" class="ch4_3_1.SamsungTV" p:speaker-ref="sony" p:price="2700000"/>

    <bean id="sony" class="ch4_3_1.SonySpeaker"></bean>
    <bean id="apple" class="ch4_3_1.AppleSpeaker"></bean>
</beans>
```


# 4.4 컬렉션(Collection) 객체 설정
- 스프링에서는 배열이나 List 같은 컬렉션 객체를의 `의존성`을 주입하기 위한  <br />
`컬렉션 매핑`과 관련된 엘리먼트를 지원한다.

|컬렉션|엘리먼트|
|--|--|
|java.util.List, 배열|<list>|
|java.util.Set|<set>|
|java.util.Map|<map>|
|java.util.Properties|<props>|

## 4.4.1 List 타입 매핑
```java
package ch4_4_1;
import java.util.List;

public class CollectionBean {
    private List<String> addressList;
    
    public void setAddressList(List<String> addressList) {
        this.addressList = addressList;
    }
    public List<String> getAddressList(){
        return addressList;
    }
}
```
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd"  >

    <bean id="collectionBean" class="ch4_4_1.CollectionBean">
        <property name="addressList">
            <list>
                <value>서울시 강남구 역삼동</value>
                <value>서울시 성동구 행당동</value>
            </list>
        </property>
    </bean>
</beans>
```

- 위의 설정은 두 개의 문자열 주소가 저장된 List 객체를 CollectionBean 객체의 setAddressList() 메소드를 호출할 때, <br />
인자로 전달하여 addressList 멤버변수를 초기화하는 설정이다.

```java
public class CollectionBeanClient {
    public static void main(String[] args) {
        AbstractApplicationContext factory = new GenericXmlApplicationContext("application.xml");
        
        CollectionBean bean = (CollectionBean) factory.getBean("collectionBean");
        List<String> addressList = bean.getAddressList();
        for(String address : addressList) {
            System.out.println(address.toString());
        }
        factory.close();
    }
}
```

실행결과
---
```
서울시 강남구 역삼동
서울시 성동구 행당동
```

## 4.4.2 Set 타입 매핑
- 중복 값을 허용하지 않는 집합 객체를 사용할 때는 java.util.Set이라는 컬렉션을 사용한다. <br />
컬렉션 객체는 `<set> 태그`를 사용하여 설정한다.

```java
import java.util.Set;
public class CollectionBean {
    private Set<String> addressList;
    
    public void setAddressList(Set<String> addressList) {
        this.addressList = addressList;
    }
    
    public Set<String> getAddressList(){
        return addressList;
    }
}
```
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd"  >

    <bean id="collectionBean" class="ch4_4_2.CollectionBean">
        <property name="addressList">
            <set value-type="java.lang.String">
                <value>서울시 강남구 역삼동</value>
                <value>서울시 성동구 성수동</value>
                <value>서울시 성동구 성수동</value>
            </set>
        </property>
    </bean>
</beans>
```

실행결과
---
```
서울시 강남구 역삼동
서울시 성동구 성수동
```


## 4.4.3 Map 타입 매핑
- 특정 key로 데이터를 등록하고 사용할 때는 java.util.Map 컬렉션을 사용하며, <br />
<map>태그를 사용하여 설정할 수 있다.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd"  >

    <bean id="collectionBean" class="ch4_4_3.CollectionBean">
        <property name="addressList">
            <map>
                <entry>
                    <key><value>고길동</value></key>
                    <value>서울시 강남구 역삼동</value>
                </entry>
                <entry>
                    <key><value>마이콜</value></key>
                    <value>서울시 강서구 화곡동</value>
                </entry>
            </map>
        </property>
    </bean>
</beans>
```

## 4.4.4 Properties 타입 매핑
- key = value 형태의 데이터를 등록하고 사용할 때는 java.util.Properties 컬렉션을 사용하며, <br />
<props> 엘리먼트를 사용하여 설정한다. 
- Properties 클래스는 키와 값이 String 타입으로 제한된다. 

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd"  >

    <bean id="collectionBean" class="ch4_4_4.CollectionBean">
        <property name="addressList">
            <props>
                <prop key="고길동">서울시 강남구 역삼동</prop>
                <prop key="마이콜">서울시 강서구 화곡동</prop>
            </props>
        </property>
    </bean>
</beans>
```


