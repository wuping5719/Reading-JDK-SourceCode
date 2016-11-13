* Number抽象类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 抽象类 Number 是 BigDecimal、BigInteger、Byte、Double、Float、Integer、Long 和 Short 类的超类。
  
  &nbsp;&nbsp; Number 的子类必须提供将表示的数值转换为 byte、double、float、int、long 和 short 的方法。
* &nbsp;&nbsp; Java基本数据类型：&nbsp; <http://blog.csdn.net/bingduanlbd/article/details/27790287>      
  &nbsp;&nbsp; 在Java源代码中，每个变量都必须声明一种类型（type）。有两种类型：primitive type和reference type。引用类型引用对象（reference to object），而基本类型直接包含值（directly contain value）。因此，Java数据类型（type）可以分为两大类：基本类型（primitive types）和引用类型（reference types）。primitive types 包括 boolean 类型以及数值类型（numeric types）。numeric types又分为整型（integer types）和浮点型（floating-point types）。整型有5种：byte short int long char(char本质上是一种特殊的int)。浮点类型有 float 和 double。关系整理如下图：
 <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_JavaDataType.png" /></p>

```java
  public abstract class Number implements java.io.Serializable {
    //以 int 形式返回指定的数值。这可能会涉及到舍入或取整。
    public abstract int intValue();
    
    //以 long 形式返回指定的数值。这可能涉及到舍入或取整。
    public abstract long longValue();
    
    //以 float 形式返回指定的数值。这可能会涉及到舍入。
    public abstract float floatValue();
    
    //以 double 形式返回指定的数值。这可能会涉及到舍入。
    public abstract double doubleValue();
    
    //以 byte 形式返回指定的数值。这可能会涉及到舍入或取整。
    public byte byteValue() {
        return (byte)intValue();
    }
    
    //以 short 形式返回指定的数值。这可能会涉及到舍入或取整。
    public short shortValue() {
        return (short)intValue();
    }
    
    private static final long serialVersionUID = -8742448824652078965L;
  }
```
