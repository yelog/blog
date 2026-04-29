---
title: Spring Boot 4 实战：Jackson 2.x 升级到 3.x 踩坑全记录
enlink: springboot4-jackson3
date: 2026-04-28 18:42:41
categories:
- 后端
tags:
- java
- springboot
- jackson
---


# Spring Boot 4 实战：Jackson 2.x 升级到 3.x 踩坑全记录

## 前言

最近在将公司的 Spring Cloud 微服务项目从 **Spring Boot 3.2 + JDK 17** 升级到 **Spring Boot 4.0 + JDK 21** 的过程中，Jackson 从 2.x 到 3.x 的升级是改动最大、坑最多的部分之一。

Jackson 3.0.0 于 2025 年 10 月 3 日正式发布，这是自 2013 年 Jackson 2.0 以来的首个大版本更新，经历了近 8 年的开发。Spring Boot 4.0 也正式将 Jackson 3 作为默认 JSON 库。

本文将结合实际项目代码，从**官方变更说明**和**真实踩坑经验**两个维度，带大家完整走一遍 Jackson 2.x → 3.x 的迁移过程。

---

## 一、Jackson 3.x 的核心变化概览

在动手之前，我们先从官方文档梳理 Jackson 3.x 相比 2.x 的核心变化：

### 1.1 包名和 Maven 坐标全面更换

这是最直观的 Breaking Change：

| 维度 | Jackson 2.x | Jackson 3.x |
|------|-------------|-------------|
| Maven GroupId | `com.fasterxml.jackson` | `tools.jackson` |
| Java 包名 | `com.fasterxml.jackson.*` | `tools.jackson.*` |
| 注解包名 | `com.fasterxml.jackson.annotation` | **不变**（仍用 2.x 的注解） |

> **特别注意**：`jackson-annotations` 是个例外！Jackson 3.x 依赖的是 `jackson-annotations` 2.20 版本，注解的包名 `com.fasterxml.jackson.annotation` **保持不变**。这意味着 `@JsonFormat`、`@JsonInclude`、`@JsonProperty` 等注解的 import 不需要改。

但 `jackson-databind` 中定义的注解（如 `@JsonSerialize`、`@JsonDeserialize`）**会**跟随包名迁移到 `tools.jackson.databind.annotation`。

### 1.2 核心类重命名

Jackson 3.x 对大量核心类进行了重命名，目的是让命名更加语义化：

| Jackson 2.x | Jackson 3.x | 说明 |
|-------------|-------------|------|
| `ObjectMapper` | `JsonMapper`（推荐） | ObjectMapper 仍存在，但推荐使用格式特定子类 |
| `JsonSerializer<T>` | `ValueSerializer<T>` | 序列化器基类 |
| `JsonDeserializer<T>` | `ValueDeserializer<T>` | 反序列化器基类 |
| `SerializerProvider` | `SerializationContext` | 序列化上下文 |
| `DeserializationContext` | `DeserializationContext` | 不变 |
| `JsonProcessingException` | `JacksonException` | 基础异常类 |
| `JsonMappingException` | `DatabindException` | 数据绑定异常 |
| `JsonParseException` | `StreamReadException` | 解析异常 |
| `Module` | `JacksonModule` | 避免与 `java.lang.Module` 冲突 |
| `TextNode` | `StringNode` | JsonNode 子类型 |

### 1.3 异常体系变为 Unchecked

这是一个重大改变：Jackson 3.x 的所有异常都继承自 `RuntimeException`，而不是 2.x 中的 `IOException`。

```
// Jackson 2.x
JsonProcessingException extends IOException  // checked

// Jackson 3.x
JacksonException extends RuntimeException    // unchecked
```

这意味着你不再需要在方法签名中声明 `throws` 子句，但 `catch` 块中的异常类型需要更新。

### 1.4 ObjectMapper 变为不可变

Jackson 3.x 中 `ObjectMapper`（及其子类型如 `JsonMapper`）是**完全不可变的**，必须通过 Builder 模式构建：

```java
// Jackson 2.x - 可变配置
ObjectMapper mapper = new ObjectMapper();
mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
mapper.registerModule(new JavaTimeModule());

// Jackson 3.x - Builder 模式，不可变
JsonMapper mapper = JsonMapper.builder()
    .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
    .changeDefaultPropertyInclusion(old ->
        JsonInclude.Value.construct(JsonInclude.Include.NON_NULL, JsonInclude.Include.NON_NULL))
    .addModule(new SomeModule())
    .build();
```

