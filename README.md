## Spring Security
>參考網址：https://blog.csdn.net/yjp19871013/article/details/53894260
>
>Spring Security是Spring實現的安全框架，可以對請求和方法進行安全保護，Spring Security根本上是一套Filter鏈，當配置使用Spring Security時，Spring會向項目中添加Filter，從而對請求進行攔截，並進行必要的安全校驗
>
>* Spring Security是⼀個功能強大、可高度定制的身份驗證和訪問控制框架。它是保護基於Spring的應用程序的事實標準
>* Spring Security是⼀個面向Java應用程序框架。與所有Spring項目⼀樣，Spring Security的真正威力在於它可以輕鬆地擴展以滿足定制需求
>
>在Spring Security中，權限管理主要包括兩個方面：認證和授權。簡單來說，認證就是用戶的登錄認證；授權就是登錄成功之後，用戶可以訪問資源

### POM File(Project Object Model file依賴項目配置清單)
>為了讓Spring支持Spring Security，需要添加必要的依賴，這是因為Spring Security不屬於Spring Framework，是一個獨立的項目

**pox.xml完整內容**
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.luv2code</groupId>
	<artifactId>spring-security-demo</artifactId>
	<version>1.0</version>
	<packaging>war</packaging>

	<name>spring-security-demo</name>

	<properties>
		<springframework.version>5.0.2.RELEASE</springframework.version>
		<springsecurity.version>5.0.0.RELEASE</springsecurity.version>

		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
	</properties>

	<dependencies>

		<!-- Spring MVC support -->
		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.4</version>
        </dependency>
		
		<!-- Spring Security -->
		<!-- spring-security-web and spring-security-config -->
		
		<dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
            <version>5.4.3</version>
        </dependency>
		
		<dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
            <version>5.4.3</version>
        </dependency>
        
        <!-- Add Spring Security Taglibs support -->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-taglibs</artifactId>
            <version>5.4.3</version>
        </dependency>
        
        <!-- Add MySQL and C3P0 support -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.45</version>
		</dependency>

		<dependency>
			<groupId>com.mchange</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.5.2</version>
		</dependency>
        
		<!-- Servlet, JSP and JSTL support -->
       	<dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.0</version>
            <scope>provided</scope>
        </dependency>
        
            <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>5.3.4</version>
        </dependency>

		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>

		<!-- to compensate for java 9+ not including jaxb -->
		<dependency>
		    <groupId>javax.xml.bind</groupId>
		    <artifactId>jaxb-api</artifactId>
		    <version>2.3.0</version>
		</dependency>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>

	</dependencies>

	<!-- TO DO: Add support for Maven WAR Plugin -->

	<build>
		<finalName>spring-security-demo</finalName>
		
		<pluginManagement>
			<plugins>
				<plugin>
					<!-- Add Maven coordinates (GAV) for: maven-war-plugin -->
					<groupId>org.apache.maven.plugins</groupId>
  					<artifactId>maven-war-plugin</artifactId>
  					<version>3.3.0</version>
				</plugin>
			</plugins>
		</pluginManagement>
	
	</build>

</project>
```
## 資料庫連線相關配置屬性資料
**persistence-mysql.properties完整內容，包含資料庫連線驅動、使用者帳號、密碼、資料庫連線路徑以及連接池等相關屬性**
```
#
# JDBC connection properties
#
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_security_demo_plaintext?useSSL=false
jdbc.user=springstudent
jdbc.password=springstudent

#
# Connection pool properties
#
connection.pool.initialPoolSize=5
connection.pool.minPoolSize=5
connection.pool.maxPoolSize=20
connection.pool.maxIdleTime=3000
```

### 配置類(包含視圖解析、資料庫連線相關配置資料路徑以及屬性讀取)
>* src/main/resources為預設路徑，Maven會自動讀取路徑下的文件內容
>* `@PropertySource("classpath:persistence-mysql.properties")`給定文件名稱，實際會自動讀取內容並透過`Environment env`保存數據，再依需求配置`DataSource`
>* 創建Bean`securityDataSource`，透過env從文件讀取數據配置JDBC驅動、JDBC URL、用戶、密碼以及連接池等相關屬性
>* 添加輔助方法，將屬性字串型態轉換為整數型態以供配置方法使用
```
@Configuration
@EnableWebMvc
@ComponentScan(basePackages="com.luv2code.springsecurity.demo")
@PropertySource("classpath:persistence-mysql.properties")
public class DemoAppConfig {

	// set up variable to hold the properties
	@Autowired
	private Environment env;
	
	// set up a logger for diagnostics
	private Logger logger = Logger.getLogger(getClass().getName());
	
