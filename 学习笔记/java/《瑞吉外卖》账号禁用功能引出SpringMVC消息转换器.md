# 《瑞吉外卖》账号禁用功能引出SpringMVC消息转换器

在练习《瑞吉外卖》这个经典项目时，看到了这个经典的问题，觉得也还是挺有意思，虽然黑马的老师已经讲得挺清楚了，但是我还是想记录一下，就当做笔记了，最后也加入了一些我自己的思考。

## 问题引出：

### 目标效果：

先简单说一下要实现的效果：在员工列表，点击“禁用”后，将“账号状态”改为“禁用”，原本的“禁用”按钮变为“启用”按钮，数据库中的“status”由“1”修改为“0”.

### 代码实现：

实现方式比较简单，大部分工作由前端完成，后端接收前端的请求及数据，调用service修改员工信息即可。

```java
 /**
     * 根据id修改员工信息
     * @param employee
     * @return
     */
    @PutMapping
    public R<String> update(HttpServletRequest request,@RequestBody Employee employee){
        Long empId = (Long) request.getSession().getAttribute("employee");
        //设置更新时间
        employee.setUpdateTime(LocalDateTime.now());

        //设置更新人
        employee.setUpdateUser(empId);
        employeeService.updateById(employee);
        return R.success("员工信息修改成功!");
    }
```

### 遇到的问题：

在实现员工账号禁用功能时，写好了controller，代码实现逻辑没有问题，可以正常运行，点击“确认禁用”后，网页也没有报错，但是禁用后，账号状态依然显示“正常”，数据库中员工状态“status”也还是“1”。

![img](https://img-blog.csdnimg.cn/7204ee2ebb2f432b8d1ca3f2fda8f7b7.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/7cf5758b536040e58e518adb1e855044.png)

##  问题探究：

这个问题要是自己遇到肯定非常麻烦，好在这次有老师领路，而且看得出老师是故意引出这个问题的。老师的探究过程就是打断点调试，一步一步发现问题：在后台传递id给前端时id与数据库中id一致，但是前端在发送请求时带的参数与数据库中不一致，后三位进行四舍五入了，导致后台拿到的id在数据库中查不到数据。

数据库中的正确id：

![img](https://img-blog.csdnimg.cn/8abba5e45d114f0193d54acdeb332b99.png)

 前端传给后端的id：

![img](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/abca65e679ea4e39b9e7aac067cfbc1e.png)

 最后老师给出了原因：js在处理long型数据时丢失精度，导致提交的id与数据库中的id不一致。

## 解决方法：

老师也给出了解决方法： 

1. 提供对象转换器JacksonObjectMapper，基于Jackson进行 java 对象到 json 数据的转换。（在common包中添加这个类，这部分是给好的，不用自己写）:

   ```java
   package com.bin.reggie_take_out.common;
   
   import com.fasterxml.jackson.databind.DeserializationFeature;
   import com.fasterxml.jackson.databind.ObjectMapper;
   import com.fasterxml.jackson.databind.module.SimpleModule;
   import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
   import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
   import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
   import com.fasterxml.jackson.datatype.jsr310.deser.LocalTimeDeserializer;
   import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;
   import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
   import com.fasterxml.jackson.datatype.jsr310.ser.LocalTimeSerializer;
   import java.math.BigInteger;
   import java.time.LocalDate;
   import java.time.LocalDateTime;
   import java.time.LocalTime;
   import java.time.format.DateTimeFormatter;
   import static com.fasterxml.jackson.databind.DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES;
   
   /**
    * 对象映射器:基于jackson将Java对象转为json，或者将json转为Java对象
    * 将JSON解析为Java对象的过程称为 [从JSON反序列化Java对象]
    * 从Java对象生成JSON的过程称为 [序列化Java对象到JSON]
    */
   public class JacksonObjectMapper extends ObjectMapper {
   
       public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
       public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
       public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";
   
       public JacksonObjectMapper() {
           super();
           //收到未知属性时不报异常
           this.configure(FAIL_ON_UNKNOWN_PROPERTIES, false);
   
           //反序列化时，属性不存在的兼容处理
           this.getDeserializationConfig().withoutFeatures(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
   
   
           SimpleModule simpleModule = new SimpleModule()
                   .addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                   .addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                   .addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)))
   
                   .addSerializer(BigInteger.class, ToStringSerializer.instance)
                   .addSerializer(Long.class, ToStringSerializer.instance)
                   .addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)))
                   .addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)))
                   .addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
   
           //注册功能模块 例如，可以添加自定义序列化器和反序列化器
           this.registerModule(simpleModule);
       }
   }
   ```

2. 在WebMvcConfig配置类中扩展SpringMVC的消息转换器，在此消息转换器中使用提供的对象转换器进行 java 对象到 json 数据的转换。

   注：converters.add(0,messageConverter);这一步是将自己的转换器放在索引为0的位置，也就是第一位，否则还是会调用默认的转换器。

   ```java
   package com.bin.reggie_take_out.config;
   
   import com.bin.reggie_take_out.common.JacksonObjectMapper;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.http.converter.HttpMessageConverter;
   import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
   import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
   import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
   
   import java.util.List;
   
   @Configuration
   @Slf4j
   public class WebMvcConfig extends WebMvcConfigurationSupport {
   
       /**
        * 扩展MVC框架的消息转换器
        * @param converters
        */
       @Override
       protected void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
           //创建一个消息转化器对象
           MappingJackson2HttpMessageConverter messageConverter = new MappingJackson2HttpMessageConverter();
           //设置对象转换器，底层使用Jackson将java对象转换成json
           messageConverter.setObjectMapper(new JacksonObjectMapper());
           //将上面的消息转换器对象追加到MVC框架的消息转换器集合中
           converters.add(0,messageConverter);
       }
   }
   ```

   如果之前一步步跟着老师的步骤做的，做完这两步问题就已经解决了。

   ![img](https://img-blog.csdnimg.cn/6ae85f60bb8247768b4242d26fbc95f7.png)

![img](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/b0c99bd3da1242a2a24d7fb84a834785.png)

但是我呢在静态资源那块偷懒了，没有跟老师一起在WebMvcConfig配置类中定义静态资源映射规则，而是直接将静态资源放在了static目录下，导致做完这两步(准确来说是第二步后)，网页（index.html）无法打开了。