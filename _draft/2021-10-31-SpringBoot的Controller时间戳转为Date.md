
从源码理解在SpringBoot中的入参date的转换

<!--more-->

# 接口返回转为时间戳

1、pom文件添加：

```
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-joda</artifactId>
    <version>2.9.10</version>
</dependency>
```

2、在**application.properties**添加如下配置：

```
# jackson 序列化日期为时间戳
spring.jackson.serialization.write-dates-as-timestamps=true

# 当然也可以指定为固定格式
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
# 为空时过滤掉
spring.jackson.default-property-inclusion=non_null
```

# 请求时间戳转为日期格式

报错内容：

```
Failed to convert property value of type 'java.lang.String' to required type 'java.util.Date' for property 'XXX'
```

看报错内容可知，前端传参为时间戳，后端接收为Date类型，那么这里需要做一层转换。

来看看SpringBoot怎么处理吧。

通过Spring源码定位：

```
org.springframework.core.convert.support.ObjectToObjectConverter#convert
org.springframework.core.convert.ConversionFailedException#ConversionFailedException
```

正确的配置：

```
# 在属性上加上如下注解
@DateTimeFormat(pattern = "yyyy-MM-dd")
```

另外说明一下，请求的Content-Type必须为`application/json`，不能是`application/x-www-form-urlencoded`，否则后端依然接收不到。


