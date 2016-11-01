* Math类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  Math类包含用于执行基本数学运算的方法，如初等指数、对数、平方根和三角函数。    
  与StrictMath类的某些数学方法不同，并非Math类所有等价函数的实现都定义为返回逐位相同的结果。此类在不需要严格重复的地方可以得到更好的执行。    
  默认情况下，很多Math方法仅调用StrictMath中的等价方法来完成它们的实现。建议代码生成器使用特定于平台的本机库或者微处理器指令(可用时)来提供Math方法更高性能的实现。这种更高性能的实现仍然必须遵守Math的规范。     
  实现规范的质量涉及到两种属性，即返回结果的准确性和方法的单调性。浮点Math方法的准确性根据ulp(units in the last place，最后一位的进退位)来衡量。对于给定的浮点格式，特定实数值的ulp是包括该数值的两个浮点值的差。当作为一个整体而不是针对具体参数讨论方法的准确性时，引入的ulp数用于任何参数最差情况下的误差。如果一个方法的误差总是小于 0.5 ulp，那么该方法始终返回最接近准确结果的浮点数；这种方法就是正确舍入。一个正确舍入的方法通常能得到最佳的浮点近似值；然而，对于许多浮点方法，进行正确舍入有些不切实际。相反，对于 Math 类，某些方法允许误差在 1 或 2 ulp 的范围内。非正式地，对于 1 ulp 的误差范围，当准确结果是可表示的数值时，应该按照计算结果返回准确结果；否则，返回包括准确结果的两个浮点值中的一个。对于值很大的准确结果，括号的一端可以是无穷大。除了个别参数的准确性之外，维护不同参数的方法之间的正确关系也很重要。因此，大多数误差大于 0.5 ulp 的方法都要求是半单调的：只要数学函数是非递减的，浮点近似值就是非递减的；同样，只要数学函数是非递增的，浮点近似值就是非递增的。并非所有准确性为 1 ulp 的近似值都能自动满足单调性要求。

