

# 基础知识

## mybatis如何实现只写接口和mapper就能进行数据库的查询的？



## MyBatis对象字段映射的细节



## MyBatis如何实现数据库字段与JavaBean间映射（I/O流读取XML文件，其中包含类全限定名，通过反射实例化对象）

  

## 如果是你实现，会使用什么技术实现数据库映射到JavaBean？（反射，面试官一直嗯嗯嗯我也不知道对不对）



## 为什么说 MyBatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？

Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而 MyBatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具

## JDBC编程有哪些不足之处，MyBatis是如何解决这些问题的？

1. 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。 
   -  解决：在`SqlMapConfig.xml`中配置数据链接池，**使用连接池管理数据库链接**。

2. Sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。 
   - 解决：将Sql语句配置在`XXXXmapper.xml`文件中与java代码分离

3. 向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。 
   - 解决： **Mybatis自动将java对象映射至sql语句**。

4. 对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。
   - 解决：Mybatis自动将sql执行结果映射至java对象。

## MyBatis编程步骤是什么样的？
1. 创建SqlSessionFactory 
2. 通过SqlSessionFactory创建`SqlSession`
3. 通过sqlsession执行数据库操作
4. 调用`session.commit()`提交事务 
5. 调用`session.close()`关闭会话

## MyBatis与Hibernate有哪些不同？

1. Mybatis 和 hibernate 不同，它不完全是一个 ORM 框架，因为 MyBatis 需要程序员自己编写 Sql 语句。 

2. **Mybatis 直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高**，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。
    但是灵活的代价是 mybatis 无法做到数据库无关性， 如果需要实现支持多种数据库的软件，则需要自定义多套 sql 映射文件，工作量大。 

3. Hibernate 对象/关系映射能力强，数据库无关性好，对于关系模型要求高的 软件，如果用 hibernate 开发可以节省很多代码，提高效率 

## Mybaits 的优点：

1. 基于 SQL 语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任 何影响。
   - SQL 写在 XML 里，解除 sql 与程序代码的耦合，便于统一管理；
   - 提供 XML 标签，支持编写动态 SQL 语句，并可重用。

2. 与 JDBC 相比，减少了 50%以上的代码量，**消除了 JDBC 大量冗余的代码**，**不需要手动开关连接**，降低了开销； 

3. 很好的与各种数据库兼容（因为 MyBatis 使用 JDBC 来连接数据库，所以只要 JDBC 支持的数据库 MyBatis 都支持）。

4. 能够与 Spring 很好的集成；
5. 提供映射标签，**支持对象与数据库的 ORM 字段关系映射**；
   - 提供对象关系映射标签，支持对象关系组件维护

## MyBatis 框架的缺点： 

1. SQL 语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写 SQL 语句的功底有一定要求。

2. SQL 语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

## #{}和${}的区别？
- `${}`是 **`Properties` 文件中的变量占位符**
  - 实现方式为**静态字符串替换**，它可以用于标签属性值和 sql 内部，比如`${driver}`会被静态替换为`com.mysql.jdbc.Driver`
  - 在处理`${}`时，就是把`${}`替换成变量的值。

- `#{}`是 **sql 的参数占位符**，实现方式为**预编译处理**
  - MyBatis 会将 sql 中的`#{}`替换为`?`号，在 sql 执行前会使用 `PreparedStatement` 的`set()`方法赋值参数
    - 比如 `ps.setInt(0, parameterValue)`，`#{item.name}` 的取值方式为**使用反射**从参数对象中获取 `item` 对象的 `name` 属性值，相当于 `param.getItem().getName()`
  
**使用#{}可以有效的防止SQL注入，提高系统安全性**。

## 通常一个Xml映射文件，都会写一个Dao接口与之对应，那么这个Dao接口的工作原理是什么？Dao接口里的方法、参数不同时，方法能重载吗？

`Dao`接口即`Mapper`接口。
- 接口的**全限定名**就是映射文件中的`namespace`的值；
- 接口的**方法名**，就是映射文件中`Mapper`的`Statement`的`id`值；
- 接口方法内的参数，就是传递给sql的参数。

Mapper接口是没有实现类的，当调用接口方法时，使用**接口全限名+方法名**的拼接字符串作为key值，可唯一定位一个`MapperStatement`，xml文件中的每个`select`、`insert`、`update`、`delete`标签都会被解析为一个`MappedStatement`对象。

之后，通过==JDK动态代理==的方式，创建Mapper接口的动态代理对象Proxy，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。

