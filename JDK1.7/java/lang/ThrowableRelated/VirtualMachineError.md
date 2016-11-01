* VirtualMachineError类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)      
  VirtualMachineError类的子类:    
  &nbsp;&nbsp; (1) InternalError类：该异常指示Java虚拟机中出现一些意外的内部错误。      
  &nbsp;&nbsp; (2) OutOfMemoryError类：因为内存溢出或没有可用的内存提供给垃圾回收器时，Java虚拟机无法分配一个对象，这时抛出该异常。       
  &nbsp;&nbsp; (3) StackOverflowError类：当应用程序递归太深而发生堆栈溢出时，抛出该错误。                
  &nbsp;&nbsp; (4) UnknownError类：当Java虚拟机中出现一个未知但严重的异常时，抛出该错误。

```java

```
