# SpringBoot笔记

## 1、`@SpringBootConfiguration`注解

- SpringBoot应用标在某个类上，说明这个类是主配置类
- SpringBoot应该运行这个类的main方法来启动SpringBoot应用

相当于以下注解等同时作用
1. @Congfiguration: 用来标注配置类

   > 配置类也是容器中的一个组件(**@Compponent**)

2. @EnableAutoConfiguration: 表示通知SpringBoot开启自动配置功能

   > - 底层为**@Import({AutoConfigurationImportSelector.class})**
   >
   > - 作用是扫描主配置类所在包下及所有子包的所有组件
   > - AutoConfigurationImportSelector是导入组件的选择器，给容器中导入了很多自动
   >   配置类(xxxAutoConfiguration)。
   > - SpringBoot在启动时从类路径下META-INF/spring.factories(没找到在哪)中获取EnableAutoConfiguration指定的值，并作为自动配置类导入到容器中，进行自动配置
   > - 自动配置在spring-boot-autoconfigure-x.x.x.RELEASE.jar包中

## 2、`@ConfigurationProperties(prefix = "xxx")`注解

通知SpringBoot把被标注的类中属性和配置文件application.yml中相关配置进行绑定，（配置文件中前缀为"xxx"的项）与被标注的类所有属性一一映射。

spring-boot-configuration-processor是进行上面绑定的配置文件绑定处理器。

这个注解必须作用在容器的组件上才会生效（@Component表示被标注类是容器的组件）。



## 3、profile文件

#### 3.1、多profile文件

在主配置文件编写时，文件名写成application-{profile}.properties/yml；

多个配置文件存在时，默认使用application.properties/yml的配置。

#### 3.2、激活制定profile

1、在application.properties中添加如下代码，就会激活后缀名为application-dev.properties的配置文件。

```properties
spring.profiles.active=dev
```

2、在命令行参数里使用指定xxxx打成的jar包使用哪个配置文件。

```shell
java -jar xxxx.jar --spring.profiles.active=dev
```



#### 3.3、在yml的用法

在yml中使用文档块实现上面的操作，代码如下

```yml
spring:
  profiles:
    active: dev
    
---

server:
  port: 8081

spring:
  profiles: dev
  
  
---

spring:
  profiles: prod

server:
  port: 8082
```



## 4、CORS跨域问题



[来源]: https://www.cnblogs.com/yuansc/p/9076604.html



#### 一、同源策略简介

**同源策略**[same origin policy]是浏览器的一个安全功能，不同源的客户端脚本在没有明确授权的情况下，不能读写对方资源。 同源策略是浏览器安全的基石。

##### 1、什么是源

**源**[origin]就是协议、域名和端口号。例如：http://www.baidu.com:80这个URL。

##### 2、什么是同源

若地址里面的协议、域名和端口号均相同则属于同源。

##### 3、是否是同源的判断

例如判断下面的`URL`是否与 http://www.a.com/test/index.html 同源

- http://www.a.com/dir/page.html 同源
- http://www.child.a.com/test/index.html 不同源，域名不相同
- https://www.a.com/test/index.html 不同源，协议不相同
- http://www.a.com:8080/test/index.html 不同源，端口号不相同

##### 4、哪些操作不受同源策略限制

1. 页面中的链接，重定向以及表单提交是不会受到同源策略限制的；
2. 跨域资源的引入是可以的。但是`JS`不能读写加载的内容。如嵌入到页面中的`  <script src="..."></script>  `，`  <img>  `，`  <iframe>  `等。

##### 5、跨域

受前面所讲的浏览器同源策略的影响，不是同源的脚本不能操作其他源下面的对象。想要操作另一个源下的对象就需要跨域。 在同源策略的限制下，**非同源**的网站之间不能发送 `AJAX` 请求。

##### 6、如何跨域

- 降域

  可以通过设置 `document.damain='a.com'`，浏览器就会认为它们都是同一个源。想要实现以上任意两个页面之间的通信，两个页面必须都设置`documen.damain='a.com'`。

- `JSONP`跨域

- `CORS` 跨域

#### 二、CORS 简介

