Table of Contents
=================



Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

# Java基础

## 1 JDK-JRE-JVM

1)

* JVM是运行java字节码的虚拟机。jvm有针对不同系统的特定实现（windows、Linux、macOS），目的是使用相同的字节码，他们都会给出相同的结果

* java程序从代码到运行一般有下面三步：



![c](.\images\c.png)

2)

* jdk 是开发工具包，它拥有jre的一切，还有编译器（javac）和工具（javadoc注释、jdb调试器等）。它能创建和编译程序。

* 扩展：Oracle JDK和OpenJDK
  * OracleJDK比OpenJDK更稳定，相对较少出现系统问题。在jvm里面响应和性能更好，它是根据二进制代码许可协议获得许可。而openJDK是根据v2.

3）

* jre是java运行时环境。他是运行已编译程序所需所有内容的集合，它包含了jvm、java类库，java命令和其它的基础构件。但是，它不能创建新程序。



## 2 java和c++的区别



![](.\images\cjj.png)



## 3 java语法

### 字符型常量和字符串常量的区别？

1）形式上：字符常量是单引号引起的一个字符；字符串常量是双引号引起的若干个字符

2）含义上：字符常量相当于一个整型值（ASCII值），可以参与表达式运算；字符串常量代表一个地址值

3）内存大小：字符常量占2个字节；字符串常量占若干个字节

### 泛型了解吗？什么是类型擦除？介绍一下常用的通配符

Java的编译器将java源文件编译成字节码.class文件，虚拟机加载并运行。对于泛型类，java编译器会将其转换为普通的非泛型代码。将类型T擦除，然后替换为Object，插入必要的强制类型转换。Java虚拟机实际执行的时候，并不知道泛型这回事，只知道普通的类及代码。

java设计时不知道我们用集合保存什么类型的对象，所以编译时是不分类型的

java是先编译后运行的，所以泛型的出现是为了编译时进行类型检查

如在代码中定义`List<Object>`和`List<String>`等类型，在编译后都会变成`List`，JVM看到的只是List，而由泛型附加的类型信息对JVM是看不到的。Java编译器会在编译时尽可能的发现可能出错的地方，但是仍然无法在运行时刻出现的类型转换异常的情况

**通过两个例子证明Java类型的类型擦除**

例1.原始类型相等



```java
public class Test {

    public static void main(String[] args) {

        ArrayList<String> list1 = new ArrayList<String>();
        list1.add("abc");

        ArrayList<Integer> list2 = new ArrayList<Integer>();
        list2.add(123);

        System.out.println(list1.getClass() == list2.getClass());
    }

}
```



在这个例子中，我们定义了两个`ArrayList`数组，不过一个是`ArrayList<String>`泛型类型的，只能存储字符串；一个是`ArrayList<Integer>`泛型类型的，只能存储整数，最后，我们通过`list1`对象和`list2`对象的`getClass()`方法获取他们的类的信息，最后发现结果为`true`。说明泛型类型`String`和`Integer`都被擦除掉了，只剩下原始类型。

例2.通过反射添加其它类型元素

```java
public class Test {

    public static void main(String[] args) throws Exception {

        ArrayList<Integer> list = new ArrayList<Integer>();

        list.add(1);  //这样调用 add 方法只能存储整形，因为泛型类型的实例为 Integer

        list.getClass().getMethod("add", Object.class).invoke(list, "asd");

        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }

}
```



在程序中定义了一个`ArrayList`泛型类型实例化为`Integer`对象，如果直接调用`add()`方法，那么只能存储整数数据，不过当我们利用反射调用`add()`方法的时候，却可以存储字符串，这说明了`Integer`泛型实例在编译之后被擦除掉了，只保留了原始类型。

**常用的通配符**

? 表示不确定的java类型

T（Type）表示具体的一个java类型

K V（key value）分别代表java的键值中的key value

E（element）代表Element

### 泛型的使用方式

```java
//1.泛型类
public class Generic<T>{
    private T key;
    public Generic(T key){
        this.key=key;
    }
    public T getKey(){
        return key;
    }
}

```

```java
//实例化
Generic<Integer> genericInteger=new Generic<Integer>(123456);
```

```java
//2.泛型接口
public interface Generator<T>{
    public T method();
}
```

