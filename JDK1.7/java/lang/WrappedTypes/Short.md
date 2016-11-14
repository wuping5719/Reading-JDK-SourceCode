* Short类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Short 类在对象中包装基本类型 short 的值。一个 Short 类型的对象只包含一个 short 类型的字段。

  &nbsp;&nbsp; 该类提供了多个方法，可以将 short 转换为 String，将 String 转换为 short，同时还提供了其他一些处理 short 时有用的常量和方法。
  
```java
  public final class Short extends Number implements Comparable<Short> {
    //
    public static final short   MIN_VALUE = -32768;
    
  }
```
