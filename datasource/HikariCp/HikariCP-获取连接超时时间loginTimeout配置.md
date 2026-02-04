com.zaxxer.hikari.HikariDataSource#getLoginTimeout() 是 HikariCP 连接池中 HikariDataSource 类的一个方法，用于获取数据库连接的登录超时时间（login timeout），单位为秒。

##### 一、com.zaxxer.hikari.HikariDataSource#getLoginTimeout方法签名

```
    public int getLoginTimeout() throws SQLException {
        HikariPool p = this.pool;
        return p != null ? p.getUnwrappedDataSource().getLoginTimeout() : 0;
    }
```

- 返回值：int，表示登录超时时间（秒）。如果返回 0，通常表示无超时限制（即无限等待）。
- 异常：可能抛出 SQLException（尽管在 HikariCP 实现中通常不会抛出，但因实现 javax.sql.CommonDataSource 接口而声明）。

##### 二、上述方法是JDBC规范javax.sql.CommonDataSource接口定义的标准方法

```java
public interface CommonDataSource {
    void setLoginTimeout(int seconds) throws SQLException;
    int getLoginTimeout() throws SQLException;
}
```

> 驱动程序或连接池在尝试建立数据库连接时，最多等待多少秒。超过此时间仍未连上，则抛出超时异常。

说明：

- HikariCP 的实现中，这个值默认为 0（无超时），并且HikariCP 实际上并不使用这个设置来控制连接超时！
- getLoginTimeout() / setLoginTimeout() 在 HikariCP 中基本被忽略。
- HikariCP 使用自己的配置项来控制连接超时，主要是connectionTimeout，默认：30秒
- 即使你调用 hikariDataSource.setLoginTimeout(10)，HikariCP 不会将其用于底层连接建立逻辑。它只是存储这个值以满足 JDBC 接口契约。