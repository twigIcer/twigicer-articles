# Springboot3.x整合Swagger

最近做了个小调查，问了我周围的Java伙伴们他们的spring和springboot的版本，发现居然没人用最新的spring6.x和springboot3.x，想想也正常，毕竟程序员们都听过“用旧不用新”的忠告，不过呢，我个人是比较喜欢探索新事物的，所以在spring6.x和springboot3.x刚出来不久就开始使用了，也因此吃了很多版本不兼容的亏，我的博客里就记录了好几次，这不今天又喜提一个——Springboot3.x整合Swagger。

## Swagger简介：

Swagger 是一个规范和工具集，用于设计、构建、文档化和使用 RESTful 风格的 Web 服务。它提供了一套简洁易用的接口描述语言（OpenAPI Specification）和交互式文档、代码生成器、测试工具等。

以下是 Swagger 的一些主要特点和优势：

1. 接口描述语言（OpenAPI Specification）: Swagger 使用基于 JSON 或 YAML 的接口描述语言来定义 API 的结构、参数、返回值等信息。这样可以使开发人员更好地理解和使用接口，提高开发效率。
2. 交互式文档：Swagger 自动生成交互式的 API 文档，通过 Web 界面提供了一种直观的方式来查看和测试 API。它包含了所有接口的详细说明、参数验证规则、示例请求和响应等信息，可以方便开发人员和其他团队成员快速了解和使用 API。
3. 代码生成器：Swagger 可以根据接口描述文件自动生成客户端代码，支持多种语言和框架，包括 Java、Python、JavaScript、Go 等。这样可以加速客户端的开发过程，并保持与服务端的接口定义一致性。
4. API 测试工具：Swagger 提供了一个内置的 API 测试工具，可以直接在文档页面上对 API 进行测试。开发人员可以通过该工具发送请求，并查看请求和响应的详细信息，从而快速验证 API 的正确性和稳定性。
5. 生态系统丰富：Swagger 拥有一个庞大的开源社区和丰富的生态系统，提供了许多与 Swagger 兼容的工具和插件。这些工具可以与各种开发框架和平台集成，为开发人员提供更多的选择和便利。

简单来说，Swagger就是一个接口测试工具，与PostMan、APIpost类似，不过由于Swagger具有代码侵入性，所以其实是不建议用的，不过在卷到飞起的今天，该掌握还是得掌握的。

## 编写一个简单的测试接口：

首先编写一个测试接口，这里就写了个很简单的接口：

```java
@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/get")
    public String get(){
        System.out.println("测试");
        return "测试";
    }
}
```

## Springboot2.x整合Swagger：

我最开始用的是Springboot2.x整合Swagger的方式，结果由于版本不兼容出现了一些问题，但是方法是对的，也就记录一下吧。

其实Java整合各种组件的基本流程就是：引依赖->写配置类->在配置文件里开启，那就从头开始说说吧。

### 1. 引依赖：

Swagger需要的依赖就下面两个，一个是UI界面依赖，一个核心依赖。

```java
 		<dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.7.0</version>
        </dependency>
```

### 2. 写配置类：

新建一个SwaggerConfig的类，代码如下：

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.bin.swaggerdemo.controller"))
                .paths(PathSelectors.any())
                .build()
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("接口文档")
                .description("接口文档")
                .version("1.0.0")
                .build();
    }
}
```

在 api() 方法中，我们通过 `select() 方法配置扫描的包路径，paths() 方法配置接口的访问路径，apiInfo() 方法配置接口文档的相关信息，@Configuration 表示该类是一个配置类，@EnableSwagger2 表示启用 Swagger。

### 3. 写配置文件：

Springboot2.x整合Swagger,如果不需要自定义配置上面的title、description、version这些内容，就不需要在配置文件里写什么。

## 版本不兼容导致的问题：

### 1. 问题描述：

如果是Springboot2.x到这就已经整合好了，可惜我用的是Springboot3.x，然后就报错啦。

