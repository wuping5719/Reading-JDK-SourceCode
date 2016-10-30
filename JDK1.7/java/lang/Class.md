* Class类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  Class类的实例代表一个运行java应用程序的类和接口。枚举（enum）是一种class和注解（annotation）是一种接口。每一个数组也属于一个类，这个类被作为一个类对象所反映，这个类对象由同一个元素类型和维度的所有数组所共享。
  Class类实例的取值为原始的java类型： boolean, byte, char, short, int, long, float和double，关键字void也代表一种类对象。    
  此类与java的类加载机制紧密相关。
```javay
  public final 
    class Class<T> implements java.io.Serializable,
                              java.lang.reflect.GenericDeclaration,
                              java.lang.reflect.Type,
                              java.lang.reflect.AnnotatedElement {
     private static final int ANNOTATION= 0x00002000;    //注解类型
     private static final int ENUM      = 0x00004000;    //枚举类型
     private static final int SYNTHETIC = 0x00001000;    //合成类型

     private static native void registerNatives();
     static {
        registerNatives();
     }
     
     //Class类没有公共构造函数。JVM加载类或调用类加载器中定义的方法自动构造Class类实例。
     private Class() {}
     
     //将对象转换为字符串。
     public String toString() {
        return (isInterface() ? "interface " : (isPrimitive() ? "" : "class "))
            + getName();
     }
     
     //@CallSensitive注解，用来找到真正发起反射请求的类，是为了堵住漏洞用的。曾经有黑客通过构造双重反射来提升权限，原理是当时反射只检查固定深度的调用者的类，看它有没有特权，例如固定看两层的调用者（getCallerClass(2)）。
     //如果我的类本来没足够权限群访问某些信息，那我就可以通过双重反射去达到目的：反射相关的类是有很高权限的，而在 我->反射1->反射2 这样的调用链上，反射2检查权限时看到的是反射1的类，这就被欺骗了，导致安全漏洞。
     //使用CallerSensitive后，getCallerClass不再用固定深度去寻找actual caller（“我”），而是把所有跟反射相关的接口方法都标注上CallerSensitive，搜索时凡看到该注解都直接跳过，这样就有效解决了前面举例的问题。
     //根据字符串参数className返回与之相关的类对象
     @CallerSensitive
     public static Class<?> forName(String className)
                throws ClassNotFoundException {
        return forName0(className, true,
                        ClassLoader.getClassLoader(Reflection.getCallerClass()));
     }
    
     //给定一个类的全限定名，尝试定位，加载，连接类或接口。指定的类加载程序用于加载类或接口。如果参数loader为空，则该类通过引导类加载程序加载。只有在参数initialize是“true”并且它还没有被初始化时才对类进行初始化。
     @CallerSensitive
     public static Class<?> forName(String name, boolean initialize,
                                   ClassLoader loader) throws ClassNotFoundException {
        if (loader == null) {
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                ClassLoader ccl = ClassLoader.getClassLoader(Reflection.getCallerClass());
                if (ccl != null) {
                    sm.checkPermission(
                        SecurityConstants.GET_CLASSLOADER_PERMISSION);
                }
            }
        }
        return forName0(name, initialize, loader);
     }
     
     //本地方法，作出安全检查后调用
     private static native Class<?> forName0(String name, boolean initialize,
                                            ClassLoader loader) throws ClassNotFoundException;
     
     //创建此Class对象所表示类的新实例。
     @CallerSensitive
     public T newInstance() throws InstantiationException, IllegalAccessException {
        if (System.getSecurityManager() != null) {
            checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), false);
        }

        // 注意：当前java内存模型下，下面的代码可能不是严格正确的
        // 构造函数的查找
        if (cachedConstructor == null) {
            if (this == Class.class) {
                throw new IllegalAccessException(
                    "Can not call newInstance() on the Class for java.lang.Class"
                );
            }
            try {
                Class<?>[] empty = {};
                final Constructor<T> c = getConstructor0(empty, Member.DECLARED);
                //禁用“构造函数”上的可访问性检查，因为我们必须在这里做安全检查（通过堆栈深度来判断构造函数安全检查是否工作是错误的）
                java.security.AccessController.doPrivileged(
                    new java.security.PrivilegedAction<Void>() {
                        public Void run() {
                                c.setAccessible(true);
                                return null;
                            }
                        });
                cachedConstructor = c;
            } catch (NoSuchMethodException e) {
                throw new InstantiationException(getName());
            }
        }
        Constructor<T> tmpConstructor = cachedConstructor;
        // 安全检查（同java.lang.reflect.Constructor中的一样)
        int modifiers = tmpConstructor.getModifiers();
        if (!Reflection.quickCheckMemberAccess(this, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            if (newInstanceCallerCache != caller) {
                Reflection.ensureMemberAccess(caller, this, null, modifiers);
                newInstanceCallerCache = caller;
            }
        }
        // 运行构造函数
        try {
            return tmpConstructor.newInstance((Object[])null);
        } catch (InvocationTargetException e) {
            Unsafe.getUnsafe().throwException(e.getTargetException());
            // 没有达到
            return null;
        }
     }
     private volatile transient Constructor<T> cachedConstructor;         //缓存的构造函数
     private volatile transient Class<?>       newInstanceCallerCache;    //新的实例调用缓存
     
     //本地方法：确定指定的对象是否为与此类所表示的对象赋值兼容。
     public native boolean isInstance(Object obj);
     
     //本地方法：确定此Class对象所表示的类或接口是不一样的，或者说是一个超类或超接口，由指定的Class参数所表示类或接口。
     public native boolean isAssignableFrom(Class<?> cls);
     
     //本地方法：判定指定Class对象是否表示一个接口类型。
     public native boolean isInterface();
     
     //本地方法：确定该Class对象是否表示一个数组类。
     public native boolean isArray();
     
     //本地方法：确定指定的Class对象是否表示一个基本类型。
     public native boolean isPrimitive();
     
     //如果此Class对象表示一个注释类型, 此方法返回true。
     public boolean isAnnotation() {
        return (getModifiers() & ANNOTATION) != 0;
     }
     
     //如果这个类是合成的类, 此方法返回true; 否则返回false。
     public boolean isSynthetic() {
        return (getModifiers() & SYNTHETIC) != 0;
     }
     
     //返回此Class对象所表示的实体（类，接口，数组类，基本类型或void）的名字。
     public String getName() {
        String name = this.name;
        if (name == null)
            this.name = name = getName0();
        return name;
     }

     // 缓存名称，以减少调用虚拟机的次数
     private transient String name;
     private native String getName0();
     
     //返回类加载器的类
     @CallerSensitive
     public ClassLoader getClassLoader() {
         ClassLoader cl = getClassLoader0();
         if (cl == null)
            return null;
         SecurityManager sm = System.getSecurityManager();
         if (sm != null) {
            ClassLoader.checkClassLoaderPermission(cl, Reflection.getCallerClass());
         }
         return cl;
     }

     // 包私有允许ClassLoader访问
     native ClassLoader getClassLoader0();
     
     //返回一个代表由GenericDeclaration对象表示的一般声明，按声明顺序声明的类型变量TypeVariable对象的数组。
     public TypeVariable<Class<T>>[] getTypeParameters() {
        if (getGenericSignature() != null)
            return (TypeVariable<Class<T>>[])getGenericInfo().getTypeParameters();
        else
            return (TypeVariable<Class<T>>[])new TypeVariable<?>[0];
     }
     
     //本地方法：表示此Class所表示的实体（类，接口，基本类型或void）的超类。
     public native Class<? super T> getSuperclass();
     
     //表示此Class所表示的实体（类，接口，基本类型或void）的直接超类的类型。
     public Type getGenericSuperclass() {
        if (getGenericSignature() != null) {
            if (isInterface())
                return null;
            return getGenericInfo().getSuperclass();
        } else
            return getSuperclass();
     }
     
     //获取这个类的包。
     public Package getPackage() {
        return Package.getPackage(this);
     }
     
     //本地方法：确定由该对象表示的类或接口实现的接口。
     public native Class<?>[] getInterfaces();
     
     //表示由该对象表示的类或接口直接实现的接口类型。
     public Type[] getGenericInterfaces() {
        if (getGenericSignature() != null)
            return getGenericInfo().getSuperInterfaces();
        else
            return getInterfaces();
     }
     
     //本地方法：返回类表示数组的组件类型。
     public native Class<?> getComponentType();
     
     //本地方法：返回这个类或接口的Java语言修饰符，编码为一个整数。
     public native int getModifiers();
     
     //本地方法：得到这个类的签名。
     public native Object[] getSigners();
     
     //本地方法：设置这个类的签名。
     native void setSigners(Object[] signers);
     
  }
```