	// define a bean for ViewResolver
	@Bean
	public ViewResolver viewResolver() {
		
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		
		viewResolver.setPrefix("/WEB-INF/view/");
		viewResolver.setSuffix(".jsp");
		
		return viewResolver;
	}
	
	@Bean
	public DataSource securityDataSource() {
		
		// create connection pool
		ComboPooledDataSource securityDataSource
									= new ComboPooledDataSource();
		
		// set the jdbc driver class
		
		try {
			securityDataSource.setDriverClass(env.getProperty("jdbc.driver"));
		} catch (PropertyVetoException exc) {
			throw new RuntimeException(exc);
		}
		
		// log the connection props
		// for sanity's sake, log this info
		// just to make sure we are REALY reading data from properties file
		
		logger.info(">>> jdbc.url=" + env.getProperty("jdbc.url"));
		logger.info(">>> jdbc.user=" + env.getProperty("jdbc.user"));
		
		// set database connection props
		
		securityDataSource.setJdbcUrl(env.getProperty("jdbc.url"));
		securityDataSource.setUser(env.getProperty("jdbc.user"));
		securityDataSource.setPassword(env.getProperty("jdbc.password"));
		
		// set connection pool props
		
		securityDataSource.setInitialPoolSize(
				getIntProperty("connection.pool.initialPoolSize"));
		
		securityDataSource.setMinPoolSize(
				getIntProperty("connection.pool.minPoolSize"));
		
		securityDataSource.setMaxPoolSize(
				getIntProperty("connection.pool.maxPoolSize"));
		
		securityDataSource.setMaxIdleTime(
				getIntProperty("connection.pool.maxIdleTime"));
		
		return securityDataSource;
	}
	
	// need a helper method
	// read environment property and convert to int
	
	private int getIntProperty(String propName) {
		
		String propVal = env.getProperty(propName);
		
		// now convert to int
		int intPropVal = Integer.parseInt(propVal);
		
		return intPropVal;
	}
}
```
**效果等同於下方原先XML配置方式**
```
<!-- Define Spring MVC view resolver -->
<bean
	class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/view/" />
	<property name="suffix" value=".jsp" />
</bean>
```
**擴展AbstractSecurityWebApplicationInitializer抽象類別，只需擴展無須添加程式碼**
>該類派生自AbstractSecurityWebApplicationInitializer，這樣，當啟用Spring Security時，會加入安全校驗需要的Filter鏈。 最後，啟用Spring Security，並配置攔截規則和用戶數據源
```
public class SecurityWebApplicationInitializer 
						extends AbstractSecurityWebApplicationInitializer {

}
```
**擴展AbstractAnnotationConfigDispatcherServletInitializer抽象類別，並給定DemoAppConfig.class配置類別以及根目錄映射配置**
>當 DispatcherServlet 啟動的時候，它會創建 Spring 應用上下文，並加載配置文件或配置類中所聲明的 bean。在以上程序中的 getServletConfigClasses() 方法中，我們要求 DispatcherServlet 加載應用上下文時，使用定義在 WebConfig 配置類（使用 Java 配置)中的bean
```
public class MySpringMvcDispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		// TODO Auto-generated method stub
		return new Class[] {DemoAppConfig.class};
	}

	@Override
	protected String[] getServletMappings() {
		// TODO Auto-generated method stub
		return new String[] {"/"};
	}
}
```
**擴展WebSecurityConfigurerAdapter抽象類別，以配置Security安全配置**
>參考網址：https://blog.csdn.net/andy_zhang2007/article/details/90023901
>
>自訂Spring Security配置時，會建立一個類別繼承WebSecurityConfigurerAdapter並覆寫配置方法如configure()來修改/自訂Spring Security的預設安全配置，所以＠EnableWebSecurity會與此Spring Security配置類放在一起。
>
>例如以下範例，透過JDBC身分驗證，將用戶資料存放在資料庫中，Spring Security將會與數據庫中的用戶資料進行驗證，以及配置任何請求都必須經過身分驗證(登入)，與登入顯示頁面、驗證處理程序(authenticateTheUser)以及添加註銷支持(登出)
```
@Configuration
@EnableWebSecurity
public class DemoSecurityConfig extends WebSecurityConfigurerAdapter {

