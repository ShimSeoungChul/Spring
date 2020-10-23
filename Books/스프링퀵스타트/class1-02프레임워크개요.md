# 2.1 프레임워크 개념
## 2.1.1 프레임워크 등장 배경
- 프레임워크의 사전적 의미는 `뼈대` 혹은 `틀`로서 이 의미를 소프트웨어 관점에서 접근하면<br />
`아키텍처`에 해당하는 `골격 코드`다.
- 애플리케이션을 개발할 때, 가장 중요한 것은 `애플리케이션의 구조를 결정`하는 아키텍처인데<br />
이 아키텍처에 해당하는 골격 코드를 프레임워크가 제공한다.
- 즉, 개발자에게 모든 것을 위임하는 것이 아니라 애플리케이션의 기본 아키텍처는 프레임워크가 제공하고, <br />
그 뼈대에 살을 붙이는 작업만 개발자가 하는 것이다.

## 2.1.2 프레임워크의 장점
- 잘 만들어진 프레임워크를 사용하면 애플리케이션에 대한 `분석`, `설계`, `구현` 모두에서 `재사용성이 증가`하며, <br />
이를 통해 다음과 같은 장점들을 얻을 수 있다,

###  1. 빠른 구현 시간
- 프레임워크를 사용하면 골격 코드를 프레임워크가 제공하므로, <br />
개발자는 비즈니스 로직만 구현하면 된다.

### 2. 쉬운 관리
- 같은 프레임워크가 적용된 애플리케이션들을 아키텍처가 같으므로 관리하기가 쉽다.  <br />
결과적으로 유지보수에 들어가는 인력과 시간도 줄일 수 있다.

### 3. 개발자들의 역량 획일화
- 프레임워크를 사용하면 숙련된 개발자와 초급 개발자가 생성한 코드가 비슷해진다 <br /> 
즉, 초급 개발자도 프레임워크를 통해서 세련되고 효율적인 코드를 생성할 수 있다. <br />
결과적으로 관리자는 개발 인력을 더 효율적으로 구성할 수 있다.

### 4. 검증된 아키텍처의 재사용과 일관성 유지
- 프레임워크에서 제공하는 아키텍처를 이용하므로 아키텍처에 관한 별다른 고민이나 검증 없이 소프트웨어를 개발 할 수 있다.
- 이렇게 개발한 시스템은 시간이 지나도 유지보수 과정에서 아키텍처가 왜곡되거나 변경되지 않는다.

## 2.1.3 자바 기반의 프레임워크
|처리 영역| 프레임워크 | 설명 |
|---|---|---------------|
|Presentation|Struts| UI Layer에 중점을 두구 개발된 MVC 프레임워크 |
| |Spring(MVC)| Struts와 동일하게 MVC 아키텍처를 제공하는 UI Layer 프레임워크. Structs처럼 독립된 프레임워크가 아니라 Spring 프레임워크에 포함되어 있다.|
|Business | Spring(IoC, AOP)| Spring은 컨테이너 성격을 가지는 프레임워크다. Spring의 IoC와 AOP 모듈을 이용하여 Spring 컨테이너에서 동작하는 엔터프라이즈 비즈니스 컴포넌트를 개발할 수 있다. |
|Persistence | Hibernate or JPA | Hibernate는 ORM(Object Relation Mapping) 프레임워크다. ORM 프레임워크는 SQL 명령어를 프레임우크가 자체적으로 생성하여 DB 연동을 처리한다. JPA는 Hibernate를 비롯한 모든 ORM의 공통 인터페이스를 제공하는 자바 표준 API이다.  |
|| Ibatis or Mybatis | ibatis 프레임워크는 개발자가 작성한 SQL 명령어와 자바 객체(VO 혹은 DTO)를 매핑해주는 기능을 제공하며, 기존에 사용하던 SQL 명령어를 재사용하여 개발하는 차세대 프로젝트에 유용하게 적용할 수 있다. Mybatis는 Ibatis에서 파생된 프레임워크로서 기본 개념과 문법은 거의 같다. |


# 2.2 스프링 프레임워크
## 2.2.1
- When: 로드 존슨(Rod Johnson)이 2004년에 만든 오픈소스 프레임워크
- Why: EJB 프레임워크의 단점을 극복하기 위해서. 
스프링 등장 이전에 자바 기반 엔터프라이즈 애플리케이션은 대부분 EJB(Enterprise Java Bean)로 개발되었다. <br />
하지만 이는 스펙이 복잡하여 `학습에 많은 시간`이 필요하며 `개발 및 유지보수 역시 복잡`했다. <br />
또한 EJB를 제대로 사용하려면 EJB의 성능을 유지해주고, 유지보수 편의성을 향상하기위해 다양한 디자인 패턴을 이해하고 적용해야만 했다.
이에 반해 스프링 프레임워크는 평병한 POJO를 사용하면서도 EJB에서만 가능했던 많은 일을 가능하도록 지원했고, <br />
이미 많은 디자인 패턴이 적용되어 배포되므로 EJB를 사용할 때 알아야만 했던 수많은 디자인 패턴 역시 신경 쓰지 않아도 되었다.

