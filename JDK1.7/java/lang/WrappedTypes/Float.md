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
    
    //返回 float 参数的十六进制字符串表示形式。下面提到的所有字符都是 ASCII 字符。
    //如果参数为 NaN，那么结果是字符串 "NaN"。
    //否则，结果是表示参数的符号和数值（绝对值）的字符串。
    //如果符号为负，那么结果的第一个字符是 '-' ('\u002D')；如果符号为正，则结果中不显示符号字符。
    //至于数值 m：
    //   如果 m 为无穷大，则用字符串 "Infinity" 表示；
    // 因此，正无穷大生成结果 "Infinity"，负无穷大生成结果 "-Infinity"。
    //   如果 m 为 0，则用字符串 "0x0.0p0" 表示；因此，负 0 生成结果 "-0x0.0p0"，正 0 生成结果 "0x0.0p0"。
    //   如果 m 是具有标准化表示形式的 float 值，则使用子字符串表示有效位数和指数。
    // 有效位数用字符串 "0x1." 表示，后跟该有效位数小数部分的小写十六进制表示形式。
    // 除非所有位数都为 0，否则移除十六进制表示形式中的尾部 0，在所有位数为 0 的情况下，可以用一个 0 表示。
    // 然后用 "p" 表示指数，后跟无偏指数的十进制字符串，该值与对指数值调用 Integer.toString 生成的值相同。
    //   如果 m 是具有 subnormal 表示形式的 float 值，则用字符 "0x0." 表示有效位数，
    // 后跟该有效位数小数部分的十六进制表示形式。移除十六进制表示形式中的尾部 0。然后用 "p-126" 表示指数。
    // 注意，在 subnormal 有效位数中，至少必须有一个非 0 位数。
    public static String toHexString(float f) {
        if (Math.abs(f) < FloatConsts.MIN_NORMAL
            &&  f != 0.0f ) {
            String s = Double.toHexString(FpUtils.scalb((double)f,
                                                        /* -1022+126 */
                                                        DoubleConsts.MIN_EXPONENT-
                                                        FloatConsts.MIN_EXPONENT));
            return s.replaceFirst("p-1022$", "p-126");
        }
        else // double string will be the same as float string
            return Double.toHexString(f);
    }
    
    //返回保存用参数字符串 s 表示的 float 值的 Float 对象。
    //如果 s 为 null，则抛出 NullPointerException 异常。
    //忽略 s 中的前导空白字符和尾部空白字符。就像调用 String.trim() 方法那样移除空白；
    //也就是说，ASCII 空格和控制字符都要移除。s 的其余部分应该根据词法语法规则描述构成 FloatValue：
    //其中， Sign、 FloatingPointLiteral、 HexNumeral、 HexDigits、 SignedInteger 
    //和 FloatTypeSuffix 与 Java Language Specification 的词法结构部分中的定义相同。
    //如果 s 的表示形式不是 FloatValue，则抛出 NumberFormatException。
    //否则，可以认为 s 表示的是常用“计算机科学记数法”表示的精确十进制值，或者是一个精确的十六进制值；
    //在概念上，这个精确的数值被转换一个“无限精确的”二进制值，
    //然后根据常用 IEEE 754 浮点算法的“舍入为最接近的数”规则将该值舍入为 float 类型，其中包括保留 0 值的符号。
    //最后，返回表示这个 float 值的 Float 对象。
    //要解释浮点值的本地化字符串表示形式，请使用 NumberFormat 的子类。
    //注意，尾部格式说明符、确定浮点字面值类型的说明符（1.0f 是一个 float 值；1.0d 是一个 double 值）
    //不会影响此方法的结果。换句话说，输入字符串的数值被直接转换为目标浮点类型。
    //通常，分两步的转换（先将字符串转换为 double 类型，然后将 double 类型转换为 float 类型）
    //不同于直接将字符串转换为 float 类型。例如，如果首先转换为中间类型 double，然后再转换为 float 类型，
    //则字符串"1.00000017881393421514957253748434595763683319091796875001d"
    //将得到 float 值 1.0000002f；如果直接将字符串转换为 float 值，则结果将是 1.0000001f。
    //为了避免对无效字符串调用此方法并导致抛出 NumberFormatException，
    //Double.valueOf 的文档中列出了一个正则表达式，可以用该表达式在屏幕上显示输入。
    public static Float valueOf(String s) throws NumberFormatException {
        return new Float(FloatingDecimal.readJavaFormatString(s).floatValue());
    }
    
    //返回表示指定的 float 值的 Float 实例。
    //如果不需要新的 Float 实例，则通常应优先使用此方法，而不是构造方法 Float(float)，
    //因为此方法可能通过缓存经常请求的值来显著提高空间和时间性能。
    public static Float valueOf(float f) {
        return new Float(f);
    }
    
    //返回一个新的 float 值，该值被初始化为用指定 String 表示的值，这与 Float 类的 valueOf 方法一样。
    public static float parseFloat(String s) throws NumberFormatException {
        return FloatingDecimal.readJavaFormatString(s).floatValue();
    }
    
    //如果指定的数是一个非数字 (NaN) 值，则返回 true，否则返回 false。
    static public boolean isNaN(float v) {
        return (v != v);
    }
    
    //如果指定数的数值是无穷大，则返回 true，否则返回 false。
    static public boolean isInfinite(float v) {
        return (v == POSITIVE_INFINITY) || (v == NEGATIVE_INFINITY);
    }
    
    //Float的值
    private final float value;
    
    //构造一个新分配的 Float 对象，它表示基本的 float 参数。
    public Float(float value) {
        this.value = value;
    }
    
    //构造一个新分配的 Float 对象，它表示转换为 float 类型的参数。
    public Float(double value) {
        this.value = (float)value;
    }
    
    //构造一个新分配的 Float 对象，它表示用字符串表示的 float 类型的浮点值。
    //字符串将被转换为 float 值，这与 valueOf 方法一样。
    public Float(String s) throws NumberFormatException {
        // 提醒:这是低效的
        this(valueOf(s).floatValue());
    }
    
    //如果此 Float 值是一个非数字 (NaN) 值，则返回 true，否则返回 false。
    public boolean isNaN() {
        return isNaN(value);
    }
    
    //如果此 Float 值的大小是无穷大，则返回 true，否则返回 false。
    public boolean isInfinite() {
        return isInfinite(value);
    }
    
    //返回此 Float 对象的字符串表示形式。
    //使用此对象表示的基本 float 值被转换为一个 String，此方法与带一个参数的 toString 方法完全一样。
    public String toString() {
        return Float.toString(value);
    }
    
    //将此 Float 值以 byte 形式返回（强制转换为 byte）。
    public byte byteValue() {
        return (byte)value;
    }
    
    //将此 Float 值以 short 形式返回（强制转换为 short）。
    public short shortValue() {
        return (short)value;
    }
    
    //将此 Float 值以 int 形式返回（强制转换为 int 类型）。
    public int intValue() {
        return (int)value;
    }
    
    //将此 Float 值以 long 形式返回（强制转换为 long 类型）。
    public long longValue() {
        return (long)value;
    }
    
    //返回此 Float 对象的 float 值。
    public float floatValue() {
        return value;
    }
    
    //返回此 Float 对象的 double 值。
    public double doubleValue() {
        return (double)value;
    }
    
    //返回此 Float 对象的哈希码。结果是此 Float 对象表示的基本 float 值的整数位表示形式，
    //与 floatToIntBits(float) 方法生成的结果完全一样。
    public int hashCode() {
        return floatToIntBits(value);
    }
    
    //将此对象与指定对象进行比较。当且仅当参数不是 null 而是 Float 对象，
    //且表示的 float 值与此对象表示的 float 值相同时，结果为 true。
    //为此，当且仅当将方法 #floatToLongBits(double) 应用于两个值所返回的 int 值相同时，
    //才认为这两个 float 值相同。
    //注意，在大多数情况下，对于 Float 类的两个实例 f1 和 f2，当且仅当
    // f1.floatValue() == f2.floatValue() 
    //的值为 true 时，f1.equals(f2) 的值才为 true。 但是，有以下两种例外情况：
    //如果 f1 和 f2 都表示 Float.NaN，那么即使 Float.NaN==Float.NaN 的值为 false，
    //equals 方法也将返回 true。
    //如果 f1 表示 +0.0f，而 f2 表示 -0.0f，或相反，那么即使 0.0f==-0.0f 的值为 true，
    //equal 测试也将返回 false。
    //该定义使得哈希表得以正确操作。
    public boolean equals(Object obj) {
        return (obj instanceof Float)
               && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
    }
    
    //根据 IEEE 754 浮点“单一格式”位布局，返回指定浮点值的表示形式。
    //第 31 位（掩码 0x80000000 选定的位）表示浮点数的符号。第 30-23 位（掩码 0x7f800000 选定的位）表示指数。
    //第 22-0 位（掩码 0x007fffff 选定的位）表示浮点数的有效位数（有时也称为尾数）。
    //如果参数为正无穷大，则结果为 0x7f800000。
    //如果参数为负无穷大，则结果为 0xff800000。
    //如果参数为 NaN，则结果为 0x7fc00000。
    //在所有情况下，结果都是一个整数，将其赋予 intBitsToFloat(int) 方法将生成一个浮点值，
    //该浮点值与 floatToIntBits 的参数相同（而所有 NaN 值则会生成一个“规范”NaN 值）。
    public static int floatToIntBits(float value) {
        int result = floatToRawIntBits(value);
        // Check for NaN based on values of bit fields, maximum exponent and nonzero significand.
        if ( ((result & FloatConsts.EXP_BIT_MASK) == FloatConsts.EXP_BIT_MASK) &&
             (result & FloatConsts.SIGNIF_BIT_MASK) != 0)
            result = 0x7fc00000;
        return result;
    }
    
    //根据 IEEE 754 浮点“单一格式”位布局，返回指定浮点值的表示形式，并保留非数字 (NaN) 值。
    //第 31 位（掩码 0x80000000 选定的位）表示浮点数的符号。
    //第 30-23 位（掩码 0x7f800000 选定的位）表示指数。
    //第 22-0 位（掩码 0x007fffff 选定的位）表示浮点数的有效位数（有时也称为尾数）。
    //如果参数为正无穷大，则结果为 0x7f800000。
    //如果参数为负无穷大，则结果为 0xff800000。
    //如果参数为 NaN，则结果是表示实际 NaN 值的整数。
    //与 floatToIntBits 方法不同，floatToRawIntBits 不压缩所有将 NaN 编码为一个“规范”NaN 值的位模式。
    //在所有情况下，结果都是一个整数，
    //将其赋予 intBitsToFloat(int) 方法将生成一个与 floatToRawIntBits 的参数相同的浮点值。
    public static native int floatToRawIntBits(float value);
    
    //返回对应于给定位表示形式的 float 值。根据 IEEE 754 浮点“单一格式”位布局，该参数被视为浮点值表示形式。
    //如果参数为 0x7f800000，则结果为正无穷大。
    //如果参数为 0xff800000，则结果为负无穷大。
    //如果参数值在 0x7f800001 到 0x7fffffff 或在 0xff800001 到 0xffffffff 之间，则结果为 NaN。
    //Java 提供的任何 IEEE 754 浮点操作都不能区分具有不同位模式的两个同类型 NaN 值。
    //不同的 NaN 值只能使用 Float.floatToRawIntBits 方法区分。
    //在所有其他情况下，设 s、e 和 m 为可以通过以下参数进行计算的三个值；
    //   int s = ((bits >> 31) == 0) ? 1 : -1;
    //   int e = ((bits >> 23) & 0xff);
    //   int m = (e == 0) ? (bits & 0x7fffff) << 1 : (bits & 0x7fffff) | 0x800000;
    //那么浮点结果等于算术表达式 s· m·2^(e^-150) 的值。
    //注意，此方法不能返回与 int 参数具有完全相同位模式的 float NaN 值。
    //IEEE 754 区分了两种 NaN：quiet NaN 和 signaling NaN。这两种 NaN 之间的差别在 Java 中通常是不可见的。
    //对 signaling NaN 进行的算术运算将它们转换为具有不同（但通常类似）位模式的 quiet NaN。
    //但在某些只复制 signaling NaN 的处理器上也执行这种转换。
    //特别是在复制 signaling NaN 以将其返回给调用方法时，可能会执行这种转换。
    //因此，intBitsToFloat 可能无法返回具有 signaling NaN 位模式的 float 值。
    //所以，对于某些 int 值，floatToRawIntBits(intBitsToFloat(start)) 可能不等于 start。
    //此外，尽管所有 NaN 位模式（不管是 quiet NaN 还是 signaling NaN）都必须在前面确定的 NaN 范围内，
    //但表示 signaling NaN 的特定位模式是与平台有关。
    public static native float intBitsToFloat(int bits);
    
    //比较两个 Float 对象所表示的数值。
    //在应用到基本 float 值时，有两种方法可以比较执行此方法产生的值与执行
    // Java 语言的数字比较运算符（ <、<=、== 和 >= >）产生的那些值之间的区别：
    //此方法认为 Float.NaN 等于其自身，且大于其他所有 float 值（包括 Float.POSITIVE_INFINITY）。
    //此方法认为 0.0f 大于 -0.0f。
    //这可以确保受此方法影响的 Float 对象的自然顺序 与 equals 一致。
    public int compareTo(Float anotherFloat) {
        return Float.compare(value, anotherFloat.value);
    }
    
    //比较两个指定的 float 值。返回整数值的符号与以下调用返回整数的符号相同：
    //  new Float(f1).compareTo(new Float(f2))
    public static int compare(float f1, float f2) {
        if (f1 < f2)
            return -1;           // Neither val is NaN, thisVal is smaller
        if (f1 > f2)
            return 1;            // Neither val is NaN, thisVal is larger

        // Cannot use floatToRawIntBits because of possibility of NaNs.
        int thisBits    = Float.floatToIntBits(f1);
        int anotherBits = Float.floatToIntBits(f2);

        return (thisBits == anotherBits ?  0 : // Values are equal
                (thisBits < anotherBits ? -1 : // (-0.0, 0.0) or (!NaN, NaN)
                 1));                          // (0.0, -0.0) or (NaN, !NaN)
    }
    
    private static final long serialVersionUID = -2671257302660747028L;
  } 
```
