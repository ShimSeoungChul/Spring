# 01 스프링 AOP
- 비즈니스 컴포넌트 개발에서 가장 중요한 두 가지 원칙은 낮은 결합도와 높은 응집도를 유지하는 것이다. <br />
IoC가 결합도와 관련된 기능이라면, `AOP`(Aspect Oriented Programming) 는 응집도와 관련된 기능이다.

## 1.1 AOP 이해하기
### 문제점
- 엔터프라이즈 애플리케이션의 메소드에는 핵심 비즈니스 로직 뿐만 아니라 로깅, 예외, 트랜잭션 처리 같은 `부가적인 코드가` 많다. <br />

- 새로 구현되는 비즈니스 메소드마다 이런 부가적인 코드들은 반복하여 등장한다. <br />
따라서 비즈니스 메소드의 `복잡도`는 증가하고, `코드 분석`과 `유지보수`를 어렵게 만든다.


<img width="422" alt="그림 1-1 엔터프라이즈 시스템의 메소드 구성" src="https://user-images.githubusercontent.com/40673012/99316712-25e03700-28a8-11eb-92ac-4fb1885371c7.png">

### 해결
- AOP는 이런 문제를 `관심 분리`(Separation of Concerns)로 해결한다.
- AOP에서 메소드마다 공통으로 등장하는 로깅, 예외, 트랜잭션 처리 같은 코드를 `횡단 관심`(Crosscutting Concerns), <br />
사용자의 요청에 따라 실제로 수행되는 핵심 비즈니스 로직을 `핵심 관심`(Core Concerns)이라고 한다.

<img width="536" alt="그림 1-2 핵심 관심과 횡단 관심" src="https://user-images.githubusercontent.com/40673012/99316713-27116400-28a8-11eb-8e27-56b01cb124be.png">

- 이 두 관심을 완벽하게 분리하면, 새로 구현하는 메소드에는 실제 비즈니스 로직만으로 구성할 수 있으므로 <br />
더 `간결`하고, `응집도 높은` 코드를 유지할 수 있다.

- 하지만 기존의 객체지향 언어에서는 횡단 관심(공통 코드)을 완벽하게 독립적인 모듈로 분리하기 여렵다. <br />

###  문제가 있는 코드

```java
public class LogAdvice {
    public void printLog() {
        System.out.println("[공통 로그] 비즈니스 로직 수행 전 동작");
    }
}
```
```java
@Service("boardService")
public class BoardServiceImpl implements BoardService{
    
    @Autowired
    private BoardDAO boardDAO;
    private LogAdvice log;
    
    public BoardServiceImpl() {
        log = new LogAdvice();
    }

    @Override
    public void insertBoard(BoardVO vo) {
        log.printLog();
        boardDAO.insertBoard(vo);
    }

    @Override
    public void updateBoard(BoardVO vo) {
        log.printLog();
        boardDAO.updateBoard(vo);
    }

    @Override
    public void deleteBoard(BoardVO vo) {
        log.printLog();
        boardDAO.deleteBoard(vo);
    }

    @Override
    public BoardVO getBoard(BoardVO vo) {
        log.printLog();
        return boardDAO.getBoard(vo);
    }

    @Override
    public List<BoardVO> getBoardList(BoardVO vo) {
        log.printLog();
        return boardDAO.getBoardList(vo);
    }
}
```

결과
---
```
[공통 로그] 비즈니스 로직 수행 전 동작
===> JDBC로 insertBoard() 기능 처리
[공통 로그] 비즈니스 로직 수행 전 동작
===> JDBC로 getBoardList() 기능 처리
---> BoardVO [seq=2, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=1, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-06, cnt=0
```

- 위와 같은 프로그램은 BoardServiceImpl 클래스와 LogAdvice 객체가 소스코드에 강하게 결합되어있다. <br />
따라서 LogAdvice 클래스를 `다른 클래스로 변경`해야 하거나, 공통 기능인 printLog() `메소드의 시그니처가 변경`되는 상황에서 유연하게 대처할 수 없다. 
    - LogAdvice 클래스를 다른 클래스로 변경하려면, BoardServiceImpl 클래스의 생성자를 변경해야한다.
    - printLog() 메소드를 변경하려면 printLog()를 호출했던 모든 메소드를 수정해야 한다.
    
