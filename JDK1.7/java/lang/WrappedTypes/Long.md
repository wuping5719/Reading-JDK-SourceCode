* Long类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Long 类在对象中包装了基本类型 long 的值。每个 Long 类型的对象都包含一个 long 类型的字段。

  &nbsp;&nbsp; 该类提供了多个方法，可以将 long 转换为 String，将 String 转换为 long，除此之外，还提供了其他一些处理 long 时有用的常量和方法。

  &nbsp;&nbsp; 实现注意事项："bit twiddling" 方法（如 highestOneBit 和 numberOfTrailingZeros）的实现基于 Henry S. Warren 和 Jr. 撰写的 Hacker's Delight (Addison Wesley, 2002) 一书中的资料。
  
```java
  public final class Long extends Number implements Comparable<Long> {
    //保持 long 类型的最小值的常量，该值为 -2^63。
    public static final long MIN_VALUE = 0x8000000000000000L;
    
    //保持 long 类型的最大值的常量，该值为 2^63-1。
    public static final long MAX_VALUE = 0x7fffffffffffffffL;
    
    //表示基本类型 long 的 Class 实例。
    public static final Class<Long> TYPE = (Class<Long>) Class.getPrimitiveClass("long");
    
    //返回在使用第二个参数指定的基数时第一个参数的字符串表示形式。
    //如果该基数小于 Character.MIN_RADIX，或大于 Character.MAX_RADIX，则使用基数 10。
    //如果第一个参数是负数，则结果的第一个元素是 ASCII 字符的减号 '-' ('\u002d')。如果第一个参数非负，
    //则结果中不会出现符号字符。结果的其余字符表示第一个参数的大小。
    //如果大小为零，则用单个零字符 '0' 表示它 ('\u0030')；否则大小表示形式中的第一个字符将不是零字符。
    //以下 ASCII 字符均被用作数字：
    //   0123456789abcdefghijklmnopqrstuvwxyz
    //这些是从 '\u0030' 到 '\u0039' 和从 '\u0061' 到 '\u007a' 的字符。
    //如果 radix 是 N，则这些字符的第一个 N 用作显示顺序中基数 N 的数字。
    //因此，该数字的十六进制（基数 16）表示形式为 0123456789abcdef。
    //如果需要使用大写字母，则可以在结果上调用 String.toUpperCase() 方法：
    //   Long.toString(n, 16).toUpperCase()
    public static String toString(long i, int radix) {
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;
        if (radix == 10)
            return toString(i);
        char[] buf = new char[65];
        int charPos = 64;
        boolean negative = (i < 0);

        if (!negative) {
            i = -i;
        }

        while (i <= -radix) {
            buf[charPos--] = Integer.digits[(int)(-(i % radix))];
            i = i / radix;
        }
        buf[charPos] = Integer.digits[(int)(-i)];

        if (negative) {
            buf[--charPos] = '-';
        }

        return new String(buf, charPos, (65 - charPos));
    }
    
  }
```
