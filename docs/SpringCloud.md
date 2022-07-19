# SpringCloud



## 依赖管理

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>spring-cloud-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>cloud-eureka-server7001</module>
        <module>cloud-eureka-server-7002</module>
        <module>cloud-provider-payment8001</module>
        <module>cloud-provider-payment8002</module>
        <module>cloud-api-common</module>
        <module>cloud-consumer-order80</module>
    </modules>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba 2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
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
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```



Eureka服务注册中心依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
```



Eureka客户服务依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.example</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>

```





## 1、RestTemplate 服务调用



注册bean组件

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    //@LoadBalanced //要使用Eureka的服务功能必须加上注解@LoadBelanced(服务轮询), 否则要调用具体的服务实例
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        return restTemplateBuilder.rootUri("localhost:8001").build();
    }
}
```

使用RestTemplate(服务消费者)

```java
package com.qibria.controller;

import com.cloud.vo.CommonResult;
import com.cloud.vo.Payment;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class HelloController {

    private final RestTemplate restTemplate;

    @Value("${server.port}")
    private String port;

    public HelloController(RestTemplate template) {
        this.restTemplate = template;
    }

    @GetMapping("hello")
    public CommonResult hello(){
        return restTemplate.getForObject("hello", CommonResult.class);
    }

    @GetMapping("hello/get")
    public CommonResult<Payment> helloGet(Payment payment){
        return restTemplate.getForObject("/hello/get?id={id}&serial={serial}",
                CommonResult.class, 10, "hello get");
    }

    @GetMapping("hello/post")
    public CommonResult<Payment> helloPost(Payment payment){
        return restTemplate.postForObject("/hello/post", payment, CommonResult.class);
    }

}

```

服务提供者

```java
@RestController
public class HelloController {

    @Value("${server.port}")
    private String port;

    @GetMapping("hello")
    public CommonResult hello(){
       return new CommonResult<>(200, "hello: " + port);
    }

    @GetMapping("hello/get")
    public CommonResult<Payment> helloGet(Payment payment){
       return new CommonResult<>(200, "hello get: " + port, payment);
    }

    @PostMapping("hello/post")
    public CommonResult<Payment> helloPost(@RequestBody Payment payment){
       return new CommonResult<>(200, "hello post: " + port, payment);
    }
}
```





## 2、Eureka服务注册中心使用

### 单注册中心实例

注册中心(单个注册中心实例)

```java
@EnableEurekaServer //代表此服务是注册中心
@SpringBootApplication
public class EurekaServer7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001.class,args);
    }
}
```

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #集群指向其它eureka
      #defaultZone: http://127.0.0.1:7002/eureka/
      #单机就是7001自己
      defaultZone: http://${eureka.instance.hostname}:7001/eureka/
    #server:
    #关闭自我保护机制，保证不可用服务被及时踢除
      #enable-self-preservation: false
      #eviction-interval-timer-in-ms: 2000
```

注册应用实例(可以有多个相同实例，注意实例名不要相同)

```java
@SpringBootApplication
@EnableEurekaClient //代表注册客户服务实例
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class, args);
    }
}
```

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机版
      defaultZone: http://127.0.0.1:7001/eureka
      # 集群版
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8001
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    #lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    #lease-expiration-duration-in-seconds: 2

```



### 多注册中心实例

```java
@EnableEurekaServer //代表此服务是注册中心
@SpringBootApplication
public class EurekaServer7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001.class,args);
    }
}
```

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #集群指向其它eureka
      defaultZone: http://eureka7002.com:7002/eureka/
      #单机就是7001自己
      #defaultZone: http://${eureka.instance.hostname}:7001/eureka/
    #server:
    #关闭自我保护机制，保证不可用服务被及时踢除
      #enable-self-preservation: false
      #eviction-interval-timer-in-ms: 2000
```



```java
@EnableEurekaServer //代表此服务是注册中心
@SpringBootApplication
public class EurekaServer7002 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer7001.class,args);
    }
}
```

```yaml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #集群指向其它eureka
      defaultZone: http://eureka7001.com:7001/eureka/
      #单机就是7001自己
      #defaultZone: http://${eureka.instance.hostname}:7001/eureka/
    #server:
    #关闭自我保护机制，保证不可用服务被及时踢除
      #enable-self-preservation: false
      #eviction-interval-timer-in-ms: 2000
```



注册服务实例(每个实例都都注册进注册中心)

```java
@SpringBootApplication
@EnableEurekaClient //代表注册客户服务实例
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class, args);
    }
}
```



```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机版
      #defaultZone: http://127.0.0.1:7001/eureka
      # 集群版
      #defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8001
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    #lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    #lease-expiration-duration-in-seconds: 2

```

服务消费者

```java
@EnableEurekaClient
@SpringBootApplication
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

```yaml
server:
  port: 80

spring:
  application:
    name: cloud-order-sevice
#  zipkin:
#    base-url: http://localhost:9411
#  sleuth:
#    sampler:
#      probability: 1

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机
      #defaultZone: http://localhost:7001/eureka
      # 集群
      #defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka  # 集群版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

RestTemplate配置（重要）

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced // 必须使用此注解否则服务中兴无法找到服务实例
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        return restTemplateBuilder.rootUri("http://CLOUD-PAYMENT-SERVICE").build(); 
        //CLOUD-PAYMENT-SERVICE服务实例名, 使用注册中心后不需要访问具体的主机名, 直接使用注册服务是定义的应用名即可
    }
}
```

