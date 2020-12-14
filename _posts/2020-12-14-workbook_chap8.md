---
layout: post
---

# 8장 스프링 IoC 컨테이너

## 스프링 IoC 컨테이너 사용준비
스프링 프레임워크에서는 빈 관리 컨테이너를 IoC(inversion of Control) 컨테이너라고 합니다.

## 의존성 주입(DI)과 역제어(IoC)

의존성 주입을 일반적인 용어로 역제어(IoC:Inversion of Control)라고 합니다.
즉 역제어(IoC)의 한 형태가 의존성 주입(DI)입니다.
역제어란 개발자가 작성한 코드의 흐름에 따라 제어가  
이루어지는 것이 아니라 외부에 의해 코드의 흐름이 바뀌는 것을 말합니다.


**역제어 사례 - 의존성 주입**  

**예전방식**  

```java
class ProjectListController{
  public void execute(){
    ProjectDao dao = new ProjectDao(); //직접 인스턴스를 생성
    ArrayList<Project> projects = dao.list();
    //...
  }
}
```

앞의 방식과 반대(Inversion)되는 방식이 의존성 주입입니다.
내부에서 생성하는 것이 아니라 외부에서 의존 객체를 주입하는 것입니다.
setter 메서드 setProjectDao()를 통해 외부에서 ProjectDao를 주입합니다.
execute()는 이렇게 외부에서 주입된 객체를 사용합니다.

**의존성 주입을 이용한 방식**
```java
class ProjectListController{
  projectDao dao;
 
  public void setProjectDao(ProjectDao dao){
    this.dao = dao;
  }
 
  public void execute(){
    ArrayList<Project> projects = dao.list();
    //...
  }
}
```

> **이전방식 : 의존성 주입 방식 = 석기시대 : 현대사회**  
석기 시대에는 사회 규모가 작기 때문에 개인이 필요한 물건을 직접 만들어 썼습니다.  
집도 직접 짓고 농작물도 직접 키우고 도구도 직접 만들었습니다.  
하지만 사회가 점점 커지고 복잡해지면서 개인이 모든 것을 직접 만들 수 없게 됩니다.  
필요한 것은 서로 교환하는 방식으로 변화합니다.  
현재사회를 보자면 개인이 직접 자동차를 사지 않고 돈을 주고 삽니다.  
프로그래밍의 세계도 같습니다.
초창기에는 애플리케이션의 크기가 작아서 객체가 필요할 때마다 직접 생성해도  
문제가 되지 않았습니다. 하지만, 애플리케이션 규모가 점점 커지면서 성능이나 유지 보수에  
문제가 생기게 된닙다. 이런 문제를 해결하기 위해 등장한 것이 의존성 주입(DI)이라는 기법입니다.  
**spms.context 패키지의 applicationContext 클래스가 DI 기법으로 빈을 관리하는 IoC 컨테이너입니다.  

## XML 기반 빈 관리 컨테이너

스프링 프레임워크는 spms.context.ApplicationContext 클래스보다 훨씬 뛰어난 객체 관리 컨테이너(IoC컨테이너라고도 부름)를 제공합니다.  
스프링에서는 자바 객체를 '빈(Bean)'이라고 합니다. 그래서 객체 관리 컨테이너를 '빈 컨테이너'라 부릅니다.  
스프링IoC컨테이너는 XML과 애노테이션 두가지 방법으로 빈정보를 다룹니다.  

## ApplicationContext 인터페이스  

스프링은 IoC 컨테이너가 갖추어야 할 기능들을 ApplicationContext 인터페이스에 정의되어있습니다.  
스프링에서 제공하는 IoC 컨테이너들은 모두 이 ApplicationContext 규칙을 따르고 있습니다.

ApplicationContext의 계층도  


스프링에서 빈 정보는 XML 파일에 저장해 두고 ClassPathXmlApplicationContext 클래스나  
**FileSystemXmlApplicationContext** 클래스를 사용하여 빈을 자동 생성합니다.
  
**ClassPathXmlApplicationContext** - 자바 클래스 경로에서 XML로 된 빈 설정 파일을 찾습니다.
  
**FileSystemXmlApplicationContext** - 파일 시스템 경로에서 빈 설정 파일을 찾습니다.



# 클래스 경로란?

JVM이 클래스를 찾을 때 참고하는 폴더 목록다. 외부 라이브러리나 개발자가 만든 클래스를 사용하려면 'CLASSPATH'라는 이름으로 운영체제의 환경변수에 등록해야 합니다.  
또는 JVM을 실행할 때 인자값(arguments)으로 클래스 경로를 전달해야 합니다.  

1) **부트스트랩(Bootstrap)** - JRE에서 제공하는 라이브러라 파일. rt.jar, i18n.jar 등  

2) **확장 라이브러리** - JRE 홈 폴더의 lib/ext 폴더에 있는 JAR 파일  

3) **CLASSPATH** - 운영체제의 CLASSPATH 환경변수에 등록된 폴더 및 JAR 파일들  

# 스프링 빈 컨테이너 ClassPathXmlApplicationContext 사용


**Score클래스 정의**  

빈 컨테이너에서 관리할 클래스를 준비합니다.  

Score.java  

```java
package exam.test01;
 
public class Score {
    String  name;
    float       kor;
    float       eng;
    float       math;
     
    public Score() { }
     
    public Score(String name, float kor, float eng, float math) {
        this.name = name;
        this.kor = kor;
        this.eng = eng;
        this.math = math;
    }
     
    public float average() {
        return sum() / (float)3;
    }
     
    public float sum() {
        return kor + eng + math;
    }
     
    public String getName() {
        return name;
    }
     
    public void setName(String name) {
        this.name = name;
    }
     
    public float getKor() {
        return kor;
    }
     
    public void setKor(float kor) {
        this.kor = kor;
    }
 
    public float getEng() {
        return eng;
    }
 
    public void setEng(float eng) {
        this.eng = eng;
    }
 
    public float getMath() {
        return math;
    }
 
    public void setMath(float math) {
        this.math = math;
    }
 
}
```


**빈 설정 XML 파일 준비**  

애플리케이션에서 사용할 객체는 빈 설정 파일에 선언합니다.  
스프링 IoC 컨테이너는 빈 설정 파일에 선언된 대로 빈을 생성.  

beans.xml  
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
         
  <bean id="score" class="exam.test01.Score"/>
     
