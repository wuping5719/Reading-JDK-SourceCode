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
    //返回得到的 long 值，该值与用该参数和基数 10 作为参数的 parseLong(java.lang.String, int) 方法得到的值非常相似。
    //注意，不允许将字符 L ('\u004C') 和 l ('\u006C') 作为类型指示符出现在字符串的结尾处，
    //这一点在 Java 编程语言源代码中是允许的。
    public static long parseLong(String s) throws NumberFormatException {
        return parseLong(s, 10);
    }
    
  }
```
