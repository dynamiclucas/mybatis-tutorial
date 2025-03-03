# **MyBatis 动态 SQL 详解**

------

## **🔹 1. 动态 SQL 介绍**

在 MyBatis 框架中，**动态 SQL** 允许我们在 XML 映射文件 (`Mapper.xml`) 中 **动态拼接 SQL 语句**。这使得 SQL 语句更灵活，能够根据不同的条件动态生成查询语句，从而避免手写大量 if-else 逻辑。

在实际开发中，动态 SQL 主要用于：

1. **条件查询**（根据传入的参数不同，动态拼接 SQL）
2. **更新操作**（仅更新非空字段）
3. **复杂查询**（多条件组合查询）
4. **SQL 片段复用**（提高 SQL 代码的可读性和复用性）

MyBatis 提供了多个 **动态 SQL 元素**，用于 **在 XML 配置文件中拼接 SQL 语句**。

------

## **🔹 2. 动态 SQL 的常用元素**

在 MyBatis 中，常见的 **动态 SQL 元素** 包括：

| **元素**      | **作用**                                  |
| ------------- | ----------------------------------------- |
| `<if>`        | 根据条件动态拼接 SQL 语句                 |
| `<choose>`    | 相当于 Java 的 `switch` 语句              |
| `<when>`      | `<choose>` 结构中的条件                   |
| `<otherwise>` | `<choose>` 结构中的默认情况               |
| `<where>`     | 解决 `WHERE` 关键字自动拼接的问题         |
| `<trim>`      | 定制 SQL 语句的前缀或后缀                 |
| `<set>`       | 主要用于 `UPDATE` 语句，防止 SQL 语法错误 |
| `<foreach>`   | 处理 `IN` 语句、批量插入等                |

------

# **🔹 3. 条件查询操作**

### **✅ 3.1 使用 `<if>` 进行条件查询**

`<if>` 标签用于 **根据条件动态拼接 SQL 语句**。

#### **示例：根据 `name` 和 `email` 查询用户**

```
<select id="getUsersByCondition" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE 1=1
    <if test="name != null">
        AND name = #{name}
    </if>
    <if test="email != null">
        AND email = #{email}
    </if>
</select>
```

**解释：**

- `WHERE 1=1`：确保 SQL 语法正确，即使后续没有拼接 `AND` 条件，也不会报错。
- `if test="name != null"`：如果 `name` 不为空，则拼接 `AND name = #{name}`。

**对应的 Java 代码调用：**

```
List<User> users = userMapper.getUsersByCondition("Alice", null);
```

执行的 SQL：

```
SELECT * FROM users WHERE 1=1 AND name = 'Alice';
```

------

### **✅ 3.2 使用 `<choose>、<when>、<otherwise>` 进行条件查询**

`<choose>` 类似于 Java 的 `switch` 语句，它 **只会匹配一个条件**，如果都不符合，则执行 `<otherwise>` 语句。

#### **示例：如果 `name` 存在，则查询 `name`，否则查询 `email`**

```
<select id="getUserByChoose" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE 1=1
    <choose>
        <when test="name != null">
            AND name = #{name}
        </when>
        <when test="email != null">
            AND email = #{email}
        </when>
        <otherwise>
            AND age > 18
        </otherwise>
    </choose>
</select>
```

**示例调用（传入 `name = 'Alice'`）：**

```
SELECT * FROM users WHERE 1=1 AND name = 'Alice';
```

------

### **✅ 3.3 使用 `<where>` 处理 `WHERE` 关键字**

- **问题**：如果查询条件都为空，SQL 可能会变成 `WHERE` 关键字后面没有任何条件，导致 **SQL 语法错误**。
- **解决方案**：使用 `<where>`，它会 **自动处理 `AND` 关键字**。

#### **示例：根据 `name` 和 `email` 过滤查询**

```
<select id="getUsersWithWhere" resultType="com.example.model.User">
    SELECT * FROM users
    <where>
        <if test="name != null">
            name = #{name}
        </if>
        <if test="email != null">
            AND email = #{email}
        </if>
    </where>
</select>
```

如果 `name` 和 `email` 都为空，则 SQL 变成：

```
SELECT * FROM users;
```

**MyBatis 自动去掉 `WHERE` 关键字，避免 SQL 语法错误。**

------

### **✅ 3.4 使用 `<trim>` 自定义 SQL 语句**

`<trim>` 可以自定义 **前缀、后缀、去掉特定字符**，类似 `<where>`，但更灵活。

#### **示例：使用 `<trim>` 代替 `<where>`**

```
<select id="getUsersWithTrim" resultType="com.example.model.User">
    SELECT * FROM users
    <trim prefix="WHERE" prefixOverrides="AND |OR">
        <if test="name != null">
            AND name = #{name}
        </if>
        <if test="email != null">
            OR email = #{email}
        </if>
    </trim>
</select>
```

**自动去掉第一个 `AND` 或 `OR`，保证 SQL 语法正确！**

------

# **🔹 4. 动态更新操作**

当我们更新用户信息时，如果某些字段未提供，我们希望 **只更新提供的字段**。

### **✅ 4.1 使用 `<set>` 进行动态更新**

```
<update id="updateUser" parameterType="com.example.model.User">
    UPDATE users
    <set>
        <if test="name != null">
            name = #{name},
        </if>
        <if test="email != null">
            email = #{email},
        </if>
        <if test="age != null">
            age = #{age},
        </if>
    </set>
    WHERE id = #{id}
</update>
```

**MyBatis 自动处理最后的逗号**，避免 SQL 语法错误！

------

# **🔹 5. 复杂查询操作**

### **✅ 5.1 使用 `<foreach>` 实现 `IN` 语句**

```
<select id="getUsersByIds" resultType="com.example.model.User">
    SELECT * FROM users
    WHERE id IN
    <foreach collection="idList" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

**示例调用：**

```
List<User> users = userMapper.getUsersByIds(Arrays.asList(1, 2, 3));
```

执行的 SQL：

```
SELECT * FROM users WHERE id IN (1, 2, 3);
```

------

# **🔹 6. 总结**

| **动态 SQL 元素** | **作用**                                   |
| ----------------- | ------------------------------------------ |
| `<if>`            | 根据条件拼接 SQL                           |
| `<choose>`        | 类似 Java `switch` 结构                    |
| `<where>`         | 自动添加 `WHERE` 关键字，避免 SQL 语法错误 |
| `<trim>`          | 定制 SQL 语句的前缀或后缀                  |
| `<set>`           | 主要用于 `UPDATE` 语句，防止逗号错误       |
| `<foreach>`       | 处理 `IN` 语句，批量操作                   |