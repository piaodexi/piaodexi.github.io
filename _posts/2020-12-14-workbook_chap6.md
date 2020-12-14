---
layout: post
title: "workbook"
categories: java_web
---

# 6장 미니 MVC 프레임워크 만들기

## 6.1 프런트 컨트롤러의 도입

- 프런트 컨트롤러 패턴
![](./6_1.PNG)
- 프런트 컨트롤러는 VO객체의 준비, 뷰 컴포넌트로의위임, 오류 처리 등과 같은 공통 작업ㅇ르 담당하고, 페이지 컨트롤러는 이름 그대로 요청한 페이지만을 위한 작업을 수행한다.

## 디자인 패턴

개발자는 늘 인스턴스의 생성과 소멸에 대해 관심을 가지고 시스템 성능을 높일 수 있는 방향으로 구현해야 한다. 또한 중복 작업을 최소화하여 유지보수를 좋게 만드는 방법을 찾아야 한다. 디자인 패턴은 검증된 방법들을 체계적으로 분류하여 정의한 것이다.



설계자의 경험(객체 생성 기법, 클래스 구조, 객체 간의 교류방법) --적용-->실무에서 검증(시스템) --분류--> 디자인 패턴(Factory Method, Singleton, MVC, Front Controller)

## 프런트 컨트롤러 만들기


```java
package spms.servlets;
 
import java.io.IOException;
 
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import spms.vo.Member;
 
@SuppressWarnings("serial")
@WebServlet("*.do") //클라이언트 요청 중에서 서블릿 경로 이름이 .do로 끝나는 경우는 DispatcherServlet이 처리하겠다는 의미
public class DispatcherServlet extends HttpServlet {
    @Override
    protected void service(// service를 오버라이딩 한 이유는 GET, POST뿐만 아니라 다양한 요청 방식에도
                            // 대응하기 위해
            HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html; charset=UTF-8");
        String servletPath = request.getServletPath(); // 서블릿의 경로를 알아내기 위해 사용
        try {
            String pageControllerPath = null;
 
            // 서블릿 경로에 따라 조건문을 사용하여 적절한 페이지 컨트롤러를 인클루딩 한다.
            if ("/member/list.do".equals(servletPath)) {
                pageControllerPath = "/member/list";
            } else if ("/member/add.do".equals(servletPath)) {
                pageControllerPath = "/member/add";
                if (request.getParameter("email") != null) { //요청 매개변수로부터 VO 객체 준비
                    request.setAttribute("member", new Member().setEmail(request.getParameter("email"))
                            .setPassword(request.getParameter("password")).setName(request.getParameter("name")));
                }
            } else if ("/member/update.do".equals(servletPath)) {
                pageControllerPath = "/member/update";
                if (request.getParameter("email") != null) { //요청 매개변수로부터 VO 객체 준비
                    request.setAttribute("member", new Member().setNo(Integer.parseInt(request.getParameter("no")))
                            .setEmail(request.getParameter("email")).setName(request.getParameter("name")));
                }
            } else if ("/member/delete.do".equals(servletPath)) {
                pageControllerPath = "/member/delete";
            } else if ("/auth/login.do".equals(servletPath)) {
                pageControllerPath = "/auth/login";
            } else if ("/auth/logout.do".equals(servletPath)) {
                pageControllerPath = "/auth/logout";
            }
 
            RequestDispatcher rd = request.getRequestDispatcher(pageControllerPath);
            rd.include(request, response);
             
             
            String viewUrl = (String) request.getAttribute("viewUrl");
            if (viewUrl.startsWith("redirect:")) {
                response.sendRedirect(viewUrl.substring(9));
                return;
 
            } else {
                rd = request.getRequestDispatcher(viewUrl);
                rd.include(request, response);
            }
 
        } catch (Exception e) {
            e.printStackTrace();
            request.setAttribute("error", e);
            RequestDispatcher rd = request.getRequestDispatcher("/Error.jsp");
            rd.forward(request, response);
        }
    }
}

```
**요청 URL에서 서블릿 경로 알아내기
EX) http://localhost:9999/web06/member/list.do?pageNo=1&pageSize=10**

매서드  | 설명  | 변환값
------------- | -------------    | -------------
getRequestURL()         | 요청 URL 리턴(단, 매개변수 제외)| http://localhost:9999/web06/member/list.do
getRequestURI()            | 서버 주소를 제외한 URL   |	/web06/member/list.do 
getContextPath()     | 웹 애플리케이션 경로                | 	/web06 
getServletPath()   | 서블릿 경로               | /member/list.do 
getQueryString()  | 요청 매개변수 정보                | 	pageNo=1&pageSize=10



**MemberListServlet.java**
```java
package spms.servlets;
package spms.servlets;
 
import java.io.IOException;
 
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import spms.dao.MemberDao;
 
@WebServlet("/member/list")
public class MemberListServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
 
    @Override
    public void doGet(
            HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        try {
            ServletContext sc = this.getServletContext();
            MemberDao memberDao = (MemberDao)sc.getAttribute("memberDao");
             
            request.setAttribute("members", memberDao.selectList());
            request.setAttribute("viewUrl", "/member/MemberList.jsp"); //JSP URL 정보를 프런트 컨트롤러에게 알려주고자 ServletRequest 보관소에 저장
             
            //response.setContentType("text/html; charset=UTF-8"); // 응답 데이터의 문자 집합은 프런트 컨트롤러에서 이미 설정
            //RequestDispatcher rd = request.getRequestDispatcher("/member/MemberList.jsp"); //화면 출력을 위해 JSP로 실행을 위임하는 것도 프런트 컨트롤러가 처리
            //rd.include(request,  response);
             
        } catch (Exception e) {
            throw new ServletException(e); //service() 메서드는 ServletException을 던지도록 선언되어 있기 때문에 기존의 예외 객체를 그냥 던질 수 없다. 그래서 ServletException 객체를 생성한다.
            //e.printStackTrace(); //오류가 발생했을 때 오류 처리 페이지로 실행을 위임하는 작업도 프런트 컨트롤러가 한다.
            //request.setAttribute("error", e);
            //RequestDispatcher rd = request.getRequestDispatcher("/Error.jsp");
            //rd.forward(request, response);
        }
    }
}
```
**MemberAddServlet.java**