### 1.5 Java 8 时间模块内置

Jackson 2.x 中需要单独引入 `jackson-datatype-jsr310` 来支持 `java.time` 类型，3.x 已将其**内置到 `jackson-databind` 中**，无需额外依赖和注册：

- `jackson-module-parameter-names` → 内置
- `jackson-datatype-jdk8`（Optional 等）→ 内置
- `jackson-datatype-jsr310`（java.time）→ 内置

### 1.6 默认配置变更

Jackson 3.x 修改了多个 Feature 的默认值，这些变更可能是升级后行为不一致的主因：

| Feature | 2.x 默认值 | 3.x 默认值 | 影响 |
|---------|-----------|-----------|------|
| `FAIL_ON_UNKNOWN_PROPERTIES` | `true` | **`false`** | 反序列化时遇到未知字段不再报错 |
| `WRITE_DATES_AS_TIMESTAMPS` | `true` | **`false`** | 日期默认输出 ISO-8601 字符串而非时间戳 |
| `SORT_PROPERTIES_ALPHABETICALLY` | `false` | **`true`** | 属性按字母排序序列化 |
| `FAIL_ON_TRAILING_TOKENS` | `false` | **`true`** | 解析后检查是否有多余内容 |
| `FAIL_ON_NULL_FOR_PRIMITIVES` | `false` | **`true`** | null 赋值给基本类型会报错 |
| `FAIL_ON_EMPTY_BEANS` | `true` | **`false`** | 空 Bean 序列化不再报错 |

---

## 二、Spring Boot 4 中的 Jackson 3 集成方式

Spring Boot 4 对 Jackson 的集成方式有重大变化，理解这些变化是顺利迁移的关键。

### 2.1 告别 Jackson2ObjectMapperBuilder

在 Spring Boot 3.x（Jackson 2.x）中，我们通常通过 `Jackson2ObjectMapperBuilderCustomizer` 来定制 `ObjectMapper`：

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer customizer() {
    return builder -> builder.indentOutput(true);
}
```

在 Spring Boot 4.x（Jackson 3.x）中，Spring 不再提供 `Jackson2ObjectMapperBuilder` 的等价物，而是**直接使用 Jackson 原生的 Builder**，通过 `JsonMapperBuilderCustomizer` 进行定制：

```java
@Bean
JsonMapperBuilderCustomizer jacksonCustomizer() {
    return builder -> builder.enable(SerializationFeature.INDENT_OUTPUT);
}
```

### 2.2 Jackson 2 和 3 可以共存

得益于包名完全不同（`com.fasterxml.jackson` vs `tools.jackson`），Jackson 2 和 3 **可以在同一个 classpath 上共存**。这为渐进式迁移提供了极大便利。

Spring Boot 4 同时提供了两套版本的依赖管理，Spring Data Redis 等组件目前仍依赖 Jackson 2.x 的 API，两套 Jackson 通过不同包名共存互不冲突。

---

## 三、实战迁移：逐个击破

下面结合我们项目的真实代码，逐一讲解迁移过程中的关键改动。

### 3.1 JacksonConfig：从 ObjectMapper Bean 到 JsonMapperBuilderCustomizer

这是改动最大的一个文件。我们原来通过 `@Bean` 直接创建 `ObjectMapper` 实例：

**迁移前（Jackson 2.x）：**

```java
@Configuration
@AutoConfigureBefore(JacksonAutoConfiguration.class)
public class JacksonConfig {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        objectMapper.setDefaultPropertyInclusion(JsonInclude.Include.NON_NULL);
        objectMapper.registerModule(javaTimeModule());
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        objectMapper.setTimeZone(TimeZone.getTimeZone(timezone));
        return objectMapper;
    }

    private Module javaTimeModule() {
        JavaTimeModule module = new JavaTimeModule();
        module.addSerializer(LocalDateTime.class, getDateSerializer(pattern));
        module.addDeserializer(LocalDateTime.class, getDateDeserializer(LocalDateTime.class));
        // ... 其他类型
        return module;
    }

    private <T> JsonSerializer<T> getDateSerializer(String pattern) {
        return new JsonSerializer<>() {
            @Override
            public void serialize(T date, JsonGenerator gen, SerializerProvider provider)
                    throws IOException {
                // 序列化逻辑
            }
        };
    }
}
```

**迁移后（Jackson 3.x）：**

```java
@Configuration
public class JacksonConfig {

