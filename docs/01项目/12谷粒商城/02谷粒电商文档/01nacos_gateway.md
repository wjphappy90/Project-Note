

# 1. Nacos概述

官方地址：https://nacos.io

github地址：https://github.com/alibaba/nacos



面试题：微服务间远程交互的过程？

1. 先去注册中心查询服务的服务器地址
2. 调用方给对方发送http请求



## 1.1.   什么是 Nacos

Nacos 是阿里巴巴推出来的一个新开源项目，这是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。



## 1.2.   为什么是Nacos

常见的注册中心：

1. Eureka（原生，2.0遇到性能瓶颈，停止维护）
2. Zookeeper（支持，专业的独立产品。例如：dubbo）
3. Consul（原生，GO语言开发）
4. Nacos

相对于 Spring Cloud Eureka 来说，Nacos 更强大。

**Nacos = Spring Cloud Eureka + Spring Cloud Config**

Nacos 可以与 Spring, Spring Boot, Spring Cloud 集成，并能代替 Spring Cloud Eureka, Spring Cloud Config。

- 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-config 实现配置的动态变更。

- 通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-discovery 实现服务的注册与发现。



## 1.3.   可以干什么

Nacos是以服务为主要服务对象的中间件，Nacos支持所有主流的服务发现、配置和管理。

Nacos主要提供以下四大功能：

1. 服务发现和服务健康监测
2. 动态配置服务
3. 动态DNS服务
4. 服务及其元数据管理



# 2.  Nacos快速开始

结构图：

![echo service](assets/1542119181336-b6dc0fc1-ed46-43a7-9e5f-68c9ca344d60.png)

Nacos 依赖 Java 环境来运行。如果您是从代码开始构建并运行Nacos，还需要为此配置 Maven环境，请确保是在以下版本环境中安装使用:

1. 64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
2. 64 bit JDK 1.8+
3. Maven 3.2.x+



## 2.1.   下载及安装

你可以通过源码和发行包两种方式来获取 Nacos。

1. 从 Github 上下载源码方式

```bash
git clone https://github.com/alibaba/nacos.git
cd nacos/
# 切换到nacos根目录，执行下列命令
mvn -Prelease-nacos clean install -U  # 要下载很多依赖 会比较慢
ls -al distribution/target/ # 切换目录

# change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin
```

 ![1565845455301](assets/1565845455301.png)



2. 下载源码压缩包方式

您可以从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 `nacos-server-$version.zip` 包。

```bash
  unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
  cd nacos/bin
```



## 2.2.   启动nacos服务

**Linux/Unix/Mac**

启动命令(standalone代表着单机模式运行，非集群模式):

```
sh startup.sh -m standalone
```



**Windows**

启动命令：

```
cmd startup.cmd
```

或者双击startup.cmd运行文件。



访问：http://localhost:8848/nacos

用户名密码：nacos/nacos

![1565850935295](assets/1565850935295.png)



## 2.3.   注册中心

首先创建两个工程：nacos-provider、nacos-consumer

![1565852100033](assets/1565852100033.png)

![1565852168387](assets/1565852168387.png)

创建生产者：

![1565852279616](assets/1565852279616.png)

![1565852355522](assets/1565852355522.png)

![1565852477523](assets/1565852477523.png)

创建消费者：

![1565852617198](assets/1565852617198.png)

![1567250727696](assets/1567250727696.png)

![1565852777390](assets/1565852777390.png)

然后，一路下一步或者ok。效果如下：

![1565852929136](assets/1565852929136.png)



### 2.3.1.   生产者基本代码

 ![1565854507349](assets/1565854507349.png)

ProviderController代码如下：

```java
@RestController
public class ProviderController {

    @Value("${myName}")
    private String name;

    @GetMapping("hello")
    public String hello(){
        return "hello " + name;
    }
}
```

application.properties配置如下：

```properties
server.port=8070
# 自定义参数
myName=nacos
```



### 2.3.2.   生产者注册到nacos

生产者注册到nacos注册中心，步骤：

1. 添加依赖：spring-cloud-starter-alibaba-nacos-discovery及springCloud

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
           <version>0.2.2.RELEASE</version>
       </dependency>
   
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   
   <!-- SpringCloud的依赖 -->
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-dependencies</artifactId>
               <version>Greenwich.SR2</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

   **注意**：版本 0.2.x.RELEASE 对应的是 Spring Boot 2.x 版本，版本 0.1.x.RELEASE 对应的是 Spring Boot 1.x 版本。

