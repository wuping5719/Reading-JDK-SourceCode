* Throwable类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Throwable 类是Java语言中所有错误或异常的超类。只有当对象是此类（或其子类之一）的实例时，才能通过Java虚拟机或者Java throw语句抛出。类似地，只有此类或其子类之一才可以是catch子句中的参数类型。    
  &nbsp;&nbsp; 两个子类的实例: Error和Exception，通常用于指示发生了异常情况。通常，这些实例是在异常情况的上下文中新近创建的，因此包含了相关的信息（比如堆栈跟踪数据）。   
  &nbsp;&nbsp; Exception类及其子类是Throwable的一种形式，它指出了合理的应用程序想要捕获的条件。   
  &nbsp;&nbsp; 注意它有四个构造函数:    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable()     
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(String message)    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(Throwable cause)    
  &nbsp;&nbsp;&nbsp;&nbsp;  Throwable(String message, Throwable cause)
```java
  
```
