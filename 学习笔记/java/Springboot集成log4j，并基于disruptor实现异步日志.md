# Springboot集成log4j，并基于disruptor实现异步日志

Java日志框架挺多的，log4j、slf4j、logback...logback与log4j的作者是同一个人，而logback实际上是log4j的改进版，SpringBoot内置了logback，而logback天然又支持slf4，所以我们在使用Springboot开发时，可能不知不觉就使用了logback+slf4j的框架，这两个框架不需要另外去集成，所以这篇文章就讲讲springboot集成log4j。

## Springboot集成log4j：

还是那“三板斧”：引依赖，写自定义配置，写配置文件。

### 1. 引依赖：

在springboot项目中先引入下面这个依赖，这里先埋个坑，后面再说：

```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
            <version>3.1.2</version>
        </dependency>
```

### 2. log4j-spring.xml里自定义配置：

其实文件名可以自己起，但是SpringBoot 官方推荐优先使用带有`-spring`的文件名作为你的日志配置，所以一般起名为：log4j-spring.xml，这些内容也可以在yml文件里配置，但是xml文件里内容也需要认识嘛，下面给一份比较完整的文件模板，复制就能使用那种，后续想自己配置功能修改即可：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身，设置间隔秒数-->
<configuration status="INFO" monitorInterval="5">
    <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
    <!--变量配置-->
    <Properties>
        <!-- 格式化输出：%date表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
        <!-- %logger{36} 表示 Logger 名字最长36个字符 -->
        <property name="LOG_PATTERN" value="%date{HH:mm:ss.SSS} %X{PFTID} [%thread] %-5level %logger{36} - %msg%n" />
        <!-- 定义日志存储的路径 -->
        <property name="FILE_PATH" value="../log" />
        <property name="FILE_NAME" value="frame.log" />
    </Properties>

    <!--https://logging.apache.org/log4j/2.x/manual/appenders.html-->
    <appenders>

        <console name="Console" target="SYSTEM_OUT">
            <!--输出日志的格式-->
            <PatternLayout pattern="${LOG_PATTERN}"/>
            <!--控制台只输出level及其以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
        </console>

        <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，适合临时测试用-->
        <File name="fileLog" fileName="${FILE_PATH}/temp.log" append="false">
            <PatternLayout pattern="${LOG_PATTERN}"/>
        </File>

        <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFileInfo" fileName="${FILE_PATH}/info.log" filePattern="${FILE_PATH}/${FILE_NAME}-INFO-%d{yyyy-MM-dd}_%i.log.gz">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="${LOG_PATTERN}"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour-->
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="10MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
            <DefaultRolloverStrategy max="15"/>
        </RollingFile>

        <!-- 这个会打印出所有的warn及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFileWarn" fileName="${FILE_PATH}/warn.log" filePattern="${FILE_PATH}/${FILE_NAME}-WARN-%d{yyyy-MM-dd}_%i.log.gz">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="${LOG_PATTERN}"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour-->
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="10MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
            <DefaultRolloverStrategy max="15"/>
        </RollingFile>

        <!-- 这个会打印出所有的error及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFileError" fileName="${FILE_PATH}/error.log" filePattern="${FILE_PATH}/${FILE_NAME}-ERROR-%d{yyyy-MM-dd}_%i.log.gz">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="${LOG_PATTERN}"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour-->
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="10MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
            <DefaultRolloverStrategy max="15"/>
        </RollingFile>

    </appenders>

    <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。-->
    <!--然后定义loggers，只有定义了logger并引入的appender，appender才会生效-->
    <loggers>

        <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
        <!--        <logger name="org.mybatis" level="info" additivity="false">-->
        <!--            <AppenderRef ref="Console"/>-->
        <!--        </logger>-->
        <!--监控系统信息-->
        <!--若是additivity设为false，则子Logger只会在自己的appender里输出，而不会在父Logger的appender里输出。-->
        <!--        <Logger name="org.springframework" level="info" additivity="false">-->
        <!--            <AppenderRef ref="Console"/>-->
        <!--        </Logger>-->

<!--                <AsyncLogger name="asyncLog" level="info" additivity="true">-->
<!--                    <appender-ref ref="RollingFileInfo"/>-->
<!--                    <appender-ref ref="Console"/>-->
<!--                </AsyncLogger>-->

<!--                <AsyncRoot level="info" includeLocation="true">-->
<!--                    <AppenderRef ref="RollingFileInfo" />-->
<!--                    <AppenderRef ref="Console"/>-->
<!--                </AsyncRoot>-->

        <root level="info">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
            <appender-ref ref="fileLog"/>
        </root>
    </loggers>

</configuration>
```

这份文件里面注释解释的也比较清楚，很多配置在需要的时候将注释符去掉就可以了，这里就不去过多解释了，想具体了解的，可以去看看这篇文章：[SpringBoot使用logback日志框架超详细教程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/555185411)虽然这篇文章讲的是logback框架，但是xml文件中的配置和log4j几乎一模一样，可以借鉴。

### 3. 在配置文件中指定.xml路径:

写完xml文件，还需要在yml配置文件里指定一下log框架的配置文件路径，也就是xml文件路径，加上这一句就i行了：

```java
logging:
  config: classpath:log4j-spring.xml
