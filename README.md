# notes





# Spring Boot Interview

## Why will you choose Spring Boot over Spring framework?

- Dependency resolution / Avoid version conflict
- Avoid additional configuration
- Embed Tomcat, Jetty (no need to deploy WAR files)
- Provide production-ready features such as metrics, health checks

application-context.xml

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/jdbc
                           http://www.springframework.org/schema/jdbc/spring-jdbc.xsd">

    <!-- Enable Spring's annotation-based configuration -->
    <context:annotation-config />
    
    <!-- Component scan for automatic bean detection -->
    <context:component-scan base-package="com.example" />

    <!-- Define a DataSource bean -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/mydatabase" />
        <property name="username" value="root" />
        <property name="password" value="password" />
    </bean>
    
    <!-- Define a JdbcTemplate bean for database operations -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- Define beans for services, repositories, or other components -->
    <bean id="userService" class="com.example.UserService">
        <property name="userRepository" ref="userRepository" />
    </bean>
    
    <bean id="userRepository" class="com.example.UserRepository">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean>

    <!-- Other Bean Definitions -->

</beans>
```

application-servlet.xml

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/mvc
                           http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- Enable Spring's annotation-based configuration -->
    <context:annotation-config />
    
    <!-- Enable Spring MVC -->
    <mvc:annotation-driven />

    <!-- Component scan for automatic bean detection -->
    <context:component-scan base-package="com.example" />

    <!-- View Resolver -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" />
        <property name="suffix" value=".jsp" />
    </bean>

    <!-- Static Resources Configuration -->
    <mvc:resources mapping="/resources/**" location="/resources/" />

    <!-- Handler Mapping -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />

    <!-- Handler Adapter -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />

    <!-- Other Bean Definitions -->

</beans>
```

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- Servlet Configuration -->
    <servlet>
        <servlet-name>springDispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/application-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springDispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- Character Encoding Filter -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- Welcome File List -->
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

</web-app>
```

## @Async

```java
// Config: AsyncConfiguration.java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@Configuration
@EnableAsync
public class AsyncConfiguration {

    @Bean("asyncTaskExecutor")
    public Executor asyncTaskExecutor(){
        ThreadPoolTaskExecutor taskExecutor=new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(4);
        taskExecutor.setQueueCapacity(150);
        taskExecutor.setMaxPoolSize(4);
        taskExecutor.setThreadNamePrefix("AsyncTaskThread-");
        taskExecutor.initialize();;
        return taskExecutor;
    }
}
```

```java
// Service: OrderService.java

/* All Required process */
/*
      1. Inventory service check order availability
      2. Process payment for order
      3. Notify to the user
      3. Assign to vendor
      4. packaging
      5. assign delivery partner
      6. assign trailer
      7. dispatch product
      **/

public Order processOrder(Order order) throws InterruptedException {
  order.setTrackingId(UUID.randomUUID().toString());
  // if the product ordered found in the inventory
  if (inventoryService.checkProductAvailability(order.getProductId())) {
    // process payment
    paymentService.processPayment(order);
  } else {
    throw new RuntimeException("Technical issue please retry");
  }
  return order;
}

@Async("asyncTaskExecutor")
public void notifyUser(Order order) throws InterruptedException {
  Thread.sleep(4000L);
  log.info("Notified to the user " + Thread.currentThread().getName());
}

@Async("asyncTaskExecutor")
public void assignVendor(Order order) throws InterruptedException {
  Thread.sleep(5000L);
  log.info("Assign order to vendor " + Thread.currentThread().getName());
}

@Async("asyncTaskExecutor")
public void packaging(Order order) throws InterruptedException {
  Thread.sleep(2000L);
  log.info("Order packaging completed " + Thread.currentThread().getName());
}

@Async("asyncTaskExecutor")
public void assignDeliveryPartner(Order order) throws InterruptedException {
  Thread.sleep(10000L);
  log.info("Delivery partner assigned " + Thread.currentThread().getName());
}

@Async("asyncTaskExecutor")
public void assignTrailerAndDispatch(Order order) throws InterruptedException {
  Thread.sleep(3000L);
  log.info("Trailer assigned and Order dispatched " + Thread.currentThread().getName());
}
```

```java
// Controller

@PostMapping
public ResponseEntity<Order> processOrder(@RequestBody Order order) throws InterruptedException {
  // synchronous
  service.processOrder(order); 
  // asynchronous
  service.notifyUser(order);
  service.assignVendor(order);
  service.packaging(order);
  service.assignDeliveryPartner(order);
  service.assignTrailerAndDispatch(order);
  return ResponseEntity.ok(order);
}
```



```java
// Order

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class Order {

    private int productId;
    private String name;
    private String productType;
    private int qty;
    private double price;
    private String trackingId;
}
```

# Exception Handling

## 1. Introduction

Exception handling in microservices applications

Food Order Flow

Client -> access to FoodOrderGateway -> load balance to FoodOrderClient -> call RestaurantService API through service client

```
# logged in service-client in FoodOrderClient
c.j.client.RestaurantServiceClient : RestaurantServiceClient::fetchOrderStatus caught the HttpServer server error ...

