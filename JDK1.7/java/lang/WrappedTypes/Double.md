* Double类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Double 类在对象中包装一个基本类型 double 的值。每个 Double 类型的对象都包含一个 double 类型的字段。

  &nbsp;&nbsp; 该类还提供多个方法，可以将 double 转换为 String，将 String 转换为 double，也提供了其他一些处理 double 时有用的常量和方法。
  
```java
  public final class Double extends Number implements Comparable<Double> {
    //保存 double 类型的正无穷大值的常量。
    //它等于 Double.longBitsToDouble(0x7ff0000000000000L) 返回的值。
    public static final double POSITIVE_INFINITY = 1.0 / 0.0;
    
    //保存 double 类型的负无穷大值的常量。
    //它等于 Double.longBitsToDouble(0xfff0000000000000L) 返回的值。
    public static final double NEGATIVE_INFINITY = -1.0 / 0.0;
    
    //保存 double 类型的 NaN 值的常量。
    //它等于 Double.longBitsToDouble(0x7ff8000000000000L) 返回的值。
    public static final double NaN = 0.0d / 0.0;
    
    //保存 double 类型的最大正有限值的常量，最大正有限值为 (2-2^-52)·2^1023。
    //它等于十六进制的浮点字面值 0x1.fffffffffffffP+1023，
    //也等于 Double.longBitsToDouble(0x7fefffffffffffffL)。
    public static final double MAX_VALUE = 0x1.fffffffffffffP+1023; // 1.7976931348623157e+308
    
    //保存 double 类型的最小正标准值的常量，最小正标准值为 2^-1022。
    //它等于十六进制的浮点字面值 0x1.0p-1022，也等于 Double.longBitsToDouble(0x0010000000000000L)。
    public static final double MIN_NORMAL = 0x1.0p-1022; // 2.2250738585072014E-308
    
    //保存 double 类型的最小正非零值的常量，最小正非零值为 2^-1074。
    //它等于十六进制的浮点字面值 0x0.0000000000001P-1022，也等于 Double.longBitsToDouble(0x1L)。
    public static final double MIN_VALUE = 0x0.0000000000001P-1022; // 4.9e-324
    
    //有限 double 变量可能具有的最大指数。它等于 Math.getExponent(Double.MAX_VALUE) 返回的值。
    public static final int MAX_EXPONENT = 1023;
    
    //标准化 double 变量可能具有的最小指数。它等于 Math.getExponent(Double.MIN_NORMAL) 返回的值。
    public static final int MIN_EXPONENT = -1022;
    
    //用于表示 double 值的位数。
    public static final int SIZE = 64;
    
    //表示基本类型 double 的 Class 实例。
    public static final Class<Double> TYPE = (Class<Double>) Class.getPrimitiveClass("double");
  }
```
