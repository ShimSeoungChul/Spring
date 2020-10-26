# 05 어노테이션 기반 설정
## 5.1 어노테이션 설정 기초
- 스프링 프레임워크의 `XML 설정`은 매우 중요하다. <br />
`문제점`: 하지만 XML 파일의 `과도한 설정`에 대한 부담도 크며, 이로 인해 `프레임워크 사용을 꺼리기도` 한다. <br />
`해결책`: 따라서 대부분 프레임워크는 `어노테이션`을 이용한 설정을 지원한다.

### 5.1.1 Context 네임스페이스 추가
- 어노테이션 설정을 추가하려면 다음과 같이 스프링 설정 파일의 루트 엘리먼트인 <beans>에 <br />
Context 관련 네임스페이스와 스키마 문서의 위치를 등록해야 한다.

- IDE에서 설정 xml의 [Namespace] 탭을 선택하고 'context' 항목만 체크하면 간단하게 추가할 수 있다.
- 나 처럼 IDE에 오류가 있으면 아래 문장들을 적절히 

xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-4.2.xsd""

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p" 
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd">
</beans>
```

### 5.1.2 컴포넌트 스캔(component-scan) 설정
- 스프링 설정 파일에 애플리케이션에서 사용할 객체들을 <bean> 등록하지 않고 자동으로 생성하려면 <br />
`<context:component-scan>`이라는 엘리먼트를 정의해야 한다. <br />
이 설정을 추가하면 스프링 컨테이너는 `클래스 패스에 있는 클래스`들을 스캔하여 `@Component`가 설정된 클래스들을 자동으로 객체 생성한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p" 
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <context:component-scan base-package="ch5_1"></context:component-scan>
    
    <!--
    <bean id="tv" class="ch5_1.SamsungTV" p:speaker-ref="sony" p:price="2700000"/>
    <bean id="sony" class="ch5_1.SonySpeaker"></bean>
    <bean id="apple" class="ch5_1.AppleSpeaker"></bean>
    -->
</beans>
```

- <context:component-scan/> 설정을 제외한 나머지 <bean> 설정은 모두 삭제하거나 주석 처리한다.
- `base-packge`:  base-package 속성값을 "ch5_1" 형태로 전하면 ch5_1 패키지로 시작하는 모든 패키지를 스캔 대상에 포함한다.

### 5.1.3 @Component
- <context:component-scan>를 설정했으면 `@Component`만 `클래스 선언부` 위에 설정하면 <br />
스프링 컨테이너는 해당 클래스를 bean으로 생성하고 관리한다. 

```xml
<bean class="ch5_1.LgTV"></bean>
```

```java
@Component
public class LgTV {
    public LgTV() {
        System.out.println("===> LgTV 객체 생성");
    }
}
```

- 위의 두 설정 모두 해당 클래스에 `기본 생성자`가 있어야만 컨테이너가 객체를 생성할 수 있다.
- 이렇게 설정했다면 `클래스의 객체가 메모리에 생성`되는 것은 문제가 없지만 <br />
클라이언트 프로그램에서 LgTV 객체를 요청할 수는 없다. 
```
//1. Spring 컨테이너를 구동한다.
AbstractApplicationContext factory = new GenericXmlApplicationContext("applicationContext.xml");

//2. Spring 컨테이너로부터 필요한 객체를 요청(Lookup)한다.
TV tv = (TV)factory.getBean("tv");
```

- `클라이언트의 요청`을 위해서 다음과 같이 `아이디 설정`이 필요하다. 

```xml
<bean id = "tv" class="ch5_1.LgTV"></bean>
```

```java
@Component("tv")
public class LgTV {
    public LgTV() {
        System.out.println("===> LgTV 객체 생성");
    }
}
```
### id 또는 name 속성 미지정 시 이름 규칙
- 이 경우 `컨테이너가 자동`으로 이름을 설정해준다. <br />
이때 이름 `규칙`은 클래스 이름의 `첫 글자`를 `소문자`로 변경하기만 하면 된다. <br />
따라서 id 또는 name 속성이 설정되지 않은 경우, LgTV 객체를 요청하려면 lgTV라는 이름을 사용하자.

## 5.2 의존성 주입 설정
## 5.2.1 의존성 주입 어노테이션
- 스프링에서 의존성 주입을 지원하는 어노테이션으로는 @Autowired, @Inject, @Qualifier, @Resource가 있다.

|어노테이션|설명|
|--|----|
|@Autowired|주로 변수 위에 설정하여 해당 타입의 객체를 찾아서 자동으로 할당한다. org.springframework.beans.factory.annotation.Autowired|
|@Qualifier|특정 객체의 이름을 이용하여 의존성 주입할 때 사용한다. org.springframework.beans.factory.annotation.Qualifier|
|@Inject|@Autowired와 동일한 기능을 제공한다. javax.inject.Inject|
|@Resource|@Autowired와 @Qualifier의 기능을 결합한 어노테이션이다. javax.annotation.Resourc|