Dao中的方法是可以重载的，但是XML文件中的ID不允许重复，因为是通过**全限名+方法名**进行MappedStatement的查找的，而没有将参数作为查找key---

**Mybatis 的 Dao 接口可以有多个重载方法，但是多个接口对应的映射必须只有一个，否则启动会报错。**

Dao接口方法在以下情况下，可以进行重载：
- 仅有一个无参方法和一个有参方法
- 多个有参方法时，参数数量必须一致。且使用相同的 @Param ，或者使用 param1 这种

测试：
```java
Person queryById();

Person queryById(@Param("id") Long id);

Person queryById(@Param("id") Long id, @Param("name") String name);
```
```xml
<select id="queryById" resultMap="PersonMap">
    select
      id, name, age, address
    from person
    <where>
        <if test="id != null">
            id = #{id}
        </if>
        <if test="name != null and name != ''">
            name = #{name}
        </if>
    </where>
    limit 1
</select>
```

获取`<if>`标签中属性值的源码如下：
```java
public Object getProperty(Map context, Object target, Object name) {
  Map map = (Map) target;

  Object result = map.get(name);
  if (map.containsKey(name) || result != null) {
    return result;
  }

  Object parameterObject = map.get(PARAMETER_OBJECT_KEY);
  if (parameterObject instanceof Map) {
    return ((Map)parameterObject).get(name);
  }

  return null;
}
```

其中：
- `queryById()`执行时，`parameterObject`为null，所以`getProperty()`返回null，动态sql可以正常执行
- `queryById(long id)`执行时，`parameterObject`为`map`，包含`id`与`param1`这两个key值，但当尝试获取`<if>`标签中的`name`属性值时，`map.get(name)`会由于`map`的key不包含`name`，直接抛出异常（
  - 这里是mybatis代码写成这样啊！不是map的锅！
- `queryById(long id, String name)`可以正常执行

## 使用MyBatis的Mapper接口调用时有哪些要求？

1. Mapper接口**方法名**和mapper.xml中定义的每个**sql的id**相同；
2. Mapper接口方法的**输入参数类型**和mapper.xml中定义的每个sql的`parameterType`类型相同；
3. Mapper接口方法的**输出参数类**型和mapper.xml中定义的每个sql的`resultType`的类型相同；
4. Mapper.xml文件中的`namespace`即是mapper接口的**类路径**。

## 在Mapper中如何传递多个参数？

1. 若Dao层函数有多个参数，那么其对应的xml中，`#{0}`代表接收的是Dao层中的第一个参数，#{1}代表Dao中的第二个参数，以此类推。

2. 使用`@Param`注解：在Dao层的参数中前加@Param注解,注解内的参数名为传递到Mapper中的参数名。

3. 多个参数封装成`Map`，以HashMap的形式传递到Mapper中。

## Mybatis动态sql有什么用？执行原理是什么？有哪些动态sql？

Mybatis动态sql可以在xml映射文件内，以标签的形式编写动态sql，执行原理是根据表达式的值完成逻辑判断，并动态拼接sql的功能。

Mybatis提供了9种动态sql标签：trim、where、set、foreach、if、choose、when、otherwise、bind

在解析xml文件时，会将其中的动态SQL段解析为树形结构，并逐个遍历，在其中对各种文本节点、动态sql标签节点进行处理，使用OGNL计算参数表达式的值，根据值动态拼接sql，最终拼接为完整的sqlNode

## xml映射文件中，不同的xml映射文件id是否可以重复？

不同的xml映射文件，如果配置了`namespace`，那么id可以重复；如果没有配置`namespace`，那么id不能重复；

原因是`namespace+id`是作为`Map<String,MapperStatement>`的key使用的。
如果没有namespace，就剩下id，那么id重复会导致数据互相覆盖。
有了namespace，自然id就可以重复，namespace不同，namespace+id自然也不同。

## Mybatis实现一对一关联查询有几种方式？具体是怎么操作的？

> 一对一关联查询：例如一个订单表，其中一个属性为用户User（不是用户id）

有**联合查询**和**嵌套查询**两种方式。

联合查询（单步查询）是**多表联合查询**，通过`resultMap`映射实体类与字段之间的一一对应关系，在其中配置`association`，并通过`javaType`指定需要嵌套的类，**实现被嵌套的Bean的关联与其属性的赋值**，从而实现一对一的关联查询；

> 多表联合查询是在一个SQL语句中联合查询多个表，而嵌套查询是组合多条SQL语句的结果