	// add a reference to our security data source
	@Autowired
	private DataSource securityDataSource;
	
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		
		// use jdbc authentication ... 
		auth.jdbcAuthentication().dataSource(securityDataSource);
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		
		// Restrict access based on the HttpServletRequest
		http.authorizeRequests()
				.antMatchers("/").hasRole("EMPLOYEE")
				.antMatchers("/leaders/**").hasRole("MANAGER")
				.antMatchers("/systems/**").hasRole("ADMIN")
				// Any request to the app must be authenticated (ie logged in)
				.anyRequest().authenticated()
				.and()
				.formLogin()
				// Show our custom form at the request mapping
				.loginPage("/showMyLoginPage")
				// Login form should POST data to this URL for processing
				.loginProcessingUrl("/authenticateTheUser")
				// Allow everyone to see login page No need to be logged in.
				.permitAll()
				.and()
				.logout().permitAll()
				.and()
				.exceptionHandling().accessDeniedPage("/access-denied");
	}
}
```
### 控制器類別(Controller)
**LoginController.java完整程式碼，對應loginPage("/showMyLoginPage")登入映射配置，以及添加訪問權限錯誤映射，將人員轉向至自訂的錯誤頁面**
```
@Controller
public class LoginController {

	@GetMapping("/showMyLoginPage")
	public String showMyLoginPage() {
		
		return "fancy-login";
	}
	
	// add request mapping for /access-denied
	@GetMapping("/access-denied")
	public String showAccessDenied() {
		
		return "access-denied";
	}
}
```
**DemoController.java完整程式碼，包含主頁面、leaders(MANAGER)以及systems(ADMIN)對應權限訪問映射**
```
@Controller
public class DemoController {
	
	@GetMapping("/")
	public String showHome() {
		
		return "home";
	}

	// add request mapping for /leaders
	@GetMapping("/leaders")
	public String showleaders() {
		
		return "leaders";
	}
	
	// add request mapping for /systems
	@GetMapping("/systems")
	public String showSystems() {

		return "systems";
	}
}
```

### 顯示頁面(View)
>參考網站：https://www.srcmini.com/36789.html
>
>包含Spring Security Taglib聲明、授權標籤、認證標籤等配置與使用方式
>`<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>`

**home.jsp主頁面完整程式碼，簡易製作顯示畫面添加登出按鈕並引用Security標記庫顯示當前登入者名稱以及權限，同時添加訪問權限，相關鏈結只有對應權限的人員才會顯示以及訪問**
```
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>

<html>

<head>
	<title>luv2code Company Home Page</title>
</head>

<body>
	<h2>luv2code Company Home Page - Yoohoo</h2>
	<hr>
	
	<p>
	Welcome to the luv2code Company home page!
	</p>
	
	<hr>
	
	<!-- display user name and role -->
	<p>
		User: <security:authentication property="principal.username" />
		<br><br>
		Roles(s): <security:authentication property="principal.authorities" />
	</p>
				
	<security:authorize access="hasRole('MANAGER')">
	
	<!-- Add a link to point to /leaders ... this is for the managers -->
	<p>
		<a href="${pageContext.request.contextPath}/leaders">LeaderShip Meeting</a>
		(Only for Manager peeps)
	</p>
	
	</security:authorize>
	
	<security:authorize access="hasRole('ADMIN')">
	
	<!-- Add a link to point to /system ... this is for the admins -->	
	<p>
		<a href="${pageContext.request.contextPath}/systems">IT Systems Meeting</a>
		(Only for Admin peeps)
	</p>
	
	</security:authorize>
	
	<hr>
	
	<!-- Add a logout button -->
	<form:form action="${pageContext.request.contextPath}/logout"
			   method="POST">
	
		<input type="submit" value="Logout" />
		
	</form:form>
	
	
</body>

</html>
```
### 普通權限
![image](https://user-images.githubusercontent.com/101872264/219935363-dbf57a5c-172b-424b-a0ab-8708c1d4e01c.png)
### MANAGER權限
![image](https://user-images.githubusercontent.com/101872264/219935345-936a3a87-04b6-4d93-8db6-b107acbe82d9.png)
### ADMIN權限
![image](https://user-images.githubusercontent.com/101872264/219935446-2b854bbf-1451-4d25-8612-908ff45f1522.png)

>`<form:form action="${pageContext.request.contextPath}/authenticateTheUser" method="POST">`
>
>* 對應`loginProcessingUrl("/authenticateTheUser")`，傳送此驗證URL，Spring會提供驗證，不須額外撰寫程式|
>
>* 引用JSTL標籤庫，判斷是否驗證錯誤以顯示錯誤訊息
>
>* 添加登出判斷機制用於確認登出與否
>
>* `name="username"`、`name="password"`默認名稱，Spring Security Filters會自動讀取參數資料
>
>* fancy-login.jsp透過Bootstrap導入引用替登入頁面添加樣式
>
>* CSRF跨域請求偽造，英文全稱是Cross Site Request Forgery，如果使用的是 Spring MVC<form:form>標記或Thymeleaf 2.1+並且正在使用@EnableWebSecurity，CsrfToken則會自動包含（使用CsrfRequestDataValueProcessor），否則須自行手動添加
>
>>```
<input type="hidden"
		   name="${_csrf.parameterName}"
		   value="${_csrf.token}" />
