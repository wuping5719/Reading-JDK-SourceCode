* ListIterator类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  列表迭代器，允许程序员按任一方向遍历列表、迭代期间修改列表，并获得迭代器在列表中的当前位置。    
  ListIterator没有当前元素；它的光标位置始终位于调用previous()所返回的元素和调用next()所返回的元素之间。      
  长度为n的列表的迭代器有n+1个可能的指针位置，如下面的插入符举例说明：                       
  cursor positions:  ^ Element(0) ^ Element(1) ^ Element(2) ^ ... ^ Element(n-1)        
  注意：remove()和set(Object)方法不是根据光标位置定义的；它们是根据对调用next()或previous()所返回的最后一个元素的操作定义的。    
  此接口是Java Collections Framework的成员。

```java
  public interface ListIterator<E> extends Iterator<E> {
     //以正向遍历列表时，如果列表迭代器有多个元素，则返回true    
     //（换句话说，如果next返回一个元素而不是抛出异常，则返回true）
     boolean hasNext();
  }
```
