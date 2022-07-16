# Springboot全局事务配置及@Transactional注解失效场景

## 一.Springboot全局事务配置

### 1.什么是事务

事务：是数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作；这些操作作为一个整体一起向系统提交，要么都执行、要么都不执行；事务是一组不可再分割的操作集合（工作逻辑单元）。

事务的四大特性：

- 1）原子性 (atomicity）:强调事务的不可分割. 事务是数据库的逻辑工作单位，事务中包含的各操作要么都做，要么都不做

  > A,B为一个事务中对数据库的两个执行操作，B执行失败，根据事务原子性，A会回滚

- 2）一致性 （consistency）:事务的执行的前后数据的完整性保持一致. 事务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。因此当数据库只包含成功事务提交的结果时，就说数据库处于一致性状态。如果数据库系统 运行中发生故障，有些事务尚未完成就被迫中断，这些未完成事务对数据库所做的修改有一部分已写入物理数据库，这时数据库就处于一种不正确的状态，或者说是不一致的状态。

  > 甲（10元）给乙（10元）转10块钱，只会出现甲0，乙20和甲10和乙10两种情况

- 3） 隔离性 （isolation）:一个事务执行的过程中,不应该受到其他事务的干扰 一个事务的执行不能其它事务干扰。即一个事务内部的操作及使用的数据对其它并发事务是隔离的，并发执行的各个事务之间不能互相干扰。

  > A和B两个事务互不干扰

- 4） 持久性 （durability） :事务一旦结束,数据就持久到数据库 也称永久性。

  > 指一个事务一旦提交，它对数据库中的数据的改变就应该是永久性的。接下来的其它操作或故障不应该对其执行结果有任何影响。

Spring Boot(Spring)事务是通过aop(通知（Advice）、连接点（Joinpoint）、切入点（Pointcut)、切面（Aspect）、目标(Target)、代理(Proxy)、织入（Weaving）)切面编程来实现的,此时我们就可以对指定的包的service的方法进行事务控制

- 智云微服务配置事务类形式：

  ```java
  private static final String AOP_POINTCUT_EXPRESSION = "execution(* cn.com.egova..service.*Manager.*(..))";
  ```

- 智云V14纯xml文件形式：

  ```xml
    <aop:config>
           <aop:pointcut id="serviceAllOperation"
                expression="execution(* cn.com.egova..service.*Manager.*(..))" />
           <aop:advisor advice-ref="txAdvice"
                pointcut-ref="serviceAllOperation" order="3" />
    </aop:config>
  ```

### 2.为什么要使用全局事务

避免在每个service类或方法中加声明式注解,维护成本高,代码可读性不高

### 3.全局事务配置方式

- 在配置类或者启动类上添加注解@EnableTransactionManagement开启事务支持
- 创建全局事务配置切面类

```java
@Aspect
@Configuration
public class TransactionAdviceConfig {

    private static final String AOP_POINTCUT_EXPRESSION = "execution(* cn.com.egova..service.*Manager.*(..))";

    @Autowired
    private PlatformTransactionManager transactionManager;//事务管理器啊

    @Bean
    public TransactionInterceptor txAdvice() {

        /*事务管理规则*/
        RuleBasedTransactionAttribute TX_ATTR_REQUIRED = new RuleBasedTransactionAttribute();
        /* transactiondefinition 事务定义信息，定义事务的隔离级别，传播属性
        PROPAGATION_REQUIRED 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中*/
        TX_ATTR_REQUIRED.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

        RuleBasedTransactionAttribute TX_ATTR_REQUIRED_READONLY = new RuleBasedTransactionAttribute();
        /*设置当前事务是否为只读事务，true为只读*/
        TX_ATTR_REQUIRED_READONLY.setReadOnly(true);
        /* 非事务地执行，并挂起任何存在的事务*/
        RuleBasedTransactionAttribute TX_ATTR_NOT_SUPPORT = new RuleBasedTransactionAttribute();
        TX_ATTR_NOT_SUPPORT.setPropagationBehavior(TransactionDefinition.PROPAGATION_NOT_SUPPORTED);
        TX_ATTR_NOT_SUPPORT.setReadOnly(true);

        NameMatchTransactionAttributeSource source = new NameMatchTransactionAttributeSource();

        source.addTransactionalMethod("get*", TX_ATTR_REQUIRED_READONLY);

        source.addTransactionalMethod("query*", TX_ATTR_NOT_SUPPORT);
        source.addTransactionalMethod("find*", TX_ATTR_NOT_SUPPORT);
        source.addTransactionalMethod("list*", TX_ATTR_NOT_SUPPORT);
        source.addTransactionalMethod("count*", TX_ATTR_NOT_SUPPORT);
        source.addTransactionalMethod("is*", TX_ATTR_NOT_SUPPORT);
        source.addTransactionalMethod("*", TX_ATTR_REQUIRED);

        return new TransactionInterceptor(transactionManager, source);
    }

/*设置切面=切点pointcut+通知TxAdvice */
    @Bean
    public Advisor txAdviceAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression(AOP_POINTCUT_EXPRESSION);
        return new DefaultPointcutAdvisor(pointcut, txAdvice());
    }
}
```

