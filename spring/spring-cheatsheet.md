# Boot

>Points to remember:

- opinionated runtime: 
- Dependency Management
- auto-configuration but customizable
- packaging

## How to setup a SpringBoot project?
```java
1. pom.xml 						- setup dependencies
2. HelloController.java 		- Spring MVC controller
3. application.properties 
spring.mvc.view.prefix=/WEB-INF/views
spring.mvc.view.suffix=.jsp
					 			- InternalResourceViewResolver config
4. /WEB-INF/views/hello.jsp     - Spring MVC view
5. Application.java 
@SpringBootApplication(Configuration, ComponentScan, EnableAutoConfiguration)
								- applcation launcher
```

## How to run a SpringBoot project?
```java
mvn package
helloApp-0.0.1-SNAPSHOT.jar
java -jar helloApp-0.0.1-SNAPSHOT.jar/war
---runing in localhost:8080
```
## Opinionated runtime: 
based on the classpath jars, Spring Boot sets up default configuration envrionment
```java
JPA-EntityMangerFactory etc, "spring-boot-starter-web"-embeddedTomcat(web app run in jar file), SpringMVC, Jackson etc.
```
## Dependency Management: enable auto configuration
- parent (spring-boot-starter-parent) : " spring-boot-starter" the version
- dependencies :  scanned, auto-configed
"spring-boot-starter-web"-embeddedTomcat, SpringMVC, Jackson etc.- DispatcherServlet and ContextLoaderListener
"spring-boot-starter-test" - spring-test, junit, mokito 
"spring-boot-starter-jpa/jdbc/batch" - JdbcTemplate
"hsqldb" - EmbeddedDataSource 
- plugins
```java
for packging
"spring-boot-maven-plugin"
```
## container vs embedded container
```java
public class Application extends SpringBootServletInitializer{}

java -jar app.war.original - war embeded-container and outside-container versions match
java -jar app.war - hybrid war
```
## `application.properties/yml`
define any properties here, i.e., 
```properties
database.host=localhost 
database.user=admin
#support any loging famework, SLF4J better
logging.level.org.springframework=DEGUG
logging.levle.com.acme.your.code=INFO
#override the default log-to-console
logging.file=rewards.log
#a pooled datasource is created by default, explicitly config datasource override the default one
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
#set up web config
server.port=900
server.address=192.168.1.20
server.session-timeout=1800
server.context-path=/rewards
server.servlet-path=/admin
```

## static resources, templates, error
/static, /public, /resources, /META-INF/resources; 

/templates; 

/error


# Test
first each unit tested, then integration test (multi units work together)
## Test in Boot
```java
@RunWith(SpringJUnit4ClassRunner.class)
//"TranserApplication class" is the SpringBootApplication class 
@SpringApplicationConfiguration(class=TranserApplication.class)
//web app test, default location of resource is src/main/webapp
@WebAppConfiguration
public final class TranserServiceTest{
	@Autowired
	private TranserService transerService;
	@Test
	public void successfulTransfer(){
		TransferConfirmation conf=transerService.transfer;
	}
}
```
## TDD
test driven development: requirements --in a form of --> tests

## How do you write unit test?
- use **stubs/mocks** as alternatives of non-tested dependencies.
- test a unit bussiness logic, success or fail

### Use a stub
```java
public class AuthenticatorImpl{
	//accRepo needs a stub
	private AccountRepository accRepo;
	public AuthenticatorImpl(AccountRepository accRepo){
		this.accRepo = accRepo;
	}
	public boolean authenticate(String username, String password){
		Account acc = accRepo.getAccount(username);
		//test a unit bussiness logic, success or fail
		return acc.getPassword().equals(password);
	}
}

public class StubAccountRepository implements AccountRepository{
	public Account getAccount(String username){
		return "lisa".equals(user)? new Account("lisa","secret"): null;
	}
}

public final class AuthenticationImplTests{
	private AuthenticationImpl authenticator;

	@Before public void setUp(){
		authenticator = new AuthenticatorImpl(new StubAccountRepository());
	}

	@Test public void successfulAuthentication(){
		assertTrue(authenticator.authenticate("lisa","secret"));
	}

	@Test public void invalidPassword(){
		assertFalse(authenticator.authenticate("lisa","invalid"));
	}
}
```
### Use a mock
```java
//generate a mock object using mock lib; 
mock(AccountRepository.class)

//record the mock with expectation that when methodA is used, then resultA is returned;
when(accRepo.getAccount("lisa"),thenReturn(new Account("lisa", "secret")

//run the test, assertTrue/assertFalse/assertEquals
assertTrue(authenticator.authencate("lisa","secret"))

//verify the mock
verify(accRepo)
```

