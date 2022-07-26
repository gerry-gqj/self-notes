# SpringCloud



![image-20220720163007385](SpringCloud.assets/image-20220720163007385.png)



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



## 5、Riboon负载均衡

Eurake默认使用Riboon对RestTemplate做了负载均衡(轮询规则)

自定义负载均衡规则

> 注意: 此包不能和主启动类在同一个包路径下

```java

@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule() {
        return new RandomRule();//定义为随机
    }
}
```



主启功类

```java
@EnableEurekaClient
@SpringBootApplication
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration= MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```



RestTemplate实例

```java
@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        return restTemplateBuilder.rootUri("http://CLOUD-PAYMENT-SERVICE").build();
    }
}
```





### 自定义实现负载均衡（轮询）

使用cps和回旋锁实现

顶层接口-只有一个方法(获取服务实例)

```java
public interface LoadBalancer{
    ServiceInstance instances(List<ServiceInstance> serviceInstances);
}
```

实现类 

```java
@Component
public class MyLB implements LoadBalancer{

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    public final int getAndIncrement(){
        int current;
        int next;
        do {
            current = this.atomicInteger.get();
            next = current >= 2147483647 ? 0 : current + 1;
        }while(!this.atomicInteger.compareAndSet(current,next));
        System.out.println("*****第几次访问，次数next: "+next);
        return next;
    }

    //负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标  ，每次服务重启动后rest接口计数从1开始。
    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstances){
        int index = getAndIncrement() % serviceInstances.size();

        return serviceInstances.get(index);
    }
}

```



```java
@Configuration
public class RestTemplateConfig {
    @Bean
    //@LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        return restTemplateBuilder.build();
    }
}
```



Controller

```java
@RestController
@Slf4j
public class OrderController{
    //public static final String PAYMENT_URL = "http://localhost:8001";

    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Resource
    private RestTemplate restTemplate;

    @Resource
    private LoadBalancer loadBalancer;
    
    @Resource
    private DiscoveryClient discoveryClient;

    @GetMapping(value = "/consumer/payment/lb")
    public String getPaymentLB()
    {
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");

        if(instances == null || instances.size() <= 0)
        {
            return null;
        }

        ServiceInstance serviceInstance = loadBalancer.instances(instances);
        URI uri = serviceInstance.getUri();

        return restTemplate.getForObject(uri+"/payment/lb",String.class);

    }
  
}
```



## 6、Openfeign服务调用(面向消费者)

依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--web-->
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
@EnableFeignClients
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class, args);
    }
}
```

```yaml
server:
  port: 80

spring:
  application:
    name: cloud-order-sevice

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
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka  # 集群版
      
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
      
```



```java
@Service
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {

    @GetMapping(value = "/hello")
    CommonResult<Payment> hello();

    @GetMapping(value = "/payment/feign/timeout")
    String paymentFeignTimeout();
}
```



```java
@RestController
public class HelloController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @Value("${server.port}")
    private String port;

    @GetMapping("hello")
    public CommonResult<Payment> hello(){
        return paymentFeignService.hello();
    }
}
```



开启日志

```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
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
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka  # 集群版

#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000

#开启feign日志
logging:
  level:
    com.qibria.cloud.service.PaymentFeignService: debug
```



## 7、Hystrix 服务熔断、降级、限流

```xml
    <dependencies>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--web-->
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



### 服务降级

服务端降级

```java
@SpringBootApplication
@EnableEurekaClient
//@EnableHystrix 包含 @EnableCircuitBreaker
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);
    }
}
```



```yaml
server:
  port: 8001


spring:
  application:
    name: cloud-provider-hystrix-payment

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
    instance-id: hystrix_payment8001
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    #lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    #lease-expiration-duration-in-seconds: 2

```



```java
package com.qibria.cloud.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixProperty;
import org.springframework.stereotype.Service;

import java.util.concurrent.TimeUnit;

@Service
public class PaymentService {

    /**
     * 正常访问，肯定OK
     * @param id
     * @return
     */
    public String paymentInfo_OK(Integer id) {
        return "线程池:  "+Thread.currentThread().getName()+"  paymentInfo_OK,id:  "+id+"\t"+"O(∩_∩)O哈哈~";
    }


    @HystrixCommand(
            fallbackMethod = "paymentInfo_TimeOut_Handler",
            commandProperties = {
                @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")})
    public String paymentInfo_TimeOut(Integer id) {

        //int age = 10/0;
        int timeNum = 5;
        try {
            TimeUnit.SECONDS.sleep(timeNum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池:  "+Thread.currentThread().getName()+
            "  paymentInfo_TimeOut,id:  "+id+"\t"+"O(∩_∩)O哈哈~"+"\t耗时(秒):"+timeNum;
    }


    public String paymentInfo_TimeOut_Handler(Integer id){
        return "线程池:  "+Thread.currentThread().getName()+"  8001系统繁忙或者运行报错，请稍后再试,id:  "+id+"\t"+"o(╥﹏╥)o";
    }

}
```



