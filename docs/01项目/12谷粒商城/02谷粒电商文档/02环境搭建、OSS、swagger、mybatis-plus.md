

# 1. 谷粒商城介绍

## 1.1.   项目分类

主要从需求方、盈利模式、技术侧重点这三个方面来看它们的不同

### 1.1.1.   传统项目

各种企业里面用的管理系统（ERP、HR、OA、CRM、物流管理系统。。。。。。。）

- 需求方：公司、企业内部
- 盈利模式：项目本身卖钱
- 技术侧重点：业务功能



### 1.1.2.   互联网项目

门户网站、电商网站：baidu.com、qq.com、taobao.com、jd.com  ...... 

- 需求方：广大用户群体
- 盈利模式：虚拟币、增值服务、广告收益......
- 技术侧重点：网站性能、业务功能

而我们今天要聊的就是互联网项目中的重要角色：电商



## 1.2.   为什么我们要讲电商？

因为就互联网平台来说，电商网站有很多典型的特征：

- 访问量大
- 数据量大
- 有一定的业务复杂性
- 涉及支付 考虑一定安全性



## 1.3. 电商行业的发展

### 1.3.1.   钱景

近年来，中国的电子商务快速发展，交易额连创新高，电子商务在各领域的应用不断拓展和深化、相关服务业蓬勃发展、支撑体系不断健全完善、创新的动力和能力不断增强。电子商务正在与实体经济深度融合，进入规模性发展阶段，对经济社会生活的影响不断增大，正成为我国经济发展的新引擎。

中国电子商务研究中心数据显示，截止到 2012 年底，中国电子商务市场交易规模达 7.85万亿人民币，同比增长 30.83%。其中，B2B 电子商务交易额达 6.25 万亿，同比增长 27%。而 2011 年全年，中国电子商务市场交易额达 6 万亿人民币，同比增长 33%，占 GDP 比重上升到 13%；2012 年，电子商务占 GDP 的比重已经高达 15%。

 ![](assets/1525686041466.png)



### 1.3.2.   数据

![img](assets/timg.jfif)

来看看双十一的成交数据：

![img](assets/timg-1565828244649.jfif)

![1525686160411](assets/1525686160411.png)

2016双11开场30分钟，创造**每秒交易峰值17.5万笔**，**每秒**支付峰值**12万笔**的新纪录。菜鸟单日物流订单量超过**4.67亿**，创历史新高。

2017阿里双11交易创建峰值 32.5 万笔／秒，支付峰值达到每秒25.6万笔。支付宝每秒处理4200万次请求。

2018年天猫双11**每秒订单创建峰值49.1万笔**。全天**实时计算处理峰值达到每秒17.18亿条**，强大技术能力保障了双11购物如丝般顺滑。消费者手指轻点下单的这一秒钟里，光是计算环节，就包括至少7个系统的运转。这对阿里巴巴来说，意味着单位时间内全球最大的多维度计算量。提交订单的环节，一个页面里，背后包括商品、价格、营销、资金、零库存、限购数、物流等，这些都在考核后台的计算能力。目前，在全球范围内都没有其他任何一个商业机构能在1秒钟内处理这么大的交易计算量。



## 1.4.   常见电商模式

电商行业的一些常见模式：

- B2C：商家对个人，如：亚马逊、当当等
- C2C平台：个人对个人，如：闲鱼、拍拍网、ebay
- B2B平台：商家对商家，如：阿里巴巴、八方资源网等
- O2O：线上和线下结合，如：饿了么、电影票、团购等
- P2P：在线金融，贷款，如：网贷之家、人人聚财等。
- B2C平台：天猫、京东、一号店等



# 2. 项目介绍

## 2.1.   项目结构说明

![1565830474055](assets/1565830474055.png)

核心是microservice这块儿，它分两类微服务：

- 后台CRUD微服务

  1. pms：商品管理系统（product） 8081

  2. ums：用户管理系统（user）8082

  3. wms：仓库管理系统（warehouse） 8083

  4. oms：订单管理系统（order） 8084

  5. sms：营销管理系统（sale） 8085

     。。。。

