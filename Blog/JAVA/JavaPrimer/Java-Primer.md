# Java-Primer

## java 运算符
1 位异或运算（^）
运算规则是：两个数转为二进制，然后从高位开始比较，如果相同则为0，不相同则为1。
比如：8^11.
8转为二进制是1000，11转为二进制是1011.从高位开始比较得到的是：0011.然后二进制转为十进制，就是Integer.parseInt("0011",2)=3;

2 位与运算符（&）
运算规则：两个数都转为二进制，然后从高位开始比较，如果两个数都为1则为1，否则为0。
比如：129&128.
129转换成二进制就是10000001，128转换成二进制就是10000000。从高位开始比较得到，得到10000000，即128.

3 位或运算符（|）
运算规则：两个数都转为二进制，然后从高位开始比较，两个数只要有一个为1则为1，否则就为0。
比如：129|128.
129转换成二进制就是10000001，128转换成二进制就是10000000。从高位开始比较得到，得到10000001，即129.

4 位非运算符（~）
运算规则：如果位为0，结果是1，如果位为1，结果是0.
比如：~37
在Java中，所有数据的表示方法都是以补码的形式表示，如果没有特殊说明，Java中的数据类型默认是int,int数据类型的长度是8位，一位是四个字节，就是32字节，32bit.
8转为二进制是100101.
补码后为： 00000000 00000000 00000000 00100101
取反为：    11111111 11111111 11111111 11011010
因为高位是1，所以原码为负数，负数的补码是其绝对值的原码取反，末尾再加1。
因此，我们可将这个二进制数的补码进行还原： 首先，末尾减1得反码：11111111 11111111 11111111 11011001 其次，将各位取反得原码：
00000000 00000000 00000000 00100110，此时二进制转原码为38
所以~37 = -38.

## Enum

### 1.避免使用 ordinal

```
public enum TrivelStyle{
            easy(0),funny(1),hot(2);
            private final int num ;
            private TrivelStyle(int num) {
                  this.num = num;
            }
            public int styleNum(){
                  return num;
            }
      }
```

### 2.根据code获取value
```
public enum OrderStatus {  
    /*订单状态:  
     0 未支付,  
    1 已支付,  
    2 支付失败,  
    3 人工退款,  
    4 一键退款,  
    5 自动退款  
     */    
    UNPAY("0", "未支付"),  
    PAID("1", "已支付"),  
    FAILED("2", "支付失败"),  
    MANUAL_REFUND("3", "人工退款"),  
    ONLINE_REFUND("4", "一键退款"),  
    AUTO_REFUND("5", "自动退款");  
  
    private String code;  
    private String name;  
  
    OrderStatus(String code, String name) {  
        this.code = code;  
        this.name = name;  
    }  
    public String getCode() {  
        return code;  
    }  
    public String getName() {  
        return name;  
    }  
  
    public static String getNameByCode(String code) {  
        String name = "";  
        for (OrderStatus orderStatus : OrderStatus.values()) {  
            if (orderStatus.getCode().equals(code)) {  
                name = orderStatus.getName();  
            }  
        }  
        return name;  
    }  
}
```

### 3.EnumMap
```
public enum Phase{

            SOLID,LIQUID,GAS;
            public enum Transition{
                  MELT(SOLID,LIQUID),FREEZE(LIQUID,SOLID),
                  BOIL(LIQUID,GAS),CONDENSE(GAS,LIQUID),
                  SUBLIME(SOLID,GAS),DEPOSIT(GAS,SOLID);
                  private final Phase src;
                  private final Phase dst;
                  Transition(Phase src,Phase dst){
                        this.src = src;
                        this.dst = dst;

                  }
                  //init map
                  private static final  Map<Phase, Map<Phase, Transition>> m =
                         new EnumMap<Phase, Map<Phase, Transition>>(Phase.class);
                  static{
                        for (Phase p : Phase.values()) {
                              m.put(p, new EnumMap<Phase, Transition>(Phase.class));
                        }
                        for (Transition t : Transition.values()) {
                              m.get(t.src).put(t.dst, t);
                        }
                  }
                  public static Transition from(Phase src, Phase dst){
                        return m.get(src).get(dst);
                  }
            }
      }

     //System.out.println("phase:"+Phase.Transition.from(Phase.SOLID, Phase.GAS));
```

## Java 序列化

