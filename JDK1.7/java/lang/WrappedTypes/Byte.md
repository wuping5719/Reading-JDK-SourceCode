* Byte类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Byte 类将基本类型 byte 的值包装在一个对象中。一个 Byte 类型的对象只包含一个类型为 byte 的字段。

  &nbsp;&nbsp; 此外，该类还为 byte 和 String 的相互转换提供了几种方法，并提供了处理 byte 时非常有用的其他一些常量和方法。
  
```java
  public final class Byte extends Number implements Comparable<Byte> {
    //一个常量，保存 byte 类型可取的最小值，即-128 (-2^7)。
    public static final byte   MIN_VALUE = -128;
    
    //一个常量，保存 byte 类型可取的最大值，即127 (2^7-1)。
    public static final byte   MAX_VALUE = 127;
    
    //表示基本类型 byte 的 Class 实例。
    public static final Class<Byte> TYPE = (Class<Byte>) Class.getPrimitiveClass("byte");
    
    //返回表示指定 byte 的一个新 String 对象。假定基数为 10。
    public static String toString(byte b) {
        return Integer.toString((int)b, 10);
    }
    
    private static class ByteCache {
        private ByteCache(){}

        static final Byte cache[] = new Byte[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Byte((byte)(i - 128));
        }
    }
    //返回表示指定 byte 值的一个 Byte 实例。
    //如果不需要新的 Byte 实例，则通常应优先使用此方法，而不是构造方法 Byte(byte)，
    //因为该方法有可能通过缓存经常请求的值来显著提高空间和时间性能。
    public static Byte valueOf(byte b) {
        final int offset = 128;
        return ByteCache.cache[(int)b + offset];
    }
    
    //将 string 参数解析为一个有符号的 byte，其基数由第二个参数指定。
    //除了第一个字符可以是表示负值的 ASCII 负号 '-' ( '\u002D') 之外
    //（这取决于 Character.digit(char, int) 是否返回非负值），
    //该 string 中的字符必须都是指定基数的数字。返回得到的 byte 值。
    //如果出现下列任何一种情况，则抛出一个 NumberFormatException 类型的异常：
    //  第一个参数为 null 或是一个长度为零的字符串。
    //  基数小于 Character.MIN_RADIX 或者大于 Character.MAX_RADIX。
    //  字符串的任一字符不是指定基数的数字，
    // 第一个字符是负号 '-' ('\u002D') 的情况除外（但此时字符串的长度应超过 1）。
    //  字符串所表示的值不是 byte 类型的值。
    public static byte parseByte(String s, int radix) throws NumberFormatException {
        int i = Integer.parseInt(s, radix);
        if (i < MIN_VALUE || i > MAX_VALUE)
            throw new NumberFormatException(
                "Value out of range. Value:\"" + s + "\" Radix:" + radix);
        return (byte)i;
    }
    
    //将 string 参数解析为有符号的十进制 byte。
    //除了第一个字符可以是表示负值的 ASCII 负号 '-' ( '\u002D') 之外，
    //该字符串中的字符必须都是十进制数字。
    //返回得到的 byte 值与以该 string 参数和基数 10 为参数的
    //parseByte(java.lang.String, int) 方法所返回的值一样。
    public static byte parseByte(String s) throws NumberFormatException {
        return parseByte(s, 10);
    }
    
    //返回一个 Byte 对象，该对象保持从指定的 String 中提取的值，
    //该值是在用第二个参数所给定的基数对指定字符串进行解析时提取的。
    //第一个参数被解释为用第二个参数所指定的基数表示一个有符号的 byte，
    //正如将该参数指定给 parseByte(java.lang.String, int) 方法一样。
    //结果是一个表示该 string 所指定的 byte 值的 Byte 对象。
    //换句话说，该方法返回一个等于以下代码的值的 Byte 对象：
    //   new Byte(Byte.parseByte(s, radix))
    public static Byte valueOf(String s, int radix) throws NumberFormatException {
        return valueOf(parseByte(s, radix));
    }
    
    //返回一个保持指定 String 所给出的值的 Byte 对象。
    //参数被解释为表示一个有符号的十进制的 byte，
    //正如将该参数指定给 parseByte(java.lang.String) 方法一样。
    //结果是一个表示该 string 所指定的 byte 值的 Byte 对象。
    //换句话说，该方法返回一个等于以下代码的值的 Byte 对象：
    //   new Byte(Byte.parseByte(s))
    public static Byte valueOf(String s) throws NumberFormatException {
        return valueOf(s, 10);
    }
    
    //将 String 解码为 Byte。接受按下列语法给出的十进制、十六进制和八进制数。
    //Java Language Specification 的 §3.10.1 中给出了
    // DecimalNumeral、 HexDigits 和 OctalDigits 的定义。
    //对（可选）负号和/或基数说明符（“0x”、“0X”、“#” 或前导零）后面的字符序列
    //进行解析就如同使用带指定基数（10、16 或 8）的 Byte.parseByte 方法一样。
    //该字符序列必须表示一个正值，否则将抛出 NumberFormatException。
    //如果指定 String 的第一个字符是负号，则结果将被求反。该 String 中不允许出现空白字符。
    public static Byte decode(String nm) throws NumberFormatException {
        int i = Integer.decode(nm);
        if (i < MIN_VALUE || i > MAX_VALUE)
            throw new NumberFormatException(
                    "Value " + i + " out of range from input " + nm);
        return valueOf((byte)i);
    }
    
    //Byte的值
    private final byte value;
    
    //构造一个新分配的 Byte 对象，以表示指定的 byte 值。
    public Byte(byte value) {
        this.value = value;
    }
    
    //构造一个新分配的 Byte 对象，以表示 String 参数所指示的 byte 值。
    //该字符串以使用基数 10 的 parseByte 方法所使用的方式被转换成一个 byte 值。
    public Byte(String s) throws NumberFormatException {
        this.value = parseByte(s, 10);
    }
    
    //作为一个 byte 返回此 Byte 的值。
    public byte byteValue() {
        return value;
    }
    
    //作为一个 short 返回此 Byte 的值。
    public short shortValue() {
        return (short)value;
    }
    
    //作为一个 int 返回此 Byte 的值。
    public int intValue() {
        return (int)value;
    }
    
    //作为一个 long 返回此 Byte 的值。
    public long longValue() {
        return (long)value;
    }
    
    //作为一个 float 返回此 Byte 的值。
    public float floatValue() {
        return (float)value;
    }
    
    //作为一个 double 返回此 Byte 的值。
    public double doubleValue() {
        return (double)value;
    }
    
    //返回表示此 Byte 的值的 String 对象。
    //该值被转换成有符号的十进制表示形式，并作为一个 string 返回，
    //正如将 byte 值作为一个参数指定给 toString(byte) 方法所返回的一样。
    public String toString() {
        return Integer.toString((int)value);
    }
    
    //返回此 Byte 的哈希码。
    public int hashCode() {
        return (int)value;
    }
    
    //将此对象与指定对象比较。当且仅当参数不为 null，
    //而是一个与此对象一样包含相同 Byte 值的 byte 对象时，结果才为 true。
    public boolean equals(Object obj) {
        if (obj instanceof Byte) {
            return value == ((Byte)obj).byteValue();
        }
        return false;
    }
    
    //在数字上比较两个 Byte 对象。
    public int compareTo(Byte anotherByte) {
        return compare(this.value, anotherByte.value);
    }
    public static int compare(byte x, byte y) {
        return x - y;
    }
    
    //用于以二进制补码形式表示 byte 值的位数。
    public static final int SIZE = 8;
    
    private static final long serialVersionUID = -7183698231559129828L;
  }
```