为了解决浏览器同源问题，`W3C` 提出了跨源资源共享，即 `CORS`([Cross-Origin Resource Sharing](https://www.w3.org/TR/cors/))。

`CORS` 做到了如下两点：

- 不破坏即有规则
- 服务器实现了 `CORS` 接口，就可以跨源通信

基于这两点，`CORS` 将请求分为两类：简单请求和非简单请求。

##### 1、简单请求

在`CORS`出现前，发送`HTTP`请求时在头信息中不能包含任何自定义字段，且 `HTTP` 头信息不超过以下几个字段：

- `Accept`
- `Accept-Language`
- `Content-Language`
- `Last-Event-ID`
- `Content-Type` 只限于 [`application/x-www-form-urlencoded` 、`multipart/form-data`、`text/plain` ] 类型

一个简单的请求例子：

```
GET /test HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate, sdch, br
Origin: http://www.examples.com
Host: www.examples.com
```

对于简单请求，`CORS`的策略是请求时在请求头中增加一个`Origin`字段，服务器收到请求后，根据该字段判断是否允许该请求访问。

1. 如果允许，则在 HTTP 头信息中添加 `Access-Control-Allow-Origin` 字段，并返回正确的结果 ；
2. 如果不 允许，则不在 HTTP 头信息中添加 `Access-Control-Allow-Origin` 字段 。

除了上面提到的 `Access-Control-Allow-Origin` ，还有几个字段用于描述 `CORS` 返回结果 ：

1. `Access-Control-Allow-Credentials`： 可选，用户是否可以发送、处理 `cookie`；
2. `Access-Control-Expose-Headers`：可选，可以让用户拿到的字段。有几个字段无论设置与否都可以拿到的，包括：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma` 。

##### 2、非简单请求

对于非简单请求的跨源请求，**浏览器会在真实请求发出前**，增加一次`OPTION`请求，称为预检请求(`preflight request`)。预检请求将真实请求的信息，包括请求方法、自定义头字段、源信息添加到 HTTP 头信息字段中，询问服务器是否允许这样的操作。

例如一个`DELETE`请求：

```
OPTIONS /test HTTP/1.1
Origin: http://www.examples.com
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: X-Custom-Header
Host: www.examples.com
```

与 `CORS` 相关的字段有：

1. 请求使用的 `HTTP` 方法 `Access-Control-Request-Method` ；
2. 请求中包含的自定义头字段 `Access-Control-Request-Headers` 。

服务器收到请求时，需要分别对 `Origin`、`Access-Control-Request-Method`、`Access-Control-Request-Headers` 进行验证，验证通过后，会在返回 `HTTP`头信息中添加 ：

```
Access-Control-Allow-Origin: http://www.examples.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

他们的含义分别是：

1. Access-Control-Allow-Methods: 真实请求允许的方法
2. Access-Control-Allow-Headers: 服务器允许使用的字段
3. Access-Control-Allow-Credentials: 是否允许用户发送、处理 cookie
4. Access-Control-Max-Age: 预检请求的有效期，单位为秒。有效期内，不会重复发送预检请求

当预检请求通过后，浏览器会发送真实请求到服务器。这就实现了跨源请求。

#### 三、Spring Boot 配置 CORS

##### 1、使用`@CrossOrigin` 注解实现

`#`如果想要对某一接口配置 `CORS`，可以在方法上添加 `@CrossOrigin` 注解 ：

```java
@CrossOrigin(origins = {"http://localhost:9000", "null"})
@RequestMapping(value = "/test", method = RequestMethod.GET)
public String greetings() {
    return "{\"project\":\"just a test\"}";
}
```

`#`如果想对一系列接口添加 CORS 配置，可以在类上添加注解，对该类声明所有接口都有效：

```java
@CrossOrigin(origins = {"http://localhost:9000", "null"})
@RestController
@SpringBootApplication
public class SpringBootCorsTestApplication {
    
}
```

`#`如果想添加全局配置，则需要添加一个配置类 ：

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("POST", "GET", "PUT", "OPTIONS", "DELETE")
                .maxAge(3600)
                .allowCredentials(true);
    }
}
```

*上面的  `WebMvcConfigurerAdapter`  已过时，当前配置类如下*

```java
/**
 * @description: 允许跨域访问
 *      配置的详细信息说明如下：
 *      addMapping：配置可以被跨域的路径，可以任意配置，可以具体到直接请求路径。
 *      allowedMethods：允许所有的请求方法访问该跨域资源服务器，如：POST、GET、PUT、DELETE等。
 *      allowedOrigins：允许所有的请求域名访问我们的跨域资源，可以固定单条或者多条内容，			 如：“http://www.aaa.com”，只有该域名可以访问我们的跨域资源。
 *      allowedHeaders：允许所有的请求header访问，可以自定义设置任意请求头信息。
 * @author: BaiPang
 * @date: 2019/11/14 17:22
 */
 
