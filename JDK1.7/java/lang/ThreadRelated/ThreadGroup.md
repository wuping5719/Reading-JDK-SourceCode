* ThreadGroup类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; 线程组表示一个线程的集合。此外，线程组也可以包含其他线程组。线程组构成一棵树，在树中，除了初始线程组外，每个线程组都有一个父线程组。

  &nbsp;&nbsp; 允许线程访问有关自己的线程组的信息，但是不允许它访问有关其线程组的父线程组或其他任何线程组的信息。

```java
 public class ThreadGroup implements Thread.UncaughtExceptionHandler {
    private final ThreadGroup parent;  //父线程组
    String name;                       //线程组名
    int maxPriority;                   //最高优先级
    boolean destroyed;                 //是否已经被销毁
    boolean daemon;                    //是否为后台程序线程组
    boolean vmAllowSuspension;         //是否允许VM在内存不足时隐式挂起该线程组

    int nUnstartedThreads = 0;         //线程组未启动的线程数
    int nthreads;                      //线程组的线程数
    Thread threads[];                  //线程组的线程数组

    int ngroups;                       //若拥有子线程组，子线程组数
    ThreadGroup groups[];              //子线程数组
    
    //创建一个不是在任何线程组中的空线程组。
    //该方法用于创建系统线程组。
    private ThreadGroup() {  // C 代码调用
        this.name = "system";
        this.maxPriority = Thread.MAX_PRIORITY;
        this.parent = null;
    }
    
    //构造一个新线程组。新线程组的父线程组是目前正在运行线程的线程组。
    //不使用任何参数调用父线程组的 checkAccess 方法；这可能导致一个安全性异常
    public ThreadGroup(String name) {
        this(Thread.currentThread().getThreadGroup(), name);
    }
    
    //创建一个新线程组。新线程组的父线程组是指定的线程组。
    //不使用任何参数调用父线程组的 checkAccess 方法；这可能导致一个安全性异常。
    public ThreadGroup(ThreadGroup parent, String name) {
        this(checkParentAccess(parent), parent, name);
    }
    
    private ThreadGroup(Void unused, ThreadGroup parent, String name) {
        this.name = name;
        this.maxPriority = parent.maxPriority;
        this.daemon = parent.daemon;
        this.vmAllowSuspension = parent.vmAllowSuspension;
        this.parent = parent;
        parent.add(this);
    }
    
    private static Void checkParentAccess(ThreadGroup parent) {
        parent.checkAccess();
        return null;
    }
    
    //返回此线程组的名称。
    public final String getName() {
        return name;
    }
    
    //返回此线程组的父线程组。
    //如果父线程组不为 null，则不使用任何参数直接调用父线程组的 checkAccess 方法；这可能导致一个安全性异常。
    public final ThreadGroup getParent() {
        if (parent != null)
            parent.checkAccess();
        return parent;
    }
    
    //返回此线程组的最高优先级。作为此线程组一部分的线程不能拥有比最高优先级更高的优先级。
    public final int getMaxPriority() {
        return maxPriority;
    }
    
    //测试此线程组是否为一个后台程序线程组。
    //在停止后台程序线程组的最后一个线程或销毁其最后一个线程组时，自动销毁这个后台程序线程组。
    public final boolean isDaemon() {
        return daemon;
    }
    
    //测试此线程组是否已经被销毁。
    public synchronized boolean isDestroyed() {
        return destroyed;
    }
    
    //更改此线程组的后台程序状态。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //在停止后台程序线程组的最后一个线程或销毁其最后一个线程组时，自动销毁此后台程序线程组。
    public final void setDaemon(boolean daemon) {
        checkAccess();
        this.daemon = daemon;
    }
    
    //设置线程组的最高优先级。线程组中已有较高优先级的线程不受影响。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //如果 pri 参数小于 Thread.MIN_PRIORITY 或大于 Thread.MAX_PRIORITY，则线程组的最高优先级保持不变。
    //否则，此ThreadGroup对象的优先级被设置为比指定的pri参数更小，所允许的最高优先级是此线程组的父线程组的优先级
    //（如果此线程组是系统线程组，没有父线程组，那么只需将最高优先级设置为 pri 即可）
    //然后使用 pri 作为此方法的参数，以递归的方式对属于此线程组的每个线程组调用此方法。
    public final void setMaxPriority(int pri) {
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot;
        synchronized (this) {
            checkAccess();
            if (pri < Thread.MIN_PRIORITY || pri > Thread.MAX_PRIORITY) {
                return;
            }
            maxPriority = (parent != null) ? Math.min(pri, parent.maxPriority) : pri;
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
        }
        for (int i = 0 ; i < ngroupsSnapshot ; i++) {
            groupsSnapshot[i].setMaxPriority(pri);
        }
    }
    
    //测试此线程组是否为线程组参数或其祖先线程组之一。
    public final boolean parentOf(ThreadGroup g) {
        for (; g != null ; g = g.parent) {
            if (g == this) {
                return true;
            }
        }
        return false;
    }
    
    //确定当前运行的线程是否有权修改此线程组。
    //如果有安全管理器，则用此线程组作为其参数调用checkAccess方法。结果可能是抛出一个SecurityException
    public final void checkAccess() {
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkAccess(this);
        }
    }
    
    //返回此线程组中活动线程的估计数。结果并不能反映并发活动，并且可能受某些系统线程的存在状态的影响。
    //由于结果所固有的不精确特性，建议只将此方法用于信息显示。
    public int activeCount() {
        int result;
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot;
        synchronized (this) {
            if (destroyed) {
                return 0;
            }
            result = nthreads;
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
        }
        for (int i = 0 ; i < ngroupsSnapshot ; i++) {
            result += groupsSnapshot[i].activeCount();
        }
        return result;
    }
    
    //把此线程组及其子组中的所有活动线程复制到指定数组中。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //应用程序可以使用activeCount方法获取数组大小的估计数，如果数组太小而无法保持所有线程，则忽略额外的线程。
    //如果获得此线程组及其子组中的所有活动线程非常重要，则调用方应该验证返回的int值是否严格小于list的长度。
    //由于使用此方法所固有的竞争条件，建议只将此方法用于信息显示。
    public int enumerate(Thread list[]) {
        checkAccess();
        return enumerate(list, 0, true);
    }
    
    //把此线程组中的所有活动线程复制到指定数组中。
    //如果 recurse 标志为 true，则还包括对此线程的子组中的所有活动线程的引用。
    //如果数组太小而无法保持所有线程，则忽略额外的线程。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //应用程序可以使用activeCount获取数组大小的估计数，如果数组太小而无法保持所有线程，则忽略额外的线程。
    //如果获得此线程组中的所有活动线程非常重要，则调用方应该验证返回的整数值是否确实小于 list 的长度。
    //由于使用此方法所固有的竞争条件，建议只将此方法用于信息显示。
    public int enumerate(Thread list[], boolean recurse) {
        checkAccess();
        return enumerate(list, 0, recurse);
    }
    
    private int enumerate(Thread list[], int n, boolean recurse) {
        int ngroupsSnapshot = 0;
        ThreadGroup[] groupsSnapshot = null;
        synchronized (this) {
            if (destroyed) {
                return 0;
            }
            int nt = nthreads;
            if (nt > list.length - n) {
                nt = list.length - n;
            }
            for (int i = 0; i < nt; i++) {
                if (threads[i].isAlive()) {
                    list[n++] = threads[i];
                }
            }
            if (recurse) {
                ngroupsSnapshot = ngroups;
                if (groups != null) {
                    groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
                } else {
                    groupsSnapshot = null;
                }
            }
        }
        if (recurse) {
            for (int i = 0 ; i < ngroupsSnapshot ; i++) {
                n = groupsSnapshot[i].enumerate(list, n, true);
            }
        }
        return n;
    }
    
    //返回此线程组中活动线程组的估计数。结果并不能反映并发活动。
    //由于结果所固有的不精确特性，建议只将此方法用于信息显示。
    public int activeGroupCount() {
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot;
        synchronized (this) {
            if (destroyed) {
                return 0;
            }
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
        }
        int n = ngroupsSnapshot;
        for (int i = 0 ; i < ngroupsSnapshot ; i++) {
            n += groupsSnapshot[i].activeGroupCount();
        }
        return n;
    }
    
    //把对此线程组中的所有活动子组的引用复制到指定数组中。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //应用程序可以使用activeGroupCount获取数组大小的估计数，如果数组太小而无法保持所有线程组，则忽略额外的线程组。
    //如果获得此线程组中的所有活动子组非常重要，则调用方应该验证返回的整数值是否确实小于 list 的长度。
    //由于使用此方法所固有的竞争条件，建议只将此方法用于信息显示。
    public int enumerate(ThreadGroup list[]) {
        checkAccess();
        return enumerate(list, 0, true);
    }
    
    //把对此线程组中的所有活动子组的引用复制到指定数组中。
    //如果 recurse 标志为 true，则还包括对子组的所有活动子组的引用。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //应用程序可以使用activeGroupCount获取数组大小的估计数，如果数组太小而无法保持所有线程组，则忽略额外的线程组。
    //如果获得此线程组中的所有活动子组非常重要，则调用方应该验证返回的整数值是否确实小于 list 的长度。
    //由于使用此方法所固有的竞争条件，建议只将此方法用于信息目的。
    public int enumerate(ThreadGroup list[], boolean recurse) {
        checkAccess();
        return enumerate(list, 0, recurse);
    }
    
    private int enumerate(ThreadGroup list[], int n, boolean recurse) {
        int ngroupsSnapshot = 0;
        ThreadGroup[] groupsSnapshot = null;
        synchronized (this) {
            if (destroyed) {
                return 0;
            }
            int ng = ngroups;
            if (ng > list.length - n) {
                ng = list.length - n;
            }
            if (ng > 0) {
                System.arraycopy(groups, 0, list, n, ng);
                n += ng;
            }
            if (recurse) {
                ngroupsSnapshot = ngroups;
                if (groups != null) {
                    groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
                } else {
                    groupsSnapshot = null;
                }
            }
        }
        if (recurse) {
            for (int i = 0 ; i < ngroupsSnapshot ; i++) {
                n = groupsSnapshot[i].enumerate(list, n, true);
            }
        }
        return n;
    }
    
    //已过时。此方法具有固有的不安全性。有关详细信息，请参阅 Thread.stop()。
    //停止此线程组中的所有线程。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //此方法将对此线程组及其所有子组中的所有线程调用 stop 方法。
    @Deprecated
    public final void stop() {
        if (stopOrSuspend(false))
            Thread.currentThread().stop();
    }
    
    //中断此线程组中的所有线程。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //此方法将对此线程组及其所有子组中的所有线程调用 interrupt 方法。
    public final void interrupt() {
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot;
        synchronized (this) {
            checkAccess();
            for (int i = 0 ; i < nthreads ; i++) {
                threads[i].interrupt();
            }
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
        }
        for (int i = 0 ; i < ngroupsSnapshot ; i++) {
            groupsSnapshot[i].interrupt();
        }
    }
    
    //已过时。此方法容易导致死锁。有关详细信息，请参阅 Thread.suspend()。
    //挂起此线程组中的所有线程。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //此方法将对该线程组及其所有子组中的所有线程调用 suspend 方法。
    @Deprecated
    public final void suspend() {
        if (stopOrSuspend(true))
            Thread.currentThread().suspend();
    }
    
    //辅助方法:递归地停止或挂起
    private boolean stopOrSuspend(boolean suspend) {
        boolean suicide = false;
        Thread us = Thread.currentThread();
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot = null;
        synchronized (this) {
            checkAccess();
            for (int i = 0 ; i < nthreads ; i++) {
                if (threads[i]==us)
                    suicide = true;
                else if (suspend)
                    threads[i].suspend();
                else
                    threads[i].stop();
            }

            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            }
        }
        for (int i = 0 ; i < ngroupsSnapshot ; i++)
            suicide = groupsSnapshot[i].stopOrSuspend(suspend) || suicide;

        return suicide;
    }
    
    //已过时。 此方法只用于联合 Thread.suspend 和 ThreadGroup.suspend 时，
    //因为它们所固有的容易导致死锁的特性，所以两者都已废弃。有关详细信息，请参阅Thread.suspend()。
    //继续此线程组中的所有线程。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    //此方法将对该线程组及其所有子组中的所有线程调用 resume 方法。
    @Deprecated
    public final void resume() {
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot;
        synchronized (this) {
            checkAccess();
            for (int i = 0 ; i < nthreads ; i++) {
                threads[i].resume();
            }
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
        }
        for (int i = 0 ; i < ngroupsSnapshot ; i++) {
            groupsSnapshot[i].resume();
        }
    }
    
    //销毁此线程组及其所有子组。此线程组必须为空，指示此线程组中的所有线程都已停止执行。
    //不使用任何参数调用此线程组的 checkAccess 方法；这可能导致一个安全性异常。
    public final void destroy() {
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot;
        synchronized (this) {
            checkAccess();
            if (destroyed || (nthreads > 0)) {
                throw new IllegalThreadStateException();
            }
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
            if (parent != null) {
                destroyed = true;
                ngroups = 0;
                groups = null;
                nthreads = 0;
                threads = null;
            }
        }
        for (int i = 0 ; i < ngroupsSnapshot ; i += 1) {
            groupsSnapshot[i].destroy();
        }
        if (parent != null) {
            parent.remove(this);
        }
    }
    
    //将指定的线程组添加到该线程组。
    private final void add(ThreadGroup g){
        synchronized (this) {
            if (destroyed) {
                throw new IllegalThreadStateException();
            }
            if (groups == null) {
                groups = new ThreadGroup[4];
            } else if (ngroups == groups.length) {
                groups = Arrays.copyOf(groups, ngroups * 2);
            }
            groups[ngroups] = g;

            //This is done last so it doesn't matter in case the thread is killed
            ngroups++;
        }
    }
    
    //从该线程组删除指定的线程组。
    private void remove(ThreadGroup g) {
        synchronized (this) {
            if (destroyed) {
                return;
            }
            for (int i = 0 ; i < ngroups ; i++) {
                if (groups[i] == g) {
                    ngroups -= 1;
                    System.arraycopy(groups, i + 1, groups, i, ngroups - i);
                    groups[ngroups] = null;
                    break;
                }
            }
            if (nthreads == 0) {
                notifyAll();
            }
            if (daemon && (nthreads == 0) && (nUnstartedThreads == 0) && (ngroups == 0))
            {
                destroy();
            }
        }
    }
    
    //自增线程组中未启动的线程数
    void addUnstarted() {
        synchronized(this) {
            if (destroyed) {
                throw new IllegalThreadStateException();
            }
            nUnstartedThreads++;
        }
    }
    
    //将指定线程添加到该线程组。
    void add(Thread t) {
        synchronized (this) {
            if (destroyed) {
                throw new IllegalThreadStateException();
            }
            if (threads == null) {
                threads = new Thread[4];
            } else if (nthreads == threads.length) {
                threads = Arrays.copyOf(threads, nthreads * 2);
            }
            threads[nthreads] = t;

            nthreads++;

            nUnstartedThreads--;
        }
    }
    
    //如果线程t启动失败，则将相应线程组的nUnstartedThreads加1
    void threadStartFailed(Thread t) {
        synchronized(this) {
            remove(t);
            nUnstartedThreads++;
        }
    }
    
    //终止线程组中的线程t
    void threadTerminated(Thread t) {
        synchronized (this) {
            remove(t);

            if (nthreads == 0) {
                notifyAll();
            }
            if (daemon && (nthreads == 0) && (nUnstartedThreads == 0) && (ngroups == 0)) {
                destroy();
            }
        }
    }
    
    //从线程组中删除线程t
    private void remove(Thread t) {
        synchronized (this) {
            if (destroyed) {
                return;
            }
            for (int i = 0 ; i < nthreads ; i++) {
                if (threads[i] == t) {
                    System.arraycopy(threads, i + 1, threads, i, --nthreads - i);

                    threads[nthreads] = null;
                    break;
                }
            }
        }
    }
    
    //将有关此线程组的信息打印到标准输出。此方法只对调试有用。
    public void list() {
        list(System.out, 0);
    }
    void list(PrintStream out, int indent) {
        int ngroupsSnapshot;
        ThreadGroup[] groupsSnapshot;
        synchronized (this) {
            for (int j = 0 ; j < indent ; j++) {
                out.print(" ");
            }
            out.println(this);
            indent += 4;
            for (int i = 0 ; i < nthreads ; i++) {
                for (int j = 0 ; j < indent ; j++) {
                    out.print(" ");
                }
                out.println(threads[i]);
            }
            ngroupsSnapshot = ngroups;
            if (groups != null) {
                groupsSnapshot = Arrays.copyOf(groups, ngroupsSnapshot);
            } else {
                groupsSnapshot = null;
            }
        }
        for (int i = 0 ; i < ngroupsSnapshot ; i++) {
            groupsSnapshot[i].list(out, indent);
        }
    }
    
    //当此线程组中的线程因为一个未捕获的异常而停止，
    //并且线程没有安装特定Thread.UncaughtExceptionHandler时，由Java Virtual Machine调用此方法。
    //ThreadGroup 的 uncaughtException 方法执行以下操作：
    //   如果此线程组有一个父线程组，那么调用此父线程组的uncaughtException方法时带有两个相同的参数。
    //   否则，此方法将查看是否安装了默认的未捕获异常处理程序，
    // 如果是，则在调用其 uncaughtException 方法时带有两个相同的参数。
    //   否则，此方法将确认 Throwable 参数是否为一个 ThreadDeath 实例。
    // 如果是，则不会做任何特殊的操作。否则，在从线程的 getName 方法返回时，
    // 会使用Throwable的printStackTrace方法，将包含线程名称的消息和堆栈跟踪信息输出到标准错误流。
    //应用程序可以重写 ThreadGroup 的子类中的方法，以提供处理未捕获异常的替代办法。
    public void uncaughtException(Thread t, Throwable e) {
        if (parent != null) {
            parent.uncaughtException(t, e);
        } else {
            Thread.UncaughtExceptionHandler ueh =
                Thread.getDefaultUncaughtExceptionHandler();
            if (ueh != null) {
                ueh.uncaughtException(t, e);
            } else if (!(e instanceof ThreadDeath)) {
                System.err.print("Exception in thread \"" + t.getName() + "\" ");
                e.printStackTrace(System.err);
            }
        }
    }
    
    //已过时。此调用的定义取决于 suspend()，它被废弃了。更进一步地说，此调用的行为从不被指定。
    //由 VM 用于控制内存不足时的隐式挂起。
    @Deprecated
    public boolean allowThreadSuspension(boolean b) {
        this.vmAllowSuspension = b;
        if (!b) {
            VM.unsuspendSomeThreads();
        }
        return true;
    }
    
    //返回此线程组的字符串表示形式。
    public String toString() {
        return getClass().getName() + "[name=" + getName() + ",maxpri=" + maxPriority + "]";
    }
 }
```