>>```

**fancy-login.jsp登入頁面完整程式碼**
```
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

<!doctype html>
<html lang="en">

<head>
	
	<title>Login Page</title>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
	
	<!-- Reference Bootstrap files -->
	<link rel="stylesheet"
		 href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
	
	<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.0/jquery.min.js"></script>
	
	<script	src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

</head>

<body>

	<div>
		
		<div id="loginbox" style="margin-top: 50px;"
			class="mainbox col-md-3 col-md-offset-2 col-sm-6 col-sm-offset-2">
			
			<div class="panel panel-info">

				<div class="panel-heading">
					<div class="panel-title">Sign In</div>
				</div>

				<div style="padding-top: 30px" class="panel-body">

					<!-- Login Form -->
					<form action="${pageContext.request.contextPath}/authenticateTheUser" 
						  method="POST" class="form-horizontal">

					    <!-- Place for messages: error, alert etc ... -->
					    <div class="form-group">
					        <div class="col-xs-15">
					            <div>		
					            
					            <!-- Check for login error -->
			   					<c:if test="${param.error != null}">
					            							            
									<div class="alert alert-danger col-xs-offset-1 col-xs-10">
										Invalid username and password.
									</div>
								</c:if>
								
								<div class="alert alert-success col-xs-offset-1 col-xs-10">
									You have been logged out.
								</div>
								    
					            </div>
					        </div>
					    </div>

						<!-- User name -->
						<div style="margin-bottom: 25px" class="input-group">
							<span class="input-group-addon"><i class="glyphicon glyphicon-user"></i></span> 
							
							<input type="text" name="username" placeholder="username" class="form-control">
						</div>

						<!-- Password -->
						<div style="margin-bottom: 25px" class="input-group">
							<span class="input-group-addon"><i class="glyphicon glyphicon-lock"></i></span> 
							
							<input type="password" name="password" placeholder="password" class="form-control" >
						</div>

						<!-- Login/Submit Button -->
						<div style="margin-top: 10px" class="form-group">						
							<div class="col-sm-6 controls">
								<button type="submit" class="btn btn-success">Login</button>
							</div>
						</div>
						
						<!-- I'm manually adding tokens ... Bro! -->
						<input type="hidden"
							   name="${_csrf.parameterName}"
							   value="${_csrf.token}" />

					</form>

				</div>

			</div>

		</div>

	</div>

</body>
</html>
```
![image](https://user-images.githubusercontent.com/101872264/219935473-204bdfb9-c3ac-4cb8-8197-ce37bfd80ac7.png)

**leaders.jsp，(MANAGER)權限才可訪問的頁面**
```
<html>

<head>
	<title>luv2code LEADERS Home Page</title>
</head>

<body>

<h2>luv2code LEADERS Home Page</h2>


<hr>

<p>
	See you in Brazil ... for our annual Leadership retreat!
	<br>
	Keep this trip a secret, don't tell the regular employees LOL
</p>

<hr>

<a href="${pageContext.request.contextPath}/">Back to Home Page</a>

</body>

</html>
```
![image](https://user-images.githubusercontent.com/101872264/219935522-6a3aa400-fae1-41f8-a4ab-8804b3179a6c.png)

**systems.jsp，(ADMIN)權限才可訪問的頁面**
```
<html>

<head>
	<title>luv2code SYSTEMS Home Page</title>
</head>

<body>

<h2>luv2code SYSTEMS Home Page</h2>


<hr>

<p>
	We have our annual holiday Caribbean cruise coming up. Register now!
	<br>
	Keep this trip a secret, don't tell the regular employees LOL
</p>

<hr>

<a href="${pageContext.request.contextPath}/">Back to Home Page</a>

</body>

</html>
```
![image](https://user-images.githubusercontent.com/101872264/219935561-fb88d1cd-b082-41b0-bf34-7ec1f1b88bf0.png)

**access-denied.jsp，用於顯示訪問錯誤頁面，取代原先錯誤頁面，當權限為匹配將會轉向至此頁面告知人員**
```
<html>

<head>
	<title>luv2code - Access Denied</title>
</head>

<body>

	<h2>Access Denied - You are not authorized to access this resource.</h2>

	<hr>
	
	<a href="${pageContext.request.contextPath}/">Back to Home Page</a>	
</body>

</html>
```
![image](https://user-images.githubusercontent.com/101872264/219935678-f9c6add9-847e-431b-8bf7-2e9b936f627f.png)
