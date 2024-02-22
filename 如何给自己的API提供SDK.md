# 如何给自己的API提供SDK

> 注：本文基于[API签名认证的实现方案 | twigicer's blog (xiaoshuzhi.love)](https://www.xiaoshuzhi.love/2024/01/30/study-7/)，模拟接口这部分请阅读上篇文章，本文不再讲述。

在上一篇文章：[API签名认证的实现方案 | twigicer's blog (xiaoshuzhi.love)](https://www.xiaoshuzhi.love/2024/01/30/study-7/)中，编写了模拟接口并进行了模拟调用，也实现了API签名认证保证接口安全性。但是在实际应用中，我们提供的接口应该最大程度的方便开发者调用，例如：不应该让开发者去实现API签名认证的。开发者重点关注的是接口的功能而不是接口的实现，所以我们应该为开发者提供接口SDK，让开发者只需要配置必要参数就可以调用接口。

## SDK简述：

### 什么是SDK?

> 以下内容来自于CHatGPT：
>
> SDK (软件开发工具包) 是一组用于开发特定软件的工具和资源的集合。SDK 包含编译器、库、文档、示例代码和其他支持开发者创建特定软件的组件。SDK 提供了开发者所需的一切，以便他们能够编写、调试和测试软件应用程序。

个人理解：SDK就类似于Utils工具类，简化开发者的开发。

### SDK和API的区别？

个人理解：API是接口，供开发者调用，而SDK是对API的封装，更加便于开发者调用API.

> 以下内容来自于CHatGPT：
>
> SDK（Software Development Kit，软件开发工具包）是一种包含开发特定软件所需工具、库、文档和示例代码的集合。SDK提供了开发者所需的一切，以便他们能够更轻松地创建特定类型的软件。
>
> API（Application Programming Interface，应用程序编程接口）是一组定义了软件组件之间交互方式的规则和协议。API允许应用程序与其他软件组件（如操作系统、库或服务）进行通信和交互。API定义了如何使用和访问特定软件或服务的功能，而不需要了解其内部实现。
>
> SDK主要是为了支持软件开发过程，提供工具和资源来编译、构建、测试和调试应用程序。SDK通常包含API，以便开发者能够使用已经定义好的函数和方法来实现特定软件的功能。
>
> API更侧重于软件组件之间的交互和通信，定义了如何使用和访问特定软件或服务的功能。API描述了使用特定软件或服务提供的函数、类、方法、参数和返回值，使开发者能够利用这些功能进行应用程序的构建。
>
> 总结来说，SDK提供开发工具和资源，帮助开发者创建特定类型的软件，而API定义了软件组件之间的交互规则和访问方式，使得应用程序能够与其他组件进行通信和使用功能。

### 怎么开发SDk?

本文将基于Spring boot starter去实现SDK，差不多就是开发一个starter，类似于：

```java
   		<dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>
```

将上篇文章中的模拟接口、模拟调用和API签名认证功能改为一个依赖包，在pom文件中引入即可调用项目的功能。

### 使用SDK（starter）的好处？

主要就是简化开发，不需要去关注一些不必要的逻辑即可实现功能，在配置参数时也会有一些代码提示。

> 以下内容来自ChatGPT：
>
> 1. **模块化和可插拔性**： Starter 提供了一种将应用程序拆分成模块的方式。通过将相关功能和配置封装到 Starter 中，可以使应用程序更加模块化，让各个模块之间更加独立、可插拔。这使得你可以更轻松地根据需求添加或删除功能。
> 2. **简化配置**： Starter 可以帮助用户轻松配置和集成某个功能。通过提供默认配置和自定义选项，用户可以很容易地调整功能的行为，而无需手动进行复杂的配置。
> 3. **减少样板代码**： Starter 可以封装一些常用的代码模板，使得开发者不再需要从头编写重复的代码。这有助于提高代码的复用性，并减少出错的机会。
> 4. **易于维护和更新**： 通过将功能封装到 Starter 中，维护者可以更轻松地管理和更新相关代码。这有助于保持项目的健康状态，确保应用程序始终使用最新的、安全的功能。
> 5. **降低学习曲线**： Starter 通常提供清晰的文档和示例，帮助使用者更快速地上手。这对于新加入项目的开发者或初学者来说是非常有益的，因为他们可以迅速了解如何使用和配置某个功能。
> 6. **社区共享和标准化**： Starter 的使用促进了开发社区中功能的共享和标准化。开发者可以更容易地找到并使用来自社区的 Starter，从而加速项目的开发进程。
> 7. **提高开发效率**： 使用 Starter 可以显著提高开发效率。通过引入现成的功能模块，开发者可以专注于业务逻辑，而无需花费大量时间在基础设施和集成细节上。

## 基于Springboot starter开发SDK:

### 1. 新建spring项目:

新建一个springboot项目，type选Maven：

![image-20240131120923829](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131120923829.png)

选择自己需要的依赖，这里演示只需要Lombok，除了项目需要的依赖外，还需要添加："spring-boot-configuration-processor"依赖，这个依赖可以自动为我们生成代码提示：

![](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-01-31%20121020.png)

### 2. 修改pom文件：

进入pom文件，修改版本号（也可以不修改，改为0.0.1方便一点，看个人喜好）：

![image-20240131121803092](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131121803092.png)

将<build></build>这块删掉，这块是meven项目构建时用的，而我们开发的SDK不需要，所以直接删掉。

![image-20240131122134962](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131122134962.png)

引入spring初始化模板中没有但是项目需要的依赖，这里需要Hutool依赖，所以直接引入，刷新。

### 3. 删掉Application启动类：

因为不需要启动，所以直接将Application 类删掉：

![image-20240131122351687](C:\Users\周斌\AppData\Roaming\Typora\typora-user-images\image-20240131122351687.png)

### 4. 实现功能接口：

这里就实现上篇文章中的模拟接口、模拟接口调用、API签名认证，基本上不需要改动什么代码，直接复制粘贴上篇文章项目里的client、model、utils：

client客户端接口调用:

```java
public class TwigApiClient {

    private String accessKey;
    private String secretKey;

    public TwigApiClient(String accessKey, String secretKey) {
        this.accessKey = accessKey;
        this.secretKey = secretKey;
    }


    private Map<String,String> getHeaderMap(String body){
        Map<String,String> headerMap = new HashMap<>();
        headerMap.put("accessKey",accessKey);
        //一定不能直接发送
//        headerMap.put("secretKey",secretKey);
        headerMap.put("body",body);
        headerMap.put("nonce", RandomUtil.randomNumbers(4));
        headerMap.put("timestamp",String.valueOf(System.currentTimeMillis()/1000));
        headerMap.put("sign", SignUtils.genSign(body,secretKey));

        return headerMap;
    }

    public String getUserNameByPost(User user){
        String json = JSONUtil.toJsonStr(user);
        HttpResponse httpResponse = HttpRequest.post("http://localhost:8201/api/name/user")
                .body(json)
                .addHeaders(getHeaderMap(json))
                .execute();
        System.out.println(httpResponse.getStatus());
        String res = httpResponse.body();
        System.out.println(res);
        return res;
    }
}
```

model的User：

```java
@Data
public class User {
    private String userName;
}
```

utils的签名生成算法：

```java
public class SignUtils {
    /**
     * 生成签名
     * @param body
     * @param secretKey
     * @return
     */
    public static String genSign(String body,String secretKey){
        Digester digester = new Digester(DigestAlgorithm.SHA256);
        String content = body + "." + secretKey;
        return digester.digestHex(content);
    }
}
```

### 5. 写ClientConfig 配置类：

SDK需要一个配置类来描述实例的生成配置，用@Bean产生对象并交给spring管理，@Configuration表明这是个配置类并交于spring管理，@ComponentScan允许包扫描，@Data生成构造函数，比较重要的是：@ConfigurationProperties("twig.client")用于将配置文件中"twig.client"开头的属性绑定到POJO中，也就是在后面我们调用这个依赖写配置文件时会给我们提示。

```java
@Configuration
@ConfigurationProperties("twig.client")
@ComponentScan
@Data
public class TwigApiClientConfig {
    private String accessKey;
    private String secretKey;

    @Bean
    public TwigApiClient twigApiClient(){
        return new TwigApiClient(accessKey,secretKey);
    }
}
```

### 6. 新建META-INF目录与spring.factories文件：

在resources目录下新建META-INF目录和spring.factories文件：

![image-20240131124541204](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131124541204.png)

并在spring.factories文件中写入：

```txt
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.twigicer.twigapiclientsdk.TwigApiClientConfig
```

这块涉及springboot自动配置的原理，就是为了spring可以扫描到第三方类，可以参考：

[springboot自动配置原理以及spring.factories文件的作用_springboot自动配置是读取什么文件-CSDN博客](https://blog.csdn.net/nsplnpbjy/article/details/106465719)

[springboot核心基础之spring.factories机制 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/444331676)

### 7. 使用meven打包：

使用maven的install或者package进行打包：

![image-20240131125449789](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131125449789.png)

这个时候会报错：

![image-20240131125559789](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131125559789.png)

很正常，因为我们删除了Application启动类，但是在test包中还有这个类的测试类，而在meven打包前有个test步骤，会走一次测试，如果测试类里有报错，或者测试失败都会导致打包失败。

解决方法：

1. 直接删除test包或者test包中的ApplicationTests启动测试类:

   ![image-20240131125949650](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131125949650.png)

2. 点击下面这个按钮，排除meven生命周期的test阶段：

   ![image-20240131130207282](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131130207282.png)

再次打包，看到提示"build success"，打包成功：

![image-20240131130309129](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131130309129.png)

到此，SDK开发结束。

## 调用测试SDK:

在接口项目中调用测试一下该SDK是否生效：

### 1. 引入SDK依赖：

在接口项目的pom文件中引入刚才打好的包（就是引入SDK项目pom文件中的groupId、artifactId和version）：

```java
		<dependency>
            <groupId>com.twigicer</groupId>
            <artifactId>demo-sdk</artifactId>
            <version>0.0.1</version>
        </dependency>
```

刷新。

### 2. 修改配置文件的AK/SK：

在yml文件里配置accessKey和secretKey，可以看到这时会有代码提示：

![image-20240131131251659](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131131251659.png)

### 3. 修改模拟接口的User、SignUtil路径为依赖包路径：

```
import com.twigicer.demosdk.model.User;
import com.twigicer.demosdk.utils.SignUtils;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/name")
public class NameController {

    @PostMapping("/user")
    public String getUserNameByPost(@RequestBody User user, HttpServletRequest request){
        String accessKey = request.getHeader("accessKey");
        String nonce = request.getHeader("nonce");
        String timestamp = request.getHeader("timestamp");
        String sign = request.getHeader("sign");
        System.out.println(sign);
        String body = request.getHeader("body");
        if(!accessKey.equals("twig") ){
            throw new RuntimeException("无权限");
        }
        if(Long.parseLong(nonce) > 10000){
            throw new RuntimeException("无权限");
        }
        // TODO 校验时间和当前时间相差不能超过5分钟
//        if(timestamp ...){
//
//        }
        String visitSign = SignUtils.genSign(body, "abcdefgh");
        if(!sign.equals(visitSign)){
            throw new RuntimeException("无权限");
        }
        return "Post 你的名字是：" + user.getUserName();
    }
}
```

### 4. 编写测试类：

编写测试类进行测试：

```java
   @Resource
    private TwigApiClient twigApiClient;

    @Test
    void contextLoads() {
        User user = new User();
        user.setUserName("twig");
        String usernameByPost = twigApiClient.getUserNameByPost(user);
        System.out.println(usernameByPost);
    }
```

测试结果，调用成功：

![image-20240131132758399](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131132758399.png)

## 打好的包放在哪？

到这有没有疑惑，为什么打的包在别的项目中可以直接引入？打好的包放在了哪里？

这个包放在了本地的仓库中，一般在C盘的用户文件夹里的.m2文件里，但是我的放在了E盘，我也不太清楚为什么。

![image-20240131133528849](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240131133528849.png)

如果想让别人使用这个依赖包，可以直接把jar包给他，也可以将jar包上传到meven中心仓库中，至于怎么上传，暂时不太了解，可以自行百度。

## 总结：

开发SDK有7步：创项目 -> 修改pom -> 删除启动类 -> 实现功能 -> 写配置类 -> 创建spring.factories文件 -> 打包。注意打包时排除掉test阶段。

