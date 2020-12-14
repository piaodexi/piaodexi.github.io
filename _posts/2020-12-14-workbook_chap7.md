---
layout: post
---

# 7장 퍼시스턴스 프레임워크의 도입
퍼시스턴스 프레임워크(persistence framework)를 사용하면  
직접 JDBC API를 호출하지 않고도 데이터베이스에 있는 데이터를 다룰 수 있습니다.  
정확하게는 개발자 대신 퍼시스턴스 프레임워크가 JDBC API를 호출하는 것입니다.


> 퍼시스턴스(Persistence)  
'퍼시스턴스'라는 용어는 개발 분야에서 많이 사용하는 용어입니다. 퍼시스턴스는 데이터의 지속성을 의미합니다.  
즉 애플리케이션을 종료하고 다시 실행하더라도 이전에 저장한 데이터를 다시 불러올 수 있는 기술입니다.  
퍼시스턴스 프레임워크는 데이터의 저장,조회,변경,삭제를 다루는 클래스 및 설정 파일들의 집합입니다.  

> 프레임워크(Framework)  
'라이브러리(library)'가 개발에 필요한 도구들을 단순히 나열해 놓은 것이라면, '프레임워크(frame-work)'는  
동작에 필요한 구조를 어느 정도 완성해 놓은 반제품 형태의 도구입니다. 프레[임워크를 사용하면 약간의 학습만으로  
안정적인 시스템을 빠르게 개발할 수 있습니다.

## 퍼시스턴스 프레임워크  
퍼시스턴스 프레임워크를 사용하면 JDBC 프로그래밍의 복잡함이나 번거로움 없이 간단한 작업만으로 데이터베이스와 연동되는 시스템을 빠르게 개발할 수 있습니다. 

> 퍼시스턴스 프레임워크에는 SQL 문장으로 직접 DB데이터를 다루는 'SQL 맵퍼(mapper)'와 자바객체를 통해 간접적으로 DB데이터를 다루는 '객체 관계 맵퍼(object-relational mapper)'가 있다.
- sql 맵퍼에는 mybatis(이전버전은 아이바티스)
- 객체 관계 맵퍼에는 하이버네이트(Hibernate)와 탑링크(toplink)가 있습니다.'

## 객체 관계 맵퍼

> 객체 관계 맵퍼는 프레임워크에서 제공하는 API와 전용 ㅐㄱㄱ체 질의어를 사용하여 데이터를 다룹니다. 프레임워크에서 제공하는 객체 질의어는 sql보다는 단순합니다.하이버네이트는  
HQL(Hibernate query language)'이라는 객체 질의어를 제공합니다. 객체 질의어를 사용하면 sql를 몰라도 되기 때문에 개발자의 부담은 줄어듭니다. 실행시에 dbms에 맞추어 sql 문을 자동 생성하기  
때문에 특정 dbms 종속되지 않은 애플리케이션을 만들 수 있습니다.

## 객체 관계 맵퍼의 한계

- 유지보수가 힘듬 
- 데이터베이스의 특징에 맞추어 최적화를 할 수 가 없음(sql문을 직접 작성하지 않기 때문에..)

# mybatis 소개 
mybatis의 핵심은 개발과 유지보수가 쉽도록 소스 코드에 박혀있는 sql을 별도의 파일로 분리하는 것이고  
단순하고 반복적인 jdbc 코드를 캡슐화하여 데이터베이스 프로그래밍을 간결하게 만드는 것

![](./7_1.PNG) 

## mybatis 사용하기 
아래 그림은 dao에서 mybatis를 사용하는 시나리오입니다. 

![](./7.2.PNG)

1. 데이터 처리를 위해 dao는 mybatis에 제공하는 객체의 메서드를 호출합니다.  
2. mybatis는 sql 문이 저장된 맵퍼 아리에서 데이터 처리에 필요한 sql 문을 찾습니다.
3. mybatis는 맵퍼 파일에서 찾은 sql을 서버에 보내고자 jdbc 드라이버를 사용합니다.  
4. jdbc 드라이버는 sql문을 데이터베이스 서버로 보냅니다.  
5. mybatis는 select문의 실행 결과를 값 객체에 담아서 반환합니다. insert, update,delete문인 경우 입력,변경, 삭제된 레코드의 개수를 반환합니다. 

## mybatis 구동하기  
아래 그림은 mybatis를 이용하여 프로젝트 목록을 가져오는 시나리오입니다.

![](./7_3.PNG)

## mybatis 프레임워크의 핵심 컴포넌트
> 프레임워크를 사용할 때는 그 프레임워크의 핵심을 담당하는 컴포너트를 먼저 파악하는 것이 중요 
아래는 mybatis의 핵심 컴포넌트를 정리를 한 표 

컴포넌트  | 설명  
------------- | -------------   
SqlSession          |실제 SQL을 실행하는 객체. 이 객체는 SQL을 처리하기 위해 JDBC드라이버를 사용함. 
SqlSessionFactory             | sqlSession 객체를 생성함.  
SqlSessionFactoryBuilder     | 디비 설정 파일의 내용을 토대로 SqlSessionFactory를 생성함.               
mybatis 설정 파일    | 데이터페이스 연결 정보, 트랜잭션 정보, mybatis 제어 정보 등의 설정 내용을 포함하고 있음. SqlSessionFactory를 만들 때 사용됨.               
SQL 맵퍼 파일   | SQL 문을 담고 있는 파일. SqlSession 객체가 참조함


**dao에서 sqlsessionfactory 사용**
```java
package spms.dao;
 
import java.util.List;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
 
import spms.annotation.Component;
import spms.vo.Project;
 
 
 
@Component("projectDao")
public class MySqlProjectDao implements ProjectDao {
    //mybatis를 사용하면서 DataSource는 필요가 없어졌다.
    //SQL을 실행할 때 사용할 도구를 만들어주는 SqlSessionFactory를 만든다.
    SqlSessionFactory sqlSessionFactory;
     
    //SqlSession을 저장할 인스턴스 변수와 setter 메서드를 선언한다.
    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory){
        this.sqlSessionFactory = sqlSessionFactory;
    }
 
    /*DataSource ds;
 
    public void setDataSource(DataSource ds) {
        this.ds = ds;
    }*/
 
    public List<Project> selectList() throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try{
            //sql SQl을 실행하는 도구이다.
            //SQLID는 SQL 맵퍼의 네임스페이스 이름 + SQL문 ID
            //ex) "spms.dao.ProjectDao" + "selectList"
            return sqlSession.selectList("spms.dao.ProjectDao.selectList");
        }finally{
            //SqlSession 객체 또한 DB커넥션처럼 사용을 완료한 후에는,
            //닫아야 한다.
            sqlSession.close();
        }
        /*Connection connection = null;
        Statement stmt = null;
        ResultSet rs = null;
 
        try {
            connection = ds.getConnection();
            stmt = connection.createStatement();
            rs = stmt
                    .executeQuery("SELECT PNO,PNAME,STA_DATE,END_DATE,STATE" + " FROM PROJECTS" + " ORDER BY PNO DESC");
 
            ArrayList<Project> projects = new ArrayList<Project>();
 
            while (rs.next()) {
                projects.add(new Project().setNo(rs.getInt("PNO")).setTitle(rs.getString("PNAME"))
                        .setStartDate(rs.getDate("STA_DATE")).setEndDate(rs.getDate("END_DATE"))
                        .setState(rs.getInt("STATE")));
            }
 
            return projects;
 
        } catch (Exception e) {
            throw e;
 
        } finally {
            try {
                if (rs != null)
                    rs.close();
            } catch (Exception e) {
            }
            try {
                if (stmt != null)
                    stmt.close();
            } catch (Exception e) {
            }
            try {
                if (connection != null)
                    connection.close();
            } catch (Exception e) {
            }
        }*/
    }
 
    public int insert(Project project) throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try{
            int count = sqlSession.insert("spms.dao.ProjectDao.insert", project);
            sqlSession.commit();
            return count;
        }finally{
            sqlSession.close();
        }
    }
 
    public Project selectOne(int no) throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try{
            return sqlSession.selectOne("spms.dao.ProjectDao.selectOne", no);
        }finally{
            sqlSession.close();
        }
    }
 
    public int update(Project project) throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try{
            int count = sqlSession.update("spms.dao.ProjectDao.update", project);
            sqlSession.commit();
            return count;
        }finally{
            sqlSession.close();
        }
    }
 
    public int delete(int no) throws Exception {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try{
            int count = sqlSession.delete("spms.dao.ProjectDao.delete", no);
            sqlSession.commit();
            return count;
        }finally{
            sqlSession.close();
        }
    }
}

```

**sqlseesion의 주요 메서드**

메서드  | 설명 
------------- | -------------    
selectList()           |SELECT문을 실행, 값 객체(VO) )목록을 반환함. 
selectOne()            | SELECT문을 싷행, 하나의 값 객체를 반환   
insert()    |INSERT문을 싷행, 반환값은 입력한 데이터 개수               
update()     |UPDATE문을 싷행, 반환값은 변경한 데이터 개수   
delete()    | DELETE문을 실행. 반환값은 삭제한 데이터 개수 


