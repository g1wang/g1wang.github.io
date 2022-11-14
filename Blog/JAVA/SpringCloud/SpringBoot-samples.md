# SpringBoot-samples

## Excel导出

### SXSSFWorkbook POI 
[项目地址](https://github.com/g1wang/springboot-samples/tree/master/excel-sample)

excel 导出单个sheet 限制100万条

#### poi tempfile
生成excel时，SXSSFWorkbook POI 临时文件夹“poifiles”问题处理
- code
```
@Component  
public class ExcelConfig {  
  
    @Value("${application.tmp.path}")  
    private String applicationTmpPath;  
  
    /**  
     * 设置使用SXSSFWorkbook对象导出excel报表时，TempFile使用的临时目录，代替{java.io.tmpdir}  
     */    @PostConstruct  
    public void setExcelSXSSFWorkbookTmpPath() {  
        String excelSXSSFWorkbookTmpPath = applicationTmpPath + "/poifiles";  
        File dir = new File(excelSXSSFWorkbookTmpPath);  
        if (!dir.exists()) {  
            dir.mkdirs();  
        }  
        TempFile.setTempFileCreationStrategy(new DefaultTempFileCreationStrategy(dir));  
    }  
}

```
- 配置文件
```
application:  
  tmp:  
    path: ./excel-sample/tmp/poifile
```


## 多数据源配置

Springboot2.0为例
### 依赖导入
```
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus</artifactId>  
    <version>3.5.2</version>  
</dependency>
<dependency>  
    <groupId>org.mybatis.spring.boot</groupId>  
    <artifactId>mybatis-spring-boot-starter</artifactId>  
    <version>2.2.2</version>  
</dependency>  
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <scope>runtime</scope>  
</dependency>
```

### 数据源配置
#### 数据源配置文件
```
spring:  
  datasource:  
    ds1:  
      driver-class-name: com.mysql.cj.jdbc.Driver  
      jdbc-url: jdbc:mysql://192.168.86.90:3306/recsys?tinyInt1isBit=false&useUnicode=true&characterEncoding=UTF-8  
      username: devlop  
      password: qwer1234  
    ds2:  
      driver-class-name: com.mysql.cj.jdbc.Driver  
      jdbc-url: jdbc:mysql://192.168.86.90:3306/recsys1?tinyInt1isBit=false&useUnicode=true&characterEncoding=UTF-8  
      username: devlop  
      password: qwer1234
```

#### 数据源1
```
package com.example.mutildatasource.configuration;  
  
import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;  
import org.apache.ibatis.session.SqlSessionFactory;  
import org.mybatis.spring.SqlSessionTemplate;  
import org.mybatis.spring.annotation.MapperScan;  
import org.springframework.beans.factory.annotation.Qualifier;  
import org.springframework.boot.context.properties.ConfigurationProperties;  
import org.springframework.boot.jdbc.DataSourceBuilder;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.context.annotation.Primary;  
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;  
import org.springframework.jdbc.datasource.DataSourceTransactionManager;  
  
import javax.sql.DataSource;  
  
/**  
 * @Description:  
 * @Author laboratory  
 * @Date 2022/11/12 10:04  
 */@Configuration  
@MapperScan(basePackages ="com.example.mutildatasource.ds1.**.mapper",  
        sqlSessionTemplateRef = "ds1SqlSessionTemplate")  
public class MybatisPlusConfig4ds1 {  
  
    @Bean(name = "dataSource1")  
    @ConfigurationProperties(prefix = "spring.datasource.ds1")  
    @Primary  
    public DataSource dataSource() {  
        return DataSourceBuilder.create().build();  
    }  
  
    //主数据源 ds1数据源  
    @Primary  
    @Bean("ds1SqlSessionFactory")  
    public SqlSessionFactory ds1SqlSessionFactory(@Qualifier("dataSource1") DataSource dataSource) throws Exception {  
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();  
        sqlSessionFactory.setDataSource(dataSource);  
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().  
                getResources("classpath*:com/example/mutildatasource/ds1/**/*.xml"));  
        return sqlSessionFactory.getObject();  
    }  
  
    @Primary  
    @Bean(name = "ds1TransactionManager")  
    public DataSourceTransactionManager ds1TransactionManager(@Qualifier("dataSource1") DataSource dataSource) {  
        return new DataSourceTransactionManager(dataSource);  
    }  
  
    @Primary  
    @Bean(name = "ds1SqlSessionTemplate")  
    public SqlSessionTemplate ds1SqlSessionTemplate(@Qualifier("ds1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {  
        return new SqlSessionTemplate(sqlSessionFactory);  
    }  
}
```

#### 数据源2
```
package com.example.mutildatasource.configuration;  
  
import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;  
import org.apache.ibatis.session.SqlSessionFactory;  
import org.mybatis.spring.SqlSessionTemplate;  
import org.mybatis.spring.annotation.MapperScan;  
import org.springframework.beans.factory.annotation.Qualifier;  
import org.springframework.boot.context.properties.ConfigurationProperties;  
import org.springframework.boot.jdbc.DataSourceBuilder;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.context.annotation.Primary;  
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;  
import org.springframework.jdbc.datasource.DataSourceTransactionManager;  
  
import javax.sql.DataSource;  
  
/**  
 * @Description:  
 * @Author laboratory  
 * @Date 2022/11/12 10:04  
 */@Configuration  
@MapperScan(basePackages ="com.example.mutildatasource.ds2.**.mapper",  
        sqlSessionTemplateRef = "ds2SqlSessionTemplate")  
public class MybatisPlusConfig4ds2 {  
  
    @Bean(name = "dataSource2")  
    @ConfigurationProperties(prefix = "spring.datasource.ds2")  
    public DataSource dataSource() {  
        return DataSourceBuilder.create().build();  
    }  
  
    //主数据源 ds1数据源  
  
    @Bean("ds2SqlSessionFactory")  
    public SqlSessionFactory ds1SqlSessionFactory(@Qualifier("dataSource2") DataSource dataSource) throws Exception {  
        MybatisSqlSessionFactoryBean sqlSessionFactory = new MybatisSqlSessionFactoryBean();  
        sqlSessionFactory.setDataSource(dataSource);  
        sqlSessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver().  
                getResources("classpath*:com/example/mutildatasource/ds2/**/*.xml"));  
        return sqlSessionFactory.getObject();  
    }  
  
    @Bean(name = "ds2TransactionManager")  
    public DataSourceTransactionManager ds1TransactionManager(@Qualifier("dataSource2") DataSource dataSource) {  
        return new DataSourceTransactionManager(dataSource);  
    }  
  
    @Bean(name = "ds2SqlSessionTemplate")  
    public SqlSessionTemplate ds1SqlSessionTemplate(@Qualifier("ds2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {  
        return new SqlSessionTemplate(sqlSessionFactory);  
    }  
}
```

#### 注意

- `@ConfigurationProperties(prefix = "spring.datasource.ds1")`：使用spring.datasource.ds1开头的配置。
- @Qualifier：指定数据源名称，与Bean中的name属性原理相同，主要是为了确保注入成功；
- @Primary：声明这是一个主数据源（默认数据源），多数据源配置时**必不可少。**

#### 连接池
```
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid</artifactId>
</dependency>

spring.datasource.datasource2.initialSize=3 # 根据自己情况设置
spring.datasource.datasource2.minIdle=3
spring.datasource.datasource2.maxActive=20


/**
 * Mybatis  第二个ds2数据源配置
 * 多数据源配置依赖数据源配置
 * @see  DataSourceConfig
 */
@Configuration
@MapperScan(basePackages ="com.web.ds2.**.dao", sqlSessionTemplateRef  = "ds2SqlSessionTemplate")
public class MybatisPlusConfig4ds2 {

    @Value("${spring.datasource.datasource2.jdbc-url}")
    private String url;
    @Value("${spring.datasource.datasource2.driver-class-name}")
    private String driverClassName;
    @Value("${spring.datasource.datasource2.username}")
    private String username;
    @Value("${spring.datasource.datasource2.password}")
    private String password;
    @Value("${spring.datasource.datasource2.initialSize}")
    private int initialSize;
    @Value("${spring.datasource.datasource2.minIdle}")
    private int minIdle;
    @Value("${spring.datasource.datasource2.maxActive}")
    private int maxActive;

    @Bean(name = "dataSource2")
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setInitialSize(initialSize);
        dataSource.setMinIdle(minIdle);
        dataSource.setMaxActive(maxActive);
        return dataSource;
    }

    //...

}
```

### issue
#### jdbcUrl is required with driverClassName
主要原因是在1.0 配置数据源的过程中主要是写成：spring.datasource.url 和spring.datasource.driverClassName。
而在2.0升级之后需要变更成：spring.datasource.jdbc-url和spring.datasource.driver-class-name即可解决！