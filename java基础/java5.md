## 继承

### 方法的签名

- 方法的名字和参数列表
- 子类覆盖超类的方法，需要是相同的签名，允许子类将覆盖方法的返回类型改为**原返回类型的子类型**

### 方法调用过程

- 编译器预先为每个类计算了一个方法表
  - 列出了所有方法的签名和要调用的实际方法
- 编译器查看对象的声明类型和方法名
  - 本类和超类中所有的符合的方法
- 编译器确定参数类型
  - 涉及到参数的类型转换
- 如果是private、static、final方法或者构造器，编译器能准确知道应该调用哪个方法，这称为静态绑定
  - 如果调用的方法依赖于隐式参数的实际类型，编译器就动态绑定一个调用它的指令
- 动态绑定时，虚拟机必须调用与x所引用对象的实际类型对应的方法f。x是子类，如果x里有f就调用，没有就调用x父类的f
- 动态绑定的特点：无需对现有的代码进行修改就可以对程序进行扩展
  - 比如变量e引用了某个新类Ex，那么e.someMethod()的代码不需要重新编译，只是变了查找方法的范围

### 强制类型转换--编译器与承诺

- 编译器不知道人是否犯错，所以要检查人是否承诺过多

  - 承诺：可以理解为指针，这个指针可以被用于赋值给其他指针或用于运算得到其他指针，即开发者给计算机一个信任的指针，这样编译器就可以根据这个指针对其他指针进行大胆地优化了

- 如果将子类引用给一个超类，编译器不会说啥

- 但如果把超类赋给子类，就承诺过多了，必须要进行强制类型转换，才能通过运行时的检查

- ```java
  Manager manager = (Manager) staff[1]; //error
  ```

  - 这个就会出现classcastexception，运行时注意到承诺不符
  - 强转前可以用instanceof判断能否成功

- 但强转和instanceof尽量少用

### 编写一个完美的equals方法的建议

- 显示参数命名为otherObject，稍后需要将它强制转换成另一个名为other的变量

- 检测this与otherObject是否相等

  - ```java
    if (this == otherObject) return true;
    ```

  - 经常采用该方式，检查身份要比逐个比较字段开销小

- 检测otherObject是否为null，是null则返回false

  - ```java
    if (otherObject == null) return false;
    ```

- 比较this与otherObject的类

  - 如果equals语义可以在子类中改变，就使用getClass检测

  - ```java
    if (getClass() != otherObject.getClass()) return false;
    ```

  - 如果所有的子类都有相同的相等性语义，可以使用instanceof检测

  - ```java
    if (!(otherObject instanceof ClassName)) return false;
    ```

- 将otherObject强制转换为相应类类型的变量

  - ```java
    ClassName other = (ClassName) otherObject;
    ```

- 根据相等性要求的概念比较字段

  - 使用==比较基本类型字段，使用Objects.equals比较对象字段，Arrays.equals比较数组字段

  - ```java
    return field1 == other.field1 
    	&& Objects.equals(field2, other.field2)
      	&& ...;
    ```

  - 如果在子类中重新定义equals，就要在里面包含一个super.equals(other)调用

### hashcode

- ```java
  var s = "ok";
  var sb = new StringBuilder(s);
  System.out.println(s.hashCode() + " " + sb.hashCode());//3548 1324119927
  var t = new String("ok");
  var tb = new StringBuilder(t);
  System.out.println(t.hashCode() + " " + tb.hashCode());//3548 1607521710
  ```

  - String的散列码是由内容导出的
  - StringBuilder类中没有定义hashcode方法，而Object类的默认hashCode方法会从对象的存储地址得出散列码
  - 这就解释了为什么s与t散列码相同，而sb和tb散列码不同

- ```java
  @Override
  public int hashCode() {
      return Objects.hash(name, salary);
  }
  ```

  - 组合多个散列值
  - 要求hashCode和equals必须相容，即x.equals(y)时，x.hashCode()与y.hashCode()要返回相同的值，那么如果比较员工的ID，就需要散列ID，而不是name或存储地址

### toString

- 类名最好不要硬编码，用下面方法，继承用super

- ```java
  //employee
  @Override
  public String toString() {
      return getClass().getName() + "{" +
              "name='" + name + '\'' +
              ", salary=" + salary +
              ", lesson=" + lesson +
              '}';
  }
  // manager
  @Override
      public String toString() {
          return super.toString() +
                  "bonus=" + bonus +
                  '}';
      }
  ```

- 数组的打印，要用Arrays.toString(array)，多维数组用Arrays.deepToString(array)


### 对象包装器与自动装箱

- ```java
  var list = new ArrayList<Integer>();
  ```

  - 由于每个值分别包装在对象中，所以这样写的效率远低于int[]数组
  - 也只有在方便性比执行效率更重要的时候，才考虑对较小的集合使用这种构造

