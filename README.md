# Tools
-	RestAssured
-	Allure
-	Jenkins


# Requirtment 
-	Code Editor you can use the best for you, I Suggest (Eclips or Intelij)
- Install plugin Lombok on your Eclipse
-	Install Allure - http://allure.qatools.ru/ -> How to Generate Report Allure? In CMD/TERMINAL you can write “allure serve”

# Step
1.	Open Code Editor, I use Eclipse
2.	Create Maven Project
3.	Edit pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>hot_api</groupId>
	<artifactId>hot_api</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<properties>
		<aspectj.version>1.9.2</aspectj.version>
	</properties>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.6.2</version>

				<configuration>
					<compilerVersion>1.8</compilerVersion>
					<source>1.6</source>
					<target>1.6</target>
				</configuration>
			</plugin>

			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>3.0.0-M4</version>

				<configuration>
					<testFailureIgnore>true</testFailureIgnore>
					<argLine>
						-javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar"
					</argLine>

					<suiteXmlFiles>
						<suiteXmlFiles>testng.xml</suiteXmlFiles>
					</suiteXmlFiles>
				</configuration>

				<dependencies>
					<dependency>
						<groupId>org.aspectj</groupId>
						<artifactId>aspectjweaver</artifactId>
						<version>${aspectj.version}</version>
					</dependency>
				</dependencies>

			</plugin>
		</plugins>
	</build>

	<dependencies>

		<dependency>
			<groupId>com.github.javafaker</groupId>
			<artifactId>javafaker</artifactId>
			<version>1.0.2</version>
		</dependency>
		<!-- Allure plugin for REST-Assured -->
		<dependency>
			<groupId>io.qameta.allure</groupId>
			<artifactId>allure-rest-assured</artifactId>
			<version>2.13.1</version>
		</dependency>
		<!--Json Schema matcher -->
		<dependency>
			<groupId>com.jayway.restassured</groupId>
			<artifactId>json-schema-validator</artifactId>
			<version>2.9.0</version>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<version>1.18.12</version>
			<scope>provided</scope>
		</dependency>

		<dependency>
			<groupId>io.rest-assured</groupId>
			<artifactId>rest-assured</artifactId>
			<version>3.3.0</version>
		</dependency>
		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
			<version>2.8.5</version>
		</dependency>
		<dependency>
			<groupId>org.testng</groupId>
			<artifactId>testng</artifactId>
			<version>6.9.10</version>
		</dependency>
		<dependency>
			<groupId>io.qameta.allure</groupId>
			<artifactId>allure-testng</artifactId>
			<version>2.13.1</version>
		</dependency>
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjweaver</artifactId>
			<version>${aspectj.version}</version>
		</dependency>
		<dependency>
			<groupId>javax.xml.bind</groupId>
			<artifactId>jaxb-api</artifactId>
			<version>2.2.11</version>
		</dependency>
	</dependencies>
	
		<reporting>
		<excludeDefaults>true</excludeDefaults>
		<plugins>
			<plugin>
				<groupId>io.qameta.allure</groupId>
				<artifactId>allure-maven</artifactId>
				<version>2.9</version>
			</plugin>
		</plugins>
	</reporting>

</project>
```
4. Create package 'config' on ```src/test/java```
5. Create Class ```BaseTest.java```
```java
package config;

import org.testng.annotations.BeforeMethod;
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.RestAssured;
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;

public class BaseTest {

	protected RequestSpecification requestSpecificationToMerge = new RequestSpecBuilder()
			.setBaseUri("https://baseApiUrl/api/v1")
			.setContentType(ContentType.JSON)
			.build();

	@BeforeMethod
	public void setFilter() {
		RestAssured.filters(new AllureRestAssured());
	}

}

```
6. Create package 'function' on ```src/test/java``` for list your function
7. Create package 'lombok' on ```src/test/java``` for post method
```java
package lombok;

@Data
@Builder
public class Comment {
	
	String username;
	int password;
}
```
8. Create package 'testcases' ```src/test/java``` for list your testcases
```java

//POST METHOD
	@Test
	public void login() {			
			lombok.Login body = lombok.Login.builder()
					.username("username@hgh.jk")
					.password("password")
					.build();
			
			given()
			.spec(requestSpecificationToMerge)
			.basePath("/login")
			.header("Authorization", new Token().login())
			.body(body)
			.log().all()
			.when()
			.post()
			.then()
			.log().body()
			.body(matchesJsonSchemaInClasspath("login.json")); // For validate JSON Schema
	}
```
```java
// METHOD GET
@Test(priority = 0)
	public void getHome() {
		
		given()
		.spec(requestSpecificationToMerge)
		.basePath("/home")
		.header("Authorization", new Token().visitor())
		.when()
		.get()
		.then()
		.log().body()
		.body(matchesJsonSchemaInClasspath("home.json")); // Validate JSON SCHEMA
	}
```
