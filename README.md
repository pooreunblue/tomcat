# 🐱 Java Servlet + Tomcat 학습 프로젝트

> Jakarta EE Servlet API와 Apache Tomcat을 직접 다루며 Java 웹 애플리케이션의 동작 원리를 이해하는 학습용 프로젝트입니다.

---

## 📁 프로젝트 구조

```
tomcat/
├── pom.xml                          # Maven 빌드 설정 파일 (의존성, 플러그인 정의)
├── mvnw / mvnw.cmd                  # Maven Wrapper (Maven 설치 없이 빌드 가능)
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/tomcat/
│   │   │       └── HelloServlet.java    # ✅ 핵심: HTTP 요청을 처리하는 서블릿 클래스
│   │   ├── resources/               # 설정 파일, 프로퍼티 등 (현재 비어 있음)
│   │   └── webapp/
│   │       ├── index.jsp            # 웹 애플리케이션 시작 페이지 (JSP)
│   │       └── WEB-INF/
│   │           └── web.xml          # 웹 애플리케이션 배포 서술자 (Deployment Descriptor)
│   └── test/
│       ├── java/                    # 테스트 코드 위치 (현재 비어 있음)
│       └── resources/               # 테스트용 리소스
└── target/                          # 빌드 결과물 (컴파일된 클래스, WAR 파일 등)
```

---

## 🔑 핵심 파일 상세 설명

### 1. `HelloServlet.java`
```java
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {
    private String message;

    public void init() { message = "Hello World!"; }       // 서블릿 초기화
    public void doGet(...) { /* GET 요청 처리 */ }          // 핵심 비즈니스 로직
    public void destroy() { }                               // 서블릿 소멸 시 자원 정리
}
```
- `HttpServlet`을 상속받아 HTTP 요청을 처리하는 클래스입니다.
- `@WebServlet` 어노테이션으로 URL 매핑(`/hello-servlet`)을 선언합니다.
- 서블릿의 생명주기(init → service → destroy)를 직접 구현합니다.

### 2. `index.jsp`
- JSP(JavaServer Pages) 파일로, 서블릿과 달리 HTML 안에 Java 코드를 삽입합니다.
- `<%= "Hello World!" %>` : 표현식 태그로 값을 출력합니다.
- `/hello-servlet` 링크를 통해 `HelloServlet`으로 요청을 전달합니다.

### 3. `WEB-INF/web.xml`
- **배포 서술자(Deployment Descriptor)** 로, 웹 애플리케이션 전체 설정을 담습니다.
- Jakarta EE 6.0 스펙을 따릅니다.
- `@WebServlet` 어노테이션 방식을 사용하면 `web.xml`에 별도 서블릿 등록이 불필요합니다.

### 4. `pom.xml`
| 의존성 | 역할 |
|--------|------|
| `jakarta.servlet-api:6.1.0` | Servlet API (컴파일 전용, 런타임은 Tomcat 제공) |
| `junit-jupiter-api` | JUnit 5 테스트 프레임워크 |
| `junit-jupiter-engine` | JUnit 5 실행 엔진 |
| `maven-war-plugin` | WAR 파일 패키징 플러그인 |

> ⚠️ `jakarta.servlet-api`의 scope가 **`provided`** 인 이유: Tomcat 서버가 이미 Servlet API를 제공하므로 빌드 시에만 필요하고 WAR 파일에는 포함하지 않습니다.

---

## 🧠 초심자를 위한 핵심 개념 정리

### 🌐 Web Server vs WAS (Web Application Server)

```
클라이언트(브라우저)
      ↓ HTTP 요청
┌─────────────────────┐
│  Web Server         │  ← 정적 파일(HTML, CSS, 이미지) 처리
│  (Apache httpd, Nginx)│
└──────────┬──────────┘
           ↓ 동적 요청 전달
┌─────────────────────┐
│  WAS (Tomcat)       │  ← 동적 콘텐츠(Servlet, JSP) 처리
│  - Servlet Container│
│  - JSP Engine       │
└─────────────────────┘
```

- **Web Server**: 정적 파일 제공에 특화 (Apache, Nginx)
- **WAS**: 동적 요청 처리 + 비즈니스 로직 실행 (Tomcat, JBoss, WebLogic)
- **Tomcat**은 WAS이면서 경량 Web Server 기능도 포함하는 **서블릿 컨테이너**입니다.

---

### 🔄 Servlet 생명주기 (Lifecycle)

