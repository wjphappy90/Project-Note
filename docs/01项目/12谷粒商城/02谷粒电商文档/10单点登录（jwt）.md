



# 1. 数据验证功能

参照课前资料中的《用户中心接口说明.md》

需要的东西都一清二楚：

- 请求方式：GET
- 请求路径：check/{param}/{type}
- 请求参数：param,type
- 返回结果：true或false



## 1.1.   controller

```java
/**
  * 校验数据是否可用
  * @param data
  * @param type
  * @return
  */
@GetMapping("check/{data}/{type}")
public Resp<Boolean> checkData(@PathVariable("data") String data, @PathVariable("type") Integer type) {
    Boolean b = this.memberService.checkData(data, type);

    return Resp.ok(b);
}
```



## 1.2.   Service

```java
@Override
public Boolean checkData(String data, Integer type) {
    QueryWrapper<MemberEntity> wrapper = new QueryWrapper<>();
    switch (type) {
        case 1:
            wrapper.eq("username", data);
            break;
        case 2:
            wrapper.eq("mobile", data);
            break;
        case 3:
            wrapper.eq("email", data);
            break;
        default:
            return null;
    }
    return this.memberDao.selectCount(wrapper) == 0;
}
```



## 1.3.   测试

我们在数据库插入一条假数据：

![1570084398728](assets/1570084398728.png)

然后在浏览器调用接口，测试：

![1570084470608](assets/1570084470608.png)

![1570084533646](assets/1570084533646.png)



# 2. 发送短信功能

参考接口文档，这里的业务逻辑是这样的：

- 1）我们接收页面发送来的手机号码
- 2）生成一个随机验证码
- 3）将验证码保存在服务端
- 4）发送短信，将验证码发送到用户手机

那么问题来了：验证码保存在哪里呢？

验证码有一定有效期，一般是5分钟，我们可以利用Redis的过期机制来保存。



具体实现略。。。。



# 3. 注册功能

基本逻辑：

- 1）校验短信验证码
- 2）生成盐
- 3）对密码加密
- 4）写入数据库
- 5）删除Redis中的验证码



## 3.1.   MemberController

```java
/**
 * 注册
 * @param memberEntity
 * @param code
 * @return
 */
@PostMapping("register")
public Resp<Object> register(MemberEntity memberEntity, @RequestParam("code") String code) {
    this.memberService.register(memberEntity, code);

    return Resp.ok(null);
}
```



## 3.2.   MemberService

```java
public void register(MemberEntity memberEntity, String code) {
    // 校验短信验证码
    // String cacheCode = this.redisTemplate.opsForValue().get(KEY_PREFIX + memberEntity.getMobile());
    // if (!StringUtils.equals(code, cacheCode)) {
    //     return false;
    // }

    // 生成盐
    String salt = StringUtils.replace(UUID.randomUUID().toString(), "-", "");
    memberEntity.setSalt(salt);

    // 对密码加密
    memberEntity.setPassword(DigestUtils.md5Hex(salt + DigestUtils.md5Hex(memberEntity.getPassword())));

    // 设置创建时间等
    memberEntity.setCreateTime(new Date());

    // 添加到数据库
    boolean b = this.save(memberEntity);

    // if(b){
    // 注册成功，删除redis中的记录
    // this.redisTemplate.delete(KEY_PREFIX + memberEntity.getMobile());
    // }
}
```



## 3.3.   测试

我们通过PostMan测试：

![1570087515131](assets/1570087515131.png)

查看数据库：

![1570088222195](assets/1570088222195.png)

查看redis中的信息也被删除



# 4. 根据用户名和密码查询用户

请求方式：GET

请求路径：/ums/member/query

请求参数：username password

响应数据：用户的json格式



## 4.1.   controller

```java
/**
 * 根据用户名和密码查询用户
 * @param username
 * @param password
 * @return
 */
@GetMapping("query")
public Resp<MemberEntity> queryUser(
    @RequestParam("username") String username,
    @RequestParam("password") String password
    ) {
        MemberEntity memberEntity = this.memberService.queryUser(username, password);

        return Resp.ok(memberEntity);
    }
```