> 전달된 'project' 객체는 SQL 맵퍼 파일을 실행할 때 사용된다.
SQL맵퍼 파일에서 #{프로퍼티명} 자리에 project 객체의 프로퍼티 값이 놓인다.  
(객체의 프로퍼티란 : 프로퍼티 이름은 getter/setter 메서드의 이름에서 추출한다.)  
SqlSession의 selectOne()과 delete()에 Integer 객체 전달시에 int와 같은 기본 데이터 형은 객체가 아니므로 매개변수로 사용할 수 없지만  
컴파일 시 Integer 객체로 오토박싱되기 때문에 오류가 발생하지 않는다.  
즉 new Integer(no) 코드로 변한다.


**commit()과 rollback() 메서드**  
update, insert에는 "sqlSession.commit();"이 있는데 SELECT문은 값을 변경하는 것이 아니므로  
commit() 메서드를 호출할 필요가 없다.

**자동 커밋**
> 자동 커밋을 하고 싶다면 SqlSession 객체를 생성할 때 다음과 같이 지정한다.
SqlSession sqlSession = sqlSessionFactory.openSession(true);
openSession()의 매개변수 값을 true로 지정하면 자동 커밋을 수행하는 SqlSession객체를 반환한다. 하지만 트랜잭셩을 다룰 수 없다.


