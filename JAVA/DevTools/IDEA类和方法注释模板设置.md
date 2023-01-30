### IDEA类和方法注释模板设置

#### 1.类的模板
File-->settings-->Editor-->File and Code Templates-->Files-->Class

```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end  
#parse("File Header.java")  
/**  
* @Description: TODO  
* @Author ${USER}  
* @Date ${DATE} ${TIME}  
*/  
public class ${NAME} {  
}
```

#### 2.方法注释模板
File-->Settings-->Editor-->Live Templates

```
快捷键： /**+Enter
```

##### 2.1新建组：命名为userDefine
添加Live Templates， abbreviation 设置为 *

##### 2.2 设置方法模板
```
*  
* @Author $user$  
* @Description //TODO $end$  
* @Date $time$ $date$  
* @Param $param$  
* @return $return$  
**/
```


##### 2.3设置模板的应用场景
点击模板页面最下方的警告，JAVA 全选
![[Pasted image 20220827105313.png]]
##### 2.4 Edit variables

![[Pasted image 20220827105506.png]]


