# 如何解决Twikoo跨域问题

> 注意：解决这个问题只需要去twikoo的管理界面，将跨域配置修改为自己的域名即可，我是最开始填错了，导致废了这么大功夫。。。顺便水了一篇文章。

前面写了一篇跨域问题的解决方案的文章，那篇文章是看了鱼皮老哥视频总结的理论，但是没有实践过，刚好我的博客接入Twikoo评论系统时出现了跨域问题，这几天想着去解决一下这个问题，过程比较艰辛，记录一下。

## 问题描述：

本博客使用的是hexo的butterfly主题，github + Vercel托管，绑定了自定义域名。Twikoo系统使用 Vercel 云函数部署了并绑定了自定义域名。博客在接入Twikoo系统后，本地测试没有问题，可以正常发送评论并推送到邮箱，但是将博客推送上线后，再使用评论时，就会出现跨域问题：

```bash
Access to XMLHttpRequest at 'twikoo自定义域名' from origin '我的博客域名' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

## 错误方案：

### 修改请求头：

> 注意：解决跨域问题，需要在响应头中加入"Access-Control-Allow-Origin":"*"，而不是请求头！！！
>
> 缤纷云：[Bitiful](https://console.bitiful.com/buckets)

因为之前对跨域问题的解决方案理解不够深入，认为解决跨域问题可以在请求头中加入` "Access-Control-Allow-Origin":"*"`即可。

费了点力气去寻找到了Twikoo的main.js文件，然后下载下来，本地修改加入了` "Access-Control-Allow-Origin":"*"`请求头：

![image-20240216141817060](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216141817060.png)

然后将这个js文件上传到缤纷云上，再调用这个js文件，结果发现，请求头中是带上了"Access-Control-Allow-Origin":"*"，但是还是报跨域的错。

我又去找了文章，发现解决跨域是需要响应头带上Access-Control-Allow-Origin而不是请求头，我也意识到上篇文章写的有点问题，去修改了。

## 可能有用的方案：

### 1. 使用代理服务器发送请求：

我就想能不能再发请求的时候带上代理，进行请求转发，于是我去问了ChatGPT，ChatGPT的回答如下：

>如果你没有自己的代理服务器，也可以考虑使用一些在线的跨域请求代理服务来解决跨域问题。这些服务将充当中间人，将请求发送到目标服务器并将响应返回给你的应用程序。
>
>以下是一些常用的在线跨域请求代理服务：
>
>1. CORS Anywhere：https://cors-anywhere.herokuapp.com/
>2. Ycors：https://ycors.com/
>3. AnyProxy：https://anyproxy.io/
>4. CORS Proxy：https://cors-proxy.htmldriven.com/
>
>你可以将你的目标服务器URL放在这些代理服务的URL前面，将请求发送到代理服务，代理服务再将请求发送到目标服务器，并将目标服务器的响应返回给你的应用程序。请注意，这些代理服务可能有使用限制或存活时间限制，所以请查阅相关文档以了解更多详情。
>
>下面是一段示例代码，展示如何在你的代码中使用 CORS Anywhere 代理解决跨域问题：
>
>```js
>try {
>  var i = localStorage.getItem("twikoo-access-token");
>  var o = new XMLHttpRequest();
>  o.onreadystatechange = function () {
>    if (4 === o.readyState) {
>      if (200 === o.status) {
>        var n = JSON.parse(o.responseText);
>        n.accessToken && localStorage.setItem("twikoo-access-token", n.accessToken);
>        e({ result: n });
>      } else {
>        t(o.status);
>      }
>    }
>  };
>
>  // 使用 CORS Anywhere 代理发送请求
>  var proxyUrl = 'https://cors-anywhere.herokuapp.com/' + a;
>  o.open("POST", proxyUrl);
>  o.setRequestHeader("Content-Type", "application/json");
>  o.send(JSON.stringify(c({ event: n, accessToken: i }, r)));
>} catch (s) {
>  t(s);
>}
>```
>
>请注意，这些在线代理服务可能会限制访问频率或提供有限的并发连接数。如果你的应用程序需要持续高频率地使用跨域请求，最好考虑搭建自己的代理服务器来更好地控制和管理跨域请求。

然后我就照着替换了这段代码，然后重新上传到缤纷云，但是好像代理并没有起作用，请求还是和原来一样。

我觉得这个方案是可行的，但是可能由于我的配置不太对，所以代理没有起作用。如果有佬了解这一块可以补充。

### 2. 使用CDN，添加跨域配置：

> 国内大部分CDN都需要加速域名备案，因为我没有服务器，所以Twikoo的域名备不了案，如果有可以加速未备案的域名的CDN可以推荐一下。
>
> 这个方法本人并没有尝试成功，我觉得可能是CDN的问题，感兴趣的朋友可以换别的CDN试试。
>
> 缤纷云：[Bitiful](https://console.bitiful.com/buckets)

这个方案是我之前在某个帖子中看到的，刚好看见缤纷云出了CDN加速功能，就想着去试试。

#### 操作步骤：

首先，将Vercel里绑定的自定义域名去掉，然后在缤纷云的CDN处添加新域名和回源地址和回源HOST（就是Vercel提供的原始域名）：

![image-20240216151222217](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216151222217.png)

然后点击设置-访问控制-跨域访问配置-开启，配置博客网站ip或域名：

![image-20240216151408012](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216151408012.png)

等待部署完成即可。

到这一步跨域问题已经解决。

#### 其他问题1：域名未配置证书，访问时被安全拦截

> twikoo.xiaoshuzhi.love这个域名的SSL已经配置，忘记截图了，所以用valine.xiaoshuzhi.love来演示。

正常到上面这一步应该已经完成了，但是由于我的域名SSL证书失效了，所以会导致访问域名时被浏览器安全拦截：

![image-20240216161726926](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216161726926.png)

![image-20240216152054516](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216152054516.png)

这时需要去申请SSL证书，腾讯云可以免费申请1年的SSL证书，然后下载，并且在缤纷云里配置：

设置-Https配置-开启-更换证书-填写证书内容-选择强制跳转-保存，等待部署完成即可。

![image-20240216152529958](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216152529958.png)

#### 其他问题2：502 bad Gateway：could not connect to the origin server.

这个问题，我在网上搜，答案是Ctrl + F5 刷新就可以解决，但是我试了还是报502，这个问题我怀疑是CDN的问题，后面再去探索。

## 终极解决方案：

### 1. twikoo系统有跨域配置：

原谅我没有仔细看配置，自己废了这么多力气，原因是最早之前把这个配置填错了，进入Twikoo管理界面-通用，将这个配置填成自己的域名。

![image-20240216170951915](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216170951915.png)

### 2. 更换其他评论系统，比如来必力评论系统：

> 我在用来必力评论系统之前尝试了Valine评论系统，这个系统也还不错，但是和Twikoo一样，会存在跨域问题，所以选择了来必力。
>
> Valine国内版好像没有跨域问题，但是需要备案的域名绑定，国际版自定义域名有跨域问题，后面再研究研究。

上面两个方案都是我认为可能有用的方案，但是我尝试时，多少有点问题，所以我暂时妥协了，更换了来必力评论系统，缺点是：必须登录才能评论，界面不好看。优点就一个：使用简单！

### 接入来必力评论系统步骤：

官网：[欢迎来到来必力 (livere.com)](https://www.livere.com/)

韩国的系统，可以用翻译软件翻译为中文。

注册登录 - 安装 - 选择City版（免费）- 输入网站信息 ，然后会生成代码，复制 UID：

![image-20240216163855959](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216163855959.png)

在_config.butterfly.yml文件中选择Livere评论系统：

![image-20240216164006928](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216164006928.png)

在Livere配置下配置UID即可，非常简单：

![image-20240216164158352](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216164158352.png)

使用起来也没有问题。

## 最后：

其实关于Twikoo跨域问题，我找了很多资料，都没有提出解决方案的，在官方Github：[hexo+butterfly 引入 twikoo 后显示访问跨域 · Issue #407 · twikoojs/twikoo (github.com)](https://github.com/twikoojs/twikoo/issues/407)里有类似的提问，但是好像没找到相关的回答，这篇文章算是一些思路尝试吧。

## 补充：

我是傻子，Twikoo有跨域配置，这个问题完全是因为我最开始将跨域配置填错导致的，官方github上有回答，是我没看见，现在文章内容已经更改。

不过，这次经历确实让我对跨域问题有了更深的理解，也让我明白细心的宝贵！！还有就是有问题可以去官方github上找找答案！

![image-20240216171822825](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240216171822825.png)