```java
  public final class Math {
     //不要让任何人实例化这个类
     private Math() {}
     
     //比任何其他值都更接近 e（即自然对数的底数）的 double 值
     public static final double E = 2.7182818284590452354;
     
     //比任何其他值都更接近 pi（即圆的周长与直径之比）的 double 值
     public static final double PI = 3.14159265358979323846;
     
     //返回角的三角正弦
     public static double sin(double a) {
        return StrictMath.sin(a); 
     }
     //返回角的三角余弦
     public static double cos(double a) {
        return StrictMath.cos(a); 
     }
     //返回角的三角正切
     public static double tan(double a) {
        return StrictMath.tan(a); 
     }
     
     //返回一个值的反正弦；返回的角度范围在 -pi/2 到 pi/2 之间
     public static double asin(double a) {
        return StrictMath.asin(a); 
     }
     //返回一个值的反余弦；返回的角度范围在 0.0 到 pi 之间
     public static double acos(double a) {
        return StrictMath.acos(a); 
     }
     //返回一个值的反正切；返回的角度范围在 -pi/2 到 pi/2 之间
     public static double atan(double a) {
        return StrictMath.atan(a); 
     }
     
     //将用角度表示的角转换为近似相等的用弧度表示的角
     public static double toRadians(double angdeg) {
        return angdeg / 180.0 * PI;
     }
     
     //将用弧度表示的角转换为近似相等的用角度表示的角
     public static double toDegrees(double angrad) {
        return angrad * 180.0 / PI;
     }
     
     //返回欧拉数 e 的 double 次幂的值
     public static double exp(double a) {
        return StrictMath.exp(a);
     }
     
     //返回 double 值的自然对数（底数是 e）
     public static double log(double a) {
        return StrictMath.log(a); 
     }
     
     //返回 double 值的底数为 10 的对数
     public static double log10(double a) {
        return StrictMath.log10(a); 
     }
     
     //返回正确舍入的 double 值的正平方根
     public static double sqrt(double a) {
        return StrictMath.sqrt(a); // 注意：频繁的硬件sqrt指令可以直接使用JITS， 它比软件做Math.sqrt快得多
     }
     
     //返回 double 值的立方根
     public static double cbrt(double a) {
        return StrictMath.cbrt(a);
     }
     
     //按照 IEEE 754 标准的规定，对两个参数进行余数运算
     public static double IEEEremainder(double f1, double f2) {
        return StrictMath.IEEEremainder(f1, f2);
     }
     
     //返回最小的（最接近负无穷大）double 值，该值大于等于参数，并等于某个整数
     public static double ceil(double a) {
        return StrictMath.ceil(a); 
     }
     
     //返回最大的（最接近正无穷大）double 值，该值小于等于参数，并等于某个整数
     public static double floor(double a) {
        return StrictMath.floor(a); 
     }
     
     //返回最接近参数并等于某一整数的 double 值
     public static double rint(double a) {
        return StrictMath.rint(a); 
     }
     
     //将矩形坐标 (x, y) 转换成极坐标 (r, theta)，返回所得角 theta
     public static double atan2(double y, double x) {
        return StrictMath.atan2(y, x); 
     }
     
     //返回第一个参数的第二个参数次幂的值
     public static double pow(double a, double b) {
        return StrictMath.pow(a, b); 
     }
     
     //返回最接近参数的 int
     public static int round(float a) {
        if (a != 0x1.fffffep-2f) // 比最大浮点值小0.5
            return (int)floor(a + 0.5f);
        else
            return 0;
     }
     //返回最接近参数的 long
     public static long round(double a) {
        if (a != 0x1.fffffffffffffp-2) // 比最大Double值小0.5
            return (long)floor(a + 0.5d);
        else
            return 0;
     }
     
     //伪随机数生成器
     private static Random randomNumberGenerator;
     
     private static synchronized Random initRNG() {
        Random rnd = randomNumberGenerator;
        return (rnd == null) ? (randomNumberGenerator = new Random()) : rnd;
     }
     
     //返回带正号的伪随机 double 值，该值大于等于 0.0 且小于 1.0
     public static double random() {
        Random rnd = randomNumberGenerator;
        if (rnd == null) rnd = initRNG();
        return rnd.nextDouble();
     }
     
     //返回 int 值的绝对值
     public static int abs(int a) {
        return (a < 0) ? -a : a;
     }
     //返回 long 值的绝对值
     public static long abs(long a) {
        return (a < 0) ? -a : a;
     }
     //返回 float 值的绝对值
     public static float abs(float a) {
        return (a <= 0.0F) ? 0.0F - a : a;
     }
     //返回 double 值的绝对值
     public static double abs(double a) {
        return (a <= 0.0D) ? 0.0D - a : a;
     }
     
     //返回两个 int 值中较大的一个
     public static int max(int a, int b) {
        return (a >= b) ? a : b;
     }
     //返回两个 long 值中较大的一个
     public static long max(long a, long b) {
        return (a >= b) ? a : b;
     }
     private static long negativeZeroFloatBits = Float.floatToIntBits(-0.0f);
     private static long negativeZeroDoubleBits = Double.doubleToLongBits(-0.0d);
     //返回两个 float 值中较大的一个
     public static float max(float a, float b) {
        if (a != a) return a;   // a is NaN
        if ((a == 0.0f) && (b == 0.0f)
            && (Float.floatToIntBits(a) == negativeZeroFloatBits)) {
            return b;
        }
        return (a >= b) ? a : b;
     }
     //返回两个 double 值中较大的一个
     public static double max(double a, double b) {
        if (a != a) return a;   // a is NaN
        if ((a == 0.0d) && (b == 0.0d)
            && (Double.doubleToLongBits(a) == negativeZeroDoubleBits)) {
            return b;
        }
        return (a >= b) ? a : b;
     }
     
     //返回两个 int 值中较小的一个
     public static int min(int a, int b) {
        return (a <= b) ? a : b;
     }
     //返回两个 long 值中较小的一个
     public static long min(long a, long b) {
        return (a <= b) ? a : b;
     }
     //返回两个 float 值中较小的一个
     public static float min(float a, float b) {
        if (a != a) return a;   // a is NaN
        if ((a == 0.0f) && (b == 0.0f)
            && (Float.floatToIntBits(b) == negativeZeroFloatBits)) {
            return b;
        }
        return (a <= b) ? a : b;
     }
     //返回两个 double 值中较小的一个
     public static double min(double a, double b) {
        if (a != a) return a;   // a is NaN
        if ((a == 0.0d) && (b == 0.0d)
            && (Double.doubleToLongBits(b) == negativeZeroDoubleBits)) {
            return b;
        }
        return (a <= b) ? a : b;
     }
     
     //返回参数的 ulp 大小
     public static double ulp(double d) {
        return sun.misc.FpUtils.ulp(d);
     }
     public static float ulp(float f) {
        return sun.misc.FpUtils.ulp(f);
     }

     //返回参数的符号函数；如果参数为 0，则返回 0；如果参数大于 0，则返回 1.0；如果参数小于 0，则返回 -1.0
     public static double signum(double d) {
        return sun.misc.FpUtils.signum(d);
     }
     public static float signum(float f) {
        return sun.misc.FpUtils.signum(f);
     }
     
     //返回 double 值的双曲线正弦
     public static double sinh(double x) {
        return StrictMath.sinh(x);
     }
     //返回 double 值的双曲线余弦
     public static double cosh(double x) {
        return StrictMath.cosh(x);
     }
     //返回 double 值的双曲线余弦
     public static double tanh(double x) {
        return StrictMath.tanh(x);
     }
     
     //返回 sqrt(x2 +y2)，没有中间溢出或下溢
     public static double hypot(double x, double y) {
        return StrictMath.hypot(x, y);
     }
     
     //返回 e^x -1
     public static double expm1(double x) {
        return StrictMath.expm1(x);
     }
     
     //返回参数与 1 之和的自然对数
     public static double log1p(double x) {
        return StrictMath.log1p(x);
     }
     
     //返回带有第二个浮点参数符号的第一个浮点参数
     public static double copySign(double magnitude, double sign) {
        return sun.misc.FpUtils.rawCopySign(magnitude, sign);
     }
     public static float copySign(float magnitude, float sign) {
        return sun.misc.FpUtils.rawCopySign(magnitude, sign);
     }

     //返回 float 表示形式中使用的无偏指数
     public static int getExponent(float f) {
        return sun.misc.FpUtils.getExponent(f);
     }
     //返回 double 表示形式中使用的无偏指数
     public static int getExponent(double d) {
        return sun.misc.FpUtils.getExponent(d);
     }
     
     //返回第一个参数和第二个参数之间与第一个参数相邻的浮点数
     public static double nextAfter(double start, double direction) {
        return sun.misc.FpUtils.nextAfter(start, direction);
     }
     public static float nextAfter(float start, double direction) {
        return sun.misc.FpUtils.nextAfter(start, direction);
     }
     
     //返回 d 和正无穷大之间与 d 相邻的浮点值
     public static double nextUp(double d) {
        return sun.misc.FpUtils.nextUp(d);
     }
     //返回 f 和正无穷大之间与 f 相邻的浮点值
     public static float nextUp(float f) {
        return sun.misc.FpUtils.nextUp(f);
     }
     
     //返回 d × 2^scaleFactor，其舍入方式如同将一个正确舍入的浮点值乘以 double 值集合中的一个值
     public static double scalb(double d, int scaleFactor) {
        return sun.misc.FpUtils.scalb(d, scaleFactor);
     }
     //返回 f × 2^scaleFactor，其舍入方式如同将一个正确舍入的浮点值乘以 float 值集合中的一个值
     public static float scalb(float f, int scaleFactor) {
        return sun.misc.FpUtils.scalb(f, scaleFactor);
     }
  }
```