```
최초 요청 또는 서버 시작
         ↓
   [1] 클래스 로딩 (ClassLoader)
         ↓
   [2] 인스턴스 생성 (new HelloServlet())  ← 딱 1번만!
         ↓
   [3] init() 호출                         ← 초기화, 딱 1번만!
         ↓
   [4] service() 호출                      ← 요청마다 반복 (멀티스레드)
      ├── doGet()   ← GET 요청
      ├── doPost()  ← POST 요청
      └── doPut() 등
         ↓
   [5] destroy() 호출                      ← 서버 종료 시 1번
```

> 💡 **핵심**: 서블릿 인스턴스는 **단 하나(싱글톤)**만 생성되고, **멀티스레드**로 동시 요청을 처리합니다.
> 따라서 서블릿 클래스의 **인스턴스 변수(필드)는 공유 상태**가 됩니다.

---

### ⚡ Servlet vs JSP 비교

| 구분 | Servlet | JSP |
|------|---------|-----|
| 파일 형식 | `.java` | `.jsp` |
| 코드 스타일 | Java 코드 안에 HTML 삽입 | HTML 안에 Java 코드 삽입 |
| 주 용도 | 비즈니스 로직, 컨트롤러 역할 | 뷰(화면) 담당 |
| 컴파일 | 개발자가 직접 컴파일 | 서버가 자동으로 Servlet으로 변환·컴파일 |
| MVC 역할 | Controller (M, C) | View (V) |

---

### 📦 WAR 파일 구조와 배포

```
myapp.war
├── index.jsp
├── WEB-INF/
│   ├── web.xml               # 배포 서술자
│   ├── classes/              # 컴파일된 .class 파일
│   │   └── com/example/tomcat/HelloServlet.class
│   └── lib/                  # 의존 JAR 파일 (scope=compile인 것들)
└── META-INF/
    └── MANIFEST.MF
```

- **WAR(Web Application Archive)**: 웹 애플리케이션을 배포하기 위한 표준 패키징 형식입니다.
- Tomcat의 `webapps/` 디렉토리에 WAR 파일을 복사하면 자동으로 배포됩니다.
- `WEB-INF` 안의 파일은 **클라이언트가 직접 접근 불가** (보안 디렉토리)

---

### 🗺️ HTTP 요청 처리 흐름

```
브라우저: GET /hello-servlet
      ↓
Tomcat: URL을 보고 web.xml 또는 @WebServlet 으로 매핑 확인
      ↓
HelloServlet 인스턴스 찾기 (없으면 init() 실행 후 생성)
      ↓
service() → doGet(request, response) 호출
      ↓
response.getWriter()로 HTML 작성 → 클라이언트에 응답
      ↓
브라우저: HTML 렌더링
```

---

### 🔀 `@WebServlet` vs `web.xml` 방식 비교

**어노테이션 방식 (현재 프로젝트)**
```java
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet { ... }
```

**web.xml 방식**
```xml
<servlet>
    <servlet-name>helloServlet</servlet-name>
    <servlet-class>com.example.tomcat.HelloServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>helloServlet</servlet-name>
    <url-pattern>/hello-servlet</url-pattern>
</servlet-mapping>
```

| 구분 | 어노테이션 | web.xml |
|------|----------|---------|
| 간결성 | ✅ 코드와 설정이 한 곳에 | ❌ 별도 파일 관리 |
| 유연성 | ❌ 재배포 없이 변경 불가 | ✅ XML만 수정하면 적용 가능 |
| 도입 버전 | Servlet 3.0+ | 모든 버전 |

---

### 🔒 Jakarta EE vs Java EE

| 구분 | Java EE | Jakarta EE |
|------|---------|-----------|
| 관리 주체 | Oracle | Eclipse Foundation |
| 패키지명 | `javax.*` | `jakarta.*` |
| 버전 전환 | ~ Java EE 8 | Jakarta EE 8+ |
| 현재 프로젝트 | ❌ | ✅ (6.1.0 사용) |

> 2019년 Oracle이 `javax` 네임스페이스 권리를 Eclipse에 이전하지 않아, `jakarta`로 패키지명이 변경되었습니다.

---

## 🏗️ MVC 패턴과 서블릿

이 프로젝트는 MVC 패턴의 기초가 되는 구조를 보여줍니다.