# logged in advice in FoodOrderClient
.j.a.FoodOrderClientGlobalExceptionHandler : FoodOrderClientGlobalExceptionHandler::handleFoodOrderClientException exception caught ...
```



## 2. Exception handling in microservices applications

### 1) FoodOrderGateway

```java
package com.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class FoodOrderGatewayApplication {

	public static void main(String[] args) {
		SpringApplication.run(FoodOrderGatewayApplication.class, args);
	}

}
```



```yaml
spring:
	application:
  	name: FOOD-ORDER-GATEWAY	
  eureka:
  	client:
    	service-url:
      	defaultZone : http://localhost:8761/eureka/
	cloud:
  	gateway:
    	routes:
      	- id: food-order-app
        	uri: lb://FOOD-ORDER-APP
        	predicates:
          	- Path=/food-order/**
       	- id: restaurant-service
        	uri: lb://RESTAURANT-SERVICE
        	predicates:
          	- Path=/restaurant/**
 
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.1.2</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.demo</groupId>
	<artifactId>food-order-gateway</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>food-order-gateway</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>17</java.version>
		<spring-cloud.version>2022.0.4</spring-cloud.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webflux</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-gateway</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
		</plugins>
	</build>
	<repositories>
		<repository>
			<id>netflix-candidates</id>
			<name>Netflix Candidates</name>
			<url>https://artifactory-oss.prod.netflix.net/artifactory/maven-oss-candidates</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

</project>
```

### 2) FoodOrderClient

```java
package com.javatechie;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient // Register to eureka
public class SwiggyAppApplication {

	public static void main(String[] args) {
		SpringApplication.run(SwiggyAppApplication.class, args);
	}

}
```

```yaml
server:
  port: 8081

spring:
  application:
    name: FOOD-ORDER-APP

eureka:
  client:
    service-url:
      defaultZone : http://localhost:8761/eureka/
```

**advice** `advice/FoodOrderServiceGlobalExceptionHandler`

```java
package com.javatechie.advice;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.demo.dto.CustomErrorResponse;
import com.demo.exception.FoodOrderClientException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
@Slf4j
public class FoodOrderClientGlobalExceptionHandler {

  // Handle the FoodOrderClientException throw in service-client
  // and return the response object to client (e.g.: postman)
  @ExceptionHandler(FoodOrderClientException.class)
  public ResponseEntity<?> handleFoodOrderClientException(FoodOrderClientException ex) 
    throws JsonProcessingException {
    // Log the exception message to console
    log.error("FoodOrderClientGlobalExceptionHandler::handleFoodOrderClientException exception caught {}",
      				ex.getMessage());
    
    CustomErrorResponse errorResponse = new ObjectMapper()
      				.readValue(ex.getMessage(), CustomErrorResponse.class);
    
    return ResponseEntity.internalServerError().body(errorResponse);
  }

  //    @ExceptionHandler(Exception.class)
  //    public ResponseEntity<?> handleGenericException(Exception ex){
  //        CustomErrorResponse errorResponse = CustomErrorResponse.builder()
  //                .status(HttpStatus.INTERNAL_SERVER_ERROR)
  //                .errorCode(GlobalErrorCode.GENERIC_ERROR)
  //                .errorMessage(ex.getMessage())
  //                .build()  ;
  //        log.error("RestaurantServiceGlobalExceptionHandler::handleGenericException exception caught {}",ex.getMessage());
  //        return ResponseEntity.internalServerError().body(errorResponse);
  //    }
}
```

**exception** `exception/FoodOrderServiceException`

```java
package com.demo.exception;

public class FoodOrderServiceException extends RuntimeException{

  public FoodOrderServiceException(String message) {
    super(message);
  }
}
```

**dto** `dto/CustomErrorResponse`

```java
package com.javatechie.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.http.HttpStatus;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class CustomErrorResponse {

  private HttpStatus status;
  private String errorMessage;
  private String errorCode;
}
```

**service** `service/FoodOrderClientService`

```java
@Service
public class FoodOrderClientService {

    @Autowired
    private RestaurantServiceClient restaurantServiceClient;

    public String greeting() {
        return "Welcome to Food Order Client Service";
    }

    public OrderResponseDTO checkOrderStatus(String orderId) {
        return restaurantServiceClient.fetchOrderStatus(orderId);
    }
}
```

**service-client** `service-client/RestaurantServiceClient`

```java
@Component
@Slf4j
public class RestaurantServiceClient {
  
  @Autowired
  private RestTemplate template;

  public OrderResponseDTO fetchOrderStatus(String orderId) {
    OrderResponseDTO orderResponseDTO = null;
    
    try {
      // Call restaurant-service API (other application service)
      orderResponseDTO = template.getForObject(
        			"http://RESTAURANT-SERVICE/restaurant/orders/status/" + orderId, OrderResponseDTO.class);
    } catch (HttpServerErrorException errorException) {
      // Log the exception message to console
      log.error("RestaurantServiceClient::fetchOrderStatus caught the HttpServer server error {}", 
                errorException.getResponseBodyAsString());
      // Throw FoodOrderServiceException which will be handled in advice
      throw new FoodOrderClientException(errorException.getResponseBodyAsString());
    } catch (Exception ex) {
      log.error("RestaurantServiceClient::fetchOrderStatus caught the generic error {}", ex.getMessage());
    }
    
    return orderResponseDTO;
  }
}
```





```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.2</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.demo</groupId>
  <artifactId>food-order-client-registry</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>food-order-client-registry</name>
  <description>Demo project for Spring Boot</description>
  <properties>
    <java.version>17</java.version>
    <spring-cloud.version>2022.0.4</spring-cloud.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
  <repositories>
    <repository>
      <id>netflix-candidates</id>
      <name>Netflix Candidates</name>
      <url>https://artifactory-oss.prod.netflix.net/artifactory/maven-oss-candidates</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

</project>
```

### 3) RestaurantService

```java
package com.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient // Register to eureka
public class RestaurantServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(RestaurantServiceApplication.class, args);
	}

}
```

```yaml
server:
  port: 8082

spring:
  application:
    name: RESTAURANT-SERVICE

eureka:
  client:
    service-url:
      defaultZone : http://localhost:8761/eureka/
```



```java
package com.demo.advice;

import com.demo.dto.CustomErrorResponse;
import com.demo.dto.GlobalErrorCode;
import com.demo.exception.OrderNotFoundException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
@Slf4j
public class RestaurantServiceGlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<?> handleOrderNotFoundException(OrderNotFoundException ex) {
        CustomErrorResponse errorResponse = CustomErrorResponse.builder()
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .errorCode(GlobalErrorCode.ERROR_ORDER_NOT_FOUND)
                .errorMessage(ex.getMessage())
                .build();
        log.error("RestaurantServiceGlobalExceptionHandler::handleOrderNotFoundException exception caught {}",
                  ex.getMessage());
        return ResponseEntity.internalServerError().body(errorResponse);
    }

//    @ExceptionHandler(Exception.class)
//    public ResponseEntity<?> handleGenericException(Exception ex){
//        CustomErrorResponse errorResponse= CustomErrorResponse.builder()
//                .status(HttpStatus.INTERNAL_SERVER_ERROR)
//                .errorCode(GlobalErrorCode.GENERIC_ERROR)
//                .errorMessage(ex.getMessage())
//                .build()  ;
//        log.error("RestaurantServiceGlobalExceptionHandler::handleGenericException exception caught {}",ex.getMessage());
//        return ResponseEntity.internalServerError().body(errorResponse);
//    }
}
```

```java
package com.demo.exception;

public class OrderNotFoundException extends RuntimeException {

    public OrderNotFoundException(String message) {
        super(message);
    }
}
```

```java
package com.demo.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.http.HttpStatus;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class CustomErrorResponse {

    private HttpStatus status;
    private String errorMessage;
    private String errorCode;
}
```

```java
package com.demo.service;

import com.demo.dao.RestaurantOrderDAO;
import com.demo.dto.OrderResponseDTO;
import com.demo.exception.OrderNotFoundException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class RestaurantService {
  
    @Autowired
    private RestaurantOrderDAO orderDAO;

    public String greeting() {
        return "Welcome to Swiggy Restaurant service";
    }

    public OrderResponseDTO getOrder(String orderId) {
        OrderResponseDTO orderResponseDTO = orderDAO.getOrders(orderId);
        if (orderResponseDTO != null) {
            return orderResponseDTO;
        } else {
          	// Throw OrderNotFoundException
            throw new OrderNotFoundException("Order not available with id :" + orderId);
        }
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.1.2</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.demo</groupId>
	<artifactId>restaurant-service</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>restaurant-service</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>17</java.version>
		<spring-cloud.version>2022.0.4</spring-cloud.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<excludes>
						<exclude>
							<groupId>org.projectlombok</groupId>
							<artifactId>lombok</artifactId>
						</exclude>
					</excludes>
				</configuration>
			</plugin>
		</plugins>
	</build>
	<repositories>
		<repository>
			<id>netflix-candidates</id>
			<name>Netflix Candidates</name>
			<url>https://artifactory-oss.prod.netflix.net/artifactory/maven-oss-candidates</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

</project>
```

# Cache

## 1. Redis

`redis-config.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
data:
  redis.conf: |
    cluster-enabled yes
    cluster-node-timeout 5000
    cluster-config-file /data/nodes.conf
    appendonly yes
```

`redis-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
  clusterIP: None
```

`redis-statefulset.yaml`

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis-service
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
          volumeMounts:
            - name: redis-data
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: redis-data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
```

How to use in springboot

```xml
<dependencies>
  <!-- 其他依赖 -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
</dependencies>
```



`application.yaml`

```yaml
spring:
  redis:
    host: redis-service
    port: 6379
```

```java
@Service
public class MyService {

  	// @Cacheable 注解将方法的返回值存储到 Redis 缓存中，并使用给定的 id 作为缓存的键
    @Cacheable(value = "myCache", key = "#id")
    public String getDataFromCache(String id) {
        // 从数据库或其他数据源获取数据的逻辑
        return "Data for " + id;
    }
}
```

```java
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {

    // 创建 RedisTemplate 对象
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
    // 设置 RedisConnection 工厂（实现多种 Java Redis 客户端接入的工厂）
    redisTemplate.setConnectionFactory(redisConnectionFactory);

    // 使用 Jackson2JsonRedisSerialize 替换默认序列化
    Jackson2JsonRedisSerializer<?> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);

    ObjectMapper mapper = new ObjectMapper();
    mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    mapper.activateDefaultTyping(mapper.getPolymorphicTypeValidator(), ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);

    jackson2JsonRedisSerializer.setObjectMapper(mapper);
    redisTemplate.setDefaultSerializer(jackson2JsonRedisSerializer);
    redisTemplate.setKeySerializer(StringRedisSerializer.UTF_8);
    return redisTemplate;
}

```







# Splunk

## 1. Introduction

**URL**: Where your splunk server will redirect the logs

**Host**: What is the host where your splunk server is running

**Token**: What is the security token to connect with your splunk server

**Index**: In which index you want to push your application logs

**Source**: What is your source type (who will send your logs)

## 2. Splunk settings

### 1). Global setting

![global_setting_01](/Users/mario_ninja/Desktop/md-notes/global_setting_01.png)

### 2). New token

![new_token_01](/Users/mario_ninja/Desktop/md-notes/new_token_01.png)

![new_token_02_create_new_index](/Users/mario_ninja/Desktop/md-notes/new_token_02_create_new_index.png)

![new_token_03_add_index_and_source_type](/Users/mario_ninja/Desktop/md-notes/new_token_03_add_index_and_source_type.png)

![new_token_04_review_and_copy](/Users/mario_ninja/Desktop/md-notes/new_token_04_review_and_copy.png)

![new_token_05_submit](/Users/mario_ninja/Desktop/md-notes/new_token_05_submit.png)

## 3. Dependency

```xml
<repositories>
  <repository>
    <id>splunk-artifactory</id>
    <name>Splunk Releases</name>
    <url>https://splunk.artifactoryonline.com/artifactory/ext-releases-local</url>
  </repository>
</repositories>

<!-- exclude spring logging -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>

<!-- log4j2 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>

<!-- Splunk -->
<dependency>
  <groupId>com.splunk.logging</groupId>
  <artifactId>splunk-library-javalogging</artifactId>
  <version>1.8.0</version>
</dependency>
```

## 4. log4j2-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout
                    pattern="%style{%d{ISO8601}} %highlight{%-5level }[%style{%t}{bright,blue}] %style{%C{10}}{bright,yellow}: %msg%n%throwable" />
        </Console>
        <SplunkHttp
                name="splunkhttp"
                url="http://localhost:8088"
                token="7f99549c-adfb-482b-af62-c3b3dc103edd"
                host="localhost"
                index="order_api_dev"
                type="raw"
                source="http-event-logs"
                sourcetype="log4j"
                messageFormat="text"
                disableCertificateValidation="true">
            <PatternLayout pattern="%m" />
        </SplunkHttp>

    </Appenders>

    <Loggers>
        <!-- LOG everything at INFO level -->
        <Root level="info">
            <AppenderRef ref="console" />
            <AppenderRef ref="splunkhttp" />
        </Root>
    </Loggers>
</Configuration>
```



![disable_other_indexer](/Users/mario_ninja/Desktop/md-notes/disable_other_indexer.png)

![searching_01](/Users/mario_ninja/Desktop/md-notes/searching_01.png)

![searching_02](/Users/mario_ninja/Desktop/md-notes/searching_02.png)

# ELK

## 1. What is ELK?

ELK stack is an acronym that refers to a collection of three popular open-source tools: Elasticsearch, Logstash, and Kibana. Together, these tools provide a powerful platform for managing and analyzing large volumes of log data in real-time. Here's a brief explanation of each component:

1. Elasticsearch: Elasticsearch is a distributed, scalable search and analytics engine. It is designed to store, search, and analyze large amounts of structured and unstructured data in near real-time. Elasticsearch uses a document-oriented approach, where data is stored as JSON documents, making it flexible for various use cases. It provides powerful search capabilities, aggregations, and supports full-text search, geospatial queries, and more.
2. Logstash: Logstash is a data processing pipeline that allows you to collect, transform, and enrich data from various sources. It can ingest data from log files, databases, message queues, and many other sources. Logstash provides a wide range of input plugins to collect data, filter plugins to process and transform data, and output plugins to send the processed data to different destinations. It is highly extensible and customizable to handle diverse data formats and sources.
3. Kibana: Kibana is a data visualization and exploration tool that works alongside Elasticsearch. It provides a web-based interface for querying, analyzing, and visualizing data in Elasticsearch. Kibana offers a rich set of features to create interactive dashboards, charts, maps, and graphs based on the indexed data. It allows you to explore data in real-time, apply filters, drill down into details, and share visualizations and dashboards with others.

The ELK stack is commonly used for log management and analysis. Log data from various sources can be collected by Logstash, processed, and stored in Elasticsearch. Kibana then provides a user-friendly interface to search, analyze, and visualize the log data, enabling users to gain valuable insights, monitor system performance, troubleshoot issues, and detect anomalies.

Note that there have been variations and extensions of the ELK stack, including components such as Beats (lightweight data shippers) and Logstash Forwarder (now deprecated), which enhance data collection capabilities. The acronym "ELK" is often used generically to refer to the broader ecosystem of tools and technologies for log management and analysis, even if the specific stack includes additional components beyond Elasticsearch, Logstash, and Kibana.

![elk](/Users/mario_ninja/Desktop/md-notes/elk.png)

## 2. Steps to install ELK

### 1). Download ELK stack from these links 

Elastic: https://www.elastic.co/downloads/elasticsearch

Logstash: https://www.elastic.co/downloads/logstash

Kibana: https://www.elastic.co/downloads/kibana

### 2). Extract each into separate folders

In kibana.yaml

```yaml
# uncomment the following
elasticsearch.hosts: ["http://localhost:9200"]
```



### 3). Start them separately

```sh
# Start Elasticsearch
# Test: localhost:9200
\bin> elasticsearch.bat

# Start Kibana 
# Test: localhost:5601
\bin> kibana.bat 

```



application.yaml

```yaml
logging:
  file: C:/Users/basan/Desktop/logs/elk-stack.log
```

It's important to note that the specified directory (`C:/Users/basan/Desktop/logs/` in the example) should exist and the application needs proper write permissions to create the log file. If the directory doesn't exist, you may need to create it manually.

Prepare `logstash.conf`, store in the logstash bin folder

```
input {
  file {
  	# The path of your log file
    path => "C:/Users/basan/Desktop/logs/elk-stack.log"
    start_position => "beginning"
    sincedb_path => "NUL"
  }
}

filter {
  # Add any necessary filters for log parsing and enrichment
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    # index => "your-index-name-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => rubydebug
  }
}
```

```sh
# Start Logstash
bin> logstash.bat -f logstash.conf
```

Check the indices in ElasticSearch

localhost:9200/_cat/indices

Create index in Kibana



Kibana => Click `Management` in sidebar => Click `Index Patterns` under Kibana => Click `Create Index Pattern` => Input `logstash-*`  in the index pattern textfield.  => Click `Next Step` => Select `I don't want to use the Time Filter` => Click `Create Index Patten`

Click `Discover` icon in sidebar





# Testing

## 1. Unit Test

### 1). Unit test for repository

```java
@DataJpaTest
@AutoConfigureTestDatabase(connection = EmbeddedDatabaseConnection.H2)
public class PokemonRepositoryTests {

    @Autowired
    private PokemonRepository pokemonRepository;

    @Test
    public void PokemonRepository_SaveAll_ReturnSavedPokemon() {

        // Arrange
        Pokemon pokemon = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();

        // Act
        Pokemon savedPokemon = pokemonRepository.save(pokemon);

        // Assert
        Assertions.assertThat(savedPokemon).isNotNull();
        Assertions.assertThat(savedPokemon.getId()).isGreaterThan(0);
    }
  
   @Test
    public void PokemonRepository_GetAll_ReturnMoreThenOnePokemon() {
        Pokemon pokemon = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();
        Pokemon pokemon2 = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();

        pokemonRepository.save(pokemon);
        pokemonRepository.save(pokemon2);

        List<Pokemon> pokemonList = pokemonRepository.findAll();

        Assertions.assertThat(pokemonList).isNotNull();
        Assertions.assertThat(pokemonList.size()).isEqualTo(2);
    }
  
  	@Test
    public void PokemonRepository_FindById_ReturnPokemon() {
        Pokemon pokemon = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();

        pokemonRepository.save(pokemon);

        Pokemon pokemonList = pokemonRepository.findById(pokemon.getId()).get();

        Assertions.assertThat(pokemonList).isNotNull();
    }

    @Test
    public void PokemonRepository_FindByType_ReturnPokemonNotNull() {
        Pokemon pokemon = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();

        pokemonRepository.save(pokemon);

        Pokemon pokemonList = pokemonRepository.findByType(pokemon.getType()).get();

        Assertions.assertThat(pokemonList).isNotNull();
    }
  
  	@Test
    public void PokemonRepository_UpdatePokemon_ReturnPokemonNotNull() {
        Pokemon pokemon = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();

        pokemonRepository.save(pokemon);

        Pokemon pokemonSave = pokemonRepository.findById(pokemon.getId()).get();
        pokemonSave.setType("Electric");
        pokemonSave.setName("Raichu");

        Pokemon updatedPokemon = pokemonRepository.save(pokemonSave);

        Assertions.assertThat(updatedPokemon.getName()).isNotNull();
        Assertions.assertThat(updatedPokemon.getType()).isNotNull();
    }

    @Test
    public void PokemonRepository_PokemonDelete_ReturnPokemonIsEmpty() {
        Pokemon pokemon = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();

        pokemonRepository.save(pokemon);

        pokemonRepository.deleteById(pokemon.getId());
        Optional<Pokemon> pokemonReturn = pokemonRepository.findById(pokemon.getId());

        Assertions.assertThat(pokemonReturn).isEmpty();
    }

}
```



```java
// ReviewRepositoryTests.java

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.jdbc.EmbeddedDatabaseConnection;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.util.List;
import java.util.Optional;

@DataJpaTest
@AutoConfigureTestDatabase(connection = EmbeddedDatabaseConnection.H2)
public class ReviewRepositoryTests {
    private ReviewRepository reviewRepository;

    @Autowired
    public ReviewRepositoryTests(ReviewRepository reviewRepository) {
        this.reviewRepository = reviewRepository;
    }

    @Test
    public void ReviewRepository_SaveAll_ReturnsSavedReview() {
        Review review = Review.builder().title("title").content("content").stars(5).build();

        Review savedReview = reviewRepository.save(review);

        Assertions.assertThat(savedReview).isNotNull();
        Assertions.assertThat(savedReview.getId()).isGreaterThan(0);
    }

    @Test
    public void ReviewRepostory_GetAll_ReturnsMoreThenOneReview() {
        Review review = Review.builder().title("title").content("content").stars(5).build();
        Review review2 = Review.builder().title("title").content("content").stars(5).build();

        reviewRepository.save(review);
        reviewRepository.save(review2);

        List<Review> reviewList = reviewRepository.findAll();

        Assertions.assertThat(reviewList).isNotNull();
        Assertions.assertThat(reviewList.size()).isEqualTo(2);
    }

    @Test
    public void ReviewRepository_FindById_ReturnsSavedReview() {
        Review review = Review.builder().title("title").content("content").stars(5).build();

        reviewRepository.save(review);

        Review reviewReturn = reviewRepository.findById(review.getId()).get();

        Assertions.assertThat(reviewReturn).isNotNull();
    }

    @Test
    public void ReviewRepository_UpdateReview_ReturnReview() {
        Review review = Review.builder().title("title").content("content").stars(5).build();

        reviewRepository.save(review);

        Review reviewSave = reviewRepository.findById(review.getId()).get();
        reviewSave.setTitle("title");
        reviewSave.setContent("content");
        Review udpatedPokemon = reviewRepository.save(reviewSave);

        Assertions.assertThat(udpatedPokemon.getTitle()).isNotNull();
        Assertions.assertThat(udpatedPokemon.getContent()).isNotNull();
    }

    @Test
    public void ReviewRepository_ReviewDelete_ReturnReviewIsEmpty() {
        Review review = Review.builder().title("title").content("content").stars(5).build();

        reviewRepository.save(review);

        reviewRepository.deleteById(review.getId());
        Optional<Review> reviewReturn = reviewRepository.findById(review.getId());

        Assertions.assertThat(reviewReturn).isEmpty();
    }

}
```

### 2). Unit test for service

In unit testing, it is generally recommended to isolate the unit under test (in this case, the `PokemonService`) from its dependencies. By using a mock repository and setting up specific behaviors using Mockito, we can focus solely on testing the logic of the `PokemonService` without involving the actual repository implementation.

There are several reasons why we use a mock repository instead of the real repository for this particular test case:

1. **Controlled Testing Environment:** Using a mock repository allows us to have full control over the behavior of the repository during the test. We can specify exactly what the repository's `save` method should return, ensuring that it behaves consistently and predictably.
2. **Test Isolation:** By mocking the repository, we can isolate the testing of the `PokemonService` from the actual data storage layer. This allows us to focus specifically on testing the service logic without worrying about the underlying repository implementation or the need for a real database connection.
3. **Speed and Efficiency:** Mocking the repository eliminates the need for database operations, making the test run faster and more efficient. It avoids the overhead of setting up and tearing down a real database connection and executing actual database queries.
4. **Avoiding Side Effects:** When using a real repository, saving a real object could have unintended side effects, such as modifying the actual data in the database. Using a mock repository ensures that the test does not have any unintended consequences on the data.
5. **Test in Isolation:** Unit tests should be independent and self-contained, and using a mock repository helps achieve this isolation. It allows the test to focus solely on the behavior of the `PokemonService` method being tested without relying on external dependencies.

It's important to note that while using a mock repository is suitable for unit testing, it does not replace the need for integration or end-to-end testing where the real repository and actual data storage are involved. In those scenarios, testing with a real repository is more appropriate to ensure the complete system functions correctly.

`PokemonServiceTests.java`

```java
import com.pokemonreview.api.repository.PokemonRepository;
import com.pokemonreview.api.service.impl.PokemonServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.mockito.Mockito.doNothing;
import static org.mockito.Mockito.when;

/*
The @ExtendWith(MockitoExtension.class) annotation is used in JUnit 5 to integrate Mockito's mocking capabilities into the test framework. It allows you to use Mockito's features, such as creating mock objects and defining their behaviors, within your JUnit tests.
*/
@ExtendWith(MockitoExtension.class)
public class PokemonServiceTests {

