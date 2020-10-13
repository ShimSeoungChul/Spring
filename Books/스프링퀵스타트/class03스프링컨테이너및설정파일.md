# 03 스프링 컨테이너 및 설정 파일
- 대부분 IoC 컨테이너는 각 컨테이너에서 관리할 객체들을 위한 별도의 설정 파일이 있다.
- `Servlet 컨테이너`는 `web.xml 파일`에, EJB 컨테이너는 ejb-jar.xml 파일에 해당 컨테이너가 생성하고 관리할 클래스들을 등록한다.
- 스프링 프레임워크도 마찬가지로 자신이 관리할 클래스들이 등록된 XML 설정 파일이 필요하며,  <br />
STS를 이용하면 간단하게 만들 수 있다.

## 3.1 스프링 IoC 시작하기
### 3.1.1 스프링 설정 파일 생성 
- 기존에 생성한 'Spring MVC Project'의  <br />
1. src/main/resource 소스 폴더를 선택하고 마우스 오른쪽 버튼을 클릭한다.
2. 'Spring' 폴더에 있는 'Spring Bean Configuration File'을 선택하고 <Next>를 클린한다.
3. 'File name'에 applicationContext를 입력하고 <Finish>를 클릭하면 스프링 설정 파일이 생성된다. 
<img width="592" alt="그림3-1" src="https://user-images.githubusercontent.com/40673012/95703091-31c55180-0c89-11eb-8540-05eac1595b10.png">

- 생성한 파일에는 기본으로 `<beans> 루트 엘리먼트`와 `네임스페이스` 관련 설정들이 제공된다.<br><br>

applicationContext.xml
---
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

