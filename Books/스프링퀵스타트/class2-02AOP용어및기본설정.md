# 02 AOP 용어 및 기본 설정
## 2.1 AOP 용어 정리

### 2.1.1 조인포인트(Joinpoint)
- 클라이언트가 호출하는 `모든 비즈니스 메소드`를 부르는 말이다. (예: BoardServiceImpl, UserServiceImpl)
- 조인포인트를 `포인트컷 대상` 또는 `포인트컷 후보`라고도 한다. 조인포인트 중에서 포인트컷이 선택되기 때문이다.

### 2.1.2 포인트컷(Pointcut)
- 필터링된 조인포인트를 의미한다.
    - 예를 들어, 트랜잭션을 처리하는 공통 기능을 만들었다고 가정하자.<br />
    이 횡단 관심 기능은 등록, 수정, 삭제 기능의 비즈니스 메소드에 대해서는 동작하지만, .<br />
    검색 기능의 메소드는 트랜잭션과 무관하므로 동작할 필요가 없다. <br />
    
- 여러개의 비즈니스 메소드 중에서 우리가 원하는 `특정 메소드`에서만 횡단 관심에 해당하는 공통 기능을 수행시키기 위해 포인트컷이 필요하다.  <br />

- 포인트컷을 이용하여 메서드가 포함된 `클래스`와 `패키지`, `메소드 시그니처`를 지정할 수 있다.
    
![그림 2-1 포인트컷으로 메소드 필터링](https://user-images.githubusercontent.com/40673012/99337686-1a036d80-28c6-11eb-918d-c2b504fdb5b2.png)

- 포인트컷을 테스트하기 위해 스프링 설정 파일에 기존에 사용하던 포인트컷을 하나 더 추가하고, <br />
<aop:before> 엘리먼트에서 allPointcut을 참조했던 부분을 getPointcut으로 수정하자.

applicationContext.xml
---
```xml
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

    <context:component-scan base-package="day2ch2"/>
    
    <bean id="log" class="day2ch1.LogAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* day2ch2..*Impl.*(..))"/>
        <aop:pointcut id="getPointcut" expression="execution(* day2ch2..*Impl.get*(..))"/>
        
        <aop:aspect ref="log">
            <aop:before pointcut-ref="getPointcut" method="printLog"/>
        </aop:aspect>
    </aop:config>
    
</beans>
```
- 포인트컷은 <aop:pointcut> 엘리먼트로 선언하며, id 속성으로 포인트컷을 식별하기 위한 유일한 문자열을 선언한다.  <br />

- 아래 그림은 expression 속성을 나타내며, 값에 따라 필터링되는 메소드가 달라진다.
![그림 2-2 포인트컷 표현식 구성 및 의미](https://user-images.githubusercontent.com/40673012/99337693-1b349a80-28c6-11eb-909e-1a48d54ffeaf.png)

- 위 applicationContext.xml 파일에서  <br />
allPointcut은 리턴타입과 매개변수를 무시하고  day2ch2 패키지로 시작하는 클래스 중 Impl로 끝나는 클래스의 모든 메소드를 포인트컷으로 설정했다. <br />
getPointcut은 allPointcut과 같지만 get으로 시작하는 메소드만 포인트컷으로 설정했다.

실행결과
---
```
===> JDBC로 insertBoard() 기능 처리
[공통 로그] 비즈니스 로직 수행 전 동작
===> JDBC로 getBoardList() 기능 처리
---> BoardVO [seq=6, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=5, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=4, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=3, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=2, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-17, cnt=0]
---> BoardVO [seq=1, title=임시 제목, writer=홍길동, content=임시 내용........., regDate=2020-11-06, cnt=0]
```

- 실행결과 insertBoard() 메소드 호출에는 반응하지 않고, get으로 시작하는 getBoardList() 메소드 호출에 대해서만 반응한다.


### 2.1.3 어드바이드(Advice)
-  어드바이스는 횡당 관심에 해당하는 공통 기능의 코드를 의미하며, 독립된 클래스의 메소드로 작성된다. <br />
이러한 어드바이스로 구현된 메소드가 언제 동작할지 스프링 설정 파일을 통해서 지정할 수 있다. <br />
ex) 트랜잭션 관리 기능을 하는 어드바이스 메소드를 비즈니스 로직 수행 후 실행하여 트랜잭션을 커밋 또는 롤백 처리한다.

- 스프링에서는 어드바이스의 동작 시점을 befor, after, after-returning, after-throwing, around 등 다섯 가지로 지정할 수 있다.


![그림 2-3 어드바이스 동작 시점](https://user-images.githubusercontent.com/40673012/99337696-1c65c780-28c6-11eb-8cf5-db259ee96dd8.png)

### 2.1.4 위빙(Weaving)
- 위빙은 포인트컷으로 지정한 핵심 관심 메소드가 호출될 때, 어드바이스에 해당하는 횡단 관심 메소드가 삽입되는 과정을 의미한다.
- 위빙을 통해 비즈니스 메소드를 수정하지 않고도 횡산 관심에 해당하는 기능을 추가, 변경할 수 있다.
- 위빙을 처리하는 방식은 1)컴파일타임 위빙, 2)로딩타임위빙, 3)런타임위빙이 있다. 스프링에서는 런타임 위빙 방식만 지원한다.


