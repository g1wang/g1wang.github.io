# Redis

## Redis原理
### redis-cli
- 登录：./redis-cli -a 123456
### redis push/sub
- 发布：publish string-topic ping-pong
- 订阅：subscribe string-topic

## 客户端工具
- Another Redis Desktop Manager

## springboot-redis

### application.properties
```
spring.redis.database=0  
spring.redis.host=192.168.86.150  
spring.redis.port=6379  
spring.redis.password=123456  
spring.redis.lettuce.pool.max-active=8  
spring.redis.lettuce.pool.max-wait=-1  
spring.redis.lettuce.pool.max-idle=8  
spring.redis.lettuce.pool.min-idle=0
```

### push/sub
- RedisConfig:RedisMessageListenerContainer
```
package com.example.redissample.redis;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.cache.annotation.CachingConfigurerSupport;  
import org.springframework.cache.annotation.EnableCaching;  
import org.springframework.cache.interceptor.KeyGenerator;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.data.redis.connection.RedisConnectionFactory;  
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;  
import org.springframework.data.redis.listener.ChannelTopic;  
import org.springframework.data.redis.listener.RedisMessageListenerContainer;  
  
import java.lang.reflect.Method;  
  
/**  
 * @Description:  
 * @Author laboratory  
 * @Date 2023/1/16  
 */
@Configuration  
@EnableCaching  
public class RedisConfig extends CachingConfigurerSupport {  
  
    @Bean  
    public ConsumerRedisListener consumerRedis() {  
        return new ConsumerRedisListener();  
    }  
  
    @Bean  
    public RedisMessageListenerContainer container(RedisConnectionFactory factory, ConsumerRedisListener listener) {  
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();  
        container.setConnectionFactory(factory);  
        //订阅频道string-topic  这个container 可以添加多个 messageListener        container.addMessageListener(listener, new ChannelTopic("string-topic"));  
        return container;  
    }  
  
}
```

- MessageListener
```
package com.example.redissample.redis;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.data.redis.connection.Message;  
import org.springframework.data.redis.connection.MessageListener;  
import org.springframework.data.redis.core.StringRedisTemplate;  
  
/**  
 * @Description:  
 * @Author laboratory  
 * @Date 2023/1/29  
 */
 public class ConsumerRedisListener implements MessageListener {  
    @Autowired  
    private StringRedisTemplate stringRedisTemplate;  
    
    @Override  
    public void onMessage(Message message, byte[] pattern) {  
        doBusiness(message);  
    }  
    public void doBusiness(Message message) {  
        Object value = stringRedisTemplate.getValueSerializer().deserialize(message.getBody());  
        System.out.println("consumer message: " + String.valueOf(value));  
    }  
}
```

- test: push
```
@SpringBootTest  
class RedisSampleApplicationTests {  
  
    @Autowired  
    private StringRedisTemplate stringRedisTemplate;  
    @Test  
    void contextLoads() {  
        stringRedisTemplate.convertAndSend("string-topic","hello world");  
    }  
}
```


### 缓存

- RedisConfig 添加 cache 的配置类，使用 @EnableCaching 开启缓存
```
@Bean  
public KeyGenerator keyGenerator() {  
    return new KeyGenerator() {  
        @Override  
        public Object generate(Object target, Method method, Object... params) {  
            StringBuilder sb = new StringBuilder();  
            sb.append(target.getClass().getName());  
            sb.append(method.getName());  
            for (Object obj : params) {  
                sb.append(obj.toString());  
            }  
            return sb.toString();  
        }  
    };  
}
```

#### 普通缓存

- 自动根据方法生成缓存
```

package com.example.redissample.controller;  
  
import com.example.redissample.entity.User;  
import com.example.redissample.service.UserService;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
import javax.annotation.Resource;  
import javax.xml.bind.ValidationException;  
  
/**  
 * @Description:  
 * @Author laboratory  
 * @Date 2023/1/29  
 */@RestController  
@RequestMapping("/user")  
public class UserController {  
  
    @Resource  
    UserService userService;  
  
    /**  
     * @return com.example.redissample.entity.User  
     * @Author laboratory  
     * @Description http://127.0.0.1:8080/user?email=aa@123.com  
     * @Date 2023/1/29  
     * @Param [user]  
     **/    @GetMapping  
    public User getUser(String email) throws ValidationException {  
        return userService.getUser(email);  
    }  
  
}
```

```
package com.example.redissample.service;  
  
import com.example.redissample.entity.User;  
import org.springframework.cache.annotation.Cacheable;  
import org.springframework.stereotype.Service;  
  
import javax.xml.bind.ValidationException;  
  
/**  
 * @Description:  
 * @Author laboratory  
 * @Date 2023/1/29  
 */@Service  
public class UserService {  
  
    @Cacheable(value = "user-key")  
    public User getUser(String email) throws ValidationException {  
        User user ;  
        System.out.println("无缓存的时候调用");  
        if ("aa@123.com".equals(email)){  
            user = new User("aa@123.com","aaa","aa123");  
        }else throw new ValidationException("user not exist");  
        return user;  
    }  
}
```

#### 共享session
分布式系统中，Session 共享有很多的解决方案，其中托管到缓存中应该是最常用的方案之一。
Spring Session 提供了一套创建和管理 Servlet HttpSession 的方案。Spring Session 提供了集群 Session（Clustered Sessions）功能，默认采用外置的 Redis 来存储 Session 数据，以此来解决 Session 共享的问题。
- 引入依赖
```
<dependency>  
    <groupId>org.springframework.session</groupId>  
    <artifactId>spring-session-data-redis</artifactId>  
</dependency>
```
- Session 配置
```
@Configuration  
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30)  
public class SessionConfig {
}
```

- 测试
```
@RequestMapping("/uid")  
String uid(HttpSession session) {  
    UUID uid = (UUID) session.getAttribute("uid");  
    if (uid == null) {  
        uid = UUID.randomUUID();  
    }  
    session.setAttribute("uid", uid);  
    return session.getId();  
}
```