- Class02의 SamsungTV 클래스를 스프링 설정 파일에 등록한다고 가정하면, <br>
다음과 같이 클래스 하나당 하나의 `<bean>` 설정이 필요하다. <br />
이때 `패키지 경로`가 코함된 전체 클래스 경로를 지정한다.
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="tv" class="ch2_3_3.SamsungTV"/>
</beans>
```

### 3.1.2 스프링 컨테이너 구동 및 테스트
```java
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class TVUser {
    public static void main(String[] args) {
        //1. Spring 컨테이너를 구동한다.
        AbstractApplicationContext factory = new GenericXmlApplicationContext("applicationContext.xml");
    }
}
```
- 위 소스는 환경설정 파일인 `applicationContext.xml`을 로딩하여 `스프링 컨테이너` 중 하나인 `GenericXmlApplicaionContext`를 구동한다.

실행결과
---
```
INFO : org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [applicationContext.xml]
INFO : org.springframework.context.support.GenericXmlApplicationContext - Refreshing org.springframework.context.support.GenericXmlApplicationContext@50de0926: startup date [Mon Oct 12 13:08:59 KST 2020]; root of context hierarchy
INFO : org.springframework.beans.factory.support.DefaultListableBeanFactory - Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@37ceb1df: defining beans [tv]; root of factory hierarchy
```
- 실행 결과를 보면 클래스 경로에 있는 `applicationContext.xml 파일이 로딩`되고, <br />
다음으로 `GenericXmlApplicationContext 객체가 생성`되어 `스프링 컨테이너가 구동`됐다는 메시지가 출력된다. 

- 위 코드를 수정하여 아래 처럼 `스프링 컨테이너를 구동`하고 이름이 tv인 객체를 `getBean() 메소드`를 이용하여 요청할 수 있다. 
```java
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class TVUser {
    public static void main(String[] args) {
        //1. Spring 컨테이너를 구동한다.
        AbstractApplicationContext factory = new GenericXmlApplicationContext("applicationContext.xml");
        
        //2. Spring 컨테이너로부터 필요한 객체를 요청(Lookup)한다.
        TV tv = (TV)factory.getBean("tv");
        tv.powerOn();
        tv.powerOn();
        tv.volumeUp();
        tv.volumeDown();
        tv.powerOff();
        
        //3.Spring 컨테이너를 종료한다.
        factory.close();
    }
}
```

![그림3-3스프링컨테이너의동작순서](https://user-images.githubusercontent.com/40673012/95807898-3a2f9200-0d46-11eb-9e0e-76d0c60e9052.png)

1. TVUser 클라이언트가 스프링 설정 파일을 로딩하여 컨테이너 구동 <br>
2. 스프링 설정 파일에 <bean> 등록된 SamsungTV 객체 생성 <br>
3. getBean() 메소드로 이름이 'tv'인 객체를 요청(Lookup) <br>
4. SamsungTV 객체 반환 <br>

- 실행되는 TV를 LgTV로 `변경`할 때 클라이언트 소스를 수정하지 않고도, 
`applicationContext.xml 파일만 수정`하면 동작하는 TV를 변경할 수 있다. <br />

### 3.1.3 스프링 컨테이너의 종류
- 스프링은 `BeanFactory`와 이를 상속한 `ApplicationContext` 두 가지 유형의 컨테이너를 제공한다.

#### BeanFactory
- 스프링 설정 파일에 등록된 <bean> 객체를 생성하고 관리하는 가장 기본적인 컨테이너 기능만 제공한다. <br />
- 컨테이너가 구동될 때 <bean> 객체를 생성하지 않고, `클라이언트의 요청(Lookup)`에 의해서만 <bean> 객체가 생성되는 `지연 로딩(Lazy Loading) 방식`을 사용한다.
- 일반적인 스프링 프로젝트에서 BeanFactory를 사용할 일은 거의 없다.

#### ApplicationContext
- BeanFactory가 제공하는 1. `<bean> 객체 관리 기능` 외에도 2. `트랜잭션 관리`, 3. `메시지 기반 다국어 처리` 등 다양한 기능을 지원한다. 
- `컨테이너가 구동되는 시점`에 <bean> 등록된 클래스들을 객체 생성하는 `즉시 로딩(pre-loading) 방식`으로 동작한다.
- `웹 애플리케이션 개발`도 지원하여 대부분 스프링 프로젝트가 ApplicationContext 유형의 컨테이너를 이용한다.


구현 클래스 예시
---
|구현클래스|기능|
|---|-----|
|GenericXmlApplicationContext|파일 시스템이나 클래스 경로에 있는 XML 설정 파일을 로딩하여 구동하는 컨테이너|
|XmlWebApplicationContext|웹 기반의 스프링 애플리케이션을 개발할 때 사용하는 컨테이너|

- `GenericXmlApplicationContext`은 클라이언트가 직접 객체를 생성하여 구동하는 컨테이너다.
- `XmlWebApplication`은 웹 애플리케이션 개발에 사용되며, 클라이언트가 직접 생성하지 않으며, <br>
구동되고 사용되는 방식은 `Spring MVC`를 학습할 떄 확인할 수 있다.


## 3.2 스프링 XML 설정
### 3.2.1 <beans> 루트 엘리먼트 
- 스프링 컨테이너는 `<bean> 저장소`에 해당하는 `XML 설정 파일`을 참조하여 <bean>의 생명주기를 관리하고 여러 서비스를 제공한다.
- 스프링 설정 파일 이름은 무엇을 사용하든 상관없지만 <beans>를 `루트 엘리먼트로 사용`하고, <br>
<beans> 엘리먼트 `시작 태그`에 `네임스페이스`를 비롯한 `XML 스키마 관련 정보`가 설정된다.
- `STS`를 이용하여 만든 스프링 설정 파일에는 `beans 네임스페이스`가 `기본 네임스페이스`로 선언되어 있고, <br />
`spring-beans.xsd 스키마 문서`가 `schemaLocation`으로 등록되어 있어 <bean>, <description>, <alias>,<import> 등 `네 개의 엘리먼트`를 자식 엘리먼트로 사용할 수 있다.
    - <bean>, <import>가 실제 프로젝트에 많이 사용

### 3.2.2 <import> 엘리먼트

-  복잡한 `스프링 설정`을 하나의 파일이 아니라 기능별 여러 XML 파일로 나누어 설정하고, `파일을 하나로 통합`할 때
<import> 엘리먼트를 사용한다.
- <import> 태그를 이용하여 여러 스프링 설정 파일을 포함함으로써 한 파일에 작성하는 것과 같은 효과를 낼 수 있다.

![3.2.2 import 엘리먼트](https://user-images.githubusercontent.com/40673012/95809525-fe96c700-0d49-11eb-9a68-bb9fe08ce379.png)

### 3.2.3 <bean> 엘리먼트
- 스프링 설정 파일에 클래스를 등록하려면 <bean> 엘리먼트를 사용한다. <br>
이때 `id`와 `class` 속성을 사용하며, id 속성은 생략 가능하지만 class 속성은 필수다.
<br>

- id 속성은 생략 가능하지만 위의 `getBean() 메소드` 사용예제 처럼 객체를 요청(Lookup)하려면 <br />
<bean> 객체를 위한 이름을 설정하기 위해 id 속성을 반드시 지정해야한다.<br />

- id 속성은 컨테이너로부터 `<bean> 객체를 요청`할 때 사용하므로 `컨테이너가 각 객체를 식별`하기 위해 저체 스프링 파일 내에서 `유일`해야한다.
<br />


- id 속성값에 해당하는 문자열은 `자바의 식별자 작성 규칙`을 따르며, 일반적으로 `CamelCase`를 사용한다. <br>

- 에러가 발생하는 경우: `숫자`로 시작, `공백` 포함, `특수기호` 사용 <br />

- `name` 속성은 id와 같은 기능을 하지만, id와 다르게 `자바 식별자 작성 규칙을 따르지 않는 문자열도 허용`한다. <br />
따라서 `특수기호`가 포함된 아이디를 <bean> 아이디로 지정할 때 id 대신 name 속성을 쓴다.  <br />
name 역시 전체 스프링 파일 내에서 유일해야 한다.

- 


### 3.2.4 <beans> 엘리먼트 속성