- ```java
  Integer aInteger = 1;
  Double aDouble = 1.2;
  System.out.println(true ? aInteger : aDouble); // 1.0
  ```

  - Integer拆箱，提升为double，装箱为Double

- 编译器在生成类的字节码时会插入必要的方法调用，虚拟机只是执行这些字节码

- 包装类是不可变的，内容也是，不能用来创建会修改数值的方法

  - 要想编写一个修改数值参数值的方法，可以用org.omg.CORBA包中定义的某个持有者（holder）类型

  - ```java
    void triple(IntHolder x) {
        x.value = 3 * x.value;
    }
    ```


### Enum

- ```java
  public enum Size {
      SMALL("S"),MEDIUM("M"),LARGE("L"),EXTRA_LARGE("XL");

      private String abbreviation;

      Size(String abbreviation) {
          this.abbreviation = abbreviation;
      }
      public String getAbbreviation() {
          return abbreviation;
      }
  }
  ```

  - 枚举的构造器总是私有的
  - 会省略private
  - 枚举类都是Enum类的子类

- ```java
  System.out.println(Size.SMALL.toString());//返回SMALL，即枚举常量名
  System.out.println(Enum.valueOf(Size.class, "SMALL"));//同样返回SMALL，是toString的逆方法，valueOf
  Arrays.toString(Size.values())//[SMALL, MEDIUM, LARGE, EXTRA_LARGE]
  Size.MEDIUM.ordinal() // 1，返回位置
  ```


## 反射

- 能够分析类能力的程序称为反射

  - 在运行时可以分析类的能力
  - 运行时检查对象，比如编写适用于所有类的toString方法
  - 实现泛型数组操作代码
  - 利用Method对象（很像C++中的函数指针）

- 程序运行时，java运行时系统始终为所有对象维护一个运行时类型标识，用来跟踪每个对象所属的类

- 虚拟机利用运行时类型信息选择要执行的正确的方法

- class类会描述特定类的属性，比如Class的getName方法返回类名

  ```java
  public static void testClass() throws ClassNotFoundException {
      Class c = new Employee().getClass();
      System.out.println(c.getName());
      var generator = new Random();
      System.out.println(generator.getClass().getName());
      Class c2 = Class.forName("java.util.Random");// throws ClassNotFoundException
      System.out.println(c2.getName());
      Class c3 = Class.forName("not exist class name");
      System.out.println(c3.getName());
  }

  public static void main(String[] args) throws ClassNotFoundException {
      ReflectTest.testClass();
      // results: 
      // com.zxy.extend.Employee
      // java.util.Random
      // java.util.Random
      // Exception in thread "main" java.lang.ClassNotFoundException: not exist class name
  }
  ```

- class缩写，class表示一种类型，实际是一种泛型类，如Class<int>

  - ```java
    Class randomAbbr = Random.class;
    Class intAbbr = int.class;
    Class doubleAbbr = Double[].class;
    ```

- class用等号==和instanceof比较的不同之处，在于前者不判断继承，后者考虑继承

  - ```java
    Employee e = new Employee();
    //Manager e = new Manager();
    if (e.getClass() == Employee.class) {
      System.out.println("true");// Employee
    } else {
      System.out.println("false");// Manager
    }
    ```

- Class类型的对象可以获取构造器

  - ```java
    Object obj = e.getClass().getConstructor().newInstance();
    ```

  - 但这里会抛出三个异常

  - getConstructor() ---> NoSuchMethodException，没有这个类

  - newInstance() ---> InvocationTargetException InstantiationException IllegalAccessException

    - 相当于C++的虚拟构造器
    - 与废弃的Class.toInstance()，相比，Constructor.newInstance()将构造器的异常包装到InvocationTargetException、InstantiationException ——有可能没有无参构造器
    - IllegalAccessException无权限

- 资源resource可以通过类获取，getResource()获取url，或者用getResourceAsStream获取流，


### 如何打印类的全部信息