    @Value("${spring.jackson.date-format:yyyy-MM-dd HH:mm:ss}")
    private String pattern;

    @Value("${spring.jackson.time-zone:}")
    private String timezone;

    @Bean
    public JsonMapperBuilderCustomizer mossJacksonCustomizer() {
        return builder -> {
            String tz = StrUtil.emptyToDefault(timezone, ZoneId.systemDefault().getId());

            SimpleModule module = new SimpleModule("MossDateTimeModule");

            // 序列化
            module.addSerializer(LocalDateTime.class, createDateSerializer(pattern));
            module.addSerializer(LocalDate.class, createDateSerializer(DatePattern.NORM_DATE_PATTERN));
            // ... 其他类型

            // 反序列化
            module.addDeserializer(LocalDateTime.class, createDateDeserializer(LocalDateTime.class));
            module.addDeserializer(LocalDate.class, createDateDeserializer(LocalDate.class));
            // ... 其他类型

            builder.addModule(module);
            builder.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
            builder.changeDefaultPropertyInclusion(old ->
                JsonInclude.Value.construct(
                    JsonInclude.Include.NON_NULL, JsonInclude.Include.NON_NULL));
            builder.defaultTimeZone(TimeZone.getTimeZone(tz));
        };
    }

    static <T> ValueSerializer<T> createDateSerializer(String pattern) {
        return new ValueSerializer<T>() {
            @Override
            public void serialize(T date, JsonGenerator gen,
                    SerializationContext ctx) throws JacksonException {
                // 序列化逻辑不变，只是参数类型改了
            }
        };
    }

    static <T> ValueDeserializer<T> createDateDeserializer(Class<T> clazz) {
        return new ValueDeserializer<T>() {
            @Override
            public T deserialize(JsonParser parser,
                    DeserializationContext ctx) throws JacksonException {
                String text = parser.getString(); // getText() -> getString()
                // 反序列化逻辑不变
            }
        };
    }
}
```

**关键变化总结：**

| 变化点 | Jackson 2.x | Jackson 3.x |
|-------|-------------|-------------|
| 配置方式 | 直接创建 `ObjectMapper` Bean | 通过 `JsonMapperBuilderCustomizer` 定制 |
| 注解 | `@AutoConfigureBefore` | 不再需要（Customizer 自动生效） |
| 序列化器 | `JsonSerializer<T>` | `ValueSerializer<T>` |
| 反序列化器 | `JsonDeserializer<T>` | `ValueDeserializer<T>` |
| 序列化上下文 | `SerializerProvider` | `SerializationContext` |
| 异常类型 | `IOException` | `JacksonException`（unchecked） |
| 获取文本 | `parser.getText()` | `parser.getString()` |
| 时间模块 | `JavaTimeModule`（需手动注册） | `SimpleModule`（内置 jsr310 支持） |
| 序列化包含策略 | `mapper.setSerializationInclusion(...)` | `builder.changeDefaultPropertyInclusion(...)` |

### 3.2 网关层：ObjectMapper 替换为 JsonMapper

在 Gateway 模块中，之前到处 `new ObjectMapper()` 来做 JSON 序列化，这在 3.x 中需要改为注入 `JsonMapper`：

**迁移前：**

```java
public class MossGlobalFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // ... 
        ObjectMapper objectMapper = new ObjectMapper(); // 每次新建一个！
        try {
            return response.writeWith(Mono.just(response.bufferFactory()
                .wrap(objectMapper.writeValueAsBytes(ResultData.fail(...)))));
        } catch (JsonProcessingException ex) {
            log.error("对象输出异常：", ex);
        }
    }
}
```

**迁移后：**

```java
public class MossGlobalFilter implements GlobalFilter {

    @Resource
    private JsonMapper jsonMapper; // 注入不可变的 JsonMapper

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // ...
        try {
            return response.writeWith(Mono.just(response.bufferFactory()
                .wrap(jsonMapper.writeValueAsBytes(ResultData.fail(...)))));
        } catch (JacksonException ex) {  // 异常类型变了
            log.error("对象输出异常：", ex);
        }
    }
}
```

改动点：
1. `ObjectMapper` → `JsonMapper`（通过 `@Resource` 注入，不再每次 new）
2. `JsonProcessingException` → `JacksonException`
3. `com.fasterxml.jackson` → `tools.jackson` 的 import 替换

### 3.3 Redis 序列化：Jackson 2.x 和 3.x 共存的典型场景

这是一个非常典型的**两套 Jackson 共存**场景。Spring Data Redis 4.0.x 的 `Jackson2JsonRedisSerializer` 仍基于 Jackson 2.x API，因此 Redis 配置类**必须继续使用 Jackson 2.x**：

```java
/**
 * Redis 配置类
 * 
 * 注意：虽然项目已将 REST 序列化迁移到 Jackson 3.x（tools.jackson.*），
 * 但 Spring Data Redis 4.0.x 的 Jackson2JsonRedisSerializer 仍依赖
 * Jackson 2.x（com.fasterxml.jackson.*），因此本类必须继续使用 Jackson 2.x API。
 * 两套 Jackson 通过不同包名在 classpath 上共存，互不冲突。
 */
