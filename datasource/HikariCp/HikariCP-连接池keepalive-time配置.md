keepalive-time是HikariCP中用于定期探测空闲连接是否仍然有效，防止数据库服务器因长时间空闲而主动断开连接

MySQL的wait_timeout  默认 8 小时 = 28800000 ms

| 参数                  | 默认值             | 作用                                                         |
| --------------------- | ------------------ | ------------------------------------------------------------ |
| `wait_timeout`        | 28800 秒（8 小时） | 控制非交互式连接（如应用程序通过 JDBC/ODBC 建立的连接）的最大空闲时间 |
| `interactive_timeout` | 28800 秒（8 小时） | 控制交互式连接（如 MySQL CLI 客户端）的最大空闲时间          |

> 对于绝大多数Java应用程序（使用连接池），生效的是wait_timeout。

- 当一个连接连续空闲时间>=wait_timeout，MySQL服务端会主动关闭该连接。
- 客户端若未感知（如：连接池仍认为连接有效），下次使用时会抛出异常：

```shell
com.mysql.cj.exceptions.CommunicationsException: 
The last packet successfully received from the server was XXXXX ms ago...
```



##### 一、com.zaxxer.hikari.HikariConfig配置类中方法签名

```java
    public long getKeepaliveTime() {
        return this.keepaliveTime;
    }

    public void setKeepaliveTime(long keepaliveTimeMs) {
        this.keepaliveTime = keepaliveTimeMs;
    }
    static {
        CONNECTION_TIMEOUT = TimeUnit.SECONDS.toMillis(30L);
        VALIDATION_TIMEOUT = TimeUnit.SECONDS.toMillis(5L);
        SOFT_TIMEOUT_FLOOR = Long.getLong("com.zaxxer.hikari.timeoutMs.floor", 250L);
        IDLE_TIMEOUT = TimeUnit.MINUTES.toMillis(10L);
        MAX_LIFETIME = TimeUnit.MINUTES.toMillis(30L);
      	//默认值2分钟
        DEFAULT_KEEPALIVE_TIME = TimeUnit.MINUTES.toMillis(2L);
        unitTest = false;
    }
```

- 单位：毫秒
- 默认值：120000毫秒（2分钟）
- 探测连接是否有效，是防止数据库服务器长时间空闲主动断开连接，造成连接池中连接无效
- idle-timeout是HikariCP连接池中的连接是否有效的检测时间间隔，必须大于keepalive-time，否则keepalive-time无效；

##### 二、keepalive-time核心目的

许多数据库（如：MySQL、PostgreSQL）会主动关闭长时间空闲的连接（例如 MySQL 的 wait_timeout 默认为 8 小时）。如果应用在这之后尝试复用这个已失效的连接，就会抛出类似：

```shell
CommunicationsException: The last packet successfully received from the server was XXX ms ago...
```

keepalive-time就是为了解决这样的问题。

保活任务调度逻辑详解com.zaxxer.hikari.pool.HikariPool#createPoolEntry：

```java
   private PoolEntry createPoolEntry() {
        try {
            PoolEntry poolEntry = this.newPoolEntry(this.getTotalConnections() == 0);
            long maxLifetime = this.config.getMaxLifetime();
            long keepaliveTime;
            .......
						//1. 获取配置保活时间间隔
            keepaliveTime = this.config.getKeepaliveTime();
            if (keepaliveTime > 0L) {
               //2. 引入随机抖动（避免所有连接同时探测），最多20%的抖动
                variance = ThreadLocalRandom.current().nextLong(keepaliveTime / 5L);
              //3. 实际探测间隔 = keepaliveTime - 随机偏移量 (范围：80%~100%)
                long heartbeatTime = keepaliveTime - variance;
              //4. 调度周期性保活任务
                poolEntry.setKeepalive(this.houseKeepingExecutorService.scheduleWithFixedDelay(
                  new KeepaliveTask(poolEntry), //任务：验证连接有效性
                  heartbeatTime, //首次延迟
                  heartbeatTime, //后续固定间隔
                  TimeUnit.MILLISECONDS));
            }

            return poolEntry;
        } catch (PoolBase.ConnectionSetupException var10) {
            ........
        }

        return null;
    }
```

##### 三、com.zaxxer.hikari.pool.HikariPool.KeepaliveTask保活任务

```java
    private final class KeepaliveTask implements Runnable {
        private final PoolEntry poolEntry;

        KeepaliveTask(PoolEntry poolEntry) {
            this.poolEntry = poolEntry;
        }

        public void run() {
            //1. 尝试“预留”该连接（防止被其它线程同时使用）
            if (HikariPool.this.connectionBag.reserve(this.poolEntry)) {
              //2. 检测连接是否已死亡
                if (HikariPool.this.isConnectionDead(this.poolEntry.connection)) {
                    //3. 若死亡：软驱逐
                    HikariPool.this.softEvictConnection(this.poolEntry, "(connection is dead)", true);
                    //4. 补充新连接（因为驱逐了一个连接，池中可用连接数减少，需尝试补充）
                    HikariPool.this.addBagItem(HikariPool.this.connectionBag.getWaitingThreadCount());
                } else {
                    //5. 若存活：释放预留，记录debug日志
                    HikariPool.this.connectionBag.unreserve(this.poolEntry);
                    HikariPool.this.logger.debug("{} - keepalive: connection {} is alive", HikariPool.this.poolName, this.poolEntry.connection);
                }
            }
						//5. 如果预留失败（连接正被使用），直接跳过本次探测
        }
    }
```

- HikariPool.this.connectionBag.reserve(this.poolEntry)原子性锁定连接，确保探测期间，该连接不会被借出给应用线程
- HikariPool.this.isConnectionDead(this.poolEntry.connection)判断连接是否失效

```java
    boolean isConnectionDead(Connection connection) {
        try {
            this.setNetworkTimeout(connection, this.validationTimeout);
            try {
                int validationSeconds = (int)Math.max(1000L, this.validationTimeout) / 1000;
                // 优先使用JDBC4+的isValid方法，isValid()通过轻量级的ping实现，不执行SQL
                if (this.isUseJdbc4Validation) {
                    boolean var16 = !connection.isValid(validationSeconds);
                    return var16;
                }
                //若驱动不支持，会回退到connectionTestQuery
                Statement statement = connection.createStatement();
                try {
                    if (this.isNetworkTimeoutSupported != 1) {
                        this.setQueryTimeout(statement, validationSeconds);
                    }
                    statement.execute(this.config.getConnectionTestQuery());
                } catch (Throwable var12) {
                    ...
                }
                
            } finally {
                this.setNetworkTimeout(connection, (long)this.networkTimeout);
                if (this.isIsolateInternalQueries && !this.isAutoCommit) {
                    connection.rollback();
                }

            }

            return false;
        } catch (Exception var14) {
           ...
        }
    }
```

- softEvictConnection安全驱逐失效连接，标记连接为无效，但不立即关闭物理连接（实际关闭通常发生在连接被归还到池时或后台清理任务执行时）