## 4.2.   service

```java
@Override
public MemberEntity queryUser(String username, String password) {
    // 查询
    MemberEntity memberEntity = this.getOne(new QueryWrapper<MemberEntity>().eq("username", username));
    // 校验用户名
    if (memberEntity == null) {
        return null;
    }
    // 校验密码
    if (!memberEntity.getPassword().equals(DigestUtils.md5Hex(memberEntity.getSalt() + DigestUtils.md5Hex(password)))) {
        return null;
    }
    // 用户名密码都正确
    return memberEntity;
}
```

要注意，查询时也要对密码进行加密后判断是否一致。



## 4.3.   测试

![1570089177601](assets/1570089177601.png)



# 5. 单点登录（SSO）

SSO英文全称Single Sign On，单点登录。

SSO是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。

![1570089652583](assets/1570089652583.png)



## 5.1.   cookie问题

电商平台通常由多个微服务组成，每个微服务都有独立的域名，而cookie是有作用域的。

查看浏览器控制台：

![1570092836836](assets/1570092836836.png)

domain：作用域名

| domain参数        | atguigu.com | sso.atguigu.com | order.atguigu.com |
| ----------------- | ----------- | --------------- | ----------------- |
| atguigu.com       | √           | √               | √                 |
| sso.atguigu.com   | √           | √               | ×                 |
| order.atguigu.com | ×           | ×               | √                 |

domain有两点要注意：

​	1. **domain参数可以设置父域名以及自身，但不能设置其它域名，包括子域名，否则cookie不起作用。**

​	2. **cookie的作用域是domain本身以及domain下的所有子域名。**



Cookie的路径（Path）：

​	response.addCookie默认放在当前路径下，访问当前路径下的所有请求都会带

​	设置/标识项目根路径，访问项目任何位置都会携带



## 5.2.   演示案例

把课前资料中的sso演示工程 《sso-example》导入idea，并且启动。

![1570103418857](assets/1570103418857.png)

在host文件中配置域名的映射：

![1570103160672](assets/1570103160672.png)

追加配置如下：

```
127.0.0.1 client.atguigu.com
127.0.0.1 sso.atguigu.com
```



### 5.2.1.   测试一：不能访问兄弟域名cookie

访问：http://client.atguigu.com:8080/hello

由于没有登录会重定向到登录页面：

![1570103522358](assets/1570103522358.png)

输入用户名密码（任意）点击登录，又回到了上述页面。

查看浏览器cookie，发现：

![1570103772200](assets/1570103772200.png)

sso.atguigu.com下已经有token信息。那么为什么又回到了登录页面呢？

这是由于点击登录时，cookie放入了sso.atguigu.com这个作用域，client域下没有cookie导致，再次访问client时，client认为没有登录，又重定向到登录页面



### 5.2.2.   测试二：可以访问父域名的cookie

修改sso-service工程LoginController类的login方法，把cookie的作用域设置为`atguigu.com`

![1570104693601](assets/1570104693601.png)

重启sso-service。

并清理掉浏览器中的cookie：![1570104808919](assets/1570104808919.png)



访问：http://client.atguigu.com:8080/hello

依然重定向到登录页面：

![1570103522358](assets/1570103522358.png)

输入任意内容，点击登录：

![1570104959025](assets/1570104959025.png)

可以登录成功！！



### 5.2.3.   测试三：cookie的作用路径

修改sso-service工程LoginController类的login方法，把cookie的作用路径设置为`/hello`

![1570105058571](assets/1570105058571.png)

重启sso-service服务，并清理掉cookie信息。



在浏览器中访问：http://client.atguigu.com:8080/hello

依然重定向到登录页面，输入任意内容，点击登录：

![1570105244367](assets/1570105244367.png)

可以登录成功，但是cookie的作用路径是/hello。

此时访问：http://client.atguigu.com:8080/hello1

又会跳转到登录页面。原因：cookie只能在/hello路径及其子路径下可以正常访问。