**SQL맵퍼 파일 작성(이후에 설명)**
```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="spms.dao.ProjectDao">
  <resultMap type="project" id="projectResultMap">
    <id column="PNO" property="no"/>
    <result column="PNAME"    property="title"/>
    <result column="CONTENT"  property="content"/>
    <result column="STA_DATE" property="startDate" javaType="java.sql.Date"/>
    <result column="END_DATE" property="endDate" javaType="java.sql.Date"/>
    <result column="STATE"    property="state"/>
    <result column="CRE_DATE" property="createdDate" javaType="java.sql.Date"/>
    <result column="TAGS"     property="tags"/>
  </resultMap>
   
  <select id="selectList" resultMap="projectResultMap">
      select PNO, PNAME, STA_DATE, END_DATE, STATE
      from PROJECTS
      order by PNO desc
  </select>
   
  <insert id="insert" parameterType="project">
    insert into PROJECTS(PNAME,CONTENT,STA_DATE,END_DATE,STATE,CRE_DATE,TAGS)
    values (#{title},#{content},#{startDate},#{endDate},0,now(),#{tags})
  </insert>
   
  <select id="selectOne" parameterType="int" resultMap="projectResultMap">
    select PNO, PNAME, CONTENT, STA_DATE, END_DATE, STATE, CRE_DATE, TAGS
    from PROJECTS
    where PNO=#{value}
  </select>
   
  <update id="update" parameterType="project">
    update PROJECTS set
      PNAME=#{title},
      CONTENT=#{content},
      STA_DATE=#{startDate},
      END_DATE=#{endDate},
      STATE=#{state},
      TAGS=#{tags}
    where PNO=#{no}
  </update>
   
  <delete id="delete" parameterType="int">
    delete from PROJECTS
    where PNO=#{value}
  </delete>
</mapper>
```
**ApplicationContext변경**