- 前台数据接口微服务

  1. cart：购物车微服务
  2. order：订单微服务
  3. search：搜索微服务
  4. item：商品详情微服务
  5. member：会员微服务

第三方微服务：物流、短信、支付、云存储。。。。

数据层：mysql（分库分表中间件：ShardingSphere）、redis、elasticsearch、activemq/rabbitmq、阿里OSS

后台管理前端：gmall-admin-vue（1000）

前台门户：gmall-shop （2000）

网关：gmall-gateway（8888）



## 2.2.   项目规范

1. 权限规则：

   ![1567344219856](assets/1567344219856.png)

   例如：pms:spu:list；商品列表获取

2. 路径访问规则

    ![1567344387621](assets/1567344387621.png)

3. 表

   1）表要有备注

   2）每一个列名要备注，备注格式   列名[列其他备注]；最终[]外面的会被作为swagger文档的entity描述

4. 项目创建项目名，包名，模块名等

    ![1567346430107](assets/1567346430107.png)



# 3. 搭建微服务

## 3.1.   数据准备

把课前资料中的数据文件导入数据库：

![1567355328875](assets/1567355328875.png)

导入数据库之后：

![1567333267252](assets/1567333267252.png)



## 3.2.   使用github创建仓库

![1567334008957](assets/1567334008957.png)

![1567339772175](assets/1567339772175.png)

![1567339859043](assets/1567339859043.png)

克隆到本地：

![1567339951241](assets/1567339951241.png)



开源协议参考：

![1567337340281](assets/1567337340281.png)



## 3.3.   项目初始创建

把课前资料/基础工程中的文件

![1567340429812](assets/1567340429812.png)

copy到 clone下来的gmall工程中

![1567340490252](assets/1567340490252.png)



接下来在idea中导入gmall工程：

 ![1567339273164](assets/1567339273164.png)

 ![1567340194859](assets/1567340194859.png)

点击“OK”，一路下一步：



在idea的Terminal窗口，查看git状态：

![1567340690436](assets/1567340690436.png)

把.idea/添加到忽略列表：

![1567341375564](assets/1567341375564.png)

![1567341727095](assets/1567341727095.png)

推送到远程仓库：`git push origin master`

![1567343213635](assets/1567343213635.png)

效果：

![1567343258305](assets/1567343258305.png)



## 3.4.   项目模块说明

gmall-core核心模块

1. 包含了通用依赖：

   nacos注册中心、配置中心

   mybatis-plus

   mysql驱动

   swagger

   lombok

2. 通用工具类：

   请求参数包装类

   通用的响应数据

   日期工具类

   sql过滤工具类

3. 自定义异常



springboot的常用依赖有：

- data-redis

- web

- security

springCloud的常用依赖有：

- hystrix

- openfeign

- alibaba-nacos-discovery（已经在core中引入）

- alibaba-nacos-config（已经在core中引入）



## 3.5.   创建项目其他模块

任何项目都要依赖核心工程。

### 3.5.1.   创建商品管理模块

![1567345499851](assets/1567345499851.png)

![1567345461335](assets/1567345461335.png)

![1567345986612](assets/1567345986612.png)

![1567345857370](assets/1567345857370.png)

创建完成：

 ![1567346540027](assets/1567346540027.png)

需要手动导入核心包的依赖，在pom.xml中导入核心包：

```xml
        <dependency>
            <groupId>com.atguigu</groupId>
            <artifactId>gmall-core</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```



### 3.5.2.   创建用户管理模块

![1567346794978](assets/1567346794978.png)

其他项，参考商品管理模块。



### 3.5.3.   创建订单管理模块

![1567347502480](assets/1567347502480.png)

其他项参考商品管理模块



### 3.5.4.   创建营销管理模块

![1567347664944](assets/1567347664944.png)



### 3.5.5.   创建仓库管理模块

![1567347872743](assets/1567347872743.png)



### 3.5.6.   完整工程

 ![1567348829577](assets/1567348829577.png)

接下来把这些新模块push到github。

先添加忽略：

![1567348936021](assets/1567348936021.png)

```shell
git add .
git commit -m "完成后台管理系统的创建"
git push origin master
```

