* Stack类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Stack类表示后进先出（LIFO）的对象堆栈。它通过五个操作对类Vector进行了扩展 ，允许将向量视为堆栈。它提供了通常的 push 和 pop 操作，以及取堆栈顶点的 peek 方法、测试堆栈是否为空的 empty 方法、在堆栈中查找项并确定到堆栈顶距离的 search 方法。     
  &nbsp;&nbsp; 首次创建堆栈时，它不包含项。     
  &nbsp;&nbsp; Deque 接口及其实现提供了 LIFO 堆栈操作的更完整和更一致的 set，应该优先使用此 set，而非此类。    
  &nbsp;&nbsp; 例如：Deque<Integer> stack = new ArrayDeque<Integer>();
 
```java
  public class Stack<E> extends Vector<E> {
     //创建一个空堆栈
     public Stack() {
     }
     
     //把项压入堆栈顶部。其作用与下面的方法完全相同：addElement(item)
     public E push(E item) {
        addElement(item);
        
        return item;
     }
     
     //移除堆栈顶部的对象，并作为此函数的值返回该对象
     public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
     }
     
     //查看堆栈顶部的对象，但不从堆栈中移除它
     public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
     }
     
     //测试堆栈是否为空
     public boolean empty() {
        return size() == 0;
     }
     
     //返回对象在堆栈中的位置，以 1 为基数。
     //如果对象 o 是堆栈中的一个项，此方法返回距堆栈顶部最近的出现位置到堆栈顶部的距离；
     //堆栈中最顶部项的距离为 1。使用 equals 方法比较 o 与堆栈中的项
     public synchronized int search(Object o) {
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
     }
     
     private static final long serialVersionUID = 1224463164541339165L;
  }
```
