* Object类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  类Object是类层次结构的根类。每个类都使用Object作为超类。所有对象（包括数组）都实现这个类的方法。
  
```java
  public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }
    
    //返回此Object的运行时类。返回的Class对象是由所表示类的static synchronized方法锁定的对象。
    //实际结果类型是Class<? extends |X|>，其中 |X| 表示清除表达式中的静态类型，该表达式调用getClass。 
    //例如，以下代码片段中不需要强制转换：
    //    Number n = 0; 
    //    Class<? extends Number> c = n.getClass();
    //返回：表示此对象运行时类的Class对象。
    public final native Class<?> getClass();
    
    //返回该对象的哈希码值。支持此方法是为了提高哈希表（例如:java.util.Hashtable提供的哈希表）的性能。
    //hashCode的常规协定是：
    //在Java应用程序执行期间，在对同一对象多次调用hashCode方法时，必须一致地返回相同的整数，
    //前提是将对象进行equals比较时所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。
    //如果根据equals(Object)方法，两个对象是相等的，那么对这两个对象中的每个对象调用hashCode方法都必须生成相同的整数结果。
    //如果根据equals(Object)方法，两个对象不相等，那么对这两个对象中的任一对象上调用hashCode方法不要求一定生成不同的整数结果。
    //但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。
    //实际上，由Object类定义的hashCode方法确实会针对不同的对象返回不同的整数。
    //（这一般是通过将该对象的内部地址转换成一个整数来实现的，但是JavaTM编程语言不需要这种实现技巧。）
    //返回：此对象的一个哈希码值。
    public native int hashCode();
    
    //指示其他某个对象是否与此对象“相等”
    public boolean equals(Object obj) {
        return (this == obj);
    }
  }
```