```java
public static void main(String[] args) throws ReflectiveOperationException {
  // read class name from command line args or user input
  String name;
  //        if (args.length() > 0) name = args[0];
  //        else {
  var in = new Scanner(System.in);
  System.out.println("Enter class name(e.g. java.util.Date):");
  name = in.next();
  //        }
  // print class name and superclass name (if != Object)
  Class cl = Class.forName(name);
  Class supercl = cl.getSuperclass();
  String modifiers = Modifier.toString(cl.getModifiers());
  if (modifiers.length() > 0) System.out.print(modifiers + " ");
  System.out.print("class " + name);
  if (supercl != null && supercl != Object.class) System.out.print(" extends " + supercl.getName());
  System.out.print(" {\n");
  printConstructors(cl);
  System.out.println();
  printMethods(cl);
  System.out.println();
  printFields(cl);
  System.out.println("}");
}

public static void printConstructors(Class cl) {
  Constructor[] constructors = cl.getConstructors();
  for (Constructor c:
       constructors) {
    String name = c.getName();
    System.out.print("  ");
    String modifiers = Modifier.toString(c.getModifiers());
    if (modifiers.length() > 0) System.out.print(modifiers + " ");
    System.out.print(name + "(");

    // print parameter types
    Class[] paramTypes = c.getParameterTypes();
    for (int i = 0; i < paramTypes.length; i++) {
      if (i > 0) System.out.print(", ");
      System.out.print(paramTypes[i].getName());
    }
    System.out.println(");");
  }
}

public static void printMethods(Class cl) {
  Method[] methods = cl.getMethods();
  for (Method m :
       methods) {
    Class retType = m.getReturnType();
    String name = m.getName();
    System.out.print("  ");

    // print modifiers, return type and method name
    String modifiers = Modifier.toString(m.getModifiers());
    if (modifiers.length() > 0) System.out.print(modifiers + " ");
    System.out.print(retType.getName() + " " + name + "(");
    // print parameter types
    Class[] paramTypes = m.getParameterTypes();
    for (int i = 0; i < paramTypes.length; i++) {
      if (i > 0) System.out.print(", ");
      System.out.print(paramTypes[i].getName());
    }
    System.out.println(");");
  }
}

public static void printFields(Class cl) {
  Field[] fields = cl.getDeclaredFields();
  for (Field f :
       fields) {
    Class type = f.getType();
    String name = f.getName();
    System.out.print("  ");
    String modifiers = Modifier.toString(f.getModifiers());
    if (modifiers.length() > 0) System.out.print(modifiers + " ");
    System.out.println(type.getName() + " " + name + ";");
  }
}

```

- 结果如下

````java
public final class java.lang.Double extends java.lang.Number {
  public java.lang.Double(double);
  public java.lang.Double(java.lang.String);

  public boolean equals(java.lang.Object);
  public static java.lang.String toString(double);
  public java.lang.String toString();
  public static int hashCode(double);
  public int hashCode();
  public static double min(double, double);
  public static double max(double, double);
  public static native long doubleToRawLongBits(double);
  public static long doubleToLongBits(double);
  public static native double longBitsToDouble(long);
  public int compareTo(java.lang.Double);
  public volatile int compareTo(java.lang.Object);
  public static int compare(double, double);
  public byte byteValue();
  public short shortValue();
  public int intValue();
  public long longValue();
  public float floatValue();
  public double doubleValue();
  public static java.lang.Double valueOf(java.lang.String);
  public static java.lang.Double valueOf(double);
  public static java.lang.String toHexString(double);
  public volatile java.lang.Object resolveConstantDesc(java.lang.invoke.MethodHandles$Lookup);
  public java.lang.Double resolveConstantDesc(java.lang.invoke.MethodHandles$Lookup);
  public java.util.Optional describeConstable();
  public boolean isNaN();
  public static boolean isNaN(double);
  public static double sum(double, double);
  public boolean isInfinite();
  public static boolean isInfinite(double);
  public static boolean isFinite(double);
  public static double parseDouble(java.lang.String);
  public final void wait(long, int);
  public final void wait();
  public final native void wait(long);
  public final native java.lang.Class getClass();
  public final native void notify();
  public final native void notifyAll();

  public static final double POSITIVE_INFINITY;
  public static final double NEGATIVE_INFINITY;
  public static final double NaN;
  public static final double MAX_VALUE;
  public static final double MIN_NORMAL;
  public static final double MIN_VALUE;
  public static final int MAX_EXPONENT;
  public static final int MIN_EXPONENT;
  public static final int SIZE;
  public static final int BYTES;
  public static final java.lang.Class TYPE;
  private final double value;
  private static final long serialVersionUID;
}
````

### 运行时分析对象

```java
public class ObjectAnalyzerTest {
    public static void main(String[] args) throws ReflectiveOperationException {
        var squares = new ArrayList<Integer>();
        for (int i = 1; i < 6; i++) {
            squares.add(i * i);
        }
        // 调用通用的toString方法，获取所有数据字段，对每个字段获取名字和值，并转换为字符串
        System.out.println(new ObjectAnalyzer().toString(squares));
    }
}
```

