## 9/14(월)

#### Maven기반 Spring 프로젝트 구성

--------

> 설정

- https://spring.io/ 사이트 접속 -> https://spring.io/projects
- https://spring.io/tools -> https://github.com/spring-projects/toolsuite-distribution/wiki/Spring-Tool-Suite-3 (spring tool suit 다운로드)

![image-20200914105108987](image/image-20200914105108987.png)



- 압축 풀기

![image-20200914110610346](image/image-20200914110610346.png)



- 관리자 권한으로 실행하기

![image-20200914110927824](image/image-20200914110927824.png)



- 워크스페이스 설정(stswork)

![image-20200914111251600](image/image-20200914111251600.png)



- 기존에 등록되어 있는 서버 삭제(두 부분 모두 삭제)

![image-20200914111939029](image/image-20200914111939029.png)

![image-20200914112004152](image\image-20200914112004152.png)



- window - preferences 열기 

![image-20200914113133781](image/image-20200914113133781.png)



- Server - Runtime Environment 에 들어가 서버 삭제

![image-20200914112134652](image/image-20200914112134652.png)



- https://blog.naver.com/heaves1 에 접속 (강사님 블로그)
- 다운로드

![image-20200914112418865](image/image-20200914112418865.png)



- 서버 등록

![image-20200914112611516](image/image-20200914112611516.png)



- new - other - spring - spring Legacy Project 

![image-20200914113652312](image/image-20200914113652312.png)



![image-20200914113738916](image/image-20200914113738916.png)

![image-20200914114036174](image/image-20200914114036174.png)

![image-20200914114354332](image/image-20200914114354332.png)



- HomeController.java 실행

![image-20200914114957665](image/image-20200914114957665.png)



- 에러 발생

![image-20200914115048522](image/image-20200914115048522.png)



- 서버 더블클릭

  <Ports 설정>

![image-20200914115133097](image/image-20200914115133097.png)

​	<Timeouts 설정>

![image-20200914115256367](C:\Users\whtpw\AppData\Roaming\Typora\typora-user-images\image-20200914115256367.png)



- 재 실행 시 에러

![image-20200914115348623](image/image-20200914115348623.png)



- firstPro까지만 남기고 재 실행

![image-20200914115437977](image/image-20200914115437977.png)



- 설정 변경(window-preferences-General-Web Browser)에서 external web browser로 변경

![image-20200914115614753](image/image-20200914115614753.png)



- firstPro

  - firstPro 
  - src/main/java : src 부분
  - Maven Dependencies : 라이브러리 자동 주입
  - pom.xml : 라이브러리 명세 
    - dependency : 하나의 라이브러리 명세 
    - groupId : 프로젝트 그룹 명
    - artifactId : 프로젝트 명 
    - 라이브러리 default 위치 : C:\Users\whtpw\.m2\repository\javax\servlet\servlet-api\2.5 (Maven Dependencies에 들어온 것)

  ![image-20200914122106169](image/image-20200914122106169.png)



- 설정

  - firstPro 오른쪽 마우스 클릭 - Properties
  - utf-8

  ![image-20200914121526007](image/image-20200914121526007.png)

  - Maven - Project Facets (자바 1.8로 변경)

  ![image-20200914121715731](image/image-20200914121715731.png)

  - pom.xml (버전 맞게 변경)

  ![image-20200914121902016](image/image-20200914121902016.png)

  - C:\Users\whtpw\.m2\repository\org\springframework\spring-core 에 확인하면 처음에 실행될 때 디폴트 버전과 수정된 버전 둘 다 있음을 확인 할 수 있다. 

  ![image-20200914123330943](image/image-20200914123330943.png)

  - C:\Users\whtpw\.m2 - STS.exe 종료 후 - repository삭제 - 재 실행 (관리자 권한)

  ![image-20200914123535700](image/image-20200914123535700.png)

  - 에러 확인

  ![image-20200914124012552](image/image-20200914124012552.png)

  - (에러 고치기)라이브러리 dependency 지우고 저장한 뒤 다시 넣어서 저장

  ![image-20200914124247886](image/image-20200914124247886.png)

  

  

