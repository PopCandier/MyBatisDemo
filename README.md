### Mybatis 学习笔记



#### Mybatis的使用

引入依赖，和mysql的依赖

```xml
 <dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis</artifactId>
     <version>3.5.1</version>
 </dependency>

  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.21</version>
  </dependency>
```

最简单的api使用（Mybatis前身-ibatis用法，并不存在mapper对象）

```java
public void testStatement() throws IOException {
    String resource = "mybatis-config.xml";//全局配置文件
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new 		    SqlSessionFactoryBuilder().build(inputStream);//解析配置文件，创建会话工厂
	//创建会话
    SqlSession session = sqlSessionFactory.openSession();
    try {
        //会话指定具体类下的方法和方法，与mybatis-config.xml中的mapper进行sql绑定
        //从而调用sql语句
        Blog blog = (Blog) session.selectOne("com.gupaoedu.mapper.BlogMapper.selectBlogById", 1);
        System.out.println(blog);
    } finally {
        session.close();
    }
}
```

现在比较常用，SqlSession.getMapper(XXXMapper.class) 接口方式

```java
 public void testSelect() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = sqlSessionFactory.openSession(); // ExecutorType.BATCH
        try {
            //接口方式，直接调用接口
            BlogMapper mapper = session.getMapper(BlogMapper.class);
            Blog blog = mapper.selectBlogById(1);
            System.out.println(blog);
        } finally {
            session.close();
        }
    }
```

##### Mybatis的核心对象

* SqlSessionFactoryBuilder

  * 这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在，以保证所有的 XML 解析资源可以被释放给更重要的事情。

* SqlSessionFactory

  * SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

* SqlSession

  * 每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例**不是线程安全**的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。 换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它。 这个关闭操作是很重要的，你应该把这个关闭操作放到 finally 块中以确保每次都能执行关闭。 下面的示例就是一个确保 SqlSession 关闭的标准模式：

    ```java
    try (SqlSession session = sqlSessionFactory.openSession()) {
      // 你的应用逻辑代码
    }
    ```

    

* Mapper

  * 映射器是一些由你创建的、绑定你映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的。因此从技术层面讲，任何映射器实例的最大作用域是和请求它们的 SqlSession 相同的。尽管如此，映射器实例的最佳作用域是方法作用域。 也就是说，映射器实例应该在调用它们的方法中被请求，用过之后即可丢弃。 并不需要显式地关闭映射器实例，尽管在整个请求作用域保持映射器实例也不会有什么问题，但是你很快会发现，像 SqlSession 一样，在这个作用域上管理太多的资源的话会难于控制。 为了避免这种复杂性，最好把映射器放在方法作用域内。下面的示例就展示了这个实践：

    ```java
    try (SqlSession session = sqlSessionFactory.openSession()) {
      BlogMapper mapper = session.getMapper(BlogMapper.class);
      // 你的应用逻辑代码
    }
    ```


| 对象                                                      | 生命周期                     |
| --------------------------------------------------------- | ---------------------------- |
| SqlSessionFactoryBuilder(创建出SqlSessionFacotry就没用了) | 方法局部（method）           |
| SqlSessionFactroy(单例，只用来生成SqlSession)             | 应用级别（application）      |
| SqlSession                                                | 请求和操作（request/method） |
| Mapper                                                    | 方法（method）               |

##### Mybatis的配置对象

`XMLConfigBuilder`用于解析`mybatis-config.xml`文件

```xml
<configuration>
<!-- 最外面的配置-->
</configuration>
```

###### Properties

```xml
<properties resource="db.properties">
	<!--用于配置一些外部的属性-->
</properties>
```

###### Settings