@Configuration
public class RedisConfig {

    public void commonSerializerConfig(RedisTemplate<String, Object> redisTemplate) {
        // 这里使用的是 com.fasterxml.jackson.databind.ObjectMapper
        // 不是 tools.jackson.databind.json.JsonMapper！
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(
            LaissezFaireSubTypeValidator.instance,
            ObjectMapper.DefaultTyping.NON_FINAL);
        om.registerModule(new JavaTimeModule());

        Jackson2JsonRedisSerializer<Object> serializer =
            new Jackson2JsonRedisSerializer<>(om, Object.class);
        redisTemplate.setValueSerializer(serializer);
    }
}
```

> `Jackson2JsonRedisSerializer` 已被标记为 `@Deprecated(forRemoval = true)`，待 Spring Data Redis 升级适配 Jackson 3.x 后，应迁移至新的序列化方案。

### 3.4 Gateway 异常处理器：包名迁移

```java
// 迁移前
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.web.reactive.error.ErrorWebExceptionHandler;

public class CustomErrorWebExceptionHandler implements ErrorWebExceptionHandler {
    private final ObjectMapper objectMapper;
    // ...
    try {
        return response.writeWith(Mono.just(response.bufferFactory()
            .wrap(objectMapper.writeValueAsBytes(...))));
    } catch (JsonProcessingException e) { ... }
}

// 迁移后
import tools.jackson.core.JacksonException;
import tools.jackson.databind.json.JsonMapper;
import org.springframework.boot.webflux.error.ErrorWebExceptionHandler;  // 包名也变了！

public class CustomErrorWebExceptionHandler implements ErrorWebExceptionHandler {
    private final JsonMapper jsonMapper;
    // ...
    try {
        return response.writeWith(Mono.just(response.bufferFactory()
            .wrap(jsonMapper.writeValueAsBytes(...))));
    } catch (JacksonException e) { ... }
}
```

注意不仅 Jackson 的包名变了，Spring Boot 4 自身的一些类的包名也发生了变化，比如 `ErrorWebExceptionHandler` 从 `org.springframework.boot.web.reactive.error` 移到了 `org.springframework.boot.webflux.error`。

---

## 四、迁移清单：import 替换速查表

以下是实际项目中涉及的 import 替换对照表，可以直接用 IDE 的全局替换功能批量处理：

```
# Jackson 核心类
com.fasterxml.jackson.core.JsonGenerator        → tools.jackson.core.JsonGenerator
com.fasterxml.jackson.core.JsonParser            → tools.jackson.core.JsonParser
com.fasterxml.jackson.core.JsonProcessingException → tools.jackson.core.JacksonException
com.fasterxml.jackson.databind.ObjectMapper      → tools.jackson.databind.json.JsonMapper
com.fasterxml.jackson.databind.JsonSerializer    → tools.jackson.databind.ValueSerializer
com.fasterxml.jackson.databind.JsonDeserializer  → tools.jackson.databind.ValueDeserializer
com.fasterxml.jackson.databind.SerializerProvider → tools.jackson.databind.SerializationContext
com.fasterxml.jackson.databind.DeserializationContext → tools.jackson.databind.DeserializationContext
com.fasterxml.jackson.databind.DeserializationFeature → tools.jackson.databind.DeserializationFeature
com.fasterxml.jackson.databind.SerializationFeature → tools.jackson.databind.SerializationFeature
com.fasterxml.jackson.databind.Module            → tools.jackson.databind.JacksonModule
com.fasterxml.jackson.databind.module.SimpleModule → tools.jackson.databind.module.SimpleModule

# 时间模块（已内置，通常可以删除 import）
com.fasterxml.jackson.datatype.jsr310.JavaTimeModule → 已内置，无需单独引入

