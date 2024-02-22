# API签名认证的实现方案

在搭建这个小破站的过程中调用了一些第三方API，比如公告栏的欢迎就调用了腾讯地图的API；如果想加天气可以去调用和风天气的API；为了防止用户频繁调用这些API，也为了做一些统计和计费，所以在调用这些API时都需要我们去申领一个Key，而这个Key就是本文要说的签名。

## API签名认证简介：

### 什么是API签名认证？

>以下内容来自ChatGPT：
>
>API签名认证是一种用于验证API请求的身份和完整性的安全机制。在使用API时，通常需要发送HTTP请求来获取或操作数据。为了确保请求的合法性和安全性，API签名认证采用了一种算法（例如HMAC-SHA256），使用密钥对请求参数和其他相关信息进行哈希或加密，生成一个唯一的签名字符串。接收方在收到API请求后，使用相同的算法和密钥对请求参数进行处理，并与请求中的签名字符串进行比较。如果两者一致，则表明请求是合法的，否则请求将被认为是伪造或篡改的。这种认证方法可以有效防止恶意请求和数据篡改，提高API的安全性。

个人理解：类似于session，在用户登录时为用户分发一个session,需要做一些操作时，需要根据session去鉴权；而API签名认证类似，为用户分发一个签名，在用户调用API时，会判断签名是否正确去鉴权，判断是否允许调用。

### 为什么需要API签名认证？

主要是防止API被无限制的调用，保证接口的安全性，看看专业术语：

> 以下内容来自ChatGPT：
>
> 1. 身份验证：API签名认证可以验证请求的发送者是否具有合法的身份。通过使用密钥和算法对请求进行签名，可以确保只有具有有效密钥的授权用户才能发送请求。这样可以防止未经授权的用户访问和使用API。
> 2. 完整性验证：API签名认证可以验证请求的完整性，即请求参数是否被篡改。通过将请求参数与签名字符串进行比较，可以确定请求是否在传输过程中被篡改。这可以防止中间人攻击和数据篡改。
> 3. 防止重放攻击：API签名认证可以防止恶意用户重放之前的有效请求。通过在签名过程中包含时间戳或随机数等参数，可以确保请求不能被多次重放。这有助于提高系统的安全性和稳定性。
> 4. 数据保护：API签名认证可以保护API传输的数据。通过使用加密算法对请求进行签名，可以防止敏感数据在传输过程中被窃取或篡改。

## API签名认证的实现：

### 1. **AccessKey**和**SecretKey**实现：

主要步骤就两个：签发签名和校验签名。

为开发者分配**AccessKey**和**SecretKey**，为了安全，尽量复杂，无序，无规律。

AccessKey：调用的标识：userA，userB

SecretKey：密钥

类似于账号和密码，但是用户名和密码只需要登录一次就可以在后续操作中可以一直使用，但是**AccessKey**和**SecretKey**是无状态的，每次使用都需要带着。

在用户调用接口时，需要在请求头中发送签名，然后查询数据库，判断签名是否一致，一致则允许调用，不一致则不允许调用。

#### 代码实现

> 为了方便演示，先将Accesskey和SecretKey写死为"twig"和"abcdefgh"。实际应该在数据库的用户表中插入这两个字段，然后为用户签发签名后将两个字段存入数据库，校验时从库里查询是否一致。

编写一个User类，方便传参，只写一个userName，使用Lombok：

```java
@Data
public class User {
    private String userName;
}
```



编写模拟接口并添加签名校验的逻辑：

```java
@RestController
@RequestMapping("/name")
public class NameController {
    @PostMapping("/user")
    public String getUserNameByPost(@RequestBody User user, HttpServletRequest request){
        String accessKey = request.getHeader("accessKey");
        String secretKey = request.getHeader("secretKey");
        
        //实际使用时在数据库中查询accessKey和secretKey并校验
        if(!accessKey.equals("twig") || !secretKey.equals("abcdefgh")){
            throw new RuntimeException("无权限");
        }
        return "Post 你的名字是：" + user.getUserName();
    }
}
```

编写调用第三方接口的客户端,，使用了Hutool工具包的http请求工具：

```java
public class TwigApiClient {

    private String accessKey;
    private String secretKey;

    public TwigApiClient(String accessKey, String secretKey) {
        this.accessKey = accessKey;
        this.secretKey = secretKey;
    }

    // 请求头参数
    private Map<String,String> getHeaderMap(){
        Map<String,String> headerMap = new HashMap<>();
        headerMap.put("accessKey",accessKey);
        headerMap.put("secretKey",secretKey);
        return headerMap;
    }

    //调用接口
    public String getUserNameByPost(@RequestBody User user){
        String json = JSONUtil.toJsonStr(user);
        HttpResponse httpResponse = HttpRequest.post("http://localhost:8201/api/name/user")
                .body(json)
                .addHeaders(getHeaderMap())
                .execute();
        //打印状态码
        System.out.println(httpResponse.getStatus());
        String res = httpResponse.body();
        System.out.println(res);
        return res;
    }
}
```

编写测试类，这里直接用main方法测试，在创建客户端对象时传入accessKey和secretKey，先传入正确的"twig"和"abcdefgh"：

```java
public class Main {
    public static void main(String[] args) {
        //在创建客户端对象时传入accessKey和secretKey
        TwigApiClient twigApiClient = new TwigApiClient("twig","abcdefgh");
        User user = new User();
        user.setUserName("twigicer");
        String res3 = twigApiClient.getUserNameByPost(user);
        System.out.println(res3);
    }
}
```

可以看到上面传入了正确的accessKey和secretKey，所以可以正常调用到接口：

![image-20240130134831198](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240130134831198.png)