</beans>
```
빈 설정 파일은 스프링에서 정의한 규칙에 따라 XML 태그를 작성해야 한다.  
bean태그는 http://www.springframework.org/schema/beans  
네임스페이스에 소속되어 있기 때문에 XML 파서가 이해할 수 있도록 네임스페이스를 밝혀야 한다.  
자바 클래스를 사용하기 전에 import를 선언하는 것과 비슷하다.  

<bean id="score" class="exam.test01.Score"/> 를 자바 코드로 표현하면? new exam.test01.Score();  

스프링 IoC 컨테이너가 생성할 자바 빈에 대한 정보는 <bean> 태그로 선언한다.  

class 속성에는 클래스 이름을 지정한다. 반드시 패키지 이름을 포함한 클래스 이름이어야 한다.  

id 속성은 객체의 식별자이다. 빈 컨테이너에서 객체를 꺼낼 때 이 식별자를 사용한다.  

**스프링 IoC 컨테이너 사용**  

ClassPathXmlApplicationContext 클래스를 사용하여 IoC 컨테이너를 준비한다.  

IoC컨테이너는 빈 설정 파일에 선언된 대로 인스턴스를 생성하여 객체 풀(pool)에 보관한다.  

Test.java  

```java
package exam.test01;
 
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Test {
 
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx =
                new ClassPathXmlApplicationContext("exam/test01/beans.xml");
         
        Score score = (Score) ctx.getBean("score");
         
        System.out.println("합계:" + score.sum());
        System.out.println("평균:" + score.average());
    }
}
```
ClassPathXmlApplicationContext의 인스턴스를 생성할 때 생성자에 빈 설정 파일의 경로를 넘긴다.  

ClassPathXmlApplicationContext는 자바 클래스 경로에서 exam/test01 폴더를 찾는다.  
그리고 beans.xml  파일을 로딩한다. 빈 설정 파일에 선언된 빈을 생성하여 객체 풀(pool)에 보관한다.  

빈 컨테이너가 준비되었으면 getBean()을 호출하여 빈을 꺼낸다.  
getBean()을 호출할 때 넘겨주는 매개변수 값은 bean id이다.  
bean id는 bean 설정 파일에 선언되어 있다.  
**getBean()의 반환타입은 Object이기 때문에 반환된 객체를 제대로 사용하려면 실제 타입으로 형변환해야 한다.**


# name 속성으로 빈 이름 지정하기
<bean> 태그를 사용하여 자바 빈을 선언할 때 id 속성 대신 name 속성에 빈 이름을 지정할 수 있다.  
id 속성과 name속성의 차이점은 다음과 같다.  

항목  | id속성 | name속성 
------------- | -------------   | ------------- 
용도           | 빈 식별자를 지정한다. 중복되어서는 안된다.      | 인스턴스의 별명을 추가할 때 사용한다. id와 마찬가지로 중복되어서는 안된다. 
여러 개의 이름 지정            | 	불가!   | 콤마, 세미콜론 또는 공백을 사용하여 여러 개의 이름을 지정할 수 있다. 첫 번째 이름은 컨테이너에 빈을 보관할 때 사용되고, 나머지 이름은 빈의 별명이 된다. 
빈 이름 작성 규칙  |스프링 프레임워크 3.1 이전 버전은 이름에 문자, 숫자, 밑줄, 하이픈, 점을 사용할 수 있었다. 3.1부터는 제약이 없다.             | 제약 없다.       


beans.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
  <bean name="score" class="exam.test02.Score"/>
  <bean name="score2,score3,score4" class="exam.test02.Score"/>
  <!-- <bean name="score2 score3 score4" class="exam.test02.Score"/> -->
  <!-- <bean name="score2 score3 score4" class="exam.test02.Score"/> -->
  <bean name="score-ok!" class="exam.test02.Score"/>
</beans>
```
구분자를 이용하면 인스턴스에 대해 여러 개의 이름을 붙일 수 있다.  
구분자로는 콤마(,)나 세미콜론(;), 공백을 사용할 수 있다.  
name 속성의 첫 번째 이름은 빈의 이름으로 사용되고 나머지는 빈의 별명이 된다.  
위 소스에서는 exam.test02.Score 인스턴스를 생성한 후 score2 이름으로 빈을 보관한다.  
나머지 이름 score3, score4는 빈의 별명이 된다.  

Test.java


```java
package exam.test02;
 
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Test {
 
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx =
                new ClassPathXmlApplicationContext("exam/test02/beans.xml");
         
        System.out.println("[컨테이너에 보관된 객체의 이름 출력]");
        for (String name : ctx.getBeanDefinitionNames()) {
            System.out.println(name);
        }
         
        System.out.println("[score2의 별명 출력]");
        for (String alias : ctx.getAliases("score2")) {
            System.out.println(alias);
        }
         
        // name="score"
        Score score = (Score) ctx.getBean("score");
         
        // name="score2,score3,score5"
        Score score2 = (Score) ctx.getBean("score2");
        Score score3 = (Score) ctx.getBean("score3");
        Score score4 = (Score) ctx.getBean("score4");
         
        // name="score-ok!"
        Score scoreOk = (Score) ctx.getBean("score-ok!");
         
        System.out.println("[빈 꺼내기]");
        if (score != null) System.out.println("score 찾았음.");
        if (score2 != null) System.out.println("score2 찾았음.");
        if (score3 != null) System.out.println("score3 찾았음.");
        if (score3 != null) System.out.println("score4 찾았음.");
        if (scoreOk != null) System.out.println("score-ok! 찾았음.");
         
        System.out.println("[생성된 빈 비교]");
        if (score == score2) System.out.println("score == score2");
        if (score2 == score3) System.out.println("score2 == score3");
        if (score3 == score4) System.out.println("score3 == score4");
        if (score4 == scoreOk) System.out.println("score4 == scoreOk");
    }
}
```

**getBeanDefinitionNames()**  

-getBeanDefinitionNames()는 빈 컨테이너에 보관된 빈의 이름들을 알고 싶을 때 사용한다. 이 메서드의 반환 타입은 String 배열(String[])이다. 출력 결과에 score3, score4가 보이지 않는 이유는 score2의 별명이기 때문이다.

getAliases()  

-빈의 별명을 알고 싶다면 getAliases()를 호출한다. 이메서드의 매개변수 값으로 빈의 이름을 주면 해당 빈의 별명을 String 배열로 반환한다. score2의 별명을 확인해 보니 score3와  score4가 출력되는 것을 알 수 있다.

getBean()
아이디(id속성), 이름(name 속성), 별명(name 속성)으로 빈을 찾아준다.  
getBean()이 반환한 인스턴스를 보면 score2와 score3, score4가 같은 객체임을 확인할 수 있다.  


