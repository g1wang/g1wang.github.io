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