> 기존에는 프로퍼티 파일과 어노테이션을 통해 객체를 생성하였지만,  
SqlSessionFactory는 이런 방식으로 생성할수 없다.  
별도로 SqlSessionFactory를 준비하여 ApplicationContext에 등록해야 한다.  
```java
package spms.context;
 
import java.io.FileReader;
import java.lang.reflect.Method;
import java.util.Hashtable;
import java.util.Properties;
import java.util.Set;
 
import javax.naming.Context;
import javax.naming.InitialContext;
 
import org.reflections.Reflections;
 
import spms.annotation.Component;
 
// 프로퍼티 파일을 이용한 객체 준비
public class ApplicationContext {
    // 프로퍼티에 설정된 대로 객체를 준비하면, 객체를 저장할 보관소가 필요한데 이를 위해 해시 테이블을 준비한다.
    Hashtable<String, Object> objTable = new Hashtable<String, Object>();
 
    // 해시테이블에서 객체를 꺼낼 getter 메서드도 정의한다.
    public Object getBean(String key) {
        return objTable.get(key);
    }
 
    public void addBean(String name, Object obj) {
        objTable.put(name, obj);
    }
 
    // 외부에서 객체를 주입하는 경우도 고려해야 하기 때문에 일괄처리 방식을 개별처리 방식으로 변경해야 한다. 그래서 기존 생성자 제거
    /*
     * public ApplicationContext(String propertiesPath) throws Exception {
     * //Properties는 '이름=값' 형태로 된 파일을 다룰 때 사용하는 클래스이다. Properties props = new
     * Properties();
     *
     * //Properties의 load() 메서드는 FileReader를 통해 읽어들인 프로퍼티 내용을 키-값 형태로 내부 맵에
     * 보관한다. props.load(new FileReader(propertiesPath));
     *
     * prepareObjects(props); prepareAnnotationObjects(); injectDependency(); }
     */
 
    // 자바 classpath를 뒤져서 @Component 애노테이션이 붙은 클래스를 찾는다. 그리고 그 객체를 생성하여 객체 테이블에
    // 담는 일을 한다.
    // Reflections라는 오픈소스 라이브러리를 활용한다.
    /* private void prepareAnnotationObjects() throws Exception{ */
    public void prepareObjectsByAnnotation(String basePackage) throws Exception {
        Reflections reflector = new Reflections(basePackage);
 
        Set<Class<?>> list = reflector.getTypesAnnotatedWith(Component.class);
        String key = null;
        for (Class<?> clazz : list) {
            key = clazz.getAnnotation(Component.class).value();
            objTable.put(key, clazz.newInstance());
        }
    }
 
    // 프로퍼티 파일의 내용을 로딩했으면 객체를 준비해야한다. prepareObjects()
    /* private void prepareObjects(Properties props) throws Exception { */
    public void prepareObjectsByProperties(String propertiesPath) throws Exception {
        Properties props = new Properties();
        props.load(new FileReader(propertiesPath));
 
        Context ctx = new InitialContext();
        String key = null;
        String value = null;
 
        for (Object item : props.keySet()) {
            key = (String) item;
            value = props.getProperty(key);
            if (key.startsWith("jndi.")) {
                objTable.put(key, ctx.lookup(value));
            } else {
                objTable.put(key, Class.forName(value).newInstance());
            }
        }
    }
 
    // 톰캣 서버로부터 객체를 가져오거나(ex: DataSource) 직접 객체를 생성했으면(ex: MemberDao), 각 객체가 필요로
    // 하는 의존 객체를 할당한다.
    public void injectDependency() throws Exception {
        for (String key : objTable.keySet()) {
            // "jndi."로 시작하는 것 제외
            if (!key.startsWith("jndi.")) {
                callSetter(objTable.get(key));
            }
        }
    }
 
    // 매개변수로 주어진 객체에 대해 setter 메서드를 찾아서 호출하는 일을 한다.
    private void callSetter(Object obj) throws Exception {
        Object dependency = null;
        for (Method m : obj.getClass().getMethods()) {
            // setter 메서드를 찾는다.
            if (m.getName().startsWith("set")) {
                // setter메서드의 매개변수와 타입이 일치하는 객체를 objTable에서 찾는다.
                dependency = findObjectByType(m.getParameterTypes()[0]);
                // 의존 객체를 찾았으면, setter메서드를 호출한다.
                if (dependency != null) {
                    m.invoke(obj, dependency);
                }
            }
        }
    }
 
    // setter 메서드를 호출할 때 넘겨줄 의존 객체를 찾는 일을 한다. objTable에 들어 있는 객체를 모두 찾는다.
    private Object findObjectByType(Class<?> type) {
        for (Object obj : objTable.values()) {
            // 주어진 객체가 해당 클래스 또는 인터페이스의 인스턴스인지 찾고 그 객체의 주소를 리턴한다.
            if (type.isInstance(obj)) {
                return obj;
            }
        }
        return null;
    }
}
```
**SqlSessionFactory 객체 준비**
ContextLoaderListener.java

```java
package spms.listeners;
 
// SqlSessionFactory 객체 준비
import java.io.InputStream;
 
import javax.servlet.ServletContext;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;
 
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
 
import spms.context.ApplicationContext;
 
@WebListener
public class ContextLoaderListener implements ServletContextListener {
    static ApplicationContext applicationContext;
 
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
 
    @Override
    public void contextInitialized(ServletContextEvent event) {
        try {
            //ApplicationContext 객체를 생성할 때 기본 생성자를 호춣하도록 코드를 변경
            applicationContext = new ApplicationContext();
 
            //SqlSessionFactoryBuilder 클래스의 build()를 호출해야만 SqlSessionFactory 객체를 생성할 수 있다.
            String resource = "spms/dao/mybatis-config.xml";
            //mybatis에서 제공하는 Resources 클래스ㅇ를 사용하면 자바 클래스 경로에 있는 파일의 입력 스트림을 쉽게 얻을 수 있다.
            InputStream inputStream = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
 
            //SqlSessionFactory 객체를 ApplicationContext에 등록한다.
            applicationContext.addBean("sqlSessionFactory", sqlSessionFactory);
 
            //프로퍼티 파일(aqpplication-context.properties) 경로를 알아내기
            ServletContext sc = event.getServletContext();
            String propertiesPath = sc.getRealPath(sc.getInitParameter("contextConfigLocation"));
 
            //프로퍼티 파일의 내용에 따라 객체를 생성하도록 ApplicationContext에 지시한다.
            applicationContext.prepareObjectsByProperties(propertiesPath);
 
            //@Component 어노테이션이 붙은 클래스를 찾아 객체를 생성한다.
            applicationContext.prepareObjectsByAnnotation("");
 
            //ApplicationContext에서 관리하는 객체들을 조사하여 의존 객체를 주입한다.
            applicationContext.injectDependency();
 
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
 
    @Override
    public void contextDestroyed(ServletContextEvent event) {
    }
}
```
**mybatis 설정 파일 준비**  
**db.properties 파일 작성**  
```java
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost/studydb
username=study
password=study
```

