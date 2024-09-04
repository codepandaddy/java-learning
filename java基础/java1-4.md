## 一、java设计概述

### JVM（java virtual machine）

- 是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟计算机功能来实现的，JVM屏蔽了与具体操作系统平台相关的信息，Java程序只需生成在Java虚拟机上运行的字节码，就可以在多种平台上不加修改的运行。JVM在执行字节码时，实际上最终还是把字节码解释成具体平台上的机器指令执行。
- 具体见https://blog.csdn.net/weixin_43771403/article/details/121288104

## 二、java基本程序设计结构

### 整型

- c和c++中，整型的大小与平台相关，比如int在16位、32位处理器上，分别占据2和4字节；而**java**所有的数据类型所占据字节数与平台无关，都是固定的

### Unicode和char类型

- utf-8用不同长度的编码表示所有unicode码点，char类型描述了utf-16编码中的一个代码单元

### 位运算符

- &、|、^、~、>>、<<、>>>

- &、|、^、~

  - 用他们相应的位，来进行按位与运算，然后得出的结果就是这两个数按位与&的结果，同样的有按位或

  - ^相同为0，~取反

  - ```java
    // 如果参与运算的位，相同的话为1，否则为0
    0 & 0 = 1;
    0 & 1 = 0;
    1 & 1 = 1;
    1 & 0 = 0;
    // 按位或与按位与不同的是，只要相应的位有一个为1，结果都可以为1，否则结果为 0
    0 | 0 = 0;
    0 | 1 = 1;
    1 | 1 = 1;
    1 | 0 = 1;
    // 按位异或，如果相应的位相同，则为0，否则为1
    0 ^ 0 = 0;
    0 ^ 1 = 1;
    1 ^ 1 = 0;
    1 ^ 0 = 1;
    011110; // 30
    110111; // 55
    // 按位与
    010110; // 22  结果
    // 按位或
    111111; // 63  结果
    // 按位异或
    101001; // 41  结果
    ```

- \>\>、<<

  - << 左移，每左移一位，就相当于把原来的数 乘以 2

  - ```java
    0000 0010  // 2

    2 << 1

    0000 0100  // 4
    -------------------------------------------
    -------------------------------------------
    0000 0010  // 2

    2 << 2 

    0000 1000 // 8
    ```

  - \>\>右移，相当于除以2

### 码点与代码单元

- ```java
  String greeting = "Hello";
  int n = greeting.length();// is 5
  int cpCount = greeting.codePointCount(0, greeting.length()); // is 5
  ```

  - 通过length方法，将返回采用utf-16编码表示给定字符串所需要的代码单元数量
  - 获取实际数量，即码点数量，用codePointCount

- 𝕆字符需要两个代码单元表示

  - ```java
    String sentence = "\uD835\uDD46 is the set of octonions";
    char ch = sentence.charAt(1); // '\uDD46'
    ```

    - 为了避免这个问题，不要使用char类型，这个太底层了
    - 除了这个符号，还有一些emoji

- ```java
  // 获取第i个码点
  int i = 1;
  // 从startIndex开始，cpCount个码点后的码点索引
  int index = sentence.offsetByCodePoints(0, i);
  int cp = sentence.codePointAt(index);

  // 遍历字符串，查每个码点
  for (int j = 0; j < sentence.length();) {
    int cp2 = sentence.codePointAt(j);
    // 是否在补充字符范围内
    if (Character.isSupplementaryCodePoint(cp)) j += 2;
    else j++;
    System.out.println(cp2 + " ---order");
  }

  // 反向遍历
  for (int j = sentence.length() - 1; j >= 0; j--) {
    // 是否为代码单元
    if (Character.isSurrogate(sentence.charAt(j))) j--;
    int cp3 = sentence.codePointAt(j);
    System.out.println(cp3 + " ---reverse");
  }

  // 简单方法获取码点
  int[] codePoints = sentence.codePoints().toArray();
  Arrays.stream(codePoints).forEach(point -> {
    System.out.println(point);
  });

  // 把一个码点数组转换为一个字符串
  String str = new String(codePoints, 0, codePoints.length);
  System.out.println(str);
  ```

  - 在java9中，只包含单字节代码单元的字符串使用byte数组实现，所有其他字符串使用char数组

### StringBuilder

- 避免每次拼接字符串时，都会构建一个新的String对象，既耗时，又浪费空间
- 前身为StringBuffer，支持多线程添加或删除字符

### 大数

- 如果基本的整数和浮点数无法满足精度计算，可以使用BigInteger和BigDecimal实现任意精度的运算，但是不能用算术运算符处理大数，需要用到各自的方法实现