```java
package spms.servlets;
 
import java.io.IOException;
import java.sql.Connection;
 
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import spms.dao.MemberDao;
import spms.vo.Member;
 
@WebServlet("/member/add")
public class MemberAddServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
 
    @Override
    protected void doGet(
            HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        request.setAttribute("viewUrl", "/member/MemberAdd.jsp");
        //response.setContentType("text/html; charset=UTF-8");
        //RequestDispatcher rd = request.getRequestDispatcher("/member/MemberAdd.jsp");
        //rd.include(request, response);
    }
     
    @Override
    protected void doPost(
            HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
 
        try {
            ServletContext sc = this.getServletContext();
            MemberDao memberDao = (MemberDao)sc.getAttribute("memberDao");
             
            Member member = (Member)request.getAttribute("member");
            memberDao.insert(member);
             
            request.setAttribute("viewUrl", "redirect:list.do");
            //memberDao.insert(new Member()
            //      .setEmail(request.getParameter("email"))
            //      .setPassword(request.getParameter("password"))
            //      .setName(request.getParameter("name"))
            //      );
             
            //response.sendRedirect("list");
             
        } catch (Exception e) {
            throw new ServletException(e);
            //request.setAttribute("error", e);
            //RequestDispatcher rd = request.getRequestDispatcher("/Error.jsp");
            //rd.forward(request, response);
             
        }
    }
}
```
# 6.2 페이지 컨트롤러의 진화
![](./6_2.PNG)

- 프런트 컨트롤러를 도입하면 페이지 컨트롤러를 굳이 서블릿으로 만들어야 할 이유가 없다.

프런트 컨트롤러와 페이지 컨트롤러의 호출 규칙을 정의하도록 한다.

**페이지 컨트롤러를 위한 인터페이스 정의**
```java
package spms.controls;
 
import java.util.Map;
 
public interface Controller {
  //String은 JSP 경로 또는 리다이렉트 URL, model은 프런트 컨트롤러와 페이지 컨트롤러 사이에 데이터 교환을 위한 바구니 역할
  String execute(Map<String, Object> model) throws Exception;
}
```
**페이지 컨트롤러 MemberListServlet을 일반 클래스로 전환**
```java
package spms.controls;
 
import java.util.Map;
 
import javax.servlet.annotation.WebServlet;
 
import spms.dao.MemberDao;
 
@WebServlet("/member/list")
public class MemberListController implements Controller{
 
    @Override
    public String execute(Map<string, object> model) throws Exception {
        //페이지 컨트롤러에서 사용할 객체를 Map에서 꺼내기
        MemberDao memberDao = (MemberDao)model.get("memberDao");
        //페이지 컨트롤러가 작업한 결과물을 Map에 담기
        model.put("members",  memberDao.selectList());
        //뷰 URL 반환
        return "/member/MemberList.jsp";
    }
}
```
**프런트 컨트롤러 변경**
```java
package spms.servlets;
 
import java.io.IOException;
import java.util.HashMap;
 
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import spms.controls.Controller;
import spms.controls.LogInController;
import spms.controls.LogOutController;
import spms.controls.MemberAddController;
import spms.controls.MemberDeleteController;
import spms.controls.MemberListController;
import spms.controls.MemberUpdateController;
import spms.vo.Member;
 
@SuppressWarnings("serial")
@WebServlet("*.do") //클라이언트 요청 중에서 서블릿 경로 이름이 .do로 끝나는 경우는 DispatcherServlet이 처리하겠다는 의미
public class DispatcherServlet extends HttpServlet {
    @Override
    protected void service(// service를 오버라이딩 한 이유는 GET, POST뿐만 아니라 다양한 요청 방식에도
                            // 대응하기 위해
            HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html; charset=UTF-8");
        String servletPath = request.getServletPath(); // 서블릿의 경로를 알아내기 위해 사용
        try {
            //Map객체를 준비한다.
            ServletContext sc = this.getServletContext();
            HashMap<String, Object> model = new HashMap<String, Object>();
            model.put("memberDao", sc.getAttribute("memberDao"));
            model.put("session", request.getSession());
             
            //회원 목록을 처리할 페이지 컨트롤러 준비
            String pageControllerPath = null;
            Controller pageController = null;
            // 서블릿 경로에 따라 조건문을 사용하여 적절한 페이지 컨트롤러를 인클루딩 한다.
            if ("/member/list.do".equals(servletPath)) {
                pageController = new MemberListController();
                //pageControllerPath = "/member/list";
            } else if ("/member/add.do".equals(servletPath)) {
                pageController = new MemberAddController();
                if(request.getParameter("email")!=null){
                    model.put("member", new Member()
                            .setEmail(request.getParameter("email"))
                            .setPassword(request.getParameter("password"))
                            .setName(request.getParameter("name")));
                }
                     
                //pageControllerPath = "/member/add";
                //if (request.getParameter("email") != null) { //요청 매개변수로부터 VO 객체 준비
                //  request.setAttribute("member", new Member().setEmail(request.getParameter("email"))
                //          .setPassword(request.getParameter("password")).setName(request.getParameter("name")));
                //}
            } else if ("/member/update.do".equals(servletPath)) {
                pageController = new MemberUpdateController();
                if (request.getParameter("email") != null) {
                  model.put("member", new Member()
                    .setNo(Integer.parseInt(request.getParameter("no")))
                    .setEmail(request.getParameter("email"))
                    .setName(request.getParameter("name")));
                }else {
                      model.put("no", new Integer(request.getParameter("no")));
                }
                //pageControllerPath = "/member/update";
                //if (request.getParameter("email") != null) { //요청 매개변수로부터 VO 객체 준비
                //  request.setAttribute("member", new Member().setNo(Integer.parseInt(request.getParameter("no")))
                //          .setEmail(request.getParameter("email")).setName(request.getParameter("name")));
                }
            } else if ("/member/delete.do".equals(servletPath)) {
                pageController = new MemberDeleteController();
                model.put("no", new Integer(request.getParameter("no")));
                //pageControllerPath = "/member/delete";
            } else if ("/auth/login.do".equals(servletPath)) {
                pageController = new LogInController();
                if (request.getParameter("email") != null) {
                    model.put("loginInfo", new Member()
                            .setEmail(request.getParameter("email"))
                            .setPassword(request.getParameter("password")));
                }
                //pageControllerPath = "/auth/login";
            } else if ("/auth/logout.do".equals(servletPath)) {
                pageController = new LogOutController();
                //pageControllerPath = "/auth/logout";
            }
 
            //RequestDispatcher rd = request.getRequestDispatcher(pageControllerPath);
            //rd.include(request, response);
             
            //페이지 컨트롤러의 실행, MemberListController가 일반 클래스이기 때문에 메서드를 호출해야 한다.
            String viewUrl = pageController.execute(model);
             
            //Map 객체에 저장된 값을 ServletRequest에 복사
            for(String key : model.keySet()){
                request.setAttribute(key,  model.get(key));
            }
             
            if (viewUrl.startsWith("redirect:")) {
                response.sendRedirect(viewUrl.substring(9));
                return;
            } else {
                RequestDispatcher rd = request.getRequestDispatcher(viewUrl);
                rd.include(request, response);
            }
 
        } catch (Exception e) {
            e.printStackTrace();
            request.setAttribute("error", e);
            RequestDispatcher rd = request.getRequestDispatcher("/Error.jsp");
            rd.forward(request, response);
        }
    }
}
```
# 6.3 DI를 이용한 빈 의존성 관리
>> Controller가 작업을 수행하려면 데이터베이스로부터 회원 정보를 가져다 줄 Dao가 필요하다. 이렇게 특정 작업을 수행할 때 사용하는 객체를 '의존 객체'라고 하고, 이런 관계를 DI(의존 관계)라고 한다.

