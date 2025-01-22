spring-web 모듈은 반응형 웹 애플리케이션을 위한 다음과 같은 핵심 지원을 포함합니다:

1. **서버 요청 처리**  
    두 가지 수준의 지원을 제공합니다:
    
    - **HttpHandler**: 비동기 I/O 및 Reactive Streams의 백프레셔(back pressure)를 지원하는 HTTP 요청 처리를 위한 기본 계약. Reactor Netty, Undertow, Tomcat, Jetty 및 모든 Servlet 컨테이너용 어댑터 포함.
    - **WebHandler API**: 요청 처리를 위한 범용 웹 API로, 주석 기반 컨트롤러와 함수형 엔드포인트 같은 구체적 프로그래밍 모델이 이 위에 구축됩니다.
2. **클라이언트 측 지원**
    
    - 비동기 I/O 및 Reactive Streams 백프레셔를 사용해 HTTP 요청을 수행하는 기본 계약 **ClientHttpConnector** 제공. Reactor Netty, Jetty HttpClient(반응형), Apache HttpComponents용 어댑터 포함.
    - 애플리케이션에서 사용되는 상위 레벨의 **WebClient**는 이 기본 계약을 기반으로 구축됩니다.
3. **클라이언트 및 서버**  
    HTTP 요청 및 응답 콘텐츠 직렬화와 역직렬화를 위한 코덱 제공.
    

---

### HttpHandler

**HttpHandler**는 단일 메서드로 요청과 응답을 처리하는 간단한 계약입니다. 이 인터페이스는 의도적으로 최소화되어 있으며, 다양한 HTTP 서버 API 위에서 최소한의 추상화를 제공하는 것을 주요 목적으로 합니다.

#### 지원하는 서버 API

아래 표는 지원되는 서버 API를 설명합니다:

|서버 이름|사용 API|Reactive Streams 지원|
|---|---|---|
|**Netty**|Netty API|Reactor Netty|
|**Undertow**|Undertow API|spring-web: Undertow에서 Reactive Streams로의 브리지|
|**Tomcat**|Servlet 비동기 I/O, Tomcat API|spring-web: Servlet 비동기 I/O에서 Reactive Streams로의 브리지|
|**Jetty**|Servlet 비동기 I/O, Jetty API|spring-web: Servlet 비동기 I/O에서 Reactive Streams로의 브리지|
|**Servlet 컨테이너**|Servlet 비동기 I/O|spring-web: Servlet 비동기 I/O에서 Reactive Streams로의 브리지|

#### 서버 종속성

다음 표는 서버 종속성과 지원 버전을 나타냅니다:

|서버 이름|Group ID|Artifact 이름|
|---|---|---|
|**Reactor Netty**|io.projectreactor.netty|reactor-netty|
|**Undertow**|io.undertow|undertow-core|
|**Tomcat**|org.apache.tomcat.embed|tomcat-embed-core|
|**Jetty**|org.eclipse.jetty|jetty-server, jetty-servlet|

---

### HttpHandler 어댑터 사용 예제

#### Reactor Netty
```
HttpHandler handler = ...;
ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(handler);
HttpServer.create().host(host).port(port).handle(adapter).bindNow();
```

#### Undertow
```
HttpHandler handler = ...;
UndertowHttpHandlerAdapter adapter = new UndertowHttpHandlerAdapter(handler);
Undertow server = Undertow.builder()
        .addHttpListener(port, host)
        .setHandler(adapter)
        .build();
server.start();

```
#### Tomcat
```
HttpHandler handler = ...;
Servlet servlet = new TomcatHttpHandlerAdapter(handler);

Tomcat server = new Tomcat();
File base = new File(System.getProperty("java.io.tmpdir"));
Context rootContext = server.addContext("", base.getAbsolutePath());
Tomcat.addServlet(rootContext, "main", servlet);
rootContext.addServletMappingDecoded("/", "main");
server.setHost(host);
server.setPort(port);
server.start();
```

#### Jetty
```
HttpHandler handler = ...;
Servlet servlet = new JettyHttpHandlerAdapter(handler);

Server server = new Server();
ServletContextHandler contextHandler = new ServletContextHandler(server, "");
contextHandler.addServlet(new ServletHolder(servlet), "/");
contextHandler.start();

ServerConnector connector = new ServerConnector(server);
connector.setHost(host);
connector.setPort(port);
server.addConnector(connector);
server.start();

```
#### Servlet 컨테이너

WAR 파일로 Servlet 컨테이너에 배포하려면 **AbstractReactiveWebInitializer**를 확장하고 WAR 파일에 포함합니다. 이 클래스는 HttpHandler를 ServletHttpHandlerAdapter로 감싸고 이를 Servlet으로 등록합니다.