# `FirstServlet.java` 설명

## 1. 초심자용 실생활 비유

이 서블릿은 "가게 입구에서 손님을 맞이하는 직원"처럼 생각하면 됩니다.

- 브라우저가 손님
- `"/first"` 주소는 가게의 특정 창구 번호
- `FirstServlet`은 그 창구를 맡은 직원
- `doGet()`은 손님이 "안녕하세요" 하고 들어왔을 때 응대하는 말하기 규칙
- `HttpServletRequest`는 손님이 들고 온 주문서
- `HttpServletResponse`는 직원이 손님에게 돌려주는 응답서

즉, 브라우저가 `GET /first`를 요청하면 Tomcat이 "아, 이 요청은 `FirstServlet` 창구로 보내야겠네" 하고 연결해 줍니다. 그 다음 `doGet()`이 실행되고, 서블릿이 화면에 보여줄 내용을 직접 작성합니다.

`resp.getWriter().println(...)`은 손님에게 말을 적어서 건네는 것과 비슷합니다.  
`resp.setContentType("text/html; charset=utf-8")`는 "이 응답은 HTML이고 한글도 제대로 표시하라"는 안내문을 붙이는 역할입니다.

---

## 2. 중급자용 실제 일어나는 일과 의존성, 문법 특성

### 실제 실행 흐름

1. Tomcat이 애플리케이션을 시작하거나 요청을 받습니다.
2. `@WebServlet("/first")`를 스캔해서 `/first` URL과 `FirstServlet`을 매핑합니다.
3. 브라우저가 `/first`로 GET 요청을 보냅니다.
4. Tomcat이 `HttpServlet` 기반의 서블릿 인스턴스를 찾고, 없으면 생성합니다.
5. `doGet(HttpServletRequest req, HttpServletResponse resp)`가 호출됩니다.
6. `resp.getWriter()`로 응답 본문을 쓰고, Tomcat이 최종 HTTP 응답을 브라우저로 보냅니다.

### 의존성 관계

이 파일은 다음 타입들에 의존합니다.

- `jakarta.servlet.http.HttpServlet`
- `jakarta.servlet.http.HttpServletRequest`
- `jakarta.servlet.http.HttpServletResponse`
- `jakarta.servlet.ServletException`
- `jakarta.servlet.annotation.WebServlet`
- `java.io.IOException`

여기서 핵심은 `jakarta.servlet-api`가 외부에서 제공되는 API라는 점입니다. 보통 Tomcat이 런타임에 제공하므로, 직접 구현하는 코드에서는 API 타입만 가져다 씁니다.

### 문법 특성

- `@WebServlet("/first")`
  - 어노테이션 기반 서블릿 등록입니다.
  - `web.xml` 대신 코드에 URL 매핑을 선언합니다.

- `extends HttpServlet`
  - 서블릿을 직접 만들 때 가장 흔한 방식입니다.
  - `HttpServlet`이 HTTP 메서드별 분기 구조를 제공하고, 개발자는 `doGet`, `doPost` 등을 오버라이드합니다.

- `@Override`
  - 상위 클래스 메서드를 재정의한다는 뜻입니다.
  - 시그니처가 조금만 달라도 컴파일 단계에서 문제를 찾는 데 도움이 됩니다.

- `protected void doGet(...) throws ServletException, IOException`
  - `GET` 요청 전용 처리 메서드입니다.
  - 체크 예외를 선언하므로 서블릿 컨테이너가 예외 흐름을 처리할 수 있습니다.

- `resp.getWriter().println(...)`
  - `PrintWriter`를 통해 응답 본문을 씁니다.
  - HTML 태그를 문자열로 직접 써서 반환할 수 있습니다.

- `resp.setContentType("text/html; charset=utf-8")`
  - 브라우저에게 응답 형식과 문자 인코딩을 알려줍니다.
  - 한글 문자열을 출력할 때 특히 중요합니다.

### 코드상 주의점

현재 코드에서는 `println()`이 `setContentType()`보다 먼저 호출됩니다.  
실제 응답 헤더는 보통 첫 출력 전에 정해지는 편이므로, 일반적으로는 `setContentType()`을 먼저 호출하는 것이 더 안전합니다.

권장 순서는 다음과 같습니다.

1. `resp.setContentType("text/html; charset=utf-8")`
2. `resp.getWriter().println(...)`

---

## 3. 면접 준비를 위한 예상 질문

### 기본 질문

1. `FirstServlet`은 어떤 역할을 하나요?
2. `@WebServlet("/first")`는 무엇을 의미하나요?
3. `HttpServlet`을 상속하는 이유는 무엇인가요?
4. `doGet()`은 언제 호출되나요?
5. `HttpServletRequest`와 `HttpServletResponse`의 차이는 무엇인가요?

### 중급 질문

1. `web.xml` 방식과 `@WebServlet` 방식의 차이는 무엇인가요?
2. 서블릿 인스턴스는 요청마다 새로 생성되나요, 아니면 재사용되나요?
3. `getWriter()`를 사용해서 응답을 쓰는 구조는 어떻게 동작하나요?
4. `setContentType("text/html; charset=utf-8")`를 왜 지정해야 하나요?
5. `doGet()` 안에서 상태를 필드에 저장하면 어떤 문제가 생길 수 있나요?

### 심화 질문

1. 서블릿은 멀티스레드 환경에서 어떻게 동작하나요?
2. 서블릿의 생명주기 `init`, `service`, `destroy`는 어떤 순서로 실행되나요?
3. `println()`보다 `print()`를 써야 하는 경우가 있나요?
4. 응답 헤더와 응답 바디는 언제 확정되나요?
5. 한글이 깨지는 원인은 무엇이고, 어떻게 예방하나요?

### 면접에서 답변할 때 잡아야 할 핵심

- 이 서블릿은 특정 URL 요청을 처리하는 HTTP 엔드포인트다.
- `@WebServlet`은 URL 매핑을 코드에 선언하는 방법이다.
- `HttpServlet`을 상속하면 HTTP 메서드별 처리 구조를 재사용할 수 있다.
- 요청 정보는 `request`, 응답 작성은 `response`가 담당한다.
- 응답의 콘텐츠 타입과 문자 인코딩을 명확히 지정해야 한다.

