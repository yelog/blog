---
title: 'Jackson 时间序列化/反序列化详解'
enlink: 'Jackson Time Serializer/Deserializer'
date: 2024-07-05 16:00:00
categories:
- 后端
tags:
- java
- translation
---

# 前言

最近在项目中遇到了时间序列化的问题，所以研究了一下 Jackson 的时间序列化/反序列化，这里做一个详细的总结。

# 0. 准备工作

准备实体类 User.java

```java
package com.example.testjava.entity;

import lombok.Builder;
import lombok.Data;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.Date;

@Builder
@Data
public class User {
    private String name;
    private Date date;
    private LocalDate localDate;
    private LocalDateTime localDateTime;
    private LocalTime localTime;
    private java.sql.Date sqlDate;
    private java.sql.Time sqlTime;
    private java.sql.Timestamp timestamp;
}

```

简单查询

```java
package com.example.testjava.controller;

import com.example.testjava.entity.User;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

@RestController
@RequestMapping("/jackson")
public class JacksonTestController {

    @GetMapping("/query")
    public User testJavaDate() {
        return User.builder()
                .name("test")
                .date(new Date())
                .localDate(java.time.LocalDate.now())
                .localDateTime(java.time.LocalDateTime.now())
                .localTime(java.time.LocalTime.now())
                .sqlDate(new java.sql.Date(System.currentTimeMillis()))
                .sqlTime(new java.sql.Time(System.currentTimeMillis()))
                .timestamp(new java.sql.Timestamp(System.currentTimeMillis()))
                .build();
    }
}
```

# 1. 序列化
## 1.1. 默认返回

```json
{
    "name": "test",
    "date": "2024-07-05T08:09:47.100+00:00",
    "localDate": "2024-07-05",
    "localDateTime": "2024-07-05T16:09:47.100462",
    "localTime": "16:09:47.100514",
    "sqlDate": "2024-07-05",
    "sqlTime": "16:09:47",
    "timestamp": "2024-07-05T08:09:47.100+00:00"
}
```

# 1.2. 添加配置

配置如下

```yaml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
```

返回效果

```json
{
    "name": "test",
    "date": "2024-07-05 08:16:07",
    "localDate": "2024-07-05",
    "localDateTime": "2024-07-05T16:16:07.097035",
    "localTime": "16:16:07.09705",
    "sqlDate": "2024-07-05",
    "sqlTime": "16:16:07",
    "timestamp": "2024-07-05 08:16:07"
}
```

可以发现, 日期时间类型中, 只有 `java.time.LocalDateTime` 没有按照配置序列化, `java.util.Date` 和 `java.sql.Timestamp` 按照配置序列化了。

# 1.3. 添加注解

```java
package com.example.testjava.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Builder;
import lombok.Data;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.Date;

@Builder
@Data
public class User {
    private String name;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date date;
    private LocalDate localDate;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime localDateTime;
    @JsonFormat(pattern = "HH:mm:ss")
    private LocalTime localTime;
    private java.sql.Date sqlDate;
    private java.sql.Time sqlTime;
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private java.sql.Timestamp timestamp;
}

```

返回效果

```json
{
    "name": "test",
    "date": "2024-07-05 08:24:36",
    "localDate": "2024-07-05",
    "localDateTime": "2024-07-05 16:24:36",
    "localTime": "16:24:36",
    "sqlDate": "2024-07-05",
    "sqlTime": "16:24:36",
    "timestamp": "2024-07-05 08:24:36"
}
```

注解是可以都有效的

# 2. 反序列化

准备请求
```java

    @PostMapping("/save")
    public User save(@RequestBody User user) {
        return user;
    }

```
请求参数

```json
{
    "name": "test",
    "date": "2024-07-05 08:24:36",
    "localDate": "2024-07-05",
    "localDateTime": "2024-07-05 16:24:36",
    "localTime": "16:24:36",
    "sqlDate": "2024-07-05",
    "sqlTime": "16:24:36",
    "timestamp": "2024-07-05 08:24:36"
}
```

## 2.1 默认效果

默认报错
```text
JSON parse error: Cannot deserialize value of type `java.util.Date` from String \"2024-07-05 08:24:36\"
```

## 2.2 添加配置
有两种方法可以解决, 一个是自定义时间序列化, 一个是自定义 objectMapper

### 2.2.1 自定义时间序列化

```java
/**
 * 此转换方法试用于 json 请求
 * LocalDateTime 时间格式转换 支持
 */
@JsonComponent
@Configuration
public class LocalDateTimeFormatConfiguration extends JsonDeserializer<LocalDateTime> {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    /**
     * LocalDate 类型全局时间格式化
     * @return
     */
    @Bean
    public LocalDateTimeSerializer localDateTimeDeserializer() {
        return new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(pattern));
    }

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
        return builder -> builder.serializerByType(LocalDateTime.class, localDateTimeDeserializer());
    }

    @Override
    public LocalDateTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {
        return StrUtil.isEmpty(jsonParser.getText()) ? null : LocalDateTimeUtil.of(new DateTime(jsonParser.getText()));
    }
}

```

