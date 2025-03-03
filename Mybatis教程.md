# Mybatis环境配置教程

### 在 **MyBatis 框架** 中，我们可以通过 **XML 配置文件 (`Mapper.xml`)** 来实现 **增、删、改、查（CRUD）** 操作。以下是 **完整的示例**，包括 **数据库表结构、MyBatis 配置、Mapper.xml 配置、Java 接口以及测试代码**。



## **整体思路：**

1. **数据库**：创建 `users` 表。

2. **MyBatis 配置**：`mybatis-config.xml` 连接数据库，引用 `UserMapper.xml`。

3. **Mapper XML**：`UserMapper.xml` 定义 `增删改查` 操作。

4. Java 代码

   ：

   - `User.java` 定义用户类。
   - `UserMapper.java` 接口映射 XML 方法。
   - `UserService.java` 处理业务逻辑。
   - `MyBatisUtil.java` 统一管理数据库连接。
   - `Main.java` 进行测试。

## 0.项目文件结构

```
MyBatisExample/
│── src/
│   ├── main/
│   │   ├── java/
│   │   │   ├── com/example/
│   │   │   │   ├── model/         # 实体类（User.java）
│   │   │   │   ├── mapper/        # MyBatis 接口（UserMapper.java）
│   │   │   │   ├── service/       # 业务逻辑（UserService.java）
│   │   │   │   ├── util/          # MyBatis 工具类（MyBatisUtil.java）
│   │   │   │   ├── Main.java      # 测试主类
│   │   ├── resources/
│   │   │   ├── mybatis-config.xml # MyBatis 核心配置文件
│   │   │   ├── com/example/mapper/ # MyBatis 映射 XML
│   │   │   │   ├── UserMapper.xml
│   ├── test/                      # 测试代码（JUnit）
│── pom.xml                         # Maven 依赖管理
│── README.md                       # 项目说明

```



## **1. 数据库表结构**

我们假设有一个 `users` 表：

```
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);

```

## **2. MyBatis 核心配置 (`mybatis-config.xml`)**

在 `resources/mybatis-config.xml` 中，配置数据库连接和 Mapper：

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_db?useSSL=false&serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="???输入密码??"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/example/mapper/UserMapper.xml"/>
    </mappers>
</configuration>

```

## **3. MyBatis Mapper 配置 (`UserMapper.xml`)**

在 `resources/com/example/mapper/UserMapper.xml` 中，编写 CRUD 操作的 SQL 语句：

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">

    <!-- 查询所有用户 -->
    <select id="getAllUsers" resultType="com.example.model.User">
        SELECT * FROM users;
    </select>

    <!-- 根据 ID 查询用户 -->
    <select id="getUserById" resultType="com.example.model.User" parameterType="int">
        SELECT * FROM users WHERE id = #{id};
    </select>

    <!-- 插入用户 -->
    <insert id="insertUser" parameterType="com.example.model.User">
        INSERT INTO users (name, email) VALUES (#{name}, #{email});
    </insert>

    <!-- 更新用户 -->
    <update id="updateUser" parameterType="com.example.model.User">
        UPDATE users SET name = #{name}, email = #{email} WHERE id = #{id};
    </update>

    <!-- 删除用户 -->
    <delete id="deleteUser" parameterType="int">
        DELETE FROM users WHERE id = #{id};
    </delete>

</mapper>

```

------

## **4. Java 实体类 (`User.java`)**

在 `com/example/model/User.java` 中，定义 `User` 对象：

```
package com.example.model;

public class User {
    private int id;
    private String name;
    private String email;

    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

```

------

## **5. Java 接口 (`UserMapper.java`)**

在 `com/example/mapper/UserMapper.java` 中，定义 Mapper 接口：

```
package com.example.mapper;

import com.example.model.User;
import java.util.List;

public interface UserMapper {
    List<User> getAllUsers();
    User getUserById(int id);
    void insertUser(User user);
    void updateUser(User user);
    void deleteUser(int id);
}

```

------

## **6. MyBatis 工具类 (`MyBatisUtil.java`)**

在 `com/example/util/MyBatisUtil.java` 中，封装 MyBatis 会话：

```
package com.example.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.IOException;
import java.io.Reader;

public class MyBatisUtil {
    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        } catch (IOException e) {
            throw new RuntimeException("Error initializing MyBatis: " + e.getMessage(), e);
        }
    }

    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}

```