```
Model      → 비즈니스 데이터/로직 (현재 프로젝트에서는 message 필드)
View       → index.jsp (화면 출력 담당)
Controller → HelloServlet.java (요청을 받아 Model 처리 후 View에 전달)
```

실무에서는 이 패턴이 Spring MVC의 `DispatcherServlet`으로 발전합니다.

---

## 🐳 Docker로 실행하기

이 저장소에는 멀티스테이지 빌드를 사용하는 `Dockerfile`이 포함되어 있습니다.

### Dockerfile 구조

1. `maven:3-eclipse-temurin-17` 이미지에서 애플리케이션을 빌드합니다.
2. `tomcat:10-jre17-temurin` 이미지에 생성된 WAR 파일을 복사합니다.
3. Tomcat 기본 포트를 `8080` 대신 `10000`으로 변경합니다.
4. 셧다운 포트 `8005`는 비활성화합니다.

### 빌드

```bash
docker build -t tomcat-demo .
```

### 실행

```bash
docker run --rm -p 10000:10000 tomcat-demo
```

브라우저에서 아래 주소로 접속합니다.

```text
http://localhost:10000
```

### 참고

- Docker 컨테이너 내부 Tomcat 포트는 `10000`입니다.
- WAR 파일은 `ROOT.war`로 배포되므로 별도 컨텍스트 경로 없이 루트 경로로 접속합니다.

---

## 🚀 프로젝트 실행 방법

### 사전 준비
- JDK 8 이상
- Apache Tomcat 10.x 이상 (Jakarta EE 지원)
- Maven 또는 Maven Wrapper(mvnw) 사용 가능

### 빌드 및 배포
```bash
# 1. WAR 파일 빌드
./mvnw clean package

# 2. target/tomcat-1.0-SNAPSHOT.war 를 Tomcat의 webapps/ 에 복사

# 3. Tomcat 시작
$CATALINA_HOME/bin/startup.sh   # macOS/Linux
$CATALINA_HOME/bin/startup.bat  # Windows

# 4. 브라우저에서 접속
# http://localhost:8080/tomcat-1.0-SNAPSHOT/
# http://localhost:8080/tomcat-1.0-SNAPSHOT/hello-servlet
```

---

## 🎯 면접 대비 예상 질문 & 답변

### 📌 서블릿 (Servlet) 기초

---

**Q1. 서블릿(Servlet)이란 무엇인가요?**

> 서블릿은 Java EE(Jakarta EE) 표준에서 정의한 **서버 측 Java 컴포넌트**로, 클라이언트의 HTTP 요청을 처리하고 동적인 응답을 생성합니다.
> `HttpServlet` 클래스를 상속받아 구현하며, **서블릿 컨테이너(Tomcat)**가 생명주기를 관리합니다.
> 핵심 특징은 멀티스레드 환경에서 단일 인스턴스로 동작한다는 점입니다.

---

**Q2. 서블릿의 생명주기(Lifecycle)를 설명해주세요.**

> 1. **로딩 & 인스턴스 생성**: 최초 요청 시(또는 서버 시작 시) 서블릿 클래스를 로드하고 인스턴스를 딱 하나 생성합니다.
> 2. **`init()`**: 초기화 메서드로 딱 1회 호출됩니다. DB 커넥션, 설정값 로딩 등 초기 자원을 할당합니다.
> 3. **`service()`**: 요청마다 호출되며, HTTP 메서드에 따라 `doGet()`, `doPost()` 등으로 분기합니다.
> 4. **`destroy()`**: 서버 종료나 서블릿 제거 시 1회 호출됩니다. 자원을 반납합니다.

---

**Q3. 서블릿이 멀티스레드 환경에서 인스턴스 변수를 사용하면 어떤 문제가 발생하나요?**

> 서블릿은 싱글톤으로 동작하므로 **인스턴스 변수는 모든 스레드(요청)가 공유**합니다.
> 예를 들어 `HelloServlet`의 `message` 필드를 요청마다 다르게 설정하면 **레이스 컨디션(Race Condition)**이 발생해 예상치 못한 결과가 나올 수 있습니다.
>
> **해결책**: 상태를 로컬 변수(스택)에 저장하거나, `ThreadLocal`을 사용하거나, 상태 없는(Stateless) 설계를 유지합니다.

---

**Q4. `doGet()`과 `doPost()`의 차이는 무엇인가요?**

