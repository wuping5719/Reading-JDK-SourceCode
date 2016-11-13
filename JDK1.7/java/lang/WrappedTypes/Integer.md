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
    
    
  }
```