# 注解（不变！）
com.fasterxml.jackson.annotation.JsonInclude     → 不变
com.fasterxml.jackson.annotation.JsonFormat       → 不变
com.fasterxml.jackson.annotation.JsonProperty     → 不变
```

---

## 五、踩坑记录

### 坑 1：JavaTimeModule 还是 SimpleModule？

Jackson 3.x 已将 `jackson-datatype-jsr310` 内置到 `jackson-databind` 中，因此 `JavaTimeModule` 不再作为独立模块存在于 `tools.jackson` 包下。

如果你之前通过 `JavaTimeModule` 注册自定义的日期序列化/反序列化器，需要改为使用 `SimpleModule`：

```java
// 2.x
JavaTimeModule module = new JavaTimeModule();
module.addSerializer(LocalDateTime.class, customSerializer);

// 3.x
SimpleModule module = new SimpleModule("CustomDateTimeModule");
module.addSerializer(LocalDateTime.class, customSerializer);
```

### 坑 2：setSerializationInclusion 被移除

`ObjectMapper.setSerializationInclusion()` 在 3.x 中被移除，需要通过 Builder 的 `changeDefaultPropertyInclusion` 方法替代：

```java
// 2.x
objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
objectMapper.setDefaultPropertyInclusion(JsonInclude.Include.NON_NULL);

// 3.x
builder.changeDefaultPropertyInclusion(old ->
    JsonInclude.Value.construct(
        JsonInclude.Include.NON_NULL,
        JsonInclude.Include.NON_NULL));
```

### 坑 3：getText() → getString()

`JsonParser.getText()` 在 3.x 中被重命名为 `getString()`。如果你有自定义的反序列化器，IDE 可能不会自动提示这个变化：

```java
// 2.x
String text = jsonParser.getText();

// 3.x
String text = jsonParser.getString();
```

### 坑 4：Redis 序列化器必须用 Jackson 2.x

Spring Data Redis 4.0 的 `Jackson2JsonRedisSerializer` 构造器只接受 `com.fasterxml.jackson.databind.ObjectMapper`，不兼容 Jackson 3.x 的 `tools.jackson.databind.json.JsonMapper`。

如果你在 RedisConfig 中也试图迁移到 Jackson 3.x 的 `JsonMapper`，会遇到编译错误。正确做法是 Redis 配置继续使用 Jackson 2.x，等待 Spring Data Redis 后续版本的适配。

### 坑 5：FAIL_ON_UNKNOWN_PROPERTIES 默认值变化

Jackson 3.x 中 `FAIL_ON_UNKNOWN_PROPERTIES` 的默认值改为了 `false`，这实际上和大多数项目的期望一致。但如果你之前**依赖**这个特性来发现字段拼写错误，升级后可能会静默忽略错误的字段名。

如果你的项目原本就显式配置了 `false`（大多数项目都是），那么这行配置可以删掉了：

```java
// 3.x 中这行可以删除，因为默认就是 false
builder.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

### 坑 6：WRITE_DATES_AS_TIMESTAMPS 默认值变化

这个影响范围很大。Jackson 2.x 默认将日期序列化为时间戳数字，3.x 改为输出 ISO-8601 字符串。如果你有大量依赖时间戳格式的前端代码或单元测试，需要特别注意。

不过对于我们的项目来说，由于本身就通过自定义序列化器输出 `yyyy-MM-dd HH:mm:ss` 格式，这个默认值变化没有实际影响。

---

## 六、单元测试验证

迁移完成后，编写覆盖性的单元测试非常重要。我们为 `JacksonConfig` 编写了完整的测试用例，验证所有日期类型的序列化/反序列化：

```java
@Slf4j
public class JacksonConfigTest {

    private JsonMapper buildMapper() {
        SimpleModule module = new SimpleModule("MossDateTimeModule");
        module.addSerializer(LocalDateTime.class,
            JacksonConfig.createDateSerializer("yyyy-MM-dd HH:mm:ss"));
        module.addDeserializer(LocalDateTime.class,
            JacksonConfig.createDateDeserializer(LocalDateTime.class));
        // ... 注册其他类型

        return JsonMapper.builder()
            .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
            .changeDefaultPropertyInclusion(old ->
                JsonInclude.Value.construct(
                    JsonInclude.Include.NON_NULL, JsonInclude.Include.NON_NULL))
            .defaultTimeZone(TimeZone.getDefault())
            .addModule(module)
            .build();
    }

    @Test
    public void testSerialize_LocalDateTime() throws Exception {
        JsonMapper mapper = buildMapper();
        DateTimeDto dto = new DateTimeDto();
        dto.setLocalDateTime(LocalDateTime.of(2026, 4, 27, 15, 30, 0));

        String json = mapper.writeValueAsString(dto);

        assertTrue(json.contains("2026-04-27 15:30:00"));
        assertFalse(json.contains("2026-04-27T15:30")); // 确认没有 ISO 的 T 分隔符
    }

    @Test
    public void testSerialize_nonNull() throws Exception {
        JsonMapper mapper = buildMapper();
        DateTimeDto dto = new DateTimeDto();
        dto.setTitle("仅标题");

        String json = mapper.writeValueAsString(dto);

        assertTrue(json.contains("仅标题"));
        assertFalse(json.contains("localDateTime")); // null 字段不应出现
    }
}
```