## SQL 맵퍼 파일

> mybatis의 가장 중요한 목적 중 하나가 DAO로부터 SQL문을 분리하는 것이다. 이렇게 분리된 SQL문은 SqlSession에서 사용한다.

```java
<!-- xml 선언 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<!-- mapper태그는 SQL문을 묶는 용도로 사용한다. -->
<mapper namespace="spms.dao.ProjectDao">
    <!-- resultMap 엘리먼트를 사용하면 칼럼 이름과 setter이름의 불일치 문제를 해결할 수 있다. -->
    <resultMap type="project" id="projectResultMap">
        <!-- mybatis는 SELECT결과를 캐싱하는데 이를 위해 id는 객체 식별자로 사용 -->
        <id column="PNO" property="no" />
         
        <!-- result 엘리먼트는 resultMap의 자식 태그로서 칼럼과 setter 메서드의 연결을 정희한다. -->
        <!-- column 속성에는 칼럼 이름을 지정하고, property 속성에는 객체의 프로퍼티 이름을 지정한다. -->
        <result column="PNAME" property="title" />
        <result column="CONTENT" property="content" />
        <!-- javaType을 사용하면 특정 자바 객체로 변환할 수 있다. -->
        <result column="STA_DATE" property="startDate" javaType="java.sql.Date" />
        <result column="END_DATE" property="endDate" javaType="java.sql.Date" />
        <result column="STATE" property="state" />
        <result column="CRE_DATE" property="createdDate" javaType="java.sql.Date" />
        <result column="TAGS" property="tags" />
    </resultMap>
 
    <!-- select, insert, update, delete 엘리먼트 / resultMap에는 클래스 이름(패키지 이름 포함)이 온다. -->
    <!-- mybatis는 resultType에 선언된 클래스의 인스턴스를 생성한다. 그리고 각 칼럼에 대응하는 setter 메서드를 찾아서 호출한다. -->
    <select id="selectList" resultMap="projectResultMap">
        select PNO, PNAME, STA_DATE, END_DATE, STATE
        from PROJECTS
        order by PNO desc
    </select>
 
    <!-- 입력 매개변수는 #{프로퍼티명}으로 표시  -->
    <insert id="insert" parameterType="project">
        insert into PROJECTS(PNAME,CONTENT,STA_DATE,END_DATE,STATE,CRE_DATE,TAGS)
        values (#{title},#{content},#{startDate},#{endDate},0,now(),#{tags})
    </insert>
 
    <select id="selectOne" parameterType="int" resultMap="projectResultMap">
        select PNO, PNAME, CONTENT, STA_DATE, END_DATE, STATE, CRE_DATE, TAGS
        from PROJECTS
        where PNO=#{value}
    </select>
 
    <!-- project 객체안에 입력 매개 변수들이 있음 -->
    <update id="update" parameterType="project">
        update PROJECTS set
        PNAME=#{title},
        CONTENT=#{content},
        STA_DATE=#{startDate},
        END_DATE=#{endDate},
        STATE=#{state},
        TAGS=#{tags}
        where PNO=#{no}
    </update>
 
    <!-- 기본타입인 경우 -->
    <delete id="delete" parameterType="int">
        delete from PROJECTS
        where PNO=#{value}
    </delete>
</mapper>
```
## 랩퍼 클래스

랩퍼 클래스(Wrapper Class)란,  
자바의 기본 데이터형 값을 객체로 다루기 위해 기본으로 제공되는 클래스  

datee type|   wrapper class
------------- | -------------   
byte         | Byte
short         | Short
int     | Integer            
long   | Long           
float  | Float     
double | Double
char | Character
boolean | Boolean

# 7.4 mybatis 설정 파일

> mybatis 프레임워크는 자체 커넥션풀을 구축할 수 있다.  
또한, 여러 개의 데이터베이스 연결 정보를 설정해 두고 실행 상황(개발, 테스트, 운영 등)에 따라 사용할 DB를 지정할 수 있다.  
실행 성능을 높이기 위해 Select 결과를 캐싱해 두기도 하고,  
SQL 맵퍼 파일에서 사용할 값 객체에 대해 별명을 부여할 수 있다.  

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
 
