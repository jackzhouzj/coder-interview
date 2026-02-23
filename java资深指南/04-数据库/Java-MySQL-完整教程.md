# Java 直接操作 MySQL 数据库 - 完整教程

## 目录
- [1. 环境准备](#1-环境准备)
- [2. JDBC 基础](#2-jdbc-基础)
- [3. 数据库连接](#3-数据库连接)
- [4. CRUD 操作](#4-crud-操作)
- [5. 事务管理](#5-事务管理)
- [6. 连接池](#6-连接池)
- [7. 最佳实践](#7-最佳实践)
- [8. 常见问题](#8-常见问题)

---

## 1. 环境准备

### 1.1 Maven 依赖

```xml
<dependencies>
    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
    
    <!-- HikariCP 连接池（可选，推荐） -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>5.0.1</version>
    </dependency>
</dependencies>
```

### 1.2 数据库准备

```sql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS test_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE test_db;

-- 创建用户表
CREATE TABLE user_info (
    id BIGINT AUTO_INCREMENT PRIMARY KEY COMMENT '主键ID',
    user_name VARCHAR(64) NOT NULL COMMENT '用户名',
    age TINYINT NOT NULL DEFAULT 0 COMMENT '年龄',
    email VARCHAR(128) NOT NULL COMMENT '邮箱',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    INDEX idx_user_name (user_name),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表';
```

---

## 2. JDBC 基础

### 2.1 JDBC 核心接口

```java
/**
 * JDBC 核心接口说明
 * 
 * @author erik.zhou
 */
public class JdbcCoreInterfaces {
    
    /**
     * 1. DriverManager：管理数据库驱动
     * 2. Connection：数据库连接对象
     * 3. Statement：执行静态 SQL 语句
     * 4. PreparedStatement：执行预编译 SQL 语句（推荐）
     * 5. CallableStatement：执行存储过程
     * 6. ResultSet：结果集对象
     */
}
```

### 2.2 JDBC 工作流程

```java
/**
 * JDBC 操作流程示例
 * 
 * @author erik.zhou
 */
public class JdbcWorkflow {
    
    public static void main(String[] args) {
        // 1. 加载驱动（MySQL 8.0+ 可省略）
        // Class.forName("com.mysql.cj.jdbc.Driver");
        
        // 2. 获取连接
        // 3. 创建 Statement
        // 4. 执行 SQL
        // 5. 处理结果
        // 6. 关闭资源
    }
}
```

---

## 3. 数据库连接

### 3.1 基础连接方式

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * 数据库连接工具类（基础版）
 * 
 * @author erik.zhou
 */
public final class DbConnectionUtil {
    
    // 数据库连接配置
    private static final String URL = "jdbc:mysql://localhost:3306/test_db?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "your_password";
    
    // 私有构造函数，禁止实例化
    private DbConnectionUtil() {
    }
    
    /**
     * 获取数据库连接
     * 
     * @return 数据库连接对象
     * @throws SQLException SQL异常
     */
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USERNAME, PASSWORD);
    }
    
    /**
     * 关闭数据库连接
     * 
     * @param connection 数据库连接对象
     */
    public static void closeConnection(Connection connection) {
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 3.2 配置文件方式


**db.properties 配置文件：**

```properties
# 数据库连接配置
db.url=jdbc:mysql://localhost:3306/test_db?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
db.username=root
db.password=your_password
db.driver=com.mysql.cj.jdbc.Driver
```

**配置文件读取工具类：**

```java
import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;

/**
 * 数据库连接工具类（配置文件版）
 * 
 * @author erik.zhou
 */
public final class DbConfigUtil {
    
    private static final Properties PROPERTIES = new Properties();
    
    // 静态代码块加载配置文件
    static {
        try (InputStream inputStream = DbConfigUtil.class.getClassLoader().getResourceAsStream("db.properties")) {
            if (inputStream == null) {
                throw new RuntimeException("配置文件 db.properties 不存在");
            }
            PROPERTIES.load(inputStream);
        } catch (IOException e) {
            throw new RuntimeException("加载配置文件失败", e);
        }
    }
    
    // 私有构造函数，禁止实例化
    private DbConfigUtil() {
    }
    
    /**
     * 获取数据库连接
     * 
     * @return 数据库连接对象
     * @throws SQLException SQL异常
     */
    public static Connection getConnection() throws SQLException {
        String url = PROPERTIES.getProperty("db.url");
        String username = PROPERTIES.getProperty("db.username");
        String password = PROPERTIES.getProperty("db.password");
        return DriverManager.getConnection(url, username, password);
    }
}
```

---

## 4. CRUD 操作

### 4.1 实体类定义

```java
import java.time.LocalDateTime;

/**
 * 用户实体类
 * 
 * @author erik.zhou
 */
public class UserInfo {
    
    private Long id;
    private String userName;
    private Integer age;
    private String email;
    private Integer status;
    private LocalDateTime createTime;
    private LocalDateTime updateTime;
    
    // 构造函数
    public UserInfo() {
    }
    
    public UserInfo(String userName, Integer age, String email) {
        this.userName = userName;
        this.age = age;
        this.email = email;
    }
    
    // Getter 和 Setter 方法
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getUserName() {
        return userName;
    }
    
    public void setUserName(String userName) {
        this.userName = userName;
    }
    
    public Integer getAge() {
        return age;
    }
    
    public void setAge(Integer age) {
        this.age = age;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
    
    public Integer getStatus() {
        return status;
    }
    
    public void setStatus(Integer status) {
        this.status = status;
    }
    
    public LocalDateTime getCreateTime() {
        return createTime;
    }
    
    public void setCreateTime(LocalDateTime createTime) {
        this.createTime = createTime;
    }
    
    public LocalDateTime getUpdateTime() {
        return updateTime;
    }
    
    public void setUpdateTime(LocalDateTime updateTime) {
        this.updateTime = updateTime;
    }
    
    @Override
    public String toString() {
        return "UserInfo{" +
                "id=" + id +
                ", userName='" + userName + '\'' +
                ", age=" + age +
                ", email='" + email + '\'' +
                ", status=" + status +
                ", createTime=" + createTime +
                ", updateTime=" + updateTime +
                '}';
    }
}
```

### 4.2 插入操作（INSERT）

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

/**
 * 用户数据访问对象（DAO）
 * 
 * @author erik.zhou
 */
public class UserDao {
    
    /**
     * 插入用户（返回自增主键）
     * 
     * @param userInfo 用户信息
     * @return 自增主键ID
     */
    public Long insertUser(UserInfo userInfo) {
        // SQL 语句（禁止使用 SELECT *）
        String sql = "INSERT INTO user_info(user_name, age, email) VALUES (?, ?, ?)";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        
        try {
            // 获取数据库连接
            connection = DbConfigUtil.getConnection();
            
            // 创建 PreparedStatement（指定返回自增主键）
            preparedStatement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
            
            // 设置参数（索引从 1 开始）
            preparedStatement.setString(1, userInfo.getUserName());
            preparedStatement.setInt(2, userInfo.getAge());
            preparedStatement.setString(3, userInfo.getEmail());
            
            // 执行插入操作
            int affectedRows = preparedStatement.executeUpdate();
            
            if (affectedRows > 0) {
                // 获取自增主键
                resultSet = preparedStatement.getGeneratedKeys();
                if (resultSet.next()) {
                    return resultSet.getLong(1);
                }
            }
            
            return null;
            
        } catch (SQLException e) {
            throw new RuntimeException("插入用户失败", e);
        } finally {
            // 关闭资源（顺序：ResultSet -> Statement -> Connection）
            closeResultSet(resultSet);
            closeStatement(preparedStatement);
            closeConnection(connection);
        }
    }
    
    /**
     * 批量插入用户（推荐方式）
     * 
     * @param userList 用户列表
     * @return 插入成功的记录数
     */
    public int batchInsertUsers(List<UserInfo> userList) {
        if (userList == null || userList.isEmpty()) {
            return 0;
        }
        
        // 批量插入 SQL（单次≤1000 条）
        String sql = "INSERT INTO user_info(user_name, age, email) VALUES (?, ?, ?)";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            
            // 批量添加参数
            for (UserInfo userInfo : userList) {
                preparedStatement.setString(1, userInfo.getUserName());
                preparedStatement.setInt(2, userInfo.getAge());
                preparedStatement.setString(3, userInfo.getEmail());
                preparedStatement.addBatch();
            }
            
            // 执行批量插入
            int[] results = preparedStatement.executeBatch();
            
            // 统计成功插入的记录数
            int successCount = 0;
            for (int result : results) {
                if (result > 0) {
                    successCount++;
                }
            }
            
            return successCount;
            
        } catch (SQLException e) {
            throw new RuntimeException("批量插入用户失败", e);
        } finally {
            closeStatement(preparedStatement);
            closeConnection(connection);
        }
    }
    
    // 资源关闭方法
    private void closeResultSet(ResultSet resultSet) {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    
    private void closeStatement(Statement statement) {
        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    
    private void closeConnection(Connection connection) {
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 4.3 查询操作（SELECT）

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * 用户查询操作
 * 
 * @author erik.zhou
 */
public class UserQueryDao {
    
    /**
     * 根据 ID 查询用户
     * 
     * @param id 用户ID
     * @return 用户信息，不存在返回 null
     */
    public UserInfo selectUserById(Long id) {
        // 显式指定字段名，禁止 SELECT *
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info WHERE id = ?";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setLong(1, id);
            
            resultSet = preparedStatement.executeQuery();
            
            if (resultSet.next()) {
                return buildUserInfo(resultSet);
            }
            
            return null;
            
        } catch (SQLException e) {
            throw new RuntimeException("查询用户失败", e);
        } finally {
            closeResources(resultSet, preparedStatement, connection);
        }
    }
    
    /**
     * 查询所有用户（返回空集合而非 null）
     * 
     * @return 用户列表
     */
    public List<UserInfo> selectAllUsers() {
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info ORDER BY id";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            resultSet = preparedStatement.executeQuery();
            
            List<UserInfo> userList = new ArrayList<>(100);
            
            while (resultSet.next()) {
                userList.add(buildUserInfo(resultSet));
            }
            
            return userList;
            
        } catch (SQLException e) {
            throw new RuntimeException("查询用户列表失败", e);
        } finally {
            closeResources(resultSet, preparedStatement, connection);
        }
    }
    
    /**
     * 分页查询用户（必须加 ORDER BY）
     * 
     * @param offset 偏移量
     * @param limit 每页数量
     * @return 用户列表
     */
    public List<UserInfo> selectUsersByPage(int offset, int limit) {
        // 分页查询必须加 ORDER BY
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info ORDER BY id LIMIT ?, ?";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setInt(1, offset);
            preparedStatement.setInt(2, limit);
            
            resultSet = preparedStatement.executeQuery();
            
            List<UserInfo> userList = new ArrayList<>(limit);
            
            while (resultSet.next()) {
                userList.add(buildUserInfo(resultSet));
            }
            
            return userList;
            
        } catch (SQLException e) {
            throw new RuntimeException("分页查询用户失败", e);
        } finally {
            closeResources(resultSet, preparedStatement, connection);
        }
    }
    
    /**
     * 根据用户名模糊查询
     * 
     * @param userName 用户名关键字
     * @return 用户列表
     */
    public List<UserInfo> selectUsersByName(String userName) {
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info WHERE user_name LIKE ? ORDER BY id";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setString(1, "%" + userName + "%");
            
            resultSet = preparedStatement.executeQuery();
            
            List<UserInfo> userList = new ArrayList<>(50);
            
            while (resultSet.next()) {
                userList.add(buildUserInfo(resultSet));
            }
            
            return userList;
            
        } catch (SQLException e) {
            throw new RuntimeException("模糊查询用户失败", e);
        } finally {
            closeResources(resultSet, preparedStatement, connection);
        }
    }
    
    /**
     * 构建用户对象
     * 
     * @param resultSet 结果集
     * @return 用户信息
     * @throws SQLException SQL异常
     */
    private UserInfo buildUserInfo(ResultSet resultSet) throws SQLException {
        UserInfo userInfo = new UserInfo();
        userInfo.setId(resultSet.getLong("id"));
        userInfo.setUserName(resultSet.getString("user_name"));
        userInfo.setAge(resultSet.getInt("age"));
        userInfo.setEmail(resultSet.getString("email"));
        userInfo.setStatus(resultSet.getInt("status"));
        
        // 处理时间字段
        if (resultSet.getTimestamp("create_time") != null) {
            userInfo.setCreateTime(resultSet.getTimestamp("create_time").toLocalDateTime());
        }
        if (resultSet.getTimestamp("update_time") != null) {
            userInfo.setUpdateTime(resultSet.getTimestamp("update_time").toLocalDateTime());
        }
        
        return userInfo;
    }
    
    /**
     * 关闭资源
     */
    private void closeResources(ResultSet resultSet, PreparedStatement preparedStatement, Connection connection) {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 4.4 更新操作（UPDATE）

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * 用户更新操作
 * 
 * @author erik.zhou
 */
public class UserUpdateDao {
    
    /**
     * 更新用户信息
     * 
     * @param userInfo 用户信息
     * @return 更新成功的记录数
     */
    public int updateUser(UserInfo userInfo) {
        String sql = "UPDATE user_info SET user_name = ?, age = ?, email = ? WHERE id = ?";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            
            preparedStatement.setString(1, userInfo.getUserName());
            preparedStatement.setInt(2, userInfo.getAge());
            preparedStatement.setString(3, userInfo.getEmail());
            preparedStatement.setLong(4, userInfo.getId());
            
            return preparedStatement.executeUpdate();
            
        } catch (SQLException e) {
            throw new RuntimeException("更新用户失败", e);
        } finally {
            closeResources(preparedStatement, connection);
        }
    }
    
    /**
     * 更新用户状态
     * 
     * @param id 用户ID
     * @param status 状态值
     * @return 更新成功的记录数
     */
    public int updateUserStatus(Long id, Integer status) {
        String sql = "UPDATE user_info SET status = ? WHERE id = ?";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            
            preparedStatement.setInt(1, status);
            preparedStatement.setLong(2, id);
            
            return preparedStatement.executeUpdate();
            
        } catch (SQLException e) {
            throw new RuntimeException("更新用户状态失败", e);
        } finally {
            closeResources(preparedStatement, connection);
        }
    }
    
    /**
     * 关闭资源
     */
    private void closeResources(PreparedStatement preparedStatement, Connection connection) {
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 4.5 删除操作（DELETE）

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * 用户删除操作
 * 
 * @author erik.zhou
 */
public class UserDeleteDao {
    
    /**
     * 根据 ID 删除用户
     * 
     * @param id 用户ID
     * @return 删除成功的记录数
     */
    public int deleteUserById(Long id) {
        String sql = "DELETE FROM user_info WHERE id = ?";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setLong(1, id);
            
            return preparedStatement.executeUpdate();
            
        } catch (SQLException e) {
            throw new RuntimeException("删除用户失败", e);
        } finally {
            closeResources(preparedStatement, connection);
        }
    }
    
    /**
     * 批量删除用户
     * 
     * @param idList 用户ID列表
     * @return 删除成功的记录数
     */
    public int batchDeleteUsers(List<Long> idList) {
        if (idList == null || idList.isEmpty()) {
            return 0;
        }
        
        String sql = "DELETE FROM user_info WHERE id = ?";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        
        try {
            connection = DbConfigUtil.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            
            for (Long id : idList) {
                preparedStatement.setLong(1, id);
                preparedStatement.addBatch();
            }
            
            int[] results = preparedStatement.executeBatch();
            
            int successCount = 0;
            for (int result : results) {
                if (result > 0) {
                    successCount++;
                }
            }
            
            return successCount;
            
        } catch (SQLException e) {
            throw new RuntimeException("批量删除用户失败", e);
        } finally {
            closeResources(preparedStatement, connection);
        }
    }
    
    /**
     * 关闭资源
     */
    private void closeResources(PreparedStatement preparedStatement, Connection connection) {
        if (preparedStatement != null) {
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

---

## 5. 事务管理

### 5.1 事务基础

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * 事务管理示例
 * 
 * @author erik.zhou
 */
public class TransactionExample {
    
    /**
     * 转账操作（事务示例）
     * 
     * @param fromUserId 转出用户ID
     * @param toUserId 转入用户ID
     * @param amount 转账金额
     * @return 是否成功
     */
    public boolean transfer(Long fromUserId, Long toUserId, Integer amount) {
        Connection connection = null;
        PreparedStatement deductStatement = null;
        PreparedStatement addStatement = null;
        
        try {
            // 获取连接
            connection = DbConfigUtil.getConnection();
            
            // 关闭自动提交
            connection.setAutoCommit(false);
            
            // 扣减转出用户余额
            String deductSql = "UPDATE user_info SET age = age - ? WHERE id = ?";
            deductStatement = connection.prepareStatement(deductSql);
            deductStatement.setInt(1, amount);
            deductStatement.setLong(2, fromUserId);
            deductStatement.executeUpdate();
            
            // 模拟异常（测试事务回滚）
            // if (true) throw new RuntimeException("模拟异常");
            
            // 增加转入用户余额
            String addSql = "UPDATE user_info SET age = age + ? WHERE id = ?";
            addStatement = connection.prepareStatement(addSql);
            addStatement.setInt(1, amount);
            addStatement.setLong(2, toUserId);
            addStatement.executeUpdate();
            
            // 提交事务
            connection.commit();
            
            return true;
            
        } catch (Exception e) {
            // 回滚事务
            if (connection != null) {
                try {
                    connection.rollback();
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
            throw new RuntimeException("转账失败", e);
        } finally {
            // 恢复自动提交
            if (connection != null) {
                try {
                    connection.setAutoCommit(true);
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            
            // 关闭资源
            closeResources(deductStatement, addStatement, connection);
        }
    }
    
    /**
     * 关闭资源
     */
    private void closeResources(PreparedStatement... statements) {
        for (PreparedStatement statement : statements) {
            if (statement != null) {
                try {
                    statement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    private void closeResources(PreparedStatement statement1, PreparedStatement statement2, Connection connection) {
        if (statement1 != null) {
            try {
                statement1.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (statement2 != null) {
            try {
                statement2.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

---

## 6. 连接池

### 6.1 HikariCP 连接池配置

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import java.sql.Connection;
import java.sql.SQLException;

/**
 * HikariCP 连接池工具类
 * 
 * @author erik.zhou
 */
public final class HikariCpUtil {
    
    private static final HikariDataSource DATA_SOURCE;
    
    // 静态代码块初始化连接池
    static {
        HikariConfig config = new HikariConfig();
        
        // 数据库连接配置
        config.setJdbcUrl("jdbc:mysql://localhost:3306/test_db?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai");
        config.setUsername("root");
        config.setPassword("your_password");
        config.setDriverClassName("com.mysql.cj.jdbc.Driver");
        
        // 连接池配置
        config.setMaximumPoolSize(10);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        
        // 连接池名称
        config.setPoolName("HikariCP-Pool");
        
        DATA_SOURCE = new HikariDataSource(config);
    }
    
    // 私有构造函数，禁止实例化
    private HikariCpUtil() {
    }
    
    /**
     * 获取数据库连接
     * 
     * @return 数据库连接对象
     * @throws SQLException SQL异常
     */
    public static Connection getConnection() throws SQLException {
        return DATA_SOURCE.getConnection();
    }
    
    /**
     * 关闭连接池
     */
    public static void close() {
        if (DATA_SOURCE != null && !DATA_SOURCE.isClosed()) {
            DATA_SOURCE.close();
        }
    }
}
```

### 6.2 配置文件方式

**hikari.properties 配置文件：**

```properties
# 数据库连接配置
jdbcUrl=jdbc:mysql://localhost:3306/test_db?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
username=root
password=your_password
driverClassName=com.mysql.cj.jdbc.Driver

# 连接池配置
maximumPoolSize=10
minimumIdle=5
connectionTimeout=30000
idleTimeout=600000
maxLifetime=1800000
poolName=HikariCP-Pool
```

**配置文件加载工具类：**

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * HikariCP 连接池工具类（配置文件版）
 * 
 * @author erik.zhou
 */
public final class HikariCpConfigUtil {
    
    private static final HikariDataSource DATA_SOURCE;
    
    static {
        try (InputStream inputStream = HikariCpConfigUtil.class.getClassLoader()
                .getResourceAsStream("hikari.properties")) {
            
            if (inputStream == null) {
                throw new RuntimeException("配置文件 hikari.properties 不存在");
            }
            
            Properties properties = new Properties();
            properties.load(inputStream);
            
            HikariConfig config = new HikariConfig(properties);
            DATA_SOURCE = new HikariDataSource(config);
            
        } catch (IOException e) {
            throw new RuntimeException("加载连接池配置失败", e);
        }
    }
    
    // 私有构造函数，禁止实例化
    private HikariCpConfigUtil() {
    }
    
    /**
     * 获取数据库连接
     * 
     * @return 数据库连接对象
     * @throws SQLException SQL异常
     */
    public static Connection getConnection() throws SQLException {
        return DATA_SOURCE.getConnection();
    }
    
    /**
     * 关闭连接池
     */
    public static void close() {
        if (DATA_SOURCE != null && !DATA_SOURCE.isClosed()) {
            DATA_SOURCE.close();
        }
    }
}
```

---

## 7. 最佳实践

### 7.1 使用 try-with-resources

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * try-with-resources 最佳实践
 * 
 * @author erik.zhou
 */
public class BestPracticeExample {
    
    /**
     * 使用 try-with-resources 自动关闭资源
     * 
     * @param id 用户ID
     * @return 用户信息
     */
    public UserInfo selectUserById(Long id) {
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info WHERE id = ?";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
            
            preparedStatement.setLong(1, id);
            
            try (ResultSet resultSet = preparedStatement.executeQuery()) {
                if (resultSet.next()) {
                    UserInfo userInfo = new UserInfo();
                    userInfo.setId(resultSet.getLong("id"));
                    userInfo.setUserName(resultSet.getString("user_name"));
                    userInfo.setAge(resultSet.getInt("age"));
                    userInfo.setEmail(resultSet.getString("email"));
                    userInfo.setStatus(resultSet.getInt("status"));
                    
                    if (resultSet.getTimestamp("create_time") != null) {
                        userInfo.setCreateTime(resultSet.getTimestamp("create_time").toLocalDateTime());
                    }
                    if (resultSet.getTimestamp("update_time") != null) {
                        userInfo.setUpdateTime(resultSet.getTimestamp("update_time").toLocalDateTime());
                    }
                    
                    return userInfo;
                }
            }
            
            return null;
            
        } catch (SQLException e) {
            throw new RuntimeException("查询用户失败", e);
        }
    }
}
```

### 7.2 SQL 注入防护

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Pattern;

/**
 * SQL 注入防护示例
 * 
 * @author erik.zhou
 */
public class SqlInjectionPreventionExample {
    
    private static final Pattern USER_NAME_PATTERN = Pattern.compile("^[a-zA-Z0-9_]{3,20}$");
    
    /**
     * 错误示例：拼接 SQL（存在 SQL 注入风险）
     * 
     * @param userName 用户名
     * @return 用户列表
     */
    @Deprecated
    public List<UserInfo> selectUsersByNameBad(String userName) {
        // 危险：直接拼接 SQL，存在 SQL 注入风险
        String sql = "SELECT id, user_name, age, email FROM user_info WHERE user_name = '" + userName + "'";
        
        // 如果 userName = "admin' OR '1'='1"，则 SQL 变为：
        // SELECT id, user_name, age, email FROM user_info WHERE user_name = 'admin' OR '1'='1'
        // 这将返回所有用户数据
        
        return new ArrayList<>();
    }
    
    /**
     * 正确示例：使用 PreparedStatement（防止 SQL 注入）
     * 
     * @param userName 用户名
     * @return 用户列表
     */
    public List<UserInfo> selectUsersByNameGood(String userName) {
        // 参数校验
        if (userName == null || !USER_NAME_PATTERN.matcher(userName).matches()) {
            throw new IllegalArgumentException("用户名格式非法");
        }
        
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info WHERE user_name = ? ORDER BY id";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
            
            // 使用占位符，PreparedStatement 会自动转义特殊字符
            preparedStatement.setString(1, userName);
            
            try (ResultSet resultSet = preparedStatement.executeQuery()) {
                List<UserInfo> userList = new ArrayList<>(10);
                
                while (resultSet.next()) {
                    UserInfo userInfo = new UserInfo();
                    userInfo.setId(resultSet.getLong("id"));
                    userInfo.setUserName(resultSet.getString("user_name"));
                    userInfo.setAge(resultSet.getInt("age"));
                    userInfo.setEmail(resultSet.getString("email"));
                    userInfo.setStatus(resultSet.getInt("status"));
                    
                    if (resultSet.getTimestamp("create_time") != null) {
                        userInfo.setCreateTime(resultSet.getTimestamp("create_time").toLocalDateTime());
                    }
                    if (resultSet.getTimestamp("update_time") != null) {
                        userInfo.setUpdateTime(resultSet.getTimestamp("update_time").toLocalDateTime());
                    }
                    
                    userList.add(userInfo);
                }
                
                return userList;
            }
            
        } catch (SQLException e) {
            throw new RuntimeException("查询用户失败", e);
        }
    }
}
```

### 7.3 完整的 DAO 示例

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

/**
 * 用户 DAO 完整示例
 * 
 * @author erik.zhou
 */
public class UserCompleteDao {
    
    /**
     * 插入用户
     * 
     * @param userInfo 用户信息
     * @return 自增主键ID
     */
    public Long insertUser(UserInfo userInfo) {
        String sql = "INSERT INTO user_info(user_name, age, email) VALUES (?, ?, ?)";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            
            preparedStatement.setString(1, userInfo.getUserName());
            preparedStatement.setInt(2, userInfo.getAge());
            preparedStatement.setString(3, userInfo.getEmail());
            
            int affectedRows = preparedStatement.executeUpdate();
            
            if (affectedRows > 0) {
                try (ResultSet resultSet = preparedStatement.getGeneratedKeys()) {
                    if (resultSet.next()) {
                        return resultSet.getLong(1);
                    }
                }
            }
            
            return null;
            
        } catch (SQLException e) {
            throw new RuntimeException("插入用户失败", e);
        }
    }
    
    /**
     * 根据 ID 查询用户
     * 
     * @param id 用户ID
     * @return 用户信息
     */
    public UserInfo selectUserById(Long id) {
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info WHERE id = ?";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
            
            preparedStatement.setLong(1, id);
            
            try (ResultSet resultSet = preparedStatement.executeQuery()) {
                if (resultSet.next()) {
                    return buildUserInfo(resultSet);
                }
            }
            
            return null;
            
        } catch (SQLException e) {
            throw new RuntimeException("查询用户失败", e);
        }
    }
    
    /**
     * 查询所有用户
     * 
     * @return 用户列表
     */
    public List<UserInfo> selectAllUsers() {
        String sql = "SELECT id, user_name, age, email, status, create_time, update_time " +
                     "FROM user_info ORDER BY id";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql);
             ResultSet resultSet = preparedStatement.executeQuery()) {
            
            List<UserInfo> userList = new ArrayList<>(100);
            
            while (resultSet.next()) {
                userList.add(buildUserInfo(resultSet));
            }
            
            return userList;
            
        } catch (SQLException e) {
            throw new RuntimeException("查询用户列表失败", e);
        }
    }
    
    /**
     * 更新用户
     * 
     * @param userInfo 用户信息
     * @return 更新成功的记录数
     */
    public int updateUser(UserInfo userInfo) {
        String sql = "UPDATE user_info SET user_name = ?, age = ?, email = ? WHERE id = ?";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
            
            preparedStatement.setString(1, userInfo.getUserName());
            preparedStatement.setInt(2, userInfo.getAge());
            preparedStatement.setString(3, userInfo.getEmail());
            preparedStatement.setLong(4, userInfo.getId());
            
            return preparedStatement.executeUpdate();
            
        } catch (SQLException e) {
            throw new RuntimeException("更新用户失败", e);
        }
    }
    
    /**
     * 删除用户
     * 
     * @param id 用户ID
     * @return 删除成功的记录数
     */
    public int deleteUser(Long id) {
        String sql = "DELETE FROM user_info WHERE id = ?";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
            
            preparedStatement.setLong(1, id);
            
            return preparedStatement.executeUpdate();
            
        } catch (SQLException e) {
            throw new RuntimeException("删除用户失败", e);
        }
    }
    
    /**
     * 构建用户对象
     * 
     * @param resultSet 结果集
     * @return 用户信息
     * @throws SQLException SQL异常
     */
    private UserInfo buildUserInfo(ResultSet resultSet) throws SQLException {
        UserInfo userInfo = new UserInfo();
        userInfo.setId(resultSet.getLong("id"));
        userInfo.setUserName(resultSet.getString("user_name"));
        userInfo.setAge(resultSet.getInt("age"));
        userInfo.setEmail(resultSet.getString("email"));
        userInfo.setStatus(resultSet.getInt("status"));
        
        if (resultSet.getTimestamp("create_time") != null) {
            userInfo.setCreateTime(resultSet.getTimestamp("create_time").toLocalDateTime());
        }
        if (resultSet.getTimestamp("update_time") != null) {
            userInfo.setUpdateTime(resultSet.getTimestamp("update_time").toLocalDateTime());
        }
        
        return userInfo;
    }
}
```

---

## 8. 常见问题

### 8.1 时区问题

```java
/**
 * 时区问题解决方案
 * 
 * @author erik.zhou
 */
public class TimeZoneIssue {
    
    /**
     * 问题：MySQL 8.0+ 连接时报时区错误
     * 
     * 错误信息：The server time zone value 'CST' is unrecognized
     * 
     * 解决方案：在 JDBC URL 中添加 serverTimezone 参数
     */
    private static final String CORRECT_URL = 
        "jdbc:mysql://localhost:3306/test_db?serverTimezone=Asia/Shanghai";
}
```

### 8.2 中文乱码问题

```java
/**
 * 中文乱码问题解决方案
 * 
 * @author erik.zhou
 */
public class ChineseEncodingIssue {
    
    /**
     * 问题：插入或查询中文数据时出现乱码
     * 
     * 解决方案：
     * 1. 数据库和表使用 utf8mb4 字符集
     * 2. JDBC URL 中添加字符编码参数
     */
    private static final String CORRECT_URL = 
        "jdbc:mysql://localhost:3306/test_db?useUnicode=true&characterEncoding=utf8";
    
    /**
     * 建表语句示例
     */
    private static final String CREATE_TABLE_SQL = 
        "CREATE TABLE user_info (" +
        "  id BIGINT AUTO_INCREMENT PRIMARY KEY," +
        "  user_name VARCHAR(64) NOT NULL" +
        ") ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表'";
}
```

### 8.3 连接泄漏问题

```java
/**
 * 连接泄漏问题解决方案
 * 
 * @author erik.zhou
 */
public class ConnectionLeakIssue {
    
    /**
     * 错误示例：未关闭连接导致连接泄漏
     */
    @Deprecated
    public void badExample() {
        try {
            Connection connection = HikariCpUtil.getConnection();
            // 执行数据库操作
            // 忘记关闭连接，导致连接泄漏
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 正确示例：使用 try-with-resources 自动关闭连接
     */
    public void goodExample() {
        try (Connection connection = HikariCpUtil.getConnection()) {
            // 执行数据库操作
            // try-with-resources 会自动关闭连接
        } catch (SQLException e) {
            throw new RuntimeException("数据库操作失败", e);
        }
    }
}
```

### 8.4 性能优化建议

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.List;

/**
 * 性能优化建议
 * 
 * @author erik.zhou
 */
public class PerformanceOptimization {
    
    /**
     * 1. 使用批量操作代替循环单条操作
     */
    public int batchInsert(List<UserInfo> userList) {
        String sql = "INSERT INTO user_info(user_name, age, email) VALUES (?, ?, ?)";
        
        try (Connection connection = HikariCpUtil.getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
            
            for (UserInfo userInfo : userList) {
                preparedStatement.setString(1, userInfo.getUserName());
                preparedStatement.setInt(2, userInfo.getAge());
                preparedStatement.setString(3, userInfo.getEmail());
                preparedStatement.addBatch();
            }
            
            int[] results = preparedStatement.executeBatch();
            
            int successCount = 0;
            for (int result : results) {
                if (result > 0) {
                    successCount++;
                }
            }
            
            return successCount;
            
        } catch (SQLException e) {
            throw new RuntimeException("批量插入失败", e);
        }
    }
    
    /**
     * 2. 使用连接池代替每次创建连接
     * 3. 使用 PreparedStatement 代替 Statement（预编译，防止 SQL 注入）
     * 4. 只查询需要的字段，避免 SELECT *
     * 5. 合理使用索引，避免全表扫描
     * 6. 分页查询大数据量时使用 LIMIT
     */
}
```

---

## 9. 测试示例

```java
/**
 * 测试类
 * 
 * @author erik.zhou
 */
public class JdbcTest {
    
    public static void main(String[] args) {
        UserCompleteDao userDao = new UserCompleteDao();
        
        // 1. 插入用户
        UserInfo newUser = new UserInfo("张三", 25, "zhangsan@example.com");
        Long userId = userDao.insertUser(newUser);
        System.out.println("插入用户成功，ID: " + userId);
        
        // 2. 查询用户
        UserInfo user = userDao.selectUserById(userId);
        System.out.println("查询用户: " + user);
        
        // 3. 更新用户
        user.setAge(26);
        int updateCount = userDao.updateUser(user);
        System.out.println("更新用户成功，影响行数: " + updateCount);
        
        // 4. 查询所有用户
        List<UserInfo> userList = userDao.selectAllUsers();
        System.out.println("用户总数: " + userList.size());
        
        // 5. 删除用户
        int deleteCount = userDao.deleteUser(userId);
        System.out.println("删除用户成功，影响行数: " + deleteCount);
        
        // 6. 关闭连接池
        HikariCpUtil.close();
    }
}
```

---

## 10. 总结

本教程涵盖了 Java 直接操作 MySQL 数据库的完整流程：

1. 环境准备：Maven 依赖、数据库建表
2. JDBC 基础：核心接口、工作流程
3. 数据库连接：基础连接、配置文件方式
4. CRUD 操作：增删改查的完整实现
5. 事务管理：事务提交与回滚
6. 连接池：HikariCP 配置与使用
7. 最佳实践：try-with-resources、SQL 注入防护
8. 常见问题：时区、乱码、连接泄漏等问题解决

遵循阿里巴巴开发规范，所有代码示例都符合企业级开发标准。