- https://mvnrepository.com/ 즐겨찾기 추가

![image-20200914124734406](image/image-20200914124734406.png)



- DB 연동하는 방법 

- spring jdbc 검색 (나중에 사용할 때 추가하면 됨)

![image-20200914124844310](image/image-20200914124844310.png)

![image-20200914124923799](image/image-20200914124923799.png)



- https://blog.naver.com/heaves1 -> DB -> 오라클

![image-20200914125059221](image/image-20200914125059221.png)



> 설정파일 한 파일에 옮기기 

- config 폴더 생성(servlet-context.xml 파일 config로 옮기기)

![image-20200914151535334](image/image-20200914151535334.png)



- servlet-content.xml 파일명 변경

![image-20200914151627375](image/image-20200914151627375.png)



- web.xml 파일 수정(경로 수정) - 서버 리스타트

![image-20200914152130761](image/image-20200914152130761.png)



- 아까 블로그에서 다운 받은 파일 view - emp 폴더 WEB-INF에 넣기

![image-20200914153738818](image/image-20200914153738818.png)

![image-20200914153758312](image/image-20200914153758312.png)



- emp폴더를 src에 넣기

![image-20200914153858078](image/image-20200914153858078.png)

![image-20200914153924887](C:\Users\whtpw\AppData\Roaming\Typora\typora-user-images\image-20200914153924887.png)



> 테이블 생성 (SQL)

- Run SQL Command Line 열기

```sql
conn encore/encore

create table emp(
  id varchar2(20) primary key,
  pass varchar2(20),
  name varchar2(20),
  addr varchar2(20),
  hiredate date,
  grade varchar2(20),
  point number,
  deptno varchar2(20));
```

![image-20200914155121359](image/image-20200914155121359.png)

![image-20200914155055283](image/image-20200914155055283.png)



- 위에서 찾은 dependency pom.xml 에 추가

![image-20200914160640886](image/image-20200914160640886.png)

![image-20200914160809433](image/image-20200914160809433.png)



- 테이블 명 모두 emp로 변경

![image-20200914160912604](image/image-20200914160912604.png)

![image-20200914161059750](image/image-20200914161059750.png)



- 강사님 블로그 - spring 고급 - maven project 필요파일 

  (https://blog.naver.com/heaves1/222088931857) - 두 파일 다운

![image-20200914161207243](image/image-20200914161207243.png)



- webapp에 META-INF 압축 풀어서 넣기

![image-20200914161437899](image/image-20200914161437899.png)



> db 연결

- context.xml 파일 수정(127.0.0.1 / encore)

![image-20200914161754694](image/image-20200914161754694.png)



- spring-config.xml 파일에서 view/ 삭제 

![image-20200914161852286](image/image-20200914161852286.png)



> mybatis 설정

- WEB-INF에 lib 폴더(임시) 생성

  - C:\oraclexe\app\oracle\product\11.2.0\server\jdbc\lib에 있는 ojdbc6.jar 넣기

  ![image-20200914162933819](image/image-20200914162933819.png)

  

- spring-config.xml 추가

  ![image-20200914164234752](image/image-20200914164234752.png)

  - < context:component-scan > : emp의 하위 파일들까지 사용 가능 / main은 추후에 추가할 예정

  - < beans:bean >

    - beans:property 의 value는 context.xml의 name으로 사용

    ![image-20200914163544777](image/image-20200914163544777.png)



- WEB-INF에 main폴더 생성

![image-20200914164954086](image/image-20200914164954086.png)



- main 폴더 안에 강사님 블로그에서 다운 받았던 index.jsp 넣기



> 컨트롤러 생성 

- src에 main 패키지 생성

![image-20200914165156566](image/image-20200914165156566.png)



- main 패키지 안에 IndexController.java 생성

```java
package main;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class IndexController {
	//첫 번째 페이지를 response하기 위해서 사용하는 메소드
	//String을 리턴하지만 내부에서 ModelAndView로 만들어서 리턴(DispatcherServlet)된다.
	@RequestMapping("/index.do")
	public String main() {
		System.out.println("sts 설정완료");
		return "main/index";
	}		
}
```







