```

## 填前面的坑（解决依赖冲突方法）：

这时候启动项目，你会发现项目压根启动不了，报错啦哈哈，差不多会报下面的错：

```
SLF4J: Class path contains multiple SLF4J providers.
SLF4J: Found provider [ch.qos.logback.classic.spi.LogbackServiceProvider@4e41089d]
SLF4J: Found provider [org.apache.logging.slf4j.SLF4JServiceProvider@32a068d1]
SLF4J: See https://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual provider is of type [ch.qos.logback.classic.spi.LogbackServiceProvider@4e41089d]
```

我在做项目时的报错信息和这个demo里的报错信息不太一样，但是看到"SLF4J"，解决方式都时一样的，出现报错的原因就是前面说过，springBoot内置了日志框架，实现了slf4j，slf4j与log4j冲突了：

![image-20230812203858054](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20230812203858054.png)

其实埋这个坑的原因是为了介绍一下解决依赖冲突的方法，这里推荐一个插件：Maven Helper

![image-20230812204054629](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20230812204054629.png)

安装了这个插件，在进入pom文件时就会有下面这个"Dependency Analyzer"按钮：
![image-20240124150632768](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240124150632768.png)

点击这个按钮，就会进入到上面第一张图中的界面，里面将各个依赖的关系罗列的很清楚，像出现了log方面的依赖冲突，就可以直接在搜索框搜索”log“，就可以看到在上面第一张图中关于log的依赖都被标识出来了，可以看到，除了log4j依赖外，spring-boot-starter-web里有内置的spring-boot-starter-logging依赖，这个时候我们既可以右键spring-boot-starter-logging，点击”Exclude“，插件就自动帮我们把spring-boot-starter-web中的log依赖排除了，很方便。

个人觉得学会排除重复依赖的方法还是满重要的，所以不要怪我前面埋坑喽。

这个时候刷新maven，启动项目，看到之前彩色的日志全部变成白色的就说明log4j整合完成了。

## 基于disruptor实现异步日志：

其实前面的内容都是铺垫，我最想记录就是这一点，在大项目中，或许每秒都有很多条日志写入文件中，如果不使用异步日志的话，就会导致进程阻塞。

> 在多线程服务程序中，异步日志是必须的，因为如果在网络IO线程或业务线程中直接往磁盘写数据的话，写操作偶尔可能阻塞长达数秒之久。这可能导致请求方超时，活着耽误发送心跳消息，在分布式系统中更可能造成多米诺骨牌效应，例如误报死锁引发自动failover等。

### disruptor简介：

Disruptor是一个高性能的异步处理框架，它可以帮助我们轻松构建数据流处理。Disruptor的核心思想是使用环形队列来存储事件，然后通过消费者线程来消费这些事件。这种方式可以避免锁和同步器，从而提高性能。

### 1. 引依赖：

使用disruptor需要引入下面的依赖：

```java
        <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>3.4.2</version>
        </dependency>
```

### 2. 修改xml配置文件：

使用异步日志，需要将xml文件里的配置修改一下，主要就是将<AsyncLogger>这块注释去了，将<root>注释掉：

```xml
        <AsyncLogger name="asyncLog" level="info" additivity="true">
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="Console"/>
        </AsyncLogger>

        <AsyncRoot level="info" includeLocation="true">
            <AppenderRef ref="RollingFileInfo" />
            <AppenderRef ref="Console"/>
        </AsyncRoot>

<!--        <root level="info">-->
<!--            <appender-ref ref="Console"/>-->
<!--            <appender-ref ref="RollingFileInfo"/>-->
<!--            <appender-ref ref="RollingFileWarn"/>-->
<!--            <appender-ref ref="RollingFileError"/>-->
<!--            <appender-ref ref="fileLog"/>-->
<!--        </root>-->
```

### 3. 在启动类上加配置：

使用异步日志还需要再启动类上加上这一句固定配置"System.setProperty("Log4jContextSelector", "org.apache.logging.log4j.core.async.AsyncLoggerContextSelector");"：

```java
@SpringBootApplication
public class Log4jTestApplication {

    public static void main(String[] args) {
        System.setProperty("Log4jContextSelector", "org.apache.logging.log4j.core.async.AsyncLoggerContextSelector");
        SpringApplication.run(Log4jTestApplication.class, args);
    }
}
```

## 测试异步日志：

简单写一个测试类，向文件中写入10000条日志，测试对比一下使用异步日志和不使用的效率：

```java
  @GetMapping("/testLog")
    public void testLog(){
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            log.info("这是{}条日志！", i);
        }
        long endTime = System.currentTimeMillis();
        log.info("当前耗时：{}", endTime - startTime);
    }
```

未使用异步日志，耗时2907ms，使用异步日志，耗时158ms，可以看到效率提升还是满明显的。

## 小结：

在springboot项目中整合log4j需要注意排除springboot内置日志框架，使用disruptor异步日志可以大幅提升日志记录性能。