8.3.4 익명 빈 선언
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">
                 
    <bean class="exam.test03.Score"/>
    <bean class="exam.test03.Score"/>
</beans>
```

위 소스와 같이 빈을 선언 할 때 id나 name값을 지정하지 않으면 **'익명(anonymous) 빈'**이라고 부른다.  
이렇게 빈의 이름을 지정하지 않으면 컨테이너에 보관할 때 **"패키지 이름 + 클래스 이름 + #인덱스"**를 빈의 이름으로 사용한다.  
(동일한 클래스의 객체가 여러 개 생성될 경우를 대비해 빈 이름 뒤에 '#인덱스'가 자동으로 붙는다.  
앞의 경우 빈의 이름은 exam.test03.Score#0이 되고 두번 째 선언된 빈의 이름은 exam.test03.Score#1이 된다.  
익명 빈의 경우 클래스 이름은(exam.test03.Score) 첫 번째 빈(exam.test03.Score#0)에 대한 별명이 된다.  

# 호출할 생성자 설정

**beans.xml**

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
         
  <bean id="score1" class="exam.test04.Score">
      <constructor-arg><value type="java.lang.String">홍길동</value></constructor-arg>
      <constructor-arg><value type="float">91</value></constructor-arg>
      <constructor-arg><value type="float">92</value></constructor-arg>
      <constructor-arg><value type="float">93</value></constructor-arg>
  </bean>
   
  <bean id="score2" class="exam.test04.Score">
      <constructor-arg><value>임꺽정</value></constructor-arg>
      <constructor-arg><value>81</value></constructor-arg>
      <constructor-arg><value>82</value></constructor-arg>
      <constructor-arg><value>83</value></constructor-arg>
  </bean>
   
  <bean id="score3" class="exam.test04.Score">
      <constructor-arg type="java.lang.String" value="장보고"/>
      <constructor-arg type="float" value="71"/>
      <constructor-arg type="float" value="72"/>
      <constructor-arg type="float" value="73"/>
  </bean>
 
  <bean id="score4" class="exam.test04.Score">
      <constructor-arg value="이순신"/>
      <constructor-arg value="100"/>
      <constructor-arg value="98"/>
      <constructor-arg value="99"/>
  </bean>
   
  <bean id="score5" class="exam.test04.Score">
      <constructor-arg value="70" index="3"/>
      <constructor-arg value="50" index="1"/>
      <constructor-arg value="강감찬" index="0"/>
      <constructor-arg value="60" index="2"/>
  </bean>
   
</beans>
```

빈을 선언할 때 <constructor-arg> 엘리먼트를 이용하여 호출될 생성자를 지정할 수 있다. <constructor-agr>...</constructor-arg>  

호출할 생성자는 매개변수 값에 의해 결정된다.  

호출할 생성자의 매개변수 순서대로 <constructor-arg>를 사용하여 매개변수 값을 설정합니다. 매개변수 값은 <value>를 사용하여 지정한다.

<constructor-arg><value type="java.lang.String">홍길동</value></constructor-arg>  

      <constructor-arg><value type="float">91</value></constructor-arg>  
      
      <constructor-arg><value type="float">92</value></constructor-arg>  
      
      <constructor-arg><value type="float">93</value></constructor-arg>  
      
<value>의 type속성은 매개변수 타입이고, 컨텐츠(위에서는 '홍길동')는 매개변수값이다.  

자바코드로 표현하면?  

new Score("홍길동", 91, 91, 91);  

다른 생성자와 혼동되지 않는다면 type속성을 생략할 수 있다. value속성의 값은 생성자의 매개변수 타입에 맞추어 자동 형변환 된다.  

<constructor-arg value="70" index="3"/>  

      <constructor-arg value="50" index="1"/>  
      
      <constructor-arg value="강감찬" index="0"/>  
      
      <constructor-arg value="60" index="2"/>  
   
index속성을 이용하면 매개변수의 순서를 지정할 수 있다. 인덱스는 0부터 순서로 생성자에 전달된다.  

**Test.java**
```java
package exam.test04;
 
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Test {
 
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx =
                new ClassPathXmlApplicationContext("exam/test04/beans.xml");
         
        Score score1 = (Score) ctx.getBean("score1");
        System.out.println(score1);
         
        Score score2 = (Score) ctx.getBean("score2");
        System.out.println(score2);
         
        Score score3 = (Score) ctx.getBean("score3");
        System.out.println(score3);
         
        Score score4 = (Score) ctx.getBean("score4");
        System.out.println(score4);
         
        Score score5 = (Score) ctx.getBean("score5");
        System.out.println(score5);
         
    }
}
```

# 프로퍼티 설정  

빈 설정 파일에서 인스턴스를 선언할 때 생성자처럼 프로퍼티 값을 설정할 수 있다.  

프로퍼티 값을 지정할 때는 <property>를 사용한다.  

p.s 필드와 프로퍼티  

인스턴스 변수는 필드(field) 또는 속성(attribute)이라고도 부른다.  
프로퍼티(property)는 setter/getter 메서드를 가리키는 용어이다.  
```
class Member{
  int _no; //필드
  String _name; //필드
  public void setNo(int no){ this._no = no;} //프로퍼티
  public int getNo(){ return this._no;} //프로퍼티
  public void setName(String name){ this._name = name;} //프로퍼티
  public String getName(){ return this._name;} //프로퍼티
}
```

위 코드에서 필드는 _no, _name이다. 프로퍼티는 getNo()/setNo(), getName()/setName()이다. 프로퍼티 이름은 getter/setter 메서드에서 set/get/을 빼고, 나머지 이름에서 첫 글자를 소문자로 한 이름이다. 즉 Member 클래스의 프로퍼티는 no와 name이다.

**beans.xml**

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
         
  <bean id="score1" class="exam.test05.Score">
    <property name="name"><value>홍길동</value></property>
    <property name="kor"><value>100</value></property>
    <property name="eng"><value>95</value></property>
    <property name="math"><value>90</value></property>
  </bean>
   
  <bean id="score2" class="exam.test05.Score">
    <property name="name" value="임꺽정"/>
    <property name="kor" value="85"/>
    <property name="eng" value="99"/>
    <property name="math" value="100"/>
  </bean>
   
</beans>


빈의 프로퍼티 값을 설정할 때는 <property> 태그를 선언한다.  
<property>의 name 속성은 프로퍼티 이름을 설정하고,  
자식 태그<value>는 프로퍼티 값을 설정한다.  

