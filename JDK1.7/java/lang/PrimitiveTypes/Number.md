* Number抽象类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 抽象类 Number 是 BigDecimal、BigInteger、Byte、Double、Float、Integer、Long 和 Short 类的超类。
  &nbsp;&nbsp; Number 的子类必须提供将表示的数值转换为 byte、double、float、int、long 和 short 的方法。
* &nbsp;&nbsp; Java基本数据类型：&nbsp; <http://blog.csdn.net/bingduanlbd/article/details/27790287>
 <p><img src="http://images.cnblogs.com/cnblogs_com/wp5719/831982/o_JavaDataType.png" /></p>

```java
  public abstract class Number implements java.io.Serializable {
    
  }
```