<!-- mybatis 설정 파일의 루트 엘리먼트 -->
<configuration>
    <!-- 데이터베이스 연결 정보처럼 자주 변경될 수 있는 값은 mybatis 설정 파일에 두지 않고 보통 프로퍼티 파일에 저장한다. -->
    <!-- 그리고 그 프로퍼티 파일을 로딩하려면 properties 태그를 사용한다. 다른경로에 있다면 url 속성을 사용. -->
    <properties resource="spms/dao/db.properties" />
 
    <!-- SQL 맵퍼파일에서 parameterType, resultType을 지정할 때 별명을 사용할수 있다. -->
    <typeAliases>
        <typeAlias type="spms.vo.Project" alias="project" />
        <typeAlias type="spms.vo.Member" alias="member" />
    </typeAliases>
 
    <!-- 데이터베이스 환경 정보를 설정할 때 사용하는 태그이고 이 태그를 이용하면 여러 개의 DB접속 정보를 설정할 수 있다. -->
    <environments default="development">
        <!-- 트랜잭션 관리 및 데이터 소스를 설정하는 태그이다. -->
        <environment id="development">
            <!-- mybatis의 트랜잭션 관리 방식 2가지 JDBC, MANAGED -->
            <transactionManager type="JDBC" />
             
            <!-- 프로퍼티는 "개별 프로퍼티 설정-프로퍼티 파일에 정의된 것 외에 추가로 정의 할 수 있다. p604", "프로퍼티 값 참조" 방법이 있다. -->
            <!-- 프로퍼티 값 참조 / 프로퍼티 파일에 저장된 값은 ${} 형식으로 찹조한다. -->
            <!-- 데이터 소스 설정 UNPOOLED, POOLED, JNDI -->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}" />
                <property name="url" value="${url}" />
                <property name="username" value="${username}" />
                <property name="password" value="${password}" />
            </dataSource>
        </environment>
    </environments>
 
    <!-- SQL 맵퍼 파일의 경로를 설정할 때 사용 -->
    <mappers>
        <mapper resource="spms/dao/MySqlProjectDao.xml" />
    </mappers>
</configuration>
```

**mybatis에서 JNDI 사용하기**

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
 
<!-- mybatis 설정 파일의 루트 엘리먼트 -->
<configuration>
    <!-- 데이터베이스 연결 정보처럼 자주 변경될 수 있는 값은 mybatis 설정 파일에 두지 않고 보통 프로퍼티 파일에 저장한다. -->
    <!-- 그리고 그 프로퍼티 파일을 로딩하려면 properties 태그를 사용한다. 다른경로에 있다면 url 속성을 사용. -->
    <!-- <properties resource="spms/dao/db.properties" /> -->
 
    <!-- SQL 맵퍼파일에서 parameterType, resultType을 지정할 때 별명을 사용할수 있다. -->
    <typeAliases>
        <typeAlias type="spms.vo.Project" alias="project" />
        <typeAlias type="spms.vo.Member" alias="member" />
    </typeAliases>
 
    <!-- 데이터베이스 환경 정보를 설정할 때 사용하는 태그이고 이 태그를 이용하면 여러 개의 DB접속 정보를 설정할 수 있다. -->
    <environments default="development">
        <!-- 트랜잭션 관리 및 데이터 소스를 설정하는 태그이다. -->
        <environment id="development">
            <!-- mybatis의 트랜잭션 관리 방식 2가지 JDBC, MANAGED -->
            <transactionManager type="JDBC" />
             
            <!-- 프로퍼티는 "개별 프로퍼티 설정-프로퍼티 파일에 정의된 것 외에 추가로 정의 할 수 있다. p604", "프로퍼티 값 참조" 방법이 있다. -->
            <!-- 프로퍼티 값 참조 / 프로퍼티 파일에 저장된 값은 ${} 형식으로 찹조한다. -->
            <!-- 데이터 소스 설정 UNPOOLED, POOLED, JNDI -->
            <!-- <dataSource type="POOLED">
                <property name="driver" value="${driver}" />
                <property name="url" value="${url}" />
                <property name="username" value="${username}" />
                <property name="password" value="${password}" />
            </dataSource> -->
            <dataSource type="JNDI">
                <property name="data_source" value="java:comp/env/jdbc/studydb"/>
            </dataSource>
        </environment>
    </environments>
 
    <!-- SQL 맵퍼 파일의 경로를 설정할 때 사용 -->
    <mappers>
        <mapper resource="spms/dao/MySqlProjectDao.xml" />
    </mappers>
</configuration>

```

# 7.5 로그 출력 시키기
> mybatis의 로그 출력 기능을 켜며 mybatis에서 실행하는 SQL 문과 매개변수 값, 실행 결과를 실시간으로 확인할 수 있다. 디버깅을 할 때 매우 유용하다.

mybatis 설정 파일에 로그 설정 추가  

mybatis-config.xml 일부분  


```
<configuration>
    <!-- 로그 출력 기능을 킨다. -->
    <settings>
        <!-- name속성과 로그출력기 이름 설정  -->
        <setting name="logImpl" value="LOG4J" />
    </settings>
```

로그 출력 구현체  

Log4J 외에 value에 넣을 수 있는 값들  

Log4J 라이브러리 파일 준비  

Log4J 설정 파일 작성  