<constructor-arg>와 마찬가지로 <value> 태그를 사용하는 대신 <property>의 value 속성을 사용하여  

프로퍼티 값을 설정할 수 있다.  

자바 코드로 표현하면?  
```
Score score1 = new Score();
score1.setName("홍길동");
score1.setKor(100);
score1.setEng(95);
score1.setMath(90);
 
Score score2 = new Score();
score2.setName("임꺽정");
score2.setKor(85);
score2.setEng(99);
score2.setMath(100);
```
**<bean>의 속성을 이용하여 생성자 및 프로퍼티 설정하기**
스프링은 <property> 태그를 사용하지 않고 프로퍼티 값을 설정하는 방법을 제공한다. 또한 <constructor-arg> 태그를 사용하는 대신 더 간단히 생성자의 매개변수 값을 설정하는 방법도 제공한다.  
<bean> 속성을 통해 프로퍼티나 생성자의 매개변수 값을 설정하는 방법을 보겠다.  

beans.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
   
  <bean id="score1" class="exam.test06.Score"
      p:name="홍길동" p:kor="100" p:eng="95" p:math="90"/>
   
  <bean id="score2" class="exam.test06.Score"
      c:name="임꺽정" c:kor="80" c:eng="90" c:math="100"/>
  <!-- 
  자바로 표현하면?
  Score score1 = new Score();
  score1.setName("홍길동");
  score1.setKor(100);
  score1.setEng(95);
  score1.setMath(90);
 
  Score score2 = new Score();
  score2.setName("임꺽정");
  score2.setKor(80);
  score2.setEng(90);
  score2.setMath(100);
  -->
</beans>
```

4번째 5번째줄  

<bean>의 속성을 통해 프로퍼티 값을 설정하려면, 먼저 <beans> 시작 태그에 이를 위한 네임스페이스 "xmlns:p="http://www.springframework.org/schema/p"를 선언한다.  

네임스페이스의 별명으로 프로퍼티의 약자 'p'를 사용한다.  

사용방법은 'p:프로퍼티명=값' 형식으로 선언한다.  

생성자도 프로퍼티처럼 <bean> 태그의 속성으로 매개변수 값을 설정할 수 있다.  

먼저 위 코드와 같이 'c' 네임스페이스를 선언한다. 네임스페이스 별명인 'c'는 생성자를 의미한다. 물론 네임스페이스 별명 'c'를 다른 문자로 변경할 수 있다.  

사용방법은 'c:매개변수값=값' 형식으로 선언한다.  



# 의존 객체 주입
어떤 객체가 작업을 수행하기 위해 다른 객체를 지속적으로 사용한다면  
그 사용되는 객체를 의존 객체라고 부른다.  
보통 지속적으로 사용할 객체는 프로퍼티에 보관한다.  

# 의존 객체 설정하기
위에 예제는 문자열 또는 숫자로 프로퍼티 값을 설정하였는데  
프로퍼티에 객체를 지정하는 방법을 안내한다.  

# Map과 Properties 값 주입  

빈 프로퍼티 타입이 java.util.Map이나  java.util.Properties인 경우, 식별자(key)와 값(value)을 한쌍으로 묶어서 저장한다.  

java.util.Properties 타입의 값을 설정할 때는 <props> 태그를 사용한다.  

Properties에 저장할 항목은 <prop> 태그로 정의한다.  

<prop>의 key 속성에는 문자열로 된 식별자가 들어가고, <prop>와 </prop> 사이에는 상숫값이 들어간다.  

java.util.Map타입인데 이 타입의 값을 설정할 때는 <map> 태그를 사용한다.  

**Map과 Properties의 용도**  

java.util.Map 타입의 클래스는 key와 value로 문자열뿐만 아니라 다른 타입(클래스 또는 인터페이스)의 객체도 사용할 수 있다.  

java.util.Properties 클래스도 Map의 일종(Map 인터페이스를 구현함)이지만 주로 문자열로 된 식별자와 값을 다룰 때 사용한다.  

맵에 들어갈 식별자와 값은 <entry>로 정의한다. 엔트리의 식별자를 설정할 때는 <key>태그를 사용한다.  

<key>태그에 직접 식별자를 넣을 수는 없고 <value>를 사용해 넣는다.  
엔트리의 값으로 상수값을 설정할 때는 <value> 태그를 사용한다.  

# 팩토리 메서드와 팩토리 빈

객체 지향 설꼐 기법 중에서 팩토리 메서드(factory method)패턴과 빌더(Builder)패턴이 있다.  

공장 역할을 하는 객체를 통해 필요한 인스턴스를 간접적으로 얻는 방식이다.  


인스턴스를 생성하는 일반적인 방법은 new 명령을 사용하는 것이다.  

new 명령을 통해 클래스와 생성자를 지정하면 해당 클래스의 인스턴스가 생성되고 초기화된다.  

문제는 인스턴스 생성작업이 복잡한 경우이다.  
매번 인스턴스를 생성할 때마다 복잡한 과정을 거쳐야 한다면 코딩할때 부담이 된다.  
이를 해결하기 위해 나온 것이 팩토리 메서드 패턴과 빌더 패턴이다.  

팩토리 클래스는 인스턴스 생성 과정을 캡슐화한다.  

인스턴스를 생성하고 초기화하는 복잡한 과정을 감추기 때문에 개발자 입장에서는  

인스턴스를 얻는 과정이 간결해진다.


# P.S 자바 객체(Java object) vs 인스턴스(instance) vs 빈(bean)

new 명령어나 팩토리 메서드로 생성된 '인스턴스'를 일반적으로 '자바 객체'라 부른다.  

스프링에서는 자바 객체를 '빈'이라고 부른다. 일상 생활에서도 하나의 사물을 다양하게 표현하듯이,  

예를 들어 물을 음료라고 부르기도 하고 '음료수', 마실 것' 등으로 부르는 것처럼, 프로그래밍 세계에서도 하나의 개념에 대해 다양한 말로 표현한다.  

TireFactory.java
```java
package exam.test11;
 
import java.text.SimpleDateFormat;
import java.util.Properties;
 
public class TireFactory {
    /*
     * createTire()는 팩토리 메서드이다. 매개변수 값으로 제조사 이름을 주면 해당 제조사의 타이어를 생산하는 메서드이다.
     * 일반 클래스의 메서드를 공장(factory)기능을 하도록 하려면 반드시 static으로 선언해야 한다. 이것이 스프링 IoC 컨테이너의 규정이다.
     */
    public static Tire createTire(String maker) {
        if (maker.equals("Hankook")) {
            return createHankookTire();
        } else {
            return createKumhoTire();
        }
    }
     