```java
@RestController
public class PaymentController {

    private final Logger log = LoggerFactory.getLogger(PaymentController.class);

    @Resource
    private PaymentService paymentService;

    @Value("${server.port}")
    private String port;


    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_OK(id);
        log.info("*****result: "+result);
        return result;
    }

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_TimeOut(id);
        log.info("*****result: "+result);
        return result;
    }

}
```



客户端降级

```java
@SpringBootApplication
@EnableFeignClients
@EnableHystrix //开启Hystrix
public class OrderHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderHystrixMain80.class,args);
    }
}

```

```yaml
server:
  port: 80


spring:
  application:
    name: cloud-comsumer-hystrix-order

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
    instance-id: hystrix_order80
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    #lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    #lease-expiration-duration-in-seconds: 2
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000

#开启hystrix服务
feign:
  hystrix:
    enabled: true
```



```java
@Component
//使用openfeign统一进行服务降级 PaymentFallbackService.class
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT" ,fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {

    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id);

}

```



实现类

```java
@Component
public class PaymentFallbackService implements PaymentHystrixService {

    @Override
    public String paymentInfo_OK(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_OK ,o(╥﹏╥)o";
    }

    @Override
    public String paymentInfo_TimeOut(Integer id) {
        return "-----PaymentFallbackService fall back-paymentInfo_TimeOut ,o(╥﹏╥)o";
    }
}
```



```java
@RestController
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod") 
//全局服务降级, 如果下面没有设置 @HystrixCommand的commandProperties属性, 就会对持有@HystrixCommand的方法进行服务降级
public class OrderHystrixController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_OK(@PathVariable("id") Integer id){
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

   @GetMapping("/consumer/payment/hystrix/timeout/{id}")
//    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
//            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
//    })
    @HystrixCommand
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        //int age = 10/0;  //模拟错误发生, 引发服务降级
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }

    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
        return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }

    // 下面是全局fallback方法
    public String payment_Global_FallbackMethod() {
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }

}
```



### 服务熔断



```java

@Service
public class PaymentService {

    //=====服务熔断=============================
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 失败率达到多少后跳闸
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {

        if(id < 0) {
            throw new RuntimeException("******id 不能负数");
        }
        String serialNumber = UUID.randomUUID().toString();

        return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
    }
 
    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
    }

}

```



```java
    //====服务熔断
    @GetMapping("/payment/circuit/{id}")
    public String paymentCircuitBreaker(@PathVariable("id") Integer id)
    {
        String result = paymentService.paymentCircuitBreaker(id);
        log.info("****result: "+result);
        return result;
    }
```



### 服务面板

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
```



## 8、Gateway服务网关

```xml
    <dependencies>
        <!--gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--eureka-client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