```java
public class ObjectAnalyzer {
    // 跟踪已经访问的对象，因为循环引用会导致无限递归
    private ArrayList<Object> visited = new ArrayList<>();

    public String toString(Object obj) throws ReflectiveOperationException {
        if (obj == null) {
            return null;
        }
        if (visited.contains(obj)) {
            return "...";
        }
        visited.add(obj);
        Class cl = obj.getClass();
        if (cl == String.class) {
            return (String) obj;
        }
        if (cl.isArray()) {
            String r = cl.getComponentType() + "[]{";
            for (int i = 0; i < Array.getLength(obj); i++) {
                if (i > 0) r += ",";
                Object val = Array.get(obj, i);
                if (cl.getComponentType().isPrimitive()) r += val;
                else r += toString(val);
            }
            return r + "}";
        }
        String r = cl.getName();
        // inspect the fields of this class and all superclasses
        do {
            r += "[";
            Field[] fields = cl.getDeclaredFields();
            AccessibleObject.setAccessible(fields, true);
            for (Field field : fields) {
                if (!Modifier.isStatic(field.getModifiers())) {
                    if (!r.endsWith("[")) r+=",";
                    r += field.getName() + "=";
                    Class t = field.getType();
                    Object val = field.get(obj);
                    if (t.isPrimitive()) r+=val;
                    else r+=toString(val);
                }
            }
            r += "]";
            cl = cl.getSuperclass();
        } while (cl != null);

        return r;
    }
}

```

### 使用反射编写泛型数组代码

- 对象数组不能强制转换为其他对象数组--classcastexception

- java数组会记住每个元素的类型，即创建数组的时候（new）中使用的元素类型

- 如果临时将Employee[] -> Object[]，再把它转换回来是可以的，但是一开始就是Object[]的不行

- Array类的静态方法newInstance可以构造新数组，两个参数：元素类型，数组长度

- ```java
  public static void main(String[] args) {
      int[] a = {1, 2, 3};
      a = (int[]) goodCopyOf(a, 10);
      System.out.println(Arrays.toString(a));

      String[] b = {"tom", "Dick", "Harry"};
      b = (String[]) goodCopyOf(b, 10);
      System.out.println(Arrays.toString(b));

      System.out.println("the following call will generate an exception");
      b = (String[]) badCopyOf(b, 10);
  }

  /**
   * allocate a new array
   * @param a
   * @param newLength
   * @return
   */
  public static Object[] badCopyOf(Object[] a, int newLength) { // not useful
      var newArray = new Object[newLength];
      System.arraycopy(a, 0, newArray, 0, Math.min(a.length, newLength));
      return newArray;
  }

  /**
   * allocate a new array of the same type
   * @param a
   * @param newLength
   * @return
   */
  public static Object goodCopyOf(Object a, int newLength) {
      Class cl = a.getClass();
      if (!cl.isArray()) return null;
      Class componentType = cl.getComponentType();
      int length = Array.getLength(a);
      Object newArray = Array.newInstance(componentType, length);
      System.arraycopy(a, 0, newArray, 0, Math.min(length, newLength));
      return newArray;
  }
  ```

### 调用任意方法和构造器

- C和C++中可以通过一个函数指针执行任意函数，java没有指针，没有提供方法存储地址的途径，接口和lambda是一种很好的解决方案

- 但java也可以用反射调用任意的方法

- ```java
  public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
      Method squere = MethodTableTest.class.getMethod("square", double.class);
      Method sqrt = Math.class.getMethod("sqrt", double.class);

      printTable(1, 10, 10, squere);
      printTable(1, 10, 10, sqrt);
  }

  public static double square(double x) {
      return x * x;
  }

  /**
   * print a table with x- and y-values for a method
   *
   * @param from
   * @param to
   * @param n    the number of rows
   * @param f
   */
  public static void printTable(double from, double to, int n, Method f) throws InvocationTargetException, IllegalAccessException {
      System.out.println(f);

      double dx = (to - from) / (n - 1);
      for (double x = from; x <= to; x += dx) {
          double y = (Double) f.invoke(null, x);
          System.out.printf("%10.4f | %10.4f%n", x, y);
      }
  }
  ```


- 缺点：
  - 错误的参数，invoke抛出异常
  - invoke参数和返回值必须是Object，意味着多次强制类型转换，编译器就会丧失检查代码的机会，以至于测试才会发现错误
  - 反射获取方法指针要比直接调用方法慢很多
- invoke(Object implicitParameter, Object[] explicitParameters): 
  - 第一个为隐式参数，如果是静态方法，传null；
  - 使用包装器传递基本数据类型的值；
  - 基本类型的返回值必须是未包装的

## 继承设计技巧

- 公共操作和字段放在超类中
- 不要使用受保护的字段
- 使用继承实现is-a关系
  - 钟点工和员工不属于is-a关系，如果钟点工继承了员工，每次都会打印薪水和税单（钟点工不需要的）
- 除非所有继承的方法都有意义，否则不要使用继承
- 覆盖方法时，不要改变预期的行为
  - 不要偏离最初的设计想法
- 使用多态，不要使用类型信息
  - 多态有动态分派机制，以执行正确的动作，用多态或接口实现的代码 比 多个类型检测的代码 更易于维护和扩展
- 不要滥用反射
  - 编译器无法帮助查找编译错误，只有运行时才会发现错误