    /*
     * createTire()에서 호출하는 메서드가 두개 있다. 외부에서 호출할 일이 없기 때문에 두 메서드 모두 private로 선언하였다.
     * 메서드는 무조건 public으로 공개해야 한다고 생각하는 건 금물.
     * 내부에서 사용할 목적으로 만든 메서드는 private으로 선언해서 외부에서 호출하지 못하도록 막는 것이 좋다.
     *
     * createTire()는 스태틱메서드이기 때문에 스태틱 멤버(변수 및 메서드)만 직접 접근할 수 있다.
     * 인스턴스 메서드는 인스턴스가 있어야 하므로, createHankookTire()와 createKumhoTire()를 static으로 선언한다.
     */
    private static Tire createHankookTire() {
        Tire tire = new Tire();
        tire.setMaker("Hankook");
         
        Properties specProp = new Properties();
        specProp.setProperty("width", "205");
        specProp.setProperty("ratio", "65");
        specProp.setProperty("rim.diameter", "14");
        tire.setSpec(specProp);
         
        /*
         * '2014-5-5' 형식의 문자열을  java.util.Date 객체로 만들어 주는 SimpleDateFormat 객체를 생성한다.
         */
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        try {
            tire.setCreatedDate(dateFormat.parse("2014-5-5"));
        } catch (Exception e) {}
         
        return tire;
    }
     
    private static Tire createKumhoTire() {
        Tire tire = new Tire();
        tire.setMaker("Kumho");
         
        Properties specProp = new Properties();
        specProp.setProperty("width", "185");
        specProp.setProperty("ratio", "75");
        specProp.setProperty("rim.diameter", "16");
        tire.setSpec(specProp);
         
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        try {
            tire.setCreatedDate(dateFormat.parse("2014-3-1"));
        } catch (Exception e) {}
         
        return tire;
    }
}
```
beans.xml
```java

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <bean id="hankookTire" class="exam.test11.TireFactory"
        factory-method="createTire">
        <constructor-arg value="Hankook" />
    </bean>
    <!--
        자바로 표현하면?
        Tire hankookTire = TireFactory.createTire("Hankook");
         
        빈의 class속성에 팩토리 클래스 이름을 지정한다.
        factory-method 속성의 값은 Tire 인스턴스를 생성할 메서드 이름이다.
        factory-method 속성에는 반드시 static 메서드 이름을 지정해야 한다.
        팩토리 메서드에 넘겨 줄 매개변수 값은 <constructor-arg> 태그로 설정한다.
     -->
 
    <bean id="kumhoTire" class="exam.test11.TireFactory"
        factory-method="createTire">
        <constructor-arg value="Kumho" />
    </bean>
    <!--
        자바로 표현하면?
        Tire kumhoTire = TireFactory.createTire("Kumho");
         
        id 'kumhoTire'는 createTire()가 반환한 빈을 가리킨다. 실제는 TireFactory안에 createKumhoTire()가 반환한 빈이다.
     -->
</beans>
```

Test.java
```java

package exam.test11;
 
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Test {
 
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx =
                new ClassPathXmlApplicationContext("exam/test11/beans.xml");
         
        Tire t1 = (Tire) ctx.getBean("hankookTire");
        Tire t2 = (Tire) ctx.getBean("kumhoTire");
        Tire t3 = (Tire) ctx.getBean("hankookTire");
         
        System.out.println("t1-->" + t1.toString());
        System.out.println("t2-->" + t2.toString());
        System.out.println("t3-->" + t3.toString());
         
        if (t1 != t2) {
            System.out.println("t1 != t2");
        }
         
        if (t1 == t3) {
            System.out.println("t1 == t3");
        }
    }
}
```

다시 빈 컨테이너에서 'hankookTire'로 등록된 빈을 꺼낸다.  

이전에 꺼낸 t1과 같은 빈이다. getBean()은 호출될 때마다 팩토리 메서드를 호출하지 않는다.  

처음 빈을 만들 때 딱 한 번 호출한다.  

bean.xml  
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <bean id="tireFactory" class="exam.test12.TireFactory" />
    <!--
    Java
    TireFactory tireFactory = new TireFactory();
     -->
 
    <bean id="hankookTire" factory-bean="tireFactory" factory-method="createTire">
        <constructor-arg value="Hankook" />
    </bean>
    <!--
    Java
    Tire hankookTire = tireFactory.createTire("Hankook");
     -->
 
    <bean id="kumhoTire" factory-bean="tireFactory" factory-method="createTire">
        <constructor-arg value="Kumho" />
    </bean>
    <!--
    Java
    Tire kumhoTire = tireFactory.createTire("Kumho");
     -->
</beans>
```

# 스프링 규칙에 따라서 팩토리 빈 만들기

스프링 프레임워크에서 제공하는 규칙(인터페이스)에 따라 팩토리 클래스를 만든다. 



# 팩토리 클래스 생성
스프링에서는 팩토리 빈이 갖추어야 할 규칙을 org.springframework.beans.factoryFactoryBean 인터페이스에 정의하였다.  

팩토리 클래스를 만들 때 이 규칙에 따라 메서드를 구현하면 된다.  

보통 FactoryBean 인터페이스를 직접 구현하기 보다는 스프링에서 제공하는 추상 클래스를 상속해서 만든다.  

org.springframework.beans.factory.config.AbstractFactoryBean은 FactoryBean 인터페이스를 미리 구현하였다.


AbstractFactoryBean에는 createInstance()라는 추상 메서드가 있다.  

빈을 생성할 때 팩토리메서드로서 getObject()가 호출되는데, getObject()는 내부적으로 바로 이 메서드를 호출한다.  

실제 빈을 생성하는 것은 createInstance()이다.  

따라서 AbstractFactoryBean 클래스를 상속받을 때는 반드시 이 메서드를 구현해야 하고 이 메서드에 빈 생성 코드를 넣는다.  

또한 FactoryBean 인터페이스로부터 받은 getObjectType() 추상 메서드가 구현되지 않은 채로 남아 있다.  

이 메서드는 팩토리 메서드(getObject())가 생성하는 객체의 타입을 알려준다. 이 메서드도 구현해야 한다.  

