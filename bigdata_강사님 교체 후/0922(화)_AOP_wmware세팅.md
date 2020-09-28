## 9/21(화)

#### AOP

-----------

> aop

- UserDAO.java

```java
package aop;

import java.util.ArrayList;

public class UserDAO implements DAO {
	public ArrayList<UserVO> getUserList(UserVO vo) {
		System.out.println("########################");
		System.out.println("##### getUserList() 호출... #####");
		System.out.println("########################");
		return null;
	}
	public UserVO getUser(UserVO vo) {
		System.out.println("########################");
		System.out.println("##### getUser() 호출... #####");
		System.out.println("########################");
		return null;
	}
	public void addUser(UserVO vo) {
		System.out.println("########################");
		System.out.println("##### addUser() 호출... #####");
		System.out.println("########################");
	}	
	public void deleteUser(UserVO vo) {
		System.out.println("########################");
		System.out.println("##### deleteUser() 호출... #####");
		System.out.println("########################");
	}
}

```



- LogAdvice.java

```java
package aop;
//모든 클래스에 적용한 공통모듈 - 비지니스 로직 외적인 부분이지만 늘 많은 메소드의 호출 전 후로 사용되는 기능

public class LogAdvice {
	public void log() {
		System.out.println("===========로그기록==========");
		
	}
}
```



- get으로 시작되는 메소드가 실행될 때만 로그기록이 남음
- aopbean.xml

