## 9/17(목)

#### Maven기반 Spring 프로젝트 구성

--------

> mybatis 연동

1. pom.xml에 라이브러리 추가
2. spring 설정파일에 등록
3. mybatis설정 파일을 작성
4. mapper에 sql정의
5. DAO를 통해 db엑세스



**1. pom.xml에 라이브러리 추가**

- https://mvnrepository.com/search?q=mybatis (spring버전과 호환이 될 수 있도록 맞춰줘야함)
  - mybatis : 3.2.8 
  - mybatis spring : 1.2.2

![image-20200917092501986](image/image-20200917092501986.png)

```xml
<!-- mybatis -->
	<dependency>
	    <groupId>org.mybatis</groupId>
	    <artifactId>mybatis</artifactId>
	    <version>3.2.8</version>
	</dependency>	

<!-- mybatis-spring -->
	<dependency>
	    <groupId>org.mybatis</groupId>
	    <artifactId>mybatis-spring</artifactId>
	    <version>1.2.2</version>
	</dependency>
```



**2. spring-config.xml 설정**

```xml
<!-- ================== mybatis 설정 ================== -->
<!-- spring과 mybatis를 연동하기 위해 필요한 객체 -->
<beans:bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<beans:property name="dataSource" ref="ds"/>
	<beans:property name="configLocation" value="/WEB-INF/config/mybatis-config.xml"/>
</beans:bean>
	
<!-- mybatis의 핵심 클래스를 등록(spring JDBC의 JdbcTemplate과 동일한 작업 )
	DB테이블을 CLRUD할 수 있는 기능을 제공-->
<beans:bean id="sqlSession"	class="org.mybatis.spring.SqlSessionTemplate">
	<beans:constructor-arg ref="sqlSessionFactory"/>
</beans:bean>
```



**3.mybatis 설정**

- mybatis-config.xml
  - configuration으로 시작하면 메인 설정파일 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
	<typeAliases>
		<typeAlias type="emp.dto.EmpDTO" alias="emp"/>
	</typeAliases>
	<mappers>
		<mapper resource="mapper/emp.xml"/>
	</mappers>
</configuration>
```



**4. mapper에 sql정의 **

- emp.xml
  - resultType : alias 적어놓으면 알아서 ArrayList로 담아온다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="kr.encore.emp">
	<select id="list" resultType="emp">
		select * from emp
	</select>
</mapper>
```



**5. DAO를 통해 db엑세스**

- MybatisEmpDAOImpl.java

  - @Repository : 퍼시스턴스 레이어, DB나 파일같은 외부 I/O 작업을 처리함

  ![image-20200917114453708](C:\Users\whtpw\AppData\Roaming\Typora\typora-user-images\image-20200917114453708.png)

  - return namespace.id

![image-20200917101213993](image/image-20200917101213993.png)

```java
@Override
	public List<EmpDTO> getMemberList() {
		// TODO Auto-generated method stub
		return sqlSession.selectList("kr.encore.emp.list");
	}
```



- 서버 리스타트 시 에러 발생
  - mybatis, empdao가 충돌 

```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'deleteController': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: emp.dao.MyEmpDAO emp.controller.DeleteController.dao; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type [emp.dao.MyEmpDAO] is defined: expected single matching bean but found 2: mybatisemp,empdao
```



- (해결) @Qualifier("mybatisemp") 추가 (Repository 에 추가한 이름) : 2개가 있음을 명시

![image-20200917103137402](image/image-20200917103137402.png)



> 쇼핑몰 연습

- 주의

![image-20200917141103522](image/image-20200917141103522.png)



- 데이터 Controller로 넘어오지 않은 문제
  - enctype="multipart/form-data"를 지워야함 : 

![image-20200917141801030](image/image-20200917141801030.png)





> JQuery 자동완성

- http://oss.opensagres.fr/tern.repository/1.2.0/ - select all

![image-20200917154244399](image/image-20200917154244399.png)



- 프로젝트 오른쪽버튼 -configure-convert to tern

![image-20200917170933788](image/image-20200917170933788.png)