## 5.3.   有状态登录

为了保证客户端cookie的安全性，服务端需要记录每次会话的客户端信息，从而识别客户端身份，根据用户身份进行请求的处理，典型的设计如tomcat中的session。

例如登录：用户登录后，我们把登录者的信息保存在服务端session中，并且给用户一个cookie值，记录对应的session。然后下次请求，用户携带cookie值来，我们就能识别到对应session，从而找到用户的信息。

缺点是什么？

- 服务端保存大量数据，增加服务端压力
- 服务端保存用户状态，无法进行水平扩展
- 客户端请求依赖服务端，多次请求必须访问同一台服务器

即使使用redis保存用户的信息，也会损耗服务器资源。



## 5.4.   无状态登录

微服务集群中的每个服务，对外提供的都是Rest风格的接口。而Rest风格的一个最重要的规范就是：服务的无状态性，即：

- 服务端不保存任何客户端请求者信息
- 客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份

带来的好处是什么呢？

- 客户端请求不依赖服务端的信息，任何多次请求不需要必须访问到同一台服务
- 服务端的集群和状态对客户端透明
- 服务端可以任意的迁移和伸缩
- 减小服务端存储压力



## 5.5.   无状态登录流程

无状态登录的流程：

- 当客户端第一次请求服务时，服务端对用户进行信息认证（登录）
- 认证通过，将用户信息进行加密形成token，返回给客户端，作为登录凭证
- 以后每次请求，客户端都携带认证的token
- 服务的对token进行解密，判断是否有效。

流程图：

 ![](assets/1527300483893.png)

整个登录过程中，最关键的点是什么？

**token的安全性**

token是识别客户端身份的唯一标示，如果加密不够严密，被人伪造那就完蛋了。

采用何种方式加密才是安全可靠的呢？

我们将采用`JWT + RSA非对称加密`



# 6. jwt实现无状态登录

JWT，全称是Json Web Token， 是JSON风格轻量级的授权和身份认证规范，可实现无状态、分布式的Web应用授权；官网：https://jwt.io

GitHub上jwt的java客户端：https://github.com/jwtk/jjwt



## 6.1.   数据格式

JWT包含三部分数据：

- Header：头部，通常头部有两部分信息：

  - token类型：JWT
  - 加密方式：base64（HS256）

- Payload：载荷，就是有效数据，一般包含下面信息：

  - 用户身份信息（注意，这里因为采用base64编码，可解码，因此不要存放敏感信息）
  - 注册声明：如token的签发时间，过期时间，签发人等

  这部分也会采用base64编码，得到第二部分数据

- Signature：签名，是整个数据的认证信息。根据前两步的数据，再加上指定的密钥（secret）（不要泄漏，最好周期性更换），通过base64编码生成。用于验证整个数据完整和可靠性

![1570109371216](assets/1570109371216.png)



## 6.2.   JWT交互流程

流程图：

 ![1527305891424](assets/1527305891424.png)

步骤翻译：

- 1、用户登录
- 2、服务的认证，通过后根据secret生成token
- 3、将生成的token返回给浏览器
- 4、用户每次请求携带token
- 5、服务端利用公钥解读jwt签名，判断签名有效后，从Payload中获取用户信息
- 6、处理请求，返回响应结果

因为JWT签发的token中已经包含了用户的身份信息，并且每次请求都会携带，这样服务的就无需保存用户信息，甚至无需去数据库查询，完全符合了Rest的无状态规范。



## 6.3.   非对称加密

加密技术是对信息进行编码和解码的技术，编码是把原来可读信息（又称明文）译成代码形式（又称密文），其逆过程就是解码（解密），加密技术的要点是加密算法，加密算法可以分为三类：  

- 对称加密，如AES
  - 基本原理：将明文分成N个组，然后使用密钥对各个组进行加密，形成各自的密文，最后把所有的分组密文进行合并，形成最终的密文。
  - 优势：算法公开、计算量小、加密速度快、加密效率高
  - 缺陷：双方都使用同样密钥，安全性得不到保证 