------

## **7. 业务逻辑 (`UserService.java`)**

在 `com/example/service/UserService.java` 中，封装数据库操作：

```
package com.example.service;

import com.example.mapper.UserMapper;
import com.example.model.User;
import com.example.util.MyBatisUtil;
import org.apache.ibatis.session.SqlSession;
import java.util.List;

public class UserService {

    // 查询所有用户
    public List<User> getAllUsers() {
        try (SqlSession session = MyBatisUtil.getSqlSession()) {
            return session.getMapper(UserMapper.class).getAllUsers();
        }
    }

    // 查询单个用户
    public User getUserById(int id) {
        try (SqlSession session = MyBatisUtil.getSqlSession()) {
            return session.getMapper(UserMapper.class).getUserById(id);
        }
    }

    // 插入用户
    public void addUser(User user) {
        try (SqlSession session = MyBatisUtil.getSqlSession()) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            mapper.insertUser(user);
            session.commit();
        }
    }

    // 更新用户
    public void updateUser(User user) {
        try (SqlSession session = MyBatisUtil.getSqlSession()) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            mapper.updateUser(user);
            session.commit();
        }
    }

    // 删除用户
    public void deleteUser(int id) {
        try (SqlSession session = MyBatisUtil.getSqlSession()) {
            UserMapper mapper = session.getMapper(UserMapper.class);
            mapper.deleteUser(id);
            session.commit();
        }
    }
}

```

------

## **8. 测试 (`Main.java`)**

```
package com.example;

import com.example.model.User;
import com.example.service.UserService;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        UserService userService = new UserService();

        // 插入新用户
        User newUser = new User();
        newUser.setName("John Doe");
        newUser.setEmail("john.doe@example.com");
        userService.addUser(newUser);
        System.out.println("新用户已插入");

        // 查询所有用户
        System.out.println("查询所有用户:");
        List<User> users = userService.getAllUsers();
        for (User user : users) {
            System.out.println(user.getId() + " - " + user.getName() + " - " + user.getEmail());
        }

        // 更新用户
        System.out.println("更新用户:");
        User updateUser = new User();
        updateUser.setId(1);
        updateUser.setName("Updated Name");
        updateUser.setEmail("updated.email@example.com");
        userService.updateUser(updateUser);
        System.out.println("用户信息已更新");

        // 删除用户
        System.out.println("删除用户:");
        userService.deleteUser(1);
        System.out.println("用户已删除");
    }
}

```

## **各部分详细说明**

### **1. Java 代码 (`src/main/java/`)**

| **目录**                | **说明**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| `com.example.model/`    | 存放 **实体类**（如 `User.java`），用于映射数据库表。        |
| `com.example.mapper/`   | 存放 **MyBatis 的 Mapper 接口**（如 `UserMapper.java`），与 XML 绑定。 |
| `com.example.service/`  | 存放 **业务逻辑代码**（如 `UserService.java`），调用 Mapper 操作数据库。 |
| `com.example.util/`     | 存放 **工具类**（如 `MyBatisUtil.java`），管理数据库连接。   |
| `com.example.Main.java` | **测试 MyBatis 功能的主类**。                                |

------

### **2. 资源文件 (`src/main/resources/`)**

| **目录**                            | **说明**                                                    |
| ----------------------------------- | ----------------------------------------------------------- |
| `mybatis-config.xml`                | **MyBatis 的全局配置文件**，配置数据库连接、Mapper 注册等。 |
| `com/example/mapper/UserMapper.xml` | **MyBatis SQL 映射文件**，定义 `UserMapper` 的 SQL 语句。   |

------

### **3. 测试代码 (`src/test/java/`)**

- 存放单元测试代码，例如 JUnit 测试 `UserService`。

------

### **4. `pom.xml`（Maven 配置）**

**用于管理 MyBatis 依赖**：

```
<dependencies>
    <!-- MyBatis 依赖 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>

    <!-- MyBatis-MySQL 连接驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.28</version>
    </dependency>

    <!-- 日志支持 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.32</version>
    </dependency>

    <!-- 单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>

```

------

## **总结**

- `java/` 目录下是 **实体类、Mapper 接口、业务逻辑、工具类** 等。
- `resources/` 目录存放 **MyBatis 配置文件和 SQL 映射文件**。
- `pom.xml` 负责 **管理 MyBatis 和数据库的依赖**。