重点测试项：

- **LocalDateTime / LocalDate / LocalTime** 的序列化格式是否符合预期
- **Date / Timestamp / java.sql.Date** 等传统日期类型是否兼容
- **时区转换**逻辑是否正确（服务端时区 ↔ 用户时区）
- **NON_NULL** 策略是否生效
- **@JsonProperty(access = READ_ONLY)** 注解是否正常工作

---

## 七、迁移步骤总结

整理一份可操作的迁移步骤清单：

### Step 1：升级依赖

Spring Boot 4.0 会自动管理 Jackson 3.x 的版本，通常不需要显式指定 Jackson 版本。

### Step 2：全局替换 import

```bash
# 批量替换包名（注意排除 annotation 包）
find . -name "*.java" -exec sed -i '' \
  's/com\.fasterxml\.jackson\.core/tools.jackson.core/g' {} +
find . -name "*.java" -exec sed -i '' \
  's/com\.fasterxml\.jackson\.databind/tools.jackson.databind/g' {} +
```

> 注意：不要替换 `com.fasterxml.jackson.annotation`，它保持不变。

### Step 3：替换核心类名

- `ObjectMapper` → `JsonMapper`
- `JsonSerializer` → `ValueSerializer`
- `JsonDeserializer` → `ValueDeserializer`
- `SerializerProvider` → `SerializationContext`
- `JsonProcessingException` → `JacksonException`

### Step 4：改造配置类

- 将 `@Bean ObjectMapper` 改为 `@Bean JsonMapperBuilderCustomizer`
- 删除 `@AutoConfigureBefore(JacksonAutoConfiguration.class)`
- 将 `JavaTimeModule` 改为 `SimpleModule`

### Step 5：处理共存场景

- Redis、第三方 SDK 等仍使用 Jackson 2.x 的组件，保持原有代码不变
- 添加注释说明共存原因，方便后续迁移

### Step 6：运行测试并修复

- 重点关注日期格式、属性排序、null 值处理等行为变化
- 检查 `SORT_PROPERTIES_ALPHABETICALLY` 默认值变化对 JSON 对比测试的影响

---

## 八、参考资料

- [Jackson 3.0 Release Notes](https://github.com/FasterXML/jackson/wiki/Jackson-Release-3.0) — 完整变更列表
- [Jackson 3 Migration Guide](https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md) — 官方迁移指南
- [Introducing Jackson 3 support in Spring](https://spring.io/blog/2025/10/07/introducing-jackson-3-support-in-spring) — Spring 官方博客
- [Jackson 3.0.0 GA Released](https://cowtowncoder.medium.com/jackson-3-0-0-ga-released-1f669cda529a) — Jackson 作者博客
- [Why Upgrade to Jackson 3?](https://medium.com/@cowtowncoder/why-upgrade-to-jackson-3-0-94c30e797bf2) — 升级收益分析
- [OpenRewrite Recipe for Jackson 2→3](https://docs.openrewrite.org/recipes/java/jackson/upgradejackson_2_3) — 自动化迁移工具

---

## 总结

Jackson 2.x → 3.x 的升级虽然是 Breaking Change，但整体迁移路径比较清晰：

1. **包名替换**是工作量最大但最机械的部分，可以批量处理
2. **核心类重命名**需要理解新旧对照关系，但命名更加直观
3. **配置方式变更**（Builder 模式）是设计理念的升级，代码更加安全
4. **默认值变化**需要逐一检查对业务的影响
5. **Jackson 2/3 共存**让渐进式迁移成为可能

Jackson 3 在安全性、API 设计、默认配置和性能上都有明显提升，建议在升级 Spring Boot 4.0 时同步完成迁移。