TireFactoryBean.java
```java
package exam.test13;
 
import java.text.SimpleDateFormat;
import java.util.Properties;
 
import org.springframework.beans.factory.config.AbstractFactoryBean;
 
//TireFactoryBean 클래스는 AbstractFactoryBean 클래스를 상속받는다.
//AbstractFactoryBean은 하나의 코드로 다양한 타입의 빈을 다룰 수 있도록 generic문법이 적용된 클래스이다.
//이 클래스를 상속받을 때 타입 매개 변수 T(AbstractFactoryBean<T>)에 들어갈 클래스 타입을 지정한다.
//즉 팩토리 빈이 생성해 줄 인스턴스의 타입을 지정한다.
public class TireFactoryBean extends AbstractFactoryBean<Tire> {
//제조사(maker) 정보를 보관할 프로퍼티를 위해 셋터와 인스턴스 변수를 정의한다.
  String maker;
   
    public void setMaker(String maker) {
        this.maker = maker;
    }
 
//AbstractFactoryBean으로부터 상속받은 추상 메서드 getObjectType()을 구현한다.
//반환값은 팩토리 빈이 만드는 객체의 타입이다. TireFactoryBean은 Tire 객체를 생성해 주는 팩토리 빈이다.
    @Override
    public Class<?> getObjectType() {
        return exam.test13.Tire.class;
    }
 
//상속받은 또 하나의 추상 메서드 createInstance()를 구현한다.
//getObject()에 의해 내부적으로 호출되는 메서드이기 때문에 protected로 되어 있다.
    protected Tire createInstance() {
        if (maker.equals("Hankook")) {
            return createHankookTire();
        } else {
            return createKumhoTire();
        }
    }
 
    private Tire createHankookTire() {
        //..
    }
 
    private Tire createKumhoTire() {
        //..
    }
}
```

beans.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <bean id="hankookTire" class="exam.test13.TireFactoryBean">
        <property name="maker" value="Hankook" />
    </bean>
    <!--
        FactoryBean tempFactoryBean = new TireFactoryBean();
        tempFactoryBean.setMaker("Hankook");
        Tire tireFactory = tempFactoryBean.getObject();
         
        스프링에 제공하는 규칙에 따라 만든 팩토리 빈은 사용법이 이전과 비슷하다.
        대신 팩토리 역할을 할 메서드를 지정할 필요가 없다. 스프링 IoC 컨테이너는 class 속성에 주어진 클래스가 일반 클래스가 아니라
        FactoryBean 타입의 클래스일 경우, 이 클래스의 인스턴스를 직접 보관하지 않고 이 클래스가 생성한 빈을 컨테이너에 보관한다.
        즉, TireFactoryBean 클래스는 FactoryBean 타입이기 때문에 TireFactoryBean이 생성한 Tire 객체를 'hankookTire'라는 이름으로
        컨테이너에 보관한다.
     -->
 
  <bean id="kumhoTire" class="exam.test13.TireFactoryBean">
    <property name="maker" value="Kumho" />
  </bean>
</beans>
```

스프링 IoC 컨테이너는 팩토리 빈(TireFactoryBean)에게 인스턴스를 만들어 달라고 요청(getObject() 호출)한다.  

팩토리 빈은 우리가 구현한 createInstance()를 호출한다.  

그리고 createInstance()가 생성해 준 인스턴스를 그대로 컨테이너에게 반환한다.  

# 빈의 범위 설정

빈의 범위(bean scope)를 설정. 스프링 IoC 컨테이너는 빈을 생성할 때 기본으로 한 개만 생성한다.  

getBean()을 호출하면 계속 동일한 객체를 반환한다.  

하지만 설정을 통해 이런 빈의 생성 방식을 조정할 수 있다.  


범위  | 설명 
------------- | -------------    
싱글턴(singleton)           |오직 하나의 빈만 생성한다(기본 설정) 
프로토타입(prototype)            | getBean()을 호출할 때마다 빈을 생성한다.    
요청(request)     |HTTP 요청이 발생할 때마다 생성되며, 웹 애플리케이션에서마 이 범위를 설정할 수 있다.                
세션(session)      |HTTP 세션이 생성될 때마다 빈이 생성되며, 웹 애플리케이션에서만 이 범위를 설정할 수 있다.   
전역 세션(globalsession)   |전역 세션이 준비될 때 빈이 생성된다. 웹 애플리케이션에서만 이 범위를 지정할 수 있으면, 보통 포틀릿 컨텍스트에서 사용한다. 
 

# 싱글톤과 프로토타입
한 개의 인스턴스만 생성하여 사용하는 싱글톤 방식  

매번 요청할 때마다 인스턴스를 생성해 주는 방식  

bean.xml
```java
 <bean id="hyundaiEngine" class="exam.test14.Engine"
        p:maker="Hyundai" p:cc="1997"/>
  <!--
     빈의 번위를 설정하지 않으면 'singletone'이 기본 범위로 지정된다.
     -->
     
    <bean id="kiaEngine" class="exam.test14.Engine"
        p:maker="Kia" p:cc="3000"
        scope="prototype"/>
  <!--
     scope 속성의 값을 'prototype'으로 설정하면 이 경우 빈 컨테이너에게 'kiaEngine'을
   요청할 때마다 매번 새 Engine인스턴스를 만들어 반환한다.
     -->