2. 在 `application.properties` 中配置nacos服务地址和应用名

   ```properties
   server.port=8070
   spring.application.name=nacos-provider
   # nacos服务地址
   spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   # 自定义参数
   myName=nacos
   ```

3. 通过Spring Cloud原生注解 `@EnableDiscoveryClient` 开启服务注册发现功能

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class NacosProviderApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(NacosProviderApplication.class, args);
       }
   
   }
   ```



效果：

![1567251987453](assets/1567251987453.png)



### 2.3.3.   消费端基本代码

 ![1567253090953](assets/1567253090953.png)

ConsumerController代码：

```java
@RestController
public class ConsumerController {

    @GetMapping("hi")
    public String hi() {
        return "hi provider!";
    }
}
```

application.properties:

```properties
server.port=8080
```



### 2.3.4.   消费者注册到nacos

消费者注册到nacos跟生产者差不多，也分3步：

1. 添加依赖：同生产者

2. 在application.properties中配置nacos的服务名及服务地址：同生产者

3. 在引导类（NacosConsumerApplication.java）中添加@EnableDiscoveryClient注解：同生产者

   

效果：

![1567254178699](assets/1567254178699.png)



### 2.3.5.   使用feign调用服务

 ![1567254545445](assets/1567254545445.png)

以前我们使用feign来远程调用，这里也一样。引入feign的依赖：

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.2.2.RELEASE</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- SpringCloud的依赖 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

在NacosConsumerApplication类上添加@EnableFeignClients注解：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class NacosConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }

}
```

编写feignClient：

```java
@FeignClient("nacos-provider")
public interface ProviderFeign {

    @RequestMapping("hello")
    public String hello();
}
```

在Controller中使用feignClient：

```java
@RestController
public class ConsumerController {

    @Autowired
    private ProviderFeign providerFeign;

    @GetMapping("hi")
    public String hi() {
        return this.providerFeign.hello();
    }
}
```



测试访问：

![1567255443076](assets/1567255443076.png)



## 2.4.   配置中心

​		在系统开发过程中，开发者通常会将一些需要变更的参数、变量等从代码中分离出来独立管理，以独立的配置文件的形式存在。目的是让静态的系统工件或者交付物（如 WAR，JAR 包等）更好地和实际的物理运行环境进行适配。配置管理一般包含在系统部署的过程中，由系统管理员或者运维人员完成。配置变更是调整系统运行时的行为的有效手段。



如果微服务架构中没有使用统一配置中心时，所存在的问题：

- 配置文件分散在各个项目里，不方便维护
- 配置内容安全与权限
- 更新配置后，项目需要重启



nacos配置中心：**系统配置的集中管理**（编辑、存储、分发）、**动态更新不重启**、**回滚配置**（变更管理、历史版本管理、变更审计）等所有与配置相关的活动。



案例：改造生产者中的动态配置项，由配置中心统一管理。

![1567261900469](assets/1567261900469.png)



### 2.4.1.   nacos中创建统一配置

![1567262976093](assets/1567262976093.png)

![1567263334245](assets/1567263334245.png)

1. `dataId` 的完整格式如下：

   ```
   ${prefix}-${spring.profile.active}.${file-extension}
   ```

   - `prefix` 默认为所属工程配置`spring.application.name` 的值（即：nacos-provider），也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
   - `spring.profile.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
   - `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。**目前只支持 `properties` 和 `yaml` 类型。**

   总结：配置所属工程的spring.application.name的值 + "." + properties/yml

2. 配置内容：

   项目中易变的内容。例如：myName

**当前案例中，nacos-provider工程的spring.application.name=nacos-provider，没有配置spring.profiles.active。所以这里的dataId填写的是nacos-provider.properties**



### 2.4.2.   从配置中心读取配置

从配置中心读取配置，分以下3步：

