> minimum-idle是HikariCP用于控制连接池中保持最小空闲连接数的配置。这个参数与maximum-pool-size配合，决定了连接池是否具备弹性伸缩能力（即：在负载低的时候释放多余连接）

##### 一、getter|setter方法

```java
    public int getMinimumIdle() {
        return this.minIdle;
    }

    public void setMinimumIdle(int minIdle) {
        if (minIdle < 0) {
            throw new IllegalArgumentException("minimumIdle cannot be negative");
        } else {
            this.minIdle = minIdle;
        }
    }
```

> 设置该值的时候必须大于等于0，否则会抛出异常

##### 二、com.zaxxer.hikari.HikariConfig#HikariConfig()构造函数初始化

```java
    public HikariConfig() {
        this.credentials = new AtomicReference(Credentials.of((String)null, (String)null));
        this.dataSourceProperties = new Properties();
        this.healthCheckProperties = new Properties();
        this.minIdle = -1;
        this.maxPoolSize = 10;
        this.maxLifetime = MAX_LIFETIME;
        this.connectionTimeout = CONNECTION_TIMEOUT;
        this.validationTimeout = VALIDATION_TIMEOUT;
        this.idleTimeout = IDLE_TIMEOUT;
        this.initializationFailTimeout = 1L;
        this.isAutoCommit = true;
        this.keepaliveTime = DEFAULT_KEEPALIVE_TIME;
        String systemProp = System.getProperty("hikaricp.configurationFile");
        if (systemProp != null) {
            this.loadProperties(systemProp);
        }

    }
```

- 构造函数初始化为-1

##### 三、com.zaxxer.hikari.HikariConfig#validateNumerics方法对配置进行校验

```java
        if (this.minIdle < 0 || this.minIdle > this.maxPoolSize) {
            this.minIdle = this.maxPoolSize;
        }
```

> 当minimum-idle小于0，或minimum-idle大于maximum-pool-size的时候数据库连接池最小连接池和最大连接池相等。

##### 四、最小连接数minimum-idle、最大连接数maximum-pool-size组合场景分析

| 场景                                      | 行为                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| minimum-idle == maximum-pool-size（默认） | 固定大小池<br/>• 启动时创建 maximum-pool-size 个连接<br/>• 空闲连接永不回收（idleTimeout 被忽略） |
| minimum-idle < maximum-pool-size          | 弹性池<br/>• 空闲连接超过 idleTimeout 且池大小 > minimumIdle 时，会被回收<br/>• 负载高时可扩容至 maximumPoolSize |
| minimum-idle =0                           | 极致弹性<br/>• 空闲时可释放所有连接（但首次请求仍需创建）<br/>• 适合极低频访问的场景（如定时任务） |