## 5.2.2 @Autowired
- `메소드` 또는 `멤버변수` 위에 사용할 수 있다. 어디에 사용하든 결과는 같지만, 대부분 `멤버변수` 위에 선언한다.
- 스프링 컨테이너는 멤버변수 위에 붙은 `@Autowired`를 확인하는 순간 `해당 변수의 타입`을 체크한다. <br />
그리고 그 타입의 객체가 `메모리`에 존재하는지 확인한 후, 그 객체를 변수에 `주입`한다.
- @Autowired가 붙은 객체가 메모리에 없다면 컨테이너가 `NoSuchBeanDefinitionException`을 발생시킨다.

```java
@Component("tv")
public class LgTV implements TV{
    
    @Autowired
    Speaker speaker;
    
    public LgTV() {
        System.out.println("===> LgTV 객체 생성");
    }
    public void powerOn() {
        System.out.println("LgTV--- 전원 켠다.");
    }
    public void powerOff() {
        System.out.println("LgTV--- 전원 끊다.");
    }
    public void volumeUp() {
        speaker.volumeUp();
    }
    public void volumeDown() {
        speaker.volumeDown();
    }
}
```
- 위처럼 설정하면 LgTV 클래스에는 의존성 주입에 사용했던 `Setter 메소드`나 `생성자`는 필요없다. <br />
스프링 설정 파일 역시 <context:component-scan> 외에는 아무런 설정도 하지 않는다. <br />

- 하지만 SonySpeaker 객체가 `메모리`에 없으면 에러가 발생하으모 반드시 SonySpeaker 객체가 메모리에 생성되어 있어야 한다. <br />
다음 두 방법 중 하나를 선택해 처리한다.

```xml
<bean id = "sony" class="ch5_2.LgTV"></bean>
```
```java
@Component("sony")
public class SonySpeaker implements Speaker{
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

## 5.2.3 @Qualifier
- 문제점: 다음과 같이 의존성 주입 대상이 되는 Speaker 타입의 객체가 두 개 이상이면 문제가 발생한다.

```
@Component
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

실행결과
---
```
Exception in thread "main" org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tv': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: ch5_2.Speaker ch5_2.LgTV.speaker; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No unique bean of type [ch5_2.Speaker] is defined: expected single matching bean but found 2: [sony, apple]
```

- 아래 클래스 다이어그램 처럼 @Autowired 대상이 되는 Speaker 타입의 객체가 AppleSpeaker, SonySpeaker로 두 개이고, <br />
둘 다 메모리에 생성되어 있으므로 `어떤 객체를 의존성 주입할지 모르`기 때문에 발생한 예외다.

- 해결: `@Qualifier` 어노테이션을 이용하여 의존성 주입될 객체의 `아이디`나 `이름`을 지정한다.

