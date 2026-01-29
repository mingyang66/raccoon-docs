##### 一、简介

基于logback布局Layouts格式化日志消息，与logback的其他部分一样，布局是可扩展的，我们可以创建自己的布局。然而，默认的PatternLayout已经提供了大多数应用程序所需的功能，甚至更多。

##### 二、使用

PatternLayout使用示例：

```xml
<encoder>
    <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder>
```

这个配置脚本包含了PatternLayoutEncoder的配置，我们将一个Encoder传递给我们的Appender，这个encoder使用PatternLayout来格式化消息。

<pattern>标签中的文本定义了日志消息的格式化方式。PatternLayout实现了多种转换词和格式修饰符，用于创建格式化模式。

PatternLay通过%来识别转换词，因此模式中的转换才会生效：

- %d{HH:mm:ss.SSS} 包含小时、分钟、秒、毫秒的时间字符串；
- [%thread] 生成日志消息的线程名，用方括号括起来；
- %-5level 日志事件的级别，填充至少5个字符；
- %logger{36} logger的名称，最多35个字符；
- %msg%n 日志消息，及换行符；

```shell
21:32:10.311 [main] DEBUG com.baeldung.logback.LogbackTests - Logging message: This is a String
```