![그림 2-4 위빙(Weaving)](https://user-images.githubusercontent.com/40673012/99337698-1e2f8b00-28c6-11eb-8bd7-24bb8569d070.png)

### 2.1.5 애스팩트(Aspect) 또는 어드바이저(Advisor) 
- `애스팩트`는 포인트컷과 어드바이스의 결합으로, 어떤 포인트컷 메소드에 대해서 어떤 어드바이스 메소드를 실행할지 결정한다. <br />
 애스팩트 설정의 각 의미를 해석하면 다음과 같다. <br />
 1) getPointcut으로 설정한 포인트컷 메소드가 호출될 때,  2) log라는 어드바이스 객체의  <br />
 3) printLog() 메소드가 실행되고, 4) 이때 printLog() 메소드 동작 시점이 <aop:before>라는 내용의 설정이다. <br />

![그림 2-5 애스펙트 설정에서 속성과 의미](https://user-images.githubusercontent.com/40673012/99337705-1f60b800-28c6-11eb-8427-20ab0335cfac.png)

- 애스팩트를 설정할 때는 <aop:aspect> 엘리먼트를 사용하지만, 트랜잭션과 같은 설정에는 <aop:advisor>를 사용한다.
애스팩트와 어드바이저는 같은 의미의 용어이며, 자세한 내용은 AOP를 이용한 트랜잭션 설정에서 다룬다.

### 2.1.6 AOP 용어 종합
- 사용자는 시스템을 사용하며 비즈니스 컴포넌트의 여러 `조인포인트`를 호출한다.
이때 특정 `포인트컷`으로 지정한 메소드가 호출 되는 순간, `어드바이스` 객체의 어드바이스 메소드가 실행된다. <br />
이 어드바이스 메소드의 동작 시점을 5가지(Before, After, After-Returning, After-Throwing, Around)로 지정할 수 있으며, <br />
포인트컷으로 지정한 메소드가 호출될 때, 어드바이스 메소드를 삽입하도록 하는 설정을 `애스팩트`라고 한다.<br />
이 애스팩트 설정에 따라 `위빙`이 처리된다.

![그림 2-6 AOP 용어 정리](https://user-images.githubusercontent.com/40673012/99337708-212a7b80-28c6-11eb-9a8a-a895bbd62be7.png)

## 2.2 AOP 엘리먼트
- 스프링은 AOP 설정을 XML 방식과 어노테이션 방식으로 지원한다. XML 설정을 먼저 알아보자.

### 2.2.1 <aop:config> 엘리먼트
- AOP 설정에서 <aop:config>는 루트 엘리먼트다.
- 스프링 설정 파일 내에서 <aop:config> 엘리먼트는 여러 번 사용할 수 있으며, 하위에는 <aop:pointcut>, <aop:aspect>엘리먼트가 위치할 수 있다.

![그림 2-7 AOP 엘리먼트들의 포함 관계](https://user-images.githubusercontent.com/40673012/99337709-225ba880-28c6-11eb-935a-2d0cb066e40d.png)


### 2.2.2 <aop:pointcut> 엘리먼트
- <aop:pointcut> 엘리먼트는 포인트컷을 지정하기 위해 사용한다.
- <aop:config>의 자식이나 <aop:aspect>의 자식 엘리먼트로 사용할 수 있으며, <aop:aspect> 하위에 설정된 포인트컷은 해당 <aop:aspect>에서만 사용할 수 있다.
- <aop:pointcut>은 여러 개 정의할 수 있으며, 유일한 아이디를 할당하여 애스팩트를 설정할 때 포인트컷을 참조하는 용도로 사용한다.

```xml
<aop:config>
    <aop:pointcut id="allPointcut" expression="execution(* day2ch2..*Impl.*(..))"/>
    <aop:aspect ref="log">
        <aop:before pointcut-ref="getPointcut" method="log"/>
    </aop:aspect>
</aop:config>
```

- 위의 설정에서 allPointcut 포인트컷은 day2ch2 패키지로 시작하는 클래스 중에서 이름이 Impl로 끝나는 클래스의 모든 메소드를 포인트컷으로 설정한다. 그리고 애스팩트 설정에서 <aop:beofore> 엘리먼트의 pointcut-ref 속성으로 포인트컷을 참조한다.

### 2.2.3 <aop:aspect> 엘리먼트
- 애스팩트는 <aop:aspect> 엘리먼트로 설정하며, 핵심 관심에 해당하는 포인트컷 메소드와 횡단 관심에 해당하는 어드바이스 메소드를 결합하기 위해 사용한다.
- 에스팩트를 어떻게 설정하느냐에 따라 위빙 결과가 달라진다.

![그림 2-8 <aop aspect> 엘리먼트의 속성과 의미](https://user-images.githubusercontent.com/40673012/99337714-24256c00-28c6-11eb-92d7-47ad9e09b09f.png)

- 위 그림은 getPoint으로 설정한 포인트컷 메소드가 호출될 때, 
log라는 어드바이스 객체의 printLog() 메소드가 실행되고,
이때 printLog() 메소드 동작 시점이 <aop:before>라는 내용의 설정이다.

### 2.2.4 <aop:advisor>
- <aop:advisor> 엘리먼트는 포인트컷과 어드바이스를 결합한다는 점에서 애스팩트와 같은 기능을 한다
- 하지만 트랜잭션 설정 같은 몇몇 특수한 경우는 에스팩트가 아닌 어드바이저를 사용한다.
- AOP 설정에서 애스팩트를 사용하려면 `어드바이스 아이디와 메소드 이름`을 알아야한다. <br />
 어드바이스 객체의 아이디를 모르거나 메소드 이름을 확인할 수 없는 경우 에스팩트를 설정할 수 없다. <br />
 이럴 때 `어드바이저`를 사용한다.

![그림 2-9 어드바이스 객체의 메소드 참조](https://user-images.githubusercontent.com/40673012/99337717-25569900-28c6-11eb-8910-d48e71b0c6a2.png)

- 아래는 트랜잭션 처리 관련 설정이 포함된 스프링 설정 파일이다. 
트랜잭션 관리 어드바이스 설정이며, 어드바이스의 아이디는 txAdvice다. 하지만 어드바이스의 메소드 이름은 확인할 수 없다. <br />

```
<!--  Transaction 설정 -->
<bean id="txManager" class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean> 

<!-- Transaction advice setting -->
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
        <tx:method name="get*" read-only="true" />
        <tx:method name="*" />
    </tx:attributes>
</tx:advice>

<!-- AOP 설정을 통한 트랜잭션 적용 -->
<aop:config>
    <aop:pointcut id="txPointcut" expression="execution(* com.springbook.biz..*(..))" />
    <aop:advisor pointcut-ref="txPointcut" advice-ref="txAdvice" />
</aop:config>
```

- 스프링 컨테이너는 설정 파일에 등록된 <tx:advice> 엘리먼트를 해석하여 트랜잭션 관리 기능의 어드바이스 객체를 메모리에 생성한다.<br />

- 어떻게 <tx:advice>를 설정했느냐에 따라서 생성되는 어드바이스의 기능과 동작 방식은 달라진다.<br />
문제는 생성된 어드바이스 아이디는 확인되지만 메소드 이름은 확인할 방법이 없다는 것이다. <br />
이럴 때는 <aop:aspect>를 쓰지 못하므로 <aop:advisor> 엘리먼트를 사용해야 한다.

## 2.3 포인트컷 표현식
- 포인트컷을 이용하면 어드바이스 메소드가 적용될 비즈니스 메소드를 정확하게 필터링할 수 있다. 이때 다양한 포인트컷 표현식을 사용한다. 

- 포인트컷 표현식은 메소드처럼 생긴 execution 명시자를 이용하며, execution 명시자 안에 포인트컷 표현식을 기술한다. 


<img width="327" alt="그림 2-10" src="https://user-images.githubusercontent.com/40673012/102688121-e22a7580-4237-11eb-8f7b-995f86ff10ee.png">

- 리턴타입을 지정할 때는 '*','void', '!void' 캐릭터를 이용한다. 가장 기본적인 방법은 `*` 캐릭터를 이용하는 것이다.

|표현식|설명|
|--|---|
|*|모든 리턴타입 허용|
|void|리턴타입이 void인 메소드 선택|
|!void|리턴타입이 void가 아닌 메소드 선택|

- 패키지 경로를 지정할 때는 '*','..' 캐릭터를 이용한다. 

|표현식|설명|
|--|---|
|com.springbook.biz| com.springbook.biz 패키지만 선택|
|com.springbook.biz..|com.springbook.biz 패키지로 시작하는 모든 패키지 선택|
|com.springbook..impl|com.springbook.biz 패키지로 시작하면서 마지막 패키지 이름이 impl로 끝나는 패키지 선택|

- 클래스 이름을 지정할 때는 '*','+' 캐릭터를 이용한다. 

|표현식|설명|
|--|---|
|BoardServiceImpl|BoardServiceImpl 클래스만 선택|
|*Impl|클래스 이름이 Impl로 끝나는 클래스만 선택|
|BoardService+|클래스 이름 뒤에 '+':해당 클래스로부터 파생된 모든 자식 클래스 선택. 인터페이스 이름 뒤에 '+': 해당 인터페이스를 구현한 모든 클래스 선택 |

- 메소드를 지정할 때는 '*' 캐릭터를 사용한다.
|표현식|설명|
|--|---|
|*(..)|가장 기본 설정으로 모든 메소드 선택|
|get*(..)|메소드 이름이 get을 시작하는 모든 메소드 선택|

- 매개변수를 지정할 때는 '..', '*' 캐릭터를 사용하거나 정확한 타입을 지정한다.
|표현식|설명|
|--|---|
|(..)|가장 기본 설정으로 매개변수의 개수와 타입에 제약이 없음을 의미|
|(*)|반드시 1개의 매개변수를 가지는 메소드만 선택|
|(com.springbook.user.UserVO)|매개변수로 UserVO를 가지는 메소드만 선택. 이때 클래스와 패키지 경로가 반드시 포함.|
|(!com.springbook.user.UserVO)|매개변수로 UserVO를 가지지 않는 메소드만 선택.|
|(Inter, ...)|한 개 이상의 매개변수를 가지되, 첫 번째 매개변수의 타입이 Integer인 메소드만 선택|
|(Integer, *)|두 개의 매개변수를 가지되, 첫 번째 매개변수의 타입이 Integer인 메소드만 선택|