RestTemplate使用(结合服务注册实例)

```java
@RestController
public class HelloController {

    private final RestTemplate restTemplate;

    @Value("${server.port}")
    private String port;

    public HelloController(RestTemplate template) {
        this.restTemplate = template;
    }

    @GetMapping("hello")
    public CommonResult hello(){
        return restTemplate.getForObject("hello", CommonResult.class);
    }

    @GetMapping("hello/get")
    public CommonResult<Payment> helloGet(Payment payment){
        return restTemplate.getForObject("/hello/get?id={id}&serial={serial}",
                CommonResult.class, 10, "hello get");
    }

    @GetMapping("hello/post")
    public CommonResult<Payment> helloPost(Payment payment){
        return restTemplate.postForObject("/hello/post", payment, CommonResult.class);
    }

}
```



![image-20220718151701864](SpringCloud.assets/image-20220718151701864.png)



### Discovery服务发现

客户服务主动获取注册中心的信息

主启动类使用注解 **@EnableDiscoveryClient**

```java
@EnableEurekaClient //客户服务注册
@EnableDiscoveryClient //服务发现
@SpringBootApplication
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class, args);
    }
}
```



```java
@RestController
public class HelloController {
    
    @GetMapping(value = "/payment/discovery")
    public Object discovery() {
        
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("*****element: "+element);
        }

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getServiceId()+"\t"+
                     instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }
        
        return this.discoveryClient;
    }
    
}
```



### Eureka自我保护

客户服务不可用时, Eureka不会立刻清理此服务实例, 根据其保护机制注册中心依然保存此实例，如果在阈值时间内无法收到服务心跳, 注册中心才会将服务实例从注册中心移除, 以此防止注册中心误删因为网络波动而无法发送心跳的服务器



怎么禁止自我保护

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
    instance-id: eureka7001
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      #集群指向其它eureka
      defaultZone: http://127.0.0.1:7002/eureka/
      #单机就是7001自己
      #defaultZone: http://eureka7001.com:7001/eureka/
  server:
    enable-self-preservation: false # 关闭自我保护机制，保证不可用服务被及时踢除
    eviction-interval-timer-in-ms: 2000
```

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service

eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #单机版
      #defaultZone: http://127.0.0.1:7001/eureka
      # 集群版
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8001
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```



## 3、Zookeeper注册中心

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- SpringBoot整合zookeeper客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
        </dependency>
    </dependencies>
```



```java
@SpringBootApplication
@EnableDiscoveryClient
public class ZkOrderMain80{
    public static void main(String[] args) {
        SpringApplication.run(ZkOrderMain80.class, args);
    }
}
```

```yaml
server:
  port: 80


#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-consumer-order
  cloud:
    zookeeper:
      connect-string: 192.168.1.7:2181
```

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        return restTemplateBuilder.rootUri("http://cloud-provider-payment").build();
    }
}
```

```java
@RestController
public class HelloController {

    @Value("${server.port}")
    private String serverPort;

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping(value = "/payment/zk")
    public String paymentZk() {
        return restTemplate.getForObject("/payment/zk",String.class);
    }
}

```



```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8004 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8004.class, args);
    }
}
```

```yaml
server:
  port: 8004


#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 192.168.1.7:2181
```

```java
@RestController
public class HelloController {

    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/zk")
    public String paymentZk() {
        return "springCloud with zookeeper: "+serverPort+"\t"+ UUID.randomUUID().toString();
    }
}

```



## 4、consul

启动命令

```shell
./consul agent -dev -ui -node=consul-dev -client=0.0.0.0
```

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

```



```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsulOrder80Main {
    public static void main(String[] args) {
        SpringApplication.run(ConsulOrder80Main.class, args);
    }
}
```

```yaml
server:
  port: 80

spring:
  application:
    name: cloud-consumer-order
  ####consul注册中心地址
  cloud:
    consul:
      host: 192.168.1.7
      port: 8500
      discovery:
        hostname: 127.0.0.1
        service-name: ${spring.application.name}
        heartbeat:
          enabled: true
```

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        return restTemplateBuilder.rootUri("http://consul-provider-payment").build();
    }
}
```

```java
@RestController
public class HelloController {

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping(value = "/payment/consul")
    public String paymentZk() {
        return restTemplate.getForObject("/payment/consul",String.class);
    }
}
```



```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8006.class, args);
    }
}
```

```yaml
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  ####consul注册中心地址
  cloud:
    consul:
      host: 192.168.1.7
      port: 8500
      discovery:
        hostname: 127.0.0.1
        service-name: ${spring.application.name}
        heartbeat:
          enabled: true
```

```java
@RestController
public class HelloController {

    @Value("${server.port}")
    private String port;

    @RequestMapping(value = "payment/consul")
    public String paymentConsul() {
        return "hello consul---" + port +"---"+ UUID.randomUUID();
    }
}
```

























































