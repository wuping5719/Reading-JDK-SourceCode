* Boolean类的源码(JDK1.7)解读, 由于本人水平有限, 如有纰漏之处, 烦请留言指正. (Email: wp571988@163.com)   
  &nbsp;&nbsp; Boolean 类将基本类型为 boolean 的值包装在一个对象中。一个 Boolean 类型的对象只包含一个类型为 boolean 的字段。

  &nbsp;&nbsp; 此类还为 boolean 和 String 的相互转换提供了许多方法，并提供了处理 boolean 时非常有用的其他一些常量和方法。
  
```java
  public final class Boolean implements java.io.Serializable, Comparable<Boolean> {
    //对应基值 true 的 Boolean 对象。
    public static final Boolean TRUE = new Boolean(true);
    
    //对应基值 false 的 Boolean 对象。
    public static final Boolean FALSE = new Boolean(false);
    
    //表示基本类型 boolean 的 Class 对象。
    public static final Class<Boolean> TYPE = Class.getPrimitiveClass("boolean");
    
    //Boolean的价值。
    private final boolean value;
    
    private static final long serialVersionUID = -3665804199014368530L;
    
    //分配一个表示 value 参数的 Boolean 对象。
    //注：一般情况下都不宜使用该构造方法。
    //若不需要新的实例，则静态工厂 valueOf(boolean) 通常是一个更好的选择。这有可能显著提高空间和时间性能。
    public Boolean(boolean value) {
        this.value = value;
    }
    
    //如果 String 参数不为 null 且在忽略大小写时等于 "true"，则分配一个表示 true 值的 Boolean 对象。
    //否则分配一个表示 false 值的 Boolean 对象。示例：
    //   new Boolean("True") 生成一个表示 true 的 Boolean 对象。
    //   new Boolean("yes") 生成一个表示 false 的 Boolean 对象。
    public Boolean(String s) {
        this(toBoolean(s));
    }
    
    //将字符串参数解析为 boolean 值。
    //如果 String 参数不是 null 且在忽略大小写时等于 "true"，则返回的 boolean 表示 true 值。
    //   示例：Boolean.parseBoolean("True") 返回 true。
    //   示例：Boolean.parseBoolean("yes") 返回 false。
    public static boolean parseBoolean(String s) {
        return toBoolean(s);
    }
    
    //将此 Boolean 对象的值作为基本布尔值返回。
    public boolean booleanValue() {
        return value;
    }
    
    //返回一个表示指定 boolean 值的 Boolean 实例。
    //如果指定的 boolean 值为 true，则此方法返回 Boolean.TRUE；
    //如果为 false，则返回 Boolean.FALSE。
    //如果不需要新的 Boolean 实例，则应优先使用此方法，而不是构造方法 Boolean(boolean)，
    //因为此方法有可能大大提高空间和时间性能。
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
    
    //返回一个用指定的字符串表示值的 Boolean 值。
    //如果字符串参数不为 null 且在忽略大小写时等于 "true"，则返回的 Boolean 表示 true 值。
    public static Boolean valueOf(String s) {
        return toBoolean(s) ? TRUE : FALSE;
    }
    
    //返回一个表示指定布尔值的 String 对象。
    //如果指定布尔值为 true，则将返回字符串 "true"，否则将返回字符串 "false"。
    public static String toString(boolean b) {
        return b ? "true" : "false";
    }
    
    //返回表示该布尔值的 String 对象。
    //如果该对象表示 true 值，则返回等于 "true" 的字符串。否则返回等于 "false" 的字符串。
    public String toString() {
        return value ? "true" : "false";
    }
    
    //返回该 Boolean 对象的哈希码。
    public int hashCode() {
        return value ? 1231 : 1237;
    }
    
    //当且仅当参数不是 null，而是一个与此对象一样，
    //都表示同一个 Boolean 值的 boolean 对象时，才返回 true。
    public boolean equals(Object obj) {
        if (obj instanceof Boolean) {
            return value == ((Boolean)obj).booleanValue();
        }
        return false;
    }
    
    //当且仅当以参数命名的系统属性存在，且等于 "true" 字符串时，才返回 true。
    //（从 Java TM 平台的 1.0.2 版本开始，字符串的测试不再区分大小写）
    //通过 getProperty 方法可访问系统属性，此方法由 System 类定义。
    //如果没有以指定名称命名的属性或者指定名称为空或 null，则返回 false。
    public static boolean getBoolean(String name) {
        boolean result = false;
        try {
            result = toBoolean(System.getProperty(name));
        } catch (IllegalArgumentException e) {
        } catch (NullPointerException e) {
        }
        return result;
    }
    
    //将此 Boolean 实例与其他实例进行比较。
    public int compareTo(Boolean b) {
        return compare(this.value, b.value);
    }
    public static int compare(boolean x, boolean y) {
        return (x == y) ? 0 : (x ? 1 : -1);
    }
    
    private static boolean toBoolean(String name) {
        return ((name != null) && name.equalsIgnoreCase("true"));
    }
    
  }
```