로그의 수준, 출력 방식, 출력 형식, 로그 대상 등에 대한 정보가 들어간다.  

log4j.properties

```
# 출력 담당자 선언 / 여기서는 루트 로거(최상위 로거, 하위로거들은 항상 부모의 출력등급을 상속받는다.)의 출력 등급을 ERROR로 설정
log4j.rootLogger=ERROR, stdout
 
# 하위 로거들은 등급을 별도로 설정하지 않으면 루트 로거의 출력 등급과 같은 ERROR가 되지만
# spms.controls는 INFO / spms.dao는 TRACE로 별도로 설정 
log4j.logger.spms.controls=INFO
log4j.logger.spms.dao=TRACE
 
# 출력 담당자의 유형을 결정, 로그를 어디로 출력할지 설정한다. 모니터에 출력할 수 있고, 파일로 출력 등을 할 수 있다.
# LOG4J.APPENDER.이름=출력 담당자(패키지명 포함한 클래스명 ConsoleAppender, FileAppender, SocketAppender가 있다.)
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
#로그의 출력 형식 정의(SimpleLayout, HTMLLayout, PatternLayout, XMLLayout이 있다.)
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
#ConversionPattern에서  로그를 출력할 때 사용할 패턴을 정의한다.
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n

```

**로그의 출력 등급 설정**

로그 출력 등급표

로그 출력 등급  | 설명  
------------- | -------------   
FATAL        | 애플리케이션을 중지해야 할 심각한 오류
ERROR           | 오류가 발생했지만, 애플리케이션은 계속 실행할 수 있는 상태   
WARN     |잠재적인 위험을 안고 있는 상태           
INFO    | 애플리케이션의 주요 실행 정보       
DEBUG | 애플리케이션의 내부 실행 상황을 추적해 볼 수 있는 상세 정보             
TRACE  |디버그보다 좀더 상세한 정보  

**주요 로그 출력자 클래스  
출력 담당자 클래스   | 설명  
------------- | -------------   
org.apache.log4j.ConsoleAppender        | System.out 또는 System.err로 로그를 출력한다.기본은 System.out이다. 즉 표준 출력 장치인 모니터로 출력한다.
org.apache.log4j.FileAppender           | 파일로 로그를 출력한다. 
org.apache.log4j.net.SocketAppender     |원격의 로그 서버에 로그 정보를 담은 LoggingEvent 객체를 보낸다.          


# 동적 sql의 사용 

동적 sql 엘리먼트  
mybatis는 동적 sql을 위한 엘리먼트를 제공한다. jstl 코어 라이브러리에 정의 된 태그들과 비슷하다

**<choose> 엘리먼트의 활용**
MySqlProjectDao.xml  
```
<select id="selectList" parameterType="map" resultMap="projectResultMap">
    select PNO, PNAME, STA_DATE, END_DATE, STATE
    from PROJECTS
    order by
    <choose> 
        <when test="orderCond == 'TITLE_ASC'">PNAME asc</when>
        <when test="orderCond == 'TITLE_DESC'">PNAME desc</when>
        <when test="orderCond == 'STARTDATE_ASC'">STA_DATE asc</when>
        <when test="orderCond == 'STARTDATE_DESC'">STA_DATE desc</when>
        <when test="orderCond == 'ENDDATE_ASC'">END_DATE asc</when>
        <when test="orderCond == 'ENDDATE_DESC'">END_DATE desc</when>
        <when test="orderCond == 'STATE_ASC'">STATE asc</when>
        <when test="orderCond == 'STATE_DESC'">STATE desc</when>
        <when test="orderCond == 'PNO_ASC'">PNO asc</when>
        <otherwise>PNO desc</otherwise>
    </choose>
  </select>
```

