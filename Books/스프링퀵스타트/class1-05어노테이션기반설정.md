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


## 5.2.3 @Qualifier
## 5.2.4 @Resource
## 5.2.5 어노테이션과 XML 설정 병행하여 사용하기

## 5.3 추가 어노테이션



