## 接口

### Arrays.sort(Object[] a)

- 要求a实现了Comparable接口

### compareTo()

- 如果y.compareTo(x)抛出异常，则x.compareTo(y)也要抛出异常
- 比如Manager和Employee，如果Manager要覆盖compareTo，绝不能仅仅将员工转换为经理
- 如果不同子类中的比较有不同的含义，就应该将属于不同类的对象之间的比较视为非法，需要以下检测

```java
if (getClass() != other.getClass()) throw new ClassCastException();
```

- 如果存在一个能够比较子类对象的通用算法，那么可以在超类中提供一个compareTo方法，并将这个方法声明为final

