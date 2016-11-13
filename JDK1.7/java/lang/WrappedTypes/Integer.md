* Integer类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Integer 类在对象中包装了一个基本类型 int 的值。Integer 类型的对象包含一个 int 类型的字段。

  &nbsp;&nbsp; 该类提供了多个方法，能在 int 类型和 String 类型之间互相转换，还提供了处理 int 类型时非常有用的其他一些常量和方法。

  &nbsp;&nbsp; 实现注意事项：“bit twiddling”方法（如 highestOneBit 和 numberOfTrailingZeros）的实现基于 Henry S. Warren, Jr.撰写的 Hacker's Delight（Addison Wesley, 2002）中的一些有关材料。
  
```java
  public final class Integer extends Number implements Comparable<Integer> {
    //值为 －2^31 的常量，它表示 int 类型能够表示的最小值。
    public static final int   MIN_VALUE = 0x80000000;
    
    //值为 2^31－1 的常量，它表示 int 类型能够表示的最大值。
    public static final int   MAX_VALUE = 0x7fffffff;
    
    //表示基本类型 int 的 Class 实例。
    public static final Class<Integer> TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
    
    //可以表示成字符串形式的数字的所有可能的字符
    final static char[] digits = {
        '0' , '1' , '2' , '3' , '4' , '5' ,
        '6' , '7' , '8' , '9' , 'a' , 'b' ,
        'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
        'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
        'o' , 'p' , 'q' , 'r' , 's' , 't' ,
        'u' , 'v' , 'w' , 'x' , 'y' , 'z'
    };
    
    //返回用第二个参数指定基数表示的第一个参数的字符串表示形式。
    //如果基数小于 Character.MIN_RADIX 或者大于 Character.MAX_RADIX，则改用基数 10。
    //如果第一个参数为负，则结果中的第一个元素为 ASCII 的减号 '-' ('\u002D')。
    //如果第一个参数为非负，则没有符号字符出现在结果中。
    //结果中的剩余字符表示第一个参数的大小。如果大小为零，则用一个零字符 '0' ('\u0030') 表示；
    //否则，大小的表示形式中的第一个字符将不是零字符。用以下 ASCII 字符作为数字：
    //   0123456789abcdefghijklmnopqrstuvwxyz
    //其范围是从 '\u0030' 到 '\u0039' 和从 '\u0061' 到 '\u007A'。
    //如果 radix 为 N, 则按照所示顺序，使用这些字符中的前 N 个作为其数字。
    //因此，十六进制（基数为 16）的数字是 0123456789abcdef。
    //如果希望得到大写字母，则可以在结果上调用 String.toUpperCase() 方法：
    //   Integer.toString(n, 16).toUpperCase()
    public static String toString(int i, int radix) {
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;

        //使用更快的版本
        if (radix == 10) {
            return toString(i);
        }

        char buf[] = new char[33];
        boolean negative = (i < 0);
        int charPos = 32;

        if (!negative) {
            i = -i;
        }

        while (i <= -radix) {
            buf[charPos--] = digits[-(i % radix)];
            i = i / radix;
        }
        buf[charPos] = digits[-i];

        if (negative) {
            buf[--charPos] = '-';
        }

        return new String(buf, charPos, (33 - charPos));
    }
    
    //以十六进制（基数 16）无符号整数形式返回一个整数参数的字符串表示形式。
    //如果参数为负，那么无符号整数值为参数加上 2^32；否则等于该参数。
    //将该值转换为十六进制（基数 16）的无前导 0 的 ASCII 数字字符串。
    //如果无符号数的大小值为零，则用一个零字符 '0' (’\u0030’) 表示它；
    //否则，无符号数大小的表示形式中的第一个字符将不是零字符。用以下字符作为十六进制数字：
    //   0123456789abcdef
    //这些字符的范围是从 '\u0030' 到 '\u0039' 和从 '\u0061' 到 '\u0066'。
    //如果希望得到大写字母，可以在结果上调用 String.toUpperCase() 方法：
    //   Integer.toHexString(n).toUpperCase()
    public static String toHexString(int i) {
        return toUnsignedString(i, 4);
    }
    
    //以八进制（基数 8）无符号整数形式返回一个整数参数的字符串表示形式。
    //如果参数为负，该无符号整数值为参数加上 2^32；否则等于该参数。
    //该值被转换成八进制（基数 8）ASCII 数字的字符串，且没有附加前导 0。
    //如果无符号数大小为零，则用一个零字符 '0' ('\u0030') 表示它；
    //否则，无符号数大小的表示形式中的第一个字符将不是零字符。用以下字符作为八进制数字：
    //   01234567
    //它们是从 '\u0030' 到 '\u0037' 的字符。
    public static String toOctalString(int i) {
        return toUnsignedString(i, 3);
    }
    
    //以二进制（基数 2）无符号整数形式返回一个整数参数的字符串表示形式。
    //如果参数为负，该无符号整数值为参数加上 2^32；否则等于该参数。
    //将该值转换为二进制（基数 2）形式的无前导 0 的 ASCII 数字字符串。
    //如果无符号数的大小为零，则用一个零字符 '0' (’\u0030’) 表示它；
    //否则，无符号数大小的表示形式中的第一个字符将不是零字符。
    //字符 '0' ('\u0030') 和 '1' ('\u0031') 被用作二进制数字。
    public static String toBinaryString(int i) {
        return toUnsignedString(i, 1);
    }
    
    //将整数转换为无符号数。
    private static String toUnsignedString(int i, int shift) {
        char[] buf = new char[32];
        int charPos = 32;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[--charPos] = digits[i & mask];
            i >>>= shift;
        } while (i != 0);

        return new String(buf, charPos, (32 - charPos));
    }
    
    final static char [] DigitTens = {
        '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
        '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
        '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
        '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
        '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
        '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
        '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
        '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
        '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
        '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
        } ;

    final static char [] DigitOnes = {
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        } ;
        
    //返回一个表示指定整数的 String 对象。
    //将该参数转换为有符号的十进制表示形式，以字符串形式返回它，
    //就好像将参数和基数 10 作为参数赋予 toString(int, int) 方法。
    public static String toString(int i) {
        if (i == Integer.MIN_VALUE)
            return "-2147483648";
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
        char[] buf = new char[size];
        getChars(i, size, buf);
        return new String(buf, true);
    }
    //将代表整数i的字符放入字符数组buf中
    static void getChars(int i, int index, char[] buf) {
        int q, r;
        int charPos = index;
        char sign = 0;

        if (i < 0) {
            sign = '-';
            i = -i;
        }

        // 每次迭代生成两个数字
        while (i >= 65536) {
            q = i / 100;
         // really: r = i - (q * 100);
            r = i - ((q << 6) + (q << 5) + (q << 2));
            i = q;
            buf [--charPos] = DigitOnes[r];
            buf [--charPos] = DigitTens[r];
        }

        for (;;) {
            q = (i * 52429) >>> (16+3);
            r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
            buf [--charPos] = digits [r];
            i = q;
            if (i == 0) break;
        }
        if (sign != 0) {
            buf [--charPos] = sign;
        }
    }
    
    final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };
    
    //要求正整数x
    static int stringSize(int x) {
        for (int i=0; ; i++)
            if (x <= sizeTable[i])
                return i+1;
    }
    
    //使用第二个参数指定的基数，将字符串参数解析为有符号的整数。
    //除了第一个字符可以是用来表示负值的 ASCII 减号 '-' ( '\u002D’)外，
    //字符串中的字符必须都是指定基数的数字
    //（通过 Character.digit(char, int) 是否返回一个负值确定）。返回得到的整数值。
    //如果发生以下任意一种情况，则抛出一个 NumberFormatException 类型的异常：
    //   第一个参数为 null 或一个长度为零的字符串。
    //   基数小于 Character.MIN_RADIX 或者大于 Character.MAX_RADIX。
    //   假如字符串的长度超过 1，那么除了第一个字符可以是减号 '-' ('u002D’) 外，
    // 字符串中存在任意不是由指定基数的数字表示的字符。
    //   字符串表示的值不是 int 类型的值。
    //示例：
    //    parseInt("0", 10) 返回 0
    //    parseInt("473", 10) 返回 473
    //    parseInt("-0", 10) 返回 0
    //    parseInt("-FF", 16) 返回 -255
    //    parseInt("1100110", 2) 返回 102
    //    parseInt("2147483647", 10) 返回 2147483647
    //    parseInt("-2147483648", 10) 返回 -2147483648
    //    parseInt("2147483648", 10) 抛出 NumberFormatException
    //    parseInt("99", 8) 抛出 NumberFormatException
    //    parseInt("Kona", 10) 抛出 NumberFormatException
    //    parseInt("Kona", 27) 返回 411787
    public static int parseInt(String s, int radix) throws NumberFormatException {
        //警告:该方法可能在VM初始化期间IntegerCache被初始化之前调用。注意不要使用 valueOf 方法。
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

        int result = 0;
        boolean negative = false;
        int i = 0, len = s.length();
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;

        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
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
    
    //将字符串参数作为有符号的十进制整数进行解析。
    //除了第一个字符可以是用来表示负值的 ASCII 减号 '-' ( '\u002D') 外，
    //字符串中的字符都必须是十进制数字。返回得到的整数值，
    //就好像将该参数和基数 10 作为参数赋予 parseInt(java.lang.String, int) 方法一样。
    public static int parseInt(String s) throws NumberFormatException {
        return parseInt(s,10);
    }
    
    //返回一个 Integer 对象，该对象中保存了用第二个参数提供的基数进行解析时从指定的 String 中提取的值。
    //将第一个参数解释为用第二个参数指定的基数表示的有符号整数，
    //就好像将这些参数赋予 parseInt(java.lang.String, int) 方法一样。
    //结果是一个表示字符串指定的整数值的 Integer 对象。
    //换句话说，该方法返回一个等于以下值的 Integer 对象：
    //   new Integer(Integer.parseInt(s, radix))
    public static Integer valueOf(String s, int radix) throws NumberFormatException {
        return Integer.valueOf(parseInt(s,radix));
    }
    
    //返回保存指定的 String 的值的 Integer 对象。
    //将该参数解释为表示一个有符号的十进制整数, 就好像将该参数赋予 parseInt(java.lang.String) 方法一样。
    //结果是一个表示字符串指定的整数值的 Integer 对象。
    //换句话说，该方法返回一个等于以下值的 Integer 对象：
    //   new Integer(Integer.parseInt(s))
    public static Integer valueOf(String s) throws NumberFormatException {
        return Integer.valueOf(parseInt(s, 10));
    }
    
    //缓存支持在-128到127之间的整数自动装箱
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // 高位值可以通过属性配置
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // 数组的最大大小是Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }
    //返回一个表示指定的 int 值的 Integer 实例。
    //如果不需要新的 Integer 实例，则通常应优先使用该方法，而不是构造方法 Integer(int)，
    //因为该方法有可能通过缓存经常请求的值而显著提高空间和时间性能。
    public static Integer valueOf(int i) {
        assert IntegerCache.high >= 127;
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    
    //Integer的值
    private final int value;
    
    //构造一个新分配的 Integer 对象，它表示指定的 int 值。
    public Integer(int value) {
        this.value = value;
    }
    
    //构造一个新分配的 Integer 对象，它表示 String 参数所指示的 int 值。
    //使用与 parseInt 方法（对基数为 10 的值）相同的方式将该字符串转换成 int 值。
    public Integer(String s) throws NumberFormatException {
        this.value = parseInt(s, 10);
    }
    
    //以 byte 类型返回该 Integer 的值。
    public byte byteValue() {
        return (byte)value;
    }
    
    //以 short 类型返回该 Integer 的值。
    public short shortValue() {
        return (short)value;
    }
    
    //以 int 类型返回该 Integer 的值。
    public int intValue() {
        return value;
    }
    
    //以 long 类型返回该 Integer 的值。
    public long longValue() {
        return (long)value;
    }
    
    //以 float 类型返回该 Integer 的值。
    public float floatValue() {
        return (float)value;
    }
    
    //以 double 类型返回该 Integer 的值。
    public double doubleValue() {
        return (double)value;
    }
    
    //返回一个表示该 Integer 值的 String 对象。
    //将该参数转换为有符号的十进制表示形式，并以字符串的形式返回它，
    //就好像将该整数值作为参数赋予 toString(int) 方法一样。
    public String toString() {
        return toString(value);
    }
    
    //返回此 Integer 的哈希码。
    public int hashCode() {
        return value;
    }
    
    //比较此对象与指定对象。
    //当且仅当参数不为 null，并且是一个与该对象包含相同 int 值的 Integer 对象时，结果为 true。
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
    
    //确定具有指定名称的系统属性的整数值。
    //第一个参数被视为系统属性的名称。通过 System.getProperty(java.lang.String) 方法可以访问系统属性。
    //然后，将该属性的字符串值解释为一个整数值，并返回表示该值的 Integer 对象。
    //使用 getProperty 的定义可以找到可能出现的数字格式的详细信息。
    //如果没有具有指定名称的属性，或者指定名称为空或 null，或者属性的数字格式不正确，则返回 null。
    //换句话说，该方法返回一个等于以下值的 Integer 对象：
    //  getInteger(nm, null)
    public static Integer getInteger(String nm) {
        return getInteger(nm, null);
    }
    
    //确定具有指定名称的系统属性的整数值。
    //第一个参数被视为系统属性的名称。通过 System.getProperty(java.lang.String) 方法可以访问系统属性。
    //然后，将该属性的字符串值解释为一个整数值，并返回表示该值的 Integer 对象。
    //使用 getProperty 的定义可以找到可能出现的数字格式的详细信息。
    //第二个参数是默认值。如果未具有指定名称的属性，或者属性的数字格式不正确，或者指定名称为空或 null，
    //则返回一个表示第二个参数的值的 Integer 对象。
    //换句话说，该方法返回一个等于以下值的 Integer 对象：
    //   getInteger(nm, new Integer(val))
    //但在实践中可能会用以下类似方式实现它：
    //   Integer result = getInteger(nm, null);
    //   return (result == null) ? new Integer(val) : result;
    //从而避免在无需默认值时分配不必要的 Integer 对象。
    public static Integer getInteger(String nm, int val) {
        Integer result = getInteger(nm, null);
        return (result == null) ? Integer.valueOf(val) : result;
    }
    
    //返回具有指定名称的系统属性的整数值。第一个参数被视为系统属性的名称。
    //通过 System.getProperty(java.lang.String) 方法可以访问系统属性。
    //然后，根据每个 Integer.decode 方法，将该属性的字符串值解释为一个整数值，并返回一个表示该值的 Integer 对象。
    //如果属性值以两个 ASCII 字符 0x 或者 ASCII 字符 # 开始，并且后面没有减号，
    //则将它的剩余部分解析为十六进制整数，就好像以 16 为基数调用 valueOf(java.lang.String, int) 方法一样。
    //如果属性值以 ASCII 字符 0 开始，后面还有其他字符，则将它解析为八进制整数，
    //就好像以 8 为基数调用 valueOf(java.lang.String, int) 方法一样。
    //否则，将属性值解析为十进制整数，就好像以 10 为基数调用 valueOf(java.lang.String, int) 方法一样。
    //第二个参数是默认值。如果未具有指定名称的属性，或者属性的数字格式不正确，或者指定名称为空或 null，则返回默认值。
    public static Integer getInteger(String nm, Integer val) {
        String v = null;
        try {
            v = System.getProperty(nm);
        } catch (IllegalArgumentException e) {
        } catch (NullPointerException e) {
        }
        if (v != null) {
            try {
                return Integer.decode(v);
            } catch (NumberFormatException e) {
            }
        }
        return val;
    }
    
    //将 String 解码为 Integer。接受通过以下语法给出的十进制、十六进制和八进制数字：
    //Java Language Specification 的第 §3.10.1 节中有 DecimalNumeral、 HexDigits 和 OctalDigits 的定义。
    //跟在（可选）负号和/或基数说明符（“0x”、“0X”、“#”或前导零）后面的字符序列是使用指示的基数（10、16 或 8）
    //通过 Integer.parseInt 方法解析的。字符序列必须表示一个正值，否则会抛出 NumberFormatException。
    //如果指定的 String 的第一个字符是减号，则对结果求反。String 中不允许出现空白字符。
    public static Integer decode(String nm) throws NumberFormatException {
        int radix = 10;
        int index = 0;
        boolean negative = false;
        Integer result;

        if (nm.length() == 0)
            throw new NumberFormatException("Zero length string");
        char firstChar = nm.charAt(0);
        // 如果存在符号标志，处理符号标志
        if (firstChar == '-') {
            negative = true;
            index++;
        } else if (firstChar == '+')
            index++;

        // 如果存在基数说明符，处理基数说明符
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
            result = Integer.valueOf(nm.substring(index), radix);
            result = negative ? Integer.valueOf(-result.intValue()) : result;
        } catch (NumberFormatException e) {
            // 如果数字是最小值Integer.MIN_VALUE，在此处结束
            String constant = negative ? ("-" + nm.substring(index))
                                       : nm.substring(index);
            result = Integer.valueOf(constant, radix);
        }
        return result;
    }
    
    //在数字上比较两个 Integer 对象。
    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }
    public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
    
    //用来以二进制补码形式表示 int 值的比特位数。
    public static final int SIZE = 32;
    
    //返回具有至多单个 1 位的 int 值，在指定的 int 值中最高位（最左边）的 1 位的位置。
    //如果指定的值在其二进制补码表示形式中不具有 1 位，即它等于零，则返回零。
    public static int highestOneBit(int i) {
        // HD, Figure 3-1
        i |= (i >>  1);
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        return i - (i >>> 1);
    }
    
    //返回具有至多单个 1 位的 int 值，在指定的 int 值中最低位（最右边）的 1 位的位置。
    //如果指定的值在其二进制补码表示形式中不具有 1 位，即它等于零，则返回零。
    public static int lowestOneBit(int i) {
        // HD, Section 2-1
        return i & -i;
    }
    
    //在指定 int 值的二进制补码表示形式中最高位（最左边）的 1 位之前，返回零位的数量。
    //如果指定值在其二进制补码表示形式中不存在 1 位，换句话说，如果它等于零，则返回 32。
    //注意，此方法与基数为 2 的对数密切相关。对于所有的正 int 值 x：
    //   floor(log2(x)) = 31 - numberOfLeadingZeros(x)
    //   ceil(log2(x)) = 32 - numberOfLeadingZeros(x - 1)
    public static int numberOfLeadingZeros(int i) {
        // HD, Figure 5-6
        if (i == 0)
            return 32;
        int n = 1;
        if (i >>> 16 == 0) { n += 16; i <<= 16; }
        if (i >>> 24 == 0) { n +=  8; i <<=  8; }
        if (i >>> 28 == 0) { n +=  4; i <<=  4; }
        if (i >>> 30 == 0) { n +=  2; i <<=  2; }
        n -= i >>> 31;
        return n;
    }
    
    //返回指定的 int 值的二进制补码表示形式中最低（“最右边”）的为 1 的位后面的零位个数。
    //如果指定值在它的二进制补码表示形式中没有为 1 的位，即它的值为零，则返回 32。
    public static int numberOfTrailingZeros(int i) {
        // HD, Figure 5-14
        int y;
        if (i == 0) return 32;
        int n = 31;
        y = i <<16; if (y != 0) { n = n -16; i = y; }
        y = i << 8; if (y != 0) { n = n - 8; i = y; }
        y = i << 4; if (y != 0) { n = n - 4; i = y; }
        y = i << 2; if (y != 0) { n = n - 2; i = y; }
        return n - ((i << 1) >>> 31);
    }
    
    //返回指定 int 值的二进制补码表示形式的 1 位的数量。此函数有时用于人口普查。
    public static int bitCount(int i) {
        // HD, Figure 5-2
        i = i - ((i >>> 1) & 0x55555555);
        i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
        i = (i + (i >>> 4)) & 0x0f0f0f0f;
        i = i + (i >>> 8);
        i = i + (i >>> 16);
        return i & 0x3f;
    }
    
    //返回根据指定的位数循环左移指定的 int 值的二进制补码表示形式而得到的值。
    //（位是从左边（即高位）移出，从右边（即低位）再进入）
    //注意，使用负距离的左循环等同于右循环：rotateLeft(val, -distance) == rotateRight(val, distance)。
    //还要注意的是，以 32 的任何倍数进行的循环都是无操作指令，因此，即使距离为负，除了最后五位外，
    //其余所有循环距离都可以忽略：rotateLeft(val, distance) == rotateLeft(val, distance & 0x1F)。
    public static int rotateLeft(int i, int distance) {
        return (i << distance) | (i >>> -distance);
    }
    
    //返回根据指定的位数循环右移指定的 int 值的二进制补码表示形式而得到的值。
    //（位是从右边（即低位）移出，从左边（即高位）再进入）
    //注意，使用负距离的右循环等同于左循环：rotateRight(val, -distance) == rotateLeft(val, distance)。
    //还要注意的是，以 32 的任何倍数进行的循环都是无操作指令，因此，即使距离为负，除了最后五位外，
    //其余所有循环距离都可以忽略：rotateRight(val, distance) == rotateRight(val, distance & 0x1F)。
    public static int rotateRight(int i, int distance) {
        return (i >>> distance) | (i << -distance);
    }
    
    //返回通过反转指定 int 值的二进制补码表示形式中位的顺序而获得的值。
    public static int reverse(int i) {
        // HD, Figure 7-1
        i = (i & 0x55555555) << 1 | (i >>> 1) & 0x55555555;
        i = (i & 0x33333333) << 2 | (i >>> 2) & 0x33333333;
        i = (i & 0x0f0f0f0f) << 4 | (i >>> 4) & 0x0f0f0f0f;
        i = (i << 24) | ((i & 0xff00) << 8) |
            ((i >>> 8) & 0xff00) | (i >>> 24);
        return i;
    }
    
    //返回指定 int 值的符号函数。（如果指定值为负，则返回 －1；
    //如果指定值为零，则返回 0；如果指定的值为正，则返回 1）
    public static int signum(int i) {
        // HD, Section 2-7
        return (i >> 31) | (-i >>> 31);
    }
    
    //返回通过反转指定 int 值的二进制补码表示形式中字节的顺序而获得的值。
    public static int reverseBytes(int i) {
        return ((i >>> 24)           ) |
               ((i >>   8) &   0xFF00) |
               ((i <<   8) & 0xFF0000) |
               ((i << 24));
    }
    
    private static final long serialVersionUID = 1360826667806852920L;
  }
```