## How do you write integration test?
```java
1. config spring-test dependency
2. no @Before method
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=SystemTestConfig.class)
@TestPropertySource(properties={"username=foo","password=bar"} location="classpath:/transfer-test.properties")
@ActiveProfiles({"jdbc","dev"})//@Profile in normal config class
public final class TranserServiceTest{
	@Autowired
	private TranserService transerService;
	@Test
	public void successfulTransfer(){
		TransferConfirmation conf=transerService.transfer;
	}
}
```
## How do you test with databases
in-memory databases
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=SystemTestConfig.class)
@Sql({"/testfiles/schema.sql","/testfiles/general-data.sql"})
public final class TranserServiceTest{
	@Test@Sql("/testfiles/test-data.sql")
	//sql runs before test executed
	public void successfulTransfer(){}

	@Test@Sql(scripts="/testfiles/test-data.sql", executionPhase=Sql.executionPhase.AFTER_TEST_METHOD, config=@SqlConfig(errorMode=ErrorMode.FAIL_ON_ERROR))
	//sql runs AFTER test executed
	public void successfulTransfer(){}
}
```

# Web
## Request processing lifecycle
request URL is handled by DispatcherServlet which dispatch the request to a controller, the controller returns a logic view name back to DispatcherServlet, dispatcherServlet consults viewResolver for the right view. Finally, dispatcherServlet render model data on the view.
## artifacts (dispatcherservlet, controllers, views)
- DispatcherServlet

is definded by WebApplicationInitializer aka "web.xml" in xml config
```java
public class MyWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{
	//get root config class which is loaded by context loader listener 
	//get servlet config class in which viewResolver is configed
	//dispatcherservlet mapping
}
```
- Controllers

(HttpServletRequest, HttpSession, Principal, Model, @RequestParam("id") long id, @PathVariable("id") long , @RequestHeader("user-agent") String agent)

"http://localhost:8080/mvc-1/rewardsadmin/" application server, webapp, servlet mapping, request mapping
```java
@Controller
@RequestMapping("/listAccounts")//ignore .html
public class AccountController{
	//"/listAccounts.html?id=123"
	public String list(Model model, @RequestParam("id") long id){}
	//"/listAccounts/{id}"
	public String list(Model model, @PathVariable("id") long id){}
}
```
- Views

InternalResourceViewResolver is created for ViewResolver bean registered in `MvcConfig.class`

by default view files are in "/WEB-INF/views/"

## how do you implement a web mvc module
1. deploy a dispatcher servlet, extends AbstractAnnotationConfigDispatcherServletInitializer, in which RootConfig, MvcConfig and DispatcherServlet mapping is initialized.
2. implement a controller
3. register the controller with the dispatcher servlet
4. implement the views
5. register a view resolver
6. deploy and test

```java
//dispather servlet
public class MyWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer{
	//get root config class which is loaded by context loader listener 
	getRootConfigClasses(){
		return new Class<?>[]{RootConfig.class};
	}
	//get servlet config class in which viewResolver is configed
	getServletConfigClasses(){
		return new Class<?>[]{MvcConfig.class};
	}
	//dispatcherservlet mapping
	getServletMapping(){
		return new String[]{"/rewardsadmin/*"};
	}
}

@configuration
@ComponentScan("package.mvc.bean.name")
public class MvcConfig{//additional beans //override view resolver
}

<html>
<body>
Amount = ${reward.amount}<br/>
Date = ${reward.date}<br/>
Account = ${reward.account}<br/>
Description = ${reward.description}<br/>
</body>
</html>
```
# REST Web Services

# Microservices

#JDBC
#JPA
#Data
#Security
#AOP
#IoC
```java
```