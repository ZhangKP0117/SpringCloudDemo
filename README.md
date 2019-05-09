SpringCloud

------

### 微服务

```java
简单地说，微服务是系统架构上的一种设计风格，它的主旨是将一个原本独立的系统拆分成多个小型服务，这些小型服务都在各自独立的进程中运行，服务之间通过基于HTTP的RESTFul API进行通信协作。被拆分成的每一个小型服务都围绕着系统中的某一项或一些耦合度较高的业务功能进行构建，且每个服务都维护自身的存储、业务并且独立部署，并且这些微服务可以使用不同语言开发。
```

### SpringCloud介绍

```java
Spring Cloud是一个基于Spring Boot实现的云应用开发工具，它为基于JVM的云应用开发中的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。

Spring Cloud包含了多个子项目（针对分布式系统中涉及的多个不同开源产品），比如：Spring Cloud Config、Spring Cloud Netflix、Spring Cloud CloudFoundry、Spring Cloud AWS、Spring Cloud Security、Spring Cloud Commons、Spring Cloud Zookeeper、Spring Cloud CLI等项目，常用子项目简介：

    1.Eureka：服务治理组件，包含服务注册中心、服务注册与发现机制实现 
    2.Hystrix：容错管理组件，实现断路器模式，对服务中出现的故障提供容错能力 
    3.Ribbon：客户端负载均衡的服务调用组件 
    4.Feign：基于Hystrix和Ribbon的声明式服务调用组件 
    5.Spring Cloud Config：配置管理工具，支持使用Git储存配置信息，使项目配置外部化存储； 
    6.Zuul：提供智能路由和访问过滤等功能 
    7.Spring Cloud Bus：事件、消息总线，用来传播集群中状态的变化，例如：动态刷新配置。
```

​        

#### 服务的注册与发现

##### 创建服务注册中心

- 创建一个基础的Spring Boot工程，并在pom.xml中引入相关依赖

  - 通过@EnableEurekaServer注解启动一个服务注册中心提供给其他应用进行对话
        

    ```java
    @EnableEurekaServer
      @SpringBootApplication
      public class EurekaServerApplication {
      
          public static void main(String[] args) {
              SpringApplication.run(EurekaServerApplication.class, args);
          }
      
      }
    ```

- 在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在application.properties中问增加如下配置：

  ```java
  server:
    port: 8761
  
  eureka:
    instance:
      hostname: localhost
    client:
      registerWithEureka: false
      fetchRegistry: false
      serviceUrl:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  
  
  #eureka.client.register-with-eureka ：表示是否将自己注册到Eureka Server，默认为true。
  #eureka.client.fetch-registry ：表示是否从Eureka Server获取注册信息，默认为true。
  #eureka.client.serviceUrl.defaultZone ：设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔。
  
  #通过eureka.client.registerWithEureka：false和fetchRegistry：false来表明自己是一个eureka server.
  
  ```

- 启动工程后，访问：http://localhost:8761