嵌套查询（分步查询）是先执行查询被嵌套的类的表，根据查询结果，再去主表里面查询数据。也是通过`association`配置，但`association`中的是`select`语句，且需要通过`column`指定用于关联的属性，通常为被嵌套表的外键。

一对多的实现方式也是类似的，但并不是使用`association`标签，而是`collection`标签

多对多查询：A可以包含多个B，B也可以包含多个A，但实际上也就是相当于一个**一对多对多查询**，即通过嵌套的`collection`标签实现

## MyBatis延迟初始化

延迟加载：在关联查询时，利用延迟加载，先加载主信息。真正需要关联信息时，再去按需加载关联信息。这样会大大提高数据库性能。

> MyBatis默认关闭延迟初始化。在**一对多**与**多对多**查询时，通常开启延迟加载

以上文的订单与用户的一对一关联查询为例：
- 当代码中使用的内容**只有订单表中的内容**的时候，不需要查询关联的用户表
- 当代码中使用的内容**包含用户表**的时候，需要调用查询用户表

它的原理是：
1. 使用 `CGLIB` 创建目标对象的代理对象
2. 当调用目标方法时，直接进入拦截器方法，比如调用 `a.getB().getName()` ，拦截器 `invoke()` 方法发现 `a.getB()` 是 `null` 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 `a.setB(b)`，于是 a 的对象 b 属性就有值了，接着完成 `a.getB().getName()` 方法的调用。这就是延迟加载的基本原理。

但是`Lazy Load`也有缺点，在按需加载时会多次连接数据库， 同时会增加数据库的压力。所以在实际使用时，会衡量是否使 用 延迟加载。

## Mybatis的一级、二级缓存

![](./../images/mybatis/缓存.png)


缓存数据更新机制：当某一个作用域（一级缓存Session/二级缓存Namespace）进行了**增/删/改**操作后，默认该作用域下所有select中的缓存将被clear。

`SqlSession`执行`commit`后，也会清空一级缓存

1. 一级缓存：基于`PerpetualCache`的`HashMap`本地缓存，其存储作用域为`Session`，不同SqlSession之间不共享。当Session flush或close之后，该Session中的所有Cache就将清空。
   - 默认打开一级缓存。
2. 二级缓存与一级缓存机制相同，默认也是采用`PerpetualCache`，`HashMap`存储，不同在于其存储作用域为`Mapper（namespace）`，并且可自定义存储源，如Ehcache。
   - 默认打不开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现`Serializable`序列化接口（可用来保存对象的状态），可在它的映射文件中配置。

> 注意：通过spring介入mybatis时，数据库处于自动提交状态，**每一条sql语句处于一个单独的事务中**，此时mybatis的一级缓存无效，需要开启事务，才能利用到一级缓存
> > - 开启事务：spring使用`ThreadLocal`获取当前资源绑定的`SQLSession`，即可以使用一级缓存
> > - 未开启事务：每次查询，spring都会关闭旧的SqlSession，创建一个新的Sqlsession对象，一级缓存不起作用

## Xml 映射文件中，除了常见的 select|insert|update|delete 标签之外，还有哪些标签？

- resultMap
- parameterMap
- sql：sql片段标签
- include：引入sql片段
- selectKey：不支持自增的主键生成策略
- 9个动态sql标签`trim、where、set、when、if、foreach、choose、otherwise、bind`等


## MyBatis 是如何进行分页的？分页插件的原理是什么？

1.  MyBatis 使用 `RowBounds` 对象进行分页，它是针对 `ResultSet` 结果集执行的**内存分页**，而非物理分页；
2.  可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，
3.  也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内，**拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数**。

## 简述 MyBatis 的插件运行原理，以及如何编写一个插件。

MyBatis 仅可以编写针对 `ParameterHandler` 、 `ResultSetHandler` 、 `StatementHandler` 、 `Executor` 这 4 种接口的插件。
MyBatis 使用 ==JDK 的动态代理==，**为需要拦截的接口生成代理对象以实现接口方法拦截功能**。
每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 `InvocationHandler` 的 `invoke()` 方法，当然，只会拦截那些你指定需要拦截的方法。

所以，要编写一个插件，只要实现 MyBatis 的 Interceptor 接口并复写 `intercept()` 方法，然后在给插件编写注解，**指定要拦截哪一个接口的哪些方法即可**，记住，别忘了在配置文件中配置你编写的插件

## MyBatis 执行批量插入，能返回数据库主键列表吗？

JDBC和MyBatis都能

## MyBatis 是如何将 sql 执行结果封装为目标对象并返回的？都有哪些映射形式？

