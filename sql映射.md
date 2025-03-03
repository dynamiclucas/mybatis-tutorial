# **1. 数据库管理**

### **创建数据库**

```
CREATE DATABASE my_database;
```

**MyBatis XML 对应写法（不常用，通常直接执行 SQL）：**

```
<update>CREATE DATABASE my_database;</update>
```

------

# **2. 表管理**

### **创建表**

```
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INT DEFAULT 18,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**MyBatis XML（通常不建议在 XML 中创建表）：**

```
<update>
    CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        email VARCHAR(100) UNIQUE NOT NULL,
        age INT DEFAULT 18,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
</update>
```

------

### **删除表**

```
DROP TABLE users;
```

**MyBatis XML**

```
<update>DROP TABLE users;</update>
```

------

### **修改表**

```
ALTER TABLE users ADD COLUMN phone VARCHAR(15);
```

**MyBatis XML**

```
<update>ALTER TABLE users ADD COLUMN phone VARCHAR(15);</update>
```

------

# **3. 数据操作（增删改查）**

### **插入数据**

```
INSERT INTO users (name, email, age) VALUES ('Alice', 'alice@example.com', 22);
```

**MyBatis XML**

```
<insert id="insertUser" parameterType="com.example.model.User">
    INSERT INTO users (name, email, age) VALUES (#{name}, #{email}, #{age});
</insert>
```

------

### **查询所有数据**

```
SELECT * FROM users;
```

**MyBatis XML**

```
<select id="getAllUsers" resultType="com.example.model.User">
    SELECT * FROM users;
</select>
```

------

### **查询单个用户**

```
SELECT * FROM users WHERE id = 1;
```

**MyBatis XML**

```
<select id="getUserById" resultType="com.example.model.User" parameterType="int">
    SELECT * FROM users WHERE id = #{id};
</select>
```

------

### **更新用户**

```
UPDATE users SET email = 'newalice@example.com' WHERE id = 1;
```

**MyBatis XML**

```
<update id="updateUser" parameterType="com.example.model.User">
    UPDATE users SET name = #{name}, email = #{email} WHERE id = #{id};
</update>
```

------

### **删除用户**

```
DELETE FROM users WHERE id = 1;
```

**MyBatis XML**

```
<delete id="deleteUser" parameterType="int">
    DELETE FROM users WHERE id = #{id};
</delete>
```

------

# **4. 高级查询**

### **查询去重数据**

```
SELECT DISTINCT age FROM users;
```

**MyBatis XML**

```
<select id="getDistinctAges" resultType="int">
    SELECT DISTINCT age FROM users;
</select>
```

------

### **范围查询**

```
SELECT * FROM users WHERE age BETWEEN 20 AND 30;
```

**MyBatis XML**

```
<select id="getUsersByAgeRange" resultType="com.example.model.User">
    SELECT * FROM users WHERE age BETWEEN #{minAge} AND #{maxAge};
</select>
```

------

### **模糊匹配**

```
SELECT * FROM users WHERE name LIKE 'A%';
```

**MyBatis XML**

```
<select id="getUsersByNamePattern" resultType="com.example.model.User">
    SELECT * FROM users WHERE name LIKE #{namePattern};
</select>
```

**调用时传入参数：** `namePattern = "A%"`

------

### **排序**

```
SELECT * FROM users ORDER BY age DESC;
```

**MyBatis XML**

```
<select id="getUsersSortedByAge" resultType="com.example.model.User">
    SELECT * FROM users ORDER BY age DESC;
</select>
```

------

### **分页查询**

```
SELECT * FROM users LIMIT 5 OFFSET 10;
```

**MyBatis XML**

```
<select id="getUsersWithPagination" resultType="com.example.model.User">
    SELECT * FROM users LIMIT #{limit} OFFSET #{offset};
</select>
```

------

# **5. 关系查询**

### **表连接（JOIN）**

```
SELECT users.name, orders.amount FROM users 
INNER JOIN orders ON users.id = orders.user_id;
```

**MyBatis XML**

```
<select id="getUserOrders" resultType="map">
    SELECT users.name, orders.amount 
    FROM users 
    INNER JOIN orders ON users.id = orders.user_id;
</select>
```

------

# **6. 视图（VIEW）**

### **创建视图**

```
CREATE VIEW user_info AS 
SELECT name, email FROM users;
```

**MyBatis XML**

```
<update>
    CREATE VIEW user_info AS 
    SELECT name, email FROM users;
</update>
```

------

# **7. 索引（INDEX）**

### **创建索引**

```
CREATE INDEX idx_users_name ON users(name);
```

**MyBatis XML**

```
<update>CREATE INDEX idx_users_name ON users(name);</update>
```

------

# **8. 存储过程**

### **创建存储过程**

```
DELIMITER //
CREATE PROCEDURE getUserById(IN userId INT)
BEGIN
    SELECT * FROM users WHERE id = userId;
END //
DELIMITER ;
```

**MyBatis XML**

```
<select id="callStoredProcedure" resultType="com.example.model.User">
    CALL getUserById(#{id});
</select>
```

------

# **9. 触发器（Trigger）**

### **创建触发器**

```
DELIMITER //
CREATE TRIGGER before_user_insert
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
END //
DELIMITER ;
```

**MyBatis XML**

```
<update>
    CREATE TRIGGER before_user_insert
    BEFORE INSERT ON users
    FOR EACH ROW
    BEGIN
        SET NEW.created_at = NOW();
    END;
</update>
```

------

# **10. 事务（Transaction）**

### **事务控制**

```
START TRANSACTION;
UPDATE users SET age = age + 1 WHERE id = 1;
DELETE FROM users WHERE id = 2;
COMMIT;
```

**MyBatis XML**

```
<update>
    START TRANSACTION;
    UPDATE users SET age = age + 1 WHERE id = 1;
    DELETE FROM users WHERE id = 2;
    COMMIT;
</update>
```

**回滚**

```
ROLLBACK;
```

**MyBatis XML**

```
<update>ROLLBACK;</update>
```

# 