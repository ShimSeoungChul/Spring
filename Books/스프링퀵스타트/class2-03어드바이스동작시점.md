# 03 어드바이스 동작 시점
- 어드바이스는 각 조인포인트에 삽입되어 동작할 횡단 관심에 해당하는 공통 기능이며, 동작시점은 각 AOP 기술마다 다르다. 스프링은 다섯 가지 동작 시점을 제공한다.

<img width="560" alt="표 3-1" src="https://user-images.githubusercontent.com/40673012/102704440-ad153600-42be-11eb-9c16-187f45a90217.png">

- 어드바이스 메소드의 동작 시점은 <aop:aspect> 엘리먼트 하위에 각각 <aop:before>, <aop:after-returning>,<aop:after-throwing>, <aop:after>, <after-around> 엘리먼트를 이용하여 지정한다.

# 3.1 Before 어드바이스

- Before 어드바이스는 포인트컷으로 지정된 메소드 호출 시, 메소드가 실행되기 전에 처리될 내용들을 기술하기 위해 사용된다.

<img width="413" alt="그림 3-1" src="https://user-images.githubusercontent.com/40673012/102688137-f8383600-4237-11eb-9cfc-3118b6d2a3cc.png">


- 먼저 비즈니스 메소드가 실행되기 전 수행될 기능을 어드바이스 클래스로 구현한다.
```
package day2ch2;

public class BeforeAdvice {
    public void beforeLog() {
        System.out.println("비즈니스 로직 수행 전 동작");
    }
}
```
- 이후 BeforeAdvice 클래스에 선언한 beforLog() 메소드가 Before 형태로 동작하도록 스프링 설정 파일을 수정한다.

```
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
    
    <bean id="before" class="day2ch2.BeforeAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* day2ch2..*Impl.*(..))"/>

        <aop:aspect ref="before">
            <aop:before pointcut-ref="allPointcut" method="beforeLog"/>
        </aop:aspect>
    </aop:config>
</beans>
```

- 어드바이스 메소드를 Before로 수행하려면 <aop:before> 엘리먼트를 사용한다. 위 설정은 allPointcut으로 지정한 모든 Impl 클래스의 메소드가 실행되지 직전에 before로 지정한 어드바이스의  beforLog() 메소드가 실행되도록 설정한 것이다.

# 3.2 After Returning 어드바이스
- After Returning 어드바이스는 포인트컷으로 지정된 메소드가 정상적으로 실행되고 나서, 메소드 수행 결과로 생성된 데이터를 리턴하는 시점에 동작한다.

<img width="403" alt="그림 3-2" src="https://user-images.githubusercontent.com/40673012/102688139-f8d0cc80-4237-11eb-8211-02a345985c33.png">

- 먼저 비즈니스 메소드가 실행되고 나서 수행될 기능을 어드바이스 클래스로 구현한다.

```
package day2ch2;

public class AfterReturningAdvice {
    public void afterLog() {
        System.out.println("비즈니스 로직 수행 후 동작");
    }
}
```
- 이후 AfterReturningAdvice 클래스에 선언한 afterLog() 메소드가 after-returning 형태로 동작하도록 스프링 설정 파일을 수정한다. 
```
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
    
    <bean id="afterReturning" class="day2ch2.AfterReturningAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="getPointcut" expression="execution(* day2ch2..*Impl.get*(..))"/>

        <aop:aspect ref="afterReturning">
            <aop:after-returning pointcut-ref="getPointcut" method="afterLog"/>
        </aop:aspect>
    </aop:config>
    
</beans>
```

- 위 설정은 XXXImpl의 모든 메소드가 실행된 이후에 afterReturningAdvice로 지정된 어드바이스 afterLog() 메소드가 실행되도록 설정한 것이다.

# 3.3 After Throwing 어드바이스
- After Throwing 어드바이스는 포인트컷으로 지정한 메소드가 실행되다가 예외가 발생하는 시점에 동작한다.

<img width="413" alt="그림 3-3" src="https://user-images.githubusercontent.com/40673012/102688134-f66e7280-4237-11eb-9cd1-57fe9048d6d8.png">

- 먼저 지정한 메소드가 예외 발생시 수행될 기능을 어드바이스 클래스로 구현한다.
```
package day2ch2;

public class AfterThrowingAdvice {
    public void exceptionLog() {
        System.out.println("비즈니스 로직 수행 중 예외 발생");
    }
}   
```

