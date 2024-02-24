# 如何解决Idea使用git push时连接不上服务器

之前有记录过一个相似的问题，但是由于上一代博客没有做好备份，这篇文章也是成功被我弄丢了。。。哎，都是经验和教训。

## 问题简述：

先说说这次的问题：在Idea里搭好项目的基本框架后，像往常一样 "Share Project on Github", 结果报错了，报错信息忘记保存了，但大概意思是：GitHub仓库搭建完成，但是项目提交失败。我去仓库看确实只有仓库，没有代码。于是我就准备自己再 push 一次，但是还是报错了（试了三次，项目名不太方便放，可以自行去我的仓库看）：

```shell
fatal: unable to access 'https://github.com/twigIcer/xxx.git/': Failed to connect to github.com port 443 after 38150 ms: Couldn't connect to server
```

![](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/1.png)

## 探究过程：

### 1. 关闭代理：

我一看 "443"，这不是我上次记录的问题嘛，我记得和代理有关，好像关闭代理就可以了，于是上网一搜，果然有这个答案：

[git提交或克隆报错fatal: unable to access ‘https://github.com/tata20191003/autowrite.git/‘: Failed to connec-CSDN博客](https://blog.csdn.net/good_good_xiu/article/details/118567249)

解决方案是取消代理：

```shell
//取消http代理
git config --global --unset http.proxy
//取消https代理 
git config --global --unset https.proxy
```

但是关闭后还是报同样的错误。

> 后来仔细一看，人家的报错是 "fatal: unable to access 'https://github.com/xxx/autowrite.git/': Failed to connect to github.com port 443: Timed out" ,连接超时，而我的报错是连接不上服务器。

### 2. 关闭魔法：

我就在想是不是由于我开了魔法，但是刚才又关闭了代理，所以导致 push 又被墙了。

可是关闭了我的魔法后，还是报一样的错误。

### 3. 关闭本机代理：

后面我又看到一个答案是说，可以去关闭电脑的代理（文章链接找不到了，抱歉），"设置-网络和Internet-使用代理服务器的编辑按钮-关闭-保存"。

![](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-01-28%20125614.png)

可是很遗憾，还是不可以。

### 4. 尝试其他IDE:

我都在想是不是我的 git 出问题了，于是在 WebStorm 里尝试把前端部分提交上传，结果成功了，很显然是Idea的问题。

## 正解：

于是我就先放下这个问题了，但是这个问题不解决，总感觉难受，直到我看到了正确解决方案：

[git clone出现 fatal: unable to access ‘https://github.com/…’的解决办法(亲测有效)-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2221680)

我使用了上面这篇文章的解决方案二，手动配置git的代理。先将上面试错时关闭的本机代理重新开启，然后可以看到代理的Ip地址和端口，在 git bash 或者 Idea 终端用下面的命令指定代理即可：

```shell
// 使用时将ip和端口换为自己的
git config --global http.proxy http://127.0.0.1:7890 

git config --global https.proxy http://127.0.0.1:7890
```

然后再次 push ，发现报错：

```shell
fatal: unable to access 'https://github.com/twigIcer/xxx.git/': Recv failure: Connection was reset
```

连接重置，这个问题很常见了，重试就行了，于是重新 push ，成功推送，github仓库里也有了代码，问题解决。

## 总结：

由于国内的限制，访问 github 需要开魔法，开了魔法可能导致Idea记录的的 ip和端口与代理服务器ip 端口不一致的情况，这时候就需要修改代理或者关闭代理。上次记录的问题是上面试错第一个，连接超时的问题，这个问题可以关闭代理来解决。而这次报错是连接不上服务器 443 端口 ，就需要去修改代理IP和端口与本机一致。