```



```java
@SpringBootApplication
public class GateWayMain9527 {

    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class, args);
    }
}
```



```yaml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          #route-id-prefix: payment_provider
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            #- Path=/payment/get/**         # 断言，路径相匹配的进行路由
            - Path=/**         # 断言，路径相匹配的进行路由

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            #- After=2020-02-21T15:51:37.485+08:00[Asia/Shanghai]
            #- Cookie=username,zzyy
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式

eureka:
  instance:
    hostname: cloud-gateway-service
    instance-id: cloud-gateway-service9527
    prefer-ip-address: true
  client:
    register-with-eureka: true
    fetchRegistry: true
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka  # 集群版

```



上面用配置方式配置网关路由断言映射，也可以使用编码方式配置

```java
@Configuration
public class GateWayConfig{
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder)    {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();

        routes.route("path_route_atguigu",
                r -> r.path("/guonei")
                        .uri("http://news.baidu.com/guonei")).build();

        return routes.build();
    }
}
```



### 网关过滤器

全局过滤器、独立过滤器、自定义过滤器



自定义过滤器

```java
package com.qibria.cloud.filter;

import com.cloud.vo.CommonResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.util.MultiValueMap;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.nio.charset.StandardCharsets;
import java.text.SimpleDateFormat;
import java.util.Date;

@Component
public class MyGateWayLogFilter implements GlobalFilter, Ordered {

    private final Logger logger = LoggerFactory.getLogger(MyGateWayLogFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        logger.info("--> start time: [{}]",new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));

        MultiValueMap<String, String> queryParams = exchange.getRequest().getQueryParams();
        queryParams.forEach((k, v) -> logger.info("key: {} || value: {}",k,v));

        String id = exchange.getRequest().getQueryParams().getFirst("id");
        if (id == null) {
            logger.error("请求参数id为空, 非法用户༼ つ ◕_◕ ༽つ");
            logger.info("<-- end time: [{}]",new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
            //自定义返回封装对象
            CommonResult<String> data = new CommonResult<>(400,"请求参数id为空, 非法用户༼ つ ◕_◕ ༽つ",null);
            ServerHttpResponse response = exchange.getResponse();
            DataBuffer wrap = response.bufferFactory().wrap(data.toString().getBytes());
            response.setStatusCode(HttpStatus.BAD_GATEWAY);
            return response.writeWith(Mono.just(wrap));
        }
        logger.info("<-- end time: [{}]",new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}

```



## 9、Config配置中心



**ssh配置(已弃用)**

如果报如下错误`reject HostKey: github.com`
原因：同一台电脑使用过两次公钥。
需要确认一下`ssh -T git@github.com`。回车yes

如果报的错误是`Auth fail`
原因：公钥不对。问题如果本地测试可以连接成功，springcloud连接失败，则是生成的问题。
原来的生成方式：`ssh-keygen -t rsa -C "yourname@your.com"`
改为：`ssh-keygen -m PEM -t rsa -b 4096 -C "yourname@your.com"`

**使用ed25515(github推荐使用的ssh连接方案，但是springcloud不支持)** 



**使用ecdrsa(最终解决方案)** 

>如果都用不了的可是使用https连接方案

use ecdsa:

Get the hostKey

```bash
ssh-keyscan -t ecdsa github.com
# github.com:22 SSH-2.0-babeld-4f04c79d
github.com ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEmKSENjQEezOmxkZMy7opKgwFB9nkt5YRrYMjNuG5N87uRgg6CLrbo5wAdT/y6v0mKV0U2w0WZ2YB/++Tpockg=
```

Generate a new key

```css
ssh-keygen -t ecdsa -b 256 -m PEM
```



ssh配置文件

~/.ssh/config

```txt
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_ecdsa
```





### 配置中心服务端配置

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
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
```



启动类

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344 {

    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```



配置文件

```yaml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: git@github.com:gerry-gqj/springcloud-config.git  #git仓库名字 git@github.com:zzyybs/springcloud-config.git
          search-paths:
            - springcloud-config # 配置文件路径
      label: master # 分支名

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
    instance-id: config-center3344
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    #lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    #lease-expiration-duration-in-seconds: 2
```



**服务端访问路径**

> ${hostname}:{server.port}/${branch}/${filename}

> 127.0.0.1:3344/master/config-dev.yaml



### 配置中心客户端配置

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
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
```



主启动类

```java
@EnableEurekaClient
@SpringBootApplication
public class ConfigClientMain3355 {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class, args);
    }
}
```



配置文件

boostrap.yaml

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://127.0.0.1:3344 #配置中心地址k
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka  # 集群版
  instance:
    prefer-ip-address: true
    instance-id: config-clent3355

```



controller

```java
@RestController
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/info")
    public String configInfo(){
        return configInfo;
    }
}
```



访问路径

> 127.0.0.1:3355/info



### 配置文件动态仓库刷新（有缺陷）

依赖于actuator监控中心post请求刷新,  不能自动刷新

**客户端修改**

配置文件添加

```yaml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



@RefreshScope 注解在Controller上

```java
@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/info")
    public String configInfo(){
        return configInfo;
    }
}
```



此时还不能自动刷新

通过actuator监控器实现手动刷新（可以不用重启服务器，可以说时另类的自动刷新手段，后续自动刷新工作由bus消息总线完成）

发送请求(post)

> POST http://127.0.0.1:3355/actuator/refresh

> GET http://127.0.0.1:3355/info

下一章使用消息总线解决定点刷新





##  10、Bus消息总线





配置中心服务端依赖

```xml
        <!--添加消息总线RabbitMQ支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
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
```

配置中心客户端依赖

```xml
         <!--添加消息总线RabbitMQ支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
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
```



### 全局通知

**服务端配置**

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344{
    
    public static void main(String[] args) {
            SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
```

```yaml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: git@github.com:gerry-gqj/springcloud-config.git  #git仓库名字 git@github.com:zzyybs/springcloud-config.git
          search-paths:
            - springcloud-config # 配置文件路径
      label: master # 分支名
  rabbitmq:
    host: 192.168.1.7
    port: 5672
    password: 1
    username: rabboy

##rabbitmq相关配置,暴露bus刷新配置的端点
management:
  endpoints: #暴露bus刷新配置的端点
    web:
      exposure:
        include: 'bus-refresh'

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
    instance-id: config-center3344
    #访问路径可以显示IP地址
    prefer-ip-address: true
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    #lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    #lease-expiration-duration-in-seconds: 2
```

客户端配置

```java
@EnableEurekaClient
@SpringBootApplication
public class ConfigClientMain3355 {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3355.class, args);
    }
}
```



bootstrap.yaml

```yaml
# bootstrap.yaml

server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://127.0.0.1:3344 #配置中心地址k
  rabbitmq:
    host: 192.168.1.7
    port: 5672
    username: rabboy
    password: 1
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka  # 集群版
  instance:
    prefer-ip-address: true
    instance-id: config-clent3355
```



```java
@RestController
@RefreshScope //用于动态刷新配置文件
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/info")
    public String configInfo(){
        return configInfo;
    }
}
```



```java
@EnableEurekaClient
@SpringBootApplication
public class ConfigClientMain3366 {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientMain3366.class,args);
    }
}
```



bootstrap.yaml

```yaml
server:
  port: 3366

spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: dev #读取后缀名称   上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
      uri: http://127.0.0.1:3344 #配置中心地址k
  rabbitmq:
    host: 192.168.1.7
    port: 5672
    username: rabboy
    password: 1
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka  # 集群版
  instance:
    prefer-ip-address: true
    instance-id: config-clent3366
```



```java
@RestController
@RefreshScope //用于动态刷新配置文件
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/info")
    public String configInfo(){
        return "config info:"+configInfo + "\tport: " + serverPort;
    }
}
```



```http
GET http://127.0.0.1:3355/info
```

```http
GET http://127.0.0.1:3366/info
```



服务端发送通知请求，刷新所有客户端配置

```shell
curl -X POST http://127.0.0.1:3344/actuator/bus-refresh
```



### 定点通知

服务端发送通知请求，刷新特定客户端配置

```shell
# curl -X POST http://127.0.0.1:3344/actuator/bus-refresh/${spring.application.name}:${server.port}
curl -X POST http://127.0.0.1:3344/actuator/bus-refresh/config-client:3355 # 只刷新3355服务
```



## 11、Stream消息驱动

消息驱动覆盖原有不同消息实现细节，对外暴露出同意的api调用



### 典型发布订阅模式搭建

#### 消息驱动服务提供者( 发布者)

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
```

```java
@SpringBootApplication
public class StreamMQMain8801 {

    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8801.class,args);
    }
}
```

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
  rabbitmq:
    host: 192.168.1.7
    port: 5672
    username: rabboy
    password: 1

eureka:
  client:
    register-with-eureka: true    #表示是否将自己注册进EurekaServer默认为true。
    fetchRegistry: true     #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka #集群版
  instance:
    instance-id: send-8801   # 服务实例名
    prefer-ip-address: true     #访问路径可以显示IP地址
    lease-renewal-interval-in-seconds: 5     #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-expiration-duration-in-seconds: 30 #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
```

```java
package com.qibria.cloud.service;

/**
 * 消息发送者
 */