- ![image](https://github.com/zhangkp1006/SpringCloudDemo/img/1.png)

  ```java
  No application available 没有服务被发现 ……
       因为没有注册服务当然不可能有服务被发现了。
  ```

##### 创建一个服务提供者

当client向server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。

- 通过注解@EnableEurekaClient 表明自己是一个eurekaclient

```java
 @SpringBootApplication
 @EnableEurekaClient
 @RestController
 public class ServiceHiApplication {  
 	
     public static void main(String[] args) {
     	SpringApplication.run(ServiceHiApplication.class, args);
 }
 
 @Value("${server.port}")
 String port;
 @RequestMapping("/hi")
 public String home(@RequestParam String name) {
     return "hi "+name+",i am from port:" +port;
 }
 }
```

- 在配置文件中注明自己的服务注册中心的地址，application.yml配置文件如下：

  ```java
  server:
    port: 8762
  
  spring:
    application:
      name: service-hi
  
  eureka:
    client:
      serviceUrl:
        defaultZone: http://localhost:8761/eureka/
            
  #  需要指明spring.application.name,这在以后的服务与服务之间相互调用一般都是根据这个name 。
  ```

- 启动此项目，访问http://localhost:8761 即可发现该服务已注册，名称为service-hi 端口为8762、

  ![image](https://github.com/zhangkp1006/SpringCloudDemo/img/2.png)

- 访问 http://localhost:8762/hi?name=123

  ​    浏览器显示：hi 123,I am from port:8762

  

  

  

  



#### Ribbon 服务消费者

​        在微服务架构中，业务都会被拆分成一个独立的服务，服务与服务的通讯是基于http restful的。Spring cloud有两种服务调用方式，一种是ribbon+restTemplate，另一种是feign。

修改SERVICE-HI 端口号为8763并启动，访问http://localhost:8761 ：
    

```java
service-hi在eureka-server注册了2个实例
```

 ![image](https://github.com/zhangkp1006/SpringCloudDemo/img/4.png)
    

##### 建立一个服务消费者

- 新建工程 service-ribbon

- 在工程的配置文件指定服务的注册中心地址为http://localhost:8761/eureka/，程序名称为 service-ribbon，程序端口为8764。配置文件application.yml如下：

  ```java
   eureka:
        client:
          serviceUrl:
            defaultZone: http://localhost:8761/eureka/
      server:
        port: 8764
      spring:
        application:
      name: service-ribbon
  
  ```

- 启动类中,通过@EnableDiscoveryClient向服务中心注册；并且向程序的ioc注入一个bean: restTemplate;并通过@LoadBalanced注解表明这个restRemplate开启负载均衡的功能。

  ​    

  ```java
        @SpringBootApplication
        @EnableEurekaClient
        @EnableDiscoveryClient
        public class ServiceRibbonApplication {
        
            public static void main(String[] args) {
                SpringApplication.run( ServiceRibbonApplication.class, args );
            }
        
            @Bean
            @LoadBalanced
            RestTemplate restTemplate() {
                return new RestTemplate();
        }
      
      }
  ```

  

- 写一个测试类HelloService，通过之前注入ioc容器的restTemplate来消费service-hi服务的“/hi”接口，在这里我们直接用的程序名替代了具体的url地址，在ribbon中它会根据服务名来选择具体的服务实例，根据服务实例在请求的时候会用具体的url替换掉服务名，代码如下：

  ​    

  ```java
      @Service
        public class HelloService {
            @Autowired
            RestTemplate restTemplate;
        
            public String hiService(String name) {
                return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
            }
        }
  ```

    

- 写一个controller，在controller中用调用HelloService 的方法，代码如下：

  ​    

  ```java
  @RestController
      public class HelloControler {
      
          @Autowired
          HelloService helloService;
      
          @GetMapping(value = "/hi")
          public String hi(@RequestParam String name) {
              return helloService.hiService( name );
          }
      }
  ```

​    

- 在浏览器上多次访问http://localhost:8764/hi?name=123，浏览器交替显示：

  ​     

  ```java
     hi 123,i am from port:8762
     hi 123,i am from port:8763
    
  ```



  ​    这说明当我们通过调用restTemplate.getForObject(“http://SERVICE-HI/hi?name=”+name,String.class)方法时，已经做了负载均衡，访问了不同的端口的服务实例。



- 此时的架构：

  - 一个服务注册中心，eureka server,端口为8761

  - service-hi工程跑了两个实例，端口分别为8762,8763，分别向服务注册中心注册

  - sercvice-ribbon端口为8764,向服务注册中心注册

  - 当sercvice-ribbon通过restTemplate调用service-hi的hi接口时，因为用ribbon进行了负载均衡，会轮流的调用service-hi：8762和8763 两个端口的hi接口；

    ![image](https://github.com/zhangkp1006/SpringCloudDemo/img/5.png)



#### Feign 服务消费者

​	Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。

简而言之：

- Feign 采用的是基于接口的注解
- Feign 整合了ribbon，具有负载均衡的能力
- 整合了Hystrix，具有熔断的能力

##### 创建Feign服务

- 创建SpringBoot工程，引入相关依赖
- 配置文件application.yml文件，指定程序名为service-feign，端口号为8765，服务注册地址为http://localhost:8761/eureka/ 

```java
eureka:
    client:
      serviceUrl:
        defaultZone: http://localhost:8761/eureka/
  server:
    port: 8765
  spring:
    application:
      name: service-feign

  feign.hystrix.enabled: true


```

- 在程序的启动类ServiceFeignApplication ，加上@EnableFeignClients注解开启Feign的功能

```java
 @SpringBootApplication
  @EnableEurekaClient
  @EnableDiscoveryClient
  @EnableFeignClients
  public class ServiceFeignApplication {

  public static void main(String[] args) {
      SpringApplication.run( ServiceFeignApplication.class, args );
  }

  }
```

- 定义一个feign接口，通过@ FeignClient（“服务名”），来指定调用哪个服务。比如在代码中调用了service-hi服务的“/hi”接口

  ```java
  @FeignClient(value = "service-hi")
  public interface SchedualServiceHi {
      @RequestMapping(value = "/hi",method = RequestMethod.GET)
      String sayHiFromClientOne(@RequestParam(value = "name") String name);
  }
  ```

  

- controller层，对外暴露一个"/hi"的API接口，通过上面定义的Feign客户端SchedualServiceHi 来消费服务

 

```java
 @RestController
  public class HiController {

  @Autowired
  SchedualServiceHi schedualServiceHi;
  @RequestMapping(value = "/hi",method = RequestMethod.GET)
  public String sayHi(@RequestParam String name){
      return schedualServiceHi.sayHiFromClientOne(name);
  }

  }
```



- 启动程序，多次访问http://localhost:8765/hi?name=123,浏览器交替显示

 

```java
 hi 123,I am from port:8762

 hi 123,I am from port:8763
```



#### 断路器

##### Ribbon中使用断路器

- 添加 spring-cloud-starter-netflix-hystrix 依赖
- 启动类ServiceRibbonApplication 加@EnableHystrix注解开启Hystrix
- 改造HelloService类，在hiService方法上加上@HystrixCommand注解。该注解对该方法创建了熔断器的功能，并指定了fallbackMethod熔断方法，熔断方法直接返回了一个字符串，字符串为"hi,"+name+",sorry,error!"
- 启动：service-ribbon 工程，访问http://localhost:8764/hi?name=123  浏览器显示

```java
 hi 123,I am from port:8762
```

- 此时关闭 service-hi 工程，再访问http://localhost:8764/hi?name=123，浏览器会显示

```java
 hi ,123,sorry,error!
```

##### Feign中使用断路器

- Feign是自带断路器的，在D版本的Spring Cloud之后，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码:

  ```java
   feign.hystrix.enabled=true
  ```

- 基于service-feign工程进行改造，只需要在FeignClient的SchedualServiceHi接口的注解中加上fallback的指定类

  ```java
  @FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
  public interface SchedualServiceHi {
      @RequestMapping(value = "/hi",method = RequestMethod.GET)
      String sayHiFromClientOne(@RequestParam(value = "name") String name);
  }
  ```



- SchedualServiceHiHystric需要实现SchedualServiceHi 接口，并注入到Ioc容器中

```java
 @Component
  public class SchedualServiceHiHystric implements SchedualServiceHi {
      @Override
      public String sayHiFromClientOne(String name) {
          return "sorry "+name;
      }
  }
```

- 启动servcie-feign工程，浏览器打开http://localhost:8765/hi?name=123,此时service-hi工程没有启动，网页显示:

  ```ja
  sorry 123
  ```



- 启动service-hi ，浏览器打开http://localhost:8765/hi?name=123

```java
hi 123,I am from port:8762
```

#### 路由网关

##### Zuul

​    Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如／api/user转发到user服务，/api/shop转发到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。

- 创建 service-zuul 工程，添加相关依赖

- 在其入口applicaton类加上注解@EnableZuulProxy，开启zuul的功能：

  

  ```java
  @SpringBootApplication
  @EnableZuulProxy
  @EnableEurekaClient
  @EnableDiscoveryClient
  public class ServiceZuulApplication {
  
      public static void main(String[] args) {
          SpringApplication.run( ServiceZuulApplication.class, args );
      }
  }
  ```

- 配置文件application.yml：

  ​    

  ```java
  eureka:
      client:
        serviceUrl:
          defaultZone: http://localhost:8761/eureka/
    server:
      port: 8769
    spring:
      application:
        name: service-zuul
    zuul:
      routes:
        api-a:
          path: /api-a/**
          serviceId: service-ribbon
        api-b:
          path: /api-b/**
          serviceId: service-feign
  ```



 首先指定服务注册中心的地址为http://localhost:8761/eureka/，服务的端口为8769，服务名为service-zuul；以/api-a/ 开头的请求都转发给service-ribbon服务；以/api-b/开头的请求都转发给service-feign服务；



- 依次运行这五个工程;打开浏览器访问：http://localhost:8769/api-a/hi?name=123 ;

  浏览器显示：	      

  

```java
hi 123,I am from port:8762
```

访问：http://localhost:8769/api-b/hi?name=123 ;浏览器显示：



```java
hi 123,I am from port:8762
```

说明zuul路由起到了作用 

##### 服务过滤

在此工程基础上添加类 MyFilter
    

```java
    @Component
    public class MyFilter extends ZuulFilter {
    
        private static Logger log = LoggerFactory.getLogger(MyFilter.class);
    
        /**
         *
         *     filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
         *         pre：路由之前
         *         routing：路由之时
         *         post： 路由之后
         *         error：发送错误调用
         *
         */
        @Override
        public String filterType() {
            return "pre";
        }

    /**
     *
     * filterOrder：通过int值来定义过滤器的执行顺序，数值越小优先级越高。
     */
        @Override
        public int filterOrder() {
            return 0;
        }
    
    /**
     *
     * shouldFilter：返回一个boolean类型来判断该过滤器是否要执行。我们可以通过此方法来指定过滤器的有效范围。
     */
        @Override
        public boolean shouldFilter() {
            return true;
        }

   /**
     *
     * run：过滤器的具体逻辑。在该方法中，我们可以实现自定义的过滤逻辑，来确定是否要拦截当前的请求，不对其进行后续的路由，或是在请求路由返回结果之后，对处理结果做一些加工等。
     */
        @Override
        public Object run() {
            RequestContext ctx = RequestContext.getCurrentContext();
            HttpServletRequest request = ctx.getRequest();
            log.info(String.format("%s >>> %s", request.getMethod(), request.getRequestURL().toString()));
            Object accessToken = request.getParameter("token");
            if(accessToken == null) {
                log.warn("token is empty");
                ctx.setSendZuulResponse(false);
                ctx.setResponseStatusCode(401);
                try {
                    ctx.getResponse().getWriter().write("token is empty");
                }catch (Exception e){}
    
                return null;
            }
            log.info("ok");
            return null;
        }
    }
```

- 此时访问http://localhost:8769/api-a/hi?name=123 ；

  网页显示：![image](https://github.com/zhangkp1006/SpringCloudDemo/img/9.png)

  - 访问 http://localhost:8769/api-a/hi?name=123&token=111 ；

  网页显示：![image](https://github.com/zhangkp1006/SpringCloudDemo/img/10.png)

- 此时的架构

  ![1557196884743](https://github.com/zhangkp1006/SpringCloudDemo/img/1557196884743.png)

#### 分布式配置中心

- 创建项目 config-server 引入相关依赖

- 在程序的入口Application类加上@EnableConfigServer注解开启配置服务器的功能

  ```java
  @SpringBootApplication
  @EnableConfigServer
  public class ConfigServerApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(ConfigServerApplication.class, args);
      }
  
  }
  ```

- application.properties文件配置

  ```java
  spring.application.name=config-server
  server.port=8888
  
  
  spring.cloud.config.server.git.uri=https://github.com/forezp/SpringcloudConfig/
  #spring.cloud.config.server.git.uri=http://47.103.33.240:8888/pengpeng/SpringcloudConfig.git
  #公司gitlab仓库地址添加.git
  spring.cloud.config.server.git.searchPaths=respo
  spring.cloud.config.label=master
  spring.cloud.config.server.git.username=
  spring.cloud.config.server.git.password=
  
  
  #spring.cloud.config.server.git.uri：配置git仓库地址
  #spring.cloud.config.server.git.searchPaths：配置仓库路径
  #spring.cloud.config.label：配置仓库的分支
  #spring.cloud.config.server.git.username：访问git仓库的用户名
  #spring.cloud.config.server.git.password：访问git仓库的用户密码
  #如果Git仓库为公开仓库，可以不填写用户名和密码，如果是私有仓库需要填写，本例子是公开仓库，放心使用。
  #
  #远程仓库https://github.com/forezp/SpringcloudConfig/ 中有个文件config-client-dev.properties文件中有一个属性：
  #
  #username = zkp
  #password =123456
  
  ```

- 启动程序：访问http://localhost:8888/zkp/dev

  ![1557285756046](https://github.com/zhangkp1006/SpringCloudDemo/img/1557285756046.png)

- 创建config-client

- 配置文件：

  ```java
  spring.application.name=config-client
  spring.cloud.config.label=master
  spring.cloud.config.profile=dev
  spring.cloud.config.uri= http://localhost:8888/
  server.port=8881
  ```

- 程序的入口类，写一个API接口“／hi”，返回从配置中心读取的变量的值

  ```java
  @SpringBootApplication
  @RestController
  public class ConfigClientApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(ConfigClientApplication.class, args);
      }
  
  
      @Value("${username}")
      String username;
  
      @Value("${password}")
      String password;
  
      @RequestMapping(value = "/hi")
      public String hi() {
          return "hi!"+username + "/
              "+ password;
      }
  
  }
  ```

- 访问：http://localhost:8881/hi，网页显示：

  ![1557388100317](https://github.com/zhangkp1006/SpringCloudDemo/img/1557388100317.png)

  