>> 의존 객체의 관리
- 의존 객체가 필요하면 즉시 생성 -> 문제점: 많은 가비지 생성, 실행 시긴 지연
- 의존 객체를 미리 생성해 두었다가 필요할 때 사용

# 의존 객체와의 결합도 증가에 따른 문제
>> 의존 객체를 직접 생성하거나 보관소에서 꺼내는 방식으로 관리하면 문제가 발생한다.
- 코드의 잦은 변경(의존 객체를 사용하는 쪽과 의존 객체(또는 보관소( 사이의 결합도가 높아져서 의존 객체나 보관소에 변경이 발생하면 바로 영향을 받는다.)
- 대체가 어렵다(의존 객체를 다른 객체로 대체하기가 어렵다. 예:DB가 달라지는 경우)

**의존 객체를 외부에서 주입**

>> 초창기 객체 지향 프로그래밍에서는 의존 객체를 직접 생성하였다. 그러다가 문제점을 해결하기 위해 의존 객체를 외부에서 주입받는(미리 주입해 두기) 방식으로 바뀌게 된다.

![](./6_3.png)

>> 의존 객체를 전문으로 관리하는 '빈 컨테이너'가 등장하게 되었다. 빈 컨테이너는 객체가 실행되기 전에 그 객체가 필요로 하는 의존 객체를 주입해주는 역할을 수행한다. 이런 방식으로 객체를 관리하는 것을 '의존성 주입'(DI)라고 한다. 역 제어(IoC)라고도 한다.

```java
package spms.controls;
 
import java.util.Map;
 
 
import spms.dao.MemberDao;
 
public class MemberListController implements Controller{
    MemberDao memberDao;
    //MemberDao를 주입 받기 위한 인스턴스 변수와 setter 메서드를 추가함.
    public MemberListController setMemberDao(MemberDao memberDao){
        this.memberDao = memberDao;
        return this;
    }
     
    @Override
    public String execute(Map<String, Object> model) throws Exception {
        //외부에서 MemberDao 객체를 주입해 줄 것이기 때문에 이제 더 이상 Map 객체에서 MemberDao를 꺼낼 필요가 없다.
        //MemberDao memberDao = (MemberDao)model.get("memberDao");
         
        //페이지 컨트롤러가 작업한 결과물을 Map에 담기
        model.put("members",  memberDao.selectList());
        //뷰 URL 반환
        return "/member/MemberList.jsp";
    }
}
```
**페이지 컨트롤러 객체들을 준비**  
ContextLoaderListener.java
```java
@Override
public void contextInitialized(ServletContextEvent event) {
  try {
    ServletContext sc = event.getServletContext();
 
    InitialContext initialContext = new InitialContext();
    DataSource ds = (DataSource) initialContext.lookup("java:comp/env/jdbc/studydb");
 
 
 
    MemberDao memberDao = new MemberDao();
    // DataSource를 주입
    memberDao.setDataSource(ds);
 
    //sc.setAttribute("memberDao", memberDao);
     
    //페이지 컨트롤러 객체를 생성하고 나서 MemberDao가 필요한 객체에 대해서는 setter메서드를 호출하여 주입해 준다.
    //ServletContext에 저장(키는 서블릿 요청 URL로하여 바인딩)
    sc.setAttribute("/auth/login.do", new LogInController().setMemberDao(memberDao));
    sc.setAttribute("/auth/logout.do", new LogOutController());
    sc.setAttribute("/member/list.do", new MemberListController().setMemberDao(memberDao));
    sc.setAttribute("/member/add.do", new MemberAddController().setMemberDao(memberDao));
    sc.setAttribute("/member/update.do", new MemberUpdateController().setMemberDao(memberDao));
    sc.setAttribute("/member/delete.do", new MemberDeleteController().setMemberDao(memberDao));
 
  } catch (Throwable e) {
    e.printStackTrace();
  }
}
```
**프런트 컨트롤러의 변경**
```java
@Override
    protected void service(// service를 오버라이딩 한 이유는 GET, POST뿐만 아니라 다양한 요청 방식에도
                            // 대응하기 위해
            HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html; charset=UTF-8");
        String servletPath = request.getServletPath(); // 서블릿의 경로를 알아내기 위해 사용
        try {
            //Map객체를 준비한다.
            ServletContext sc = this.getServletContext();
            HashMap<String, Object> model = new HashMap<String, Object>();
            //model.put("memberDao", sc.getAttribute("memberDao"));
            model.put("session", request.getSession());
 
                       //페이지 컨트롤러는 ServletContext 보관소에 저장되어 있다. 그래서 이 보관소에서 페이지 컨트롤러를 꺼낼 때 서블릿 url을 사용한다.
       
            Controller pageController = (Controller)sc.getAttribute(servletPath);
             
            //Controller pageController = null;
 
            if ("/member/add.do".equals(servletPath)) {
                //pageController = new MemberAddController();
                if(request.getParameter("email")!=null){
                    model.put("member", new Member()
                            .setEmail(request.getParameter("email"))
                            .setPassword(request.getParameter("password"))
                            .setName(request.getParameter("name")));
                }
            } else if ("/member/update.do".equals(servletPath)) {
                //pageController = new MemberUpdateController();
                if (request.getParameter("email") != null) {
                  model.put("member", new Member()
                    .setNo(Integer.parseInt(request.getParameter("no")))
                    .setEmail(request.getParameter("email"))
                    .setName(request.getParameter("name")));
                }else {
                      model.put("no", new Integer(request.getParameter("no")));
                }
```
# 인터페이스를 활용하여 공급처를 다변화 하자
![](./6_4.png)
>> 실무에서는 데이터베이스가 다를 수 있기 때문에 각 데이터베이스에 맞추어 DAO 클래스를 준비해야 한다 데이터베이스를 바꿀 때마다  DAO 클래스를 사용하는 코드도 변경해야 하기 때문에 이런 경우에 인터페이스를 활용하면 해결 할 수 있다. 인터페이스로 DAO가 갖춰야 할 규격을 정의하고, 그규격을 준수하는 클래스라면 어떤 클래스를 상속받았는지 따지지 않고 허용한다.

사용할 의존 객체에 대해 선택 폭을 넓히고 싶고 향후 확장을 고려하여 교체하기 쉽게 만들고 싶다면 인터페이스 문법을 통해 사용 규칙을 정의하는게 좋다.

**MemberDao 인터페이스 정의**
```java
package spms.dao;
 
import java.util.List;
 
import spms.vo.Member;
 
public interface MemberDao {
    List<Member> selectList() throws Exception;
    int insert(Member member) throws Exception;
    int delete(int no) throws Exception;
    Member selectOne(int no) throws Exception;
    int update(Member member) throws Exception;
    Member exist(String email, String password) throws Exception;
}
```
**MySqlMemberDao 클래스를 MemberDao 규격에 맞추기**
```java
public class MySqlMemberDao implements MemberDao {
```
추가적으로 ContextLoaderListener.java도 수정해준다.

# 6.4 리플랙션 API를 이용하여 프런트 컨트롤러 개선하기
>> 
지금까지는 서블릿 컨텍스트 보관소에서 페이지 컨트롤러를 찾아서, 페이지 컨트롤러가 누구냐에 따라 파라미터 값을 직접 꺼내 값 객체에 할당 해주었다. 문제점은 새 페이지 컨트롤러가 추가되면, 프런트 컨트롤러에 값 객체를 준비하는 조건문을 추가해야 한다.

```java
if ("/member/update.do".equals(servletPath)) {
                //pageController = new MemberUpdateController();
        if (request.getParameter("email") != null) {
          model.put("member", new Member()
                .setNo(Integer.parseInt(request.getParameter("no")))
                    .setEmail(request.getParameter("email"))
                    .setName(request.getParameter("name")));
        }else{
          model.put("no", new Integer(request.getParameter("no")));
    }
}
//매개변수 값에 대해 VO객체를 준비해야한다.
//데이터를 사용하는 페이지 컨트롤러를 추가할 때마다 계속 프런트 컨트롤러를 변경해야 하는 문제가 발생한다. 유지보수를 어렵게 만든다.
```
**페이지 컨트롤러를 추가하더라도 프런트 컨트롤러를 변경하지 않으려면?**  
>> 구조를 변경 해야한다. 
값 객체를 직접 생성하는 코드를 제거한다.
페이지 컨트롤러에게 필요한 데이터가 무엇인지 물어보고(메서드 호출:데이터바인딩할) 그에 맞춰 준비해준다.
프런트 컨트롤러와 페이지 컨트롤러 사이에 호출 규칙 정의한다.

**신규 회원 정보 추가 자동화**

**DataBinding 인터페이스 정의**
>> 프런트 컨트롤러가 페이지 컨트롤러를 실행하기 전에 원하는 데이터가 무엇인지 묻기 위해서 인터페이스에 호출 규칙을 정의한다

```java
package spms.bind;
 
public interface DataBinding {
    //new Object[]{"데이터이름", 데이터타입, "데이터이름", 데이터타입, ///}
    Object[] getDataBinders();
}
```
**페이지 컨트롤러의 DataBinding 구현**
MemberAddController.java의 예
```java
package spms.controls;
 
import java.util.Map;
 
import spms.bind.DataBinding;
import spms.dao.MySqlMemberDao;
import spms.vo.Member;
 
public class MemberAddController implements Controller, DataBinding {
    MySqlMemberDao memberDao;
 
    //클라이언트가 보낸 매개변수 값을 Member 인스턴스에 담아서 "member"라는 이름으로 Map객체에 저장해라라는 뜻.
    //프런트 컨트롤러는 Object 배열에 지정된 대로 Member 인스턴스를 준비하여 Map 객체에 저장하고, execute()를 호출할 때 매개변수로 이 Map 객체를 넘긴다.
    public Object[] getDataBinders() {
        return new Object[]{
                "member", spms.vo.Member.class
        };
    }
 
    public MemberAddController setMemberDao(MySqlMemberDao memberDao) {
        this.memberDao = memberDao;
        return this;
    }
 
    @Override
    public String execute(Map<string, object=""> model) throws Exception {
        //getDataBinders()에서 지정한 대로 프런트 컨트롤러가 VO객체를 무조건 생성할 것이기 때문에 Member가 있는지 여부로 판단해서는 안 된다. 그래서 email이 null인지 체크한다.
        Member member = (Member)model.get("member");
        if (member.getEmail() == null) {
            return "/member/MemberAdd.jsp";
        } else {
            memberDao.insert(member);
            return "redirect:list.do";
        }
    }
}
</string,>
```
Login, Update, Delete도 수정

**프런트 컨트롤러의 변경**
DispatcherServlet.java
```java
package spms.servlets;
 
import java.io.IOException;
import java.util.HashMap;
 
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
 
import spms.bind.DataBinding;
import spms.bind.ServletRequestDataBinder;
import spms.controls.Controller;
 
@SuppressWarnings("serial")
@WebServlet("*.do") //클라이언트 요청 중에서 서블릿 경로 이름이 .do로 끝나는 경우는 DispatcherServlet이 처리하겠다는 의미
public class DispatcherServlet extends HttpServlet {
    @Override
    protected void service(// service를 오버라이딩 한 이유는 GET, POST뿐만 아니라 다양한 요청 방식에도
                            // 대응하기 위해
            HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html; charset=UTF-8");
        String servletPath = request.getServletPath(); // 서블릿의 경로를 알아내기 위해 사용
        try {
            ServletContext sc = this.getServletContext();
            HashMap<String, Object> model = new HashMap<String, Object>();
            model.put("session", request.getSession());
             
            Controller pageController = (Controller)sc.getAttribute(servletPath);
             
            //매개변수 값을 사용하는 페이지 컨트롤러를 추가하더라도 조건문을 삽입할 필요가 없다. 대신 데이터 준비를 자동으로 수행하는 prepareRequestData()를 호출한다.
            //DataBinding을 구현했는지 여부를 검사하여, 해당 인터페이스를 구현한 경우에만 prepareRequestData()를 호출하여 페이지컨트롤러를 위한 데이터를 준비한다.
            if(pageController instanceof DataBinding){
                prepareRequestData(request, model, (DataBinding)pageController);
            }
             
            String viewUrl = pageController.execute(model);
             
            for(String key : model.keySet()){
                request.setAttribute(key,  model.get(key));
            }
             
            if (viewUrl.startsWith("redirect:")) {
                response.sendRedirect(viewUrl.substring(9));
                return;
            } else {
                RequestDispatcher rd = request.getRequestDispatcher(viewUrl);
                rd.include(request, response);
            }
 
        } catch (Exception e) {
            e.printStackTrace();
            request.setAttribute("error", e);
            RequestDispatcher rd = request.getRequestDispatcher("/Error.jsp");
            rd.forward(request, response);
        }
    }
 
    private void prepareRequestData(HttpServletRequest request, HashMap<String, Object> model,
            DataBinding dataBinding) throws Exception {
        //컨트롤러에게 필요한 데이터가 무엇인지 묻는다.
        Object[] dataBinders = dataBinding.getDataBinders();
        String dataName = null;
        Class<?> dataType = null;
        Object dataObj = null;
         
        //데이터 이름과 타입을 꺼내기 쉽게 2씩 증가한다.
        for(int i=0; i<dataBinders.length; i+=2){
            dataName = (String)dataBinders[i];
            dataType = (Class<?>)dataBinders[i+1];
            dataObj = ServletRequestDataBinder.bind(request, dataType, dataName);
            model.put(dataName, dataObj);
        }
    }
}
```
**ServletRequestDataBinder.java**

```java
package spms.bind;
 
import java.lang.reflect.Method;
import java.util.Date;
import java.util.Set;
 
import javax.servlet.ServletRequest;
 
//클라이언트가 보낸 매개변수 값을 자바 객체에 담아 주는 역할을 수행한다.
public class ServletRequestDataBinder {
    public static Object bind(ServletRequest request, Class<?> dataType, String dataName) throws Exception {
        //dataType이 기본 타입인지 아닌지 검사한다. 기본 타입이라면 즉시 객체를 생성하여 반환하도록 한다.
        if (isPrimitiveType(dataType)) {
            return createValueObject(dataType, request.getParameter(dataName));
        }
 
        Set<String> paramNames = request.getParameterMap().keySet();
        Object dataObject = dataType.newInstance();
        Method m = null;
 
        for (String paramName : paramNames) {
            m = findSetter(dataType, paramName);
            if (m != null) {
                m.invoke(dataObject, createValueObject(m.getParameterTypes()[0], request.getParameter(paramName)));
            }
        }
        return dataObject;
    }
 
    //isPrimitiveTrype메서드는 매개변수로 주어진 타입이 기본 타입인지 검사하는 메서드이다.
    private static boolean isPrimitiveType(Class<?> type) {
        if (type.getName().equals("int") || type == Integer.class || type.getName().equals("long") || type == Long.class
                || type.getName().equals("float") || type == Float.class || type.getName().equals("double")
                || type == Double.class || type.getName().equals("boolean") || type == Boolean.class
                || type == Date.class || type == String.class) {
            return true;
        }
        return false;
    }
 
    private static Object createValueObject(Class<?> type, String value) {
        if (type.getName().equals("int") || type == Integer.class) {
            return new Integer(value);
        } else if (type.getName().equals("float") || type == Float.class) {
            return new Float(value);
        } else if (type.getName().equals("double") || type == Double.class) {
            return new Double(value);
        } else if (type.getName().equals("long") || type == Long.class) {
            return new Long(value);
        } else if (type.getName().equals("boolean") || type == Boolean.class) {
            return new Boolean(value);
        } else if (type == Date.class) {
            return java.sql.Date.valueOf(value);
        } else {
            return value;
        }
    }
 
    private static Method findSetter(Class<?> type, String name) {
        Method[] methods = type.getMethods();
 
        String propName = null;
        for (Method m : methods) {
            if (!m.getName().startsWith("set"))
                continue;
 
            propName = m.getName().substring(3);
            if (propName.toLowerCase().equals(name.toLowerCase())) {
                return m;
            }
        }
        return null;
    }
}
```

# 리플랙션 API
>> 클래스에 어떤 메서드가 있는지, 메서드의 이름은 무엇인지, 클래스의 이름은 무엇인지 알 수 있게 해준다. 클래스나 메서드의 내부 구조를 들여다 볼 때 사용하는 도구

- 위에서 사용한 리플랙션 API


메서드   | 설명   
------------- | -------------   
Class.newInstance()           | 클래스 정보만 있다면 리플렉션 API를 이용하게되면 인스턴스를 생성할 수 있다.             
Class.getName()          | 클래스의 이름을 반환              
Class.getMethods()     | 클래스에 선언된 모든 public 메서드의 목록을 배열로 반환  
Method.invoke()      | 해당 메서드를 호출 
Method.getParameterTypes()    | 메서드의 매개변수 목록을 배열로 반환

# 6.5 프로퍼티를 이용한 객체 관리
>> 페이지 컨트롤러뿐만 아니라 DAO를 추가하는 경우에도 ContextLoaderListener 클래스에 코드를 추가해야한다.

DAO나 페이지 컨트롤러가 추가되더라도 ContextLoaderList 클래스를 변경하는 방법? -> 구조변경

**프로퍼티 파일 작성**
>> 준비해야 할 객체 정보를 외부 파일로 분리하자(.property)

객체 정보는 key = value 형태로 작성.

- 이 규칙은 이 미니 프레임워크에만 해당한다.
- 톰캣 서버에서 제공하는 객체
- jndi.{객체이름}={JNDI이름} //ex : jndi.dataSource=java:comp/env/jdbc/studydb
- 일반 객체
- {객체이름}={패키지 이름을 포함한 클래스 이름} // ex: /auth/login.do=spms.controls.LogInController

**application-context.properties**

```java
jndi.dataSource=java:comp/env/jdbc/studydb
memberDao=spms.dao.MySqlDao
/auth/login.do=spms.controls.LogInController
/auth/logout.do=spms.controls.LogOutController
/member/list.do=spms.controls.MemberListController
/member/add.do=spms.controls.MemberAddController
/member/update.do=spms.controls.MemberUpdateController
/member/delete.do=spms.controls.MemberDeleteController
```
**ApplicationContext 클래스**
>> .properties 파일에 작성된 내용에 따라 객체를 준비할 자동화 클래스 추가(property 파일의 정보를 읽어서 객체를 생성하고 보관하는 역할)

ApplicationContext.java
```java
package spms.context;
 
import java.io.FileReader;
import java.lang.reflect.Method;
import java.util.Hashtable;
import java.util.Properties;
 
import javax.naming.Context;
import javax.naming.InitialContext;
 
// 프로퍼티 파일을 이용한 객체 준비
public class ApplicationContext {
    //프로퍼티에 설정된 대로 객체를 준비하면, 객체를 저장할 보관소가 필요한데 이를 위해 해시 테이블을 준비한다.
    Hashtable<String, Object> objTable = new Hashtable<String, Object>();
 
    //해시테이블에서 객체를 꺼낼 getter 메서드도 정의한다.
    public Object getBean(String key) {
        return objTable.get(key);
    }
       
    public ApplicationContext(String propertiesPath) throws Exception {
        //Properties는 '이름=값' 형태로 된 파일을 다룰 때 사용하는 클래스이다.
        Properties props = new Properties();
         
        //Properties의 load() 메서드는 FileReader를 통해 읽어들인 프로퍼티 내용을 키-값 형태로 내부 맵에 보관한다.
        props.load(new FileReader(propertiesPath));
         
        prepareObjects(props);
        injectDependency();
    }
 
    //프로퍼티 파일의 내용을 로딩했으면 객체를 준비해야한다. prepareObjects()
    private void prepareObjects(Properties props) throws Exception {
        //JNDI 객체를 찾을 때 사용할 InitialContext를 준비한다.
        Context ctx = new InitialContext();
        String key = null;
        String value = null;
 
        //반복문을 통해 프로퍼티에 들어있는 정보를 꺼내서 객체를 생성한다.
        for (Object item : props.keySet()) {
            key = (String) item;
            value = props.getProperty(key);
            //플고퍼티의 키가 "jndi."로 시작한다면 객체를 생성하지 않고 InitialContext를 통해 얻는다.
            if (key.startsWith("jndi.")) {
                //JNDI 인터페이스를 통해 톰캣 서버에 등록된 객체를 찾아 objTable객체 테이블에 저장된다.
                objTable.put(key, ctx.lookup(value));
            } else {
                //그밖의 객체는 Class.forName()을 호출하여 클래스를 로딩하고, newInstance()를 사용하여 인스턴스를 생성해서 objTable객체 테이블에 저장된다.
                objTable.put(key, Class.forName(value).newInstance());
            }
        }
    }
 
    //톰캣 서버로부터 객체를 가져오거나(ex: DataSource) 직접 객체를 생성했으면(ex: MemberDao), 각 객체가 필요로 하는 의존 객체를 할당한다.
    private void injectDependency() throws Exception {
        for (String key : objTable.keySet()) {
            //"jndi."로 시작하는 것 제외
            if (!key.startsWith("jndi.")) {
                callSetter(objTable.get(key));
            }
        }
    }
 
    //매개변수로 주어진 객체에 대해 setter 메서드를 찾아서 호출하는 일을 한다.
    private void callSetter(Object obj) throws Exception {
        Object dependency = null;
        for (Method m : obj.getClass().getMethods()) {
            //setter 메서드를 찾는다.
            if (m.getName().startsWith("set")) {
                //setter메서드의 매개변수와 타입이 일치하는 객체를 objTable에서 찾는다.
                dependency = findObjectByType(m.getParameterTypes()[0]);
                //의존 객체를 찾았으면, setter메서드를 호출한다.
                if (dependency != null) {
                    m.invoke(obj, dependency);
                }
            }
        }
    }
 
    //setter 메서드를 호출할 때 넘겨줄 의존 객체를 찾는 일을 한다. objTable에 들어 있는 객체를 모두 찾는다.
    private Object findObjectByType(Class<?> type) {
        for (Object obj : objTable.values()) {
            //주어진 객체가 해당 클래스 또는 인터페이스의 인스턴스인지 찾고 그 객체의 주소를 리턴한다.
            if (type.isInstance(obj)) {
                return obj;
            }
        }
        return null;
    }
}
```

>> ContextLoaderListener는 객체 생성 및 관리를 ApplicationContext에게 위임

ContextLoaderListener.java

```java
package spms.listeners;
 
import javax.servlet.ServletContext;
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;
 
import spms.context.ApplicationContext;
 
@WebListener
public class ContextLoaderListener implements ServletContextListener {
    static ApplicationContext applicationContext;
     
    //ContextLoaderListener에서 만든 ApplicationContext 객체를 얻을 때 사용한다.
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
      }
     
    @Override
    public void contextInitialized(ServletContextEvent event) {
        try {
            ServletContext sc = event.getServletContext();
 
            //프로퍼티 파일의 이름과 경로 정보도 web.xml 파일로부터 읽어 오게 처리되었다.
            //ServletContext의 getInitParameter()를 호출하여 web.xml에 설정된 매개변수 정보를 가져온다.
            String propertiesPath = sc.getRealPath(
                      sc.getInitParameter("contextConfigLocation"));
             
            //Application 객체는 프런트 컨트롤러에서 사용할 수 있게 ContextLoaderListener의 클래스 변수 'applicationContext'에 저장한다.
            applicationContext = new ApplicationContext(propertiesPath);
             
            //InitialContext initialContext = new InitialContext();
            //DataSource ds = (DataSource) initialContext.lookup("java:comp/env/jdbc/studydb");
 
            //MySqlMemberDao memberDao = new MySqlMemberDao();
            //memberDao.setDataSource(ds);
             
            //sc.setAttribute("/auth/login.do", new LogInController().setMemberDao(memberDao));
            //sc.setAttribute("/auth/logout.do", new LogOutController());
            //sc.setAttribute("/member/list.do", new MemberListController().setMemberDao(memberDao));
            //sc.setAttribute("/member/add.do", new MemberAddController().setMemberDao(memberDao));
            //sc.setAttribute("/member/update.do", new MemberUpdateController().setMemberDao(memberDao));
            //sc.setAttribute("/member/delete.do", new MemberDeleteController().setMemberDao(memberDao));
 
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
 
    @Override
    public void contextDestroyed(ServletContextEvent event) {
        // context.xml에 이미 closeMethod 속성이 있어서 톰캣 서버가 종료되면 close를 알아서 해준다.
    }
}
```

>> web.xml 추가
```java
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/application-context.properties</param-value>
</context-param>
```
>> DispatcherServlet.java 수정
```java
protected void service(// service를 오버라이딩 한 이유는 GET, POST뿐만 아니라 다양한 요청 방식에도
                            // 대응하기 위해
            HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html; charset=UTF-8");
        String servletPath = request.getServletPath(); // 서블릿의 경로를 알아내기 위해 사용
        try {
            //ServletContext sc = this.getServletContext();
            ApplicationContext ctx = ContextLoaderListener.getApplicationContext();
            HashMap<String, Object> model = new HashMap<String, Object>();
            model.put("session", request.getSession());
             
            //Controller pageController = (Controller)sc.getAttribute(servletPath);
            Controller pageController = (Controller)ctx.getBean(servletPath);
            if(pageController == null){
                throw new Exception("요청한 서비스를 찾을 수 없습니다.");
            }
             
            if(pageController instanceof DataBinding){
                prepareRequestData(request, model, (DataBinding)pageController);
            }

```

# 6.6 애노테이션을 이용한 객체 관리
![](./6_5.png)

>> 기존방식 -> 프로퍼티를 이용한 방식

Dao 또는 페이지 컨트롤러를 추가할 때마다 프로퍼티 파일을 변경해야 한다.

해결방안  
- 애노테이션을 이용하여 클래스 파일에 객체 정보를 담는다.
- DAO 객체 이름을 클래스 파일에 보관
- 페이지 컨트롤러의 서블릿 경로 정보를 클래스 파일에 보관
- DAO 객체나 페이지 컨트롤러를 추가할 때도 단지 애노테이션만 클래스에 선언해 주면 된다.

**1단계** 객체 이름을 보관할 애노테이션 정의(유지정책:RUNTIME)  
**2단계** DAO 및 페이지 컨트롤러에 애노테이션 적용  
**3단계** 프로퍼티 파일변경  
**4단계** applicationcontext 클래스 변경  
참고 : 클래스 파일에서 애노테이션 정보를 추출할 때 Reflections 오픈 소스 라이브러리 사용(getTypesAnnotationedWith(Component.class)

>> 애노테이션 활용
'애노테이션'은 컴파일이나 배포, 실행할 때 참조할 수 있는 아주 특별한 주석이다. 애노테이션을 사용하면 클래스나 필요, 메서드에 대해 부가 정보를 등록할 수 있다.


**애노테이션 정의**  
Component.java
```java
package spms.annotation;
 
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
 
//애노테이션 유지 정책
//애노테이션 정보를 언제까지 유지할 것인지 설정하는 문법이다.
@Retention(RetentionPolicy.RUNTIME)
//애노테이션 문법은 인터페이스 문법과 비슷하다. interface 키워드 앞에 @가 붙는다.
public @interface Component {
    //객체 이름을 저장하는 용도로 사용할 'value'라는 기본 속성을 정의합니다. value 속성은 값을 설정할 때 이름을 생략할 수 있다.
    //인터페이스의 메서드와 달리 '기본값'을 지정할 수 있다. 속성 선언 다음에 오는 'default'키워드가 기본값을 지정하는 문법이다.
    //value 속성의 값을 지정하지 않으면 default로 지정한 값이 할당 된다.
    String value() default "";
}
```

## 애노테이션 유지 정책
정책  | 설명 
------------- | -------------  
RetentionPolicy.SOURCE         | 소스 파일에서만 유지, 컴파일할 때 제거됨, 즉 클래스 파일에 애노테이션 정보가 남아 있지 않음              
RetentionPolicy.CLASS           | 클래스 파일에 기록됨, 실행 시에는 유지되지 않음, 즉 실행 중에서는 클래스에 기록된 애노테이션 값을 꺼낼 수 없음(기본 정책)            
RetentionPolicy.RUNTIME     |클래스 파일에 기록됨. 실행 시에도 유지됨. 즉, 실행 중에 클래스에 기록된 애노테이션 값을 참조할 수 있음              

>> 애노테이션 적용

...
@Component("memberDao")
public class MySqlMemberDao implements MemberDao {
...

**페이지 컨트롤러에도 적용**  
@Component("/member/list.do")

**프로퍼티 파일 변경**  

**ApplicationContext 변경**  
@Component 애노테이션이 붙은 클래스에 대해서도 객체를 생성하도록 ApplicationContext를 변경한다.

...
    // 자바 classpath를 뒤져서 @Component 애노테이션이 붙은 클래스를 찾는다.
    // 그리고 그 객체를 생성하여 객체 테이블에 담는 일을 한다. Reflections 오픈 소스 라이브러리를 활용했다.
    private void prepareAnnotationObjects() throws Exception {
        // Reflections 클래스는 원하는 클래스를 찾아 주는 도구이다. 생성자에 넘겨 주는 매개변수 값은 클래스를 찾을 때
        // 출발하는 패키지이다.
        // new Reflections(""); 빈 문자열은 자바 classpath에 있는 모든 패키지를 검색하라는 뜻
        Reflections reflector = new Reflections("");
 
        //getTypesAnnotatedWith()는 애노테이션이 붙은 클래스들을 찾을 수 있다.
        //@Component 애노테이션이 붙은 클래스를 찾고 싶다는 뜻이다.
        Set<Class<?>> list = reflector.getTypesAnnotatedWith(Component.class);
        String key = null;
        for (Class<?> clazz : list) {
            //getAnnotation()을 통해 클래스로부터 애노테이션을 추출한다.
            //@Component의 기본 속성값을 꺼내고 싶으면, 다음과 같이 value()를 호추한다.
            key = clazz.getAnnotation(Component.class).value();
            //애노테이션을 통해 알아낸 객체 이름(key)으로 인스턴스를 저장한다.
            objTable.put(key, clazz.newInstance());
        }
    }
...

## Reflections 라이브러리 준비
https://code.google.com/p/reflections  
reflections-0.9.9-RC1-uberjar.jar(압축풀어 루트에 있는 jar파일들만 WEB-INF/lib에 넣기)  
reflections-0.9.9-RC1.jar(WEB-INF/lib에 넣기)  