public interface IMessageProvider {

    /**
     * 发送消息
     * @return str
     */
    String send();

}

```

```java

@EnableBinding(Source.class)
public class MessageProviderImpl implements IMessageProvider {

    private final Logger logger = LoggerFactory.getLogger(MessageProviderImpl.class);

    @Resource
    private MessageChannel output;

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        logger.info("-->serial:[{}] ",serial);
        return null;
    }
}
```

```java
@RestController
public class SendMessageController {

    @Resource
    private IMessageProvider messageProvider;

    @GetMapping("send")
    public String sendMessage(){
        return messageProvider.send();
    }
}
```



#### 消息驱动服务消费者（订阅者）

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

```java
@SpringBootApplication
public class StreamMQMain8802 {

    public static void main(String[] args) {
        SpringApplication.run(StreamMQMain8802.class,args);
    }
}
```

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  rabbitmq:
    host: 192.168.1.7
    port: 5672
    username: rabboy
    password: 1
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client:
    register-with-eureka: true  #表示是否将自己注册进EurekaServer默认为true。
    fetchRegistry: true         #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    service-url:
      defaultZone: http://127.0.0.1:7001/eureka,http://127.0.0.1:7002/eureka #集群版
  instance:
    instance-id: provider-8802
    prefer-ip-address: true     #访问路径可以显示IP地址
    lease-renewal-interval-in-seconds: 5     #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-expiration-duration-in-seconds: 30 #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
```