- 事务传播类型

| 传播类型                  | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 支持一个已经存在的事务。如果没有事务，则开始一个新的事务     |
| PROPAGATION_SUPPORTS      | 支持一个已经存在的事务。如果没有事务，则以非事务方式运行     |
| PROPAGATION_MANDATORY     | 支持一个已经存在的事务。如果没有活动事务，则抛异常           |
| PROPAGATION_REQUIRES_NEW  | 始终开始新事务。如果活动事物已经存在，将其暂停               |
| PROPAGATION_NOT_SUPPORTED | 不支持活动事务的执行。始终以非事务方式执行，并暂停任何现有事务 |
| PROPAGATION_NEVER         | 即使存在活动事务，也始终以非事务方式执行。如果存在活动事物，抛出异常 |
| PROPAGATION_NESTED        | 如果存在活动事务，则在嵌套事务中运行。如果没有活动事务，则与PROPAGATION_REQUIRED相同 |

- 事务隔离级别

| 隔离类型                   | 脏读 | 不可重复读 | 幻读 | 描述                                                         |
| -------------------------- | ---- | ---------- | ---- | ------------------------------------------------------------ |
| ISOLATION_READ_UNCOMMITTED | √    | √          | √    | Read uncommitted读未提交，就是一个事务可以读取另一个未提交事务的数据。 |
| ISOLATION_READ_COMMITTED   | ×    | √          | √    | Read committed读提交，就是一个事务要等另一个事务提交后才能读取数据。解决了脏读，但不能解决不可重复读和幻读。 |
| ISOLATION_REPEATABLE_READ  | ×    | ×          | √    | Repeatable read重复读，就是在开始读取数据（事务开启）时，不再允许修改操作解决了不可重复读，但不能解决幻读。 |
| ISOLATION_SERIALIZABLE     | ×    | ×          | ×    | Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。 |

**√ 为会发生，×为不会发生**
ISOLATION_DEFAULT 使用数据库本身使用的隔离级别ORACLE（读已提交）MySQL（可重复读）
事务隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大

- **脏读**

| 时间 | tab1                                                         | tab2                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| T1   | set session transaction isolation level read uncommitted; start transaction; update account set balance =balance + 1000 where id = 1; select * from account where id = 1; |                                                              |
| T2   |                                                              | set session transaction isolation level read uncommitted; start transaction; select * from account where id = 1; |
| T3   | rollback                                                     |                                                              |
| T4   |                                                              | commit                                                       |
| T5   |                                                              | select * from account where id = 1;                          |

> 脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据
>
> 

**脏写**

| 时间 | tab1                                                         | tab2                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| T1   | set session transaction isolation level read uncommitted;<br/>start transaction;<br/>update account set balance = balance - 1000 where id = 1;<br/>update account set balance = balance + 1000 where id = 2; |                                                              |
| T2   |                                                              | set session transaction isolation level read uncommitted;<br/>start transaction;<br/>select balance from account where id = 2;<br/>update account set balance = balance - 1000 where id = 2; |
| T3   | rollback                                                     |                                                              |
| T4   |                                                              | commit                                                       |



- **不可重复读**

| 时间 | tab1                                                         | tab2                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| T1   | set session transaction isolation level read committed; start transaction; select * from account where id = 2; |                                                              |
| T2   |                                                              | set session transaction isolation level read committed; start transaction; update account set balance =balance + 1000 where id = 2; select * from account where id = 2; commit; |
| T3   | select * from account where id = 2; commit;                  |                                                              |

> 不可重复读是指在事务1内，读取了一个数据，事务1还没有结束时，事务2也访问了这个数据，修改了这个数据，并提交。紧接着，事务1又读这个数据。由于事务2的修改，那么事务1两次读到的数据可能是不一样的，因此称为是不可重复读。

可以在T2时间段客户端B修改完id=2的账户余额但没有commit的时候，在客户端A查询id=2的账户余额，发现账户余额为0，可以证明提交读这个隔离级别不会发生脏读。

- **幻读**

| 时间 | tab1                                                         | tab2                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| T1   | set session transaction isolation level repeatable read; start transaction; select count(*) from account where id <= 10; |                                                              |
| T2   |                                                              | set session transaction isolation level repeatable read; start transaction; insert into account (id, name, balance) values (3, "王五", 0); select count(*) from account where id <= 10; commit; |
| T3   | select count(*) from account where id <= 10; commit;         |                                                              |

