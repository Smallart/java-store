# 反射[-[Reflect]-]
> 反射使得我们能够在程序运行时动态的改变相关类的行为。

使用反射提供的一些好处：
* 扩展性：可以通过全限定类名就可以实例化对象，也就是把创建对象的时机延后到了编译期，并且还可以动态赋值。

反射的一些缺点：
由于反射是在运行时进行的，所以JVM的优化是无效的。安全问题，反射需要运行时权限，在安全管理器下运行时可能不存在该权限。

对于每种类型的对象，Java虚拟级都会实例化`Java.lang.Class`的一个不可变实例，该实例提供检查对象的运行属性（包括成员和类型信息）的方法。Class对象还提供了创建新类和对象的能力，最重要的是，它是所有反射API的入口点。

获取某个类型的Class对象的几种方式：
* Object.getClass() 调用实例化对象的getClass方法。不适合基础数据类型
* 类型.class
* Class.forName() 如果知道某个类的全限定类型，就可以使用这种方式。不适合基础数据类型
* 基础数据类型的包装类型：Double.Type

Java类的成员包括了以下三类：属性字段、构造函数、方法。

`java.lang.reflect.Field`、`java.lang.reflect.Method`、`java.lang.reflect.Constructor`

Class对象中包含了一些获取父类、接口的方法，也包含了获取如上类成员的方法，其中有一些需要注意的地方：
* getDeclaredxxxs 获得该类的相应的所有东西（getDeclaredFields全部字段，getDeclaredMethods全部方法）），不包括继承而来的东西。
* getxxxs 获得该类的public修饰的东西，包括从父类继承而来的

## 反射获取构造函数
```java
Class clazz = String.class;
//获取所有公有构造方法
Constructor[] conArray = clazz.getConstructors();
// 获取该类中所有构造函数(private,protect,public)
conArray = clazz.getDeclaredConstructors();
// 获取参数为String的构造函数
Constructor con = clazz.getConstructor(String.class);
// 实例化对象，
String s = con.newInstance("test");
```

## 反射获取成员变量
```java
Class clazz = Student.class;
//  这些都类似
Field field = clazz.getField();
clazz.getField(name);
clazz.getFields()
clazz.getDeclaredFields();
// 对于 private 修饰的字段，访问前
field.setAccessible(true);

//取值 需要传入对象，才能取得该对象的该字段的值
field.get(Object)
//赋值
field.set(Object,value)
```

## 反射获得成员方法
```java
//获取方法与上面类似，其对应名称功能也类似，只说调用

method.invoke(Objec,value)

```
## 反射创建数组
```java
Object arr = Array.newInstance(String.class,size);
Array.set(arr,0,"")
Array.set(arr,1,"")
Array.set(arr,2,"")
```