```
# 날짜 값 주입
java.util.Date 타입의 프로퍼티 값을 설정한다. 스프링에서는 빈의 프로퍼티에 값을 설정할 때 태그의 콘텐츠(시작 태그와 끝 태그 사이의 값)나 태그의 속성(시작 태그 안에 작성하는 속성의 값)을 사용한다.
빈 설정 파일은 XML이기 때문에 결국 프로퍼티의 값은 문자열로 표현한다. 문자열은 숫자로 변환하기 쉽기 때문에 숫자 타입 프로퍼티인 경우 별도의 추가 작업없이 간단히 값을 설정할 수 있다. 하지만, 그 외 타입은 자동으로 변환되지 않는다. 개발자가 프로퍼티 타입에 맞는 객체를 생성해야 한다. 다음 실습은 개발자가 직접 java.util.Date 타입의 프로퍼티 값을 설정하는 예이다.


# SimpleDateFormat 클래스와 인스턴스 팩토리 메서드 활용
SimpleDateFormat 클래스와 팩토리 메서드 방식을 활용하여 날짜 값을 설정해본다. SimpleDateFormat의 parse()를 팩토리 메서드로 활용하여 java.util.Date 객체를 얻는다.

bean.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
    <bean id="dateFormat" class="java.text.SimpleDateFormat">
        <constructor-arg value="yyyy-MM-dd" />
    </bean>
    <!--
        자바코드
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
         
        SimpleDateFormat 객체는 날짜 형식의 문자열을 java.util.Date 객체로 변환해 준다.
        생성자 매개변수 값으로 날짜 형식을 지정한다.
     -->
 
    <bean id="hankookTire" class="exam.test15.Tire">
        <property name="maker" value="Hankook" />
        <property name="createdDate">
            <bean factory-bean="dateFormat" factory-method="parse">
                <constructor-arg value="2014-5-5" />
            </bean>
        </property>
    </bean>
    <!--
        hankookTire.setCreatedDate(dateFormat.parse("2014-5-5"));
         
        hankookTire 객체의 createdDate는 java.util.Date 타입의 프로퍼티이다. 이 프로퍼티에 넣을 수 있는 값은 날짜 객체이다.
        날짜 객체를 얻기 위해 팩토리 메서드를 선언한다. factory-bean 속성에는 앞에서 정의한 SimpleDateFormat 객체를 지정하고,
        factory-method 속성에는 parse()를 지정한다. parse()는 문자열르 가지고 날짜 객체를 만들어 주는 인스턴스 메서드이다.
     -->
     
    <bean id="kumhoTire" class="exam.test15.Tire">
    <property name="maker" value="Kumho" />
    <property name="createdDate">
      <bean factory-bean="dateFormat" factory-method="parse">
        <constructor-arg value="2014-1-14" />
      </bean>
    </property>
  </bean>
</beans>
```

# 커스텀 프로퍼티 에디터 활용

> 앞의 방식의 문제는 날짜 프로퍼티 값을 설정할 때마다 팩토리 메서드 빈을 선언해야 한다는 것이다.  
이런 불편한 점을 해결하기 위해 스프링 IoC 컨테이너는 프로퍼티 에디터를 도입하였다.  
'프로퍼티 에디터(property editor)'란, 문자열을 특정 타입의 값으로 변환해 주는 객체이다.  

`"문자열" -> Propery Editor -> 인스턴스`


스프링에서는 java.util.Date처럼 자주 사용하는 타입에 대해 몇 가지 프로퍼티 에디터를 제공한다.  

날짜 형식의 문자열을 java.util.Date 객체로 변환해 주는 CustomDateEditor 클래스와 URL형식의 문자열을 java.net.URL 객체로 변환해 주는 URLEditor 클래스 등이 있다.  

p.s 그 외 프로퍼티 에디터 클래스들은 스프링 API에서 org.springframework.beans.propertyeditors를 참고  


# 애노테이션을 이용한 의존 객체 자동 주입

스프링에서는 자바 애노테이션을 이용해 간단히 의존 객체를 주입할 방법을 제공한다.  

빈이 셋터 메서드에 @Autowired를 선언하면 빈 컨테이너가 셋터의 매개변수 타입과 일치하는 빈을 찾아 자동으로 설정해준다.  

이 기능을 이용하려면 빈 설정 파일에 다음 객체를 선언해야 한다.  

`<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />` 


AutowiredAnnotationBeanPostProcessor 클래스는 빈의 후 처리기로서 빈을 생성한 후 즉시 @Autowired로 선언된 셋터를 찾아서 호출하는 역할을 수행한다.  

즉 @Autowired가 붙은 프로퍼티에 대해 의존 객체(프로퍼티 타입과 일치하는 빈)를 찾아 주입(프로퍼티에 할당)하는 일을 한다.  


Car.java  
```java
package exam.test17;
 
import java.util.Map;
 
import org.springframework.beans.factory.annotation.Autowired;
 
public class Car {
  //..
  @Autowired
  public void setEngine(Engine engine) {
    this.engine = engine;
  }
  //..
}
```

bean.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
 
  <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />
  <!--
        자바코드
        AutowiredAnnotationBeanPostProcessor tempAutowiredPostProcessor =
            new AutowiredAnnotationBeanPostProcessor();
     
    @Autowired 애노테이션을 처리할 빈(AutowiredAnnotationBeanPostProcessor)을 선언한다.
    이 객체는 빈 컨테이너에 등록된 모든 빈을 조사하여 @Autowired가 붙은 셋터 메서드가 있다면 호출한다.
    이때 셋터 메서드에 넘기는 매개변수 값은 빈 컨테이너에서 매개변수 타입으로 찾은 객체이다.
    만약 빈 컨테이너에 매개변수 타입에 해당하는 객체가 없다면 예외가 발생하는 데 @Autowired가 붙은 프로퍼티는 값을 반드시 설정해야 하기 때문이다.
    빈 컨테이너에 매개변수 타입에 해당하는 객체가 두 개 이상 있다면 그 때도 예외가 발생한다. 그 중 어떤 것을 선택해야 할지 알 수 없다.
   -->
     
  <bean id="hyundaiEngine" class="exam.test17.Engine">
      <constructor-arg value="Hyundai"/>
  </bean>
    <!--
        Engine hyundaiEngine = new Engine("Hyundai");
    -->
   
  <bean id="car1" class="exam.test17.Car">
      <property name="model" value="Sonata"/>
  </bean>
  <bean id="car2" class="exam.test17.Car">
      <property name="model" value="Grandeur"/>
  </bean>
    <!--
        1)객체 준비
        Car car1 = new Car();
        Car car2 = new Car();
 
        2)프로퍼티 설정
        car1.setModel("Sonata");
        car2.setModel("Grandeur");
 
        3) @Autowired 처리 - AutowiredAnnotationBeanPostProcessor
        car1.setEngine(hyundaiEngine);
        car2.setEngine(hyundaiEngine);
 
        두 개의 Car 객체를 생성한다. 'engine' 프로퍼티를 설정하는 코드가 없다.
        'engine'프로퍼티가 @Autowired로 선언되었기 때문에, @Autowired 애노테이션 처리기
        AutowiredAnnotationBeanPostProcessor에 의해 hyundaiEngine 빈이 자동 설정된다.
    -->
</beans>
```
# @Autowired의 required 속성

프로퍼티(셋터 메서드)에 대해 @Autowired를 선언하면 해당 프로퍼티는 기본으로 필수 입력 항목이 된다.  

만약 프로퍼티에 주입할 의존 객체를 찾을 수 없다면 예외가 발생한다.  

@Autowired의 required 속성을 이용하면 필수 입력 여부를 조정할 수 있다.  

required 속성을 false로 설정하면 프로퍼티에 주입할 의존 객체를 찾지 못하더라도 예외가 발생하지 않는다.  

Car.java
```java
package exam.test18;
 