  	// Create a mock object
    @Mock
    private PokemonRepository pokemonRepository;

  	// Inject the mock objects into the PokemonServiceImpl class
    @InjectMocks
    private PokemonServiceImpl pokemonService;

    @Test
    public void PokemonService_CreatePokemon_ReturnsPokemonDto() {
      	// Arrange
        Pokemon pokemon = Pokemon.builder()
                .name("pikachu")
                .type("electric")
          			.build();
      
        PokemonDto pokemonDto = PokemonDto.builder()
          			.name("pickachu")
          			.type("electric")
          			.build();
	
        when(pokemonRepository.save(Mockito.any(Pokemon.class))).thenReturn(pokemon);
      
      	// Act
      	// Call the method will return the savedPokemon 
        PokemonDto savedPokemon = pokemonService.createPokemon(pokemonDto);

      	// Assert
        Assertions.assertThat(savedPokemon).isNotNull();
    }

  /*
   	public PokemonResponse getAllPokemon(int pageNo, int pageSize) {
        Pageable pageable = PageRequest.of(pageNo, pageSize);
        Page<Pokemon> pokemons = pokemonRepository.findAll(pageable);
        List<Pokemon> listOfPokemon = pokemons.getContent();
        List<PokemonDto> content = listOfPokemon.stream().map(p -> mapToDto(p)).collect(Collectors.toList());

        PokemonResponse pokemonResponse = new PokemonResponse();
        pokemonResponse.setContent(content);
        pokemonResponse.setPageNo(pokemons.getNumber());
        pokemonResponse.setPageSize(pokemons.getSize());
        pokemonResponse.setTotalElements(pokemons.getTotalElements());
        pokemonResponse.setTotalPages(pokemons.getTotalPages());
        pokemonResponse.setLast(pokemons.isLast());

        return pokemonResponse;
    }
    */