1. 引入依赖

   在生产者中引入依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
       <version>0.2.2.RELEASE</version>
   </dependency>
   ```

   **注意**：版本 [0.2.x.RELEASE](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 2.x 版本，版本 [0.1.x.RELEASE](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-alibaba-nacos-config) 对应的是 Spring Boot 1.x 版本。

2. 在 **`bootstrap.properties`** 中配置 Nacos server 的地址和应用名

   ```properties
   spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   # 该配置影响统一配置中心中的dataId，之前已经配置过
   spring.application.name=nacos-provider
   ```

   说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

   在springboot工程中，bootstrap.properties的加载优先级更高。

3. 通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新：

   ```java
   @RestController
   @RefreshScope
   public class ProviderController {
   
       @Value("${myName}")
       private String name;
   
       @RequestMapping("hello")
       public String hello(){
           return "hello " + name;
       }
   }
   ```

   

### 2.4.3.   名称空间切换环境

在实际开发中，通常有多套不同的环境（默认只有public），那么这个时候可以根据指定的环境来创建不同的 namespce，例如，开发、测试和生产三个不同的环境，那么使用一套 nacos 集群可以分别建以下三个不同的 namespace。以此来实现多环境的隔离。

![1567300201637](assets/1567300201637.png)

切换到配置列表：

![1567300201637](assets/nacos-namespace.gif)

可以发现有四个名称空间：public（默认）以及我们自己添加的3个名称空间（prod、dev、test），可以点击查看每个名称空间下的配置文件，当然现在只有public下有一个配置。

默认情况下，项目会到public下找 `服务名.properties`文件。

接下来，在dev名称空间中也添加一个nacos-provider.properties配置。这时有两种方式：

1. 切换到dev名称空间，添加一个新的配置文件。缺点：每个环境都要重复配置类似的项目
2. **直接通过clone方式添加配置，并修改即可。推荐**

![1567301645987](assets/1567301645987.png)

![1567301689032](assets/1567301689032.png)

![1567301731019](assets/1567301731019.png)

点击编辑：修改配置内容，以作区分

![1567301779793](assets/1567301779793.png)



在服务提供方nacos-provider中切换命名空间，修改bootstrap.properties添加如下配置

```properties
spring.cloud.nacos.config.namespace=7fd7e137-21c4-4723-a042-d527149e63e0
```

namespace的值为：

![1567302053790](assets/1567302053790.png)



重启服务提供方服务，在浏览器中访问测试：

![1567302261494](assets/1567302261494.png)



### 2.4.4.   回滚配置（了解）

**目前版本该功能有bug，回滚之后配置消失。**

回滚配置只需要两步：

1. 查看历史版本
2. 回滚到某个历史版本

![1567303117328](assets/1567303117328.png)

![1567303168315](assets/1567303168315.png)



### 2.4.5.   加载多配置文件

偶尔情况下需要加载多个配置文件。假如现在dev名称空间下有三个配置文件：nacos-provider.properties、redis.properties、jdbc.properties。

![1567305611637](assets/1567305611637.png)



nacos-provider.properties默认加载，怎么加载另外两个配置文件？

在bootstrap.properties文件中添加如下配置：

```properties
spring.cloud.nacos.config.ext-config[0].data-id=redis.properties
# 开启动态刷新配置，否则配置文件修改，工程无法感知
spring.cloud.nacos.config.ext-config[0].refresh=true
spring.cloud.nacos.config.ext-config[1].data-id=jdbc.properties
spring.cloud.nacos.config.ext-config[1].refresh=true
```



修改ProviderController使用redis.properties和jdbc.properties配置文件中的参数：

```java
@RestController
@RefreshScope
public class ProviderController {

    @Value("${myName}")
    private String name;

    @Value("${jdbc.url}")
    private String jdbcUrl;

    @Value("${redis.url}")
    private String redisUrl;