### 什么是序列化
Java序列化是指把Java对象保存为二进制字节码的过程，Java反序列化是指把二进制码重新转换成Java对象的过程。

### 为什么需要序列化
第一种情况是：一般情况下Java对象的声明周期都比Java虚拟机的要短，实际应用中我们希望在JVM停止运行之后能够持久化指定的对象，这时候就需要把对象进行序列化之后保存。
第二种情况是：需要把Java对象通过网络进行传输的时候。因为数据只能够以二进制的形式在网络中进行传输，因此当把对象通过网络发送出去之前需要先序列化成二进制数据，在接收端读到二进制数据之后反序列化成Java对象。

### 如何序列化
调用ObjectOutputStream.writeObject()和ObjectInputStream.readObject()
```
public static void main(String[] args) throws Exception {
FileOutputStream fos = new FileOutputStream("temp.out");
ObjectOutputStream oos = new ObjectOutputStream(fos);
TestObject testObject = new TestObject();
oos.writeObject(testObject);
oos.flush();
oos.close();

FileInputStream fis = new FileInputStream("temp.out");
ObjectInputStream ois = new ObjectInputStream(fis);
TestObject deTest = (TestObject) ois.readObject();
System.out.println(deTest.testValue);
System.out.println(deTest.parentValue);
System.out.println(deTest.innerObject.innerValue);

}
```

## JAVA反射

```
package com.stars.javaz.reflect.test;

import com.stars.javaz.reflect.base.Cat;
import com.stars.javaz.reflect.base.service.CatService;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;


/**
* @author CodeX
* @version 1.0
* @date 2021/5/28
*/

public class TestCatService {

    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException, NoSuchFieldException {

        //获取类
        //两种方法
        //Class cls = Class.forName("com.stars.javaz.reflect.base.service.CatService");

        Class cls = CatService.class;

        //获取构造方法
        //Constructor[] constructors = cls.getConstructors();
        Class[] params = {};
        Constructor constructor =cls.getConstructor(params);
        //构造对象
        Object obj = constructor.newInstance();

        //获取public method
        Method method = cls.getMethod("createCat", String.class);
        method.invoke(obj,"open cat");
        Cat pblCat = (Cat) method.invoke(obj,"pub cat");
        //获取私有方法
        Method priMtd = cls.getDeclaredMethod("create", String.class);
        //解除私有
        priMtd.setAccessible(true);

        //私有属性
        Field f = cls.getDeclaredField("name");
        f.setAccessible(true);
        f.set(obj,"yx");
        System.out.println(f.get(obj));
          //setAccessible 只对当前方法有效
//        Method priMtd2 = cls.getDeclaredMethod("create", String.class);
//        priMtd2.invoke(obj,"pri cat");
    }
}
```

```
package com.stars.javaz.reflect.base.service;
import com.stars.javaz.reflect.base.Cat;
public interface ICatService {
    public Cat createCat(String name);
}

package com.stars.javaz.reflect.base.service;
import com.stars.javaz.reflect.base.Cat;

/**
*
* @author CodeX
* @version 1.0
* @date 2021/5/28
*/

public class CatService implements ICatService{

    private String name;
    public Cat createCat(String name) {
        System.out.println("createCat:" + name);
        return new Cat(name);
    }
    private Cat create(String name) {
        System.out.println("create:" + name);
        return new Cat(name);
    }
}
```

```
package com.stars.javaz.reflect.test;

import com.stars.javaz.reflect.base.service.CatService;
import com.stars.javaz.reflect.base.service.ICatService;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;


/**
* @author CodeX
* @version 1.0
* @date 2021/5/28
*/

public class DynamicProxyTest {

    public static void main(String[] args) {
        //代理对象
        ICatService catService = new CatService();
        CatServiceHandler catHandler = new CatServiceHandler(catService);
        ICatService catServiceProxy = (ICatService) Proxy.newProxyInstance(ICatService.class.getClassLoader(),
                new Class[]{ICatService.class}, catHandler);
        System.out.println("--- before");
        String name ="miao";
        catServiceProxy.createCat(name);
        System.out.println("--- after");
    }
}

class CatServiceHandler implements InvocationHandler {

    ICatService target;
    public CatServiceHandler(ICatService target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
        return method.invoke( target,objects);
    }
}
```

## Java的泛型解析

TODO