![image-20200922091656849](image\image-20200922091656849.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">

	<!-- 나의 비지니스 로직(핵심모듈) -->
	<bean id="dao" class="aop.UserDAO"/>
	
	<!-- 공통관심사항이 정의된 빈 -->
	<bean id="logAdvice" class="aop.LogAdvice"/>
	
	<!-- aop적용 -->
	<aop:config>
		<!-- aop패키지에 UserDAO 클래스의 get으로 시작하는 메소드가 호출되기 전
			로그를 기록할 수 있도록 조건을 정의
			 -->
		<aop:pointcut expression="execution(* aop.*.get*(..))" id="mygetPointcut"/>
		<!-- 공통관심사를 어느 시점에 적용할 것인지 
		즉, pointcut으로 등록된 조건을 만족하는 모든 메소드들이 실행되기 전에 공통관심모듈이
		실행되도록 설정 -->
		<!-- ref속성은 공통관심모듈 즉 advice빈을 의미 -->
		<aop:aspect id="aop01" ref="logAdvice">
			<aop:before method="log" pointcut-ref="mygetPointcut"/>
		</aop:aspect>
	</aop:config>
</beans>

```

- AOPClient.java

```java
package aop;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPClient {
	public static void main(String[] args) {
		ApplicationContext factory
		  = new  ClassPathXmlApplicationContext("config/aopbean.xml");
		DAO dao  = (DAO)factory.getBean("dao");
		UserVO vo =  new UserVO();
		dao.getUser(vo);
		dao.getUserList(vo);
		dao.addUser(vo);
		dao.deleteUser(vo);
	}
}
```



> 예제

- [aop.exam 패키지 작성]
  - aop패키지의 모든 클래스를 copy(LogAdvice는 제외)
- [공통관심모듈]
  - CalcAdvice를 작성
  - 랜덤수를 발생시켜서 멤버변수에 세팅하는 메소드(setNum)
  - 1부터 랜덤수까지의 합을 계산하는 calc메소드 정의
- [설정파일작성] - aopconfig02
  - aop.exam패키지의 모든 클래스의 모든 메소드 적용
  - 공통관심모듈, 핵심모듈, 설정 정의
  - setNum은 메소드 호출 전에 실행
  - calc는 메소드 호출 후에 실행 



- CalcAdvice.java

```java
package aop.exam;

import java.util.Random;

public class CalcAdvice {
	int num; //랜덤수를 저장하기 위한 변수
	public void setNum() {
		Random r = new Random();
		num = r.nextInt(100)+1;
		System.out.println("랜덤수===>"+num);
	}
	
	public void calc() {
		int sum=0;
		for (int i = 1; i<=num; i++) {
			sum=sum+i;
		}
		System.out.println("합계===>"+sum);
	}
}
```



- aopbean2.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">

	<!-- 나의 비지니스 로직(핵심모듈) -->
	<bean id="dao" class="aop.exam.UserDAO"/>
	
	<!-- 공통관심사항이 정의된 빈 -->
	<bean id="calcAdvice" class="aop.exam.CalcAdvice"/>
	
	<!-- aop적용 -->
	<aop:config>
		<!-- aop패키지에 UserDAO 클래스의 get으로 시작하는 메소드가 호출되기 전
			로그를 기록할 수 있도록 조건을 정의
			 -->
		<aop:pointcut expression="execution(* aop.exam.*.*(..))" id="calcPointcut"/>
		<!-- 공통관심사를 어느 시점에 적용할 것인지 
		즉, pointcut으로 등록된 조건을 만족하는 모든 메소드들이 실행되기 전에 공통관심모듈이
		실행되도록 설정 -->
		<!-- ref속성은 공통관심모듈 즉 advice빈을 의미 -->
		<aop:aspect id="aop02" ref="calcAdvice">
			<aop:before method="setNum" pointcut-ref="calcPointcut"/>
			<aop:after method="calc" pointcut-ref="calcPointcut"/>
		</aop:aspect>
	</aop:config>
</beans>

```



- AOPClient.java (실행)

```java
package aop.exam;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPClient {
	public static void main(String[] args) {
		ApplicationContext factory
		  = new  ClassPathXmlApplicationContext("config/aopbean2.xml");
		DAO dao  = (DAO)factory.getBean("dao");
		UserVO vo =  new UserVO();
		dao.getUser(vo);
		dao.getUserList(vo);
		dao.addUser(vo);
		dao.deleteUser(vo);
	}
}
```

![image-20200922111536347](image\image-20200922111536347.png)



> aop 패키지를 어노테이션으로 변경

- annoaop.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd">
	<context:component-scan base-package="aop"></context:component-scan>
	
	<!-- aop를 어노테이션으로 사용하는 경우 aop작업을 수행하는 proxy가 자동으로 감지해서 
		aop등록된 것을 확인하고 동작할 수 있도록 등록 -->
	<aop:aspectj-autoproxy/>
</beans>
```



- LogAdvice.java

```java
package aop.anno;


import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Service;
//모든 클래스에 적용한 공통모듈 - 비지니스 로직 외적인 부분이지만 늘 많은 메소드의 호출 전 후로 사용되는 기능
@Service
//aop를 적용하는 advice임을 나타내주어야 한다.
@Aspect
public class LogAdvice {
	//어노테이션으로 포인트컷을 정의하는 경우 메소드를 반드시 한 개 정의해서 메소드 위에 패턴을 정의
	// - 메소드 위에 @Pointcut이라는 어노테이션을 선언
	// - 아래와 같이 pointcut을 정의하면 메소드 명을 pointcut으로 인식
	@Pointcut("execution(* aop.anno.*.get*(..))")
	public void mylogPointcut() {}
	
	//위에서 정의한 포인트컷을 적용(메소드 실행 후)
	//()를 포함한 메소드명이 포인트컷명 
	@After("mylogPointcut()")
	public void log() {
		System.out.println("===========로그기록==========");
		
	}
}

```



- AOPClient.java

```java
package aop.anno;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPClient {
	public static void main(String[] args) {
		ApplicationContext factory
		  = new  ClassPathXmlApplicationContext("config/annoaop.xml");
		DAO dao  = (DAO)factory.getBean("dao");
		UserVO vo =  new UserVO();
		dao.getUser(vo);
		dao.getUserList(vo);
		dao.addUser(vo);
		dao.deleteUser(vo);
	}
}
```



> 쇼핑몰에 트랜잭션 추가
>
> board insert 트랜잭션

- spring-config 의 namespace에 다음 내용 추가

![image-20200922131902100](image\image-20200922131902100.png)

- spring-config.xml 에 내용 추가 

```xml
<!-- ======================aop를 이용한 선언적 트랜잭션의 처리========================= -->
	<!-- 1. 트랜잭션 처리를 위해 제공하는 spring클래스를 등록 -->
	<beans:bean id="transactionManager" 															class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<beans:property name="dataSource" ref="ds"/>
	</beans:bean>
	
	<!-- 2. 1번에서 등록한 트랜잭션 처리 클래스를 advice로 등록 -->
	<tx:advice id="transactionAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="insert" rollback-for="Exception"/>
		</tx:attributes>
	</tx:advice>
	
	<!-- 3. aop처리 -->
	<aop:config>
		<aop:pointcut expression="execution(* 														kr.encore.bigdataShop.board.BoardServiceImpl.insert(..))" 
			id="txpointcut"/>
		<aop:advisor advice-ref="transactionAdvice" pointcut-ref="txpointcut"/>
	</aop:config>
```



- pom.xml 추가

```xml
<!-- AOP기능 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>4.2.4.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>4.2.4.RELEASE</version>
    </dependency>
```



- BoardServiceImpl.java
  - insert 오류 시 rollback

```java
//게시글 등록
	@Override
	public int insert(BoardDTO board, ArrayList<String> filelist) {
		//1. 게시글을 board테이블에 insert
		//2. 게시글에 추가된 첨부파일들을 board_file 테이블에 저장
		//1번과 2번
		int result = 0; //전체 결과
		int boardResult = 0; //board테이블에 insert결과
		int boardfileResult = 0 ;//board_file 테이블에 insert한 결과
		boardResult = dao.insert(board);
		if(filelist.size()!=0) {
			boardfileResult = dao.fileInsert(filelist);
			if(boardResult>=1 & boardfileResult>=1) {
				result = 1;
			}
		}else {
			if(boardResult>=1) {
				result = 1;
			}
		}
		
		return result;
	}
```



#### 빅데이터 플랫폼 구축

--------

> 세팅

  **1. vmware 설치**  

  **2. vmware 가상머신 설치하기 - CentOS 7버전을 설치한다.**

  **3. 가상머신 복제하기**

​     **- 가상머신이 네 대 있다 가정하고 네 개의 가상머신을 만들어준다.**

​     **: ip확인**

  **4. 하둡 서버를 구축하기 위한 클러스터링 설정하기**

​     **- 방화벽해제**

​     **- 네트워크 설정**

​     **- DNS설정**

  **5. 각종 프로그램 설치**

​     **- SSH 프로토콜 설정**

​     **- hadoop을 테스트하기 위해서는 자바가 반드시 필요하므로**

​     **- java, hadoop을 설치하고 설정을 한 후 테스트한다.**

  **6. hadoop의 EchoSystem을 살펴보고 EchoSystem을 설치하여 테스트한다.** 



**1.wmware 설치**

> wmware 다운로드

- https://www.vmware.com/kr/products/workstation-player/workstation-player-evaluation.html

![image-20200922082218119](image\image-20200922082218119.png)



- https://wiki.centos.org/Download
- 미러사이트 접속

![image-20200922082459755](image\image-20200922082459755.png)

- CentOS 7 다운로드

![image-20200922082553962](image\image-20200922082553962.png)



> wmware 설치

![image-20200922143625512](image\image-20200922143625512.png)

![image-20200922143646659](image\image-20200922143646659.png)

![image-20200922143713096](image\image-20200922143713096.png)

![image-20200922143849146](image\image-20200922143849146.png)

![image-20200922144000330](image\image-20200922144000330.png)

=> install -> finish

![image-20200922144133993](image\image-20200922144133993.png)



- 실행

![image-20200922144808767](image\image-20200922144808767.png)

![image-20200922144829035](image\image-20200922144829035.png)

![image-20200922144922415](image\image-20200922144922415.png)

> 머신 생성하기 

- create a new virtual machine

![image-20200922145010935](image\image-20200922145010935.png)

![image-20200922145053332](image\image-20200922145053332.png)

![image-20200922145147990](image\image-20200922145147990.png)

![image-20200922145400428](image\image-20200922145400428.png)

![image-20200922145624619](image\image-20200922145624619.png)

![image-20200922145733180](image\image-20200922145733180.png)

![image-20200922145756752](image\image-20200922145756752.png)

![image-20200922145828977](image\image-20200922145828977.png)

- C:\Users\whtpw\Documents\Virtual Machines\hadoop01 에 생성

![image-20200922151253988](image\image-20200922151253988.png)



> vmware 전원 관리

- 전원켜기

![image-20200922151319023](image\image-20200922151319023.png)

![image-20200922151338078](image\image-20200922151338078.png)

- ctrl + alt : 마우스 제어 

![image-20200922151619739](image\image-20200922151619739.png)

- suspend : 현재 작업까지 상태 저장
- power off : 끄기

![image-20200922152012675](image\image-20200922152012675.png)



- 네트워크를 실행할 수 있는 실행파일 설치
  - C:\Program Files (x86)\VMware\VMware Player 폴더에 파일 넣기

![image-20200922155425281](image\image-20200922155425281.png)

- 실행 

![image-20200922155503508](image\image-20200922155503508.png)



> vmware 네트워크

- change settings

![image-20200922162234193](image\image-20200922162234193.png)

- ip 변경 가능

![image-20200922162347983](image\image-20200922162347983.png)

![image-20200922162754969](image\image-20200922162754969.png)

![image-20200922162617361](image\image-20200922162617361.png)



- power off 후 settings

![image-20200922162721967](image\image-20200922162721967.png)

![image-20200922163000078](image\image-20200922163000078.png)

- 파일 선택 후 ok 버튼

![image-20200922162945214](image\image-20200922162945214.png)













