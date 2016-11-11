* Thread类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 线程是程序中的执行线程。Java 虚拟机允许应用程序并发地运行多个执行线程。

  &nbsp;&nbsp; 每个线程都有一个优先级，高优先级线程的执行优先于低优先级线程。每个线程都可以或不可以标记为一个守护程序。当某个线程中运行的代码创建一个新 Thread 对象时，该新线程的初始优先级被设定为创建线程的优先级，并且当且仅当创建线程是守护线程时，新线程才是守护程序。

  &nbsp;&nbsp; 当 Java 虚拟机启动时，通常都会有单个非守护线程（它通常会调用某个指定类的 main 方法）。Java 虚拟机会继续执行线程，直到下列任一情况出现时为止：  
  &nbsp;&nbsp; 1.调用了 Runtime 类的 exit 方法，并且安全管理器允许退出操作发生。         
  &nbsp;&nbsp; 2.非守护线程的所有线程都已停止运行，无论是通过从对 run 方法的调用中返回，还是通过抛出一个传播到 run 方法之外的异常。
  
  &nbsp;&nbsp; 创建新执行线程有两种方法。一种方法是将类声明为 Thread 的子类。该子类应重写 Thread 类的 run 方法。接下来可以分配并启动该子类的实例。例如，计算大于某一规定值的质数的线程可以写成：
  
```java
  class PrimeThread extends Thread {
         long minPrime;
         PrimeThread(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
             ...
         }
  }
```  

 &nbsp;&nbsp;  然后，下列代码会创建并启动一个线程：
 
```java
  class PrimeRun implements Runnable {
         long minPrime;
         PrimeRun(long minPrime) {
             this.minPrime = minPrime;
         }
 
         public void run() {
             // compute primes larger than minPrime
             ...
         }
     }
```    

  &nbsp;&nbsp; 创建线程的另一种方法是声明实现 Runnable 接口的类。该类然后实现 run 方法。然后可以分配该类的实例，在创建 Thread 时作为一个参数来传递并启动。采用这种风格的同一个例子如下所示：
  
```java
  PrimeThread p = new PrimeThread(143);
  p.start();
```       

  &nbsp;&nbsp; 然后，下列代码会创建并启动一个线程：
  
```java
  PrimeRun p = new PrimeRun(143);
  new Thread(p).start();
``` 
 
  &nbsp;&nbsp; 每个线程都有一个标识名，多个线程可以同名。如果线程创建时没有指定标识名，就会为其生成一个新名称。
  
```java
  
```