```xml
<settings>
        <!-- 打印查询语句 -->
        <setting name="logImpl" value="STDOUT_LOGGING" />

        <!-- 控制全局缓存（二级缓存）-->
        <setting name="cacheEnabled" value="true"/>

        <!-- 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。默认 false  -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!-- 当开启时，任何方法的调用都会加载该对象的所有属性。默认 false，可通过select标签的 fetchType来覆盖-->
        <setting name="aggressiveLazyLoading" value="false"/>
        <!--  Mybatis 创建具有延迟加载能力的对象所用到的代理工具，默认JAVASSIST -->
        <!--<setting name="proxyFactory" value="CGLIB" />-->
        <!-- STATEMENT级别的缓存，使一级缓存，只针对当前执行的这一statement有效 -->
        <!--
                <setting name="localCacheScope" value="STATEMENT"/>
        -->
        <setting name="localCacheScope" value="SESSION"/>
</settings>
```

配置方言，当你的mapper文件的resultType就可以直接配置缩写

###### TypeAliases

```xml
<typeAliases>
        <typeAlias alias="blog" type="com.pop.domain.Blog" />
</typeAliases>
```

`TypeAliasRegistry`会统一注册在这里。

```java
public class TypeAliasRegistry {

  private final Map<String, Class<?>> typeAliases = new HashMap<>();

  public TypeAliasRegistry() {
    registerAlias("string", String.class);

    registerAlias("byte", Byte.class);
    registerAlias("long", Long.class);
    registerAlias("short", Short.class);
 //...
```

则意味着，你在写这样的标签的时候可以这样写。

```xml
<select id="selectBlog" parameterType="int" resultType="com.pop.domain.Blog">
        select blog_id blogId, blog_name blogName
        from blog where blog_id = #{blogId}
</select>
<!--改写-->
<select id="selectAuthor" parameterType="int" resultType="blog">
        select blog_id blogId, blog_name blogName
        from blog where blog_id = #{blogId}
</select>
<!--或者-->
<select id="xxx" parameterType="blog" >
      <!--....-->
</select>
```

###### TypeHandlers

具体类型java对象->jdbc自定义转换

```xml
<typeHandlers>
    <typeHandler handler="com.pop.type.MyTypeHandler"></typeHandler>
</typeHandlers>
```
Mybatis里默认的类型处理。

```java
public final class TypeHandlerRegistry {

//....

  private Class<? extends TypeHandler> defaultEnumTypeHandler = EnumTypeHandler.class;

  public TypeHandlerRegistry() {
    register(Boolean.class, new BooleanTypeHandler());
    register(boolean.class, new BooleanTypeHandler());
    register(JdbcType.BOOLEAN, new BooleanTypeHandler());
    register(JdbcType.BIT, new BooleanTypeHandler());

    register(Byte.class, new ByteTypeHandler());
    register(byte.class, new ByteTypeHandler());
    register(JdbcType.TINYINT, new ByteTypeHandler());
//...
```

自己定义的jdbc到java类型的规则，java到jdbc的规则。

```java
public class MyTypeHandler extends BaseTypeHandler<String> {
    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType)
            throws SQLException {
        // 设置 String 类型的参数的时候调用，Java类型到JDBC类型
        // 注意只有在字段上添加typeHandler属性才会生效
        // insertBlog name字段
        // java->jdbc的转换
        System.out.println("---------------setNonNullParameter1："+parameter);
        ps.setString(i, parameter);
    }

    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
        // 根据列名获取 String 类型的参数的时候调用，JDBC类型到java类型
        // 注意只有在字段上添加typeHandler属性才会生效
        // jdbc->java的转换 传入行名
        System.out.println("---------------getNullableResult1："+columnName);
        return rs.getString(columnName);
    }

    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        // 根据下标获取 String 类型的参数的时候调用
        System.out.println("---------------getNullableResult2："+columnIndex);
        // jdbc->java的转换 行位置索引
        return rs.getString(columnIndex);
    }

    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        System.out.println("---------------getNullableResult3：");
        // jdbc->java的转换 行位置索引，存储过程
        return cs.getString(columnIndex);
    }
}
```

你也可以指定扫描一个包下的类型转换规则。

```xml
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

当你的类似配置完成后，用到的话就是插入的时候和查询的时候，因为都设计类型转换的问题。

```xml
<resultMap id="BaseResultMap" type="blog">
        <id column="bid" property="bid" jdbcType="INTEGER"/>
<!--
       <result column="name" property="name" jdbcType="VARCHAR"/>
