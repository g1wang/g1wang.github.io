### 引用devtools依赖

```
<dependency>  
  <groupId>
    org.springframework.boot
  </groupId>
  <artifactId>
    spring-boot-devtools
  </artifactId>
  <optional>
    true
  </optional>
</dependency>
```

### 自定义配置热部署
```
# 热部署开关，false即不启用热部署
spring.devtools.restart.enabled: true

# 指定热部署的目录
#spring.devtools.restart.additional-paths: src/main/java

# 指定目录不更新
spring.devtools.restart.exclude: test/**

```

### Intellij Idea修改
#### 勾上自动编译
```
File > Settings > Compiler > Build Project automatically
```

#### 注册
```
ctrl + shift + alt + / > Registry > 勾选Compiler autoMake allow when app running
```

### 注意事项

1、生产环境devtools将被禁用，如java -jar方式或者自定义的类加载器等都会识别为生产环境。

2、打包应用默认不会包含devtools，除非你禁用SpringBoot Maven插件的 `excludeDevtools`属性