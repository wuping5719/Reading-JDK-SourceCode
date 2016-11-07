* Iterator接口的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  对collection进行迭代的迭代器。迭代器取代了Java Collections Framework中的Enumeration。
  迭代器与枚举有两点不同：       
     迭代器允许调用者利用定义良好的语义在迭代期间从迭代器所指向的collection移除元素。             
     方法名称得到了改进。
  此接口是Java Collections Framework的成员。
 
```java
  public interface Iterator<E> {
     //如果仍有元素可以迭代，则返回true。（换句话说，如果next返回了元素而不是抛出异常，则返回true）
     boolean hasNext();
     
     //返回迭代的下一个元素
     E next();
     
     //从迭代器指向的collection中移除迭代器返回的最后一个元素（可选操作）。
     //每次调用next只能调用一次此方法。
     //如果进行迭代时用调用此方法之外的其他方式修改了该迭代器所指向的collection，则迭代器的行为是不确定的。
     void remove();
  }
```