| 구분 | GET | POST |
|------|-----|------|
| 데이터 위치 | URL 쿼리스트링 | HTTP Body |
| 보안 | URL에 노출됨 | 상대적으로 안전 |
| 데이터 크기 | URL 길이 제한 (~2KB) | 제한 없음 |
| 캐싱 | 캐싱 가능 | 캐싱 불가 |
| 멱등성 | ✅ 멱등 | ❌ 비멱등 |
| 주 용도 | 데이터 조회 | 데이터 생성/수정 |

---

**Q5. `web.xml`의 역할은 무엇이며, 어노테이션 방식과 어떤 차이가 있나요?**

> `web.xml`은 **배포 서술자(Deployment Descriptor)**로, 서블릿 매핑, 필터, 리스너, 보안 설정 등 웹 애플리케이션 전반의 설정을 XML로 정의합니다.
> - **어노테이션 방식(`@WebServlet`)**: Servlet 3.0부터 도입. 코드와 설정이 함께 있어 간편하지만, 변경 시 재컴파일이 필요합니다.
> - **web.xml 방식**: 설정을 외부화할 수 있어 재배포 없이 변경 가능합니다. 대형 프로젝트나 환경별 설정이 필요한 경우 유리합니다.

---

### 📌 JSP (JavaServer Pages)

---

**Q6. JSP가 실행되는 내부 원리를 설명해주세요.**

> JSP 파일은 클라이언트가 최초 요청할 때 **서블릿 컨테이너가 자동으로 Java 서블릿 코드로 변환(translation)하고 컴파일(compilation)합니다**.
> 이후 요청부터는 컴파일된 서블릿 클래스가 실행됩니다. 즉, JSP는 결국 Servlet입니다.
>
> **JSP → Java(Servlet) → .class → 실행** 순서입니다.

---

**Q7. JSP에서 `<% %>`, `<%= %>`, `<%! %>`, `<%@ %>`의 차이는 무엇인가요?**

| 태그 | 이름 | 역할 | 예시 |
|------|------|------|------|
| `<% %>` | 스크립틀릿 | Java 코드 실행 | `<% int a = 1; %>` |
| `<%= %>` | 표현식 | 값을 출력 | `<%= a + 1 %>` |
| `<%! %>` | 선언문 | 멤버 변수/메서드 선언 | `<%! int count = 0; %>` |
| `<%@ %>` | 지시자 | 페이지 설정 | `<%@ page contentType="text/html" %>` |

---

### 📌 Tomcat & 서블릿 컨테이너

---

**Q8. 서블릿 컨테이너(Servlet Container)란 무엇이며, 어떤 역할을 하나요?**

> 서블릿 컨테이너는 **서블릿의 생명주기를 관리하고 HTTP 요청/응답을 처리하는 런타임 환경**입니다.
> Tomcat이 대표적입니다.
>
> 주요 역할:
> - **서블릿 생명주기 관리** (init, service, destroy)
> - **HTTP 요청 파싱 및 `HttpServletRequest`/`HttpServletResponse` 객체 생성**
> - **URL과 서블릿 매핑**
> - **멀티스레드 처리** (스레드 풀 관리)
> - **세션 관리**
> - **보안(SSL/TLS, 인증)** 지원

---

**Q9. Web Server와 WAS의 차이점은 무엇인가요?**

> - **Web Server**(Apache httpd, Nginx): **정적 콘텐츠**(HTML, CSS, 이미지, JS 파일)를 빠르게 제공하는 데 특화되어 있습니다. 동적 요청은 처리하지 못합니다.
> - **WAS**(Tomcat, JBoss, WebLogic): **동적 콘텐츠**를 생성합니다. Servlet, JSP, EJB 등을 실행하여 비즈니스 로직을 처리합니다.
>
> 실무에서는 **Nginx(정적) + Tomcat(동적)** 조합으로 많이 사용합니다.

---

**Q10. WAR 파일이란 무엇인가요?**

> **WAR(Web Application Archive)**는 Java 웹 애플리케이션을 배포하기 위한 표준 패키지 형식(.war)입니다.
> JAR 파일처럼 ZIP 형식이지만, 웹 표준 디렉토리 구조(`WEB-INF/`, `META-INF/` 등)를 따릅니다.
> Tomcat의 `webapps/` 폴더에 복사하면 자동으로 압축 해제 및 배포됩니다.

---

### 📌 Java EE / Jakarta EE

---

**Q11. Jakarta EE와 Java EE의 차이는 무엇인가요?**