![1567349112626](assets/1567349112626.png)



## 3.6.   逆向工程生成代码

课前资料/逆向工程：

![1567352524638](assets/1567352524638.png)

copy到gmall项目下：

![1567352574800](assets/1567352574800.png)

回到idea，把逆向工程作为maven工程导入idea

 ![1567352736557](assets/1567352736557.png)

 ![1567352802337](assets/1567352802337.png)



下面以生成pms的代码为例，其他模块操作类似！！！！！！！



### 3.6.1.   修改配置

修改application.yml，把数据库及其连接信息改成自己的：

![1567353516902](assets/1567353516902.png)

修改generator.properties：

![1567353654521](assets/1567353654521.png)



### 3.6.2.   启动逆向工程

启动RenrenApplication：

 ![1567353884488](assets/1567353884488.png)

启动成功，监听端口号为80。浏览器访问

![1567354272950](assets/1567354272950.png)

生成一个压缩包，被下载下来：

![1567354334616](assets/1567354334616.png)

即找到压缩包。



### 3.6.3.   把文件copy到对应工程

1. 打开压缩包，压缩包/main/java下的com目录，copy到gmall-pms模块的src/main/java：

   ![1567354454625](assets/1567354454625.png)

   效果如下：

    ![1567354640217](assets/1567354640217.png)

2. 把压缩包/main/resources下的mapper目录，copy到gmall-pms模块的src/java/resources目录下：

   ![1567354984421](assets/1567354984421.png)

   如下：

    ![1567355365129](assets/1567355365129.png)



## 3.7.   各个模块工程的配置

后台管理模块，每个工程的配置方式基本一样。下面还是以gmall-pms工程为例：

### 3.7.1.   工程参数配置

application.yml配置文件：

```yaml
server:
  port: 8081
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://172.16.116.100:3306/guli_pms?useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: root
  redis:
    host: 172.16.116.100
mybatis-plus:
  mapper-locations: classpath:mapper/pms/*.xml  # 读取映射文件
  global-config:
    db-config:
      id-type: auto  # 默认使用数据库自增
```



bootstrap.yml配置文件：

```yaml
spring:
  application:
    name: pms-service
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
```



### 3.7.2.   在引导类上添加注解

在gmall-pms工程的引导类（GmallPmsApplication）上添加注解：

```java
@SpringBootApplication
@MapperScan(basePackages = "com.atguigu.gmall.pms.dao")
@EnableSwagger2
public class GmallPmsApplication {

    public static void main(String[] args) {
        SpringApplication.run(GmallPmsApplication.class, args);
    }

}
```



**使用mybatis-plus分两步：**

1. **配置文件中引入映射文件**
2. **在引导类上添加@MapperScan扫描所有mapper/dao接口**
3. **编写mapper接口实现BaseMapper<T>即可**



### 3.7.3.   配置到nacos

创建名称空间：

![1567391643890](assets/1567391643890.png)

给pms添加配置：

![1567391685012](assets/1567391685012.png)

修改bootstrap.yml配置：

```yaml
spring:
  application:
    name: pms-service
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        namespace: 3aeaadb8-203f-44a1-a384-af429c486c0a
        group: dev
        file-extension: yml  # 默认找服务名.properties文件
```

在GmallPmsApplication类上添加@RefreshScope注解：

![1567393477982](assets/1567393477982.png)



## 3.8.   添加security配置放行所有请求

运行gmall-pms并访问：http://localhost:8081

![1567359760668](assets/1567359760668.png)

访问成功，但是需要登录。这是因为集成了spring-security，需要放行所有请求。

添加一个配置类：

 ![1567361258408](assets/1567361258408.png)