    @RequestMapping("hello")
    public String hello(){
        return "hello " + name + ", redis-url=" + redisUrl + ", jdbc-url=" + jdbcUrl;
    }
}
```

测试效果：

![1567306296896](assets/1567306296896.png)



问题：

​	修改一下配置中心中redis.properties中的配置，不重启服务。能否加载配置信息

​	删掉`spring.cloud.nacos.config.ext-config[0].refresh=true`，再修改redis.properties中的配置试试



### 2.4.6.   配置的分组

在实际开发中，除了不同的环境外。不同的微服务或者业务功能，可能有不同的redis及mysql数据库。

区分不同的环境我们使用名称空间（namespace），区分不同的微服务或功能，使用分组（group）。

当然，你也可以反过来使用，名称空间和分组只是为了更好的区分配置，提供的两个维度而已。

新增一个redis.properties，所属分组为provider：

![1567307163038](assets/1567307163038.png)

现在开发环境中有两个redis.propertis配置文件，一个是默认分组（DEFAULT_GROUP），一个是provider组

默认情况下从DEFAULT_GROUP分组中读取redis.properties，如果要切换到provider分组下的redis.properties，需要添加如下配置：

```properties
# 指定分组
spring.cloud.nacos.config.ext-config[0].group=provider
```



缺点：

​		将来每个分组下会有太多的配置文件，不利于维护。

最佳实践：

​		**命名空间区分业务功能，分组区分环境。**



# 3. 服务网关Gateway

API 网关出现的原因是微服务架构的出现，不同的微服务一般会有不同的网络地址，而外部客户端可能需要调用多个服务的接口才能完成一个业务需求，如果让客户端直接与各个微服务通信，会有以下的问题：

1. 破坏了服务无状态特点。

   为了保证对外服务的安全性，我们需要实现对服务访问的权限控制，而开放服务的权限控制机制将会贯穿并污染整个开放服务的业务逻辑，这会带来的最直接问题是，破坏了服务集群中REST API无状态的特点。

     	从具体开发和测试的角度来说，在工作中除了要考虑实际的业务逻辑之外，还需要额外考虑对接口访问的控制处理。

2. 无法直接复用既有接口。

   当我们需要对一个即有的集群内访问接口，实现外部服务访问时，我们不得不通过在原有接口上增加校验逻辑，或增加一个代理调用来实现权限控制，无法直接复用原有的接口。

以上这些问题可以借助 API 网关解决。API 网关是介于客户端和服务器端之间的中间层，所有的外部请求都会先经过 API 网关这一层。也就是说，API 的实现方面更多的考虑业务逻辑，而安全、性能、监控可以交由 API 网关来做，这样既提高业务灵活性又不缺安全性，典型的架构图如图所示：

 ![1567310156296](assets/1567310156296.png)

## 3.1.   快速开始

创建网关module：

![1567310403942](assets/1567310403942.png)

![1567310497409](assets/1567310497409.png)

完成后：

 ![1567310618844](assets/1567310618844.png)



### 3.1.1.   引入依赖

已引入，如下。pom.xml中的依赖：

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
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
```



### 3.1.2.   编写路由规则

为了演示路由到不同服务，这里把消费者和生产者都配置在网关中

application.yml

```yml
server:
  port: 8090
spring:
  cloud:
    gateway:
      routes:
        - id: nacos-consumer
          uri: http://127.0.0.1:8080
          predicates:
            - Path=/hi
        - id: nacos-provider
          uri: http://127.0.0.1:8070
          predicates:
            - Path=/hello
```



### 3.1.3.   启动测试

通过网关路径访问消费者或者生产者。

![1567318777833](assets/1567318777833.png)



## 3.2.   路由规则详解

![1568906267000](assets/1568906267000.png)

基本概念：

- **Route**：路由网关的基本构建块。它由ID，目的URI，断言（Predicate）集合和过滤器（filter）集合组成。如果断言聚合为真，则匹配该路由。
- **Predicate**：这是一个 Java 8函数式断言。允许开发人员匹配来自HTTP请求的任何内容，例如请求头或参数。
- **过滤器**：可以在发送下游请求之前或之后修改请求和响应。

**路由根据断言进行匹配，匹配成功就会转发请求给URI，在转发请求之前或者之后可以添加过滤器。**



### 3.2.1.   断言工厂

Spring Cloud Gateway包含许多内置的Route Predicate工厂。所有这些断言都匹配HTTP请求的不同属性。多路由断言工厂通过`and`组合。

官方提供的路由工厂：

 ![1567318676724](assets/1567318676724.png)

这些断言工厂的配置方式，参照官方文档：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html

![1567318580097](assets/1567318580097.png)

![img](assets/20190502120932678.png)

这里重点掌握请求路径路由断言的配置方式：

![1567318968400](assets/1567318968400.png)

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://example.org
        predicates:
        - Path=/foo/{segment},/bar/{segment}
