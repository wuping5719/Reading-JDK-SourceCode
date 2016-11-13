* Float类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Float 类在对象中包装一个基本类型 float 的值。Float 类型的对象包含一个 float 类型的字段。

  &nbsp;&nbsp; 此外，此类提供了几种方法，可将 float 类型与 String 类型互相转换，还提供了处理 float 类型时非常有用的其他一些常量和方法。
  
```java
  public final class Float extends Number implements Comparable<Float> {
    //保存 float 类型的正无穷大值的常量。它等于 Float.intBitsToFloat(0x7f800000) 返回的值。
    public static final float POSITIVE_INFINITY = 1.0f / 0.0f;
    
    //保存 float 类型的负无穷大值的常量。它等于 Float.intBitsToFloat(0xff800000) 返回的值。
    public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;
    
    //保存 float 类型的非数字 (NaN) 值的常量。它等于 Float.intBitsToFloat(0x7fc00000) 返回的值。
    public static final float NaN = 0.0f / 0.0f;
    
    //保存 float 类型的最大正有限值的常量，即 (2-2^-23)·2^127。
    //它等于十六进制的浮点字面值 0x1.fffffeP+127f，也等于 Float.intBitsToFloat(0x7f7fffff)。
    public static final float MAX_VALUE = 0x1.fffffeP+127f;  // 3.4028235e+38f
    
    //保存 float 类型数据的最小正标准值的常量，即 2^-126。
    //它等于十六进制的浮点字面值 0x1.0p-126f，也等于 Float.intBitsToFloat(0x00800000)。
    public static final float MIN_NORMAL = 0x1.0p-126f; // 1.17549435E-38f
    
    //保存 float 类型数据的最小正非零值的常量，即 2^-149。
    //它等于十六进制的浮点字面值 0x0.000002P-126f，也等于 Float.intBitsToFloat(0x1)。
    public static final float MIN_VALUE = 0x0.000002P-126f; // 1.4e-45f
    
    //有限 float 变量可能具有的最大指数。它等于 Math.getExponent(Float.MAX_VALUE) 返回的值。
    public static final int MAX_EXPONENT = 127;
    
    //标准化 float 变量可能具有的最小指数。它等于 Math.getExponent(Float.MIN_NORMAL) 返回的值。
    public static final int MIN_EXPONENT = -126;
    
    //表示一个 float 值所使用的位数。
    public static final int SIZE = 32;
    
    //表示 float 基本类型的 Class 实例。
    public static final Class<Float> TYPE = Class.getPrimitiveClass("float");
    
    //返回 float 参数的字符串表示形式。下面提到的所有字符都是 ASCII 字符。
    //如果参数是 NaN，那么结果是字符串 "NaN"。
    //否则，结果是表示参数的符号和数值（绝对值）的字符串。
    //如果符号为负，那么结果的第一个字符是 '-' ('\u002D')；如果符号为正，则结果中不显示符号字符。
    //至于数值 m：
    //    如果 m 为无穷大，则用字符串 "Infinity" 表示；
    //  因此，正无穷大生成结果 "Infinity"，负无穷大生成结果 "-Infinity"。
    //    如果 m 为 0，则用字符 "0.0" 表示；因此，负 0 生成结果 "-0.0"，正 0 生成结果 "0.0"。
    //    如果 m 大于等于 10^-3，但小于 10^7，则采用不带前导 0 的十进制形式，用 m 的整数部分表示，
    //  后跟 '.' ('\u002E')，再后面是表示 m 小数部分的一个或多个十进制位数。
    //    如果 m 小于 10^-3 或大于等于 10^7，则用所谓的“计算机科学记数法”表示。
    //  设 n 为满足 10^n <= m < 10^(n+1) 的唯一整数；然后设 a 为 m 与 10^n 的精确算术商数值，
    //  从而 1 <= a < 10。那么，数值便表示为 a 的整数部分，其形式为：一个十进制位数，后跟 '.'，
    //  接着是表示 a 小数部分的十进制位数，再后面是字母 'E' ('\u0045')，最后是用十进制整数形式表示的 n，
    //  这与方法 Integer.toString(int) 生成的结果一样。
    //必须为 m 或 a 的小数部分打印多少位呢？至少必须有一位数来表示小数部分，除此之外，
    //需要更多（但只能和需要的一样多）位数来唯一地区分参数值和 float 类型的邻近值。
    //也就是说，假设 x 是用十进制表示法表示的精确算术值，是通过对有限非 0 参数 f 调用此方法生成的。
    //那么 f 一定是最接近 x 的 float 值，如果有两个 float 值同等地接近于 x，那么 f 必须是这两个值中的一个，
    //并且 f 的最低有效位必须是 0。 要创建浮点值的本地化字符串表示形式，请使用 NumberFormat 的子类。
    public static String toString(float f) {
        return new FloatingDecimal(f).toJavaFormatString();
    }
    
  } 
```