> Java EE는 Oracle이 관리하던 엔터프라이즈 Java 플랫폼입니다.
> 2017년 Oracle이 Java EE를 Eclipse Foundation에 기부하면서 **Jakarta EE**로 이름이 바뀌었습니다.
> 가장 큰 변화는 패키지명이 **`javax.*` → `jakarta.*`** 로 변경된 것입니다.
> Oracle이 `javax` 네임스페이스에 대한 상표권을 이전하지 않았기 때문입니다.
>
> Spring Boot 3.x부터 Jakarta EE 9+ 기반으로 전환되어, `javax.servlet` 대신 `jakarta.servlet`을 사용합니다.

---

**Q12. `jakarta.servlet-api`의 Maven scope가 `provided`인 이유는 무엇인가요?**

> Servlet API는 **Tomcat 서버가 이미 포함하고 있는 라이브러리**입니다.
> 만약 `compile` scope(기본값)으로 설정하면 Servlet API가 WAR 파일 안에도 포함되어,
> 런타임에 Tomcat이 제공하는 버전과 **충돌(ClassLoader 충돌)**이 발생할 수 있습니다.
>
> `provided` scope는 "컴파일할 때는 필요하지만, 실행 환경(컨테이너)이 제공하므로 빌드 결과물에는 포함하지 않는다"는 의미입니다.

---

### 📌 HTTP & 웹 기초

---

**Q13. HTTP 요청/응답의 구조를 설명해주세요.**

> **요청(Request)**:
> ```
> GET /hello-servlet HTTP/1.1       ← 요청 라인 (메서드, URL, 버전)
> Host: localhost:8080              ← 헤더
> Accept: text/html                 ← 헤더
>                                   ← 빈 줄 (헤더 끝)
> (GET은 Body 없음)                  ← 바디
> ```
>
> **응답(Response)**:
> ```
> HTTP/1.1 200 OK                   ← 상태 라인 (버전, 상태코드, 메시지)
> Content-Type: text/html           ← 헤더
>                                   ← 빈 줄
> <html><body>...</body></html>     ← 바디
> ```

---

**Q14. HTTP 상태 코드를 설명해주세요.**

| 범위 | 의미 | 주요 코드 |
|------|------|----------|
| 2xx | 성공 | 200(OK), 201(Created), 204(No Content) |
| 3xx | 리다이렉션 | 301(영구 이동), 302(임시 이동), 304(Not Modified) |
| 4xx | 클라이언트 오류 | 400(Bad Request), 401(Unauthorized), 403(Forbidden), 404(Not Found) |
| 5xx | 서버 오류 | 500(Internal Server Error), 503(Service Unavailable) |

---

**Q15. 세션(Session)과 쿠키(Cookie)의 차이는 무엇인가요?**

| 구분 | 쿠키(Cookie) | 세션(Session) |
|------|-------------|--------------|
| 저장 위치 | 클라이언트(브라우저) | 서버 메모리 |
| 보안 | 상대적으로 취약 | 상대적으로 안전 |
| 용량 | 4KB 제한 | 서버 메모리 한도까지 |
| 만료 | 설정된 시간 | 브라우저 종료 또는 타임아웃 |
| 서버 부하 | 없음 | 있음 (서버가 저장) |
| 주 용도 | 자동 로그인, 사용자 선호 설정 | 로그인 인증 상태 유지 |

> 서블릿에서는 `request.getSession()`으로 세션을 관리하고, 쿠키는 `Cookie` 클래스를 사용합니다.

---

## 📚 더 깊이 공부하려면

- [Jakarta Servlet 명세](https://jakarta.ee/specifications/servlet/)
- [Apache Tomcat 공식 문서](https://tomcat.apache.org/tomcat-10.1-doc/)
- **다음 단계**: Spring MVC → Spring Boot → REST API 설계 → JPA/Hibernate

---

## 🛠️ 기술 스택

![Java](https://img.shields.io/badge/Java-8-007396?style=flat-square&logo=openjdk)
![Jakarta EE](https://img.shields.io/badge/Jakarta_EE-6.1.0-F80000?style=flat-square)
![Apache Tomcat](https://img.shields.io/badge/Apache_Tomcat-10.x-F8DC75?style=flat-square&logo=apachetomcat)
![Maven](https://img.shields.io/badge/Maven-3.x-C71A36?style=flat-square&logo=apachemaven)
![JUnit 5](https://img.shields.io/badge/JUnit-5.13.2-25A162?style=flat-square&logo=junit5)
