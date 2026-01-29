##### 一、简介

> 自1.5.18版本起，Logback还支持XZ压缩。XZ通常比GZ实现更高的压缩比——尽管这需要更高的计算和内存使用——从而产生更小的文件大小，这对于日志保留是非常有用的。

##### 二、使用方法

要激活 XZ压缩，只需要将文件以.xz结尾，就像其它归档压缩文件使用TimeBasedRollingPolicy一样，示例：

```xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_DIR}/${LOG_FILE}.log</file> 
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${LOG_DIR}/%d{yyyy/MM}/${LOG_FILE}.xz</fileNamePattern>
        <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>
    <!-- ... -->
</appender>
```

此外，还需要为项目添加一个依赖项：

```xml
<!-- https://mvnrepository.com/artifact/org.tukaani/xz -->
<dependency>
    <groupId>org.tukaani</groupId>
    <artifactId>xz</artifactId>
    <version>1.10</version>
</dependency>
```

如果运行时缺少此库，则即使文件扩展名为.xz，Logback也会回退到GZ压缩。

参考文档：[XZ压缩使用说明文档](https://www.baeldung.com/logback#bd-4-rollingfileappender---compression)