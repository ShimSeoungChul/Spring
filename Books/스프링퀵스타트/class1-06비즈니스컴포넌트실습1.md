# 06 비즈니스 컴포넌트 실습 1 

- 이번 장에서는 일반적으로 프로젝트에서 사용하는 구조로 비즈니스 컴포넌트를 구현한 후, <br />
스프링의 Dependency Lookup과 Dependency Injection을 점검해보자.

## 6.1 BoardService 컴포넌트 구조

- 프로젝트마다 조금씩은 다르겠지만 일반적으로 `비즈니스 컴포넌트`는 `네 개의 자바 파일`로 구성된다
- 그림 6-1은 BOARD 테이블과 관련된 BoardService 컴포넌트에 대한 클래스 다이어그램이며, <br />
`BoardVO`, `BoardDAO`, `BoardService`, `BoardServiceImpl` 클래스로 구현되어 있다.

![그림 6-1](https://user-images.githubusercontent.com/40673012/97261490-46e1d900-1862-11eb-886c-6d759e48d702.png)

![그림 6-2](https://user-images.githubusercontent.com/40673012/97261480-43e6e880-1862-11eb-905b-fbedd5eea7fd.png)

## 6.2 Value Object 클래스 작성

- `VO`(Value Object) 클래스는 `레이어와 레이어 사이`에서 `관련된 데이터`를 `한꺼번에 주고받을 목적`으로 사용하는 클래스이다.  
- `DTO`(Data Transfer Object)라 하기도 하는데, `데이터 전달`을 목적으로 사용하는 객체이므로 결국 `같은 의미`의 용어다. <br />

- 첫째,  데이터베이스에 생성되어 있는 `테이블의 구조`를 `확인`하자.

```sql
CREATE TABLE BOARD(
    SEQ NUMBER(5) PRIMARY KEY,
    TITLE VARCHAR2(200),
    WRITER VARCHAR2(20),
    CONTENT VARCHAR2(2000),
    REGDATE DATE DEFAULT SYSDATE,
    CNT NUMBER(5) DEFAULT 0
);
```
- 둘 째, BOARD 테이블 이름 뒤에 VO나 DTO를 붙여서 `클래스 이름`으로 사용한다. <br />
그리고 BOARD 테이블에 포함된 칼럼과 같은 이름의 멤버변수를 private 접근제한자로 선언한다. 

- 마지막으로, `private 멤버변수`에 접근하여 `Getter/Setter 메소드`를 선언한다. 

```java
// VO(Value Object)
public class BoardVO {

    private int seq;
    private String title;
    private String writter;
    private String content;
    private Date regDate;
    private int cnt;

    public int getSeq() {
        return seq;
    }
    public void setSeq(int seq) {
        this.seq = seq;
    }
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    public String getWritter() {
        return writter;
    }
    public void setWritter(String writter) {
        this.writter = writter;
    }
    public String getContent() {
        return content;
    }
    public void setContent(String content) {
        this.content = content;
    }
    public Date getRegDate() {
        return regDate;
    }
    public void setRegDate(Date regDate) {
        this.regDate = regDate;
    }
    public int getCnt() {
        return cnt;
    }
    public void setCnt(int cnt) {
        this.cnt = cnt;
    }
    
    @Override
    public String toString() {
        return "BoardVO [seq=" + seq + ", title=" + title + ", writter=" + writter + ", content=" + content
                + ", regDate=" + regDate + ", cnt=" + cnt + "]";
    }
}
```
- 옵션으로 toString() 메소드도 생성하면 나중에 `VO 객체의 값을 출력`할 때 요긴하게 사용할 수 있다. <br />
이클립스에서 제공하는 `코드 제너레이션` 기능을 이용하면 Getter/Seter, toString `메소드를 자동`으로 만들 수 있다.


![그림 6-3,4](https://user-images.githubusercontent.com/40673012/97261494-48130600-1862-11eb-8027-f03d27cd7eb1.png)

- H2 설치 및 실행 방법 <br />
https://hothoony.tistory.com/890 <br />
(MAC) 1. 다운로드 2. bin 디렉토리로 이동 3. 터미널에서 h2.sh 실행(./h2.sh) 4. 접속 불가하다면, 포트 앞 주소를  localhost로 대체 <br />
ex) http://localhost:8082/login.do?jsessionid=56bbd370622a4cf17750246c59409359


## 6.3 DAO 클래스 작성
- DAO(Data Access Object) 클래스는 `데이터베이스 연동`을 담당하는 클래스이다.<br />
 따라서 DAO 클래스에는 `CRUD 기능의 메소드`가 구현되어야 한다. <br />
 
- 이때 예제에서 사용할 H2 데이터베이스에서 제공하는 `JDBC 드라이버`가 필요하다.

### 6.3.1 드라이버 내려받기

pom.xml
---
```xml
~생략~
<dependencies>
<!-- H2 데이터베이스 -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
<!-- Spring -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${org.springframework-version}</version>
~생략~
```

-  다음과 같이 H2 드라이버가 `Maven Dependencies`에 추가되었다.

<img width="242" alt="그림 6-5" src="https://user-images.githubusercontent.com/40673012/97267344-5b779e80-186d-11eb-982e-f3834e9e2449.png">

### 6.3.2 JDBC Utility 클래스
- 앞으로 예제에서 Mybatis 같은 프레임워크를 사용하기 전까지 데이터베이스 연동 처리는 JDBC로 한다. <br />
따라서 모든 DAO 클래스에서 공통으로 사용할 JDBCUtil 클래스를 작성하여 Connection 획득과 해제 작업을 공통으로 처리한다.

```java
public class JDBCUtil {
    public static Connection getConnection() {
        try {
            Class.forName("org.h2.Driver");
            return DriverManager.getConnection("jdbc:h2:tcp://localhost/~/test","sa","");
        }catch(Exception e) {
            e.printStackTrace();
        }
        return null;
    }
    
    public static void close(PreparedStatement stmt, Connection conn) {
        if(stmt != null) {
            try {    
                if(!stmt.isClosed()) stmt.close();
            }catch(Exception e) {
                e.printStackTrace();
            } finally {
                stmt = null;
            }
        }
        
        if(conn != null) {
            try {
                if(!conn.isClosed()) conn.close();
            }catch (Exception e) {
                e.printStackTrace();
            }finally {
                conn = null;
            }
        }
    }
    
    public static void close(ResultSet rs, PreparedStatement stmt, Connection conn) {
        if(rs != null) {
            try {
                if(!rs.isClosed()) rs.close();
            }catch(Exception e) {
                e.printStackTrace();
            }finally {
                rs = null;
            }
        }
        if(stmt != null) {
            try {
                if(!stmt.isClosed()) stmt.close();
            }catch(Exception e) {
                e.printStackTrace();
            }finally {
                stmt = null;
            }
        }
        if(conn != null) {
            try {
                if(!conn.isClosed()) conn.close();
            }catch(Exception e) {
                e.printStackTrace();
            }finally {
                conn = null;
            }
        }
    }
}
```

### 6.3.3 DAO 클래스 작성
```java
//DAO(Data Access Object)
@Repository("boardDAO")
public class BoardDAO {
    // JDBC 관련 변수 
    private Connection conn = null;
    private PreparedStatement stmt = null;
    private ResultSet rs = null;
    
    // SQL 명령어들
    private final String BOARD_INSERT = "insert into board(seq, title, writer, content) values((select nvl(max(seq),0)+1 from board),?,?,?)";
    private final String BOARD_UPDATE = "update board set title=?, content=? where seq=?";
    private final String BOARD_DELETE = "delete board where seq=?";
    private final String BOARD_GET = "select * from board where seq=?";
    private final String BOARD_LIST = "select * from board order by seq desc";

    // CRUD 기능의 메소드 구현
    // 글 등록
    public void insertBoard(BoardVO vo) {
        System.out.println("===> JDBC로 insertBoard() 기능 처리;");
        try {
            conn = JDBCUtil.getConnection();
            stmt = conn.prepareStatement(BOARD_INSERT);
            stmt.setString(1, vo.getTitle());
            stmt.setString(2, vo.getWritter());
            stmt.setString(3, vo.getContent());
            stmt.executeUpdate();
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtil.close(stmt, conn);
        }
    }
    
    // 글 수정
    public void updateBoard(BoardVO vo) {
        System.out.println("===> JDBC로 updateBoard() 기능 처리;");
        try {
            conn = JDBCUtil.getConnection();
            stmt = conn.prepareStatement(BOARD_UPDATE);
            stmt.setString(1, vo.getTitle());
            stmt.setString(2, vo.getContent());
            stmt.setInt(3, vo.getSeq());
            stmt.executeUpdate();
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtil.close(stmt, conn);
        }
    }
    
    //글 삭제 
    public void deleteBoard(BoardVO vo) {
        System.out.println("===> JDBC로 deleteBoard() 기능 처리;");
        try {
            conn = JDBCUtil.getConnection();
            stmt = conn.prepareStatement(BOARD_DELETE);
            stmt.setInt(1,  vo.getSeq());    
            stmt.executeUpdate();
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtil.close(stmt, conn);
        }
    }
    
    //글 상세 조회 
    public BoardVO getBoard(BoardVO vo) {
        System.out.println("===> JDBC로 getBoard() 기능 처리;");
        BoardVO board = null;
        try {
            conn = JDBCUtil.getConnection();
            stmt = conn.prepareStatement(BOARD_GET);
            stmt.setInt(1, vo.getSeq());
            rs = stmt.executeQuery();
            if(rs.next()) {
                board = new BoardVO();
                board.setSeq(rs.getInt("SEQ"));
                board.setTitle(rs.getString("TITLE"));
                board.setWriter(rs.getString("WRITER"));
                board.setContent(rs.getString("CONTENT"));
                board.setRegDate(rs.getDate("REGDATE"));
                board.setCnt(rs.getInt("CNT"));
            }
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtil.close(stmt, conn);
        }
        return board;
    }
    
    //글 목록 조회 
    public List<BoardVO> getBoardList(BoardVO vo) {
        System.out.println("===> JDBC로 getBoardList() 기능 처리;");
        List<BoardVO> boardList = new ArrayList<BoardVO>();
        try {
            conn = JDBCUtil.getConnection();
            stmt = conn.prepareStatement(BOARD_LIST);
            rs = stmt.executeQuery();
            while(rs.next()) {
                BoardVO board = new BoardVO();
                board.setSeq(rs.getInt("SEQ"));
                board.setTitle(rs.getString("TITLE"));
                board.setWriter(rs.getString("WRITER"));
                board.setContent(rs.getString("CONTENT"));
                board.setRegDate(rs.getDate("REGDATE"));
                board.setCnt(rs.getInt("CNT"));
                boardList.add(board);
            }
        }catch(Exception e) {
            e.printStackTrace();
        }finally {
            JDBCUtil.close(stmt, conn);
        }
        return boardList;
    }

}
```
- `DAO 클래스 이름`은 Board 테이블 이름 뒤에 DAO만 추가하여 사용
- 이 클래스 객체를 스프링 컨테이너가 생성할 수 있도록 클래스 선언부에 @Repository 어노테이션 추가   
- @Componet를 사용해도 문제는 없으나, 이렇게 하면 어떤 클래스가 어떤 `역할`을 수행하는지 파악하기 어려움
- CRUP 기능의 메소드 이름은 일관성 유지를 위해 대부분 다음과 같은 규칙을 적용
|기능|메소드 이름|
|--|---|
|등록|insert테이블명|
|수정|update테이블명|
|삭제|delete테이블명|
|상세 조회|get테이블명(혹은 select테이블명)|
|목록 검색|get테이블명List(혹은 select테이블명List)|

## 6.4 Service 인터페이스 작성
- DAO 클래스에서 <Alt> + <Shift> + <T> 단축키를 이용하여 BoardService 인터페이스를 작성
```java
public interface BoardService {
    public void insertBoard(BoardVO vo);
    public void updateBoard(BoardVO vo);
    public void deleteBoard(BoardVO vo);
    public BoardVO getBoard(BoardVO vo);
    public List<BoardVO> getBoardList(BoardVO vo);
}
```


## 6.5 Service 구현 클래스 작성
- BoardService 인터페이스를 구현한 BoardServiceImpl 클래스를 구현하고, <br />
BoardServiceImpl 클래스의 비즈니스 메소드를 구현하여 멤버변수로 선언된 BoardDAO를 이용
```java
@Service("boardService")
public class BoardServiceImpl implements BoardService{
    
    @Autowired
    private BoardDAO boardDAO;

    @Override
    public void insertBoard(BoardVO vo) {
        boardDAO.insertBoard(vo);
    }

    @Override
    public void updateBoard(BoardVO vo) {
        boardDAO.updateBoard(vo);
    }

    @Override
    public void deleteBoard(BoardVO vo) {
        boardDAO.deleteBoard(vo);
    }

    @Override
    public BoardVO getBoard(BoardVO vo) {
        return boardDAO.getBoard(vo);
    }

    @Override
    public List<BoardVO> getBoardList(BoardVO vo) {
        return boardDAO.getBoardList(vo);
    }
}
```
- 클래스 선언부에 객체 생성을 위한 `@Service`가 선언되어 있으며, <br />
클라이언트 프로그램에서 boardService라는 이름으로 객체를 요청할 수 있도록 `아이디`도 설정
- `데이터 베이스 연동이 포함된 비즈니스 로직 처리`를 위해 BoardDAO 타입의 객체를 멤버변수로 갖고, <br />
이 변수에 BoardDAO 타입의 객체를 의존성 주입하기 위해 변수 위에 `@Autowired` 설정

## 6.6 BoardService 컴포넌트 테스트
### 6.6.1 스프링 설정 파일 수정
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

    <context:component-scan base-package="day1ch6">
    </context:component-scan>
</beans>
```

### 6.6.2 클라이언트 작성 및 실행
- 스프링 컨테이너를 구동하고 BoardService 컴포넌트를 사용하는 클라이언트 프로그램
```java
public class BoardServiceClient {
    public static void main(String[] args) {
        //1. Spring 컨테이너를 구동한다.
        AbstractApplicationContext container = new GenericXmlApplicationContext("applicationContext.xml");
        
        //2. Spring 컨테이너로부터 BoardServiceImpl 객체를 요청(Lookup)한다.
        BoardService boardService = (BoardService) container.getBean("boardService");
        
        //3. 글 등록 기능 테스트
        BoardVO vo = new BoardVO();
        vo.setTitle("임시 제목");
        vo.setWriter("홍길동");
        vo.setContent("임시 내용.........");
        boardService.insertBoard(vo);
        
        //4. 글 목록 검색 기능 테스트
        List<BoardVO> boardList = boardService.getBoardList(vo);
        for(BoardVO board : boardList) {
            System.out.println("---> " + board.toString());
        }
        
        //5.  Spring 컨테이너 종료
        container.close();
    }
}
```
#### 주의!
- 클라이언트 프로그램 실행 전 반드시 H2 서버가 구동 중인지 확인하자



![그림 6-6](https://user-images.githubusercontent.com/40673012/97261497-49443300-1862-11eb-99af-2d88ec9ca179.png)