```java
//实现泛型接口，不指定类型
class GeneratorImpl<T> implement Generator<T>{
    @Override
    public T method(){
        return null;
    }
}
```

```java
//实现泛型接口，指定类型
class GeneratorImpl<T> implement Generator<String>{
    @Override
    public T method(){
        return "hello";
    }
}
```

```java
//3.泛型方法：
public static <E> void printArray(E[] inputArray){
	for(E element:inputArray){
		System.out.printf("%s",element);
	}
}
```

```java
//创建不同类型数组：Integer，Double和Character
Integer[] intArray={1,2,3};
String[] stringArray={“Hello”，“World”}；
printArray(intArray);
printArray(stringArray);
```



### Java 关于强引用，软引用，弱引用和虚引用的区别与用法



> **jvm就像一个国家,gc就是城管,强引用就是当地人,软引用就是移民的人,地方不够就赶走，弱引用就是黑户口,哪天城管逮到就遣走,虚引用就是一个带病的黑户口,指不定哪天自己就挂了。**

![这里写图片描述](https://img-blog.csdn.net/20180606220747457?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2p1bmp1bmJhMjY4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1 强引用

> 特点：我们平常典型编码Object obj = new Object()中的obj就是强引用。通过关键字new创建的对象所关联的引用就是强引用。
> 当JVM内存空间不足，JVM宁愿抛出OutOfMemoryError运行时错误（OOM），使程序异常终止，也不会靠随意回收具有强引用的“存活”对象来解决内存不足的问题。对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为 null，就是可以被垃圾收集的了，具体回收时机还是要看垃圾收集策略。

2 软引用

> 特点：软引用通过SoftReference类实现。 软引用的生命周期比强引用短一些。
> 只有当 JVM 认为内存不足时，才会去试图回收软引用指向的对象：即JVM 会确保在抛出 OutOfMemoryError 之前，清理软引用指向的对象。
> 软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。后续，我们可以调用ReferenceQueue的poll()方法来检查是否有它所关心的对象被回收。如果队列为空，将返回一个null,否则该方法返回队列中前面的一个Reference对象。

> 应用场景：软引用通常用来实现内存敏感的缓存。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

3 弱引用

> 弱引用通过WeakReference类实现。 弱引用的生命周期比软引用短。
> 在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。由于垃圾回收器是一个优先级很低的线程，因此不一定会很快回收弱引用的对象。
> 弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

> 应用场景：弱应用同样可用于内存敏感的缓存。

4 虚引用

> 特点：虚引用也叫幻象引用，通过PhantomReference类来实现。无法通过虚引用访问对象的任何属性或函数。幻象引用仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。
> 虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。
> ReferenceQueue queue = new ReferenceQueue ();
> PhantomReference pr = new PhantomReference (object, queue);
> 程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取一些程序行动。

> 应用场景：可用来跟踪对象被垃圾回收器回收的活动，当一个虚引用关联的对象被垃圾收集器回收之前会收到一条系统通知。

### 抽象类和接口

* JDK1.8之前：
  * 语法：
    * 抽象类：方法可以有抽象的，也可以有非抽象的，有构造器
    * 接口：方法都是抽象的，属性都是常量，默认有public static final修饰
  * 设计：
    * 抽象类：同一事物的抽取，比如针对Dao层操作的封装
    * 接口：更像一种标准的制定，规定系统之间的对接
    * 例子：单体项目，分层开发，interface作为各层之间的纽带，在controller注入；在分布式项目，面向服务的开发，抽取服务service，这个时候，就会产生服务的提供者和服务的消费者两个角色，两者之间的纽带依旧是接口。
* JDK1.8之后：
  * 接口里面可以有实现的方法，注意要在方法的声明上加上default或者static（空实现）

###  ==和equals的区别

==：它的作用是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象。

当是基本数据类型==比较的是值，引用数据类型时比较的时内存地址。

equals：它的作用也是判断两个对象是否相等，它不能用于比较基本数据类型的变量

Object的equals方法：默认的实现就是比较对象的内存地址

String的equals方法：比较的时对象的值。当创建String类型的对象时，虚拟机会在常量池查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个String对象。直接使用String a=“a”，是指向常量池的地址中。String b=new String("b")是指向堆的地址，String c=a+a，我们知道他里面是final修饰的，这时它里面会有new Stringbuilder的方式new一个新对象，final String a1="a",String d=a1+a1,这个时候它不是变量了，它是常量。编译的时候常量相加的会转变成常量。

###hashCode()与equals()

hashCode的作用是获取哈希码。它返回的是一个int整数。这个哈希码的作用是确定对象再哈希表的索引位置。hashCode()是定义在Object类中的，也就是任意类中都有，另外，native关键字修饰的本地方法，也就是本质上里面调用的是c或c++实现的。hashCode（）默认是在对堆的对象产生独特值。

##  4 基本数据类型

### 几种基本数据类型？对应的封装类？各占多少字节？

java中有8种基本数据类型，分别为

6种数字类型：byte、short、int、long、float、double

1种字符类型：char

1种布尔型：boolean

![](.\images\shuju6.png)

### 引用数据类型

| 基本类型 | 引用类型  |
| -------- | --------- |
| boolean  | Boolean   |
| byte     | Byte      |
| short    | Short     |
| int      | Integer   |
| long     | Long      |
| float    | Float     |
| double   | Double    |
| char     | Character |

此外，BigInteger、BigDecimal 用于高精度的运算，BigInteger 支持任意精度的整数，也是引用类型，但它们没有相对应的基本类型。



### 自动装箱和自动拆箱

简单一点说，装箱就是 自动将基本数据类型转换为包装器类型；拆箱就是 自动将包装器类型转换为基本数据类型。

```java
Integer i = ``10``; ``//装箱``
int` `n = i;  ``//拆箱
```

注意：

```java
public static Integer valueOf(int i) {//装箱
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
}
```

```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

* Integer引用类型和int数值比较是否相同，看数值

* Integer引用类型和Integer引用类型比较是否相同，看边界
  * 在-128到127范围内时从IntegerCache缓存里拿出来，所以当数值相同时，引用也相同，超过这个范围Integer是new的形式开辟新内存。

## 5 修饰符

### final 

* final修饰的类，不能被继承，final类中的所有成员方法都会被隐式指定为final方法
* final 修饰方法，表示该方法不可重写
* final修饰变量，表示这个变量是常量
* 注意final
  * 修饰的基本数据类型，那么这个值本身不能修改；
  * 修饰的是引用类型，引用的指向内存地址无法改变。（但是该内存地址所指向的那个对象还是可以变的）

### static

* **修饰成员变量和成员方法:** 被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。调用格式：`类名.静态变量名` `类名.静态方法名()`
* **静态代码块:** 静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块只执行一次.
* **静态内部类（static修饰类的话只能修饰内部类）：** 静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：1. 它的创建是不需要依赖外围类的创建。2. 它不能使用任何外围类的非static成员变量和方法。
* **静态导包(用来导入类中的静态资源，1.5之后的新特性):** 格式为：`import static` 这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法。

### this

* this关键字用于引用类的当前实例

### super

* super关键字用于从子类访问父类的变量和方法
* 注意：
  * this、super不能用在static方法中。因为**this和super是属于对象范畴的东西，而静态方法是属于类范畴的东西**

## 6 方法

### 重载和重写

* **重载**：发生在一个类里面，方法名相同，但是参数列表中的参个数、类型、顺序的导致参数列表不同，调用这些相同名方法的时候就会发生重载。本质上就是根据输入的参数列表不同，做不同的处理。

  * 特别注意的是：1当参数一样，顺序不一样也是重载;
  * 2它跟返回类型没关系，假设返回值作为判定是否重载！如果参数列表相同，方法名相同，返回值类型不同，那么调用该函数时无法知道应该返回哪个返回值，这个时候也会报红线。
  * 3如果参数列表不一样，返回值不同和相同也没什么区别，它还是重载成功。

  

* **重写**：发生在父类子类之间，方法名相同，参数列表相同。当子类调用使用这种方法的时候会覆盖父类的方法。

## 7 类和对象

### 面向对象和面向过程

**面向过程**是一种自顶向下的编程。

* 面向过程优点：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源;比如单片机、嵌入式开发、 Linux/Unix等一般采用面向过程开发，性能是最重要的因素。

* 缺点：没有面向对象易维护、易复用、易扩展

**面向对象**是将事物高度抽象化。面向对象必须先建立抽象模型，之后直接使用模型就行了，注重对象和职责，不同的对象承担不同的职责。

* 优点：由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统 易维护、易复用、易扩展。

* 缺点：性能比面向过程低

###构造器可以被重写吗？

构造器是不能被继承的，因为每个类的类名都不相同，而构造器名称与类名相同，所以根本谈不上继承。 

又由于构造器不能继承，所以就不能被重写。但是，在同一个类中，构造器是可以被重载的。

###一个类的构造方法的作用是什么？若一个类没有声明构造方法，该程序能正确执行吗？why？

构造方法的作用是完成对类对象的初始化。

可以执行，因为即使没有声明它也会默认携带无参构造器，但是如果我们自己写了构造方法，java就不会默认添加无参构造方法了，这个时候我们的new（）就没办法成功，所以需要无参构造的要特别注意写无参构造方法。

特别注意的是：比如父类和子类的构造的时候，java默认的在调用子类构造方法前先调用父类的构造方法，如果你没有指定调用父类的哪个构造方法，那么java默认调用父类无参数的构造方法（也可以用super指定是父类有参构造方法解决）。如果你已经在父类写了有参构造方法，父类不会默认添加无参构造方法，子类去调用的时候构造不出父类，从而也构造不出他自己了。



###面向对象的三大特征

封装、继承、多态

* 封装是指把一个对象的属性隐藏在对象内部，不允许外部对象直接俄访问，但是可以提供一些可以被外界访问的方法来操作属性。
* 继承是指新类在旧类的基础上创建新类。这种新类叫子类，1子类拥有符类对象的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法，子类只是拥有但是无法访问。2子类可以对父类进行扩展，拥有自己的属性和方法。3子类可以用重写父类的方法。
* 多态的意思多态是同一个行为具有多个不同表现形式或形态的能力。实现方式有重写、接口、抽象类和抽象方法

## 8 常见类

### Object

Java中所有的类都继承自`java.lang.Object`类，Object类中一共有11个方法：

```java
public final native Class<?> getClass();

public native int hashCode();

public boolean equals(Object obj) {
    return (this == obj);
}

protected native Object clone() throws CloneNotSupportedException;

public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}