**프로젝트 목록 페이지에 정렬 링크 추가**
ProjectList.jsp
```java
<%-- 정렬 링크 추가 --%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>프로젝트 목록</title>
</head>
<body>
    <jsp:include page="/Header.jsp" />
    <h1>프로젝트 목록</h1>
    <p>
        <a href='add.do'>신규 프로젝트</a>
    </p>
    <table border="1">
        <tr>
            <th><c:choose>
                    <c:when test="${orderCond == 'PNO_ASC'}">
                        <a href="list.do?orderCond=PNO_DESC">번호↑</a>
                    </c:when>
                    <c:when test="${orderCond == 'PNO_DESC'}">
                        <a href="list.do?orderCond=PNO_ASC">번호↓</a>
                    </c:when>
                    <c:otherwise>
                        <a href="list.do?orderCond=PNO_ASC">번호︎</a>
                    </c:otherwise>
                </c:choose></th>
            <th><c:choose>
                    <c:when test="${orderCond == 'TITLE_ASC'}">
                        <a href="list.do?orderCond=TITLE_DESC">제목↑</a>
                    </c:when>
                    <c:when test="${orderCond == 'TITLE_DESC'}">
                        <a href="list.do?orderCond=TITLE_ASC">제목↓</a>
                    </c:when>
                    <c:otherwise>
                        <a href="list.do?orderCond=TITLE_ASC">제목︎</a>
                    </c:otherwise>
                </c:choose></th>
            <th><c:choose>
                    <c:when test="${orderCond == 'STARTDATE_ASC'}">
                        <a href="list.do?orderCond=STARTDATE_DESC">시작일↑</a>
                    </c:when>
                    <c:when test="${orderCond == 'STARTDATE_DESC'}">
                        <a href="list.do?orderCond=STARTDATE_ASC">시작일↓</a>
                    </c:when>
                    <c:otherwise>
                        <a href="list.do?orderCond=STARTDATE_ASC">시작일</a>
                    </c:otherwise>
                </c:choose></th>
            <th><c:choose>
                    <c:when test="${orderCond == 'ENDDATE_ASC'}">
                        <a href="list.do?orderCond=ENDDATE_DESC">종료일↑</a>
                    </c:when>
                    <c:when test="${orderCond == 'ENDDATE_DESC'}">
                        <a href="list.do?orderCond=ENDDATE_ASC">종료일↓</a>
                    </c:when>
                    <c:otherwise>
                        <a href="list.do?orderCond=ENDDATE_ASC">종료일</a>
                    </c:otherwise>
                </c:choose></th>
            <th><c:choose>
                    <c:when test="${orderCond == 'STATE_ASC'}">
                        <a href="list.do?orderCond=STATE_DESC">상태↑</a>
                    </c:when>
                    <c:when test="${orderCond == 'STATE_DESC'}">
                        <a href="list.do?orderCond=STATE_ASC">상태↓</a>
                    </c:when>
                    <c:otherwise>
                        <a href="list.do?orderCond=STATE_ASC">상태</a>
                    </c:otherwise>
                </c:choose></th>
            <th></th>
        </tr>
        <c:forEach var="project" items="${projects}">
            <tr>
                <td>${project.no}</td>
                <td><a href='update.do?no=${project.no}'>${project.title}</a></td>
                <td>${project.startDate}</td>
                <td>${project.endDate}</td>
                <td>${project.state}</td>
                <td><a href='delete.do?no=${project.no}'>[삭제]</a></td>
            </tr>
        </c:forEach>
    </table>
    <jsp:include page="/Tail.jsp" />
</body>
</html>
```

**프로젝트 목록 컨트롤러 변경**  
ProjectListController.java

```java
package spms.controls;
 
import java.util.HashMap;
import java.util.Map;
 
import spms.annotation.Component;
import spms.bind.DataBinding;
import spms.dao.MySqlProjectDao;
 
@Component("/project/list.do")
public class ProjectListController implements Controller, DataBinding{
    MySqlProjectDao projectDao;
     
    public ProjectListController setProjectDao(MySqlProjectDao projectDao){
        this.projectDao = projectDao;
        return this;
    }
 
    @Override
    public String execute(Map<String, Object> model) throws Exception{
        HashMap<String, Object> paramMap = new HashMap<String, Object>();
        paramMap.put("orderCond", model.get("orderCond"));
         
        model.put("projects",  projectDao.selectList(paramMap));
        return "/project/ProjectList.jsp";
    }
    //@Override
    //public String execute(Map<String, Object> model) throws Exception {
    //  model.put("projects",  projectDao.selectList());
    //  return "/project/ProjectList.jsp";
    //}
     
    //클라이언트가 보낸 orderCond값을 받기위한 것.
    @Override
    public Object[] getDataBinders(){
        return new Object[]{
                "orderCond", String.class
        };
    }
 
}
```
**ProjectDao 인터페이스 변경**  
ProjectDao.java

```
List<Project> selectList(HashMap<String, Object> paramMap) throws Exception;
```

MySqlProjectDao.java
```
public List<Project> selectList(HashMap<String,Object> paramMap)
        throws Exception {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
      return sqlSession.selectList("spms.dao.ProjectDao.selectList", paramMap);
    } finally {
      sqlSession.close();
    }
  }
```


**<set> 엘리먼트의 활용**  
ProjectDao.java  
```
<!-- mybatis는 SET절의 끝에 콤마(,)가 있으면 제거한다. 뒤에 붙은 콤마걱정은 안해도됨 -->
  <update id="update" parameterType="map">
    update PROJECTS
    <set>
      <if test="title != null">PNAME=#{title},</if>
      <if test="content != null">CONTENT=#{content},</if>
      <if test="startDate != null">STA_DATE=#{startDate},</if>
      <if test="endDate != null">END_DATE=#{endDate},</if>
      <if test="state != null">STATE=#{state},</if>
      <if test="tags != null">TAGS=#{tags}</if>
    </set>
    where PNO=#{no}
  </update>
```