```java
@Service
@EnableBinding({Sink.class})
public class ReceiveMessageListener {

    private final Logger logger = LoggerFactory.getLogger(ReceiveMessageListener.class);

    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
        logger.info("server port:[{}]", serverPort);
        logger.info("message:[{}]", message);
        logger.info("消息内容 message payload: [{}]", message.getPayload());
    }
}
```



### stream消息重复消费和持久化问题

微服务默认将不同分组，只要完成消息订阅，一旦发布者发布消息，所有分组都会收到并且消费

如果微服务在不同分组中，全部服务都会去消费

在rabbitmq中，这就是典型exchange的fanout（扇出模式）

如果微服务是在相同分组中，那么只有一个服务去完成消费（类似于rabbitmq中的工作队列模式）

在rabbitmq中的队列(queue)对应stream中的分组(group)概念，在springcloud steam中相同分组的的消息指挥被一个服务器消费，不会出现所有服务器都消费同一个消息问题，在rabbitmq中就是说，有多个消费者都订阅了同一个消息队列queue，这时候当且只有一个消费者能收到队列中的消息（work queue---默认轮询方式，也可以手动设置消费方式）

完成分组只需要在配置文件设置一下分组属性就可以

```yaml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
      stream:
        binders: # 在此处配置要绑定的rabbitmq的服务信息；
          defaultRabbit: # 表示定义的名称，用于于binding整合
            type: rabbit # 消息组件类型
            environment: # 设置rabbitmq的相关的环境配置
              spring:
                rabbitmq:
                  host: localhost
                  port: 5672
                  username: guest
                  password: guest
        bindings: # 服务的整合处理
          input: # 这个名字是一个通道的名称
            destination: studyExchange # 表示要使用的Exchange名称定义
            content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
            binder: defaultRabbit # 设置要绑定的消息服务的具体设置
            group: group1111 #分组
```



消息持久化



## 12、zipkin与sleuth服务监控

在第二章的Eureka服务基础上进行扩展

添加依赖

```xml
        <!--包含了sleuth+zipkin-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

Eurake注册中心7001，7002不需要修改

修改80（服务消费者---调用方），8001（服务提供者---被调用方）

配置文件修改

新增zipkin（注意zipkin是否启动9411）

```yaml
spring:
    zipkin:
      base-url: http://localhost:9411
    sleuth:
      sampler:
        probability: 1 # 采样率
```



添加controller---用于服务之间的请求调用

80---消费者端

```java
    // ====================> zipkin+sleuth
    @GetMapping("/consumer/payment/zipkin")
    public String paymentZipkin(){
        String result = restTemplate.getForObject("http://localhost:8001"+"/payment/zipkin/", String.class);
        return result;
    }
```



8001---提供者端

```java
    @GetMapping("/payment/zipkin")
    public String paymentZipkin(){
        return "hi ,i'am paymentzipkin server fall back，welcome to atguigu，O(∩_∩)O哈哈~";
    }
```



重复几次请求

```http
GET http://127.0.0.1/consumer/payment/zipkin
```

然后访问面板---查看请求之间的关系调用

```http
GEH http://127.0.0.1:9411/zipkin/
```



## 13、springcloudAlibaba nacos



安装与启动

window10下启动方式

在nacos/bin目录下启动

启动命令 --- 单价版启动方案（默认是集群版）

```shell
startup.cmd -m standalone
```

然后访问

```shell
GET localhost:8848/nanos/
```

账号密码都是nocas



### SpringCloudAlibaba与nacos集成

#### 服务提供者

依赖

> **pom.xml**

```xml
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
```

> **PaymentMain9001.java**

```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9001 {

    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class, args);
    }
}
```

> **application.yaml**

```yaml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.7:8848 #配置Nacos地址

#端点暴露
management:
  endpoints:
    web:
      exposure:
        include: '*'