```

这个路由匹配以/foo或者/bar开头的路径，转发到http:example.org。例如 `/foo/1` or `/foo/bar` or `/bar/baz`.



### 3.2.2.   过滤器工厂

路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路径过滤器的范围限定为特定路由。Spring Cloud Gateway包含许多内置的GatewayFilter工厂。

 ![1567323105648](assets/1567323105648.png)

这些过滤器工厂的配置方式，同样参照官方文档：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html

 ![1567323164745](assets/1567323164745.png)

过滤器 有 20 多个 实现类,根据过滤器工厂的用途来划分，可以分为以下几种：Header、Parameter、Path、Body、Status、Session、Redirect、Retry、RateLimiter和Hystrix

![img](assets/20181202154250869.png)

这里重点掌握PrefixPath GatewayFilter Factory

![1567325859435](assets/1567325859435.png)

上面的配置中，所有的`/foo/**`开始的路径都会命中配置的router，并执行过滤器的逻辑，在本案例中配置了RewritePath过滤器工厂，此工厂将/foo/(?.*)重写为{segment}，然后转发到http://example.org。比如在网页上请求localhost:8090/foo/forezp，此时会将请求转发到http://example.org/forezp的页面



​		在开发中由于所有微服务的访问都要经过网关，为了区分不同的微服务，通常会在路径前加上一个标识，例如：访问服务提供方：`http://localhost:8090/provider/hello` ；访问服务消费方：`http://localhost:8090/consumer/hi`  如果不重写地址，直接转发的话，转发后的路径为：`http://localhost:8070/provider/hello`和`http://localhost:8080/consumer/hi`明显多了一个provider或者consumer，导致转发失败。

这时，我们就用上了路径重写，配置如下：

```yaml
server:
  port: 8090
spring:
  cloud:
    gateway:
      routes:
        - id: nacos-consumer
          uri: http://127.0.0.1:8080
          predicates:
            - Path=/consumer/**
          filters:
            - RewritePath=/consumer/(?<segment>.*),/$\{segment}
        - id: nacos-provider
          uri: http://127.0.0.1:8070
          predicates:
            - Path=/provider/**
          filters:
            - RewritePath=/provider/(?<segment>.*),/$\{segment}
```

**注意**：`Path=/consumer/**`及`Path=/provider/**`的变化

测试：

![1567325669010](assets/1567325669010.png)



## 3.3.   面向服务的路由

![1568908873341](assets/1568908873341.png)

如果要做到负载均衡，则必须把网关工程注册到nacos注册中心，然后通过服务名访问。

### 3.3.1.   把网关服务注册到nacos

1. 引入nacos的相关依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       <version>0.2.2.RELEASE</version>
   </dependency>
   
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
       <version>0.2.2.RELEASE</version>
   </dependency>
   ```

2. 配置nacos服务地址及服务名：

      ![1567316026329](assets/1567316026329.png)

   bootstrap.yml中的配置：

   ```yaml
   spring:
     application:
       name: gateway-demo
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848
   ```

   application.yml中的配置：

   ```yaml
   server:
     port: 8090
   spring:
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
       gateway:
         routes:
           - id: nacos-consumer
             uri: http://127.0.0.1:8080
             predicates:
               - Path=/consumer/**
             filters:
               - RewritePath=/consumer/(?<segment>.*),/$\{segment}
           - id: nacos-provider
             uri: http://127.0.0.1:8070
             predicates:
               - Path=/provider/**
             filters:
               - RewritePath=/provider/(?<segment>.*),/$\{segment}
   ```

3. 把网关注入到nacos

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class GatewayDemoApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(GatewayDemoApplication.class, args);
       }
   
   }
   ```



### 3.3.2.   修改配置，通过服务名路由

```yaml
server:
  port: 8090
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      routes:
        - id: nacos-consumer
          uri: lb://nacos-consumer
          predicates:
            - Path=/consumer/**
          filters:
            - RewritePath=/consumer/(?<segment>.*),/$\{segment}
        - id: nacos-provider
          uri: lb://nacos-provider
          predicates:
            - Path=/provider/**
          filters:
            - RewritePath=/provider/(?<segment>.*),/$\{segment}
```

语法：lb://服务名

lb：LoadBalance，代表负载均衡的方式

服务名取决于nacos的服务列表中的服务名

![1567322529258](assets/1567322529258.png)



## 3.4.   路由的java代码配置方式（了解）

参见官方文档：

![1567328302375](assets/1567328302375.png)

代码如下：

```
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder, ThrottleGatewayFilterFactory throttle) {
    return builder.routes()
            .route(r -> r.host("**.abc.org").and().path("/image/png")
                .filters(f ->
                        f.addResponseHeader("X-TestHeader", "foobar"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.path("/image/webp")
                .filters(f ->
                        f.addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://httpbin.org:80")
            )
            .route(r -> r.order(-1)
                .host("**.throttle.org").and().path("/get")
                .filters(f -> f.filter(throttle.apply(1,
                        1,
                        10,
                        TimeUnit.SECONDS)))
                .uri("http://httpbin.org:80")
            )
            .build();
}
```

