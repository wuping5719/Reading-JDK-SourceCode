* ListIterator接口的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
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
     
     //返回列表中的下一个元素。可以重复调用此方法来迭代此列表，或混合调用previous来前后移动.
     //（注意交替调用next和previous将重复返回相同的元素）
     E next();
     
     //如果以逆向遍历列表，列表迭代器有多个元素，则返回true。
     //（换句话说，如果previous返回一个元素而不是抛出异常，则返回true）
     boolean hasPrevious();
     
     //返回列表中的前一个元素。可以重复调用此方法来迭代列表，或混合调用next来前后移动.
     //（注意交替调用next和previous将重复返回相同的元素）
     E previous();
     
     //返回对next的后续调用所返回元素的索引。（如果列表迭代器在列表的结尾，则返回列表的大小）
     int nextIndex();
     
     //返回对previous的后续调用所返回元素的索引。（如果列表迭代器在列表的开始，则返回-1）
     int previousIndex();
     
     //从列表中移除由next或previous返回的最后一个元素(可选操作).对于每个next或previous调用，只能执行一次此调用.
     //只有在最后一次调用next或previous之后，尚未调用ListIterator.add时才可以执行该调用。
     void remove();
     
     //用指定元素替换next或previous返回的最后一个元素(可选操作).
     //只有在最后一次调用next或previous后既没有调用ListIterator.remove也没有调用ListIterator.add时才可以进行该调用.
     void set(E e);
     
     //将指定的元素插入列表(可选操作).该元素直接插入到next返回的下一个元素的前面(如果有)，
     //或者previous返回的下一个元素之后(如果有)；如果列表没有元素，那么新元素就成为列表中的唯一元素。
     //新元素被插入到隐式光标前：不影响对next的后续调用，并且对previous的后续调用会返回此新元素
     //（此调用把调用nextIndex或previousIndex所返回的值增加1）.
     void add(E e);
  }
```