   	@Test
    public void PokemonService_GetAllPokemon_ReturnsResponseDto() {
      	// Arrange
        Page<Pokemon> pokemons = Mockito.mock(Page.class);

        when(pokemonRepository.findAll(Mockito.any(Pageable.class))).thenReturn(pokemons);

      	// Act
        PokemonResponse savePokemon = pokemonService.getAllPokemon(1, 10);

      	// Assert
        Assertions.assertThat(savePokemon).isNotNull();
    }
  
    @Test
    public void PokemonService_FindById_ReturnPokemonDto() {
        int pokemonId = 1;
        Pokemon pokemon = Pokemon.builder()
          	.id(1).name("pikachu")
          	.type("electric")
          	.type("this is a type")
          	.build();
      
        when(pokemonRepository.findById(pokemonId)).thenReturn(Optional.ofNullable(pokemon));

        PokemonDto pokemonReturn = pokemonService.getPokemonById(pokemonId);

        Assertions.assertThat(pokemonReturn).isNotNull();
    }

   	@Test
    public void PokemonService_UpdatePokemon_ReturnPokemonDto() {
      	// Arrange
      	// Prepare the object for the return of repository
        Pokemon pokemon = Pokemon.builder()
          	.id(1)
          	.name("pikachu")
          	.type("electric")
          	.build();
      
      	// Prepare the dto for the parameter of the method
        PokemonDto pokemonDto = PokemonDto.builder()
          	.id(1)
          	.name("pikachu")
          	.type("electric")
          	.build();
     		
      	int pokemonId = 1;

      	// Mock the repository operations in this service method
        when(pokemonRepository.findById(pokemonId)).thenReturn(Optional.ofNullable(pokemon));
        when(pokemonRepository.save(pokemon)).thenReturn(pokemon);

      	// Act
      	// Call the service method
        PokemonDto updateReturn = pokemonService.updatePokemon(pokemonDto, pokemonId);

      	// Assert
        Assertions.assertThat(updateReturn).isNotNull();
    }