-->
    	 <result column="name" property="name" jdbcType="VARCHAR" typeHandler="com.gupaoedu.type.MyTypeHandler"/>
        <result column="author_id" property="authorId" jdbcType="INTEGER"/>
    </resultMap>
```

###### ObjectFactory

MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。 如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。比如

```java
public class ExampleObjectFactory extends DefaultObjectFactory {
   //适用于无参构造器
  @Override
  public Object create(Class type) {
     System.out.println("创建对象方法：" + type);
        if (type.equals(Blog.class)) {
            //在这里面加一些自己的配置
            //例如你想对某个商品的卖出数目好看点，可以进行价格乘以多少
            //又或者身份证号或者手机号中间几位用*号显示，这样可以隐藏
            Blog blog = (Blog) super.create(type);
            blog.setName("object factory");
            blog.setBid(1111);
            blog.setAuthorId(2222);
            return blog;
        }
        Object result = super.create(type);
     return result;
  }
  // 适合有参的构造器
  @Override
  public Object create(Class type, List<Class> constructorArgTypes, List<Object> constructorArgs) {
    return super.create(type, constructorArgTypes, constructorArgs);
  }
  public void setProperties(Properties properties) {
    super.setProperties(properties);
  }
  public <T> boolean isCollection(Class<T> type) {
    return Collection.class.isAssignableFrom(type);
  }}
//你可以选择适合自己的去重写方法
```

在xml文件中配置

```xml
<objectFactory type="com.pop.objectfactory.IObjectFactory">
      <property name="pop" value="666"/>
</objectFactory>
```

#### 动态SQL

```xml
<!-- List 批量删除  -->
    <delete id="deleteByList" parameterType="java.util.List">
        delete from tbl_emp where emp_id in
        <foreach collection="list" item="item" open="(" separator="," close=")">
            #{item.empId,jdbcType=VARCHAR}
        </foreach>
    </delete>