import java.util.Map;
 
import org.springframework.beans.factory.annotation.Autowired;
 
public class Car {
  //..
  @Autowired(required=false)
  public void setEngine(Engine engine) {
    this.engine = engine;
  }
  //..
}
```
@Autowired 애노테이션의 required 속성값을 false로 설정하고  

bean.xml에서 engine프로퍼티에 주입할 객체를 주석처리해도 오류가 발생하지 않는다.  

# @Qualifier로 주입할 객체를 지정하기  

@Autowired는 프로퍼티에 주입할 수 있는 의존 객체가 여러 개 있을 경우 오류를 발생시킨다.  

즉 같은 타입의 객체가 여러 개 있으면 그 중에서 어떤 객체를 주입해야 할지 알 수 없기 때문이다.  

이 경우 애노테이션 @Qualifier를 사용하면 된다. @Qualifer는 빈의 이름(또는 아이디)으로 의존 객체를 지정할 수 있다.  

이 애노테이션을 사용하려면 빈의 후 처리기(bean post processor)가 필요하다.  

이런식으로 하다보면 애노테이션을 사용할 때 마다 그 애노테이션을 처리할 객체를 등록해야 하는 불편함이 발생한다.  

이런 불편함을 해소하기 위해 스프링에서는 애노테이션 처리와 관련된 빈(bean post processor)을 한꺼번에 등록할 수 있는 방법을 제공한다.  


`<context:annotation-config/> `

beans.xml
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
 
  <context:annotation-config />
  <!--
    이 태그를 선언하면 애노테이션 처리를 담당하는 다음의 빈들이 자동 등록 된다.
    annotation-config 캐그는 context 네임스페이스에 들어 있다. 따라서 이 태그를 사용하기 전에 먼저 네임스페이스를 선언해야 한다.
   
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd"
   -->
           
  <bean id="hyundaiEngine" class="exam.test19.Engine">
      <constructor-arg value="Hyundai"/>
  </bean>
 
  <bean id="kiaEngine" class="exam.test19.Engine">
      <constructor-arg value="Kia"/>
  </bean>
 
  <bean id="car1" class="exam.test19.Car">
      <property name="model" value="Sonata"/>
  </bean>
</beans>
```
**@Qualifier를 사용하여 의존 객체 지정**

Car.java  
```java
package exam.test19;
 
import java.util.Map;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
 
public class Car {
    //..
    @Autowired(required=false)
    @Qualifier("kiaEngine")
    public void setEngine( Engine engine) {
        this.engine = engine;
    }
    //..
}
```
engine 프로퍼티(setEngine()) 앞에 @Qualifier 선언을 추가한다.  

이 애노테이션에 들어가는 값은 beans.xml에 선언된 엔진 객체 중에서 'kiaEngine'을 가리키는 이름이다.  

빈 컨테이너는 eingine 프로퍼티의 값으로 해당 이름을 갖는 객체를 찾아 주입한다.  

# @Autowired + @Qualifier = @Resource

> JSR-250 명세에 정의된 @Resource를 사용하면 더 간단히 특정 의존 객체를 지정할 수 있다.  

이름으로 의존 객체를 지정할 경우 이 애노테이션을 사용할 것을 권장한다.  

단, @Autowired는 required 속성을 사용하여 프로퍼티 값의 필수 여부를 지정할 수 있지만  

@Resource에는 그런 기능이 없으므로 @Resource는 무조건 필수 입력이다.  

해당 프로퍼티에 주입할 의존 객체가 없으면 오류가 발생하므로 의존 객체 주입을 선택 사항으로 만들고 싶다면 @Autowired를 사용하도록 한다.


Car.java  
```java
package exam.test20;
 
import java.util.Map;
 
import javax.annotation.Resource;
 
public class Car {
    //..
    @Resource(name="kiaEngine")
    public void setEngine( Engine engine) {
        this.engine = engine;
    }
    //..
}
```

# 빈 자동 등록
> 컴포넌트 스캔과 애노테이션을 이용해 빈을 자동 등록하는 방법을 본다.  

1)빈 생성 대상이 되는 클래스에 @Component 애노테이션으로 표시를 한다.  

2)스프링 IoC 컨테이너는 @Component가 붙은 클래스를 찾아 빈을 생성한다.  


스프링에서는 @Component 외에 클래스를 관리하고 이해하는데 도움을 주기 위해  

클래스의 역할에 따라 붙일 수 있는 애노테이션을 추가로 제공한다.  


**자동으로 생성할 빈을 지정하는 애노테이션**


애노테이션   | 설명 
------------- | -------------    
@Component           |빈 생성 대상이 되는 모든 클래스에 대해 붙일 수 있다.  
@Repository             | DAO와 같은 퍼시스턴스(persistence)역할을 수행하는 클래스에 붙인다.    
@Service     |서비스 역할을 수행하는 클래스에 붙인다.                
@Controller     |MVC 구조에서 Controller 역할을 수행하는 클래스에 붙인다.   

**※앞의 표에 제공하는 애노테이션 중에서 어떤 것을 붙이더라도 상관없다.  

하지만 이왕이면 클래스의 역할에 따라 알맞은 애노테이션을 붙인다면 코드의 가독성에 도움이 될 것이다.  

또한 특정 애노테이션이 붙은 클래스만 선별하여 어떤 작업을 하고자 할 때 유용하다.  **


# @Component가 붙은 클래스 자동으로 찾기

Car.java
```java
package exam.test21;
 
import java.util.Map;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
 
@Component("car")
public class Car {
    //..
    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
    //..
}
```
Engine.java
```java
package exam.test21;
 
import org.springframework.stereotype.Component;
 
@Component("engine")
public class Engine {
    //..
}
```
beans.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
 
  <context:component-scan base-package="exam.test21"/>
  <!--
    <context:component-scan/> 태그를 사용하여 @Component, @Repository 등 빈 생성 표시자(애노테이션)가 붙은 클래스를 검색하라고 선언한다.
    <context:component-scan/> 태그를 선언하면 <context:annotation-config/> 태그의 기능이 자동으로 활성화된다.
     
    컴포넌트 탐색의 포함 조건과 제외 조건
    context:include-filter 태그는 빈탐색에 포함할 대상자를 지정할 때 사용
    context:exclude-filter 태그는 빈 탐색에서 제외할 대상자를 지정할 대 사용
   -->
 
</beans>
```


