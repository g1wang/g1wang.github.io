## Mybatis



## MyBatis问题汇总

### select语句根据属性查询出主键为null

我们对 id标签的理解是，它配置的字段为表的主键（联合主键时可以配置多个 id 标签），因为myBatis 中resultMap 只用于配置结果如何映射，并不知道这个表具体如何。 id 的唯一作用就是在嵌套的映射配置中判断数据是否相同，当配置id标签时， Mybatis只需要逐条比较所有数据中 id 标签配置的字段值是否相同即可。在配置嵌套结果查询时，配置 id 标签可以提高处理效率。

```
@Select({  
        "select * from user_info"  
})  
@Results({  
        @Result(property = "userId", column = "user_id")  
})  
List<UserEntity> getAll();
```