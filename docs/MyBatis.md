### 1、基本配置

> **MyBatis工具类**

> ```java
> /**
>  * 读取配置文件
>  * 搭建会话工厂
> */
> String resource = "mybatis-config.xml";
> 
> InputStream inputStream = Resources.getResourceAsStream(resource);
> 
> sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
> ```

> **数据库连接配置参数 db.properties**

> ```properties
> driver=com.mysql.jdbc.Driver
> url=jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf-8&useSSL=false
> username=root
> password=root
> ```

> MyBatis基本配置

> ```xml
> <?xml version="1.0" encoding="UTF-8" ?>
> <!DOCTYPE configuration
>         PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
>         "http://mybatis.org/dtd/mybatis-3-config.dtd">
> <configuration>
> 
> 
> <!--标签顺序-->
> <!--    properties?,
>         settings?,
>         typeAliases?,
>         typeHandlers?,
>         objectFactory?,
>         objectWrapperFactory?,
>         reflectorFactory?,
>         plugins?,
>         environments?,
>         databaseIdProvider?,
>         mappers?
>         -->
> 
> <!--    外部文件优先内部文件-->
> <!--    引入外部文件-->
>     <properties resource="db.properties">
>         
> <!--        内部文件-->
> <!--        <property name="driver" value="com.mysql.jdbc.Driver"/>-->
> <!--        <property name="url" value="jdbc:mysql://localhost:3306/demo?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"/>-->
> <!--        <property name="username" value="root"/>-->
> <!--        <property name="password" value="root"/>-->
>     </properties>
> 
> 
>     <settings>
> <!--        日志配置-->
> <!--        标准日志输出-->
> <!--        <setting name="logImpl" value="STDOUT_LOGGING"/>-->
>         <setting name="logImpl" value="LOG4J"/>
>     </settings>
> 
> 
> 
> 
> <!--    别名-->
>     <typeAliases>
> <!--    实体类别名-->
> <!--        <typeAlias type="com.demo.pojo.User" alias="user"></typeAlias>-->
> <!--     包扫描 直接使用类名访问;如果需要自定别名，在实体类上面添加注解@alias（“user”）-->
>         <package name="com.demo.pojo"/>
>     </typeAliases>
> 
>     <environments default="development">
>         <environment id="development">
>             <transactionManager type="JDBC"/>
>             <dataSource type="POOLED">
>                 <property name="driver" value="${driver}"/>
>                 <property name="url" value="${url}"/>
>                 <property name="username" value="${username}"/>
>                 <property name="password" value="${password}"/>
>             </dataSource>
>         </environment>
>     </environments>
> 
> 
> <!--    映射器-->
>     <mappers>
> <!--        [推荐使用]方式一 任意使用,不受任何限制-->
>         <mapper resource="UserMapper.xml"/>
> <!--        方式二-->
> <!--        方式二和方式三必须同包同名-->
> <!--        <mapper class="com.demo.dao.UserDao"/>-->
> <!--        方式三-->
> <!--        <package name="com.demo.dao"/>-->
>     </mappers>
> 
> </configuration>
> 
> ```

> **mapper文件**

> ```xml
> <?xml version="1.0" encoding="UTF-8" ?>
> <!DOCTYPE mapper
>         PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
>         "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
> 
> <!--    namespace绑定一个对应的Dao/Mapper接口-->
> <mapper namespace="com.demo.dao.UserDao">
> 
> <!--    id对应的namespace中的方法名-->
> <!--    resultType ：sql语句的返回值-->
> <!--    parameterType：参数类型-->
> <!--    map-->
> 
> 
> 
> <!--    resultMap 结果集映射-->
>     <resultMap id="UserMap" type="User">
> <!--        column 数据库中的字段,property实体类中的属性-->
> <!--        此项目实体类中的属性名已经和数据库的字段完全对应,可以不用设置-->
> <!--        <result column="id" property="id"/>-->
> <!--        <result column="username" property="username"/>-->
> <!--        <result column="password" property="password"/>-->
>     </resultMap>
> 
> 
> 
> 
>     <select id="findUserById" parameterType="int"  resultMap="UserMap">
>         select * from user where `id`=#{id};
>     </select>
> 
>     <!--    模糊查询-->
>     <!--    "%"#{username}"%" 预处理,提前规范好,防止SQL注入-->
>     <select id="getUserLike" resultType="User">
>         select * from user where username like "%"#{username}"%";
>     </select>
> 
>     <!--    分页查询-->
>     <select id="getUserByLimit" parameterType="Map" resultType="User">
>         select *
>         from user limit #{startIndex},#{pageSize} ;
>     </select>
> 
>     <!--    RowBounds分页查询-->
>     <select id="getUserByRowBounds" resultMap="UserMap">
>         select * from user;
>     </select>
> 
> 
>     <select id="getUserList" resultType="User">
>         select * from user;
>     </select>
> 
> 
>     <insert id="insertUser" parameterType="com.demo.pojo.User">
>         insert into user (`username`,`password`)value(#{username},#{password});
>     </insert>
> 
> <!--    map-->
>     <insert id="addUser" parameterType="map">
>         insert into user (`username`,`password`)value(#{username},#{password});
>     </insert>
> 
>     <update id="updateUserPassword" parameterType="User">
>         update user set `password` = #{password} where `id` =#{id};
>     </update>
> 
>     <delete id="deleteUserById" parameterType="int">
>         delete from user where `id` =#{id};
>     </delete>
> 
> </mapper>
> ```