内容如下：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/**").permitAll();
        // 禁用掉csrf
        http.csrf().disable();
    }
}
```



测试访问：http://localhost:8081/swagger-ui.html

![1567362190525](assets/1567362190525.png)

拿品牌功能来测试：

![1567362366913](assets/1567362366913.png)

新增后：

![1567362394266](assets/1567362394266.png)



再来一个查询看看：

![1567362434386](assets/1567362434386.png)

什么都不填，直接执行，执行结果如下：有两条数据

![1567362468428](assets/1567362468428.png)



## 3.9.   解决mysql驱动的版本问题

项目虽然可以正常运行，但是查看gmall-pms中mysql驱动的版本：

 ![1567381275980](assets/1567381275980.png)

mysql驱动的版本是8.0.17，甚至更新。而在gmall-core引入mysql驱动时，指定的版本：

**注意：5.1.47版本有bug，统一使用5.1.46**

![1567381619075](assets/1567381619075.png)

由于mysql-connector-java:8.0.17版本太新，控制台已有警告，甚至可能会报错

![1567381957640](assets/1567381957640.png)

为什么引入的是5.1.47，而实际引入的是8.0.17？

问题肯定出在gmall-pms项目上，找到它的父工程（spring-boot-starter-parent）-->再找父工程的父工程（spring-boot-dependencies）

![1567381957640](assets/mysql-version.gif)

里面维护者mysql的版本号是8.0.17，所以接下来需要在gmall-pms中定义mysql的版本号，以覆盖父父工程的版本号：

![1567388313142](assets/1567388313142.png)

搞定！！！！



## 3.10.   参照PMS完成其他管理模块

其他模块都按照gmall-pms的方式完成搭建。

所有后台管理模块，都需要完成以下操作：

1. 逆向工程，copy生成的java文件及映射文件到对应的目录下

2. 在pom.xml中配置mysql的版本号，如下

   ```xml
   <mysql.version>5.1.46</mysql.version>
   ```

3. 添加application.yml配置，建议把该配置放到nacos中。例如：

   ```yaml
   server:
     port: 8081
   spring:
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
     datasource:
       driver-class-name: com.mysql.jdbc.Driver
       url: jdbc:mysql://172.16.116.100:3306/guli_pms?useUnicode=true&characterEncoding=UTF-8&useSSL=false
       username: root
       password: root
     redis:
       host: 172.16.116.100
   
   mybatis-plus:
     mapper-locations: classpath:mapper/pms/*.xml  # 读取映射文件
     global-config:
       db-config:
         id-type: auto  # 默认使用分布式id生成技术
   ```

   **注意：不同模块需要修改server.port、spring.datasource.url以及mybatis-plus.mapper-locations的值**

4. 添加bootstrap.yml配置，例如：

   ```yaml
   spring:
     application:
       name: pms-service
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848
           namespace: 3aeaadb8-203f-44a1-a384-af429c486c0a
           group: dev
           file-extension: yml  # 默认找服务名.properties文件
   ```

   **注意：不同模块需要修改spring.application.name及spring.cloud.nacos.config.namespace的值**

5. 给引导类添加注解，例如：

   ```java
   @SpringBootApplication
   @MapperScan(basePackages = "com.atguigu.gmall.pms.dao")
   @EnableSwagger2
   @RefreshScope
   public class GmallXxxApplication {
       .....
   }
   ```

   **注意：需要修改basePackages的值**

6. 添加security配置，例如：

   ```java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
   
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.authorizeRequests().antMatchers("/**").permitAll();
           // 禁用掉csrf
           http.csrf().disable();
       }
   }
   ```



## 3.11.   搭建网关服务

**nacos容器内的所有服务，可以直接相互访问。nacos外的服务，为了保证安全，必须通过网关访问后台各个管理模块**

![1567394311232](assets/1567394311232.png)

![1567394381963](assets/1567394381963.png)

pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.atguigu</groupId>
    <artifactId>gmall-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>gmall-gateway</name>
    <description>谷粒商城网关</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>com.atguigu</groupId>
            <artifactId>gmall-core</artifactId>
            <version>1.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>com.baomidou</groupId>
                    <artifactId>mybatis-plus-boot-starter</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

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

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```



添加配置：

 ![1567395922784](assets/1567395922784.png)

bootstrap.yml配置如下：

```yml
spring:
  application:
    name: gateway-api
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
```

application.yml配置如下：

```yaml
server:
  port: 8888
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      routes: 
        - id: pms-route # 商品管理路由
          uri: lb://pms-service
          predicates:
            - Path=/pms/**
        - id: oms-route # 订单管理路由
          uri: lb://oms-service
          predicates:
            - Path=/oms/**
        - id: ums-route # 用户管理路由
          uri: lb://ums-service
          predicates:
            - Path=/ums/**
        - id: wms-route # 仓库管理路由
          uri: lb://wms-service
          predicates:
            - Path=/wms/**
        - id: sms-route # 营销管理路由
          uri: lb://sms-service
          predicates:
            - Path=/sms/**
```



测试：

![1567404703573](assets/1567404703573.png)



# 4. 系统管理前后端项目联调

nodejs  vuejs      在线教育

## 4.1.   启动系统管理前端工程

把课前资料/前端工程/gmall-admin-vue.zip解压到工作空间下，例如：

![1567401975255](assets/1567401975255.png)

这是一个用nodejs构建的项目，解压后进入项目根目录，打开cmd窗口执行：

```shell
npm install  # 安装npm依赖包
npm start	# 启动该项目
```

启动成功：

![1567402119344](assets/1567402119344.png)

通过1000端口即可访问：

![1567402421699](assets/1567402421699.png)

此时还不能登录，因为没有对应的后台管理系统。



## 4.2.   启动系统管理后台工程

要想正常登陆，并使用后台管理系统，还需要启动如下工程

 ![1567402510618](assets/1567402510618.png)

启动前，需要修改配置：

 ![1567403029427](assets/1567403029427.png)

数据库连接信息：

![1567403142563](assets/1567403142563.png)

然后开始启动gmall-admin工程，启动成功后，刷新管理前端页面即可通过admin/admin登录

![1567404118329](assets/1567404118329.png)



打开品牌管理试试：

![1567404885308](assets/1567404885308.png)

这时，你使用浏览器直接该接口，发现访问没事，但是这里却报错，什么原因？

这其实是浏览器的同源策略造成的跨域问题。



# 5. 跨域问题

跨域：浏览器对于javascript的同源策略的限制 。

以下情况都属于跨域：

| 跨域原因说明       | 示例                                   |
| ------------------ | -------------------------------------- |
| 域名不同           | `www.jd.com` 与 `www.taobao.com`       |
| 域名相同，端口不同 | `www.jd.com:8080` 与 `www.jd.com:8081` |
| 二级域名不同       | `item.jd.com` 与 `miaosha.jd.com`      |





如果**域名和端口都相同，但是请求路径不同**，不属于跨域，如：

`www.jd.com/item` 

`www.jd.com/goods`

http和https也属于跨域

而我们刚才是从`localhost:1000`去访问`localhost:8888`，这属于端口不同，跨域了。



## 5.1.   为什么有跨域问题？

跨域不一定都会有跨域问题。

因为跨域问题是浏览器对于ajax请求的一种安全限制：**一个页面发起的ajax请求，只能是与当前页域名相同的路径**，这能有效的阻止跨站攻击。

因此：**跨域问题 是针对ajax的一种限制**。

但是这却给我们的开发带来了不便，而且在实际生产环境中，肯定会有很多台服务器之间交互，地址和端口都可能不同，怎么办？



## 5.2.   解决跨域问题的方案

目前比较常用的跨域解决方案有3种：

- Jsonp

  最早的解决方案，利用script标签可以跨域的原理实现。

  限制：

  - 需要服务的支持
  - 只能发起GET请求

- nginx反向代理

  思路是：利用nginx把跨域反向代理为不跨域，支持各种请求方式

  缺点：需要在nginx进行额外配置，语义不清晰 

- CORS

  规范化的跨域请求解决方案，安全可靠。

  优势：

  - 在服务端进行控制是否允许跨域，可自定义规则
  - 支持各种请求方式

  缺点：

  - 会产生额外的请求

我们这里会采用cors的跨域方案。



## 5.3.   什么是cors

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

它允许浏览器向跨源服务器，发出[`XMLHttpRequest`](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)请求，从而克服了AJAX只能[同源](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)使用的限制。

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

- 浏览器端：

  目前，所有浏览器都支持该功能（IE10以下不行）。整个CORS通信过程，都是浏览器自动完成，不需要用户参与。

- 服务端：

  CORS通信与AJAX没有任何差别，因此你不需要改变以前的业务逻辑。只不过，浏览器会在请求中携带一些头信息，我们需要以此判断是否允许其跨域，然后在响应头中加入一些信息即可。这一般通过过滤器完成即可。



## 5.4.   原理有点复杂

> 预检请求

跨域请求会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的`XMLHttpRequest`请求，否则就报错。

一个“预检”请求的样板：

```http
OPTIONS /cors HTTP/1.1
Origin: http://localhost:1000
Access-Control-Request-Method: GET
Access-Control-Request-Headers: X-Custom-Header
User-Agent: Mozilla/5.0...
```

- Origin：会指出当前请求属于哪个域（协议+域名+端口）。服务会根据这个值决定是否允许其跨域。

- Access-Control-Request-Method：接下来会用到的请求方式，比如PUT
- Access-Control-Request-Headers：会额外用到的头信息

> 预检请求的响应

服务的收到预检请求，如果许可跨域，会发出响应：

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://localhost:1000
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Max-Age: 1728000
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

如果服务器允许跨域，需要在返回的响应头中携带下面信息：

- Access-Control-Allow-Origin：可接受的域，是一个具体域名或者*（代表任意域名）
- Access-Control-Allow-Credentials：是否允许携带cookie，默认情况下，cors不会携带cookie，除非这个值是true
- Access-Control-Allow-Methods：允许访问的方式
- Access-Control-Allow-Headers：允许携带的头
- Access-Control-Max-Age：本次许可的有效时长，单位是秒，**过期之前的ajax请求就无需再次进行预检了**



> 有关cookie：

要想操作cookie，需要满足3个条件：

- 服务的响应头中需要携带Access-Control-Allow-Credentials并且为true。
- 浏览器发起ajax需要指定withCredentials 为true
- 响应头中的Access-Control-Allow-Origin一定不能为*，必须是指定的域名



## 5.5.实现非常简单

虽然原理比较复杂，但是前面说过：

- 浏览器端都有浏览器自动完成，我们无需操心
- 服务端可以通过拦截器统一实现，不必每次都去进行跨域判定的编写。

事实上，Spring已经帮我们写好了CORS的跨域过滤器，内部已经实现了刚才所讲的判定逻辑。

```
spring-webmvc：CorsFilter
spring-webflux：CorsWebFilter
```

springcloud-gateway集成的是webflux，所以这里使用的是CorsWebFilter

在`gmall-gateway`中编写一个配置类，并且注册CorsWebFilter：

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsWebFilter corsWebFilter() {

        // 初始化CORS配置对象
        CorsConfiguration config = new CorsConfiguration();
        // 允许的域,不要写*，否则cookie就无法使用了
        config.addAllowedOrigin("http://127.0.0.1:1000");
        // 允许的头信息
        config.addAllowedHeader("*");
        // 允许的请求方式
        config.addAllowedMethod("*");
        // 是否允许携带Cookie信息
        config.setAllowCredentials(true);

        // 添加映射路径，我们拦截一切请求
        UrlBasedCorsConfigurationSource corsConfigurationSource = new UrlBasedCorsConfigurationSource();
        corsConfigurationSource.registerCorsConfiguration("/**", config);

        return new CorsWebFilter(corsConfigurationSource);
    }
}
```



测试：完美解决！

![1567409144221](assets/1567409144221.png)



# 6. mybatis-plus分页

虽然记录已经展示出来了，但是总记录数没有显示：

![1567409694012](assets/1567409694012.png)

需要配置mybatis-plus的分页插件，参照官方文档：https://mp.baomidou.com/guide/page.html

![1567409812472](assets/1567409812472.png)

有xml配置方式和springboot配置方式：

![1567410077461](assets/1567410077461.png)



# 7. 图片上传

这里使用阿里云对象存储（OSS）

官方文档：https://help.aliyun.com/document_detail/64041.html?spm=5176.87240.400427.54.1bfd4614VN7fDp

我们之前采用sdk的方式，图片-->后台服务器-->阿里云。

这样后台服务器面临并发压力，既然是上传给阿里云，可不可以直接传给阿里云服务器。

上传成功后，只需要给我一个图片地址保存到数据库即可。

查看官方文档，发现提供了浏览器直接上传到阿里云的参考文档：

![1567410599428](assets/1567410599428.png)



## 7.1.   阿里云配置

登录到个人阿里云控制台，并创建bucket。

![1567411636891](assets/1567411636891.png)

找到`基础设置`-->`跨域设置`

![1567411710328](assets/1567411710328.png)

点击`设置`-->`创建规则`-->如下填写表单-->点击`确定`

![1567411872750](assets/1567411872750.png)

![1567411945299](assets/1567411945299.png)



## 7.2.   服务端签名后直传

![1567412332108](assets/1567412332108.png)

用户发送请求到应用服务端，服务端怎么返回policy和签名？

官方文档再往下翻，有java示例：

![1567413057869](assets/1567413057869.png)

点进去就有示例代码：

![1567413433392](assets/1567413433392.png)



## 7.3.   编写代码实现签名获取

![1567412666674](assets/1567412666674.png)

请求路径：/pms/oss/policy

请求方式：GET

请求参数：无

响应数据：统一都是Resp对象



### 7.3.1.   导入OSS的依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.5.0</version>
</dependency>
```



### 7.3.2.   编写Controller方法

```java
@RequestMapping("pms/oss")
@RestController
public class PmsOssController {

    String accessId = "LTAI4FtMmjBv3yrV7YGLiv2w"; // 请填写您的AccessKeyId。
    String accessKey = "ACORwTB6WUpKFCu2sfHHfXce6MeMOJ"; // 请填写您的AccessKeySecret。
    String endpoint = "oss-cn-shanghai.aliyuncs.com"; // 请填写您的 endpoint。
    String bucket = "ggmall"; // 请填写您的 bucketname 。
    String host = "https://" + bucket + "." + endpoint; // host的格式为 bucketname.endpoint
    // callbackUrl为 上传回调服务器的URL，请将下面的IP和Port配置为您自己的真实信息。
    //String callbackUrl = "http://88.88.88.88:8888";
    // 图片目录，每天一个目录
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    String dir = sdf.format(new Date()); // 用户上传文件时指定的前缀。

    @GetMapping("policy")
    public Resp<Object> policy() throws UnsupportedEncodingException {

        OSSClient client = new OSSClient(endpoint, accessId, accessKey);

        long expireTime = 30;
        long expireEndTime = System.currentTimeMillis() + expireTime * 1000;
        Date expiration = new Date(expireEndTime);
        PolicyConditions policyConds = new PolicyConditions();
        policyConds.addConditionItem(PolicyConditions.COND_CONTENT_LENGTH_RANGE, 0, 1048576000);
        policyConds.addConditionItem(MatchMode.StartWith, PolicyConditions.COND_KEY, dir);

        String postPolicy = client.generatePostPolicy(expiration, policyConds);
        byte[] binaryData = postPolicy.getBytes("utf-8");
        String encodedPolicy = BinaryUtil.toBase64String(binaryData);
        String postSignature = client.calculatePostSignature(postPolicy);

        Map<String, String> respMap = new LinkedHashMap<String, String>();
        respMap.put("accessid", accessId);
        respMap.put("policy", encodedPolicy);
        respMap.put("signature", postSignature);
        respMap.put("dir", dir);
        respMap.put("host", host);
        respMap.put("expire", String.valueOf(expireEndTime / 1000));
        // respMap.put("expire", formatISO8601Date(expiration));

        return Resp.ok(respMap);
    }
}
```



### 7.3.3.   上传测试

 ![1567419631674](assets/1567419631674.png)

![1567419692132](assets/1567419692132.png)



## 7.4.   参数获取

### 7.4.1.   AccessKeyID及AccessKeySecret

![1567413839486](assets/1567413839486.png)

![1567413992798](assets/1567413992798.png)

![1567414550420](assets/1567414550420.png)

选择用户，添加权限：

![1567414675731](assets/1567414675731.png)

选择OSS所有权限

![1567414228538](assets/1567414228538.png)

完成权限的添加



### 7.4.2.   bucktName及EndPoint

![1567415000311](assets/1567415000311.png)



### 7.4.3.   公共读权限

为了方便回显，需要把读写权限改为公共读

![1567419407674](assets/1567419407674.png)



# 8. Swagger-UI

丝袜哥

## 8.1.   什么是OpenAPI

随着互联网技术的发展，现在的网站架构基本都由原来的后端渲染，变成了：前端渲染、前后端分离的形态，而且前端技术和后端技术在各自的道路上越走越远。  前端和后端的唯一联系，变成了API接口；API文档变成了前后端开发人员联系的纽带，变得越来越重要。

没有API文档工具之前，大家都是手写API文档的，在什么地方书写的都有，而且API文档没有统一规范和格式，每个公司都不一样。这无疑给开发带来了灾难。

OpenAPI规范（OpenAPI Specification 简称OAS）是Linux基金会的一个项目，试图通过定义一种用来描述API格式或API定义的语言，来规范RESTful服务开发过程。目前V3.0版本的OpenAPI规范已经发布并开源在github上 。

官网：https://github.com/OAI/OpenAPI-Specification

## 8.2.   什么是swagger？

OpenAPI是一个编写API文档的规范，然而如果手动去编写OpenAPI规范的文档，是非常麻烦的。而Swagger就是一个实现了OpenAPI规范的工具集。

官网：https://swagger.io/

看官方的说明：

 ![1528724925709](assets/1528724925709.png)

Swagger包含的工具集：

- **Swagger编辑器：** Swagger Editor允许您在浏览器中编辑YAML中的OpenAPI规范并实时预览文档。
- **Swagger UI：** Swagger UI是HTML，Javascript和CSS资产的集合，可以从符合OAS标准的API动态生成漂亮的文档。
- **Swagger Codegen：**允许根据OpenAPI规范自动生成API客户端库（SDK生成），服务器存根和文档。
- **Swagger Parser：**用于解析来自Java的OpenAPI定义的独立库
- **Swagger Core：**与Java相关的库，用于创建，使用和使用OpenAPI定义
- **Swagger Inspector（免费）：** API测试工具，可让您验证您的API并从现有API生成OpenAPI定义
- **SwaggerHub（免费和商业）：** API设计和文档，为使用OpenAPI的团队构建。



## 8.3.   快速入门

SpringBoot已经集成了Swagger，使用简单注解即可生成swagger的API文档。

### 8.3.1.   引入依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```



### 8.3.2.   编写配置

 ![1567421527268](assets/1567421527268.png)

```java
@Configuration
@EnableSwagger2 // 加载引导类上或者配置类上都可以
public class SwaggerConfig {

    @Bean("商品平台")
    public Docket userApis() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("商品平台")
                .select()
                .apis(RequestHandlerSelectors.withClassAnnotation(Api.class)) // 所有标了API注解的才在文档中展示
                .paths(PathSelectors.regex("/pms.*")) // pms下的所有请求
                .build()
                .apiInfo(apiInfo())
                .enable(true);
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("谷粒商城-商品平台接口文档")
                .description("提供商品平台的文档")
                .termsOfServiceUrl("http://www.atguigu.com/")
                .version("1.0")
                .build();
    }

}
```



### 8.3.3.   接口声明

![1567420938889](assets/1567420938889.png)

常用注解说明：

```java
/**
 @Api：修饰整个类，描述Controller的作用
 @ApiOperation：描述一个类的一个方法，或者说一个接口
 @ApiParam：单个参数描述
 @ApiModel：用对象来接收参数
 @ApiProperty：用对象接收参数时，描述对象的一个字段
 @ApiResponse：HTTP响应其中1个描述
 @ApiResponses：HTTP响应整体描述
 @ApiIgnore：使用该注解忽略这个API
 @ApiError ：发生错误返回的信息
 @ApiImplicitParam：一个请求参数
 @ApiImplicitParams：多个请求参数
 */
```



### 8.3.4.   启动测试

启动服务，然后访问：http://localhost:8081/swagger-ui.html

![1567421405789](assets/1567421405789.png)



## 8.4.   所有工程添加swagger配置