1. 使用 `<resultMap>` 标签，逐一定义列名和对象属性名之间的映射关系。
2. 使用 sql 列的别名功能，将列别名书写为对象属性名，比如 `T_NAME AS NAME`，对象属性名一般是 `name`，**小写**，但是**列名不区分大小写**，MyBatis 会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成 `T_NAME AS NaMe`，MyBatis 一样可以正常工作。

有了列名与属性名的映射关系后，MyBatis 通过**反射**创建对象，同时使用**反射**给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的


## MyBatis 的 Xml 映射文件中，不同的 Xml 映射文件，id 是否可以重复？

- 不同的 Xml 映射文件，如果配置了 `namespace`，那么 **id 可以重复**；
- 如果没有配置 `namespace`，那么**id 不能重复**；毕竟 namespace 不是必须的，只是最佳实践而已。

原因就是 `namespace+id` 是作为 `Map<String, MappedStatement>` 的 key 使用的，如果没有 `namespace`，就剩下 id，那么，**id 重复会导致数据互相覆盖**。有了 namespace，自然 id 就可以重复，namespace 不同，namespace+id 自然也就不同


## MyBatis 都有哪些 Executor 执行器？它们之间的区别是什么？

- `SimpleExecutor`：
  - **每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象**。
- `ReuseExecutor`： 
  - 执行 `update` 或 `select`，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建。
  - 用完后，不关闭 Statement 对象，而是放置于 `Map<String, Statement>`内，供下一次使用。简言之，就是**重复使用 Statement 对象**
- `BatchExecutor`： 
  - 执行 `update`（**没有 select，JDBC 批处理不支持 select**），将所有 sql 都添加到批处理中（`addBatch()`），等待统一执行（`executeBatch()`）
  - BatchExecutor缓存了多个 `Statement` 对象，每个 `Statement` 对象都是 `addBatch()`完毕后，等待逐一执行 `executeBatch()`批处理。与 JDBC 批处理相同

Executor的共同点：作用范围都控制在SqlSession的生命周期范围内

## MyBatis 中如何指定使用哪一种 Executor 执行器？

1. 在MyBatis的配置文件中，指定默认的`ExecutorType`
2. 手动给`DefaultSqlSessionFactory`的创建`SqlSession`的方法传递`ExecutorType`类型参数

## MyBatis 是否可以映射 Enum 枚举类？

**MyBatis 可以映射枚举类**，不单可以映射枚举类，**MyBatis 可以映射任何对象到表的一列上**。

映射方式为自定义一个 `TypeHandler` ，实现 `TypeHandler` 的 `setParameter()` 和 `getResult()` 接口方法。 
TypeHandler 有两个作用：
1. 完成从 `javaType` 至 `jdbcType` 的转换
2. 完成 `jdbcType` 至 `javaType` 的转换

体现为 `setParameter()` 和 `getResult()` 两个方法，分别代表**设置 sql 问号占位符参数**和**获取列查询结果**

## MyBatis 映射文件中，如果 A 标签通过 include 引用了 B 标签的内容，请问，B 标签能否定义在 A 标签的后面，还是说必须定义在 A 标签的前面？

虽然 MyBatis 解析 Xml 映射文件是按照顺序解析的，但是，被引用的 B 标签依然可以定义在任何地方，MyBatis 都可以正确识别

原理是，MyBatis 解析 A 标签，发现 A 标签引用了 B 标签，但是 B 标签尚未解析到，尚不存在。
此时，MyBatis 会将 A 标签标记为**未解析状态**，然后继续解析余下的标签，包含 B 标签。
待所有标签解析完毕，MyBatis 会**重新解析那些被标记为未解析的标签**，此时再解析 A 标签时，B 标签已经存在，A 标签也就可以正常解析完成了

## 简述 MyBatis 的 Xml 映射文件和 MyBatis 内部数据结构之间的映射关系？

MyBatis 将所有 Xml 配置信息都封装到 `All-In-One` 重量级对象 `Configuration` 内部。

在 Xml 映射文件中：
1. `<parameterMap>` 标签会被解析为 `ParameterMap` 对象，其每个子元素会被解析为 ParameterMapping 对象。 
2. `<resultMap>` 标签会被解析为 `ResultMap 对象`，其每个子元素会被解析为 ResultMapping 对象。
3. 每一个 `<select>、<insert>、<update>、<delete>` 标签均会被解析为 `MappedStatement` 对象，标签内的 sql 会被解析为 BoundSql 对象。