![그림 5-2](https://user-images.githubusercontent.com/40673012/96969261-2b6a8b80-154d-11eb-920d-6d604789af5b.png)

```java
@Component("tv")
public class LgTV implements TV{
    
    @Autowired
    @Qualifier("apple")
    Speaker speaker;
    
    public LgTV() {
        System.out.println("===> LgTV 객체 생성");
    }
    ~생략~
}
```
## 5.2.4 @Resource
- `객체의 이름`을 이용하여 의존성 주입을 처리한다. 
    - `@Autowired`는 이와 다르게 `변수의 타입`을 기준으로 객체를 검색하여 의존성 주입을 처리
    - Java 9 부터는 @Resource가 존재하는  java.xml.ws.annotation annotation가 deprecated되어서 더이상 사용 불가
    
```java
@Component("tv")
public class LgTV implements TV{
    @Resource(name="apple")
    private Speaker speaker;
    
    public LgTV() {
        System.out.println("===> LgTV 객체 생성");
    }
        }
        ~생략~
    }
```

- 위 소스는 apple이라는 이름으로 메모리에 생성된 AppleSpeaker 객체를 speaker 변수에 할당한다.
- `@Inject` 어노테이션도 @Resource와 동일하게 이름을 기반으로 의존성 주입을 처리하낟.

## 5.2.5 어노테이션과 XML 설정 병행하여 사용하기
- 스프링으로 `의존성 주입`을 처리할 때, XML 설정과 어노테이션 설정은 `장단점이 서로 상충`한다. <br />

- `XML` 방식은 자바 소스를 수정하지 않고 XML 파일의 설정만 변경하면 실행되는 Speaker를 교체할 수 있어 유지보수가 편하지만, <br />
XML `설정`에 대한 `부담`이 존재한다. <br />
또한 자바 소스에 `의존관계와 관련된 어떤 메타데이터`도 없으므로 XML 설정을 해석해야만 무슨 객체가 의존성 주입되는지 확인할 수 있다. <br />

- `어노테이션` 기반 설정은 `XML 설정 부담`도 없고, `의존 관계에 대한 정보`가 `자바 소스`에 들어있어 사용하기 편하다. <br />
하지만 의존성 주입할 `객체의 이름이 자바 소스에 명시`되므로,  자바 소스 수정 없이 Speaker를 교체할 수 없다는 문제가 있다. <br />

- 이런 문제를 서로의 `장점을 조합`하는 것으로 해결할 수 있다.
- 1. LgTV 클래스의 speaker 변수를 원래대로 @Autowired 어노테이션만 설정한다.
```java
@Component("tv")
public class LgTV implements TV{
    
    @Autowired
    private Speaker speaker;
    
    public LgTV() {
        System.out.println("===> LgTV 객체 생성");
    }
}
```

- 2. Speaker 타입의 객체가 메모리에 두 개 있어서, 위 설정만으로는 에러가 발생한다. <br />
따라서 @Component를 제거하여 객체가 자동으로 생성되는 것을 차단한다.

```java
public class SonySpeaker implements Speaker{
    public SonySpeaker() {
            System.out.println("===> SonySpeaker 객체 생성");
    }
}

public class AppleSpeaker implements Speaker{
    public AppleSpeaker() {
        System.out.println("===> AppleSpeaker 객체 생성");
    }
}
```
- 3. 둘 중 하나만 스프링 설정 파일에 <bean> 등록하여 처리한다.

```xml
~생략~
<beans>
    <context:component-scan base-package="ch5_2_5"></context:component-scan>
    <bean id="apple" class="ch5_2_5.SonySpeaker"></bean>
</beans>
```

- 결국 `클라이언트가 요청`할 LgTV는 @Component `어노테이션`으로 처리하고, 의존성 주입 역시 @Autowired로 처리한다. <br />
`변경될 Speaker`만 `스프링 설정 파일`에 <bean> 등록하여 `자바 코드 수정 없이 XML 수정만으로 Speaker를 교체`할 수 있다. <br />

- 즉, `변경되지 않는 객체`는 어노테이션으로 설정하고, `변경될 가능성이 있는 객체`는 XML 설정으로 사용한다. <br />

- 또한 `직접 개발한 클래스`는 어노테이션, XML 설정 모두 사용할 수 있지만, <br />
`라이브러리로 제공되는 클래스`는 반드시 XML 설정을 통해서만 사용할 수 있다. 
    - 예: 아래 처럼 아파치에서 제공하는 BasicDataSource 클래스를 사용하여 DB 연동을 처리하면<br />
    'commons-dbcp-1.4.jar' 파일에 있는 BasicDataSource 클래스에 관련된 어노테이션을 추가할 수는 없다.

<img width="451" alt="그림 5-3" src="https://user-images.githubusercontent.com/40673012/97127095-11fb5680-177c-11eb-8fdc-587621deb2f7.png">



## 5.3 추가 어노테이션
- 그림 5-4는 `시스템의 전체 구조`를 `두 개의 레이어`로 표현한 것이다. <br />
`프레젠테이션 레이어`: 사용자와의 커뮤니케이션 담당<br />
`비즈니스 레이어`: 사용자 요청에 대한 비즈니스 로직 처리 담당 <br /> 

- 이 구조의 `가장 핵심 요소`는 `Controller`, `ServiceImpl`, `DAO` 클래스다. <br />
1. Controller 클래스는 `사용자의 요청`을 제어하고,  <br /> 
2. ServiceImple 클래스는 실질적인 `비즈니스 로직`을 처리하며, <br />
3. DAO(Data Access Object) 클래스는 `데이터베이스 연동`을 담당한다. <br /> 

- @Component를 이용하여 스프링 컨테이너가 해당 클래스 객체를 생성하도록 설정할 수 있다. <br />
문제점: 하지만 이렇게 하면. 어떤 클래스가 어떤 `역할`을 수행하는지 `파악하기 어렵`다. <br />
해결: 스프링 프레임워크는 이런 클래스들을 `분류하`기 위해 `@Component를 상속`하여 다음과 같은 세 개의 어노테이션을 추가로 제공한다.

|어노테이션|위치|의미| 
|--|--|--|
|@Service|XXXServiceimpl|비즈니스 로직을 처리하는 Service 클래스|
|@Repository|XXXDAO|데이터베이스 연동을 처리하는 DAO 클래스|
|@Controller|XXXController|사용자 요청을 제어하는 Controller 클래스|

<img width="554" alt="그림 5-4" src="https://user-images.githubusercontent.com/40673012/97126706-f2aff980-177a-11eb-8036-932688e6cf14.png">

- 추가된 어노테이션은 `클래스 분류` 뿐만 아니라 `다양한 기능`을 제공한다. <br />
1. @Controller는 해당 객체를 `MVC`(Model-View-Controller) 아키텍처에서 `컨트롤러 객체`로 인식하게 한다.<br />
2. @Repository는 DB 연동 과정에서 발생하는 예외를 변환해주는 기능이 있다. <br />
- 자세한 내용은 뒷 장에서 다룬다.