<!-- 批量插入 -->
    <insert id="batchInsert" parameterType="java.util.List" useGeneratedKeys="true">
        <selectKey resultType="long" keyProperty="id" order="AFTER">
            SELECT LAST_INSERT_ID()
        </selectKey>
        insert into tbl_emp (emp_id, emp_name, gender,email, d_id)
        values
        <!--对应的方法参数上的集合名字就叫list，如果是其他名字也可以写-->
        <foreach collection="list" item="emps" index="index" separator=",">
            ( #{emps.empId},#{emps.empName},#{emps.gender},#{emps.email},#{emps.dId} )
        </foreach>
    </insert>
<!--有选择的插入-->
    <insert id="insertSelective" parameterType="com.pop.crud.bean.Employee">
        insert into tbl_emp
        <trim prefix="(" suffix=")" suffixOverrides=",">
            <if test="empId != null">
                emp_id,
            </if>
            <if test="empName != null">
                emp_name,
            </if>
            <if test="gender != null">
                gender,
            </if>
            <if test="email != null">
                email,
            </if>
            <if test="dId != null">
                d_id,
            </if>
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            <if test="empId != null">
                #{empId,jdbcType=INTEGER},
            </if>
            <if test="empName != null">
                #{empName,jdbcType=VARCHAR},
            </if>
            <if test="gender != null">
                #{gender,jdbcType=CHAR},
            </if>
            <if test="email != null">
                #{email,jdbcType=VARCHAR},
            </if>
            <if test="dId != null">
                #{dId,jdbcType=INTEGER},
            </if>
        </trim>
    </insert>

```

关于批处理的语句，带一句Mybatis中由SqlSessionFactory创建出来的SqlSession中正在执行方法的是Excutor（执行器），在官网Excutor拥有三种类型`simple,reuse,batch`

* SimpleExcutor 是Mybatis的默认配置，他支持statment，也就是语句创建，用完就会销毁
* ReuseExcutor 使用的preparestatement，支持预编译，会将编译后的语句存储起来，方便下次使用
* BatchExcutor 可以执行批处理操作，其实就是使用了preparestatement进行了addbatch操作最后excutebatch，可以将语句缓存起来最后一期发送给数据库，提高批量删除和批量插入的效率。



#### 嵌套查询

例如一个博客有一名作者，这就是简单的一对一的关系 `one to one`，其中一个关系会变成属性放在对象里面。像是这样。

```java
public class BlogAndAuthor implements Serializable {
    Integer bid; // 文章ID
    String name; // 文章标题
    Author author; // 作者
    //...
```

当你使用这个对象去sql操作的时候mybatis是如何将他转换的。

##### 嵌套查询

在mapper文件里，你需要这样配置，这个author的属性要写成这样。

```xml
<!-- 第一种的嵌套查询-->
<!-- 根据文章查询作者，一对一查询的结果，嵌套查询 -->
<resultMap id="BlogWithAuthorResultMap" type="com.gupaoedu.domain.associate.BlogAndAuthor">
        <id column="bid" property="bid" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <!-- 联合查询，将author的属性映射到ResultMap -->
        <association property="author" javaType="com.gupaoedu.domain.Author">
            <id column="author_id" property="authorId"/>
            <result column="author_name" property="authorName"/>
        </association>
    </resultMap>

<!-- 根据文章查询作者，一对一，嵌套结果，无N+1问题 可以根据设置的map变成相应对象-->
    <select id="selectBlogWithAuthorResult" resultMap="BlogWithAuthorResultMap" >
        select b.bid, b.name, b.author_id, a.author_id , a.author_name
        from blog b
        left join author a
        on b.author_id=a.author_id
        where b.bid = #{bid, jdbcType=INTEGER}
    </select>

<!-- 第二种嵌套查询-->
<!-- 另一种联合查询(一对一)的实现，但是这种方式有“N+1”的问题 -->
    <resultMap id="BlogWithAuthorQueryMap" type="com.gupaoedu.domain.associate.BlogAndAuthor">
        <id column="bid" property="bid" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <!--这里不再定义完整的结构，而是变成了对 select 属性配置 
		-->
        <association property="author" javaType="com.gupaoedu.domain.Author"
                     column="author_id" select="selectAuthor"/> <!-- selectAuthor 定义在下面-->
    </resultMap>

<!-- 嵌套查询 -->
    <select id="selectAuthor" parameterType="int" resultType="com.gupaoedu.domain.Author">
        select author_id authorId, author_name authorName
        from author where author_id = #{authorId}
    </select>

<!-- 根据文章查询作者，一对一，嵌套查询，存在N+1问题，可通过开启延迟加载解决 -->
    <select id="selectBlogWithAuthorQuery" resultMap="BlogWithAuthorQueryMap" >
        select b.bid, b.name, b.author_id, a.author_id , a.author_name
        from blog b
        left join author a
        on b.author_id=a.author_id
        where b.bid = #{bid, jdbcType=INTEGER}
    </select>
```

**N+1问题**

以author和blog作为例子，在第二种嵌套查询中，每当你执行`selectBlogWithAuthorQuery`这个查询时候，作为结果返回的`BlogWithAuthorQueryMap`中也同样会进行`association`中`selectAuthor`的再次查询。也就是意味着，如果你的一条查询语句查询出了多条结果，由于每条结果都有`association`这样的属性，即便你没有使用这个属性也会进一步触发你下一次查询，也就是`selectAuthor`里面定义的这条语句。1指的是一条查询语句，N指的是可能由于查询多条结果再次触发多条查询。

如果我没有用到author属性，所以我不想让他触发查询的话，可以使用延迟加载机制，在属性文件中配置，这个属性默认是false的

```xml
<settings>
        <!-- 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。默认 false  -->
        <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```

配置这个属性后，当你调用某个特殊的方法时才会触发查询。例如`getAuthor()`

如果你配置了

```xml
<settings>
        <!-- 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。默认 false  -->
        <setting name="lazyLoadingEnabled" value="true"/>
         <!-- 当开启时，任何方法的调用都会加载该对象的所有属性。默认 false，可通过select标签的 fetchType来覆盖-->
        <setting name="aggressiveLazyLoading" value="true"/>
</settings>
```

无论你调用什么方法，都会触发一下次查询，toString，HashCode都会触发查询。你也可以自定义方法来触发

```xml

<settings>
        <!-- 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。默认 false  -->
        <setting name="lazyLoadingEnabled" value="true"/>
         <!-- 当开启时，任何方法的调用都会加载该对象的所有属性。默认 false，可通过select标签的 fetchType来覆盖-->
        <setting name="aggressiveLazyLoading" value="true"/>
        <!--指定哪个对象的方法触发一次延迟加载-->
        <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

https://mybatis.org/mybatis-3/zh/configuration.html

##### 嵌套结果

这其实是个很常见的操作，例如一个作者会有多篇博客，就是一对多的关系 `one to many`，然而

##### 一对多

entity

```java
public class BlogAndComment {
    Integer bid; // 文章ID
    String name; // 文章标题
    Integer authorId; // 文章作者ID
    List<Comment> comment; // 文章评论
```

mapper配置

```xml
<resultMap id="BaseResultMap" type="blog">
        <id column="bid" property="bid" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <result column="author_id" property="authorId" jdbcType="INTEGER"/>
</resultMap>

 <!--  查询文章带评论的结果（一对多） -->
    <resultMap id="BlogWithCommentMap" type="com.pop.domain.associate.BlogAndComment" extends="BaseResultMap" >
        <collection property="comment" ofType="com.pop.domain.Comment">
            <id column="comment_id" property="commentId" />
            <result column="content" property="content" />
        </collection>
    </resultMap>

!-- 根据文章查询评论，一对多 -->
    <select id="selectBlogWithCommentById" resultMap="BlogWithCommentMap" >
        select b.bid, b.name, b.author_id authorId, c.comment_id commentId, c.content
        from blog b, comment c
        where b.bid = c.bid
        and b.bid = #{bid}
    </select>
```

##### 多对多（尽量避免）

entity

```java
public class AuthorAndBlog {
    Integer author_id; // 作者ID
    String author_name; // 作者名称
    List<BlogAndComment> blog; // 文章和评论列表
```

mapper

```xml
<!--  按作者查询文章评论的结果（多对多） -->
    <resultMap id="AuthorWithBlogMap" type="com.pop.domain.associate.AuthorAndBlog" >
        <id column="author_id" property="authorId" jdbcType="INTEGER"/>
        <result column="author_name" property="authorName" jdbcType="VARCHAR"/>
        <collection property="blog" ofType="com.pop.domain.associate.BlogAndComment">
            <id column="bid" property="bid" />
            <result column="name" property="name" />
            <result column="author_id" property="authorId" />
                <collection property="comment" ofType="com.pop.domain.Comment">
                    <id column="comment_id" property="commentId" />
                    <result column="content" property="content" />
                </collection>
    	</collection>
    </resultMap>

<!-- 根据作者文章评论，多对多 -->
    <select id="selectAuthorWithBlog" resultMap="AuthorWithBlogMap" >
        select b.bid, b.name, a.author_id authorId, a.author_name authorName, c.comment_id commentId, c.content
        from blog b, author a, comment c
        where b.author_id = a.author_id and b.bid = c.bid
    </select>
```

#### Mapper文件的继承

如果你有一个Mapper.xml文件，你想要重用里面的内容，你可以选择拷贝一份，再加点什么东西，可以这里面的东西就太多了，能不能像java一样继承父类就自动拥有了父类的方法，只需要在子类里加自定义的参数就可以了呢。

```java
public interface BlogMapper {
    /**
     * 根据主键查询文章
     * @param bid
     * @return
     */
    public Blog selectBlogById(Integer bid);

    /**
     * 根据实体类查询文章
     * @param blog
     * @return
     */
    public List<Blog> selectBlogByBean(Blog blog );
//...
```

mapper文件

```xml
<mapper namespace="com.gupaoedu.mapper.BlogMapper">
    <!-- ...-->
```

然后你再定义一个

```java
public interface BlogMapperExt extends BlogMapper {
    /**
     * 根据名称查询文章
     * @param name
     * @return
     */
    public Blog selectBlogByName(String name);
}
```

mapper

```xml
<mapper namespace="com.gupaoedu.mapper.BlogMapperExt">
    <!-- 只能继承statement，不能继承sql、resultMap等标签 -->
    <resultMap id="BaseResultMap" type="blog">
        <id column="bid" property="bid" jdbcType="INTEGER"/>
        <result column="name" property="name" jdbcType="VARCHAR"/>
        <result column="author_id" property="authorId" jdbcType="INTEGER"/>
    </resultMap>

    <!-- 在parent xml 和child xml 的 statement id相同的情况下，会使用child xml 的statement id -->
    <select id="selectBlogByName" resultMap="BaseResultMap" statementType="PREPARED">
        select * from blog where name = #{name}
    </select>
</mapper>
```

#### Mybatis的流程

![1581004106379](./img/1581004106379.png)

`解析配置文件->创建工厂类->创建会话->会话操作数据库`

##### 架构图

![1581004333689](./img/1581004333689.png)

##### 缓存

![1581005070018](./img/1581005070018.png)



MyBatis 跟缓存相关的类都在 cache 包里面，其中有一个 Cache 接口，只有一个默 认的实现类 `PerpetualCache`，它是用 HashMap 实现的。 除此之外，还有很多的装饰器，通过这些装饰器可以额外实现很多的功能：回收策 略、日志记录、定时刷新等等。 

| 缓存实现类          | 描述             | 作用                                                         | 装饰条件                                          |
| ------------------- | ---------------- | ------------------------------------------------------------ | ------------------------------------------------- |
| 基本缓存            | 缓存基本实现类   | 默认是 PerpetualCache，也可以自定义比如 RedisCache、EhCache 等，具备基本功能的缓存类 | 无                                                |
| LruCache            | LRU 策略的缓存   | 当缓存到达上限时候，删除最近最少使用的缓存 （Least Recently Use） | eviction="LRU"（默认）                            |
| FifoCache           | FIFO 策略的缓存  | 当缓存到达上限时候，删除最先入队的缓存                       | eviction="FIFO"                                   |
| SoftCache WeakCache | 带清理策略的缓存 | 通过 JVM 的软引用和弱引用来实现缓存，当 JVM 内存不足时，会自动清理掉这些缓存，基于SoftReference 和 WeakReference | eviction="SOFT" eviction="WEAK"                   |
| LoggingCache        | 带日志功能的缓存 | 比如：输出缓存命中率                                         | 基本                                              |
| SynchronizedCache   | 同步缓存         | 基于 synchronized 关键字实现，解决并发问题                   | 基本                                              |
| SerializedCache     | 支持序列化的缓存 | 将对象序列化以后存到缓存中，取出时反序列化                   | readOnly=false（默 认）                           |
| ScheduledCache      | 定时调度的缓存   | 在进行 get/put/remove/getSize 等操作前，判断 缓存时间是否超过了设置的最长缓存时间（默认是一小时），如果是则清空缓存--即每隔一段时间清 | flushInterval 不为 空                             |
| TransactionalCache  | 事务缓存         | 在二级缓存中使用，可一次存入多个缓存，移除多 个缓存          | 在TransactionalCacheManager 中用 Map 维护对应关系 |

###### 一级缓存

他是会话级别，SqlSession级别的。他存在Executor中。

```java
public class DefaultSqlSession implements SqlSession {

  private final Configuration configuration;// 缓存不太可能存在配置里面
  private final Executor executor;// 只有他了
    //...
    
public abstract class BaseExecutor implements Executor {

  private static final Log log = LogFactory.getLog(BaseExecutor.class);

  protected Transaction transaction;
  protected Executor wrapper;

  protected ConcurrentLinkedQueue<DeferredLoad> deferredLoads;
  protected PerpetualCache localCache; // 在这里 PerpetualCache 是Cache的唯一实现
  protected PerpetualCache localOutputParameterCache;
  protected Configuration configuration;
    // ....
```

![1581006740443](./img/1581006740443.png)

​	尝试去一级缓存中取得，如果没有就去数据库去查。

要测试一级缓存，首先要将二级缓存关闭。

```xml
<settings>
	<!-- 控制全局缓存（二级缓存）默认是开启的-->
    <setting name="cacheEnabled" value="false"/>
</settings>
```

mapper中也不能有相关二级缓存的配置。同一个Session中会共享，不同的Session是无法共享的。当你执行了一次删除修改更新操作时，一级缓存就会被更新。如果你不想执行某个操作的时候不清空缓存的话，可以在mapper文件中配置。

```xml
<!-- select 标签默认是关闭的，也就是false-->
<select id="selectBlogList" resultMap="BaseResultMap" flushCache="false">
        select bid, name, author_id authorId from blog
    </select>
<!-- update 标签默认是开启的，也就是true-->
<update id="updateByPrimaryKey" parameterType="blog" flushCache="true">
        update blog
        set name = #{name,jdbcType=VARCHAR}
        where bid = #{bid,jdbcType=INTEGER}
    </update>
<!--你也指定哪个语句不要更新缓存，设置成false就可以了-->
```

###### 二级缓存

二级缓存的作用域更广，是`namespace`级别，也就是一个mapper.xml文件里的所有sql语句都可以共享同一个会话，同时，二级缓存开启后，他是优先与一级缓存的。

![1581007426063](./img/1581007426063.png)

同时，如果你的事务不提交的话，是无法写入二级缓存的，所以二级缓存还是支持事务的。

```java
public class CachingExecutor implements Executor {

  private final Executor delegate;
    //这里
  private final TransactionalCacheManager tcm = new TransactionalCacheManager();
    //...

public class TransactionalCacheManager {
	// 这里
  private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<>();

  public void clear(Cache cache) {
    getTransactionalCache(cache).clear();
  }

  public Object getObject(Cache cache, CacheKey key) {
    return getTransactionalCache(cache).getObject(key);
  }

  public void putObject(Cache cache, CacheKey key, Object value) {
    getTransactionalCache(cache).putObject(key, value);
  }

  public void commit() {
    for (TransactionalCache txCache : transactionalCaches.values()) {
      txCache.commit();//当事务提交了以后，缓存才会被提交
    }
  }
    //....
    
public class TransactionalCache implements Cache {

  private static final Log log = LogFactory.getLog(TransactionalCache.class);

  private final Cache delegate;
  private boolean clearOnCommit;
  private final Map<Object, Object> entriesToAddOnCommit;
  private final Set<Object> entriesMissedInCache;
//...
```

**如何开启二级缓存**

`mybatis-config.xml`里面

```xml
<settings>
	<!-- 控制全局缓存（二级缓存）默认是开启的-->
    <setting name="cacheEnabled" value="true"/>
</settings>
```

同时，还有在mapper里面指定。

```xml
<mapper namespace="com.gupaoedu.mapper.BlogMapper">
    <!-- 声明这个namespace使用二级缓存 他的其他配置都可以是默认的-->
    <cache/>
    <!-- 使用Redis作为二级缓存 -->
<!--
    <cache type="org.mybatis.caches.redis.RedisCache"
           eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
-->
    <!--        <cache type="org.apache.ibatis.cache.impl.PerpetualCache"
               size="1024"
               eviction="LRU"
               flushInterval="120000"
               readOnly="false"/>-->
    <!--....-->
```

如果你的映射方法里，有个方法不希望使用二级缓存，可以配置。

```xml
<select id="selectBlogList" resultMap="BaseResultMap" useCache="false">
        select bid, name, author_id authorId from blog
    </select>
```

二级的缓存的缺点也是很明显的，就是由于可以跨Session共享，当某个Session执行了一个更新操作的时候就会清空其它Session的缓存。

所以我们尽量在查询为主的语句中使用二级缓存。



#### MyBatis源码分析

最原始的mybatis编程

```java
public void testSelect() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = sqlSessionFactory.openSession(); // ExecutorType.BATCH
        try {
            BlogMapper mapper = session.getMapper(BlogMapper.class);
            Blog blog = mapper.selectBlogById(1);
            System.out.println(blog);
        } finally {
            session.close();
        }
    }
```

