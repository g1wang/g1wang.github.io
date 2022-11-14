
# SpringPrimer
## Spring事务

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