# **Mybatis plus内置Mybatis-spring版本问题**

最近在做一个写轮子的项目，没想到才刚刚到整合mybatis plus这部分就翻车了，有点难受，这个问题个人觉得还是比较有意思，给大家分享一下。

## **问题引出：**

在项目大体框架搭建完成，引入了mysql和mybatis plus需要的依赖后，写了一个很简单的添加用户的接口测试项目架构时，就报错了：Caused by: java.lang.IllegalArgumentException: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required，

截取部分报错信息：

```java
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'userController': Unsatisfied dependency expressed through field 'userService': Error creating bean with name 'userServiceImpl': Unsatisfied dependency expressed through field 'userMapper': Error creating bean with name 'userMapper' defined in file [E:\projects\twig-frame\twig-user\target\classes\com\bin\user\mapper\UserMapper.class]: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:716) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:696) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:145) ~[spring-beans-6.0.11.jar:6.0.11]
	...
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:326) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:324) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:200) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:973) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:942) ~[spring-context-6.0.11.jar:6.0.11]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:608) ~[spring-context-6.0.11.jar:6.0.11]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:146) ~[spring-boot-3.1.2.jar:3.1.2]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:734) ~[spring-boot-3.1.2.jar:3.1.2]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:436) ~[spring-boot-3.1.2.jar:3.1.2]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:312) ~[spring-boot-3.1.2.jar:3.1.2]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1306) ~[spring-boot-3.1.2.jar:3.1.2]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1295) ~[spring-boot-3.1.2.jar:3.1.2]
	at com.bin.user.UserApplication.main(UserApplication.java:13) ~[classes/:na]
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'userServiceImpl': Unsatisfied dependency expressed through field 'userMapper': Error creating bean with name 'userMapper' defined in file [E:\projects\twig-frame\twig-user\target\classes\com\bin\user\mapper\UserMapper.class]: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:716) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:696) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:145) ~[spring-beans-6.0.11.jar:6.0.11]
    ...
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:324) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:200) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:254) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1417) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1337) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:713) ~[spring-beans-6.0.11.jar:6.0.11]
	... 20 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'userMapper' defined in file [E:\projects\twig-frame\twig-user\target\classes\com\bin\user\mapper\UserMapper.class]: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1770) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:598) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:520) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:326) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:324) ~[spring-beans-6.0.11.jar:6.0.11]
 ...   org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1337) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:713) ~[spring-beans-6.0.11.jar:6.0.11]
	... 34 common frames omitted
Caused by: java.lang.IllegalArgumentException: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required
	at org.springframework.util.Assert.notNull(Assert.java:204) ~[spring-core-6.0.11.jar:6.0.11]
	at org.mybatis.spring.support.SqlSessionDaoSupport.checkDaoConfig(SqlSessionDaoSupport.java:122) ~[mybatis-spring-2.0.7.jar:2.0.7]
	at org.mybatis.spring.mapper.MapperFactoryBean.checkDaoConfig(MapperFactoryBean.java:73) ~[mybatis-spring-2.0.7.jar:2.0.7]
	at org.springframework.dao.support.DaoSupport.afterPropertiesSet(DaoSupport.java:44) ~[spring-tx-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1817) ~[spring-beans-6.0.11.jar:6.0.11]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1766) ~[spring-beans-6.0.11.jar:6.0.11]
	... 44 common frames omitted
```

大致就是找不到'sqlSessionFactory' 或者 'sqlSessionTemplate'导致创建不了mapper这个bean，也就导致后续一系列bean都无法创建。

## **探究过程：**

刚好这个报错信息我前两天在使用mybatis时也见过，当时参考了这篇文章：https://blog.csdn.net/ZHENFENGSHISAN/article/details/128010240

大概意思就是：spring6和spring boot3删除了一些类或者方法，但是mybatis还没有同步更新导致的，再简单来说就是mybatis版本与springboot/spring版本不兼容导致的，当然这篇文章作者遇到这个问题时，mybatis还没有更新到3.0x，所以他的解决方案就是配置远程仓库，自己导入兼容性依赖包，但是我遇到这个问题时，mybatis早就更新到了3.0x，只是我在项目中引入的是2.0x,所以我的解决方案很简单：导入mybatis3.0x的依赖，也就是：

```java
<dependency>
     <groupId>org.mybatis.spring.boot</groupId>
     <artifactId>mybatis-spring-boot-starter</artifactId>
     <version>3.0.0</version>
</dependency>
```

当时也是顺利解决了问题，我本以为这次可能是相似的问题，也就是mybatis plus版本与spring/springboot版本不兼容导致的，但是我查看了依赖，发现我的mybatis plus依赖是可以适配springboot3.x的，这个时候就有点大事不好的感觉了，然后我找到了下面三种解决方案：

![](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image.png)

由于这个回答针对的也是mybatis使用过程中的问题，我项目中已经导入了mybatis-plus-spring-boot-starter依赖，所以想当然的忽略了方法一，方法二、三我的项目中也都是正确的。

于是我从头到尾检查了我的项目各部分：代码、数据库配置、MyBatis  plus配置等等，很遗憾没有用发现问题。

## **解决方案：**

感觉这只能是依赖的问题了，我又仔细去看了报错的日志，看见了这三句：

```java
at org.springframework.util.Assert.notNull(Assert.java:204) ~[spring-core-6.0.11.jar:6.0.11]
	at org.mybatis.spring.support.SqlSessionDaoSupport.checkDaoConfig(SqlSessionDaoSupport.java:122) ~[mybatis-spring-2.0.7.jar:2.0.7]
	at org.mybatis.spring.mapper.MapperFactoryBean.checkDaoConfig(MapperFactoryBean.java:73) ~[mybatis-spring-2.0.7.jar:2.0.7]
```

后面标注的都是mybatis-spring-2.0.7，我就想是不是mybatis plus内置的mybatis是2.0.7版本的，2.x版与springboot3不兼容导致的问题，于是我单独导入了mybatis-spring-boot-starter3.0版:

```java
<dependency>
     <groupId>org.mybatis.spring.boot</groupId>
     <artifactId>mybatis-spring-boot-starter</artifactId>
     <version>3.0.0</version>
</dependency>
```

问题解决。

## **小结：**

Caused by: java.lang.IllegalArgumentException: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required，这个报错的排查方向就在依赖和配置文件两方面，而依赖方面就需要排查所需依赖是不是导全了，数据库这块常用的依赖也总结下吧，以后方便点：

```java
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
        <version>3.1.2</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.2.16</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.23</version>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.2</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>3.0.0</version>
    </dependency>
```

如果依赖导入没有问题，但是还是报错，就需要考虑依赖于spring/springboot版本是不是兼容了，说实话这块挺烦的，老是因为版本问题遇到一些奇奇怪怪的问题，且行且积累吧。

## **思考：**

我专门去看了一下mybatis plus3.5.2的内置依赖，如下：

![](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/4.png)

显示的是在mybatis-plus-extension里内置了mybatis-spring:2.0.7，在mybatis-plus-core中内置了mybatis:3.5.10，但是mybatis-spring:2.0.7与springboot3不适配，所以会出问题，有点奇怪为什么mybatis plus3.x中不内置mybatis-spring:3.x而要内置mybatis-spring:2.0.7？有没有大佬解答一下呢？