- 比如在计算中彩概率时

  - ```java
    // 从1~n个数里面取出k个来抽奖，求中奖概率
    Scanner in = new Scanner(System.in);
    System.out.println("how many numbers do you need to draw? ");
    int k = in.nextInt();
    System.out.println("what is the highest number you can draw? ");
    int n = in.nextInt();
    BigInteger lotteryOdds = BigInteger.valueOf(1);
    // 组合数
    for (int i = 1; i <= k; i++) {
      // multiply和divide，除此之外还有add等
      lotteryOdds = lotteryOdds.multiply(BigInteger.valueOf(n - i + 1)).divide(BigInteger.valueOf(i));
    }
    System.out.println("your odds are 1 in " + lotteryOdds + ". Good luck!");
    ```

## 三、对象与类

### 类之间的关系

- 依赖关系：一个类的方法使用或操纵另一个类的对象
  - 应减少相互依赖的类。如果类A不知道类B的存在，它就不会关心B的任何改变（这意味着B的改变不会导致A产生任何bug）——尽量减少类之间的耦合

### java自动重新编译

- 如果java版本已有的class文件版本更新，java就会自动重新编译这个文件。这与unix的make工具（或者win的nmake工具）相似
  - make功能——简化从多个程序模块编译构建可执行文件的自动化编译工具，在编辑好的makefile文件之后，执行make即可一键进行项目构建，不用手动输入过多编译、链接的命令

### 隐式参数与显式参数

- 隐式参数implicit：出现在方法名前的对象，关键字this可指代，可以区分实例字段和局部变量
- 显式参数explicit：在方法明后面的参数/
- 在java中，所有方法都要在类内部定义，但不一定是**内联方法**。内联由jvm决定，即时编辑器会监视那些简短的、经常调用而且没有被覆盖的方法，并进行优化

### 方法参数

- 方法不可以修改基本数据类型的参数
- 方法可以改变对象参数的状态
- 方法不能让一个对象参数引用一个新的对象

### 一个构造器中调用另一个构造器

- ```java
  public Employee(double s) {
    // calls Employee(String, double)
    this("employee# " + nextId, s);
    nextId++;
  }
  ```

- 用this调用的好处是：不用再写一次Employee(String,double)，毕竟是公共的构造器，当前对象直接调用即可

- 这只能在构造器中使用，且必须写在第一行

### C++的析构器与Java的GC

- 析构器：用来完成事后所必须的清理工作，最常见的操作是回收分配给对象的存储空间
- 但java不支持析构器，因为有自动回收机制GC，不需要人工回收内存

### 对象的句柄

- 获取对象的方法，一个广义的指针，它的目的就是建立起与被访问对象之间的唯一联系
- 虚拟地址，简而言之数据的地址需要变动，变动以后就需要有人来记录、管理变动，因此系统用句柄来记载数据地址的变更

### 静态导入

- import static java.lang.System.out; 
- 我们不必在System.out.println中键入System.out
- 但是这种方法建议在有很多重复调用的时候使用，如果仅有一到两次调用，不如直接写来的方便

### 在包中增加类

- 编译器处理文件，即.java结尾的、带有文件分割符的

- 解析器加载类，即带有 . 分隔符的

- ```java
  package com.youth.risk.change.dao;
  ```

- 编译器不检查目录结构，因此，如果文件不再在原先的目录下时，且它不依赖其他包，那么就可以通过编译而不报错✅，但是最终程度将无法运行❌，除非将它重新移回去

  - 如果包与目录不匹配，虚拟机就找不到类

### 类路径class path

- 所有包含类文件的路径的集合
- 类路径包括：
  - 基目录/home/user/classdir 或 c:\classes
  - 当前目录（.）
  - JAR文件/home/user/archives/archive.jar 或 c:\archives\archive.jar
- javac编译器总是在当前目录下查找文件，但java虚拟机仅在类路径中包含"."目录的时候才查看当前目录
  - 如果没有类路径，无妨，因为默认的类路径会包含"."目录
  - 但如果设置了类路径，却忘记包含"."目录，则程序可能通过编译，但不能运行
- 示例：/home/user/classdir:.:/home/user/archives/'*'
  - 目录+归档文件是搜索类的起始点
  - 比如要搜索com/horstmann/corejava/Employee类的类文件，jvm首先查看Java API，没有就查看类路径，查找以下文件
    - /home/user/classdir/com/horstmann/corejava/Employee.class
    - com/horstmann/corejava/Employee.class(从当前目录开始)
    - com/horstmann/corejava/Employee.class(/home/user/archives/archive.jar中)

