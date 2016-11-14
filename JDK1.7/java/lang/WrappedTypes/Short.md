* Short类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Short 类在对象中包装基本类型 short 的值。一个 Short 类型的对象只包含一个 short 类型的字段。

  &nbsp;&nbsp; 该类提供了多个方法，可以将 short 转换为 String，将 String 转换为 short，同时还提供了其他一些处理 short 时有用的常量和方法。
  
```java
  public final class Short extends Number implements Comparable<Short> {
    //保存 short 可取的最小值的常量，最小值为 -2^15。
    public static final short   MIN_VALUE = -32768;
    
    //保存 short 可取的最大值的常量，最大值为 2^15-1。
    public static final short   MAX_VALUE = 32767;
    
    //表示基本类型 short 的 Class 实例。
    public static final Class<Short> TYPE = (Class<Short>) Class.getPrimitiveClass("short");
    
    //返回表示指定 short 的一个新 String 对象。假定用十进制表示。
    public static String toString(short s) {
        return Integer.toString((int)s, 10);
    }
    
    //将字符串参数解析为由第二个参数指定的基数中的有符号的 short。
    //该字符串中的字符必须都是指定基数（这取决于 Character.digit(char, int) 是否返回非负值）的数字，
    //除非第一个字符是表示负值的 ASCII 符号中的负号 “-” ( '\u002D')。返回得到的 byte 值。
    //如果出现以下情形之一，则抛出 NumberFormatException 类型的异常：
    //   第一个参数是 null 或零长度的字符串。
    //   基数小于 Character.MIN_RADIX 或大于 Character.MAX_RADIX。
    //   除了在字符串长度超过 1 的情况下第一个字符可能是负号 “-” ('\u002D') 之外，
    // 字符串的任何字符都不是指定基数的数字。
    //   字符串所表示的值不是 short 类型的值。
    public static short parseShort(String s, int radix) throws NumberFormatException {
        int i = Integer.parseInt(s, radix);
        if (i < MIN_VALUE || i > MAX_VALUE)
            throw new NumberFormatException("Value out of range. Value:\"" + s + "\" Radix:" + radix);
        return (short)i;
    }
    
    //将字符串参数解析为有符号的十进制 short。该字符串中的字符必须都是十进制数字，
    //除非第一个字符是表示负值的 ASCII 符号中的负号 '-' ( '\u002D')。返回得到的 short 值，
    //此值与用该参数和基数 10 作为参数的 parseShort(java.lang.String, int) 方法得到的值相同。
    public static short parseShort(String s) throws NumberFormatException {
        return parseShort(s, 10);
    }
    
    //返回一个 Short 对象，该对象保持从指定的 String 中提取的值，
    //该值是在使用第二个参数给出的基数对指定字符串进行解析时提取的。
    //第一个参数被解释为表示在使用第二个参数所指定基数时的一个有符号的 short，
    //此值与用该参数作为参数的 parseShort(java.lang.String, int) 方法得到的值相同。
    //结果是一个表示该字符串所指定的 short 值的 Short 对象。
    //换句话说，此方法返回一个 Short 对象，它的值等于：
    //  new Short(Short.parseShort(s, radix))
    public static Short valueOf(String s, int radix) throws NumberFormatException {
        return valueOf(parseShort(s, radix));
    }
    
    //返回一个保持指定 String 所给出的值的 Short 对象。
    //该参数被解释为表示一个有符号的十进制 short，
    //此值与用该参数作为参数的 #parseLong(java.lang.String) 方法得到的值相同。
    //结果是一个表示该字符串所指定的 short 值的 Short 对象。
    //换句话说，此方法返回一个 Short 对象，它的值等于：
    //  new Short(Short.parseShort(s))
    public static Short valueOf(String s) throws NumberFormatException {
        return valueOf(s, 10);
    }
    
    private static class ShortCache {
        private ShortCache(){}

        static final Short cache[] = new Short[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Short((short)(i - 128));
        }
    }
    
    //返回表示指定 short 值的 Short 实例。
    //如果不需要新的 Short 实例，则通常应该优先采用此方法，而不是构造方法 Short(short)，
    //因为此方法很可能通过缓存经常请求的值来显著提高空间和时间性能。
    public static Short valueOf(short s) {
        final int offset = 128;
        int sAsInt = s;
        if (sAsInt >= -128 && sAsInt <= 127) { // must cache
            return ShortCache.cache[sAsInt + offset];
        }
        return new Short(s);
    }
    
    //将 String 解码为 Short。接受通过以下语法给出的十进制、十六进制和八进制数：
    //DecimalNumeral、 HexDigits 和 OctalDigits 在
    // Java Language Specification 的 §3.10.1 中已经定义。
    //对（可选）负号和/或基数说明符（“0x”、“0X”、“#” 或前导零）后面的字符序列进行解析就如同用
    // Short.parseByte 方法来解析指定的基数（10、16 或 8）一样。
    //该字符序列必须表示为一个正值，否则将抛出 NumberFormatException。
    //如果指定 String 的第一个字符是减号，则结果无效。String 中不允许出现空白字符。
    public static Short decode(String nm) throws NumberFormatException {
        int i = Integer.decode(nm);
        if (i < MIN_VALUE || i > MAX_VALUE)
            throw new NumberFormatException("Value " + i + " out of range from input " + nm);
        return valueOf((short)i);
    }
    
    //Short的值
    private final short value;
    
    //构造一个新分配的 Short 对象，用来表示指定的 short 值。
    public Short(short value) {
        this.value = value;
    }
    
    //构造一个新分配的 Short 对象，用来表示 String 参数所指示的 short 值。
    //将字符串转换为 short 值，转换方式与基数为 10 的 parseShort 方法所用的方式完全相同。
    public Short(String s) throws NumberFormatException {
        this.value = parseShort(s, 10);
    }
    
    //以 byte 形式返回此 Short 的值。
    public byte byteValue() {
        return (byte)value;
    }
    
    //以 short 形式返回此 Short 的值。
    public short shortValue() {
        return value;
    }
    
    //以 int 形式返回此 Short 的值。
    public int intValue() {
        return (int)value;
    }
    
    //以 Long 形式返回此 Short 的值。
    public long longValue() {
        return (long)value;
    }
    
    //以 float 形式返回此 Short 的值。
    public float floatValue() {
        return (float)value;
    }
    
    //以 double 形式返回此 Short 的值。
    public double doubleValue() {
        return (double)value;
    }
    
    //返回表示此 Short 的值的 String 对象。
    //该值被转换成有符号的十进制表示形式，并作为一个字符串返回，
    //正如将 short 值作为一个参数指定给 toString(short) 方法所得到的值那样。
    public String toString() {
        return Integer.toString((int)value);
    }
    
    //返回此 Short 的哈希码。
    public int hashCode() {
        return (int)value;
    }
    
    //将此对象与指定对象比较。当且仅当参数不是 null，
    //而是一个与该对象一样包含相同 short 值的 Short 对象时，结果才为 true。
    public boolean equals(Object obj) {
        if (obj instanceof Short) {
            return value == ((Short)obj).shortValue();
        }
        return false;
    }
    
    //比较两个 Short 对象所表示的数值。
    public int compareTo(Short anotherShort) {
        return compare(this.value, anotherShort.value);
    }
    
    public static int compare(short x, short y) {
        return x - y;
    }
    
    //用来以二进制补码形式表示 short 值的位数。
    public static final int SIZE = 16;
    
    //返回通过反转指定 short 值的二进制补码表示形式中字节的顺序而获得的值。
    public static short reverseBytes(short i) {
        return (short) (((i & 0xFF00) >> 8) | (i << 8));
    }
    
    private static final long serialVersionUID = 7515723908773894738L;
  }
```