@Configuration
public class CORSConfiguration extends WebMvcConfigurationSupport {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedMethods("*")
                .allowedOrigins("*")
                .allowedHeaders("*");
        super.addCorsMappings(registry);
    }
}
```



另外，还可以通过添加 Filter 的方式，配置 CORS 规则，并手动指定对哪些接口有效。

```java
@Bean
public FilterRegistrationBean corsFilter() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowCredentials(true);   config.addAllowedOrigin("http://localhost:9000");
    config.addAllowedOrigin("null");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");
    source.registerCorsConfiguration("/**", config); // CORS 配置对所有接口都有效
    FilterRegistrationBean bean = newFilterRegistrationBean(new CorsFilter(source));
    bean.setOrder(0);
    return bean;
}
```

##### 2、原理剖析

无论是通过哪种方式配置 `CORS`，其实都是在构造 `CorsConfiguration`。 一个 `CORS` 配置用一个 `CorsConfiguration`类来表示，它的定义如下：

```java
public class CorsConfiguration {
    private List<String> allowedOrigins;
    private List<String> allowedMethods;
    private List<String> allowedHeaders;
    private List<String> exposedHeaders;
    private Boolean allowCredentials;
    private Long maxAge;
}
```

`Spring` 中对 `CORS` 规则的校验，都是通过委托给 `DefaultCorsProcessor`实现的。

`DefaultCorsProcessor` 处理过程如下：

1. 判断依据是 `Header`中是否包含 `Origin`。如果包含则说明为 `CORS`请求，转到 2；否则，说明不是 `CORS` 请求，不作任何处理。
2. 判断 `response` 的 `Header` 是否已经包含 `Access-Control-Allow-Origin`，如果包含，证明已经被处理过了, 转到 3，否则不再处理。
3. 判断是否同源，如果是则转交给负责该请求的类处理
4. 是否配置了 `CORS` 规则，如果没有配置，且是预检请求，则拒绝该请求，如果没有配置，且不是预检请求，则交给负责该请求的类处理。如果配置了，则对该请求进行校验。

校验就是根据 `CorsConfiguration` 这个类的配置进行判断：

1. 判断 `origin` 是否合法
2. 判断 `method` 是否合法
3. 判断 `header`是否合法
4. 如果全部合法，则在 `response header`中添加响应的字段，并交给负责该请求的类处理，如果不合法，则拒绝该请求。



## 5、定时任务

### 1. `cron`表达式格式：

```
{秒数} {分钟} {小时} {日期} {月份} {星期} {年份(可为空)}
```

-  `“*”`字符代表所有可能的值。`“*”`在`{月份}`里表示每个月的含义。
-  `“/”`字符用来指定数值的增量。
   在`{分钟}` 里的`“0/15”`表示从第0分钟开始，每15分钟。
   在`{分钟}`里的`“3/20”`表示从第3分钟开始，每20分钟（它和“3，23，43”）的含义一样。
-  `“L”` 字符仅被用于`{日期}`和`{星期}`，它是单词`“last”`的缩写。
   在`{日期}`，“L”表示一个月的最后一天。
   在`{星期}`，“L”表示一个星期的最后一天，也就是`SAT`。
- 如果在`“L”`前有具体的内容，它就具有其他的含义了。
   `“6L”`表示这个月的倒数第６天，`“ＦＲＩＬ”`表示这个月的最一个星期五。
   **注意**：在使用“L”参数时，不要指定列表或范围，
- 由于`{日期}`和`{星期}`这两个元素互斥的, 其中之一被指定了值以后, 必须要对另一个设为`”?”`。

### 2. `cron`表达式各占位符解释：

- ###### {秒数}{分钟} ==> 允许值范围: 0~59 ,不允许为空值。

-  `“*”` 代表每隔1秒钟触发。

-  `“,”` 代表在指定的秒数触发。比如`”0,15,45”`代表`0秒、15秒和45秒`时触发任务。

-  `“-“`代表在指定的范围内触发，比如`”25-45”`代表`从25秒开始触发到45秒结束`触发，每隔1秒触发1次。

- ###### {小时} ==> 允许值范围: 0~23 ,不允许为空值，若值不合法。占位符和秒数一样。

- ###### {日期} ==> 允许值范围: 1~31 ,不允许为空值。

- ###### {月份} ==> 允许值范围: 0~11

- ###### {星期} ==> 允许值范围: 1~7 (或 SUN，MON，TUE，WED，THU，FRI，SAT), 1代表星期天(SUN)，以此类推，7代表星期六(SAT)，不允许为空值。

- ###### {年份} ==> 允许值范围: 1970~2099 ,允许为空。

### 3. 经典案例

`“30 * * * * ?”` 每半分钟触发任务

`“30 10 * * * ?”` 每小时的10分30秒触发任务
`“30 10 1 * * ?”` 每天1点10分30秒触发任务
`“0 0 10,14,16 * * ?”` 每天上午10点，下午2点，4点
`“0 0/30 9-17 * * ?”`   朝九晚五工作时间内每半小时
`“0 * 14 * * ?”` 在每天下午2点到下午2:59期间的每1分钟触发
`“0 0/5 14 * * ?”` 在每天下午2点到下午2:55期间的每5分钟触发
`“0 0/5 14,18 * * ?”` 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
`“0 0-5 14 * * ?”` 在每天下午2点到下午2:05期间的每1分钟触发

`“30 10 1 20 * ?”` 每月20号1点10分30秒触发任务
`“30 10 1 20 10 ? *”` 每年10月20号1点10分30秒触发任务

`“30 10 1 20 10 ? 2011”` 2011年10月20号1点10分30秒触发任务
`“30 10 1 ? 10 * 2011”` 2011年10月每天1点10分30秒触发任务
`“30 10 1 ? 10 SUN 2011”` 2011年10月每周日1点10分30秒触发任务

`“15,30,45 * * * * ?”` 每15秒，30秒，45秒时触发任务
`“15-45 * * * * ?”` 15到45秒内，每秒都触发任务
`“15/5 * * * * ?”` 每分钟的每15秒开始触发，每隔5秒触发一次
`“15-30/5 * * * * ?”` 每分钟的15秒到30秒之间开始触发，每隔5秒触发一次
`“0 0/3 * * * ?”` 每小时的第0分0秒开始，每三分钟触发一次
`“0 15 10 ? * MON-FRI”` 星期一到星期五的10点15分0秒触发任务

`“0 15 10 L * ?”` 每个月最后一天的10点15分0秒触发任务
`“0 15 10 LW * ?”` 每个月最后一个工作日的10点15分0秒触发任务
`“0 15 10 ? * 5L”` 每个月最后一个星期四的10点15分0秒触发任务
`“0 15 10 ? * 5#3”` 每个月第三周的星期四的10点15分0秒触发任务



### 4.fixedRate 

`@Scheduled` 参数可以接受两种定时的设置，一种是我们常用的`cron="*/6 * * * * ?"`,一种是 `fixedRate = 6000`，两种都表示每隔六秒打印一下内容。

**fixedRate 说明**

- `@Scheduled(fixedRate = 6000)` ：上一次开始执行时间点之后6秒再执行
- `@Scheduled(fixedDelay = 6000)` ：上一次执行完毕时间点之后6秒再执行
- `@Scheduled(initialDelay=1000, fixedRate=6000)` ：第一次延迟1秒后执行，之后按fixedRate的规则每6秒执行一次