```
POJO(Plain Old Java Object)란?
```


## 2.2.2 스프링 프레임워크의 특징 
- 스프링의 특징을 한 줄로 서술하면 `IoC와 AOP를 지원하는 경량의 컨테이너 프레임워크`로 표현할 수 있다.

### 1. 경량(Lightweight)
- 스프링은 `크기 측면에서 가볍다`. <br />
스프링은 `여러 개의 모듈로 구성`돠며, 각 `모듈`은 하나 이상의 `jAR 파일`로 구성된다. <br />
그리고 이 몇 개의 JAR 파일만 있으면 `개발`과 `실행`이 모두 가능하다.
- 따라서 스프링을 이용해서 만든 애플리케이션의 배포 또한 빠르고 쉽다.
- 스프링 프레임워크는 `POJO 형태의 객체`를 관리한다. 
POJO는 클래스를 구현하는 데 특별한 규칙이 없는 단순하고 가벼운 객체이므로 POJO를 관리하는 것은 
기존의 EJB 객체를 관리하는 것보다 빠르고 가볍다.

### 2. 제어의 역행(Inversion of Control)
- 스프링은 제어의 역행(Inversion of Control)을 통해 애플리케이션을 구성하는 객체의 느슨한 결합, <br />
즉 낮은 결합도를 유지한다. <br />

