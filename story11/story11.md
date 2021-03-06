# Story11. JSP와 서블릿, Spring에서 발생할 수 있는 여러 문제점

## JSP와 Servlet의 기본적인 동작 원리는 꼭 알아야 한다.  

### JSP의 라이프 사이클
<pre>
1. JSP URL 호출
2. 페이지 번역
3. JSP 페이지 컴파일
4. 클래스 로드
5. 인스턴스 생성
6. jspInit 메소드 호출
7. _jspService 메소드 호출
8. jspDestory 메소드 호출  
</pre>
해당 jsp 페이지가 이미 컴파일되어 있고, 클래스가 로드되어 있고, JSP파일이 변경되지 않았다면,   
가장 많은 시간이 소요되는 2~4 프로세스는 생략된다.

### 서블릿의 라이프 사이클
<pre>
* Servlet 객체 [생성], [초기화]
* [사용가능] 상태로 대기
* 예외가 발생하면 [사용불가능] 상태로 빠졌다가 다시 [사용가능] 상태로 변환되기도 함
* 해당 서블릿이 더 이상 필요 없을 때는 [파기] 상태로 넘어간 후 JVM에서 [제거]  
</pre>
서블릿은 JVM에 살아 있고, 여러 스레드에서 해당 서블릿의 service() 메소드를 호출하여 공유한다.   
*service()메소드를 구현할 때는 멤버 변수나 static한 클래스 변수를 선언하여 지속적으로 변경하는 작업은 피하기 바란다.*

### 적절한 include 사용하기
* 정적인 방식: JSP 페이지 번역 및 컴파일 단계에서 필요한 JSP를 읽어서 메인 JSP의 자바 소스 및 클래스에 포함을 시키는 방식.
* 동적인 방식: 페이지가 호출될 때마다 지정된 페이지를 불러들여서 수행하도록 되어 있다.
* 동적인 방식이 더 느리나,  
정적인 방식을 사용하면 메인 JSP에 추가되는 JSP가 생긴다.  
*이 때 추가된 JSP와 메인 JSP에 동일한 이름의 변수가 있으면 심각한 오류가 발생할 수 있다.*

### 자바 빈즈, 잘 쓰면 약 못 쓰면 독
* 자바 빈즈: UI에서 서버 측 데이터를 담아서 처리하기 위한 컴포넌트.  
* 너무 많이 사용하면 JSP에서 소요되는 시간 증가  
*시간을 줄이기 위해서는 TO 클래스를 만들어 사용하자.*

### 태그 라이브러리도 잘 써야 한다
* 태그 라이브러리: JSP에서 공통적으로 반복되는 코드를 클래스로 만들고, 그 클래스를 HTML 태그와 같이 정의된 태그로 사용할 수 있도록 하는 라이브러리.
* 태그 라이브러리 클래스를 잘못 작성하거나 태그 라이브러리 클래스로 전송되는 데이터가 많을 때 성능 저하.  
*목록을 처리하면서 대용량의 데이터를 처리할 경우 태그 라이브러리의 사용을 자제하기 바란다.*

## 스프링 프레임워크 간단 정리
* 스프링 프레임워크: 데스크톱과 웹 애플리케이션, 간단한 애플리케이션부터 엔터프라이즈 애플리케이션도 범용적인 애플리케이션 프레임워크.

### 특징
* 복잡한 애플리케이션도 POJO로 개발할 수 있다.
* 그로 인해 개발자가 보다 쉽게 자신이 작성한 코드를 테스트 할 수 있다. 그래서 더 빠르고 쉽게 문제를 확인할 수 있으며 이는 곧 높은 개발 생산성으로 이어진다.

### 스프링의 핵심 기술
* Dependency Injection: '의존성 주입'  
객체 간의 의존 관계를 관리하는 기술.  
어떤 객체가 필요로 하는 객체를 자기 자신이 직접 생성하여 사용하는 것이 아니라 외부에 있는 다른 무언가로부터 필요로 하는 객체를 주입 받는 기술. 

```java
public class A{
  private B b;
  public A(B b){
    this.b = b;
  }
}
```

스프링은 의존성을 쉽게 주입하는 틀을 제공해 준다.

* AOP: '관점 지향 프로그래밍'   
OOP(객체지향 프로그래밍 Object Oriented Programming)를 보다 OOP스럽게 보완해주는 기술.  
트랜잭션, 로깅 등 이런 코드들은 여러 모듈, 여러 계층에 스며들기 마련이다. 이런 코드를 실제 비즈니스 로직과 분리할 수 있도록 도와주는 것이 바로 AOP다.  
Spring은 사용하기 쉬운 스프링 AOP를 제공한다. 이를 사용하면 핵심 비즈니스 코드의 가독성을 높여준다.

* PSA
오픈 소스로 제공되는 자바 라이브러리는 매우 많다. 이렇게 비슷한 개술을 구현하기 위해 코딩하는 방법은 사용할 라이브러리나 프레임워크에 따라 달라지기 때문에,
추상화가 매우 중요하다.  
스프링은 비슷한 기술을 모두 아우를 수 잇는 추상화 계층을 제공하여, 여러분이 사용하는 기술이 바뀌더라도 비즈니스 로직의 변화가 없도록 도와준다.  

### 스프링 프레임워크를 사용하면서 발생할 수 있는 문제점
* 프록시
스프링 프록시는 기본적으로 실행 시에 생성된다. 요청량이 많은 운영 상황으로 넘어가면 문제가 나타날 수 있다.
@Transaction 어노테이션을 사용하면 해당 어노테이션을 사용한 클래스의 인스턴스를 처음 만들 때 프록시 객체를 만든다.  
개발자가 직접 스프링 AOP를 사용해서 별도의 기능을 추가하는 경우에도 프록시를 사용  
*간단한 부하 툴을 사용해서라도 성능적인 면을 테스트해야만 한다.*

* 스프링이 내부 매커니즘에서 사용하는 캐시
이미 찾아 본 뷰 객체를 캐싱해두면 다음에도 동일한 문자열이 반환됐을 때 훨씬 빠르게 뷰 객체를 찾을 수 있다.  
InternalResourceViewREsorvler에는 그러한 캐싱 기능이 내장되어 있다.  
*매번 다른 문자열이 생성될 가능성이 높은 경우 뷰 이름을 문자열로 반환하기보다는 뷰 객체 자체를 반환하는 방법이 메모리 릭을 방지하는데 도움이 된다.*

## 정리하며
1. include 구문도 정적인 구문을 사용할지, 동적인 구문을 사용할지 잘 선택해야 한다.
2. 자바 빈즈를 너무 많이 사용하는 것은 성능에 많은 영향을 줄 수 있다.
3. 태그 라이브러리도 상황에 맞게 사용해야 한다.
4. 오류가 발생했을 때 사용자가 즉시 문제 요청을 할 수 있게 하고, 담당자가 문제를 쉽게 확인할 수 있도록 구성해야 한다.
5. 스프링 프레임워크에서 제공하는 각 어노테이션이 어떤 의미인지 정확히 알고 사용해야만 한다.


