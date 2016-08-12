---
layout: post
title:  "서블릿이란?"
date:   2016-08-12 20:00:00 +0900
tags: [jsp]
---

*서블릿은 웹서버가 동적인 페이지를 제공할 수 있도록 도와주는 애플리케이션이다.*


웹서버에서는 정적인 페이지만을 제공한다. 그렇기 때문에 동적인 페이지를 제공하기 위해서 웹서버는 다른 무언가에게 도움을 요청하여 동적인 페이지를 작성해야한다. 동적인 페이지란 임의의 이미지를 골라 사용자에게 보여준다거나, 도서 목록이나 방문 로그 등 사용자가 요청한 시점에 페이지를 생성해서 전달해주는 것을 의미한다.

이렇게 웹서버를 도와서 동적인 페이지를 만들어주는 도우미 애플리케이션을 CGI 프로그램이라고 부른다. 대부분의 CGI 프로그램은 Perl 스크립트로 작성되는데, C, 파이썬, PHP 와 같은 언어로도 가능하다.

서블릿은 CGI 프로그램과 같은 도우미 애플리케이션인데, 자바로 구현되어있다.(그러므로 개발도 자바로 한다.) 펄을 사용하는 CGI 프로그램과 같은 경우에는 요청마다 프로세스를 띄워야하기 때문에 성능 상의 이슈가 발생할 수 있지만 서블릿은 오직 한번만 구동되어 요청이 올때마다 스레드가 생성된다. 즉, JVM이 주는 부하나 클래스를 메모리에 로딩하는 부하도 줄일 수 있다.

아파치와 같은 웹 서버가 사용자로부터 서블릿에 대한 요청을 받으면, 서블릿을 바로 호출하는 것이 아니라, 서블릿을 관리하고 있는 컨테이너에게 요청을 넘긴다. 요청을 넘겨받은 컨테이너는 HTTP Request와 HTTP Response 객체를 만들어, 이를 인자로 서블릿 doPost()나 doGet() 메서드 중 하나를 호출한다.

만약 서블릿만 사용해서 사용자가 요청한 웹 페이지를 보여주려면, out 객체의 println() 메서드를 사용해서 HTML 문서를 작성해야 한다. 이는 추가/수정 작업을 어렵게 만들고, 가독성 또한 떨어지게 된다. JSP를 사용하게 되면 비즈니스 로직과 프리젠테이션 로직을 분리할 수 있는데 서블릿에서는 사용자로부터 요청을 받아 데이터베이스를 조회하거나 데이터를 입력, 수정 후 요청에 대한 제어를 JSP에게 넘겨서 프리젠테이션 로직을 수행한 후 컨테이너에게 response를 전달한다.

이렇게 만들어진 결과물은 사용자가 해당 페이지를 요청하면 컴파일 되어 java 파일을 통해 .class파일이 만들어지는데 해당 내용을 보면 비즈니스 로직과 프리젠테이션 로직이 결합되어 클래스화 된 것을 확인할 수 있다. out 객체의 println() 메서드를 사용해서 구현하는 번거로움을 jsp가 대신 해주고 있는 것이다.

예를 들어 아래와 같은 jsp파일을 작성하여,

{% highlight html %}
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String name = "Yongho";
%>
<html>
<head>
    <title>${name}</title>
</head>
<body>
<h1>Hello world!</h1>
<p>My name is ${name}</p>
</body>
</html>
{% endhighlight %}

톰캣에서 구동시켜 보면 페이지 요청시 컴파일되어 실행되는데 만들어진 java파일을 열어보면 아래와 같은 코드를 볼 수 있다.

{% highlight java %}
public final class index_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent {

  ... 생략 ...

  public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
        throws java.io.IOException, javax.servlet.ServletException {

    final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;


    try {
      response.setContentType("text/html;charset=UTF-8");
      pageContext = _jspxFactory.getPageContext(this, request, response,
      			null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;

      out.write('\r');
      out.write('\n');

    String name = "Yongho";

      out.write("\r\n");
      out.write("<html>\r\n");
      out.write("<head>\r\n");
      out.write("    <title>");
      out.write((java.lang.String) org.apache.jasper.runtime.PageContextImpl.proprietaryEvaluate("${name}", java.lang.String.class, (javax.servlet.jsp.PageContext)_jspx_page_context, null, false));
      out.write("</title>\r\n");
      out.write("</head>\r\n");
      out.write("<body>\r\n");
      out.write("<h1>Hello world!</h1>\r\n");
      out.write("<p>My name is ");
      out.write((java.lang.String) org.apache.jasper.runtime.PageContextImpl.proprietaryEvaluate("${name}", java.lang.String.class, (javax.servlet.jsp.PageContext)_jspx_page_context, null, false));
      out.write("</p>\r\n");
      out.write("</body>\r\n");
      out.write("</html>\r\n");
    } catch (java.lang.Throwable t) {
      if (!(t instanceof javax.servlet.jsp.SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          try {
            if (response.isCommitted()) {
              out.flush();
            } else {
              out.clearBuffer();
            }
          } catch (java.io.IOException e) {}
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
        else throw new ServletException(t);
      }
    } finally {
      _jspxFactory.releasePageContext(_jspx_page_context);
    }
  }
}
{% endhighlight %}

## 참고

* [Head First Servlets & JSP]

[Head First Servlets & JSP]: http://goo.gl/bMCLzV