- 아래는 IoC가 적용되지 않은 상황과 적용된 상황이다. <br />
IoC가 적용되지 전에는 애플리케이션 수행에 필요한 `객체의 생성`이나 `객체와 객체의 의존관계`를 `개발자가 직접 자바 코드로 처리`했다. <br />
때문에 의존관계에 있는 객체를 변경할 때는, 반드시 자바 코드를 수정해야 했다. <br />
![그림2-5IoC가아닌경우와IoC가적용된구조](https://user-images.githubusercontent.com/40673012/95156477-a56ee680-07d1-11eb-83e2-3d74fd90cb22.png)

- 하지만 IoC가 적용되면 이를 `컨테이너`가 대신 처리하여, 소스에 의존관계가 명시되지 않으므로 <br />
`결합도`가 떨어져서 `유지보수`가 편리해진다.

### 3. 관점지향 프로그래밍(Aspect Oriented Programming)
- 비즈니스 메소드를 개발할 때,  `핵심 비지니스 로직`과 각 비즈니스 메소드마다 반복해서 등장하는 <br />
`공통 로직`을 분리하여 `응집도` 높은 개발을 지원한다.

- AOP의 기본 개념: `공통으로 사용하는 기능`들을 `외부의 독립된 클래스`로 분리하고, <br  />
해당 기능을 프로그램 코드에 직접 명시하지 않고 `선언적으로 처리`하여 적용한다.

- Why? 장점: 1.응집도 높은 비즈니스 컴포넌트를 만들 수 있고, 2. 유지보수를 향상시킬 수 있다.

### 4. 컨테이너
- What? : 1. 특정 객체의 생성과 관리는 담당하고, 2. 객체 운용에 필요한 다양한 기능을 제공, 3. 일반적으로 서버 안에 포함되어 배포 및 구도

- 예시: 대표적인 컨테이너로 Servlet 객체를 생성하고 관리하는 `Servlet 컨테이너`와  <br />
EJB 객체를 생성하고 관리하는 EJB 컨테이너가 있다. <br />
Servlet 컨테이너는 우리가 사용하는 톰캣 서버에도 포함되어 있다. <br />
애플리케이션 운용에 필요한 `객체를 생성`하고` 객체 간의 의존관계를 관리`한다는 점에서 `스프링도 일종의 컨테이너`다.


# 2.3 IoC(Inversion of Control) 컨테이너

- What?: 제어의 역행(IoC)은 `결합도`와 관련된 개념으로 일련의 작업들(객체의 생성 및 실행 등)을 `소스코드로 처리하지 않고 컨테이너로 처리`하는 것을 의미한다. <br />
- Why? : 따라서 제어의 역핵을 이용하면 소스에서 `객체 생성`과 `의존관계에 대한 코드`가 사라져 `낮은 결합도의 컴포넌트`를 구현할 수 있다.

### 컨테이너
- 컨테이너의 개념은 스프링 이전의 기존 서블릿이나 EJB 기술에서 이미 사용해왔다. <br />
대부분 컨테이너는 `비슷한 구조와 동작 방식`을 가지고 있으므로 <br />
서블릿 컨테이너를 통해 스프링 컨테이너의 동작 방식을 `유추`해볼 수 있다. 

> HelloServlet.java
```java
public class HelloServlet extends HttpServlet{

    public HelloServlet() {
        System.out.println("===> HelloServlet 객체 생성");
    }
    
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        System.out.println("doGet() 메소드 호출");
    }
}
```

- 이클립스를 이용하여 서블릿 클래스를 개발한다면 작성된 Servlet 클래스는 web.xml 파일에 자동으로 등록된다.

> /WEB-INF/web.xml
```xml
<web-app>
    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>hello.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello.do</url-pattern>
    </servlet-mapping>
</web-app>
```

- 위 설정은 브라우저에서 `/hello.do`라는 `URL 요청`을 전송하면, <br />
`hello라는 이름`으로 등록된 `hello.HelloServlet 클래스`를 찾아 객체를 생성하고 실행한다는 설정이다.

- HelloServlet 프로그램을 실행하면 다음과 같은 메시지가 콘솔에 출력되며, <br />
`web.xml` 설정대로 `객체가 생성`되고 `실행`된다.

> <실행결과>
```
===> HelloServlet 객체 생성
===> doGet () 메소드 호출
```
- 서블릿은 `자바로 만든 클래스`이므로 반드시 `객체 생성`을 해야 객체가 가진 메서드도 호출할 수 있다. <br />
하지만 위 예제 처럼 `객체 생성 코드 없이`도 `서블릿 컨테이너`가 `서블릿 객체`를 생성하고 `doGet() 메서드`를 호출하여 실행결과를 얻을 수 있다.

![서블릿 컨테이너의 서블릿 객체 관리](https://user-images.githubusercontent.com/40673012/95410001-d419c800-095d-11eb-95b7-17baca7b6d3b.png)
출처:https://medium.com/@fntldpf12/%EC%84%9C%EB%B8%94%EB%A6%BF-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EB%9E%80-d5095e661a85

- 컨테이너는 자신이 관리할 클래스들이 등록된 XML 설정 파일을 로딩하여 구동한다. <br />
그리고 클라이언트의 요청이 들어오는 순간 XML 설정 파일을 참조하여 객체를 생성하고, <br />
객체의 생명주기를 관리한다. <br />

- 스프링 컨테이너 역시 서블릿 컨테이너와 유사하여 살펴본 요소들과 비슷한 요소들이 존재한다. <br />

## 2.3.1 결합도(Coupling)가 높은 프로그램 
- What? : 하나의 클래스가 다른 클래스와 얼마나 많이 `연결`되어 있는지를 나타내는 표현이다. <br />
결합도가 높은 프로그램은 `유지보수`가 어렵다.

- 아래와 같이 SamsungTV, LgTV 클래스가 있고, 두 클래스는 TV 시청에 필요한 필수 기능인 네 개의 메소드가 있다. <br />
두 클래스는 같은 기능을 수행하는 메소드가 있지만, 이름이 서로 다르다.

```java
public class SamsungTV {
    public void powerOn() {
        System.out.println("SumsungTV--- 전원 켠다.");
    }
    public void powerOff() {
        System.out.println("SumsungTV--- 전원 끊다.");
    }
    public void volumeUp() {
        System.out.println("SumsungTV--- 소리 올린다.");
    }
    public void volumeDown() {
        System.out.println("SumsungTV--- 소리 내린다.");
    }
}

```

```java
public class LgTV {
    public void turnOn() {
        System.out.println("LgTV--- 전원 켠다.");
    }
    public void turnOff() {
        System.out.println("LgTV--- 전원 끊다.");
    }
    public void soundUp() {
        System.out.println("LgTV--- 소리 올린다.");
    }
    public void soundDown() {
        System.out.println("LgTV--- 소리 내린다.");
    }

}
```

- 아래 코드는 위에 SamsungTV, LgTV 클래스를 번갈아 사용하는 TVUser 프로그램이다.
```java
public class TVUser {
    public static void main(String[] args) {
        SamsungTV tv = new SamsungTV();
        tv.powerOn();
        tv.volumeUp();
        tv.volumeDown();
        tv.powerOff();
    }
}
```

```java
public class TVUser {
    public static void main(String[] args) {
        LgTV tv = new LgTV();
        tv.turnOn();
        tv.soundUp();
        tv.soundDown();
        tv.turnOff();
    }
}
```

- SamsungTV, LgTV 클래스는 메소드 시그니처가 다르므로 TVUser `코드 대부분을 수정`해야 TV를 교체할 수 있고, <br />
`두 클래스가 같은 메소드를 갖도록 강제할 수 없다`.
- 또한 TVUser와 같은 클라이언트 프로그램이 여러 개가 된다면 `유지보수`는 더욱더 힘들어 지며, TV 교체를 결정하기가 어려줘진다.


## 2.3.2 다형성 이용하기 
- `결합도`를 낮추기 위해서 객체지향 언어의 핵심 개념인 `다형성(Polymorphism)`을 이용할 수 있다. <br />
다형성을 이용하려면 `상속`과 `메소드 재정의(Overriding)`, `형변환`이 필요하다. 

- 아래 코드와 같이 `인터페이스`를 이용하여 모든 TV 클래스가 `같은 메소드를 가질 수밖에 없도록 강제`할 수 있다. 

```java
public interface TV {
    public void powerOn();
    public void powerOff();
    public void volumeUp();
    public void volumeDown();
}
```
```java
public class SamsungTV implements TV{
    public void powerOn() {
        System.out.println("SumsungTV--- 전원 켠다.");
    }
    public void powerOff() {
        System.out.println("SumsungTV--- 전원 끊다.");
    }
    public void volumeUp() {
        System.out.println("SumsungTV--- 소리 올린다.");
    }
    public void volumeDown() {
        System.out.println("SumsungTV--- 소리 내린다.");
    }
}
```
```java
public class LgTV implements TV{
    @Override
    public void powerOn() {
        System.out.println("LgTV--- 전원 켠다.");
    }
    @Override
    public void powerOff() {
        System.out.println("LgTV--- 전원 끊다.");
    }
    @Override
    public void volumeUp() {
        System.out.println("LgTV--- 소리 올린다.");
    }
    @Override
    public void volumeDown() {
        System.out.println("LgTV--- 소리 내린다.");
    }
}
```

- 아래 코드 처럼 TVUser 클래스는 TV 인터페이스 타입의 변수로 `묵시적 형변환(promotion)`을 이용하여 SamsungTV 객체를 참조한다.  <br />
이렇게 `다형성`을 이용하면 참조하는 객체만 변경하면 되므로(SamsungTV에서 LgTV )  TVUser와 같은 클라이언트 프로그램이 여러 개 있더라도,
최소한의 수정으로 `객체를 쉽게 교체`할 수 있다. 즉, `유지보수`가 더 편해진다.

```java 
public class TVUser {
    public static void main(String[] args) {
        TV tv = new SamsungTV();
        tv.powerOn();
        tv.volumeUp();
        tv.volumeDown();
        tv.powerOff();
    }
}
```


## 2.3.3 디자인 패턴 이용하기
- 위 코드의 `단점`은 TV 클래스 객체를 변경할 때, TV 클래스 객체를 생성하는 소스를 수정해야 한다는 점이다.
- TV를 교체할 때, `클라이언트 소스를 수정하지 않고` TV를 교체할 수 있다면 `유지보수`는 더 편리해질 것이다. <br />
이를 위해 Factory 패턴을 적용할 수 있다.

### Factory 패턴
- Factory 패턴은 클라이언트에서 사용할 `객체 생성을 캡슐화`하여 클라이언트(TVUser)와 객체(TV) 사이를 `느슨한 결합` 상태로 만들어준다. 
- 아래 코드에서 실행되는 TV를 변경하고 싶을 때는 명령행 매개변수만 수정하여 실행하면 된다. <br />
즉, 클라이언트는 자신이 `원하는 객체를 직접 생성하지 않아`, `클라이언트 소스를 수정하지 않고`도 실행되는 객체를 변경할 수 있다.
- 
```java
public class BeanFactory {
    public Object getBean(String beanName) {
        if(beanName.equals("samsung")) {
            return new SamsungTV();
        }else if(beanName.equals("lg")){
            return new LgTV();
        }
        return null;
    }
}
```
```java
public class TVUser {
    public static void main(String[] args) {
        BeanFactory factory = new BeanFactory();
        TV tv = (TV)factory.getBean(args[0]);
        tv.powerOn();
        tv.volumeUp();
        tv.volumeDown();
        tv.powerOff();
    }
}
```

![TVUser프로그램실행과정](https://user-images.githubusercontent.com/40673012/95699166-b8c0fc80-0c7e-11eb-9648-3d7a3e8940cb.png)