- 이처럼 객체지향 처럼 모듈화가 뛰어난 언어를 사용하여 개발해도, <br />
공통 모듈에 해당하는 클래스 객체를 생성하고 공통 메소드를 호출하는 코드가 비즈니스 메소드에 있다면, <br />
핵심 관심(BoardServiceImpl)과 횡단 관심(LogAdvice)을 완벽하게 분리할 수 없다. <br />
스프링의 AOP는 이런 OOP의 한계를 극복한다.

# 1.2 AOP 시작하기

## 1.2.1 비즈니스 클래스 수정
- 앞서 작성했던 BoardServiceImpl 클래스를 원래 상태로 되돌린다.(LogAdvice 클래스 지우기)

## 1.2.2 AOP 라이브러리 추가
```xml
<!-- AspectJ -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>${org.aspectj-version}</version>
</dependency>    
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.8</version>
</dependency>     
```
-  pom.xml 파일에 <dependency>  설정을 추가하여 aspectjweaver 내려받는다. <br />
Maven Dependencies에 라이브러리가 추가되었는지 확인한다.

<img width="491" alt="그림 1-3 AOP 관련 라이브러리 추가" src="https://user-images.githubusercontent.com/40673012/99316719-28429100-28a8-11eb-8d07-14b26e035877.png">


## 1.2.3 네임스페이스 추가 및 AOP 설정
- AOP 설정을 추가하려면 AOP에서 제공하는 엘리먼트들을 사용해야 한다. <br />
따라서 스프링 설정 파일(applicationContext.xml)에서 [Namespace]탭을 클릭하고, aop 네임스페이스를 추가한다.

<img width="408" alt="그림 1-4 aop 네임스페이스 추가" src="https://user-images.githubusercontent.com/40673012/99316721-2973be00-28a8-11eb-9e49-79b8c7a08858.png">


applicationContext.xml
---
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p" 
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

    <context:component-scan base-package="day2ch1"/>
    
    <bean id="log" class="day2ch1.LogAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* day2ch1..*Impl.*(..))"/>
        <aop:aspect ref="log">
            <aop:before pointcut-ref="allPointcut" method="printLog"/>
        </aop:aspect>
    </aop:config>
    
</beans>
```

## 테스트 및 결과 확인
- BoardServiceClient 프로그램을 실행하여 insertBoard()와 getBoardList() 메소드가 호출될때 <br />
LogAdvice 클래스의 printLog() 메소드가 실행된다.

결과
---
```
[공통 로그] 비즈니스 로직 수행 전 동작
===> JDBC로 insertBoard() 기능 처리
[공통 로그] 비즈니스 로직 수행 전 동작
===> JDBC로 getBoardList() 기능 처리
---> BoardVO [seq=4, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=3, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=2, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=1, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-06, cnt=0]
```

- LogAdvice를 Log4jAdvice로 교체하고 싶다면 BoardServiceImpl은 수정하지 않고, <br />
스프링 설정 파일의 AOp 설정만 수정하면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p" 
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd
    http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

    <context:component-scan base-package="day2ch1"/>
    
    <bean id="log" class="day2ch1.Log4jAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* day2ch1..*Impl.*(..))"/>
        <aop:aspect ref="log">
            <aop:before pointcut-ref="allPointcut" method="printLogging"/>
        </aop:aspect>
    </aop:config>
    
</beans>
```

결과
---
```xml
[공통 로그 Log4-j] 비즈니스 로직 수행 전 동작
===> JDBC로 insertBoard() 기능 처리
[공통 로그 Log4-j] 비즈니스 로직 수행 전 동작
===> JDBC로 getBoardList() 기능 처리
---> BoardVO [seq=5, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=4, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=3, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=2, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=1, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-06, cnt=0]
```

- 스프링 AOP는 클라이언트가 핵심 관심에 해당하는 비즈니스 메소드를 호출할 때, 횡단 관심에 해당하는 메소드를 적절하게 실행해준다. <br />
이때 핵심 관심 메소드와 횡단 관심 메소드 사이에 `소스상의 결합은 발생하지 않는다.` <br />
이것이 AOP를 사용하는 주된 목적이다.
