# SpringBoot-Primer

### SpringBoot参数校验

#### 注解

- @Validated注解是Spring基于 @Valid注解的进一步封装,并提供比如分组,分组顺序的高级功能使用位置

- 不同:
  - @Valid注解  可以使用在方法,构造函数,方法参数和成员属性上
  - @Validated注解 :可以用在类型,方法和方法参数上. 但是不能用在成员属性上

#### 常用的校验注解

- @Null：被注释的元素必须为null
- @NotNull：被注释的元素不能为null
- @AssertTrue：该字段只能为true@AssertFalse：该字段的值只能为false
- @Min(value)：被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- @Max(value)：被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- @DecimalMin(“value”)：被注释的元素必须是一个数字，验证小数的最小值
- @DecimalMax(“value”)：被注释的元素必须是一个数字，验证小数的最大值
- @Size(max,min)：查该字段的size是否在min和max之间，可以是字符串、数组、集合、Map等
- @Past：被注释的元素必须是一个过去的日期
- @Future：被注释的元素必须是一个将来的日期
- @Pattern(regexp = “[abc]”)：被注释的元素必须符合指定的正则表达式。
- @Email：被注释的元素必须是电子邮件地址
- @Length(max=5,min=1,message=“长度在1~5”)：检查所属的字段的长度是否在min和max之间，只能用于字符串@NotEmpty：被注释的字符串必须非空
- @Range：被注释的元素必须在合适的范围内
- @NotBlank：不能为空，检查时会将空格忽略
- @NotEmpty：不能为空，这里的空是指空字符串


### SpringBoot定时器

#### Scheduled
##### 基本配置
- 启动类添加注解 @EnableScheduling
```
#main
@EnableScheduling  
public class ScheduledSampleApplication {

```

- 定时任务使用 @Scheduled(cron = "${scheduled.printCron}")  
```
@Component  
public class PrintTask {  
  
    @Scheduled(cron = "${scheduled.printCron}")  
    private void doTask(){  
        System.out.println("print:"+new Date());  
    }  
}
```
- 配置文件 application.yml
```
scheduled:  
  printCron: "0 35 14 * * ?"
```
##### 是否启用定时任务 
- 定时任务添加 @ConditionalOnProperty
```
@Component
@ConditionalOnProperty(prefix = "scheduled", name = "enabled", havingValue = "true")
public class PrintTask {  
  
    @Scheduled(cron = "${scheduled.printCron}")  
    private void doTask(){  
        System.out.println("print:"+new Date());  
    }  
}
```
- 配置文件
```
scheduled:  
  printCron: "0 12 17 * * ?"  
  enabled: false
```

### SpringBoot Property 读取配置

####  读取application文件
- application配置文件
```
info.title=Home
```
- 方式一：@Value注解读取方式
```
@Component  
public class InfoConfigValue {  
    @Value("${info.title}")  
    private String title;  
    
```

- 方式二：@ConfigurationProperties注解读取方式
```
@Component  
@ConfigurationProperties(prefix = "info")  
public class InfoConfigProp {  
    private String title;
```

#### 读取指定文件
- 方式一： @PropertySource+@Value注解读取方式
```
@Component  
@PropertySource(value = {"config/db-config.properties"})  
public class DBConfigValue {  
  
    @Value("${db.username}")  
    private String username;
```

- 方式二：@PropertySource+@ConfigurationProperties注解读取方式
```
@Component  
@ConfigurationProperties(prefix = "db")  
@PropertySource(value = {"config/db-config.properties"})  
public class DBConfigProp {  
  
    private String username;
```

#### Profile不同环境配置
假如有开发、测试、生产三个不同的环境，需要定义三个不同环境下的配置。

##### 基于properties文件类型
你可以另外建立3个环境下的配置文件：
applcation.properties
application-dev.properties
application-test.properties
application-prod.properties
然后在applcation.properties文件中指定当前的环境spring.profiles.active=test,这时候读取的就是application-test.properties文件。

##### 基于yml文件类型
只需要一个applcation.yml文件就能搞定，推荐此方式。
```
info:  
  title: "prop-sample"  
  company: "stars"  
  
spring:  
  profiles:  
    active: prod  
      
---  
spring:  
  config:  
    activate:  
      on-profile: dev  
server:  
  port:  
  8080  
---  
spring:  
  config:  
    activate:  
      on-profile: test  
server:  
  port: 8081  
  
---  
spring:  
  config:  
    activate:  
      on-profile: prod  
  
  profiles:  
    include:  
      - proddb  
      - prodmq  
server:  
  port: 8082  
---  
  
spring:  
  config:  
    activate:  
      on-profile: proddb  
db:  
  name: mysql  
  
---  
spring:  
  config:  
    activate:  
      on-profile: prodmq  
mq:  
  address: localhost
```

##### jar运行方式
```
java -jar xx.jar --spring.profiles.active=prod
```