    @Test
    public void PokemonService_DeletePokemonById_ReturnVoid() {
      	// Arrange
        int pokemonId = 1;
        Pokemon pokemon = Pokemon.builder()
          	.id(1)
          	.name("pikachu")
          	.type("electric")	
          	.build();

      	// Mock the repository operations in this service method
      	/* 
      	In Mockito, when you want to set up the behavior for a void method on a mock object, such as delete in 
      	this case, you can use doNothing().when(mockObject).voidMethod(...). This syntax is specifically 
      	designed for void methods.
      	
      	By instructing the mock repository to do nothing, you can ensure that the test case does not perform 
      	any unwanted or unexpected side effects, such as actually deleting data from the repository.
      	*/
        when(pokemonRepository.findById(pokemonId)).thenReturn(Optional.ofNullable(pokemon));
      	doNothing().when(pokemonRepository).delete(pokemon);

      	// Act and assert
      	/*
      	assertAll()is a utility method provided by JUnit 5 that allows you to perform multiple assertions 	
        within a single test case, indicating that the deletePokemonIdmethod of thepokemonService should be 
        invoked.
        */
        assertAll(() -> pokemonService.deletePokemonId(pokemonId));
    }
}
```

`ReviewServiceTests.java`

```java
import com.pokemonreview.api.repository.PokemonRepository;
import com.pokemonreview.api.repository.ReviewRepository;
import com.pokemonreview.api.service.impl.ReviewServiceImpl;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
public class ReviewServiceTests {

    @Mock
    private ReviewRepository reviewRepository;
    @Mock
    private PokemonRepository pokemonRepository;
    @InjectMocks
    private ReviewServiceImpl reviewService;

    private Pokemon pokemon;
    private Review review;
    private ReviewDto reviewDto;
    private PokemonDto pokemonDto;

    @BeforeEach
    public void init() {
        pokemon = Pokemon.builder().name("pikachu").type("electric").build();
        pokemonDto = PokemonDto.builder().name("pickachu").type("electric").build();
        review = Review.builder().title("title").content("content").stars(5).build();
        reviewDto = ReviewDto.builder().title("review title").content("test content").stars(5).build();
    }

    @Test
    public void ReviewService_CreateReview_ReturnsReviewDto() {
        when(pokemonRepository.findById(pokemon.getId())).thenReturn(Optional.of(pokemon));
        when(reviewRepository.save(Mockito.any(Review.class))).thenReturn(review);

        ReviewDto savedReview = reviewService.createReview(pokemon.getId(), reviewDto);

        Assertions.assertThat(savedReview).isNotNull();
    }

    @Test
    public void ReviewService_GetReviewsByPokemonId_ReturnReviewDto() {
        int reviewId = 1;
        when(reviewRepository.findByPokemonId(reviewId)).thenReturn(Arrays.asList(review));

        List<ReviewDto> pokemonReturn = reviewService.getReviewsByPokemonId(reviewId);

        Assertions.assertThat(pokemonReturn).isNotNull();
    }

    @Test
    public void ReviewService_GetReviewById_ReturnReviewDto() {
        int reviewId = 1;
        int pokemonId = 1;

        review.setPokemon(pokemon);

        when(pokemonRepository.findById(pokemonId)).thenReturn(Optional.of(pokemon));
        when(reviewRepository.findById(reviewId)).thenReturn(Optional.of(review));

        ReviewDto reviewReturn = reviewService.getReviewById(reviewId, pokemonId);

        Assertions.assertThat(reviewReturn).isNotNull();
        Assertions.assertThat(reviewReturn).isNotNull();
    }

    @Test
    public void ReviewService_UpdatePokemon_ReturnReviewDto() {
        int pokemonId = 1;
        int reviewId = 1;
        pokemon.setReviews(Arrays.asList(review));
        review.setPokemon(pokemon);

        when(pokemonRepository.findById(pokemonId)).thenReturn(Optional.of(pokemon));
        when(reviewRepository.findById(reviewId)).thenReturn(Optional.of(review));
        when(reviewRepository.save(review)).thenReturn(review);

        ReviewDto updateReturn = reviewService.updateReview(pokemonId, reviewId, reviewDto);

        Assertions.assertThat(updateReturn).isNotNull();
    }

    @Test
    public void ReviewService_DeletePokemonById_ReturnVoid() {
        int pokemonId = 1;
        int reviewId = 1;

        pokemon.setReviews(Arrays.asList(review));
        review.setPokemon(pokemon);

        when(pokemonRepository.findById(pokemonId)).thenReturn(Optional.of(pokemon));
        when(reviewRepository.findById(reviewId)).thenReturn(Optional.of(review));

        assertAll(() -> reviewService.deleteReview(pokemonId, reviewId));
    }


}
```

### 3). Unit test for controller

```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PokemonResponse {
    private List<PokemonDto> content;
    private int pageNo;
    private int pageSize;
    private long totalElements;
    private int totalPages;
    private boolean last;
}
```



```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.pokemonreview.api.controllers.PokemonController;
import com.pokemonreview.api.dto.PokemonDto;
import com.pokemonreview.api.dto.PokemonResponse;
import com.pokemonreview.api.dto.ReviewDto;
import com.pokemonreview.api.models.Pokemon;
import com.pokemonreview.api.models.Review;
import com.pokemonreview.api.service.PokemonService;
import org.hamcrest.CoreMatchers;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.ArgumentMatchers;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.result.MockMvcResultHandlers;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

import java.util.Arrays;

import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.doNothing;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

@WebMvcTest(controllers = PokemonController.class)
@AutoConfigureMockMvc(addFilters = false)
@ExtendWith(MockitoExtension.class)
public class PokemonControllerTests {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private PokemonService pokemonService;

    @Autowired
    private ObjectMapper objectMapper;
    private Pokemon pokemon;
    private Review review;
    private ReviewDto reviewDto;
    private PokemonDto pokemonDto;

  	// Will be executed before each test case in the test class
    @BeforeEach
    public void init() {
        pokemon = Pokemon.builder().name("pikachu").type("electric").build();
        pokemonDto = PokemonDto.builder().name("pickachu").type("electric").build();
        review = Review.builder().title("title").content("content").stars(5).build();
        reviewDto = ReviewDto.builder().title("review title").content("test content").stars(5).build();
    }
  
  	/*
    @PostMapping("pokemon/create")
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<PokemonDto> createPokemon(@RequestBody PokemonDto pokemonDto) {
        return new ResponseEntity<>(pokemonService.createPokemon(pokemonDto), HttpStatus.CREATED);
    }
  	*/