public final native void notify();

public final native void notifyAll();

public final native void wait(long timeout) throws InterruptedException;

public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
                            "nanosecond timeout value out of range");
    }

    if (nanos > 0) {
        timeout++;
    }

    wait(timeout);
}

public final void wait() throws InterruptedException {
    wait(0);
}

protected void finalize() throws Throwable { }
复制代码
```

####getClass方法

这是一个native方法，并且是'final'的，也就是说这个方法不允许在子类中覆写。 getClass方法返回的是当前实例对应的Class类，也就是说不管一个类有多少个实例，每个实例的getClass返回的Class对象是一样的。请看下面的例子:

```java
Integer i1 = new Integer(1);
Class i1Class = i1.getClass();

Integer i2 = new Integer(1);
Class i2Class = i2.getClass();
System.out.println(i1Class == i2Class);
复制代码
```

上面的代码运行结果为`true`，也就是说两个Integer的实例的getClass方法返回的Class对象是同一个。

####Integer.class和int.class

Java中还有一个方法可以获取Class，例如我们想获取一个Integer实例对应的Class，可以直接通过`Integer.class`来获取，请看下面的例子：

```java
Integer num = new Integer(1);
Class numClass = num.getClass();
Class integerClass = Integer.class;
System.out.println(numClass == integerClass);
复制代码
```

上面代码的运行结果为`true`，也就是说通过调用实例的getClass方法和`类.class`返回的Class是一样的。与Integer对象的还有int类型的原生类，与`Integer.class`对应，`int.class`用于获取一个原生类型int的Class。但是他们两个返回的不是同一个对象。请看下面的例子：

```java
Class intClass = int.class;
Class intTYPE = Integer.TYPE;
Class integerClass = Integer.class;
System.out.println(intClass == integerClass);
System.out.println(intClass == intTYPE);
复制代码
```

上面的代码运行结果是：

```java
false
true
复制代码
```

`Integer.class`和`int.class`返回的不是一个对象，而`int.class`返回的和`Integer.TYPE`是同一个对象。`Integer.TYPE`定义如下：

```java
public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");
复制代码
```

**Java中为原生类型(boolean,byte,char,short,int,long,float,double)和void都创建了一个预先定义好的类型，可以通过包装类的TYPE静态属性获取。上述的Integer类中TYPE和`int.class`是等价的。**

####hashCode方法

对象的哈希码主要用于在哈希表中的存放和查找等。Java中对于对象hashCode方法的规约如下：

1. 在java程序执行过程中，在一个对象没有被改变的前提下，无论这个对象被调用多少次，hashCode方法都会返回相同的整数值。对象的哈希码没有必要在不同的程序中保持相同的值。
2. 如果2个对象使用equals方法进行比较并且相同的话，那么这2个对象的hashCode方法的值也必须相等。
3. 如果根据equals方法，得到两个对象不相等，那么这2个对象的hashCode值不需要必须不相同。但是，不相等的对象的hashCode值不同的话可以提高哈希表的性能。

为了理解这三条规约，我们来先看一下HashMap中put一个entry的过程：

```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) // 这里key的hashcode被用来定位当前entry在哈希表中的index
        tab[i] = newNode(hash, key, value, null); // 如果当前哈希表中没有key对应的entry，则直接插入
    else {
        // 当前哈希表中已经有了key对应的entry了，则找到这个节点，然后看是否需要更新这个entry的value
        Node<K,V> e; K k;
        // 判断当前节点就是已经存在的entry的条件：1：hashcode相等；2：当前节点的key和key是同一个对象（==）或者两者的equals方法判定相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 是否需要更新旧值，如果需要更新则更新
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
复制代码
```

HashMap中put一个键值对时有以下几个步骤：

1. 计算key的哈希码，通过哈希码定位新增的entry应该处于哈希表中的哪个位置，如果当前位置为null，则代表哈希表中没有key对应的entry，直接新插入一个节点就好了。
2. 如果当前key在哈希表中已经有了映射，则先查找这个节点，判定当前是否为目标节点的条件有两个：**1）两者的哈希码必须相等；2）两者是同一个对象（==成立），或者两者的equals方法判定两者相等。**
3. 判断是否需要更新旧值，需要的话就更新。

分析完了HashMap的put操作后，我们再来看看这三条规约：

1. 第一条规约要求多次调用一个对象的hashCode方法返回的值要相等。设想如果一个key的hashCode方法每次返回值如果不同，则在put的时候就可能定位到哈希表中不同的位置，就产生了歧义：明明两个key是同一个，但是哈希表中存在同一个key的多个不同映射。这就违背了哈希表的key不能重复的原则了。
2. 第二条规约也很好理解：如果两个key的equals方法判定两者相等，则说明哈希表中只需要保留一个key就行了。如果equals判定相等，而hashcode不同，则就会违背这个事实。
3. 第三条规约是用来优化哈希表的性能的，如果哈希表put时"碰撞"太多，势必会造成查找性能下降。

####equals方法

equals方法用于判定两个对象是否相等。Object中的equals方法其实默认比较的是两个对象是否拥有相同的地址。也就是"=="对应的内存语义。

但是在子类中我们可以覆写equals来判定子类的两个实例是否为同一个对象。例如对于一个人来说，他有很多属性，但是每个人的id是唯一的，如果两个人的id一样则证明两个人是同一个人，而不管其他属性是否一样。这个时候我们就可以覆写人的equals方法来保证这点了。

```
public class Person {
    
    private long id;
    private String name;
    private int age;
    private String nation;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return id == person.id; // 如果两个person的id相同，则我们认为他们是同一个对象
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
复制代码
```

equals方法在非空对象引用上的特性：

1. reflexive，自反性。任何非空引用值x，对于x.equals(x)必须返回true
2. symmetric，对称性。任何非空引用值x和y，如果x.equals(y)为true，那么y.equals(x)也必须为true
3. transitive，传递性。任何非空引用值x、y和z，如果x.equals(y)为true并且y.equals(z)为true，那么x.equals(z)也必定为true
4. consistent，一致性。任何非空引用值x和y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改
5. 对于任何非空引用值 x，x.equals(null) 都应返回 false

Java要求一个类的**equals方法和hashCode方法同时覆写**。我们刚刚分析了HashMap中对于key的处理过程：首先根据key的哈希码定位哈希表中的位置，其次根据"=="或者equals方法判定两个key是否相同。如果Person的equals方法没有被覆写，则两个Person对象即使id一样，但是不是指向同一块内存地址，那么哈希表中就查找不到已经存在的映射entry了。

####clone方法

用于克隆一个对象，被克隆的对象需要implements Cloneable接口，否则调用这个对象的clone方法，将会抛出CloneNotSupportedException异常。克隆的对象通常情况下满足以下三条规则：

1. x.clone() != x，克隆出来的对象和原来的对象不是同一个，指向不同的内存地址
2. x.clone().getClass() == x.getClass()
3. x.clone().equals(x)

**一个对象进行clone时，原生类型和包装类型的field的克隆原理不同。对于原生类型是直接复制一个，而对于包装类型，则只是复制一个引用而已，并不会对引用类型本身进行克隆。**

#####浅拷贝

浅拷贝例子：

```
public class ShallowCopy {

    public static void main(String[] args){
        Man man = new Man();
        Man manShallowCopy = (Man) man.clone();
        System.out.println(man == manShallowCopy);
        System.out.println(man.name == manShallowCopy.name);
        System.out.println(man.mate == manShallowCopy.mate);
    }
}

class People implements Cloneable {

    // primitive type
    public int id;
    // reference type
    public String name;

    @Override
    public Object clone() {
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }
}

class Man extends People implements Cloneable {
    // reference type
    public People mate = new People();

    @Override
    public Object clone() {
        return super.clone();
    }
}
复制代码
```

上面的代码的运行结果是:

```
false
true
true
复制代码
```

通过浅拷贝出来的Man对象manShallowCopy的name和mate属性和原来的对象man都指向了相同的内存地址。在对Man的name和mate进行拷贝时浅拷贝只是对引用进行了拷贝，指向的还是同一块内存地址。

#####深拷贝

对一个对象深拷贝时，对于对象的包装类型的属性，会对其再进行拷贝，从而达到深拷贝的目的，请看下面的例子：

```
public class DeepCopy {

    public static void main(String[] args){
        Man man = new Man();
        Man manDeepCopy = (Man) man.clone();
        System.out.println(man == manDeepCopy);
        System.out.println(man.mate == manDeepCopy.mate);
    }
}

class People implements Cloneable {

    // primitive type
    public int id;
    // reference type
    public String name;

    @Override
    public Object clone() {
        try {
            return super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }
}

class Man extends People implements Cloneable {
    // reference type
    public People mate = new People();

    // 深拷贝Man
    @Override
    public Object clone() {
        Man man = (Man) super.clone();
        man.mate = (People) this.mate.clone(); // 再对mate属性进行clone，从而达到深拷贝
        return man;
    }
}
复制代码
```

上面代码的运行结果为：

```
false
false
复制代码
```

Man对象的clone方法中，我们先对Man进行了clone，然后对mate属性也进行了拷贝。因此man的mate和manDeepCopy的mate指向了不同的内存地址。也就是深拷贝。

**通常来说对一个对象进行完完全全的深拷贝是不现实的，例如上面的例子中，虽然我们对Man的mate属性进行了拷贝，但是无法对name（String类型）进行拷贝，拷贝的还是引用而已。**

####toString方法

Object中默认的toString方法如下：

```
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
复制代码
```

也就是class的名称+对象的哈希码。一般在子类中我们可以对这个方法进行覆写。

####notify方法

notify方法是一个final类型的native方法，子类不允许覆盖这个方法。

notify方法用于唤醒正在等待当前对象监视器的线程，唤醒的线程是随机的。一般notify方法和wait方法配合使用来达到多线程同步的目的。 在一个线程被唤醒之后，线程必须先重新获取对象的监视器锁（线程调用对象的wait方法之后会让出对象的监视器锁），才可以继续执行。

一个线程在调用一个对象的notify方法之前必须获取到该对象的监视器(synchronized)，否则将抛出IllegalMonitorStateException异常。同样一个线程在调用一个对象的wait方法之前也必须获取到该对象的监视器。

wait和notify使用的例子：

```
Object lock = new Object();
Thread t1 = new Thread(() -> {
    synchronized (lock) {
        System.out.println(Thread.currentThread().getName() + " is going to wait on lock's monitor");
        try {
            lock.wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " relinquishes the lock's monitor");
    }
});
Thread t2 = new Thread(() -> {
    synchronized (lock) {
        System.out.println(Thread.currentThread().getName() + " is going to notify a thread that waits on lock's monitor");
        lock.notify();
    }
});
t1.start();
t2.start();
复制代码
```

####notifyAll方法

notifyAll方法用于唤醒所有等待对象监视器锁的线程，notify只唤醒所有等待线程中的一个。

同样，如果当前线程不是对象监视器的所有者，那么调用notifyAll同样会发生IllegalMonitorStateException异常。

####wait方法

```
public final native void wait(long timeout) throws InterruptedException;
复制代码
```

wait方法一般和上面说的notify方法搭配使用。一个线程调用一个对象的wait方法后，线程将进入`WAITING`状态或者`TIMED_WAITING`状态。直到其他线程唤醒这个线程。

线程在调用对象的wait方法之前必须获取到这个对象的monitor锁，否则将抛出IllegalMonitorStateException异常。线程的等待是支持中断的，如果线程在等待过程中，被其他线程中断，则抛出InterruptedException异常。

如果wait方法的参数timeout为0，代表等待过程是不会超时的，直到其他线程notify或者被中断。如果timeout大于0，则代表等待支持超时，超时之后线程自动被唤醒。

####finalize方法

```
protected void finalize() throws Throwable { }
复制代码
```

垃圾回收器在回收一个无用的对象的时候，会调用对象的finalize方法，我们可以覆写对象的finalize方法来做一些清除工作。下面是一个finalize的例子：

```
public class FinalizeExample {

    public static void main(String[] args) {
        WeakReference<FinalizeExample> weakReference = new WeakReference<>(new FinalizeExample());
        weakReference.get();
        System.gc();
        System.out.println(weakReference.get() == null);
    }

    @Override
    public void finalize() {
        System.out.println("I'm finalized!");
    }
}
复制代码
```

输出结果：

```
I'm finalized!
true
复制代码
```

finalize方法中对象其实还可以"自救"，避免垃圾回收器将其回收。在finalize方法中通过建立this的一个强引用来避免GC：

```
public class FinalizeExample {

    private static FinalizeExample saveHook;

    public static void main(String[] args) {
        FinalizeExample.saveHook = new FinalizeExample();
        saveHook = null;
        System.gc();
        System.out.println(saveHook == null);
    }

    @Override
    public void finalize() {
        System.out.println("I'm finalized!");
        saveHook = this;
    }
}
复制代码
```

输出结果：

```
I'm finalized!
false
```



### String、StringBuffer、StringBuilder区别

String跟其它两个类的区别是

> String是final类型，每次声明都是不可变的对象
>
> 所以每次操作都会产生新的String对象，然后将指针指向新的String对象



StringBuffer、StringBuilder都是在原有对象上进行操作

> 所以，如果需要经常改变字符串内容，则建议采用这两者。

StringBuffer vs StringBuilder

> 前者是线程安全的，后者是线程不安全的。
>
> 线程不安全性能更高，所以开发中，优先采用StringBuilder

9异常

10文件和io

11反射

12类加载机制

13类初始化的对象创建过程

14元注解







#查漏补缺

##序列化和反序列化

比如，ArrayList 实现**java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。

那什么是序列化和反序列化

**把对象转换为字节序列的过程称为对象的序列化。**

**把字节序列恢复为对象的过程称为对象的反序列化**。
　　对象的序列化主要有两种用途：
　　1） 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；
　　2） 在网络上传送对象的字节序列。

　　在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便长期保存。比如最常见的是Web服务器中的Session对象，当有 10万用户并发访问，就有可能出现10万个Session对象，内存可能吃不消，于是Web容器就会把一些seesion先序列化到硬盘中，等要用了，再把保存在硬盘中的对象还原到内存中。

　　当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个Java对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为Java对象。

##java中同步和异步有什么异同？

同步交互：指发送一个请求,需要等待返回,然后才能够发送下一个请求，有个等待过程；

异步交互：指发送一个请求,不需要等待返回,随时可以再发送下一个请求，即不需要等待。 区别：一个需要等待，一个不需要等待，在部分情况下，我们的项目开发中都会优先选择不需要等待的异步交互方式。









