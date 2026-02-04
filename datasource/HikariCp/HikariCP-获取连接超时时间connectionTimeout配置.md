> com.zaxxer.hikari.HikariConfig#getConnectionTimeout() 是 HikariCP（高性能 JDBC 连接池）中用于获取连接超时时间的核心配置方法。

##### 一、com.zaxxer.hikari.HikariConfig#getConnectionTimeout配置方法签名

```java
    public long getConnectionTimeout() {
        return this.connectionTimeout;
    }




```

- 返回值类型是long
- 单位毫秒（milliseconds）
- 默认值30000毫秒（即30秒）
- 控制客户端在尝试从连接池获取连接时最多等待多久

##### 二、com.zaxxer.hikari.HikariConfig对connectionTimeout初始化

```java
    static {
        //默认值30秒初始化
        CONNECTION_TIMEOUT = TimeUnit.SECONDS.toMillis(30L);
        VALIDATION_TIMEOUT = TimeUnit.SECONDS.toMillis(5L);
        SOFT_TIMEOUT_FLOOR = Long.getLong("com.zaxxer.hikari.timeoutMs.floor", 250L);
        IDLE_TIMEOUT = TimeUnit.MINUTES.toMillis(10L);
        MAX_LIFETIME = TimeUnit.MINUTES.toMillis(30L);
        DEFAULT_KEEPALIVE_TIME = TimeUnit.MINUTES.toMillis(2L);
        unitTest = false;
    }
```

HikariConfig类提供了静态代码块对CONNECTION_TIMEOUT进行了初始化；

##### 三、com.zaxxer.hikari.HikariConfig#setConnectionTimeout方法签名

```java
    public void setConnectionTimeout(long connectionTimeoutMs) {
        if (connectionTimeoutMs == 0L) {
            //关键：0 表示 "无限等待"（无超时）2147483647L = 2^31 - 1 ≈ 24.8 天
            this.connectionTimeout = 2147483647L;
        } else {
            //SOFT_TIMEOUT_FLOOR在二中静态代码块中初始化为250
            if (connectionTimeoutMs < SOFT_TIMEOUT_FLOOR) {
                throw new IllegalArgumentException("connectionTimeout cannot be less than " + SOFT_TIMEOUT_FLOOR + "ms");
            }

            this.connectionTimeout = connectionTimeoutMs;
        }

    }
```

- 当配置属性值为0时，会被设置为Integer最大整数；
- 当配置属性小于250毫秒时会抛出非法参数，不允许值小于250毫米；

##### 三、总结

- 永远不要设置为0；
- 生产环境建议设置1000~10000以实现快速失败和系统弹性；