如果在创建客户端时,传入错误的accessKey或者secretKey，则调用失败：

![image-20240130135108182](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240130135108182.png)

## 2. 改进方案：

上面的方法是不安全的，因为直接把accessKey和secretKey放在了请求头里，但是请求是可能被拦截的，被拦截之后，就可以使用我们的accessKey和secretKey去发请求，所以我们**一定不能把密钥直接在服务器之间传递！！！**

我们可以将accessKey和secretKey通过签名算法加密为sig(签名)，传递sign而不传递accessKey和secretKey。

accessKey + secretKey => **签名生成算法** => 不可解密的sign (签名)<br>twig + abcdefgh => ahcch16763661hcdgykl（不可解密）

### sign是不可解密的，那怎么去做校验呢？

类似于数据库用户名和密码的加密和解密，因为accessKey和secretKey是服务端颁发的，记录在数据库里，服务端在校验时，查询到accessKey和secretKey，再利用上述同样的签名算法进行加密后，对比与sign是否一致即可。

### 重放风险：

但是上面这种方法还存在被重放的风险。

#### 什么是重放？

> 以下内容来自ChatGPT:
>
> 重放攻击是一种网络安全攻击，其基本原理是攻击者截获网络通信中的数据包，并在稍后的某个时间重新发送这些数据包，以模拟合法用户的行为。这种攻击通常针对那些未经充分保护的通信协议或系统，攻击者可以通过重放合法用户的认证凭证或请求来获取未经授权的访问权限或执行某些未经授权的操作。
>
> 重放攻击可以针对各种类型的系统和协议，包括但不限于网络身份验证系统、API接口、加密通信等。攻击者通过截获合法用户的认证凭证、会话令牌或其他敏感数据，然后重新发送这些数据以获取对系统的未经授权访问。例如，在网络身份验证中，攻击者可能截获用户的用户名和密码，然后将这些凭据重新发送到服务器以获得对用户账户的访问权限。

简单来说重放就是：重复使用请求参数伪造二次请求。

实现重放也很简单，如下图，点击"重播XHR"，就可以再发一次请求：

![image-20240130142504596](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240130142504596.png)

#### 怎么防重放？

1. 加nonce随机数，每个随机数只能使用一次，弊端是服务端需要记录每次的随机数。
2. 加timestamp时间戳，校验时间戳是否过期，弊端是再时间戳未过期时还是会被重放。
3. nonce和timestamp配合使用，在时间戳未过期时加随机数校验，时间戳过期时清空随机数，这样可以解决单一使用时的弊端。

参考文章：[开放API接口签名验证，让你的接口从此不再裸奔 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/220033777)

> nonce指唯一的随机字符串，用来标识每个被签名的请求。通过为每个请求提供一个唯一的标识符，服务器能够防止请求被多次使用（记录所有用过的nonce以阻止它们被二次使用）。
>
> 然而，对服务器来说永久存储所有接收到的nonce的代价是非常大的。可以使用timestamp来优化nonce的存储。
>
> 假设允许客户端和服务端最多能存在15分钟的时间差，同时追踪记录在服务端的nonce集合。当有新的请求进入时，首先检查携带的timestamp是否在15分钟内，如超出时间范围，则拒绝，然后查询携带的nonce，如存在已有集合，则拒绝。否则，记录该nonce，并删除集合内时间戳大于15分钟的nonce（可以使用redis的expire，新增nonce的同时设置它的超时失效时间为15分钟）。

## 3. 最终实现方案：

所以一次标准的API签名认证实现，需要上述的所有参数和用户的请求参数：

* 参数1：AccessKey
* 参数2：SecretKey  //该参数不传递到请求头中
* 参数3：用户的请求参数
* 参数4：sign
* 参数5：nonce
* 参数6：timestamp

#### 代码实现：

> 为了方便演示，先将Accesskey和SecretKey写死为"twig"和"abcdefgh"。实际应该在数据库的用户表中插入这两个字段，然后为用户签发签名后将两个字段存入数据库，校验时从库里查询是否一致。

在请求头中放入上述参数，注意：secretKe一定不能直接放在请求头中！！！！

```java
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
```

编写一个签名生成算法类（这里使用的时huTool的工具包）：

```java
    public static String genSign(String body,String secretKey){
        Digester digester = new Digester(DigestAlgorithm.SHA256);
        String content = body + "." + secretKey;
        return digester.digestHex(content);
    }
```

调用接口方法和之前一样：

```java
    public String getUserNameByPost(@RequestBody User user){
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
```

在接口中校验参数和签名，时间戳校验可以使用redis，注意：这里visitSign生成方法要和之前sign生成方法一样：

```java
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
```

编写测试类，和之前一样直接用mian方法测试，先传入正确的"twig"和"abcdefgh"：

```java
public class Main {
    public static void main(String[] args) {
        //在创建客户端对象时传入accessKey和secretKey
        TwigApiClient twigApiClient = new TwigApiClient("twig","abcdefgh");
        User user = new User();
        user.setUserName("twigicer");
        String res3 = twigApiClient.getUserNameByPost(user);
        System.out.println(res3);
    }
}
```

可以看到上面传入了正确的accessKey和secretKey，所以可以正常调用到接口：

![](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-01-30%20163836.png)

如果在创建客户端时,传入错误的accessKey或者secretKey，则调用失败：

![image-20240130165055136](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240130165055136.png)

## 总结：

API签名认证是一个很灵活的设计，本文只是记录较为规范的实现方式，具体实现需要根据实际业务场景去设计。

参考文章：

[开放API接口签名验证，让你的接口从此不再裸奔 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/220033777)

鱼皮老哥：[主页 - 编程导航 (code-nav.cn)](https://www.code-nav.cn/)