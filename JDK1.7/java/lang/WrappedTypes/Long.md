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
    
    //以十六进制无符号整数形式返回 long 参数的字符串表示形式。
    //如果参数为负，则无符号 long 值为该参数加上 2^64；否则，它等于该参数。
    //此值将被转换为不带附加前导 0 的十六进制（基数 16）ASCII 数字字符串。
    //如果无符号大小为零，则该数字由单个零字符 '0' 表示 ('\u0030')；
    //否则，无符号大小表示形式中的第一个字符将不是零字符。下列字符都被用作十六进制数字：
    //   0123456789abcdef
    //这些是从 '\u0030' 到 '\u0039' 和从 '\u0061' 到 '\u0066' 的字符。
    //如果需要使用大写字母，则可以在结果上调用 String.toUpperCase() 方法：
    //   Long.toHexString(n).toUpperCase()
    public static String toHexString(long i) {
        return toUnsignedString(i, 4);
    }
    
    //以八进制无符号整数形式返回 long 参数的字符串表示形式。
    //如果参数为负，则无符号 long 值为该参数加上 2^64；否则，它等于该参数。
    //此值将被转换为不带附加前导 0 的八进制（基数 8）ASCII 数字字符串。
    //如果无符号大小为零，则该数字用单个零字符 '0' ('\u0030') 表示，
    //否则无符号大小表示形式中的第一个字符将不是零字符。以下字符都用作八进制数字：
    //  01234567
    //这些是从 '\u0030' 到 '\u0037' 的字符。
    public static String toOctalString(long i) {
        return toUnsignedString(i, 3);
    }
    
    //以二进制无符号整数形式返回 long 参数的字符串表示形式。
    //如果参数为负数，则无符号 long 值为该参数加上 2^64；否则，它等于该参数。
    //此值将被转换为不带附加前导 0 的二进制（基数 2）ASCII 数字字符串。
    //如果无符号大小为零，则用单个零字符 '0' 表示它 ('\u0030')；
    //否则，无符号大小表示形式中的第一个字符将不是零字符。
    //字符 '0' ('\u0030') 和 '1' ('\u0031') 被用作二进制位。
    public static String toBinaryString(long i) {
        return toUnsignedString(i, 1);
    }
    
    //将整数转换为无符号数。
    private static String toUnsignedString(long i, int shift) {
        char[] buf = new char[64];
        int charPos = 64;
        int radix = 1 << shift;
        long mask = radix - 1;
        do {
            buf[--charPos] = Integer.digits[(int)(i & mask)];
            i >>>= shift;
        } while (i != 0);
        return new String(buf, charPos, (64 - charPos));
    }
    
    //返回表示指定 long 的 String 对象。
    //该参数被转换为有符号的十进制表示形式，并作为字符串返回，
    //该字符串与用该参数和基数 10 作为参数的 toString(long, int) 方法所得到的值非常相似。
    public static String toString(long i) {
        if (i == Long.MIN_VALUE)
            return "-9223372036854775808";
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
        char[] buf = new char[size];
        getChars(i, size, buf);
        return new String(buf, true);
    }
    
    //将代表long i的字符存储到字符数组buf
    static void getChars(long i, int index, char[] buf) {
        long q;
        int r;
        int charPos = index;
        char sign = 0;

        if (i < 0) {
            sign = '-';
            i = -i;
        }

        // Get 2 digits/iteration using longs until quotient fits into an int
        while (i > Integer.MAX_VALUE) {
            q = i / 100;
            // really: r = i - (q * 100);
            r = (int)(i - ((q << 6) + (q << 5) + (q << 2)));
            i = q;
            buf[--charPos] = Integer.DigitOnes[r];
            buf[--charPos] = Integer.DigitTens[r];
        }

        // Get 2 digits/iteration using ints
        int q2;
        int i2 = (int)i;
        while (i2 >= 65536) {
            q2 = i2 / 100;
            // really: r = i2 - (q * 100);
            r = i2 - ((q2 << 6) + (q2 << 5) + (q2 << 2));
            i2 = q2;
            buf[--charPos] = Integer.DigitOnes[r];
            buf[--charPos] = Integer.DigitTens[r];
        }

        // 更小的数字通过快速模式处理
        // assert(i2 <= 65536, i2);
        for (;;) {
            q2 = (i2 * 52429) >>> (16+3);
            r = i2 - ((q2 << 3) + (q2 << 1));  // r = i2-(q2*10) ...
            buf[--charPos] = Integer.digits[r];
            i2 = q2;
            if (i2 == 0) break;
        }
        if (sign != 0) {
            buf[--charPos] = sign;
        }
    }
    
    //要求正整数
    static int stringSize(long x) {
        long p = 10;
        for (int i=1; i<19; i++) {
            if (x < p)
                return i;
            p = 10*p;
        }
        return 19;
    }
    
    //将 string 参数解析为有符号的 long，基数由第二个参数指定。
    //字符串中的字符必须为指定基数中的数字（由 Character.digit(char, int) 是否返回一个非负值来确定），
    //除非第一个字符为 ASCII 字符的减号 '-' ( '\u002D')，它表示一个负值。返回得到的 long 值。
    //注意，不允许将字符 L ('\u004C') 和 l ('\u006C') 作为类型指示符出现在字符串的结尾处，
    //而这一点在 Java 编程语言源代码中是允许的——除非 L 或 l 以大于 22 的基数形式出现。
    //如果出现以下情形之一，则抛出 NumberFormatException 类型的异常：
    //   第一个参数是 null 或零长度的字符串。
    //   radix 小于 Character.MIN_RADIX 或者大于 Character.MAX_RADIX。
    //   任何字符串的字符都不是指定基数的数字，除非第一个字符是减号 '-' ('\u002d')，假定字符串的长度大于 1。
    //   字符串表示的值不是 long 类型的值。
    //示例：
    //    parseLong("0", 10) returns 0L
    //    parseLong("473", 10) returns 473L
    //    parseLong("-0", 10) returns 0L
    //    parseLong("-FF", 16) returns -255L
    //    parseLong("1100110", 2) returns 102L
    //    parseLong("99", 8) returns NumberFormatException
    //    parseLong("Hazelnut", 10) returns NumberFormatException
    //    parseLong("Hazelnut", 36) returns 1356099454469L
    public static long parseLong(String s, int radix) throws NumberFormatException {
        if (s == null) {
            throw new NumberFormatException("null");
        }

        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }
        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        long result = 0;
        boolean negative = false;
        int i = 0, len = s.length();
        long limit = -Long.MAX_VALUE;
        long multmin;
        int digit;

        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Long.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);

                if (len == 1) // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                digit = Character.digit(s.charAt(i++),radix);
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix;
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        return negative ? result : -result;
    }
    
    //将 string 参数解析为有符号十进制 long。字符串中的字符必须都是十进制数字，
    //除非第一个字符是 ASCII 字符的减号 '-' ( \u002D')，它表示一个负值。
    //返回得到的 long 值，该值与用该参数和基数 10 
    //作为参数的 parseLong(java.lang.String, int) 方法得到的值非常相似。
    //注意，不允许将字符 L ('\u004C') 和 l ('\u006C') 作为类型指示符出现在字符串的结尾处，
    //这一点在 Java 编程语言源代码中是允许的。
    public static long parseLong(String s) throws NumberFormatException {
        return parseLong(s, 10);
    }
    
    //当使用第二个参数给出的基数进行解析时，返回保持从指定 String 中提取的值的 Long 对象。
    //第一个参数被解释为有符号的 long，基数由第二个参数指定，
    //该值与用该参数作为参数的 parseLong(java.lang.String, int) 方法得到的值非常类似。
    //结果是表示字符串指定的 long 值的 Long 对象。
    //换句话说，此方法返回一个 Long 对象，它的值等于：
    //   new Long(Long.parseLong(s, radix))
    public static Long valueOf(String s, int radix) throws NumberFormatException {
        return Long.valueOf(parseLong(s, radix));
    }
    
    //返回保持指定 String 的值的 Long 对象。
    //该参数被解释为表示一个有符号的十进制 long，
    //该值与用该参数作为参数的 parseLong(java.lang.String) 方法得到的值非常相似。
    //结果是表示由字符串指定的整数值的 Long 对象。
    //换句话说，此方法返回一个 Long 对象，它的值等于：
    //   new Long(Long.parseLong(s))
    public static Long valueOf(String s) throws NumberFormatException
    {
        return Long.valueOf(parseLong(s, 10));
    }
    
    private static class LongCache {
        private LongCache(){}

        static final Long cache[] = new Long[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Long(i - 128);
        }
    }
    
    //返回表示指定 long 值的 Long 实例。
    //如果不需要新的 Long 实例，则通常优先使用此方法，而不是使用构造方法 Long(long)，
    //因为此方法通过缓存频繁请求的值，可以显著提高时间和空间性能。
    public static Long valueOf(long l) {
        final int offset = 128;
        if (l >= -128 && l <= 127) { // will cache
            return LongCache.cache[(int)l + offset];
        }
        return new Long(l);
    }
    
    //将 String 解码成 Long。接受通过以下语法给出的十进制、十六进制和八进制数：
    //DecimalNumeral、 HexDigits 和 OctalDigits 在
    // Java Language Specification 中的 §3.10.1 中已经定义。
    //跟在（可选）负号和/或基数说明符（"0x"、"0X"、"#" 或前导零）后面的字符的顺序
    //由 Long.parseLong 方法通过指示的基数（10、16 或 8）来解析。
    //字符的顺序必须表示为一个正值，否则将抛出 NumberFormatException。
    //如果指定 String 的第一个字符是减号，则结果无效。String 中不允许出现空白字符。
    public static Long decode(String nm) throws NumberFormatException {
        int radix = 10;
        int index = 0;
        boolean negative = false;
        Long result;

        if (nm.length() == 0)
            throw new NumberFormatException("Zero length string");
        char firstChar = nm.charAt(0);
        // Handle sign, if present
        if (firstChar == '-') {
            negative = true;
            index++;
        } else if (firstChar == '+')
            index++;

        // Handle radix specifier, if present
        if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
            index += 2;
            radix = 16;
        }
        else if (nm.startsWith("#", index)) {
            index ++;
            radix = 16;
        }
        else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
            index ++;
            radix = 8;
        }

        if (nm.startsWith("-", index) || nm.startsWith("+", index))
            throw new NumberFormatException("Sign character in wrong position");

        try {
            result = Long.valueOf(nm.substring(index), radix);
            result = negative ? Long.valueOf(-result.longValue()) : result;
        } catch (NumberFormatException e) {
            String constant = negative ? ("-" + nm.substring(index))
                                       : nm.substring(index);
            result = Long.valueOf(constant, radix);
        }
        return result;
    }
    
    //Long的值
    private final long value;
    
    //构造新分配的 Long 对象，表示指定的 long 参数。
    public Long(long value) {
        this.value = value;
    }
    
    //构造新分配的 Long 对象，表示由 String 参数指示的 long 值。
    //该字符串被转换为 long 值，其方式与 radix 参数为 10 的 parseLong 方法所使用的方式一致。
    public Long(String s) throws NumberFormatException {
        this.value = parseLong(s, 10);
    }
    
    //以 byte 形式返回此 Long 的值。
    public byte byteValue() {
        return (byte)value;
    }
    
    //以 short 形式返回此 Long 的值。
    public short shortValue() {
        return (short)value;
    }
    
    //以 int 形式返回此 Long 的值。
    public int intValue() {
        return (int)value;
    }
    
    //以 long 值的形式返回此 Long 的值。
    public long longValue() {
        return (long)value;
    }
    
    //以 float 形式返回此 Long 的值。
    public float floatValue() {
        return (float)value;
    }
    
    //以 double 形式返回此 Long 的值。
    public double doubleValue() {
        return (double)value;
    }
    
    //返回表示 Long 值的 String 对象。
    //该值被转换为有符号十进制表示形式，并作为字符串返回，
    //该字符串与用 long 值作为参数的 toString(long) 方法得到的字符串非常相似。
    public String toString() {
        return toString(value);
    }
    
    //返回 Long 的哈希码。
    //结果是此 Long 对象保持的基本 long 值的两个部分的异或 (XOR)。也就是说，哈希码就是表达式的值：
    //  (int)(this.longValue()^(this.longValue()>>>32))
    public int hashCode() {
        return (int)(value ^ (value >>> 32));
    }
    
    //将此对象与指定对象进行比较。
    //当且仅当该参数不是 null，且 Long 对象与此对象包含相同的 long 值时，结果才为 true。
    public boolean equals(Object obj) {
        if (obj instanceof Long) {
            return value == ((Long)obj).longValue();
        }
        return false;
    }
    
    //确定具有指定名称的系统属性的 long 值。
    //第一个参数被视为系统属性的名称。
    //通过 System.getProperty(java.lang.String) 方法可以访问该系统属性。
    //然后，以 long 值的形式解释此属性的字符串值，并返回表示此值的 Long 对象。
    //在 getProperty 的定义中可以找到可能的数字格式的详细信息。
    //如果指定名称没有属性，或者指定名称为空或 null，抑或属性不具有正确的数字格式时，则返回 null。
    //换句话说，此方法返回一个 Long 对象，它的值等于：getLong(nm, null)
    public static Long getLong(String nm) {
        return getLong(nm, null);
    }
    
    //使用指定名称确定系统属性的 long 值。
    //第一个参数被视为系统属性的名称。
    //通过 System.getProperty(java.lang.String) 方法可以访问该系统属性。
    //然后，以 long 值的形式解释此属性的字符串值，并返回表示此值的 Long 对象。
    //在 getProperty 的定义中可以找到可能的数字格式的详细信息。
    //第二个参数是默认值。如果指定的名称没有属性，或者该属性不具备正确的数字格式，
    //抑或指定名称为空或 null，则返回表示第二个参数的值的 Long 对象。
    //换句话说，此方法返回一个 Long 对象，它的值等于：
    //   getLong(nm, new Long(val))
    //但是实际上，它可能通过以下方式实现：
    //   Long result = getLong(nm, null);
    //   return (result == null) ? new Long(val) :result;
    //这样可以避免不需要默认值时进行的不必要的 Long 对象分配。
    public static Long getLong(String nm, long val) {
        Long result = Long.getLong(nm, null);
        return (result == null) ? Long.valueOf(val) : result;
    }
    
    //使用指定名称返回系统属性的 long 值。第一个参数被视为系统属性的名称。
    //通过 System.getProperty(java.lang.String) 方法可以访问该系统属性。
    //然后，以 long 值的形式解释此属性的字符串值，并且按照 Long.decode 方法返回表示此值的 Long 对象。
    //  如果该属性值以两个 ASCII 字符 0x 或 ASCII 字符 # 开头，后面没有跟减号，
    // 则将该属性的其余部分解析为一个十六进制整数，
    // 该值与调用参数 radix 为 16 的 valueOf(java.lang.String, int) 方法得到的值非常相似。
    //  如果该属性值以 ASCII 字符 0 开头，后跟别的字符，则将它解析为一个八进制整数，
    // 该值与调用参数 radix 为 8 的 valueOf(java.lang.String, int) 方法得到的值非常相似。
    //  否则，将属性值解析为一个十进制整数，
    // 该值与调用参数 radix 为 10 的 valueOf(java.lang.String, int) 方法得到的值非常相似。
    //注意，在所有情况下，都不允许将 L ('\u004C') 和 l ('\u006C') 作为类型指示符出现在属性值的结尾处，
    //这一点在 Java 编程语言源代码中是允许的。
    //第二个参数是默认值。
    //如果指定的名称没有属性，或者属性不具有正确的数字格式，抑或指定名称为空或 null，则返回默认值。
    public static Long getLong(String nm, Long val) {
        String v = null;
        try {
            v = System.getProperty(nm);
        } catch (IllegalArgumentException e) {
        } catch (NullPointerException e) {
        }
        if (v != null) {
            try {
                return Long.decode(v);
            } catch (NumberFormatException e) {
            }
        }
        return val;
    }
    
    //在数字上比较两个 Long 对象。
    public int compareTo(Long anotherLong) {
        return compare(this.value, anotherLong.value);
    }
    public static int compare(long x, long y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
    
    //用来以二进制补码形式表示 long 值的位数。
    public static final int SIZE = 64;
    
    //返回至多有一个 1 位的 long 值，在指定的 long 值中最高位（最左边）的 1 位的位置。
    //如果指定值在其二进制补码表示形式中没有 1 位，即等于零，则返回零
    public static long highestOneBit(long i) {
        // HD, Figure 3-1
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        i |= (i >> 32);
        return i - (i >>> 1);
    }
    
    //返回至多有一个 1 位的 long 值，在指定的 long 值中最低位（最右边）的 1 位的位置。
    //如果指定值在其二进制补码表示形式中没有 1 位，即等于零，则返回零。
    public static long lowestOneBit(long i) {
        // HD, Section 2-1
        return i & -i;
    }
    
    //在指定 long 值的二进制补码表示形式中最高位（最左边）的 1 位之前，返回零位的数量。
    //如果指定值在其二进制补码表示形式中不存在 1 位，换句话说，如果它等于零，则返回 64。
    //注意，此方法与二进制对数密切相关。对于所有的正 long 值 x：
    //  floor(log2(x)) = 63 - numberOfLeadingZeros(x)
    //  ceil(log2(x)) = 64 - numberOfLeadingZeros(x - 1)
    public static int numberOfLeadingZeros(long i) {
        // HD, Figure 5-6
        if (i == 0)
            return 64;
        int n = 1;
        int x = (int)(i >>> 32);
        if (x == 0) { n += 32; x = (int)i; }
        if (x >>> 16 == 0) { n += 16; x <<= 16; }
        if (x >>> 24 == 0) { n +=  8; x <<=  8; }
        if (x >>> 28 == 0) { n +=  4; x <<=  4; }
        if (x >>> 30 == 0) { n +=  2; x <<=  2; }
        n -= x >>> 31;
        return n;
    }
    
    //返回在指定 long 值的二进制补码表示形式中最低位（最右边）的 1 位之后的零位的数量。
    //如果指定值在其二进制补码表示形式中不存在 1 位，换句话说，如果它等于零，则返回 64。
    public static int numberOfTrailingZeros(long i) {
        // HD, Figure 5-14
        int x, y;
        if (i == 0) return 64;
        int n = 63;
        y = (int)i; if (y != 0) { n = n -32; x = y; } else x = (int)(i>>>32);
        y = x <<16; if (y != 0) { n = n -16; x = y; }
        y = x << 8; if (y != 0) { n = n - 8; x = y; }
        y = x << 4; if (y != 0) { n = n - 4; x = y; }
        y = x << 2; if (y != 0) { n = n - 2; x = y; }
        return n - ((x << 1) >>> 31);
    }
    
    //返回指定 long 值的二进制补码表示形式中的 1 位的数量。此功能有时被称为填充计算。
    public static int bitCount(long i) {
        // HD, Figure 5-14
        i = i - ((i >>> 1) & 0x5555555555555555L);
        i = (i & 0x3333333333333333L) + ((i >>> 2) & 0x3333333333333333L);
        i = (i + (i >>> 4)) & 0x0f0f0f0f0f0f0f0fL;
        i = i + (i >>> 8);
        i = i + (i >>> 16);
        i = i + (i >>> 32);
        return (int)i & 0x7f;
    }
    
    //返回根据指定的位数循环左移指定的 long 值的二进制补码表示形式而得到的值。
    //（位是从左边（即高位）移出，从右边（即低位）再进入）
    //注意，使用负距离的左循环等同于右循环：
    //  rotateLeft(val, -distance) == rotateRight(val, distance)。
    //另请注意，使用 64 的倍数循环无效，因此除了最后六位，所有循环距离都可以忽略，
    //即使距离是负值：rotateLeft(val, distance) == rotateLeft(val, distance & 0x3F)。
    public static long rotateLeft(long i, int distance) {
        return (i << distance) | (i >>> -distance);
    }
    
    //返回根据指定的位数循环右移指定的 long 值的二进制补码表示形式而得到的值。
    //（位是从右边（即低位）移出，从左边（即高位）再进入）
    //注意，使用负距离右循环等于左循环：
    //  rotateRight(val, -distance) == rotateLeft(val, distance)。
    //另请注意，使用 64 的倍数循环无效，因此除了最后六位，所有循环距离都可以忽略，
    //即使距离是负值：rotateRight(val, distance) == rotateRight(val, distance & 0x3F)。
    public static long rotateRight(long i, int distance) {
        return (i >>> distance) | (i << -distance);
    }
    
    //返回通过反转指定 long 值的二进制补码表示形式中位的顺序而获得的值。
    public static long reverse(long i) {
        // HD, Figure 7-1
        i = (i & 0x5555555555555555L) << 1 | (i >>> 1) & 0x5555555555555555L;
        i = (i & 0x3333333333333333L) << 2 | (i >>> 2) & 0x3333333333333333L;
        i = (i & 0x0f0f0f0f0f0f0f0fL) << 4 | (i >>> 4) & 0x0f0f0f0f0f0f0f0fL;
        i = (i & 0x00ff00ff00ff00ffL) << 8 | (i >>> 8) & 0x00ff00ff00ff00ffL;
        i = (i << 48) | ((i & 0xffff0000L) << 16) |
            ((i >>> 16) & 0xffff0000L) | (i >>> 48);
        return i;
    }
    
    //返回指定 long 值的符号函数。
    //（如果指定值为负，则返回值 -1；如果指定值为零，则返回 0；如果指定值为正，则返回 1）
    public static int signum(long i) {
        // HD, Section 2-7
        return (int) ((i >> 63) | (-i >>> 63));
    }
    
    //返回通过反转指定 long 值的二进制补码表示形式中字节的顺序而获得的值。
    public static long reverseBytes(long i) {
        i = (i & 0x00ff00ff00ff00ffL) << 8 | (i >>> 8) & 0x00ff00ff00ff00ffL;
        return (i << 48) | ((i & 0xffff0000L) << 16) |
            ((i >>> 16) & 0xffff0000L) | (i >>> 48);
    }
    
    private static final long serialVersionUID = 4290774380558885855L;
  }
```
