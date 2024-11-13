#java代码简易知识
#一基础方法
###1.如何在代码里面使用POST或GET请求
```
Map<String, String> accessParams = new HashMap<>();
accessParams.put("appId", appId);
accessParams.put("timestamp", timestamp);
accessParams.put("sign", sign);
accessParams.put("accessToken", accessToken);
 Map<String, Object> map =restTemplate.exchange(
 serverUrl + "/auth/user/info", // URL endpoint
 HttpMethod.POST, // HTTP method
 new HttpEntity<>(accessParams, createHeaders()),
 Map.class // Expected response type
);
```
这个方法是通过restTemplate.exchange()方法来发送POST请求，并且将请求参数封装成一个HttpEntity对象，最后通过createHeaders()方法来设置请求头。
##2.如何把信息放在header中
```
ServerHttpRequest.Builder mutate2 = request.mutate();
mutate.header("accessToken", accessToken);
```
这里面放到是request.mutate()，然后通过header()方法来设置。
```
HttpHeaders headers = request.getHeaders();
String accessToken = headers.getFirst("accessToken");
```
这里面是获取header里面的信息，通过getFirst()方法来获取。
##3MyBatis Mapper XML 文件写法
基本的 CRUD 操作
```
   插入数据（INSERT）
   <insert id="insertUser" parameterType="com.example.User">
   INSERT INTO users (name, age)
   VALUES (#{name}, #{age})
   </insert>
   parameterType="com.example.User" 表示该插入操作需要传入一个 User 对象，SQL 中的字段值会对应到 User 类的属性（如 name 和 age）。
```
{} 是占位符，用于替换 User 对象的属性。

更新数据（UPDATE）
```
<update id="updateUser" parameterType="com.example.User">
UPDATE users
SET name = #{name}, age = #{age}
WHERE id = #{id}
</update>
```
更新操作的 SQL 根据 id 查找到特定的用户，然后更新该用户的 name 和 age 字段。

删除数据（DELETE）
```
<delete id="deleteUser" parameterType="int">
DELETE FROM users WHERE id = #{id}
</delete>
```
删除操作根据用户的 id 来删除对应的记录。

##复杂查询
1. 多表查询（JOIN）
   在 MyBatis 中，可以通过执行 JOIN 查询来获取来自多个表的数据。假设你有两个表：users 和 orders，用户表存储用户信息，订单表存储订单信息。
```
<select id="getUserWithOrders" parameterType="int" resultMap="userWithOrders">
SELECT u.id, u.name, u.age, o.id as order_id, o.product_name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.id = #{id}
</select>
<resultMap id="userWithOrders" type="com.example.User">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <result property="age" column="age"/>
    <collection property="orders" ofType="com.example.Order">
        <id property="id" column="order_id"/>
        <result property="productName" column="product_name"/>
    </collection>
</resultMap>
```
LEFT JOIN 查询将用户表和订单表连接起来，返回用户信息和对应的订单信息。
通过 resultMap 将结果映射到 User 和 Order 两个类。<collection> 标签表示一个用户可以拥有多个订单（多对一关系）。

2.动态 SQL
在 MyBatis 中，动态 SQL 可以根据不同的条件生成不同的 SQL 查询。比如，如果你想查询用户列表，按 name 或 age 过滤，你可以使用 <if>, <choose>, <where> 等动态标签。
```
<select id="getUsersByCondition" resultType="com.example.User">
SELECT id, name, age
FROM users
<where>
<if test="name != null">
AND name = #{name}
</if>
<if test="age != null">
AND age = #{age}
</if>
</where>
</select>
<where> 
```
<where> 标签会自动添加 WHERE 子句并处理多个条件。
<if test="..."> 标签表示如果条件满足（如 name != null），才会将对应的 SQL 片段添加到查询中。

3. 分页查询
   分页查询通常会用到 LIMIT 或 ROWNUM（根据数据库类型不同而不同），MyBatis 可以方便地与分页插件一起工作，但这里我们先看一个基本的分页查询示例。
```
<select id="getUsersWithPagination" resultType="com.example.User">
SELECT id, name, age
FROM users
LIMIT #{offset}, #{pageSize}
</select>
```
offset 表示从第几行开始，pageSize 是每页显示的记录数。
这个查询会返回一部分用户数据，适用于简单的分页功能。

4. 模糊查询
   如果你需要根据用户的 name 模糊匹配查询用户，可以使用 LIKE 语句。
```
<select id="getUsersByName" resultType="com.example.User">
SELECT id, name, age
FROM users
WHERE name LIKE CONCAT('%', #{name}, '%')
</select>
```
LIKE 用于模糊匹配。
CONCAT('%', #{name}, '%') 将用户输入的名字与 % 连接，表示前后匹配。

5. 复杂的条件查询
   假设你需要根据多个条件查询用户列表，条件可能有 name, age, 和 status 等，可以使用 choose 和 when 标签来进行条件判断。
```
<select id="getUsersByComplexCondition" resultType="com.example.User">
SELECT id, name, age, status
FROM users
<where>
<choose>
<when test="name != null">
AND name = #{name}
</when>
<when test="age != null">
AND age = #{age}
</when>
<otherwise>
AND status = 'active'
</otherwise>
</choose>
</where>
</select>
```
&lt;choose&gt; </choose>标签类似于 Java 的 switch 语句，根据不同的条件执行不同的查询逻辑。
标签<otherwise>&lt;otherwise&gt;可以用来指定当所有条件都不满足时的默认条件。


##3. 使用 ResultMap 进行复杂映射
   ResultMap 用于处理复杂的结果映射，特别是当查询结果涉及多个表时。你可以使用 resultMap 映射复杂的查询结果。
```
<resultMap id="userResultMap" type="com.example.User">
<id property="id" column="id"/>
<result property="name" column="name"/>
<result property="age" column="age"/>
<collection property="orders" ofType="com.example.Order">
<id property="orderId" column="order_id"/>
<result property="productName" column="product_name"/>
</collection>
</resultMap>

<select id="getUserWithOrders" resultMap="userResultMap">
    SELECT u.id, u.name, u.age, o.id AS order_id, o.product_name
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.id = #{id}
</select>
```

