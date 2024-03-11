![[Pasted image 20240311171247.png]]
Bootstrap, Extension, Application - 세 가지 컴포넌트들에 의해 클래스들이 로드됨.
이 세가지 클래스 로더들은 모두 상속관계로 정의되어 있으며 위임 방식으로 작업을 진행함.

#### Bootstrap ClassLoader
가장 최상위 클래스 로더로써 가장 최우선으로 로드됨.
jre의 lib폴더에 rt.jar 파일을 찾아서 기본 자바 API 라이브러리를 메모리에 로드함.
rt.jar은 런터임 시 java.lang.String 같은 Java에서 제공되는 모든 라이브러리가 포함되어 있음.  (rt = RunTime의 약자)

#### Extension ClassLoader / Platform ClassLoader
Java8에서는 Extension ClassLoader라고 불리며 Java9에서는 Platform ClassLoader라 불림.
$JAVA_HOME/jre/lib/ext 경로의 외부 라이브러리를 로딩함

#### Application ClassLoader / System ClassLoader
Java8에서는 Application ClassLoader라고 불리며 Java9에서는 System ClassLoader라 불림.
애플리케이션을 실행할 때 classpath의 class들을 메모리에 로딩하는 역할을 함.

### 위임 방식이란?
만약 Application Class loader가 라는 Internal이라는 클래스를 로딩할 때 그 로딩 요청은 부모 클래스 로더들로 거슬러 올라가서 부트 스트랩 로더에 다다른 후 그 밑으로 로딩 요청을 수행함
![[Pasted image 20240311172948.png]]