    @Test
    public void PokemonController_CreatePokemon_ReturnCreated() throws Exception {
      	/*
      	ArgumentMatchers.any() 是 Mockito 的匹配器，匹配传递给 createPokemon 方法的任意参数。它用于设置方法的模拟行为，
      	不论传递了什么参数。
      	
      	invocation 表示方法的调用。在这种情况下，它代表了 createPokemon 方法的调用。
      	
      	invocation.getArgument(0) 获取传递给 createPokemon 方法的第一个参数。该方法使用索引 0，因为只有一个参数与 
      	ArgumentMatchers.any() 匹配。
      	*/
        given(pokemonService.createPokemon(ArgumentMatchers.any()))
          			.willAnswer((invocation -> invocation.getArgument(0)));

      	// Mock post http request
        ResultActions response = mockMvc.perform(
          			post("/api/pokemon/create")
                				.contentType(MediaType.APPLICATION_JSON)
                				.content(objectMapper.writeValueAsString(pokemonDto)));

      	/*
      	表示对响应的 JSON 数据进行断言，验证其 "name" 属性的值是否与 pokemonDto.getName() 返回的值相等。
      	$.name 是 JSON 路径表达式，用于指定要断言的属性路径。
      	*/
        response.andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.jsonPath("$.name", CoreMatchers.is(pokemonDto.getName())))
                .andExpect(MockMvcResultMatchers.jsonPath("$.type", CoreMatchers.is(pokemonDto.getType())));
    }
  
    /*
     @GetMapping("pokemon")
      public ResponseEntity<PokemonResponse> getPokemons(
              @RequestParam(value = "pageNo", defaultValue = "0", required = false) int pageNo,
              @RequestParam(value = "pageSize", defaultValue = "10", required = false) int pageSize
      ) {
          return new ResponseEntity<>(pokemonService.getAllPokemon(pageNo, pageSize), HttpStatus.OK);
      }
    */
  
 		@Test
    public void PokemonController_GetAllPokemon_ReturnResponseDto() throws Exception {
      	// Arrange
        PokemonResponse responseDto = PokemonResponse.builder()
          			.pageSize(10)
          			.last(true)
          			.pageNo(1)
          			.content(Arrays.asList(pokemonDto))
          			.build();
      
      	// Use Mockito to mock the service method pokemonService.getAllPokemon(),
      	// indicating this method will return the created responseDto as above
        when(pokemonService.getAllPokemon(1, 10)).thenReturn(responseDto);

      	// Act
      	// Use MockMvc to make a GET request to "/api/pokemon",
      	// 同时传递了 pageNo 和 pageSize 作为参数
        ResultActions response = mockMvc.perform(get("/api/pokemon")
                .contentType(MediaType.APPLICATION_JSON)
                .param("pageNo", "1")
                .param("pageSize", "10"));

      	// Assert
      	// Use MockMvcResultMatchers to verify the resonse status is 200,
      	// Use MockMvcResultMatchers.jsonPath to verify the response 'content' in JSON, 
      	// make sure it is the same as taht of responseDto in size.
        response.andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath(
                         "$.content.size()", CoreMatchers.is(responseDto.getContent().size())));
    }
  
  	/*  	
    @GetMapping("pokemon/{id}")
    public ResponseEntity<PokemonDto> pokemonDetail(@PathVariable int id) {
        return ResponseEntity.ok(pokemonService.getPokemonById(id));
    }
  	*/

    @Test
    public void PokemonController_PokemonDetail_ReturnPokemonDto() throws Exception {
      	// Arrange
        int pokemonId = 1;
      	// Use Mockito to mock the service method pokemonService.getPokemonById(),
      	// indicating this method will return the created pokemonDto object
        when(pokemonService.getPokemonById(pokemonId)).thenReturn(pokemonDto);

      	// Act
      	// 使用 MockMvc 框架发送一个GET请求到 "/api/pokemon/1" 路径
      	// 设置请求的消息体内容为将 pokemonDto 对象转换为 JSON 字符串后的结果
        ResultActions response = mockMvc.perform(get("/api/pokemon/1")
                .contentType(MediaType.APPLICATION_JSON);
                //.content(objectMapper.writeValueAsString(pokemonDto)));

        // 断言响应的状态码是否为200（表示成功）
        // 断言响应的 JSON 内容中的 "name" 字段是否与 pokemonDto 对象的名称属性相匹配
        // 断言响应的JSON内容中的"type"字段是否与pokemonDto对象的类型属性相匹配
        response.andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.name", CoreMatchers.is(pokemonDto.getName())))
                .andExpect(MockMvcResultMatchers.jsonPath("$.type", CoreMatchers.is(pokemonDto.getType())));
    }
  
  	@Test
    public void PokemonController_UpdatePokemon_ReturnPokemonDto() throws Exception {
      	// Arrange
        int pokemonId = 1;
        when(pokemonService.updatePokemon(pokemonDto, pokemonId)).thenReturn(pokemonDto);

      	// Act
        ResultActions response = mockMvc.perform(put("/api/pokemon/1/update")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(pokemonDto)));

      	// Arrange
        response.andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.name", CoreMatchers.is(pokemonDto.getName())))
                .andExpect(MockMvcResultMatchers.jsonPath("$.type", CoreMatchers.is(pokemonDto.getType())));
    }

    @Test
    public void PokemonController_DeletePokemon_ReturnString() throws Exception {
        int pokemonId = 1;
        doNothing().when(pokemonService).deletePokemonId(1);

        ResultActions response = mockMvc.perform(delete("/api/pokemon/1/delete")
                .contentType(MediaType.APPLICATION_JSON));

        response.andExpect(MockMvcResultMatchers.status().isOk());
    }

}

```

## 2. Integration Test

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.6.7</version>
  <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
  </dependency>
  <dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <exclusions>
      <exclusion>
        <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
      </exclusion>
    </exclusions>
  </dependency>
</dependencies>
```



`resources/application.properties`

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.h2.console.enabled=true
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true 
```

`TestH2Repository.java`

```java
import com.javatechie.crud.example.entity.Product;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TestH2Repository extends JpaRepository<Product,Integer> {
}
```

`SpringBootCrudApplicationTests.java`

```java
import com.javatechie.crud.example.entity.Product;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.jdbc.Sql;
import org.springframework.web.client.RestTemplate;