- 이후 AfterThrowingAdvice 클래스에 선언한 exceptionLog() 메소드가 after-throwing 형태로 동작하도록 스프링 설정 파일을 수정한다.
```
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

     <context:component-scan base-package="day2ch2"/>
    
    <bean id="afterThrowing" class="day2ch2.AfterThrowingAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* day2ch2..*Impl.*(..))"/>

        <aop:aspect ref="afterThrowing">
            <aop:after-returning pointcut-ref="allPointcut" method="exceptionLog"/>
        </aop:aspect>
    </aop:config>
    
</beans>    
```

-  After Thrwoing 어드바이스는 <aop:after-throwing> 엘리먼트를 이용하여 설정한다. 위 설정은 allPointcut이라는 포인트컷으로 지정한 메소드에서 예외가 발생할 경우, afterThrowing 어드바이스의 exceptionLog() 메소드를 실행하기 위한 설정이다

- 마지막으로 BoardServiceImple 클래스에 예외가 발생하는 코드를 추가한다.



# 3.4 After 어드바이스
- try-catch-finally 구문에서 finally 블록처럼 예외 발생 여부에 상관없이 무조건 수행되는 어드바이스를 등록할 때 After 어드바이스를 사용한다.

<img width="440" alt="그림3-4" src="https://user-images.githubusercontent.com/40673012/102688136-f7070900-4237-11eb-9f22-32a1d4801ee5.png">

- 먼저 지정한 메소드 수행 후 무조건 수행되는 기능을 어드바이스 클래스로 구현한다.

```
package day2ch2;

public class AfterAdvice {
    public void finallyLog() {
        System.out.println("비즈니스 로직 수행 후 무조건 동작");
    }
}
```
- After 어드바이스를 <aop:after> 엘리먼트를 이용하여 설정한다. 

```
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

     <context:component-scan base-package="day2ch2"/>
    
    <bean id="afterThrowing" class="day2ch2.AfterThrowingAdvice"></bean>
    <bean id="after" class="day2ch2.AfterAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* day2ch2..*Impl.*(..))"/>

        <aop:aspect ref="afterThrowing">
            <aop:after-returning pointcut-ref="allPointcut" method="exceptionLog"/>
        </aop:aspect>
        
        <aop:aspect>
            <aop:after pointcut-ref="allPointcut" method="finallyLog"/>
        </aop:aspect>
    </aop:config>
    
</beans>    
```

# 3.5 Around 어드바이스
- 하나의 어드바이스가 비즈니스 메소드 실행 전과 후에 모두 동작하여 로직을 처리하는 경우 Around 어드바이스를 사용한다.

<img width="503" alt="그림 3-5" src="https://user-images.githubusercontent.com/40673012/102688130-f2daeb80-4237-11eb-895a-da6af77c8481.png">

- 먼저 Around 어드바이스 클래스를 작성한다. 

```
package day2ch2;

import org.aspectj.lang.ProceedingJoinPoint;

public class AroundAdvice {
    public Object aroundLog(ProceedingJoinPoint pjp) throws Throwable{
        
        System.out.println("비즈니스 메소드 수행 전에 처리할 내용");
        Object returnObj = pjp.proceed();
        System.out.println("비즈니스 에소드 수행 후에처리할 내용");

        return returnObj;
    }
}
    
```

- Around 어드바이스는 클라이언트의 메소드 호출을 가로챈다. 그래서 클라이언트가 호출한 비즈니스 메소드 실행 전후에 로직을 수행할 수 있다. 클라이언트의 요청을 가로챈 어드바이스는 클라이언트가 호출한 비즈니스 메소드를 호출하기 위해서 ProceedingJoinPoint 객체를 매개변수로 받아야 한다.

- ProceedingJoinPoint 객체의 proceed() 메소드로 비즈니스 메소드를 호출할 수 있다. pjp.proceed() 메소드 호출 앞에 작성된 코드는 Before 어드바이스와 동일하게 동작하며, pjp.proceed() 메소드 호출 뒤에 작성된 코드는 After 어드바이스와 동일하게 동작한다.

- 이제 Around 어드바이스를 적용하기 위해 스프링 설정 파일에 애스팩트 설정을 추가한다.
```
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

     <context:component-scan base-package="day2ch2"/>
    
    <bean id="around" class="day2ch2.AroundAdvice"></bean>
    
    <aop:config>
        <aop:pointcut id="allPointcut" expression="execution(* day2ch2..*Impl.*(..))"/>

        <aop:aspect ref="around">
            <aop:around pointcut-ref="allPointcut" method="aroundLog"/>
        </aop:aspect>
    </aop:config>
    
</beans>    
```

- Around 어드바이스에 대한 애스팩트 설정은 <aop:around> 엘리먼트를 사용한다.


