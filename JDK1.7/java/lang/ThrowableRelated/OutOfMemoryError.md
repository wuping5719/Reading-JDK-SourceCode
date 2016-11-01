* OutOfMemoryError类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  OutOfMemoryError类: 因为内存溢出或没有可用的内存提供给垃圾回收器时，Java虚拟机无法分配一个对象，这时抛出该异常。
  
```java
  public class OutOfMemoryError extends VirtualMachineError {
    private static final long serialVersionUID = 8228564086184010517L;
    
    //构造不带详细消息的OutOfMemoryError
    public OutOfMemoryError() {
        super();
    }
    
    //构造带指定详细消息的OutOfMemoryError
    public OutOfMemoryError(String s) {
        super(s);
    }
  }
```