import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SpringBootCrudApplicationTests {

    @LocalServerPort
    private int port;

    private String baseUrl = "http://localhost";

    private static RestTemplate restTemplate;

    @Autowired
    private TestH2Repository h2Repository;

    @BeforeAll
    public static void init() {
        restTemplate = new RestTemplate();
    }

    @BeforeEach
    public void setUp() {
        baseUrl = baseUrl.concat(":").concat(port + "").concat("/products");
    }

    @Test
    public void testAddProduct() {
        Product product = new Product("headset", 2, 7999);
        Product response = restTemplate.postForObject(baseUrl, product, Product.class);
        assertEquals("headset", response.getName());
        assertEquals(1, h2Repository.findAll().size());
    }

    @Test
    @Sql(statements = "INSERT INTO PRODUCT_TBL (id,name, quantity, price) VALUES (4,'AC', 1, 34000)",  				
         executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
    @Sql(statements = "DELETE FROM PRODUCT_TBL WHERE name='AC'", 
         executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
    public void testGetProducts() {
      	// Act
        List<Product> products = restTemplate.getForObject(baseUrl, List.class);
      	// Assert
        assertEquals(1, products.size());
        assertEquals(1, h2Repository.findAll().size());
    }

    @Test
    @Sql(statements = "INSERT INTO PRODUCT_TBL (id, name, quantity, price) VALUES (1, 'CAR', 1, 334000)", 
         executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
    @Sql(statements = "DELETE FROM PRODUCT_TBL WHERE id=1", 
         executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
    public void testFindProductById() {
      	// Act
        Product product = restTemplate.getForObject(baseUrl + "/{id}", Product.class, 1);
      	// Assert
        assertAll(
                () -> assertNotNull(product),
                () -> assertEquals(1, product.getId()),
                () -> assertEquals("CAR", product.getName())
        );

    }

    @Test
    @Sql(statements = "INSERT INTO PRODUCT_TBL (id, name, quantity, price) VALUES (2, 'shoes', 1, 999)", 
         executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
    @Sql(statements = "DELETE FROM PRODUCT_TBL WHERE id=1", 
         executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
    public void testUpdateProduct() {
      	// Arrange
        Product product = new Product("shoes", 1, 1999);
      
      	// Act
        restTemplate.put(baseUrl+"/update/{id}", product, 2);
        Product productFromDB = h2Repository.findById(2).get();
      
      	// Assert
        assertAll(
                () -> assertNotNull(productFromDB),
                () -> assertEquals(1999, productFromDB.getPrice())
        );
    }

    @Test
    @Sql(statements = "INSERT INTO PRODUCT_TBL (id, name, quantity, price) VALUES (8, 'books', 5, 1499)", 
         executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
    public void testDeleteProduct() {
      	// Act
        int recordCount = h2Repository.findAll().size();
      
      	// Assert
        assertEquals(1, recordCount);
      
      	// Act
        restTemplate.delete(baseUrl + "/delete/{id}", 8);
      
      	// Assert
        assertEquals(0, h2Repository.findAll().size());
    }

}
```





# Kafka

## 1. What is Kafka?

Kafka is an open-source distributed streaming platform developed by the Apache Software Foundation. It is designed to handle high-throughput, fault-tolerant, and real-time data streaming. Kafka is widely used for building real-time data pipelines and streaming applications.

Key concepts in Kafka include:

1. **Topics**: A topic is a category or feed name to which messages are published. It represents a stream of records in Kafka. Topics are partitioned and replicated across multiple Kafka brokers for scalability and fault tolerance.
2. **Producers**: Producers are applications that publish data records to Kafka topics. They send messages to one or more topics of their choice.
3. **Consumers**: Consumers are applications that subscribe to one or more topics and consume messages published to those topics. They read messages in the order they were produced and can be part of a consumer group to parallelize processing.
4. **Brokers**: Brokers are the Kafka servers that manage the persistence and replication of the topic data. They handle the incoming messages from producers and serve the messages to consumers. There can be one or more brokers in the Kafka cluster.
5. **Partitions**: A topic can be divided into multiple partitions, which are ordered and immutable sequences of records. Each partition is hosted on a single broker, and multiple partitions allow for parallelism and scalability in data processing.
6. **Offsets**: Each message within a partition is assigned a unique identifier called an offset. Consumers keep track of the offsets to specify the position from which they want to read messages.
7. **Consumer Groups**: Consumers can be organized into consumer groups, where each group has one or more consumers. Each partition of a topic is consumed by only one consumer from each group, allowing for load balancing and parallel processing.

Kafka provides a high-throughput, fault-tolerant, and durable messaging system. It is known for its low-latency and high scalability, making it suitable for real-time data processing, event streaming, log aggregation, and various use cases in big data and distributed systems.

Developers can interact with Kafka using the Kafka API, which provides libraries in various programming languages, or by leveraging higher-level frameworks and tools built on top of Kafka, such as Apache Kafka Streams and Confluent Platform.

## 2. Kafka installation

- Open Source: Apache Kafka

- Commercial distribution: Confluent Kafka

- Managed Kafka Service: confluent & AWS

Many Companies still use the open source with proper infrastructure setup and they have dedicated developers or experts to handle any kind of inference related issues.

### 1). Start ZooKeeper server

 导航到 Kafka 解压目录的根目录。执行以下命令来启动 ZooKeeper

```sh
bin/zookeeper-server-start.sh config/zookeeper.properties
# 看到类似 "binding to port 0.0.0.0/0.0.0.0:2181" 的消息，表示 ZooKeeper 成功启动并监听在默认端口 2181
```

### 2). Start Kafka server / broker

 导航到 Kafka 解压目录的根目录。执行以下命令来启动 Kafka server：

```sh
bin/kafka-server-start.sh config/server.properties 
# 看到类似 "INFO [KafkaServer id=1] started" 的消息，表示 Kafka 服务器成功启动
```

### 3). Install Kafka using docker

kafka-docker-compose.yaml

```yaml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper
    container_name: zookeeper
    ports:
      - 2181:2181
    environment:
      - ZOOKEEPER_CLIENT_PORT=2181
      - ZOOKEEPER_TICK_TIME=2000

  kafka:
    image: confluentinc/cp-kafka
    container_name: kafka 
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_ADVERTISED_HOST_NAME=localhost
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092
      - KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1
    #volumes:
      #- ./data/kafka:/var/lib/kafka/data
```

```sh
docker-compose -f kafka-docker-compose.yaml up -d
docker images
docker ps
```

```sh
# Access to Kafka in one terminal
docker exec -it kafka /bin/sh
	cd opt/kafka_2.13-2.8.1/bin
	
	# Create a topic
	kafka-topics.sh --create \
	--zookeper zookeeper:2181
  --topic my_topic \
  --partitions 1 \
  --replication-factor 1
  
  # Produce some messages
  kafka-console-producer.sh --topic my_topic --bootstrap-server localhost:9092
  # Then input any message
```

```sh
# Access to Kafka in another terminal
docker exec -it kafka /bin/sh
	cd opt/kafka_2.13-2.8.1/bin
  
  # Comsume messages
  kafka-console-consumer.sh \
	--topic my_topic \
	--bootstrap-server localhost:9092 \
	--from-beginning
```

## 3. Kafka CLI

#### 1). Create Topic

```sh
bin/kafka-topics.sh --create \
--topic my_topic \
--bootstrap-server localhost:9092 \
--partitions 3 \
--replication-factor 1 \
--config max.message.bytes=1000000
```

#### 2). List all Kafka topics

```sh
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

#### 3). Describe Kafka topics

```sh
bin/kafka-topics.sh --describe --topic my_topic --bootstrap-server localhost:9092
```

#### 4). Consume Kafka topic by consumer

```sh
bin/kafka-console-consumer.sh \
--topic my_topic \
--bootstrap-server localhost:9092 \
--from-beginning
```

#### 5). Produce Kafka topic by producer

```sh
bin/kafka-console-producer.sh --topic my_topic --bootstrap-server localhost:9092
```

**使用生产者将** **CSV 文件中的内容发送到 Kafka 主题**

```
bin/kafka-console-producer.sh --topic my_topic --bootstrap-server localhost:9092 < input.csv
```

## 4. Run Apache Kafka without Zookeeper 

In traditional Kafka deployments, ZooKeeper is used as the metadata store for maintaining information about topics, partitions, consumer offsets, and other cluster-related data. ZooKeeper coordinates the Kafka brokers and ensures consistency across the cluster. However, ZooKeeper has certain limitations, such as single-point-of-failure and scalability challenges.

With KRaft, Kafka introduces an alternative to ZooKeeper for metadata storage and replication. KRaft uses an internal consensus algorithm to replicate and synchronize the metadata across multiple Kafka brokers, eliminating the dependency on ZooKeeper. This approach provides the following benefits:

1. **Fault Tolerance**: KRaft enables fault-tolerant metadata storage. Each Kafka broker participating in the KRaft cluster maintains a copy of the metadata, ensuring that metadata remains available even if some brokers fail.
2. **Consistency**: KRaft provides strong consistency guarantees for the metadata. All participating brokers agree on the order of metadata updates through a consensus protocol, ensuring that all brokers have the same view of the metadata.
3. **Simplicity**: KRaft simplifies the operational aspects of running Kafka clusters by removing the dependency on ZooKeeper. It reduces the complexity of managing ZooKeeper ensemble and eliminates the need for running and maintaining a separate ZooKeeper cluster.
4. **Scalability**: KRaft allows for dynamically adding or removing brokers from the cluster without impacting metadata availability. This provides better scalability and flexibility in handling growing Kafka deployments.

| Kafka version | State                                                        |
| ------------- | ------------------------------------------------------------ |
| 2.8           | KRaft early access                                           |
| 3.3           | Kraft production-ready                                       |
| 3.4           | Migration scripts early access                               |
| 3.5           | Migration scripts production-ready; use of ZooKeeper deprecated |
| 4.0           | ZooKeeper not supported                                      |

## 5. Spring Boot Kafka implementation

### 1). Create topic in CLI

```sh
bin/kafka-topics.sh --create \
--topic my_topic_1 \
--bootstrap-server localhost:9092 \
--partitions 3 \
--replication-factor 1 \
--config max.message.bytes=1000000
```

### 2). Producer

KafkaMessagePublisher.java

```java
package com.javatechie.service;

import com.javatechie.dto.Customer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Service;

import java.util.concurrent.CompletableFuture;

@Service
public class KafkaMessagePublisher {

    @Autowired
    private KafkaTemplate<String,Object> template;

    public void sendMessageToTopic(String message) {
      /*
        CompletableFuture<SendResult<String, Object>> future = 
          		template.send("my_topic_1", 3, null, message);
      
        future.whenComplete((result, ex) -> {
            if (ex == null) {
                System.out.println("Sent message: [" + message + "]");
                System.out.println("Offset: [" + result.getRecordMetadata().offset() + "]");
            } else {
                System.out.println("Failed to send message: [" + message + "]");
              	System.out.println("Reason: " + ex.getMessage());
            }
        });
       */
    		template.send("my_topic_1", 3, null, "Hi");
      	template.send("my_topic_1", 1, null, "Mario");
      	template.send("my_topic_1", 2, null, "Ninja");
      	template.send("my_topic_1", 2, null, "Developer");
    }

    public void sendEventsToTopic(Customer customer) {
        try {
            CompletableFuture<SendResult<String, Object>> future = 
              		template.send("my_topic_2", customer);
          
            future.whenComplete((result, ex) -> {
                if (ex == null) {
                    System.out.println("Sent event: [" + customer.toString() + "]"); 
                  	System.out.println("Offset: [" + result.getRecordMetadata().offset() + "]");
                } else {
                    System.out.println("Failed to send event: [" + customer.toString() + "]");
                  	System.out.println("Reason: " + ex.getMessage());
                }
            });
        } catch (Exception ex) {
            System.out.println("ERROR : "+ ex.getMessage());
        }
    }
}
```

PublisherController.java

```java
package com.javatechie.controller;

import com.javatechie.dto.Customer;
import com.javatechie.service.KafkaMessagePublisher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/producer-app")
public class PublisherController {

    @Autowired
    private KafkaMessagePublisher publisher;

    @GetMapping("/publish/{message}")
    public ResponseEntity<?> publishMessage(@PathVariable String message) {
        try {
            for (int i = 0; i <= 100000; i++) {
                publisher.sendMessageToTopic(message + " : " + i);
            }
            return ResponseEntity.ok("Message published successfully ..");
        } catch (Exception ex) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }
    }

    @PostMapping("/publish")
    public void sendEvents(@RequestBody Customer customer) {
        publisher.sendEventsToTopic(customer);
    }

}
```

### 3). Consumer

#### Consumer.java

```java
package com.javatechie.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Customer {

    private int id;
    private String name;
    private String email;
    private String contactNo;

}
```

#### KafkaMessageListener.java

```java
package com.javatechie.consumer;

import com.javatechie.dto.Customer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class KafkaMessageListener {

    Logger log = LoggerFactory.getLogger(KafkaMessageListener.class);

    @KafkaListener(
      	topics = "my_topic_2", 
      	groupId = "jt-group"
    )
    public void consumeEvents(Customer customer) {
        log.info("Consumer consume the event {} ", customer.toString());
    }

    @KafkaListener(
      	topics = "my_topic_1", 
      	groupId = "jt-group-new"
      	topicPartitions= {@TopicPartition(topic="my_topic_1", partitions={"2"})}
    )
    public void consume2(String message) {
        log.info("Consumer2 consume the message {} ", message);
    }

//    @KafkaListener(topics = "javatechie-demo1",groupId = "jt-group-new")
//    public void consume3(String message) {
//        log.info("consumer3 consume the message {} ", message);
//    }
//
//    @KafkaListener(topics = "javatechie-demo1",groupId = "jt-group-new")
//    public void consume4(String message) {
//        log.info("consumer4 consume the message {} ", message);
//    }
}
```

#### application.yaml

```yaml
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:9092
      group-id: jt-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring:
          json:
            trusted:
              packages : com.javatechie.dto

server:
  port: 9292
```

You can use Java class for the consumer configurations instead

KafkaConsumerConfig.java

```java
package com.javatechie.config;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.support.serializer.JsonDeserializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaConsumerConfig {

    @Bean
    public Map<String, Object> consumerConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        props.put(JsonDeserializer.TRUSTED_PACKAGES, "com.javatechie.dto");
        return props;
    }

    @Bean
    public ConsumerFactory<String,Object> consumerFactory() {
      	// 
        return new DefaultKafkaConsumerFactory<>(consumerConfig());
    }

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, Object>> 
      		kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, Object> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

# CQRS

## Demo 1

### product-command-service

#### controller

##### ProductCommandController.java

```java
package com.javatechie.controller;

import com.javatechie.dto.ProductEvent;
import com.javatechie.entity.Product;
import com.javatechie.service.ProductCommandService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/products")
public class ProductCommandController {

    @Autowired
    private ProductCommandService commandService;

    @PostMapping
    public Product createProduct(@RequestBody ProductEvent productEvent) {
        return commandService.createProduct(productEvent);
    }

    @PutMapping("/{id}")
    public Product updateProduct(@PathVariable long id, @RequestBody ProductEvent productEvent) {
        return commandService.updateProduct(id, productEvent);
    }
}
```

#### dto

##### ProductEvent.java

```java
package com.javatechie.dto;

import com.javatechie.entity.Product;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class ProductEvent {

    private String eventType;
    private Product product;
}
```

#### entity

##### Product.java

```java
package com.javatechie.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "PRODUCT_COMMAND")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String description;
    private double price;
}
```

#### repository

##### ProductRepository.java

```java
package com.javatechie.repository;

import com.javatechie.entity.Product;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductRepository extends JpaRepository<Product,Long> {
}
```

#### service

##### ProductService.java

```java
package com.javatechie.service;

import com.javatechie.dto.ProductEvent;
import com.javatechie.entity.Product;
import com.javatechie.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class ProductCommandService {

    @Autowired
    private ProductRepository repository;

    @Autowired
    private KafkaTemplate<String,Object> kafkaTemplate;

    public Product createProduct(ProductEvent productEvent) {
        Product product = repository.save(productEvent.getProduct());     
      	// Create event object
      	ProductEvent event = new ProductEvent("CreateProduct", product);
      	// Send event to the topic
        kafkaTemplate.send("product-event-topic", event);
        return product;
    }

    public Product updateProduct(long id, ProductEvent productEvent) {
      	// Update the found product
        Product existingProduct = repository.findById(id).get();
        Product newProduct = productEvent.getProduct();
        existingProduct.setName(newProduct.getName());
        existingProduct.setPrice(newProduct.getPrice());
        existingProduct.setDescription(newProduct.getDescription());
      
      	// Save the found product again
        Product product = repository.save(existingProduct);
        
      	// Create event to the topic
      	ProductEvent event = new ProductEvent("UpdateProduct", product);
      
      	// Send the event to the topic
        kafkaTemplate.send("product-event-topic", event);
        return product;
    }
}
```



#### resources

##### application.properties

```proper
spring.datasource.driver-class-name = com.mysql.cj.jdbc.Driver
spring.datasource.url = jdbc:mysql://localhost:3306/javatechie
spring.datasource.username = root
spring.datasource.password = Password
spring.jpa.show-sql = true
spring.jpa.hibernate.ddl-auto = update
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQLDialect
server.port = 9191
```

##### application.yaml

```yaml
spring:
  kafka:
    producer:
      bootstrap-servers: localhost:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer:  org.springframework.kafka.support.serializer.JsonSerializer
```

#### pom.xml

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
  </dependency>
  <dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

### product-query-service

#### controller

##### ProductQueryController.java

```java
package com.javatechie.controller;

import com.javatechie.dto.ProductEvent;
import com.javatechie.entity.Product;
import com.javatechie.service.ProductQueryService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RequestMapping("/products")
@RestController
public class ProductQueryController {

    @Autowired
    private ProductQueryService queryService;

    @GetMapping
    public List<Product> fetchAllProducts(){
        return queryService.getProducts();
    }

}
```

#### dto

##### ProductEvent.java

```java
package com.javatechie.dto;

import com.javatechie.entity.Product;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class ProductEvent {

    private String eventType;
    private Product product;
}
```

#### entity

##### Product.java

```java
package com.javatechie.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "PRODUCT_QUERY")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String description;
    private double price;
}
```

#### repository

##### ProductRepository.java

```java
package com.javatechie.repository;

import com.javatechie.entity.Product;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

#### service

##### ProductQueryService.java

```java
package com.javatechie.service;

import com.javatechie.dto.ProductEvent;
import com.javatechie.entity.Product;
import com.javatechie.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class ProductQueryService {

    @Autowired
    private ProductRepository repository;

    public List<Product> getProducts() {
        return repository.findAll();
    }

  	// Consume the message (event object) from the topic
    @KafkaListener(topics = "product-event-topic", groupId = "product-event-group")
    public void processProductEvents(ProductEvent productEvent) {
      	// Get the product object form the event
        Product product = productEvent.getProduct();
      
      	// If the event is "create", create the object
        if (productEvent.getEventType().equals("CreateProduct")) {
            repository.save(product);
        }
      
      	// If the event is "update", update the object
        if (productEvent.getEventType().equals("UpdateProduct")) {
            Product existingProduct = repository.findById(product.getId()).get();
            existingProduct.setName(product.getName());
            existingProduct.setPrice(product.getPrice());
            existingProduct.setDescription(product.getDescription());
            repository.save(existingProduct);
        }
    }
}
```

#### resources

##### application.properties

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url = jdbc:mysql://localhost:3306/javatechie
spring.datasource.username = root
spring.datasource.password = Password
spring.jpa.show-sql = true
spring.jpa.hibernate.ddl-auto = update
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQLDialect
server.port=9292
```

##### application.yaml

````yaml
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:9092
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring:
          json:
            trusted:
              packages: com.javatechie.dto
````

### pom.xml

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
  </dependency>
  <dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```









