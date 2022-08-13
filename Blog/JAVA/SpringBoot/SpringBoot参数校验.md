## springboot参数校验

### 注解

- @Validated注解是Spring基于 @Valid注解的进一步封装,并提供比如分组,分组顺序的高级功能使用位置

- 不同:
  - @Valid注解  可以使用在方法,构造函数,方法参数和成员属性上
  - @Validated注解 :可以用在类型,方法和方法参数上. 但是不能用在成员属性上

### 常用的校验注解

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