- 非对称加密，如RSA
  - 基本原理：同时生成两把密钥：私钥和公钥，私钥隐秘保存，公钥可以下发给信任客户端
    - 私钥加密，持有公钥才可以解密
    - 公钥加密，持有私钥才可解密
  - 优点：安全，难以破解
  - 缺点：算法比较耗时
- 不可逆加密，如MD5，SHA 
  - 基本原理：加密过程中不需要使用[密钥](https://baike.baidu.com/item/%E5%AF%86%E9%92%A5)，输入明文后由系统直接经过加密算法处理成密文，这种加密后的数据是无法被解密的，无法根据密文推算出明文。



RSA算法历史：

1977年，三位数学家Rivest、Shamir 和 Adleman 设计了一种算法，可以实现非对称加密。这种算法用他们三个人的名字缩写：RSA



# 7. 搭建授权中心

![1570120069198](assets/1570120069198.png)

用户鉴权：

- 接收用户的登录请求，通过用户中心的接口进行校验，通过后生成JWT
- 使用私钥生成JWT并返回

有一些生成jwt，解析jwt这样行为的工具类，以后在其它微服务中也会用到，因此放在gmall-core中。



## 7.1.   创建工程

![1570112397783](assets/1570112397783.png)

![1570112559077](assets/1570112559077.png)

 ![1570112740224](assets/1570112740224.png)

pom.xml中添加依赖：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.atguigu</groupId>
    <artifactId>gmall-auth</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>gmall-auth</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

启动类：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class GmallAuthApplication {

    public static void main(String[] args) {
        SpringApplication.run(GmallAuthApplication.class, args);
    }

}
```

bootstrap.yml：

```yml
spring:
  application:
    name: auth-service
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
```

application.yml：

```yml
server:
  port: 8089
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```



网关工程gmall-gateway添加用户授权的网关路由：

```yml
server:
  port: 8888
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
    gateway:
      routes: 
        - id: pms-route 
          uri: lb://pms-service
          predicates:
            - Path=/pms/**
        - id: oms-route 
          uri: lb://oms-service
          predicates:
            - Path=/oms/**
        - id: ums-route 
          uri: lb://ums-service
          predicates:
            - Path=/ums/**
        - id: wms-route 
          uri: lb://wms-service
          predicates:
            - Path=/wms/**
        - id: sms-route 
          uri: lb://sms-service
          predicates:
            - Path=/sms/**
        - id: search-route
          uri: lb://search-service
          predicates:
            - Path=/api/search/**
          filters:
            - RewritePath=/api/(?<segment>.*), /$\{segment}
        - id: search-route # 首页路由
          uri: lb://index-service
          predicates:
            - Path=/api/index/**
          filters:
            - RewritePath=/api/(?<segment>.*), /$\{segment}
        - id: item-route # 商品详情路由
          uri: lb://item-service
          predicates:
            - Path=/api/item/**
          filters:
            - RewritePath=/api/(?<segment>.*), /$\{segment}
        - id: auth-service # 授权中心
          uri: lb://auth-service
          predicates:
            - Path=/api/auth/**
          filters:
            - RewritePath=/api/(?<segment>.*), /$\{segment}
```



## 7.2.   JWT工具类

把课前资料中的jwt工具类引入gmall-core工程中：

 ![1570114224698](assets/1570114224698.png)

并在gmall-core的pom.xml中引入新的依赖：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.10.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.10.6</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.10.6</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.10.3</version>
</dependency>
```



## 7.3.   测试工具类

```java
public class JwtTest {
	private static final String pubKeyPath = "C:\\tmp\\rsa\\rsa.pub";

    private static final String priKeyPath = "C:\\tmp\\rsa\\rsa.pri";

    private PublicKey publicKey;

    private PrivateKey privateKey;

    @Test
    public void testRsa() throws Exception {
        RsaUtils.generateKey(pubKeyPath, priKeyPath, "234");
    }

    @Before
    public void testGetRsa() throws Exception {
        this.publicKey = RsaUtils.getPublicKey(pubKeyPath);
        this.privateKey = RsaUtils.getPrivateKey(priKeyPath);
    }

    @Test
    public void testGenerateToken() throws Exception {
        Map<String, Object> map = new HashMap<>();
        map.put("id", "11");
        map.put("username", "liuyan");
        // 生成token
        String token = JwtUtils.generateToken(map, privateKey, 5);
        System.out.println("token = " + token);
    }

    @Test
    public void testParseToken() throws Exception {
        String token = "eyJhbGciOiJSUzI1NiJ9.eyJpZCI6IjExIiwidXNlcm5hbWUiOiJsaXV5YW4iLCJleHAiOjE1NzAxMjEyODZ9.GioCiqMt_ZcN6_RAuDBcOzcHQ5WdqdhA9QYu-2IqCQqnAef1VyXczEInj1Ef1xo7AvcjxnkIMuZK48OoczUy1iqtPQPDchUzTl03b8h_J3xMBaxOAaKSwMpm20DH25VrTgBExUafyxHwxfOa-PVHW0Kk41KrWDncayzXbZ_lYLoa9Cuvacr8eAFz-ckriIiZ9bRzFkhX-wYHSHFlym2IJRjBRhFtpkN5GLAVsmsdm-yD4eiJXqioWspqXiBSdROsjrTRiFe511yujR0y2ngL9OnZ1QH6bHDQ2WmhPTrswKjjy-HWIxk1FQ7uXtSpPa5diymmPVTWA0clys7R1MK9oQ";

        // 解析token
        Map<String, Object> map = JwtUtils.getInfoFromToken(token, publicKey);
        System.out.println("id: " + map.get("id"));
        System.out.println("userName: " + map.get("username"));
    }
}
```



测试生成公钥和私钥，我们运行这段代码：**注意需要把@Before方法注释掉

![1570121361261](assets/1570121361261.png)

运行之后，查看目标目录：

![1570121408048](assets/1570121408048.png)



测试生成token，把@Before的注释去掉的：

![1570121462492](assets/1570121462492.png)

![1570121524935](assets/1570121524935.png)



测试解析token：

![1570121569190](assets/1570121569190.png)

正常情况：

![1570121735824](assets/1570121735824.png)

任意改动一下：

![1570121792041](assets/1570121792041.png)



# 8. 编写登录接口

接下来，我们需要在`gmall-auth`编写一个接口，对外提供登录授权服务。基本流程如下：

- 客户端携带用户名和密码请求登录
- 授权中心调用用户中心接口，根据用户名和密码查询用户信息
- 如果用户名密码正确，能获取用户，否则登录失败
- 如果校验成功，则生成JWT并返回



## 8.1.   编写远程调用接口

创建gmall-ums-interface工程：

 ![1570148351952](assets/1570148351952.png)

pom.xml中的依赖，参照其他interface工程。并在gmall-ums和gmall-auth工程中引入该接口工程

GmallUmsApi:

```java
public interface GmallUmsApi {

    /**
     * 根据用户名和密码查询用户
     * @param username
     * @param password
     * @return
     */
    @GetMapping("ums/member/query")
    public Resp<MemberEntity> queryUser(
            @RequestParam("username") String username,
            @RequestParam("password") String password
    );
}
```



## 8.2.   生成公钥和私钥

我们需要在授权中心生成真正的公钥和私钥。我们必须有一个生成公钥和私钥的secret，这个可以配置到`application.yml`中：

```yaml
gmall:
  jwt:
    pubKeyPath: C:\\tmp\\rsa\\rsa.pub # 公钥地址
    priKeyPath: C:\\tmp\\rsa\\rsa.pri # 私钥地址
    secret: sf3423jsdf#3$@FDS32
    expire: 30 # 过期时间,单位分钟
    cookieName: GMALL_TOKEN
