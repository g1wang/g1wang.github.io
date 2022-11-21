
# SpringPrimer

## Spring IOC
### IOC & DI
#### Spring的核心 IOC控制反转
- inverse of control控制反转,采用ioc模式后，创建对象实例由容器和框架来完成，应用程序直接使用即可利用接口的方法实现解耦控制：对象应该调用哪个类的控制权反转：控制权应该由调用对象转移到容器或框架

#### Spring的核心 依赖注入（DI）
- IOC的另一种解释
- Dependency Injection ：由框架或容器将被调用类注入给调用对象，以此来解除调用对象和被调用类的依赖关系
  三种方式；构造函数注入，设置方法注入，接口注入
  
#### 注解
- @Resource(重要)
   可以指定特定名称,推荐使用
-   @Component @Service @Controller @Repository
     - 初始化的名字默认为类名首字母小写
     - 初始化，容器自动new某个类
     - 可以指定初始化bean的名字
- @Autowired
- @Scope
- @PostConstruct = init-method; 
- @PreDestroy = destroy-method;

## Spring AOP
- 原理：动态代理
- AspectJ 面向切面的开发框架，Spring 依赖 AspectJ来实现动态代理（一般使用AspectJ,不用spring自带的）
- 面向切面编程Aspect-Oriented-Programming,是对面向对象的思维方式的有力补充
- 好处：可以动态的添加和删除在切面上的逻辑而不影响原来的执行代码
- 两种方式：
  - 使用Annotation
  - 使用xml

### AOP 概念
- JoinPoint
- PointCut 切入点
- Aspect（切面类）
- Advice（Before、AfterReturning...）
- Target 被代理对象
- Weave织入
- 
### Annotation
#### 配置

    ```
    <?xml version="1.0" encoding="UTF-8"?>
    	<beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="
    	http://www.springframework.org/schema/beans 
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/aop
    	http://www.springframework.org/schema/aop/spring-aop.xsd">
    	<!-- bean definitions here -->
        <aop:aspectj-autoproxy />
    </beans>
    ```

####  常用的Annotation(见官方文档)
  - @Pointcut
  - @Before
  - @AfterReturning
  - @AfterThrowing
  - @After
  - @Around

示例：切面类
  ```
      @Aspect
      @Component
      public class LogInterceptor {
      @Before("execution(public * com.dao.impl.UserDAOImpl.addUser(com.model.User))")
      public void beforeMethod() {
      	System.out.println("start");
      }        
   ```

## Spring事务

### Spring 事务类型
编程式事务管理：这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。
声明式事务管理：这意味着你可以将业务代码和事务管理分离，你只需用注解和XML配置来管理事务。

### ACID
- Atomic原子性：事务是由一个或多个活动所组成的一个工作单元。原子性确保事务中的所有操作全部发生或者全部不发生。
- Consistent一致性：一旦事务完成，系统必须确保它所建模的业务处于一致的状态
- Isolated隔离机制：事务允许多个用户对象头的数据进行操作，每个用户的操作不会与其他用户纠缠在一起。
- Durable持久性：一旦事务完成，事务的结果应该持久化，这样就能从任何的系统崩溃中恢复过来


### 声明式事务
尽管Spring提供了多种声明式事务的机制，但是所有的方式都依赖这五个参数来控制如何管理事务策略。因此，如果要在Spring中声明事务策略，就要理解这些参数。(@Transactional)

#### 1. 隔离级别(isolation)
- ISOLATION_DEFAULT: 使用底层数据库预设的隔离层级
- ISOLATION_READ_COMMITTED: 允许事务读取其他并行的事务已经送出（Commit）的数据字段，可以防止Dirty read问题
- ISOLATION_READ_UNCOMMITTED: 允许事务读取其他并行的事务还没送出的数据，会发生Dirty、Nonrepeatable、Phantom read等问题
- ISOLATION_REPEATABLE_READ: 要求多次读取的数据必须相同，除非事务本身更新数据，可防止Dirty、Nonrepeatable read问题
- ISOLATION_SERIALIZABLE: 完整的隔离层级，可防止Dirty、Nonrepeatable、Phantom read等问题，会锁定对应的数据表格，因而有效率问题

#### 2. 传播行为(propagation)
- PROPAGATION_REQUIRED–支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
- PROPAGATION_SUPPORTS–支持当前事务，如果当前没有事务，就以非事务方式执行。
- PROPAGATION_MANDATORY–支持当前事务，如果当前没有事务，就抛出异常。
- PROPAGATION_REQUIRES_NEW–新建事务，如果当前存在事务，把当前事务挂起。
- PROPAGATION_NOT_SUPPORTED–以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER–以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_NESTED–如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。

#### 3. 只读(read-only)
如果事务只进行读取的动作，则可以利用底层数据库在只读操作时发生的一些最佳化动作，由于这个动作利用到数据库在只读的事务操作最佳化，因而必须在事务中才有效，也就是说要搭配传播行为PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED来设置。

#### 4. 事务超时(timeout)
有的事务操作可能延续很长一段的时间，事务本身可能关联到数据表的锁定，因而长时间的事务操作会有效率上的问题，对于过长的事务操作，考虑Roll back事务并要求重新操作，而不是无限时的等待事务完成。 可以设置事务超时期间，计时是从事务开始时，所以这个设置必须搭配传播行为PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED来设置。

#### 5. 回滚规则(rollback-for, no-rollback-for)
rollback-for指事务对于那些检查型异常应当回滚而不提交；no-rollback-for指事务对于那些异常应当继续运行而不回滚。默认情况下，Spring声明事务对所有的运行时异常都进行回滚。

### 事务失效情况
#### 1.数据库引擎不支持事务
以 MySQL 为例，其 MyISAM 引擎是不支持事务操作的，InnoDB 才是支持事务的引擎，一般要支持事务都会使用 InnoDB

#### 2.没有被 Spring 管理
```
如果此时把 `@Service` 注解注释掉，这个类就不会被加载成一个 Bean，那这个类就不会被 Spring 管理了，事务自然就失效了。
// @Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
}
```

#### 3.方法不是 public
`@Transactional` 只能用于 public 的方法上，否则事务会失效，如果要用在非 public 方法上，可以开启 `AspectJ` 代理模式

#### 4.自身调用问题
```
@Service
public class OrderServiceImpl implements OrderService {
    public void update(Order order) {
        updateOrder(order);
    }
    @Transactional
    public void updateOrder(Order order) {
        // update order
    }
}

发生了自身调用，就调该类自己的方法，而没有经过 Spring 的代理类，默认只有在外部调用事务才会生效
```

### 5.数据源没有配置事务管理器
```
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```

#### 6.不支持事务
```
Propagation.NOT_SUPPORTED： 表示不以事务运行，当前若存在事务则挂起

@Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void update(Order order) {
        updateOrder(order);
    }
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void updateOrder(Order order) {
        // update order
    }
}
```
#### 7.异常被吃了
```
把异常吃了，然后又不抛出来，事务怎么回滚?
// @Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
        }
    }
}
```

#### 8.异常类型错误
```
// @Service
public class OrderServiceImpl implements OrderService {
    @Transactional
    public void updateOrder(Order order) {
        try {
            // update order
        } catch {
            throw new Exception("更新错误");
        }
    }
}
```
这样事务也是不生效的，因为默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，如：
```
@Transactional(rollbackFor = Exception.class)
```