> 幻读指的是当某个事务在读取某个范围内的记录时，另外一个事务又在该范围内插入了新的记录，当之前的事务再次读取该范围的记录时，会产生幻行。InnoDB存储引擎通过多版本并发控制（MVCC）解决了幻读的问题。
> 幻读和不可重复读的区别是，前者是一个范围，后者是本身

| 时间 | tab1                                                         | tab2                                                         |      |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| T1   | set session transaction isolation level repeatable read; start transaction; select count(*) from account where id = 3; |                                                              |      |
| T2   |                                                              | set session transaction isolation level repeatable read; start transaction; insert into account (id, name, balance) values (3, "王五", 0); commit; |      |
| T3   | insert into account (id, name, balance) values (3, "王五", 0); |                                                              |      |
| T4   | select count(*) from account where id = 3;                   |                                                              |      |
| T5   | rollback;                                                    |                                                              |      |



## 二.@Transactional注解失效场景

事务实现的两种方式：

- 全局事务配置
- @Transactional注解实现

### 1.应用在非 public 修饰的方法上

```java
protected TransactionAttribute computeTransactionAttribute(Methodmethod,
   Class<?> targetClass) {
       // Don't allow no-public methods as required.
       if (allowPublicMethodsOnly() &&!Modifier.isPublic(method.getModifiers())) {
       return null;
}
```

Modifier.isPublic会检查目标方法的修饰符是否为 public，不是 public则不会获取@Transactional 的属性配置信息。

注：protected、private 修饰的方法上使用 @Transactional 注解，虽然事务无效，但不会有任何报错，这是我们很容犯错的一点。

### 2.propagation 设置错误，导致注解失效

propagation 属性设置为PROPAGATION_SUPPORTS、PROPAGATION_NOT_SUPPORTED、PROPAGATION_NEVER这三种类别时，@Transactional 注解就不会产生效果

### 3.rollbackFor 设置错误，@Transactional 注解失效

Spring默认回滚事务分别为抛出了未检查unchecked异常（继承自 RuntimeException 的异常）和 Error两种情况，其他异常不会回滚，希望抛出其他异常Spring亦能回滚事务，需要指定rollbackFor属性

```java
//自定义的异常可以进行回滚,如设置错误，则不会回滚
@Transactional(propagation= Propagation.REQUIRED,rollbackFor=MyException.class
```

### 4.方法之间的互相调用导致@Transactional失效

```java
  //@Transactional
   @GetMapping("/user")
   private Integer A() throws Exception {
       User user = new User();
       user.setName("javaHuang");
       /**
        * B 插入字段为 topJavaer的数据
        */
       this.insertB();
       /**
        * A 插入字段为 2的数据
        */
       int insert = userMapper.insert(user);

       return insert;
  }

   @Transactional()
   public Integer insertB() throws Exception {
       User user = new User();
       user.setName("topJavaer");
       return userMapper.insert(user);
  }
```

一个类User，它的一个方法A，A再调用本类的方法B（不论方法B是用public还是private修饰），但方法A没有声明注解事务，而B方法有。则外部调用方法A之后，方法B的事务是不会起作用

### 5.异常被 catch捕获导致@Transactional失效

```java
@Transactional
    private Integer A() throws Exception {
        int insert = 0;
        try {
            CityInfoDict cityInfoDict = new CityInfoDict();
            cityInfoDict.setCityName("2");
            cityInfoDict.setParentCityId(2);
            /**
             * A 插入字段为 2的数据
             */
            insert = cityInfoDictMapper.insert(cityInfoDict);            
            /**
             * B 插入字段为 3的数据
             */
            b.insertB();        } catch (Exception e) {
            e.printStackTrace();        }    }
```

如果B方法内部抛了异常，而A方法此时try catch了B方法的异常，那这个事务就不能正常回滚，而是会报出异常

```
org.springframework.transaction.UnexpectedRollbackException: Transactionrolled back because it has been marked as rollback-only
```

当ServiceB中抛出了一个异常以后，ServiceB标识当前事务需要rollback。但是ServiceA中由于你手动的捕获这个异常并进行处理，ServiceA认为当前事务应该正常commit。此时就出现了前后不一致，也就是因为这样，抛出了前面的UnexpectedRollbackException异常。

spring的事务是在调用业务方法之前开始的，业务方法执行完毕之后才执行commit or rollback，事务是否执行取决于是否抛出runtime异常。如果抛出runtime exception 并在你的业务方法中没有catch到的话，事务会回滚。

在业务方法中一般不需要catch异常，如果非要catch一定要抛出throw new RuntimeException()，或者注解中指定抛异常类型@Transactional(rollbackFor=Exception.class)，否则会导致事务失效，数据commit造成数据不一致。

### 6.数据库引擎不支持事务

这种情况出现的概率并不高，事务能否生效数据库引擎是否支持事务是关键。常用的MySQL数据库默认使用支持事务的innodb引擎。一旦数据库引擎切换成不支持事务的myisam，那事务就从根本上失效了。