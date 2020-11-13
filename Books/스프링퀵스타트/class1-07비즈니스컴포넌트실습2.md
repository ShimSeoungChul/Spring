# 07 비즈니스 컴포넌트 실습 2
- 이번 장은 회원 정보를 관리하는 UserService 컴포넌트를 개발한다. <br />
이번에는 어노테이션 없이, Setter 인젝션으로 의존성 주입을 처리하고 나서 어노테이션으로 변경하는 과정을 거친다.

## 7.1 UserService 컴포넌트 구조

![그림7-1 UserService클래스 다이어그램](https://user-images.githubusercontent.com/40673012/98322806-a80c6800-202b-11eb-8a6a-b4f1f87236bd.png)


![그림7-2 실습 파일들의 위치와 프로젝트 구조](https://user-images.githubusercontent.com/40673012/98322809-aa6ec200-202b-11eb-9c4d-d75a684662fa.png)

## 7.2 Value Object 클래스 작성

### USERS 테이블 생성
```sql
CREATE TABLE USERS(
    ID VARCHAR(8) PRIMARY KEY,
    PASSWORD VARCHAR2(8),
    NAME VARCHAR2(20),
    ROLE VARCHAR2(5)
);

INSERT INTO USERS VALUES('test', 'test123', '관리자', 'Admin');
INSERT INTO USERS VALUES('user1', 'user1', '홍길동', 'User');
```

### UserVO 클래스 작성
- USERS 테이블의 칼럼 이름과 매핑되는 멤버변수를 가진 클래스다.
```java
public class UserVO {
    private String id;
    private String password;
    private String name;
    private String role;

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getRole() {
        return role;
    }
    public void setRole(String role) {
        this.role = role;
    }

    @Override
    public String toString() {
        return "UserVO [id=" + id + ", password=" + password + ", name=" + name + ", role=" + role + ", getId()="
                + getId() + ", getPassword()=" + getPassword() + ", getName()=" + getName() + ", getRole()=" + getRole()
                + "]";
    }
}
```

## 7.3 DAO 클래스 작성
- JDBCUtil 클래스를 이용하여 UserDAO 클래스의 메소드를 구현한다.
    - 단, 회원 정보 하나를 검색하는 getUser() 메소드만 구현
```java
// DAO(Data Access Object)
public class UserDAO {
    // JDBC 관련 변수 
    private Connection conn = null;
    private PreparedStatement stmt = null;
    private ResultSet rs = null;
    
    
    // SQL 명령
    private final String USER_GET = "select * from users where id=? and password=?";

    // CRUD 기능의 메소드 구현
    // 회원 등록
    public UserVO getUser(UserVO vo) {
        UserVO user = null;
        try {
            System.out.println("===> JDBC로 getUser() 기능 처리");
            conn = JDBCUtil.getConnection();
            stmt = conn.prepareStatement(USER_GET);
            stmt.setString(1, vo.getId());
            stmt.setString(2, vo.getPassword());
            rs = stmt.executeQuery();
            if(rs.next()) {
                user = new UserVO();
                user.setId(rs.getString("ID"));
                user.setPassword(rs.getString("PASSWORD"));
                user.setName(rs.getString("NAME"));
                user.setRole(rs.getString("ROLE"));
            }
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtil.close(stmt, conn);
        }
        return user;
    }
}

```

## 7.4 Service 인터페이스 작성
- UserDAO 클래스에서 이클립스의 <Alt> + <Shift> + <T> 단축키를 이용하여 간단하게 UserService 인터페이스를 작성한다. <br />
```java
public interface UserService {
    // CRUD 기능의 메소드 구현
    // 회원 등록 
    public UserVO getUser(UserVO vo);
    
}
```

## 7.5 Service 구현 클래스 작성
- UserServiceImpl 클래스에는 Setter 인젝션 처리를 위한 Setter 메소드가 추가된다.
```java
public class UserServiceImpl implements UserService{
    private UserDAO userDAO;
    
    public void setUserDAO(UserDAO userDAO) {
        this.userDAO = userDAO;
    }
    
    public UserVO getUser(UserVO vo) {
        return userDAO.getUser(vo);
    }
}
```

## 7.6 UserService 컴포넌트 테스트
- applicationContext.xml 설정 파일에 UserServiceImpl와 UserDAO 클래스를 각각 `<bean> 등록`한다.
- 그리고 UserServiceImpl 클래스에서 UserDAO 객체를 의존성 주입하기 위한 `<property> 설정을 추가`한다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p" 
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <context:component-scan base-package="day1ch7"/>
    
    <bean id="userService" class="day1ch7.UserServiceImpl">
        <property name="userDAO" ref="userDAO" />
    </bean>
    
    <bean id="userDAO" class="day1ch7.UserDAO"/>
</beans>
```

실행결과
---
```
===> JDBC로 getUser() 기능 처리
관리자님 환영합니다.
```

## 7.7 어노테이션 적용
- 위 예제의 Setter 인젝션 설정으로 테스트한 UserService 컴포넌트를 어노테이션 설정으로 변경한다.
- 1. 스프링 설정 파일에 Setter 인젝션 관련 설정을 주석 처리한다. <br /> 
2. UserServiceImpl, UserDAO 클래스에 각각 관련된 어노테이션을 추가한다. 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:p="http://www.springframework.org/schema/p" 
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <context:component-scan base-package="day1ch7"/>
<!--     
    <bean id="userService" class="day1ch7.UserServiceImpl">
        <property name="userDAO" ref="userDAO" />
    </bean>
    
    <bean id="userDAO" class="day1ch7.UserDAO"/> -->
</beans>
```

```java
@Service("userService")
public class UserServiceImpl implements UserService{
    @Autowired
    private UserDAO userDAO;
    
    public void setUserDAO(UserDAO userDAO) {
        this.userDAO = userDAO;
    }
    
    public UserVO getUser(UserVO vo) {
        return userDAO.getUser(vo);
    }
}
```

```java
// DAO(Data Access Object)
@Repository("userDAO")
public class UserDAO {
    // JDBC 관련 변수 
    private Connection conn = null;
    private PreparedStatement stmt = null;
    private ResultSet rs = null;
    
    
    // SQL 명령
    private final String USER_GET = "select * from users where id=? and password=?";

    // CRUD 기능의 메소드 구현
    // 회원 등록
    public UserVO getUser(UserVO vo) {
        UserVO user = null;
        try {
            System.out.println("===> JDBC로 getUser() 기능 처리");
            conn = JDBCUtil.getConnection();
            stmt = conn.prepareStatement(USER_GET);
            stmt.setString(1, vo.getId());
            stmt.setString(2, vo.getPassword());
            rs = stmt.executeQuery();
            if(rs.next()) {
                user = new UserVO();
                user.setId(rs.getString("ID"));
                user.setPassword(rs.getString("PASSWORD"));
                user.setName(rs.getString("NAME"));
                user.setRole(rs.getString("ROLE"));
            }
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtil.close(stmt, conn);
        }
        return user;
    }
}

```


