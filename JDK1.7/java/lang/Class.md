* Class类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  Class 类的实例表示正在运行的Java应用程序中的类和接口。枚举是一种类，注释是一种接口。每个数组属于被映射为Class对象的一个类，所有具有相同元素类型和维数的数组都共享该Class对象。基本的Java类型（boolean、byte、char、short、int、long、float和double）和关键字void也表示为Class对象。
  Class没有公共构造方法。Class对象是在加载类时由Java虚拟机以及通过调用类加载器中的defineClass方法自动构造的。
  以下示例使用 Class 对象来显示对象的类名：
     void printClassName(Object obj) {
         System.out.println("The class of " + obj + " is " + obj.getClass().getName());
     }
 还可以使用一个类字面值（JLS Section 15.8.2）来获取指定类型（或 void）的Class对象。
 例如：System.out.println("The name of class Foo is: "+Foo.class.getName());
```java
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
     
     //如果此Class对象表示方法中的一个本地或匿名类，则返回一个代表底层类的内部Method对象。
     @CallerSensitive
     public Method getEnclosingMethod() {
         EnclosingMethodInfo enclosingInfo = getEnclosingMethodInfo();

         if (enclosingInfo == null)
             return null;
         else {
             if (!enclosingInfo.isMethod())
                return null;

             MethodRepository typeInfo = MethodRepository.make(enclosingInfo.getDescriptor(),
                                                              getFactory());
             Class<?>   returnType       = toClass(typeInfo.getReturnType());
             Type []    parameterTypes   = typeInfo.getParameterTypes();
             Class<?>[] parameterClasses = new Class<?>[parameterTypes.length];

             // 转换类型到类；返回类型应该是类的对象，因为methodDescriptor没有泛型信息
             for(int i = 0; i < parameterClasses.length; i++)
                parameterClasses[i] = toClass(parameterTypes[i]);

             // 执行访问检查
             Class<?> enclosingCandidate = enclosingInfo.getEnclosingClass();
             // 要非常小心，不要改变checkMemberAccessd的堆栈深度，详细查看java.lang.securityManager.checkMemberAccess
             // 注意，我们需要在外围类上做这个
             enclosingCandidate.checkMemberAccess(Member.DECLARED,
                                                 Reflection.getCallerClass(), true);
             //循环所有声明的方法，匹配方法名称、参数的数量和类型，和返回类型。匹配返回类型也是必要的因为协变返回。
             for(Method m: enclosingCandidate.getDeclaredMethods()) {
                if (m.getName().equals(enclosingInfo.getName()) ) {
                    Class<?>[] candidateParamClasses = m.getParameterTypes();
                    if (candidateParamClasses.length == parameterClasses.length) {
                        boolean matches = true;
                        for(int i = 0; i < candidateParamClasses.length; i++) {
                            if (!candidateParamClasses[i].equals(parameterClasses[i])) {
                                matches = false;
                                break;
                            }
                        }

                        if (matches) { // 最后，检查返回类型
                            if (m.getReturnType().equals(returnType) )
                                return m;
                        }
                    }
                }
             }

            throw new InternalError("Enclosing method not found");
         }
     }
    
     private native Object[] getEnclosingMethod0();

     private EnclosingMethodInfo getEnclosingMethodInfo() {...}
     
     private final static class EnclosingMethodInfo {
        private Class<?> enclosingClass;
        private String name;
        private String descriptor;

        private EnclosingMethodInfo(Object[] enclosingInfo) {
            if (enclosingInfo.length != 3)
                throw new InternalError("Malformed enclosing method information");
            try {
                // 数组预计有三个元素：
                // 立即封闭类
                enclosingClass = (Class<?>) enclosingInfo[0];
                assert(enclosingClass != null);

                // 立即封闭方法或构造函数的名称 (can be null).
                name            = (String)   enclosingInfo[1];

                // 立即封闭方法或构造函数的描述符 (null if name is null).
                descriptor      = (String)   enclosingInfo[2];
                assert((name != null && descriptor != null) || name == descriptor);
            } catch (ClassCastException cce) {
                throw new InternalError("Invalid type in enclosing method information");
            }
        }

        boolean isPartial() {
            return enclosingClass == null || name == null || descriptor == null;
        }

        boolean isConstructor() { return !isPartial() && "<init>".equals(name); }

        boolean isMethod() { return !isPartial() && !isConstructor() && !"<clinit>".equals(name); }

        Class<?> getEnclosingClass() { return enclosingClass; }

        String getName() { return name; }

        String getDescriptor() { return descriptor; }
     }
     
     private static Class<?> toClass(Type o) {...}
     
     //如果此Class对象表示一个构造函数中的一个本地或匿名类，则返回一个代表底层类的立即封闭构造函数构造对象。
     @CallerSensitive
     public Constructor<?> getEnclosingConstructor() {...}
     
     //如果此Class对象所表示的类或接口是另一个类的成员，返回被声明的类的Class对象。
     @CallerSensitive
     public Class<?> getDeclaringClass() {
        final Class<?> candidate = getDeclaringClass0();

        if (candidate != null)
            candidate.checkPackageAccess(
                    ClassLoader.getClassLoader(Reflection.getCallerClass()), true);
        return candidate;
     }
     private native Class<?> getDeclaringClass0();
     
     //返回直接封闭类的底层类。
     @CallerSensitive
     public Class<?> getEnclosingClass() {
        // 有五种类（或接口）类型：
        // a) 顶层类
        // b) 嵌套类（静态成员类）
        // c) 内部类（非静态成员类）
        // d) 本地类（在方法中声明的命名类）
        // e) 匿名类

        // JVM规范4.8.6：类必须有一个EnclosingMethod属性当且仅当它是本地类或匿名类。
        EnclosingMethodInfo enclosingInfo = getEnclosingMethodInfo();
        Class<?> enclosingCandidate;

        if (enclosingInfo == null) {
            // 这是一个顶层类, 嵌套类或内部类（a，b，或c）
            enclosingCandidate = getDeclaringClass();
        } else {
            Class<?> enclosingClass = enclosingInfo.getEnclosingClass();
            // 这是一个本地类或匿名类 (d 或 e)
            if (enclosingClass == this || enclosingClass == null)
                throw new InternalError("Malformed enclosing method information");
            else
                enclosingCandidate = enclosingClass;
        }

        if (enclosingCandidate != null)
            enclosingCandidate.checkPackageAccess(
                    ClassLoader.getClassLoader(Reflection.getCallerClass()), true);
        return enclosingCandidate;
     }
     
     // 返回在源代码中给定的底层类的简单名称。如果底层类是匿名的，则返回一个空字符串。
     public String getSimpleName() {...}
     
     //判断字符是否是数字
     private static boolean isAsciiDigit(char c) {
        return '0' <= c && c <= '9';
     }
     
     //返回底层类的Java语言规范中定义的标准名称。
     public String getCanonicalName() {...}
     
     //当且仅当底层类是匿名类时, 此方法返回true.
     public boolean isAnonymousClass() {
        return "".equals(getSimpleName());
     }
     
     //当且仅当底层类是局部类时, 此方法返回true。
     public boolean isLocalClass() {
        return isLocalOrAnonymousClass() && !isAnonymousClass();
     }
     
     //当且仅当底层类是成员类时，此方法返回true。
     public boolean isMemberClass() {
        return getSimpleBinaryName() != null && !isLocalOrAnonymousClass();
     }
     //返回底层类的“简单二进制名称”，即，没有引导的封闭类名称的二进制名称。
     private String getSimpleBinaryName() {...}
     
     // 判断是否是局部类或匿名类
     private boolean isLocalOrAnonymousClass() {
        // JVM规范4.8.6：类必须有一个EnclosingMethod属性当且仅当它是本地类或匿名类。
        return getEnclosingMethodInfo() != null;
     }
     
     //返回一个包含所有的所代表公共类(接口)的Class对象的数组。
     @CallerSensitive
     public Class<?>[] getClasses() {
        checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), false);

        //访问权限判断
        return java.security.AccessController.doPrivileged(
            new java.security.PrivilegedAction<Class<?>[]>() {
                public Class[] run() {
                    List<Class<?>> list = new ArrayList<>();
                    Class<?> currentClass = Class.this;
                    while (currentClass != null) {
                        Class<?>[] members = currentClass.getDeclaredClasses();
                        for (int i = 0; i < members.length; i++) {
                            if (Modifier.isPublic(members[i].getModifiers())) {
                                list.add(members[i]);
                            }
                        }
                        currentClass = currentClass.getSuperclass();
                    }
                    return list.toArray(new Class[0]);
                }
            });
     }
     
     //返回此Class对象所表示类或接口的所有可访问公共字段的数组。
     @CallerSensitive
     public Field[] getFields() throws SecurityException {
        checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
        return copyFields(privateGetPublicFields(null));
     }
     
     //返回此Class对象所表示类或接口的所有公共成员方法数组。
     @CallerSensitive
     public Method[] getMethods() throws SecurityException {
        checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
        return copyMethods(privateGetPublicMethods());
     }
     
     //返回此Class对象所表示类或接口的所有公共构造器数组。
     @CallerSensitive
     public Constructor<?>[] getConstructors() throws SecurityException {
        checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
        return copyConstructors(privateGetDeclaredConstructors(true));
     }
     
     //返回一个Field对象，它反映此Class对象所表示的类或接口的指定公共成员字段。
     @CallerSensitive
     public Field getField(String name) throws NoSuchFieldException, SecurityException {...}
     
     //返回一个Method对象，它反映此Class对象所表示的类或接口的指定公共成员方法。
     @CallerSensitive
     public Method getMethod(String name, Class<?>... parameterTypes) 
                                   throws NoSuchMethodException, SecurityException {...}
                                   
     @CallerSensitive
     public Constructor<T> getConstructor(Class<?>... parameterTypes)
           throws NoSuchMethodException, SecurityException {...}
     
     //返回Class对象反映声明此Class对象所表示类成员的类和接口组成的数组。
     @CallerSensitive
     public Class<?>[] getDeclaredClasses() throws SecurityException {...}
     
     //返回所有Class对象表示的类或接口中声明的字段的数组。
     @CallerSensitive
     public Field[] getDeclaredFields() throws SecurityException {...}
     
     @CallerSensitive
     public Method[] getDeclaredMethods() throws SecurityException {...}
     
     @CallerSensitive
     public Constructor<?>[] getDeclaredConstructors() throws SecurityException {...}
     
     @CallerSensitive
     public Field getDeclaredField(String name) throws NoSuchFieldException, SecurityException {...}
     
     @CallerSensitive
     public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
         throws NoSuchMethodException, SecurityException {...}
     
     @CallerSensitive
     public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)
          throws NoSuchMethodException, SecurityException {...}
     
     //找到指定名称的资源
     public InputStream getResourceAsStream(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // 系统类
            return ClassLoader.getSystemResourceAsStream(name);
        }
        return cl.getResourceAsStream(name);
     }
     //查找给定名称的资源
     public java.net.URL getResource(String name) {...}
     
     //当内部域为空时返回的保护域
     private static java.security.ProtectionDomain allPermDomain;
     
     //返回类的保护域
     public java.security.ProtectionDomain getProtectionDomain() {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            sm.checkPermission(SecurityConstants.GET_PD_PERMISSION);
        }
        java.security.ProtectionDomain pd = getProtectionDomain0();
        if (pd == null) {
            if (allPermDomain == null) {
                java.security.Permissions perms =
                    new java.security.Permissions();
                perms.add(SecurityConstants.ALL_PERMISSION);
                allPermDomain =
                    new java.security.ProtectionDomain(null, perms);
            }
            pd = allPermDomain;
        }
        return pd;
    }
    
    //本地方法：返回该类的保护域。
    private native java.security.ProtectionDomain getProtectionDomain0();
    
    native void setProtectionDomain0(java.security.ProtectionDomain pd);
    
    //静态本地方法：返回用于命名原始类型的虚拟机类对象。
    static native Class getPrimitiveClass(String name);
    
    private static boolean isCheckMemberAccessOverridden(SecurityManager smgr) {...}
    
    //检查在当前包准入政策下客户端加载的类加载器是否被允许访问该类。如果访问被拒绝，引发SecurityException。
    private void checkMemberAccess(int which, Class<?> caller, boolean checkProxyInterfaces) {...}
    
    //解析名称
    private String resolveName(String name) {
        if (name == null) {
            return name;
        }
        if (!name.startsWith("/")) {
            Class<?> c = this;
            while (c.isArray()) {
                c = c.getComponentType();
            }
            String baseName = c.getName();
            int index = baseName.lastIndexOf('.');
            if (index != -1) {
                name = baseName.substring(0, index).replace('.', '/')
                    +"/"+name;
            }
        } else {
            name = name.substring(1);
        }
        return name;
    }
    
    //反射支持
    // 某些反射结果的缓存
    private static boolean useCaches = true;
    private volatile transient SoftReference<Field[]> declaredFields;
    private volatile transient SoftReference<Field[]> publicFields;
    private volatile transient SoftReference<Method[]> declaredMethods;
    private volatile transient SoftReference<Method[]> publicMethods;
    private volatile transient SoftReference<Constructor<T>[]> declaredConstructors;
    private volatile transient SoftReference<Constructor<T>[]> publicConstructors;
    // getFields和getMethods的中间结果
    private volatile transient SoftReference<Field[]> declaredPublicFields;
    private volatile transient SoftReference<Method[]> declaredPublicMethods;
    
    //重新定义该类或超类的次数
    private volatile transient int classRedefinedCount = 0;
    
    //上次清除缓存时的值
    private volatile transient int lastRedefinedCount = 0;
    
    private void clearCachesOnClassRedefinition() {...}
    
    private native String getGenericSignature();
    
    //通用信息库；延迟初始化
    private transient ClassRepository genericInfo;
    
    private GenericsFactory getFactory() {
        // 创建范围和工厂
        return CoreReflectionFactory.make(this, ClassScope.make(this));
    }
    
    //通用信息库的访问
    private ClassRepository getGenericInfo() {...}
    
    //注释处理
    private native byte[] getRawAnnotations();

    native ConstantPool getConstantPool();
    
    //处理java.lang.reflect.Field，返回“根”字段数组。
    private Field[] privateGetDeclaredFields(boolean publicOnly) {...}
    private Field[] privateGetPublicFields(Set<Class<?>> traversedInterfaces) {...}
    private static void addAll(Collection<Field> c, Field[] o) {...}
    
    //处理java.lang.reflect.Constructor，返回“根”构造器数组。
    private Constructor<T>[] privateGetDeclaredConstructors(boolean publicOnly) {...}
    
    //处理java.lang.reflect.Method，返回“根”方法数组。
    private Method[] privateGetDeclaredMethods(boolean publicOnly) {...}
    
    static class MethodArray {...}
    
    private Method[] privateGetPublicMethods() {...}
    
    //处理字段，构造函数和方法的辅助器
    private Field searchFields(Field[] fields, String name) {...}
    private Field getField0(String name) throws NoSuchFieldException {...}
    private static Method searchMethods(Method[] methods, String name, Class<?>[] parameterTypes) {...}
    private Method getMethod0(String name, Class<?>[] parameterTypes) {...}
    private Constructor<T> getConstructor0(Class<?>[] parameterTypes, int which) throws NoSuchMethodException {...}
    
    //其他辅助器和基本实现
    private static boolean arrayContentsEq(Object[] a1, Object[] a2) {...}
    private static Field[] copyFields(Field[] arg) {...}
    private static Method[] copyMethods(Method[] arg) {...}
    private static <U> Constructor<U>[] copyConstructors(Constructor<U>[] arg) {...}
    
    private native Field[]       getDeclaredFields0(boolean publicOnly);
    private native Method[]      getDeclaredMethods0(boolean publicOnly);
    private native Constructor<T>[] getDeclaredConstructors0(boolean publicOnly);
    private native Class<?>[]   getDeclaredClasses0();
    
    private static String        argumentTypesToString(Class<?>[] argTypes) {...}
    
    private static final long serialVersionUID = 3206093459760846163L;
    
    private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];
     
    //返回将被分配给该类的断言状态，同时在调用该方法时初始化。
    public boolean desiredAssertionStatus() {...}
    
    //从虚拟机中检索该类的所需的断言状态
    private static native boolean desiredAssertionStatus0(Class<?> clazz);
    
    public boolean isEnum() {
        return (this.getModifiers() & ENUM) != 0 &&
        this.getSuperclass() == java.lang.Enum.class;
    }
    
    private static ReflectionFactory getReflectionFactory() {...}
    private static ReflectionFactory reflectionFactory;
    
    //一旦该类初始化就尽快查询系统属性
    private static boolean initted = false;
    private static void checkInitted() {...}
    
    public T[] getEnumConstants() {...}
    T[] getEnumConstantsShared() {...}
    private volatile transient T[] enumConstants = null;
    
    Map<String, T> enumConstantDirectory() {...}
    private volatile transient Map<String, T> enumConstantDirectory = null;
    
  }
```