```java
@JsonComponent
@Configuration
public class DateFormatConfiguration extends JsonDeserializer<Date> {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    /**
     * date 类型全局时间格式化
     *
     * @return
     */
    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilder() {
        return builder -> {
            TimeZone tz = TimeZone.getTimeZone("UTC");
            DateFormat df = new SimpleDateFormat(pattern);
            df.setTimeZone(tz);
            builder.failOnEmptyBeans(false)
                    .failOnUnknownProperties(false)
                    .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                    .dateFormat(df);
        };
    }

    @Override
    public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {
        return StrUtil.isEmpty(jsonParser.getText()) ? null : new DateTime(jsonParser.getText());
    }
}

```

### 2.2.2 自定义 objectMapper


```java
package com.example.testjava.config;

import cn.hutool.core.date.DatePattern;
import cn.hutool.core.date.DateTime;
import cn.hutool.core.date.LocalDateTimeUtil;
import cn.hutool.core.util.StrUtil;
import com.fasterxml.jackson.core.JacksonException;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.databind.Module;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalTimeSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.AutoConfigureBefore;
import org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.io.IOException;
import java.sql.Time;
import java.sql.Timestamp;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.util.Date;

@Configuration
@AutoConfigureBefore(JacksonAutoConfiguration.class)
public class JacksonConfig {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        // 在反序列化时, 如果对象没有对应的字段, 不抛出异常
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        objectMapper.registerModule(javaTimeModule());
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return objectMapper;
    }

    private Module javaTimeModule() {
        JavaTimeModule module = new JavaTimeModule();
        // 序列化
        module.addSerializer(new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(pattern)));
        module.addSerializer(new LocalTimeSerializer(DateTimeFormatter.ofPattern(DatePattern.NORM_TIME_PATTERN)));
        module.addSerializer(new LocalDateSerializer(DateTimeFormatter.ofPattern(DatePattern.NORM_DATE_PATTERN)));
        module.addSerializer(Date.class, new JsonSerializer<>() {
            @Override
            public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                SimpleDateFormat sdf = new SimpleDateFormat(pattern);
                jsonGenerator.writeString(sdf.format(date));
            }
        });
        module.addSerializer(java.sql.Date.class, new JsonSerializer<>() {
            @Override
            public void serialize(java.sql.Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                SimpleDateFormat sdf = new SimpleDateFormat(pattern);
                jsonGenerator.writeString(sdf.format(date));
            }
        });
        module.addSerializer(Timestamp.class, new JsonSerializer<>() {
            @Override
            public void serialize(Timestamp timestamp, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                SimpleDateFormat sdf = new SimpleDateFormat(pattern);
                jsonGenerator.writeString(sdf.format(timestamp));
            }
        });
        module.addSerializer(Time.class, new JsonSerializer<>() {
            @Override
            public void serialize(Time time, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                SimpleDateFormat sdf = new SimpleDateFormat(DatePattern.NORM_TIME_PATTERN);
                jsonGenerator.writeString(sdf.format(time));
            }
        });

        // 反序列化
        module.addDeserializer(LocalDateTime.class, new JsonDeserializer<LocalDateTime>() {
            @Override
            public LocalDateTime deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JacksonException {
                return StrUtil.isEmpty(jsonParser.getText()) ? null : LocalDateTimeUtil.of(new DateTime(jsonParser.getText()));
            }
        });
        module.addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DatePattern.NORM_DATE_PATTERN)));
        module.addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DatePattern.NORM_TIME_PATTERN)));
        module.addDeserializer(Date.class, new JsonDeserializer<Date>() {
            @Override
            public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
                return StrUtil.isEmpty(jsonParser.getText()) ? null : new DateTime(jsonParser.getText());
            }
        });
        module.addDeserializer(java.sql.Date.class, new JsonDeserializer<java.sql.Date>() {
            @Override
            public java.sql.Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
                return StrUtil.isEmpty(jsonParser.getText()) ? null : new java.sql.Date(new DateTime(jsonParser.getText()).getTime());
            }
        });
        module.addDeserializer(Timestamp.class, new JsonDeserializer<Timestamp>() {
            @Override
            public Timestamp deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JacksonException {
                return StrUtil.isEmpty(jsonParser.getText()) ? null : new Timestamp(new DateTime(jsonParser.getText()).getTime());
            }
        });
        module.addDeserializer(Time.class, new JsonDeserializer<Time>() {
            @Override
            public Time deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JacksonException {
                return StrUtil.isEmpty(jsonParser.getText()) ? null : Time.valueOf(jsonParser.getText());
            }
        });
        // 添加默认处理
        return module;
    }
}
```

效果可以返回正确的数据

```json
{
    "name": "test",
    "date": "2024-07-05 08:24:36",
    "localDate": "2024-07-05",
    "localDateTime": "2024-07-05 16:24:36",
    "localTime": "16:24:36",
    "sqlDate": "2024-07-05 00:00:00",
    "sqlTime": "16:24:36",
    "timestamp": "2024-07-05 08:24:36"
}
```