```

然后编写属性类读取jwt配置，并从秘钥配置文件中读取出响应的公钥及私钥，加载这些数据：

```java
@Data
@Slf4j
@ConfigurationProperties(prefix = "gmall.jwt")
public class JwtProperties {

    private String secret; // 密钥

    private String pubKeyPath;// 公钥

    private String priKeyPath;// 私钥

    private int expire;// token过期时间

    private PublicKey publicKey; // 公钥

    private PrivateKey privateKey; // 私钥

    private String cookieName; // cookie名称

    /**
     * @PostContruct：在构造方法执行之后执行该方法
     */
    @PostConstruct
    public void init() {
        try {
            File pubKey = new File(pubKeyPath);
            File priKey = new File(priKeyPath);
            if (!pubKey.exists() || !priKey.exists()) {
                // 生成公钥和私钥
                RsaUtils.generateKey(pubKeyPath, priKeyPath, secret);
            }
            // 获取公钥和私钥
            this.publicKey = RsaUtils.getPublicKey(pubKeyPath);
            this.privateKey = RsaUtils.getPrivateKey(priKeyPath);
        } catch (Exception e) {
            log.error("初始化公钥和私钥失败！", e);
            throw new RuntimeException();
        }
    }
}
```



## 8.3.   AuthController

编写授权接口，我们接收用户名和密码，校验成功后，写入cookie中。

- 请求方式：post
- 请求路径：/auth/accredit
- 请求参数：username和password
- 返回结果：无

代码：

```java
@RestController
@RequestMapping("auth")
@EnableConfigurationProperties(JwtProperties.class)
public class AuthController {

