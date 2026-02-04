> allow-pool-suspension用于控制是否允许在运行时暂停/恢复整个连接池，这是一个高级功能，主要用于维护、故障隔离或蓝绿部署等场景。

##### 一、com.zaxxer.hikari.HikariConfig#setAllowPoolSuspension方法

```java
    public boolean isAllowPoolSuspension() {
        return this.isAllowPoolSuspension;
    }

    public void setAllowPoolSuspension(boolean isAllowPoolSuspension) {
        this.checkIfSealed();
        this.isAllowPoolSuspension = isAllowPoolSuspension;
    }
    private void checkIfSealed() {
        if (this.sealed) {
            throw new IllegalStateException("The configuration of the pool is sealed once started. Use HikariConfigMXBean for runtime changes.");
        }
    }
```

- isAllowPoolSuspension()：返回是否启用连接池挂起功能，默认值：false。

- setAllowPoolSuspension(boolean)：设置该开关（必须在池初始化前调用）。

- checkIfSealed() 确保该配置只能在 HikariDataSource 初始化之前修改。

- 一旦 `HikariDataSource` 被创建（即连接池已启动），再调用 `setAllowPoolSuspension()` 会抛出：

  ```shell
  The configuration of the pool is sealed once started. Use HikariConfigMXBean for runtime changes.
  ```

  

##### 二、isAllowPoolSuspension默认值

- isAllowPoolSuspension：默认值是false;
- allowPoolSuspension是一个启动前的配置，不能动态开启；

##### 三、挂起行为描述

| 操作      | 对getConnection的影响                   |
| --------- | --------------------------------------- |
| 正常状态  | 返回可用连接（或等待新建连接）          |
| 挂起状态  | 永久阻塞（无视connectionTimeout）       |
| 挂起+超时 | connectionTimeout不生效，线程会一直等待 |

严重警告：如果启用了allowPoolSuspension，但是忘记resume()，应用将永久阻塞。

##### 四、总结

| 属性     | `allowPoolSuspension`                                        |
| -------- | ------------------------------------------------------------ |
| 作用     | 允许通过 `suspend()/resume()` 动态挂起/恢复连接池            |
| 默认值   | `false`（强烈建议保持默认）                                  |
| 配置时机 | 必须在 `HikariDataSource` 创建前设置                         |
| 风险     | 挂起后 `getConnection()` 永久阻塞（无视 `connectionTimeout`） |
| 适用场景 | 高级运维操作（部署、维护、故障隔离）                         |
| 替代方案 | 大多数场景可用 熔断器（如 Resilience4j） 或 关闭数据源（`close()`） 替代 |