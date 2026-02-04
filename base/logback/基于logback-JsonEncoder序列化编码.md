##### 一、简介

JsonEncoder是一个JSON编码器，日志文档中的每一行日志对应一个JSON对象，适用于结构化记录日志。其中包含时间戳、级别、消息等字段。

此编码器支持通过布尔值进行配置，以控制在输出中包含或排除特定字段，从而满足不同的日志记录需求。

##### 二、通过Java编码方式控制编码器

```java
    public Encoder<ILoggingEvent> getEncoder(String pattern) {
        JsonEncoder encoder = new JsonEncoder();
        encoder.setContext(context);
        //启用或禁用在编码输出中包含日志级别，默认：true
        encoder.setWithLevel(true);
        //启用或禁用在编码输出中包含记录器名称的功能，默认：true
        encoder.setWithLoggerName(true);
        //启用或禁用包含记录器上下文信息的功能。
        encoder.setWithContext(true);
        //启用或禁用将格式化消息包含在编码输出中的功能。默认：false
        encoder.setWithFormattedMessage(false);
        //启用或禁用在编码输出中包含原始消息文本的功能。默认：true
        encoder.setWithMessage(true);
        //设置是否在每个编码事件中包含序列号。默认：true
        encoder.setWithSequenceNumber(true);
        //设置是否在每个编码事件中包含事件时间戳。默认：true
        encoder.setWithTimestamp(true);
        //设置时间戳输出中是否包含纳秒。默认：true
        encoder.setWithNanoseconds(true);
        //启用或禁用在编码输出中包含MDC属性。默认：true
        encoder.setWithMDC(true);
        //启用或禁用在编码输出中包含标记的功能。默认：true
        encoder.setWithMarkers(true);
        //启用或禁用包含附加到日志事件中的键值对。默认：true
        encoder.setWithKVPList(true);
        //启用或禁用在编码输出中包含线程名称的功能。默认：true
        encoder.setWithThreadName(true);
        //启用或禁用在编码输出中包含可抛出信息。默认：true
        encoder.setWithThrowable(true);
        encoder.addInfo("Build LogbackJsonEncoder Success");
        encoder.start();
        return encoder;
    }
```

输出的日志【手动格式化后的结果】：

```json
{
  "sequenceNumber": 0,
  "timestamp": 1769740613951,
  "nanoseconds": 951757000,
  "level": "INFO",
  "threadName": "http-nio-8080-exec-5",
  "loggerName": "MODULEtest1.tt0.com.emily.sample.logger.controller.LogbackController",
  "context": {
    "name": "default",
    "birthdate": 1769740399374,
    "properties": {
      "APPLICATION_GROUP": "null",
      "LOG_CORRELATION_PATTERN": "null",
      "LOG_DATEFORMAT_PATTERN": "null",
      "PID": "12602",
      "LOG_EXCEPTION_CONVERSION_WORD": "null",
      "CONSOLE_LOG_CHARSET": "UTF-8",
      "CONSOLE_LOG_THRESHOLD": "TRACE",
      "FILE_LOG_PATTERN": "%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX} %5p 12602 --- %esb(){APPLICATION_NAME}%esb{APPLICATION_GROUP}[%t] %-40.40logger{39} : %m%n%wEx",
      "FILE_LOG_CHARSET": "UTF-8",
      "FILE_LOG_STRUCTURED_FORMAT": "",
      "CONSOLE_LOG_STRUCTURED_FORMAT": "",
      "CONSOLE_LOG_PATTERN": "%clr(%d{yyyy-MM-dd'T'HH:mm:ss.SSSXXX}){faint} %clr(%5p){} %clr(12602){magenta} %clr(--- %esb(){APPLICATION_NAME}%esb{APPLICATION_GROUP}[%15.15t] ){faint}%clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n%wEx",
      "APPLICATION_NAME": "null",
      "FILE_LOG_THRESHOLD": "TRACE",
      "LOG_LEVEL_PATTERN": "null"
    }
  },
  "mdc": {},
  "message": "ni-----------------1769740613951",
  "throwable": null
}
```