    @Autowired
    private AuthService authService;

    @Autowired
    private JwtProperties jwtProperties;

    /**
     * 登录授权
     *
     * @param username
     * @param password
     * @return
     */
    @PostMapping("accredit")
    public Resp<Object> authentication(
            @RequestParam("username") String username,
            @RequestParam("password") String password,
            HttpServletRequest request,
            HttpServletResponse response) {
        // 登录校验
        String token = this.authService.authentication(username, password);
        if (StringUtils.isBlank(token)) {
            return Resp.fail("登录失败，用户名或密码错误");
        }
        // 将token写入cookie,并指定httpOnly为true，防止通过JS获取和修改
        CookieUtils.setCookie(request, response, jwtProperties.getCookieName(), token, jwtProperties.getExpire());
        return Resp.ok("登录成功");
    }
}
```



## 8.4.   AuthService

在leyou-auth-service：

```java
@Service
public class AuthService {

    @Autowired
    private GmallUmsClient umsClient;

    @Autowired
    private JwtProperties jwtProperties;

    public String authentication(String username, String password) {

        try {
            // 调用微服务，执行查询
            Resp<MemberEntity> resp = this.umsClient.queryUser(username, password);
            MemberEntity memberEntity = resp.getData();

            // 如果查询结果为null，则直接返回null
            if (memberEntity == null) {
                return null;
            }

            // 如果有查询结果，则生成token
            Map<String, Object> map = new HashMap<>();
            map.put("id", memberEntity.getId());
            map.put("username", memberEntity.getUsername());
            String token = JwtUtils.generateToken(map, jwtProperties.getPrivateKey(), jwtProperties.getExpire());
            return token;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```



## 8.5.   GmallUmsClient

接下来我们肯定要对用户密码进行校验，所以我们需要通过FeignClient去访问 ums-service微服务：

在gmall-auth中引入gmall-ums-interface依赖：

```xml
<dependency>
    <groupId>com.atguigu</groupId>
    <artifactId>gmall-ums-interface</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

编写GmallUmsClient：

```java
@FeignClient("ums-service")
public interface GmallUmsClient extends GmallUmsApi {
}
```



## 8.6.   CookieUtils

要注意，这里我们使用了一个工具类，CookieUtils，可以在课前资料中找到，我们把它添加到`gmall-core`中，然后引入servlet相关依赖即可：

```xml
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-core</artifactId>
</dependency>
```

代码：略

 ![1570149969329](assets/1570149969329.png)



## 8.7.   完整的项目结构

 ![1570150064016](assets/1570150064016.png)



## 8.8.   测试

失败时：

![1570150850207](assets/1570150850207.png)

成功时：

![1570150816244](assets/1570150816244.png)



# 9. 网关登录拦截

接下来，我们在gmall-gateway编写过滤器，对用户的token进行校验，如果发现未登录，则进行拦截。



## 9.1.   引入jwt相关配置

既然是登录拦截，一定是前置拦截器，我们在`leyou-gateway`中定义。

首先在pom.xml中，引入所需要的依赖：

```xml
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
```

然后编写application.yml属性文件，添加如下内容：

```yaml
gmall:
  jwt:
    pubKeyPath: C:\\tmp\\rsa\\rsa.pub # 公钥地址
    cookieName: GMALL_TOKEN # cookie的名称
```

编写属性类，读取公钥：

 ![1570159675427](assets/1570159675427.png)



```java
@Data
@Slf4j
@ConfigurationProperties(prefix = "gmall.jwt")
public class JwtProperties {

    private String pubKeyPath;// 公钥

    private PublicKey publicKey; // 公钥

    private String cookieName;

    @PostConstruct
    public void init(){
        try {
            // 获取公钥和私钥
            this.publicKey = RsaUtils.getPublicKey(pubKeyPath);
        } catch (Exception e) {
            log.error("初始化公钥失败！", e);
            throw new RuntimeException();
        }
    }

}
```



## 9.2.   自定义token过滤器

gateway自定义过滤器分3步：

1. 自定义定义认证过滤器
2. 自定义认证过滤器工厂
3. 配置中使用自定义过滤器

 ![1570159785246](assets/1570159785246.png)



### 9.2.1.   过滤器AuthGatewayFilter

过滤器的基本逻辑：

- 获取cookie中的token
- 通过JWT对token进行校验
- 通过：则放行；不通过：则响应认证未通过

```java
@Component
@EnableConfigurationProperties(JwtProperties.class)
public class AuthGatewayFilter implements GatewayFilter {

    @Autowired
    private JwtProperties jwtProperties;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        // 获取request和response，注意：不是HttpServletRequest及HttpServletResponse
        ServerHttpRequest request = exchange.getRequest();
        ServerHttpResponse response = exchange.getResponse();
        // 获取所有cookie
        MultiValueMap<String, HttpCookie> cookies = request.getCookies();
        // 如果cookies为空或者不包含指定的token，则相应认证未通过
        if (CollectionUtils.isEmpty(cookies) || !cookies.containsKey(this.jwtProperties.getCookieName())) {
            // 响应未认证！
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            // 结束请求
            return response.setComplete();
        }
        // 获取cookie
        HttpCookie cookie = cookies.getFirst(this.jwtProperties.getCookieName());
        try {
            // 校验cookie
            JwtUtils.getInfoFromToken(cookie.getValue(), this.jwtProperties.getPublicKey());
        } catch (Exception e) {
            e.printStackTrace();
            // 校验失败，响应未认证
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            return response.setComplete();
        }

        // 认证通过放行
        return chain.filter(exchange);
    }
}
```



### 9.2.2.   过滤器工厂AuthGatewayFilterFactory

```java
@Component
public class AuthGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {

    @Autowired
    private AuthGatewayFilter authGatewayFilter;

    @Override
    public GatewayFilter apply(Object config) {
        return authGatewayFilter;
    }
}
```



### 9.2.3.   在配置文件中使用

现在拿gmall-pms工程尝试使用吧

![1570160533717](assets/1570160533717.png)



### 9.2.4.   测试

随便找个接口测试一下：http://localhost:8888/pms/brand/info/4

![1570160626585](assets/1570160626585.png)

过滤器生效了！！！

测试完成后，不要忘记在把配置中的过滤器去掉哟。



## 9.3.   异常解决

如果网关报如下错误：

![1570158724129](assets/1570158724129.png)

原因：springCloud-gateway内部集成的是webflux而不是servlet，所以需要排除servlet相关的依赖。

tomcat是servlet容器

![1570158894801](assets/1570158894801.png)