```

> **PaymentController.java**

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public CommonResult getPayment(@PathVariable("id") Integer id) {

        String message = "nacos registry, server port:" + serverPort + "\t-->id:" + id;
        CommonResult result = new CommonResult<>(200,message);
        return result;
    }
}

```



重复上面步骤新建一个一样的服务

```
启动类  PaymentMain9001.java
配置文件 application.yaml server.port:9002
```



#### 服务消费者

> **pom.xml**

```xml
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
```



> **主启动类 OrderNacosMain83.java**

```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderNacosMain83 {

    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
}
```



> **application.yaml**

```yaml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.7:8848


#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
restTemplate:
  service-url:
    nacos-user-service: http://nacos-payment-provider
```



> **RestTemplate.java**

```java
@Configuration
public class RestTemplateConfig {

    private final Logger logger = LoggerFactory.getLogger(RestTemplateConfig.class);

    @Value("${restTemplate.service-url.nacos-user-service}")
    private String nacosServer;

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder){
        logger.info("nacosServer:[{}]", nacosServer);
        return restTemplateBuilder.rootUri(nacosServer).build();
    }
}
```



> **OrderNacosController.java**

```java
@RestController
public class OrderNacosController {

    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {

        return restTemplate.getForObject("/payment/nacos/"+id,String.class);
    }
}
```



## 14、springcloudalibaba nacos config



### 基本配置

> **pom.xml**

```xml
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--nacos-discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```



> **NacosConfigClientMain3377.java**

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfigClientMain3377 {

    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
```



> **bootstrap.yaml**

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.7:8848 #Nacos服务注册中心地址
      config:
        server-addr: 192.168.1.7:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        #group: DEFAULT_GROUP
        #namespace: ab2faf1fe8da9df7ed774d883fcf13c1

# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml

# nacos-config-client-test.yaml   ----> config.info


```



> **application.yaml**

```yaml
spring:
  profiles:
    active: dev # 表示开发环境
    #active: test # 表示测试环境
    #active: info

```



> **application-dev.yaml**

```yaml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client


#配置中心
#config:
#  info: |
#    nacos config center.
#    path = nacos-config-client-dev.yaml.
#    version = 1.
```



> **application-test.yaml**

```yaml
server:
  port: 3378

spring:
  application:
    name: nacos-config-client


#配置中心
#config:
#  info: |
#    nacos config center.
#    path = nacos-config-client-test.yaml.
#    version = 1.
```



配置中心 config副本

> nacos-config-client-dev.yaml

```yaml
config:
  info: |
    nacos config center.
    path = nacos-config-client-dev.yaml.
    version = 1.
```



> nacos-config-client-test.yaml

```yaml
config:
  info: |
    nacos config center.
    path = nacos-config-client-test.yaml.
    version = 1.
```





> **ConfigClientController.java**

```java
@RestController
//@RefreshScope(proxyMode = ScopedProxyMode.DEFAULT)//支持Nacos的动态刷新功能。
@RefreshScope//支持Nacos的动态刷新功能。
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @Value("${spring.profiles.active}")
    private String profiles;


    @GetMapping("/config/info")
    public String getConfigInfo() {
        return "profiles active: "+ profiles +
                "\n\rconfig info: "+ configInfo;
    }
}
```

```http
GET http://localhost:3378/config/info

GET http://localhost:3377/config/info
```



### 分组与命名空间

#### - 分组

配置文件修改

> **bootsrap.yaml**

```yaml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  profiles:
    #active: dev # 表示开发环境
    #active: test # 表示测试环境
    active: info
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.1.7:8848 #Nacos服务注册中心地址
      config:
        server-addr: 192.168.1.7:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP 
        #group: TEST_GROUP
        #namespace: ab2faf1fe8da9df7ed774d883fcf13c1


# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# nacos-config-client-dev.yaml

# nacos-config-client-test.yaml   ----> config.info

```



**配置中心 nacos config**



作用: 使用分组进行服务环境的隔离与切换



添加分组与配置文件

> Group: DEV_GROUP

> Data Id: nacos-config-client-info.yaml

```yaml
config:
  info: |
    nacos config center.
    group = DEV_GROUP.
    path = nacos-config-client-info.yaml.
    version = 1.
```



> Group: TEST_GROUP

> Data Id: nacos-config-client-info.yaml

```yaml
config:
  info: |
    nacos config center.
    group = TEST_GROUP.
    path = nacos-config-client-info.yaml.
    version = 1.
```



测试链接

```http
GET http://localhost:3377/config/info
```



#### - 命名空间