#### java.sql.Date和java.util.Date在import时报错

- 前提是在代码中引用了Date类
- 报错是因为完全限定类名必须是唯一的，如果在类路径的所有位置搜索这两个类，找到一个以上，就会产生编译时错误，与import的顺序无关

### JAR文件（java archive Java归档）

- 一个jar文件可以包含类文件，也可以包含诸如图像、声音等其他类型文件，用到了zip压缩格式
- 创建jar文件：jar cvf jarFileName file1 file2 ...
- 还可包含清单文件MANIFEST.MF
  - 创建：jar cfm jarFileName manifestFileName ...
  - 更新则把c改成u，后面不用再跟打包文件
- 可执行jar，使用e选项，指定程序的入口点，调用java程序启动器时指定的类：
  - jar cvfe jarFileName com.mycompany.mypkg.MainAppClass files to add 
  - 或者在清单文件中指定程序的主类 Main-Class: com.mycompany.mypkg.MainAppClass
  - **可以对比项目的Application**
    - springboot项目中，用JarLauncher返回可执行jar文件的启动类
    - 程序的启动入口`Main-Class`和`Start-Class`，利用反射调用定义好的`Start-Class`（也就是Application.java）的main方法
- java9引入了 *多版本jar*
  - 用途是 支持某个特定版本的程序或库能够在多个不同的jdk版本上运行


### -------------分割线-------------


### 设置对象属性默认值的方式

#### 1、hutool工具类beanUtil

- ```java
  Map<String, Object> map = BeanUtil.beanToMap(source);
  map.replaceAll((k, v) -> ObjectUtil.defaultIfNull(v, EXPORT_EMPTY));
  // 直接copy，添加ignoreError属性，会给为转换类型不成功的值设置为null，且不报错
  BeanUtil.copyProperties(map, source, CopyOptions.create().ignoreError());
  // 或者mapToBean
  return BeanUtil.mapToBean(map, ExportReceiveRecordBO.class, true);
  ```

#### 2、反射field获取并设置字段（包括所有父类字段）

```java
/**
 * 设置默认值 - 字符串为null 设置为”-“
 *
 * @param source 导出数据记录
 */
private void fillExportField(ExportReceiveRecordBO source) {
  Class<?> clazz = source.getClass();
  // 遍历往上获取父类，直至最后一个父类
  for (; clazz != Object.class; clazz = clazz.getSuperclass()) {
    // 获取当前类所有的字段
    Field[] field = clazz.getDeclaredFields();
    for (Field f : field) {
      // spring提供的方法，设置对象的访问权限
      ReflectionUtils.makeAccessible(field);
      try {
        //System.out.println("属性名：" + f.getName());
        //System.out.println("属性类型：" + f.getGenericType());
        //System.out.println("属性值：" + f.get(bean).toString());
        Class<?> fieldType = f.getType();
        if (fieldType == String.class) {
          // 字符串 -> "-"
          f.set(source, ObjectUtil.defaultIfNull(f.get(source), EXPORT_EMPTY));
        }
      } catch (IllegalAccessException e) {
        throw new SystemRuntimeException(SysErrorEnum.EXPORT_ERROR);
      }
    }
  }
}
```

- 直接这样写，sonarlint会报错：Reflection should not be used to increase accessibility of classes, methods, or fields
  - 内部暴露，访问私有属性，降低可移植性
  - 性能很低

#### 3、SQL 中判断取值——很麻烦，判断每个字段

#### 4、Objects.requireNonNullElse(T obj, T defaultObj) ——JDK9

- ```java
  class Employee {
      private String name;
      private static final String UNKNOWN = "UNKNOWN";
      public Employee(String name) {
          this.name = Objects.requireNonNullElse(name, UNKNOWN);
      }
  }
  ```

#### 5、在构造方法中调用另一个构造方法

- 不建议，会给之后的依赖、扩展造成困难

### 判断两个Object是否相等

#### 1、hutool的ObjectUtil.equals

- 其中包括对BigDecimal的判断，需要用到compareTo


- ```java
  public static boolean equal(Object obj1, Object obj2) {
     if (obj1 instanceof BigDecimal && obj2 instanceof BigDecimal) {
        return NumberUtil.equals((BigDecimal) obj1, (BigDecimal) obj2);
     }
     return Objects.equals(obj1, obj2);
  }
  ```

#### 2、Objects.equals

- 如果无需对BigDecimal做对比，直接用该方法即可