```java
Caused by: java.lang.NoClassDefFoundError: org/springframework/util/comparator/InvertibleComparator
	at org.springframework.plugin.core.OrderAwarePluginRegistry.<clinit>(OrderAwarePluginRegistry.java:45) ~[spring-plugin-core-1.2.0.RELEASE.jar:na]
	at org.springframework.plugin.core.support.PluginRegistryFactoryBean.getObject(PluginRegistryFactoryBean.java:36) ~[spring-plugin-core-1.2.0.RELEASE.jar:na]
	at org.springframework.plugin.core.support.PluginRegistryFactoryBean.getObject(PluginRegistryFactoryBean.java:28) ~[spring-plugin-core-1.2.0.RELEASE.jar:na]
	at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean(FactoryBeanRegistrySupport.java:148) ~[spring-beans-6.0.11.jar:6.0.11]
	... 71 common frames omitted
Caused by: java.lang.ClassNotFoundException: org.springframework.util.comparator.InvertibleComparator
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:641) ~[na:na]
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:188) ~[na:na]
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521) ~[na:na]
	... 75 common frames omitted
```

这日志就不解读了（其实我也看不太懂，咳咳），不过百度了一下，也没找到好的解决方案。

### 2. 探究过程：

检查了接口代码、配置类代码确保没问题，我就想到可能因为依赖的版本低不兼容了，其实我在mvn仓库引依赖时就发现springfox-swagger2这依赖2020年后就不更新了，我居然天真的认为是技术已经很成熟了，然后傻乎乎的引入了上面最新的3.0版本的依赖：

```java
 		<dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>3.0.0</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>3.0.0</version>
        </dependency>
```

还是报错，不过这次的报错信息短了很多：

```java
Caused by: java.lang.ClassNotFoundException: javax.servlet.http.HttpServletRequest
	at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:641) ~[na:na]
	at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:188) ~[na:na]
	at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521) ~[na:na]
	at java.base/java.lang.Class.forName0(Native Method) ~[na:na]
	at java.base/java.lang.Class.forName(Class.java:496) ~[na:na]
	at java.base/java.lang.Class.forName(Class.java:475) ~[na:na]
	at java.base/sun.reflect.generics.factory.CoreReflectionFactory.makeNamedType(CoreReflectionFactory.java:114) ~[na:na]
	... 32 common frames omitted
```

其实经验告诉我，每次报错信息中有“~[spring-boot-3.1.2.jar:3.1.2]”这种，差不多就是spring/springboot版本不兼容导致的问题，然后喝了杯水冷静了一下，搜索了“springboot3整合Swagger”，得到了答案。

## springboot3整合Swagger：

还是那几步。

### 1. 引依赖：

因为集成SpringFox只支持SpringBoot2.x，而基于Swagger的SpringDoc的社区现在十分活跃，版本不断更新。SpringFox自从2020年7月14号之后就不更新了，官方建议是springdoc替代springfox。

```java
 		<dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.0.4</version>
        </dependency>
        <!-- 官方建议是springdoc替代springfox-->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-api</artifactId>
            <version>2.0.4</version>
        </dependency>
```

### 2. 写配置类：

SpringDoc的配置类比上面springfox的配置类简单些：

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI springShopOpenAPI() {
        return new OpenAPI()
                .info(new Info().title("Swagger")
                        .description("abin")
                        .version("v1")
                        .license(new License().name("Apache 2.0").url("http://springdoc.org")))
                .externalDocs(new ExternalDocumentation()
                        .description("外部文档")
                        .url("https://springshop.wiki.github.org/docs"));
    }
}
```

也还是配置了上面的哪些信息，springdoc里不需要@EnableSwagger2注解。

### 3. 写配置文件：

使用springdoc需要在配置文件里指定swagger的ui界面访问路径：

```java
springdoc:
  swagger-ui:
    path: /swagger-ui.html
```

### 4. 启动项目测试：

启动项目，没有报错，浏览器访问http://localhost:8080/swagger-ui/index.html#/，出现Swagger接口测试界面：

![image-20230811094036252](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20230811094036252.png)

选择我们创建好的test/get接口，点击“try it out”进行测试，响应200，表示请求成功，到此swagger整合完成。

## 总结：

本文只是简单介绍了Springboot整合swagger的方式，Swagger还有一些高级应用，可以阅读官方文档了解，springboot2.x与springboot3.x中很多组件的整合方式都不同，需要整合时，先看看官方文档，可以少走很多弯路。最后，你们的spring/springboot是什么版本呢？
