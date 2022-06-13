# 补充

## 说说Comparable和Comparator

Comparable是排序接口，类实现了该接口，这说明该类本身支持排序，而不需要指定排序器
通过`x.compareTo(y)`来比较x和y的大小

Comparator是比较器接口，我们若需要控制某个类的次序，但该类本身不支持排序，就可以通过该接口建立该类的比较器，从而进行外部比较
通过`int compare(T o1, T o2)`来实现外部比较

## 基本类型和包装类的区别



## Java NIO学过嘛，讲讲三大组件？（Selector、Channel、Buffer）


**Buffer缓冲区**

**Buffer最终的实现是一个内存中的字节数组，所以一个Buffer本质上是内存中的一块区域**，我们可以将数据写入这块内存，之后从这块内存获取数据。

> 传统IO模式中，是从Stream流中逐字节地读取数据的，且只能顺序读取，不能回溯


**Channel通道**

Channel是对传统IO包中的`Stream`的模拟，通过Channel这个对象，我们可以**读取和写入数据**，并且**通过Channel的所有发送数据都要读到缓冲区Buffer中，所有要接收的数据都要写到缓冲区Buffer里**。

Channel与Stream的区别：
- 流是单向的，读写分离，通道是双向的，可读可写
- 流读写是阻塞的，通道可以异步读写
- 流中的数据可以选择性的先读到缓存中，通道的数据总是要先读到一个缓存中，或从缓存中写入


**Selector选择器**

Java实现IO多路复用的核心机制。

它做到了一个线程管理多个Channel：可以向Selector注册感兴趣的事件，当事件就绪，通过 `Selector.select()`方法获取注册的事件，进行相应的操作。

Selector通过事件循环的机制来处理读写事件：
1. 首先你需要把你的连接，也就是 Channel 绑定到 Selector 上
2. 在接收数据的线程来调用 Selector.select() 方法来等待数据到来
   - select是阻塞方法，会阻塞线程
3. Selector.select()方法的返回值是一个迭代器，可以从这个迭代器里面获取所有 Channel 收到的数据，然后执行业务逻辑 

## Buffer缓冲区原理（Buffer对象，数组/内存块，直接写入，写入读出Channel过程






## 如何中断一个正在运行的线程？如果不响应interrupt呢？


# 基础

## Java语言有哪些特点？

1. 面向对象（**封装，继承，多态**）；

2. 平台无关性，**一次编写，到处运行**（通过JVM保证，在不同平台上不需要重新编译）

3. 可靠性、安全性；

4. 支持多线程。C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计；

5. 支持网络编程并且很方便。Java 语言诞生本身就是为简化网络编程设计的，因此 Java 语言不仅支持网络编程而且很方便；

6. **编译与解释并存**；

## Java和C++有什么关系，它们有什么区别？

* 都是面向对象的语言，都支持封装、继承和多态；
* C++ 支持指针，而 Java 没有指针的概念；
* C++ 支持**多继承**，而 Java 不支持多重继承，但**允许一个类实现多个接口**；
* Java的方法默认为虚方法，但C++中的虚方法需要进行额外标注
* Java **自动进行无用内存回收**操作，不再需要程序员进行手动删除，而 C++ 中必须由程序释放内存资源，这就增加了程序员的负担。
* Java **不支持操作符重载**，操作符重载则被认为是 C++ 的突出特征；
* Java 允许预处理，但不支持预处理器功能，所以为了实现预处理，它提供了引入语句（import），但它与 C++ 预处理器的功能类似；
* Java 不支持缺省参数函数，而 C++ 支持；
* C 和 C++ 不支持**字符串变量**，在 C 和 C++ 程序中使用“Null”终止符代表字符串的结束。在 Java 中字符串是用类对象（String 和 StringBuffer）来实现的；
* goto 语句是 C 和 C++ 的“遗物”，Java 不提供 goto 语句，虽然 Java 指定 goto 作为关键字，但不支持它的使用，这使程序更简洁易读；
* Java 不支持 C++ 中的**自动强制类型转换**，如果需要，必须由程序显式进行强制类型转换。

## JVM、JRE和JDK的关系是什么？

JDK是（Java Development Kit）的缩写，它包含JRE、编译器（javac）和工具（如 javadoc 和 jdb）。它能够创建和编译程序。

JRE是Java Runtime Environment缩写，它是运行已编译 Java 程序所需的所有内容的集合，包括 **Java 虚拟机（JVM），Java 类库，java 命令和其他的一些基础构件**。但是，它不能用于创建新程序。

JDK包含JRE，JRE包含JVM。

<img src="http://blog-img.coolsen.cn/img/image-20210219163725268.png" alt="image-20210219163725268" style="zoom: 67%;" />

## **什么是字节码?**

> 这个问题，面试官可以扩展提问，Java 是编译执行的语言，还是解释执行的语言?

Java之所以可以**一次编译，到处运行**，主要有两个原因：
1. JVM针对各种操作系统、平台都进行了定制
2. 无论在什么平台，都可以编译生成**固定格式的字节码（.class文件）供JVM使用**。

字节码是通过javac命令对源代码进行编译生成的数据，是程序的一种低级表示，以`.class`文件的形式储存。

字节码是JVM 执行引擎的数据入口，由**十六进制值**组成，JVM以两个十六进制值为一组，即**以字节为单位**进行读取。JVM位于Java编译器与OS平台之间，将字节码翻译成成真实机器所能理解的二进制机器码。

## 采用字节码的好处是什么?

1. 实现了**解释+编译**。
   - 通过字节码，Java在一定程度上解决了传统**解释型**语言执行效率低的问题，同时保留了**解释型**语言可移植的特点。所以Java程序运行时比较高效，
2. 字节码并不专对一种特定的机器，因此，Java程序无须重新编译便可在多种不同的计算机上运行。

## Oracle JDK 和 OpenJDK 的区别是什么？

可能在看这个问题之前很多人和我一样并没有接触和使用过 OpenJDK 。下面通过我通过我收集到一些资料对你解答这个被很多人忽视的问题。

* Oracle JDK 版本将每三年发布一次，而 OpenJDK 版本每三个月发布一次；
* OpenJDK 是一个参考模型并且是完全开源的，而 Oracle JDK 是OpenJDK 的一个实现，并不是完全开源的；
* Oracle JDK 比 OpenJDK 更稳定。OpenJDK 和 Oracle JDK 的代码几乎相同，但 Oracle JDK 有更多的类和一些错误修复。因此，如果您想开发企业/商业软件，建议选择 Oracle JDK，因为它经过了彻底的测试和稳定。某些情况下，有些人提到在使用 OpenJDK 可能会遇到了许多应用程序崩溃的问题，但是，只需切换到 Oracle JDK 就可以解决问题；
* 在响应性和 JVM 性能方面，Oracle JDK 与 OpenJDK 相比提供了更好的性能；
* Oracle JDK 不会为即将发布的版本提供长期支持，用户每次都必须通过更新到最新版本获得支持来获取最新版本；
* Oracle JDK 根据二进制代码许可协议获得许可，而 OpenJDK 根据 GPLv2 许可获得许可。

## Java 与 Python 的区别？

1. PVM的功能不如JVM全面，JVM是Java的核心，python的核心是可以很方便的使用C和C++库
2. python的变量是动态的，而java的变量是静态的，需要事先声明
3. Java的设计遵循面向对象的思想，但python支持面向对象与面向过程两种方式

# 基础语法

## Java有哪些数据类型？

Java 语言的数据类型分为两种：基本数据类型和引用数据类型。

后者包含interface、class、数组

| 基本类型 | 位数 | 字节 | 默认值  |
| -------- | ---- | ---- | ------- |
| int      | 32   | 4    | 0       |
| short    | 16   | 2    | 0       |
| long     | 64   | 8    | 0L      |
| byte     | 8    | 1    | 0       |
| char     | 16   | 2    | 'u0000' |
| float    | 32   | 4    | 0f      |
| double   | 64   | 8    | 0d      |
| boolean  | 1    |      | false   |

> 对于 boolean，官方文档未明确定义，它依赖于 JVM 厂商的具体实现。
> 逻辑上理解是占用 `1` 位，但是实际中会考虑计算机高效存储因素。
> 
> Java虚拟机规范讲到：在JVM中并没有提供boolean专用的字节码指令，而**boolean类型数据在经过编译后在JVM中会通过int类型来表示**，此时boolean数据4字节32位，而boolean数组将会被编码成Java虚拟机的byte数组，**此时每个boolean数据1字节占8bit。**

## int是如何表示负数的，char能否存汉字

负数是通过补码的方式保存的：第一步，每一个二进制位都取相反值；第二步，将上一步得到的值加1。

符号扩展（Sign Extension）用于在数值类型转换时扩展二进制位的长度，以保证转换后的数值和原数值的符号（正或负）和大小相同，一般用于较窄的类型（如byte）向较宽的类型（如int）转换。扩展二进制位长度指的是，在原数值的二进制位左边补齐若干个符号位（0表示正，1表示负）。

> 举例来说，如果用6个bit表示十进制数10，二进制码为`00 1010`，如果将它进行符号扩展为16bits长度，结果是`0000 0000 0000 1010`，即**在左边补上10个0**（因为10是正数，符号为0），符号扩展前后数值的大小和符号都保持不变；
> 如果用10bits表示十进制数-15，使用“2的补码”编码后，二进制码为`11 1111 0001`，如果将它进行符号扩展为16bits，结果是`1111 1111 1111 0001`,即**在左边补上6个1（因为-15是负数，符号为1）**，符号扩展前后数值的大小和符号都保持不变。

- 如果最初的数值类型是有符号的，那么就执行**符号扩展**；
- 如果是`char`类型，那么不管它要被转换成什么类型，都执行**零扩展**。
- 如果目标类型的长度小于源类型的长度，则直接截取目标类型的长度。
   例如将int型转换成byte型，直接截取int型的右边8位

如果想要除去符号扩展，可以如下操作：`int i = s & 0xffff`：s为`short`类型，通过`0xffff`的与操作，将s的前16位强制设置为0，即可去除符号扩展

## 对应的数据类型的取值范围，比如 int 类型，对应的取值范围如何计算的

在java中，int是通过32位表示的，且可以表示正负数，而负数是通过补码表示的，因此最高位用于符号位，所以数值范围为`-2^31 ~ 2^31 - 1`。

其中，最大正数为`2^31 - 1`，是因为当取到`0111..11`时，此时的值即`2^31-1`，如果再+1，就会变成`100..0`，即负数


## switch 是否能作用在 byte 上，是否能作用在 long 上，是否能作用在 String 上？

Java5 以前 switch(expr)中，expr 只能是 `byte、short、char、int`。

从 Java 5 开始，Java 中引入了`枚举类型`， expr 也可以是 enum 类型。

从 Java 7 开始，expr还可以是`字符串(String)`

> 长整型(long)在目前所有的版本中都是不可以的，这是为了保证switch的精度（感觉有点扯）

## **访问修饰符public、private、protected、以及不写（默认）时的区别**？

Java中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。Java 支持 4 种不同的访问权限。

- **default** (即默认，什么也不写）: 在**同一包**内可见，不使用任何修饰符。使用对象：类、接口、变量、方法。
- **private** : 在**同一类**内可见。使用对象：变量、方法。 **注意：不能修饰类（外部类）**
- **public** : 对**所有类**可见。使用对象：类、接口、变量、方法
- **protected** : 对**同一包内的类和所有子类**可见。使用对象：变量、方法。 **注意：不能修饰类（外部类）**。

![image-20210219173433142](http://blog-img.coolsen.cn/img/image-20210219173433142.png)



## Arrays.sort的底层策略

1. 数组元素个数小于47，使用**插入排序**；
2. 数组元素个数大于等于47，小于286，使用**双轴快速排序**；
3. 数组元素个数大于等于286，使用**归并排序**，当然，若数组**降序组**太多，又会使用**快速排序（快排数据越无序越快）**；

> 对于小数组来说，插入排序效率更高，每次递归到小于47的大小时，用插入排序代替快排，明显提升了性能
> 双轴快排使用两个`pivot`，每轮把数组分成3段，在没有明显增加比较次数的情况下巧妙地减少了递归次数。思想;
> 1. 将数组长度/7，从数组中选取 5 个均匀间隔的元素，并将这 5 个元素进行插入排序
> 2. 使用五个排序元素中的第二个和第四个作为枢轴pivot，且 pivot1 <= pivot2
> 3. 定义两个指针 `less` 与 `great` ，`less` 从数组最左端向右遍历，直到找到**第一个>=pivot1的元素**，great 从数组最右端向左遍历，直到找到**第一个<=pivot2的元素**。
> 4. 此时再将 `less` 与 `great` 中间的元素进行移动，**将小于枢纽1的元素移至枢纽1左边**，**将大于枢纽2的元素移至枢纽2右边**，最后位于两个枢纽之间的元素就是 pivot1 <= && <= pivot2 的元素了。    
> 5. 再对枢纽左中右三部分进行递归快排（中间部分右特殊处理）。同样的，如果这些部分的数组长度小于47，又会改为插入排序。

注意：上述是对基本数据类型的排序策略，如果是`Object及其子类`，则使用归并排序，而不是快排
> 1. 如果数组元素个数小于`MIN_MERGE(32)`，那么就会调用**二分插入排序binarySort**方法进行排序。
>   所谓二分排序，是指在插入的过程中，使用二分查找的方法查找待插入的位置，这种查找方法会比线性查找快一点。
> 2. 如果大于MIN_MERGE，则将数组**划分成多个有序块进行归并排序**。

使用归并排序的原因：快速排序不是稳定的，而归并排序是稳定的。

> 对于基本数据类型，稳定性没有意义，而对于对象类型，稳定性是比较重要的，因为对象相等的判断可能只是判断关键属性，最好保持相等对象的非关键属性的顺序与排序前一致；
> 另外一个原因是由于**合并排序相对而言比较次数比快速排序少，移动（对象引用的移动）次数比快速排序多**，而对于对象来说，比较一般比移动耗时。


## 字符常量和字符串常量的区别？

1. 形式上: 字符常量是单引号引起的一个字符，字符串常量是双引号引起的若干个字符；

2. 含义上: 字符常量相当于一个**整型值( ASCII 值)**,可以参加表达式运算；字符串常量代表一个**地址值(该字符串在内存中存放位置，相当于对象)**；

3. 占内存大小：字符常量只占2个字节；字符串常量占若干个字节(至少一个字符结束标志) (注意: char 在Java中占两个字节)。

## 什么是字符串常量池？

java中常量池的概念主要有三个：`全局字符串常量池`，`class文件常量池`，`运行时常量池`。我们现在所说的就是`全局字符串常量池`。

jvm为了提升性能和减少内存开销，避免字符的重复创建，其维护了一块特殊的内存空间，即字符串池，当需要使用字符串时，先去字符串池中查看该字符串是否已经存在，如果存在，则可以直接使用，如果不存在，初始化，并将该字符串放入字符串常量池中。

字符串常量池的位置也是随着jdk版本的不同而位置不同。在jdk6中，常量池的位置在**永久代（方法区）**中，==此时常量池中存储的是对象==。在jdk7中，常量池的位置在**堆**中，此时，==常量池存储的就是引用==了。

## String str="aaa" 与 String str=new String("aaa") 一样吗？`new String(“aaa”);`创建了几个字符串对象?

- 使用`String a = “aaa” ;`，程序运行时会在常量池中查找”aaa”字符串，若没有，会将”aaa”字符串放进常量池，再将其地址赋给a；若有，将找到的”aaa”字符串的地址赋给a。
- 使用`String b = new String("aaa");`，程序会在堆内存中开辟一片新空间存放新对象`str`，同时会将`aaa`字符串放入常量池，相当于**创建了两个对象**。无论常量池中有没有”aaa”字符串，程序都会在堆内存中开辟一片新空间存放新对象。

具体分析，见以下代码：

```java
 @Test
    public void test(){
        String s = new String("2");
        s.intern();
        String s2 = "2";
        System.out.println(s == s2);


        String s3 = new String("3") + new String("3");
        //如果这里加一个String s5 = new String("33");，则下面返回false
        s3.intern();
        String s4 = "33";
        System.out.println(s3 == s4);
    }
```

运行结果：

```java
jdk6
false
false

jdk7
false
true
```
**先来认识下intern()函数**：

　　`intern`函数的作用是**返回对常量池中的String的引用**，在JDK1.6以前 和 JDK1.7以后有不同的处理；

- 在JDK1.6中，`intern`的处理是 先判断字符串常量是否在字符串常量池中，如果存在直接返回该常量，如果没有找到，则**将该字符串常量加入到字符串常量区，返回对该常量的引用**；

- 在JDK1.7中，`intern`的处理是 先判断字符串常量是否在字符串常量池中，如果存在直接返回该常量，如果没有找到，**说明该字符串常量在堆中**，则处理是**把堆区该对象的引用加入到字符串常量池中**，拿到的是对常量池中该字符串对象的引用，实际存在堆中

```java
//这里创建了一个StringObject和一个`helloworld`对象
String str3 = new StringBuilder("hello").append("world").toString();
String str4 = str3.intern();
System.out.println(str3 == str4);// 结果是true，
```

**JDK1.6**

- `String s = new String("2");`：创建了**两个对象**，一个在堆中的`StringObject`对象，一个是在常量池中的“2”对象。==此时常量池中已经有`2`字符串的对象了==
- `s.intern();`：在常量池中寻找与s变量内容相同的对象，发现已经存在内容相同对象“2”，返回对象2的地址。
- `String s2 = "2";`：使用字面量创建，在常量池寻找是否有相同内容的对象，发现有，**返回对象"2"在常量池中的地址**。
- `System.out.println(s == s2);`：从上面可以分析出，s变量和s2变量地址指向的是不同的对象，所以返回`false`
 
---

- `String s3 = new String("3") + new String("3");`：创建了两个对象，一个在堆中的`StringObject`对象`s3`，一个是在常量池中的“3”对象，==但此时常量池中没有`33`对象==。
  - 如果算上2个匿名的`new String("3")`，那应该是4个对象？
- `s3.intern();`：在常量池中寻找与s3变量内容相同的对象，没有发现“33”对象，在常量池中创建“33”对象，返回“33”对象的地址。
- `String s4 = "33";`：使用字面量创建，在常量池寻找是否有相同内容的对象，发现有，返回对象"33"的地址。
- `System.out.println(s3 == s4);`：从上面可以分析出，s3变量和s4变量地址指向的是不同的对象，所以返回`false`

**JDK1.7**

`String s = new String("2");`：创建了两个对象，一个在**堆**中的`StringObject`对象，一个是在**堆中的“2”对象**，并**在字符串常量池中保存“2”对象的引用地址**。
 `s.intern();`在常量池中寻找与s变量内容相同的对象，发现已经存在内容相同对象“2”，返回对象“2”的引用地址。
 `String s2 = "2";`使用字面量创建，在常量池寻找是否有相同内容的对象，发现有，返回对象“2”的引用地址。
 `System.out.println(s == s2);`从上面可以分析出，s变量和s2变量地址指向的是不同的对象，所以返回`false`

`String s3 = new String("3") + new String("3");`创建了**两个对象**，一个在堆中的`StringObject`对象`s3`，**一个是在堆中的“3”对象**，并在常量池中保存“3”对象的引用地址。中间还有2个匿名的new String("3")我们不去讨论它们。
 `s3.intern();`在常量池中寻找与s3变量内容相同的对象，没有发现“33”对象，说明该字符串常量在堆中，因此直接**将s3对应的StringObject对象的地址保存到常量池中**，返回StringObject对象的地址。
 `String s4 = "33";`使用字面量创建，在常量池寻找是否有相同内容的对象，发现有，返回其地址，也就是`StringObject`对象的引用地址。
 `System.out.println(s3 == s4);`从上面可以分析出，s3变量和s4变量地址指向的是相同的对象，即堆中的`StringObject`对象s3，所以返回`true`。

```java
public class TestDemo {
    
    @Test
    public void test01() {
        String str1 = new StringBuilder("ja").append("va").toString();
        String str2 = str1.intern();
        System.out.println(str1 == str2);// 结果是false
        //这里返回true，是因为在进行类加载时，就已经把一些常用字符串加载到常量池里了，例如这里的`java`

        String str3 = new StringBuilder("hello").append("world").toString();
        String str4 = str3.intern();
        System.out.println(str3 == str4);// 结果是true
--------------------------------------------------------------------
        String str1 = "a";
        String str2 = "b";
        String str5 = new String("ab");
        String str3 = "ab";
        String str4 = str1 + str2;

        System.out.println(str5.equals(str3));//true
        //内容相同，没啥问题

        System.out.println(str5 == str3);//false，
        //str5指向堆中的StringObject，而str3指向常量池中的"ab"，当然是false

        System.out.println(str5.intern() == str3);//true
        //"ab"存在于常量池，所以intern()返回的是对常量池中的引用，当然是相等的

        System.out.println(str5.intern() == str4);//false
        // str4 = str1 + str2，包含StringObject，所以str4是一个StringObject，而不是对常量池对象的引用

        System.out.println(str5.intern() == str4.intern());//true
        // 都是对常量池对象的引用
--------------------------------------------------------------------
        String a = new String("ab");
        String b = new String("ab");
        String c = "ab";
        String d = "a" + "b";
        String e = "b";
        String f = "a" + e;

        System.out.println(b.intern() == a);//false
        System.out.println(b.intern() == c);//true
        System.out.println(b.intern() == d);//true
        System.out.println(b.intern() == f);//false
        // 由于f = "a" + e，包含了StringObject，所以f也是一个StringObject，而不是对常量池对象的引用

        System.out.println(b.intern() == a.intern());//true
    }
}
```

## String 是最基本的数据类型吗?

不是。Java 中的基本数据类型只有 8 个 ：byte、short、int、long、float、double、char、boolean；除了基本类型（primitive type），剩下的都是引用类型（referencetype），Java 5 以后引入的枚举类型也算是一种比较特殊的引用类型。

## String有哪些特性?

* **不变性**：String 是只读字符串，是一个典型的 immutable 对象，**对它进行任何操作，其实都是创建一个新的对象，再把引用指向该对象**。
  * 不变模式的主要作用在于**当一个对象需要被多线程共享并频繁访问时，可以保证数据的一致性**；

* **常量池优化**：String 对象创建之后，会在字符串常量池中进行缓存，如果下次创建同样的对象时，会直接返回缓存的引用；

* **final**：使用 final 来定义 String 类，表示 String 类不能被继承，提高了系统的安全性。

## 在使用 HashMap 的时候，用 String 做 key 有什么好处？

HashMap 内部实现是通过 key 的 hashcode 来确定 value 的存储位置，因为字符串是不可变的，所以当创建字符串时，**它的 hashcode 被缓存下来，不需要再次计算**，所以相比于其他对象更快。


# 关键字

## final、finally、finalize的区别？

final 用于修饰变量、方法和类。

- final 变量：被修饰的变量**不可变**，不可变分为`引用不可变`和`对象不可变`，final 指的是**引用不可变**，即对象的引用不可变，但对象的内容是可变的，**final 修饰的变量必须初始化**，且只能被赋值一次，通常称被修饰的变量为`常量`。
- final 方法：被修饰的**方法不允许任何子类重写**，且在编译时静态绑定（应该是在解析时就完成了方法调用）。
  - private方法被隐式的指定为final方法
- final 类：被修饰的**类不能被继承**，**所有方法不能被重写**。
  - final类中的所有成员方法都会被隐式的指定为final方法

> 在匿名类中所有变量都必须是final变量

finally 作为异常处理的一部分，它只能在 `try/catch` 语句中，并且附带一个语句块表示这段语句最终一定被执行（无论是否抛出异常），经常被用在需要释放资源的情况下，`System.exit (0)` 可以阻断 finally 执行。

finalize 是在 `java.lang.Object` 里定义的方法，也就是说每一个对象都有这么个方法，这个方法在 `gc` 启动，该对象被回收的时候被调用。

一个对象的 finalize 方法只会被调用一次，finalize 被调用不一定会立即回收该对象，所以有可能调用 finalize 后，该对象又不需要被回收了，然后到了真正要被回收的时候，因为前面调用过一次，所以不会再次调用 finalize 了，进而产生问题，因此不推荐使用 finalize 方法。



## final在所有情况下都一定会被执行吗？

1. try语句执行之前，就出现异常或返回了
2. try-catch语句块包含System.exit(0)等语句，会直接终止JVM，也不会运行finally


## Final的实现原理，static final（编译时确定）

对于final域，编译器和处理器要遵守两个重排序规则：

1. 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

    **先写入final变量，后调用该对象引用**

    > 原因：编译器会在final域的写之后，插入一个`StoreStore`屏障，保证对final域的写已同步到主存，且对其他线程可见

2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。
    **先读对象的引用，后读final变量**

    > 编译器会在读final域操作的前面插入一个`LoadLoad`屏障，保证对包含final域的对象的引用的读取已完成

https://cloud.tencent.com/developer/article/1379380

## 为什么要用static关键字？ 

`static`用于标识静态的类、方法、对象。

当我们只想**为特定域分配单一存储空间，不考虑要创建多少对象或者说根本就不创建任何对象**，再就是我们**想在没有创建对象的情况下也想调用方法**。在这两种情况下，static关键字，满足了我们的需求。

##  ”static”关键字是什么意思？Java中是否可以覆盖(override)一个private或者是static的方法？

“static”关键字表明**一个成员变量或者是成员方法可以在没有所属的类的实例变量的情况下被访问。**

Java中`static`方法不能被覆盖，因为方法覆盖是基于**运行时动态绑定**的，而`static方法`是**编译时静态绑定**的。static方法跟类的任何实例都不相关，所以概念上不适用。

> static是静态方法，在类加载的解析阶段就已经被确定了方法调用的结果
> 而重载是动态分派的结果，二者不在同一阶段



##  是否可以在static环境中访问非static变量？

static变量在Java中是属于类的，它在所有的实例中的值是一样的。当类被Java虚拟机载入的时候，会对static变量进行初始化。如果你的代码尝试不用实例来访问非static的变量，编译器会报错，因为这些变量还没有被创建出来，还没有跟任何实例关联上。

##  static静态方法能不能引用非静态资源？

不能，new的时候才会产生的东西，对于初始化后就存在的静态资源来说，根本不认识它。

##  static静态方法里面能不能引用静态资源？

可以，因为都是类初始化的时候加载的，大家相互都认识。

##  非静态方法里面能不能引用静态资源？

可以，非静态方法就是实例方法，那是`new`之后才产生的，那么属于类的内容它都认识。

## java静态变量、代码块、和静态方法的执行顺序是什么？

基本上代码块分为三种：Static静态代码块、构造代码块、普通代码块

代码块执行顺序**静态代码块——> 构造代码块 ——> 构造函数——> 普通代码块** 

继承中代码块执行顺序：**父类静态块——>子类静态块——>父类代码块——>父类构造器——>子类代码块——>子类构造器**


想要深入了解，可以参考这篇文章 ：https://juejin.cn/post/6844903986475040781

# 面向对象

## 面向对象和面向过程的区别？

**面向过程**：

* 优点：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗资源;比如单片机、嵌入式开发、Linux/Unix等一般采用面向过程开发，性能是最重要的因素。

* 缺点：没有面向对象易维护、易复用、易扩展。

**面向对象**：

* 优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统更加灵活、更加易于维护。

* 缺点：性能比面向过程低。

## 讲讲面向对象三大特性

- 封装。封装最好理解了。封装是面向对象的特征之一，是对象和类概念的主要特性。封装，也就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。
- 继承。继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。通过继承创建的新类称为“子类”或“派生类”，被继承的类称为“基类”、“父类”或“超类”。
- 多态性。它是指在父类中定义的属性和方法被子类继承之后，可以具有不同的数据类型或表现出不同的行为，这使得同一个属性或方法在父类及其各个子类中具有不同的含义。



## Java语言是如何实现多态的？

本质上多态分两种：

> **1、编译时多态（又称静态多态）**
>
> **2、运行时多态（又称动态多态）**

重载（`overload`）就是编译时多态的一个例子，编译时多态在编译时就已经确定，运行的时候调用的是确定的方法。

> 方法调用分为解析与分派，其中，解析在类加载阶段，而重写与重载属于分派，在编译阶段

**我们通常所说的多态指的都是运行时多态，也就是编译时不确定究竟调用哪个具体方法，一直延迟到运行时才能确定** 这也是为什么有时候多态方法又被称为延迟方法的原因。

Java实现多态有 3 个必要条件：**继承、重写和向上转型**。
只有满足这 3 个条件，开发人员才能够在同一个继承结构中使用统一的逻辑实现代码处理不同的对象，从而执行不同的行为。

- 继承：在多态中必须存在有继承关系的子类和父类。
- 重写Override：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
- 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才既能可以调用父类的方法，又能调用子类的方法。

Java多态的实现原理可看这篇文章：https://my.oschina.net/u/4432600/blog/4535042

## 重载（Overload）和重写（Override）的区别是什么？

方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。

* 重写`override`发生在子类与父类之间，**外壳不变，核心重写**：
  1. 重写方法的参数列表都不能改变
  2. 返回类型可以不同但必须值父类返回值的派生类
  3. 访问权限不能比父类更低(父类是public，子类就不能是protected)。
  4. 声明static、final、private的不能重写
* 重载`overload` 针对的是同一个类中的方法、构造器：
  1. **方法名字相同，而参数个数或类型不同**,每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表
  2. 返回类型可以相同也可以不同。
最常用的地方就是构造器的重载。

![image-20210219181506507](http://blog-img.coolsen.cn/img/image-20210219181506507.png)

## 重载的方法能否根据返回值类型进行区分？

不能根据返回值类型来区分重载的方法。因为动态分派时不指定类型信息，编译器不知道你要调用哪个函数。
而且修饰符、异常检查也可以被修改

```java
float max(int a, int b);
int max(int a, int b);
```

当调用`max（1,2）;`时无法确定调用的是哪个，单从这一点上来说，仅返回值类型不同的重载是不应该允许的。

## 构造器（constructor）是否可被重写（override）？

构造器**不能被继承，因此不能被重写，但可以被重载**。
每一个类必须有自己的构造函数，负责构造自己这部分的构造。
子类不会覆盖父类的构造函数，相反必须一开始调用父类的构造函数。

## 枚举类的实现原理



## 抽象类和接口的区别是什么？

语法层面上的区别：

* 抽象类可以提供成员方法的实现细节，而接口中只能存在`public abstract` 方法；
  > 当然，JDK8可以在接口中设置默认方法`default`，默认方法必须提供实现 
* 抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是`public static final`类型的；
  > 就算没有标注，在编译时，也会自动加上
* 接口中不能含有**静态代码块以及静态方法**，而抽象类可以有静态代码块和静态方法；
* 一个类只能继承一个抽象类，而一个类却可以实现多个接口。

设计层面上的区别：

* 抽象类是对一种事物的抽象，即对类抽象，而接口是对行为的抽象。抽象类是对整个类整体进行抽象，包括属性、行为，但是接口却是对类局部（行为）进行抽象。
* 设计层面不同，抽象类作为很多子类的父类，它是一种模板式设计。而接口是一种行为规范，它是一种辐射式设计。

想要深入了解，可以参考这篇文章 ：https://www.cnblogs.com/dolphin0520/p/3811437.html

## 抽象类能使用 final 修饰吗？

不能，定义抽象类就是让其他类继承的，如果定义为 final 该类就不能被继承，这样彼此就会产生矛盾，所以 final 不能修饰抽象类

## Object有哪些主要方法，都有什么用途

1. clone方法

保护方法，实现对象的**浅复制**，只有实现了`Cloneable`接口才可以调用该方法，否则抛出CloneNotSupportedException异常。

主要是JAVA里除了8种基本类型传参数是值传递，其他的类对象传参数都是引用传递，我们有时候不希望在方法里讲参数改变，这是就需要在类中复写clone方法。

2. getClass方法

final方法，获得运行时类型。

3. toString方法

该方法用得比较多，一般子类都有覆盖。

4. finalize方法

该方法用于释放资源。因为无法确定该方法什么时候被调用，很少使用。

5. equals方法

该方法是非常重要的一个方法。一般equals和==是不一样的，但是在Object中两者是一样的。子类一般都要重写这个方法。

6. hashCode方法

该方法用于哈希查找，可以减少在查找中使用equals的次数，重写了equals方法一般都要重写hashCode方法。这个方法在一些具有哈希功能的Collection中用到。

一般必须满足obj1.equals(obj2)==true。可以推出obj1.hash- Code()==obj2.hashCode()，但是hashCode相等不一定就满足equals。不过为了提高效率，应该尽量使上面两个条件接近等价。

如果不重写hashcode(),在HashSet中添加两个equals的对象，会将两个对象都加入进去。

7. wait方法

wait方法就是使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。wait()方法一直等待，直到获得锁或者被中断。wait(long timeout)设定一个超时间隔，如果在规定时间内没有获得锁就返回。

调用该方法后当前线程进入睡眠状态，直到以下事件发生。

（1）其他线程调用了该对象的notify方法。

（2）其他线程调用了该对象的notifyAll方法。

（3）其他线程调用了interrupt中断该线程。

（4）时间间隔到了。

此时该线程就可以被调度了，如果是被中断的话就抛出一个InterruptedException异常。

8. notify方法

该方法唤醒在该对象上等待的某个线程。

9. notifyAll方法

该方法唤醒在该对象上等待的所有线程。

## 有一个Object对象数组，这个数组所用的内存是连续的吗？

数组的分配空间都是连续的，虚拟机开创好空间后会返回一串引用地址.
但由于虚拟内存技术，在物理上可能是不连续的，只是程序从逻辑上认为自己使用的是连续的内存

## java 创建对象有哪几种方式？

java中提供了以下四种创建对象的方式:

- new创建新对象
- 通过反射机制
- 采用clone机制
- 通过序列化机制

前两者都需要显式地调用构造方法。对于clone机制,需要注意浅拷贝和深拷贝的区别，对于序列化机制需要明确其实现原理，在java中序列化可以通过实现`Externalizable`或者`Serializable`来实现。

## 什么是不可变对象?好处是什么?

不可变对象指**对象一旦被创建,状态就不能再改变,任何修改都会创建一个新的对象**,如 String、Integer及其它包装类.不可变对象最大的好处是线程安全.

## 能否创建一个包含可变对象的不可变对象?

当然可以,比如`final Person[] persons = new Persion[]{}`. `persons`是不可变对象的引用,但**其数组中的Person实例却是可变的**.
这种情况下需要特别谨慎，不要共享可变对象的引用。这种情况下，如果数据需要变化时，就返回原对象的一个拷贝。

## 值传递和引用传递的区别的什么？为什么说Java中只有值传递？

值传递：指的是在方法调用时，传递的参数是按值的拷贝传递，传递的是值的拷贝，也就是说传递后就互不相关了。

引用传递：指的是在方法调用时，传递的参数是按引用进行传递，其实传递的是引用的地址，也就是变量所对应的内存空间的地址。传递的是值的引用，也就是说传递前和传递后都指向同一个引用（也就是同一个内存空间）。

基本类型作为参数被传递时肯定是值传递；引用类型作为参数被传递时也是值传递，只不过“值”为对应的引用。

想要深入了解，可以参考这篇文章 ：http://www.itwanger.com/java/2019/11/26/java-yinyong-value.html

# 对象相等判断

## == 和 equals 区别是什么？

`==`常用于相同的基本数据类型之间的比较，也可用于相同类型的对象之间的比较；

- 如果`==`比较的是基本数据类型，那么比较的是两个基本数据类型的值是否相等；
- 如果`==`是比较的两个对象，那么比较的是两个对象的引用，也就是**判断两个对象是否指向了同一块内存区域**；

equals方法主要用于两个对象之间，检测一个对象是否等于另一个对象

看一看Object类中equals方法的源码：

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

它的作用也是**判断两个对象是否相等**，般有两种使用情况：

* 情况1，类没有覆盖`equals()`方法。则通过`equals()`比较该类的两个对象时，等价于通过`==`比较这两个对象。
* 情况2，类覆盖了`equals()`方法。一般，我们都覆盖`equals()`方法来两个对象的内容相等；若它们的内容相等，则返回true(即，认为这两个对象相等)。

java语言规范要求equals方法具有以下特性：

- 自反性。对于任意不为null的引用值x，`x.equals(x)`一定是true。
- 对称性。对于任意不为null的引用值x和y，当且仅当`x.equals(y)`是true时，`y.equals(x)`也是true。
- 传递性。对于任意不为null的引用值x、y和z，如果`x.equals(y)是true`，同时`y.equals(z)`是true，那么`x.equals(z)`一定是true。
- 一致性。对于任意不为null的引用值x和y，如果用于equals比较的对象信息没有被修改的话，多次调用时x.equals(y)要么一致地返回true要么一致地返回false。
- 对于任意不为null的引用值x，x.equals(null)返回false。


不过有一个特殊情况，那就是 `Integer`，我们会发现，有时候比较的两个 `Integer` 对象具有相同的值时候，使用 `==` 的结果是 `true` ，但实际上，`==` 比较的是对象的引用，应该返回 `false` 才对

这是因为 `Integer` 内部维护着一个 `IntegerCache` 的缓存，默认缓存范围是 `[-128, 127]`，所以 `[-128, 127]` 之间的值用 `==` 比较也能能到正确的结果，但是不推荐用关系运算符比较

而 `equals` 呢，适用于对象之间的比较，而且比较的也是内存地址。
但是！大部分类都会重写 `equals` 方法，使其变成**基于值**的比较，比如 `Integer`、`String` 等等   


## **介绍下hashCode()？**

`hashCode()` 的作用是获取哈希码，也称为散列码；它实际上是返回一个根据对象在内存中的地址计算得到的`int整数`。这个哈希码的作用是**确定该对象在哈希表中的索引位置**。

> 两个不同对象可能hashcode相等，但两个不同hashcode指向的对象一定不同，因此，不能通过hashCode判断对象是否相等
> > 在HashMap的put与get方法中，在hash值相等的基础上，还要通过key.equals()方法判断是否相等

## **为什么要有 hashCode?**

**以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode**：

减小比较对象是否相等的开销

当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。

但是如果发现有相同 hashcode 值的对象，这时会调用 `equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

## 如果一个对象有两个成员变量 但是只重写了一个hashcode，放到 hashmap 中去会有什么后果

当对象作为key时，可能导致错误的覆盖，即明明是不同的对象但由于计算出的hashcode相同，导致覆盖


## hashCode(),equals()两种方法是什么关系?
**hashCode()与equals()的相关规定**：

* 如果两个对象相等，则hashcode一定也是相同的；
* 两个对象相等，对两个对象分别调用equals方法都返回true；
* 两个对象有相同的hashcode值，它们也不一定是相等的；

## 为什么重写 equals 方法必须重写 hashcode 方法 ？

判断对象是否相等时，先根据`hashcode`进行判断，相同的情况下再根据`equals()`方法进行判断。

如果只重写了`equals`方法，而不重写hashcode的方法，会造成**equals()方法判断出来的结果为true，而hashcode的值不同**，违背了对象相等判断的基本规则

> 在Java中的一些容器中，不允许有两个完全相同的对象，插入的时候，如果判断相同则会进行覆盖。
> 这时候如果只重写了`equals()`的方法，而不重写`hashcode`的方法，Object中`hashcode是`根据对象的存储地址转换而形成的一个哈希值。
> 这时候就有可能因为没有重写`hashcode`方法，造成**相同的对象散列到不同的位置，而不是覆盖**的问题

## String,StringBuffer, StringBuilder 的区别是什么？

1.可变与不可变。String类中使用字符数组保存字符串，因为有“final”修饰符，所以string对象是不可变的。**对于已经存在的String对象的修改都是重新创建一个新的对象,然后把新的值保存进去。**

String类利用了final修饰的char类型数组存储字符，源码如下:

`private final char value[];`

`StringBuilder`与`StringBuffer`都继承自`AbstractStringBuilder`类，在AbstractStringBuilder中也是使用**字符数组保存字符串**，**这两种对象都是可变的**。

源码如下:

`char[] value;`

2.是否多线程安全。

String中的对象是不可变的，也就可以理解为常量，显然线程安全。

StringBuilder是非线程安全的。

`StringBuffer`对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

源码如下:

```java
    @Override
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
```

3.性能

每次对String 类型进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String 对象。StringBuffer每次都会对StringBuffer对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用StirngBuilder 相比使用StringBuffer 仅能获得10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

## String为什么要设计成不可变的？

1.便于实现字符串池（String pool）

在Java中，由于会大量的使用String常量，如果每一次声明一个String都创建一个String对象，那将会造成极大的空间资源的浪费。
Java提出了`String pool（全局字符创常量池）`的概念，在`堆`中开辟一块存储空间`String pool`，当初始化一个String变量时，**如果该字符串已经存在了，就不会去创建一个新的字符串变量，而是会返回已经存在了的字符串的引用**。

```
String a = "Hello world!";
String b = "Hello world!";
```

如果字符串是可变的，某一个字符串变量改变了其值，那么其指向的变量的值也会改变，String pool将不能够实现！

2.使多线程安全

在并发场景下，多个线程同时读一个资源，是安全的，不会引发竞争，但对资源进行写操作时是不安全的，**不可变对象不能被写，所以保证了多线程的安全**。

3.避免安全问题

在网络连接和数据库连接中字符串常常作为参数，例如，网络连接地址URL，文件路径path，反射机制所需要的String参数。其不可变性可以保证连接的安全性。如果字符串是可变的，黑客就有可能改变字符串指向对象的值，那么会引起很严重的安全问题。

4.加快字符串处理速度

由于String是不可变的，保证了hashcode的唯一性，于是在创建对象时其hashcode就可以放心的缓存了，不需要重新计算。这也就是Map喜欢将String作为Key的原因，处理速度要快过其它的键对象。所以HashMap中的键往往都使用String。

总体来说，String不可变的原因要包括 设计考虑，效率优化，以及安全性这三大方面。



# 包装类型

## 包装类型是什么？基本类型和包装类型有什么区别？

Java 为每一个基本数据类型都引入了对应的包装类型（wrapper class），int 的包装类就是 Integer，从 Java 5 开始引入了**自动装箱/拆箱机制**，把基本类型转换成包装类型的过程叫做装箱（boxing）；反之，把包装类型转换成基本类型的过程叫做拆箱（unboxing），使得二者可以相互转换。

Java 为每个原始类型提供了包装类型：

原始类型: boolean，char，byte，short，int，long，float，double

包装类型：Boolean，Character，Byte，Short，Integer，Long，Float，Double

**基本类型和包装类型的区别主要有以下 几点**：

* **包装类型可以为 null，而基本类型不可以**。它使得包装类型可以应用于 POJO 中，而基本类型则不行。
     > 那为什么 POJO 的属性必须要用包装类型呢？
     《阿里巴巴 Java 开发手册》上有详细的说明， **数据库的查询结果可能是 null**，如果使用**基本类型**的话，因为要自动拆箱（将包装类型转为基本类型，比如说把 Integer 对象转换成 int 值），就会抛出 `NullPointerException` 的异常。

* **包装类型可用于泛型，而基本类型不可以**。泛型不能使用基本类型，因为使用基本类型时会编译出错。

  ```java
  List<int> list = new ArrayList<>(); // 提示 Syntax error, insert "Dimensions" to complete ReferenceType
  List<Integer> list = new ArrayList<>();
  ```

  因为泛型在编译时会进行类型擦除，最后只保留原始类型，而原始类型只能是 `Object 类及其子类`，但基本类型并不属于其子类。

* **基本类型比包装类型更高效**。基本类型在栈中直接存储的具体数值，而包装类型则存储的是堆中的引用。 很显然，相比较于基本类型而言，包装类型需要占用更多的内存空间。

## 解释一下自动装箱和自动拆箱？

**自动装箱：将基本数据类型重新转化为对象**

```java
    public class Test {  
        public static void main(String[] args) {  
            // 声明一个Integer对象，用到了自动的装箱：解析为:Integer num = Integer.valueOf(9);
	        Integer num = 9;
        }  
    }  
```

`9`是属于基本数据类型的，原则上它是不能直接赋值给一个对象Integer的。但jdk1.5 开始引入了自动装箱/拆箱机制，就可以进行这样的声明，自动将基本数据类型转化为对应的封装类型，成为一个对象以后就可以调用对象所声明的所有的方法。

**自动拆箱：将对象重新转化为基本数据类型**

```java
 public class Test {  
        public static void main(String[] args) {  
            / /声明一个Integer对象
	        Integer num = 9;
            
            // 进行计算时隐含的有自动拆箱
		    System.out.print(num--);
        }  
    }  
```

因为**对象时不能直接进行运算的，而是要转化为基本数据类型后才能进行加减乘除**。

## int 和 Integer 有什么区别?

- Integer是int的包装类；int是基本数据类型；
- Integer变量必须实例化后才能使用；int变量不需要；
- Integer实际是对象的引用，指向此new的Integer对象；int是直接存储数据值 ；
- Integer的默认值是`null`；int的默认值是`0`。

## 两个new生成的Integer变量的对比

由于Integer变量实际上是对一个Integer对象的引用，所以**两个通过new生成的Integer变量永远是不相等的**（因为new生成的是两个对象，其内存地址不同）。

```java
Integer i = new Integer(10);
Integer j = new Integer(10);
System.out.print(i == j); //false
```

## Integer变量和int变量的对比

Integer变量和int变量比较时，**只要两个变量的值是向等的，则结果为true**
>（因为包装类Integer和基本数据类型int比较时，java会自动拆包装为int，然后进行比较，实际上就变为两个int变量的比较）

```java
    int a = 10000;
    Integer b = new Integer(10000);
    Integer c=10000;
    System.out.println(a == b); // true
    System.out.println(a == c); // true
```

## 非new生成的Integer变量和new Integer()生成变量的对比

**非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。**
>因为非new生成的`Integer`变量指向的是**java常量池中的对象**
而`new Integer()`生成的变量指向**堆中新建的对象**，两者在内存中的地址不同）

```java
    Integer b = new Integer(10000);
    Integer c=10000;
    System.out.println(b == c); // false
```

## 两个非new生成的Integer对象的对比

**对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true**，如果两个变量的值不在此区间，则比较结果为false

```java
Integer i = 100;
Integer j = 100;
System.out.print(i == j); //true

Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```

> 当值在 `-128 ~ 127`之间时，java会进行自动装箱，然后会对值进行缓存，如果下次再有相同的值，会直接在缓存中取出使用。缓存是通过Integer的内部类`IntegerCache`来完成的。当值超出此范围，会在堆中new出一个对象来存储。

给一个`Integer`对象赋一个`int值`的时候，会调用`Integer`类的静态方法`valueOf`，并根据数值范围，选择返回常量池中的引用，还是新建一个对象。

> 静态常量池中cache数组是`static final`类型，cache数组对象会被存储于静态常量池中。
>  * cache数组里面的元素却不是static final类型，而是cache[k] = new Integer(j++)，
>  * 那么这些元素是存储于堆中，只是cache数组对象存储的是指向了堆中的Integer对象（引用地址）


# 反射

## 什么是反射？

反射是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种**动态获取的信息以及动态调用对象的方法**的功能称为 Java 语言的反射机制。

## 反射机制的优缺点有哪些？

优点：能够运行时动态获取类的实例，提高灵活性；可与动态编译结合`Class.forName('com.mysql.jdbc.Driver.class');`，加载MySQL的驱动类。

缺点：使用反射性能较低，需要解析字节码，将内存中的对象进行解析。其解决方案是：
  1. 通过`setAccessible(true)`关闭JDK的安全检查来提升反射速度；
  2. 多次创建一个类的实例时，有缓存会快很多；
  3. ReflflectASM工具类，通过字节码生成的方式加快反射速度。

## 如何获取反射中的Class对象？

1. Class.forName(“类的路径”)；当你知道该类的全路径名时，你可以使用该方法获取 Class 类对象。

   ```java
   Class clz = Class.forName("java.lang.String");
   ```

2. 类名.class。这种方法只适合在编译前就知道操作的 Class。

   ```java
   Class clz = String.class;
   ```

3. 对象名.getClass()。

   ```java
   String str = new String("Hello");
   Class clz = str.getClass();
   ```

4. 如果是基本类型的包装类，可以调用包装类的Type属性来获得该包装类的Class对象。

## Java反射API有几类？

反射 API 用来生成 JVM 中的类、接口或则对象的信息。

* `Class` 类：反射的核心类，可以获取类的属性，方法等信息。

* `Field` 类：Java.lang.reflec 包中的类，表示类的成员变量，可以用来获取和设置类之中的属性值。

* `Method` 类：Java.lang.reflec 包中的类，表示类的方法，它可以用来获取类中的方法信息或者执行方法。

* `Constructor` 类：Java.lang.reflec 包中的类，表示类的构造方法。

## 反射使用的步骤？

1. 获取想要操作的类的Class对象，这是反射的核心，通过Class对象我们可以任意调用类的方法。

2. 调用 Class 类中的方法，既就是反射的使用阶段。

3. 使用反射 API 来操作这些信息。

具体可以看下面的例子：

```java
public class Apple {

    private int price;

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public static void main(String[] args) throws Exception{
        //正常的调用
        Apple apple = new Apple();
        apple.setPrice(5);
        System.out.println("Apple Price:" + apple.getPrice());
        //使用反射调用

        //1. 获取类的 Class 对象实例
        Class clz = Class.forName("com.chenshuyi.api.Apple");
        //2.1. 获得方法的Method对象
        Method setPriceMethod = clz.getMethod("setPrice", int.class);
        //2.2. 根据 Class 对象实例获取 Constructor 对象
        Constructor appleConstructor = clz.getConstructor();
        //3.1. 使用 Constructor 对象的 newInstance 方法获取反射类对象
        Object appleObj = appleConstructor.newInstance();
        //3.2. 通过invoke调用方法
        setPriceMethod.invoke(appleObj, 14);
        Method getPriceMethod = clz.getMethod("getPrice");
        System.out.println("Apple Price:" + getPriceMethod.invoke(appleObj));
    }
}
```


## 为什么引入反射概念？反射机制的应用有哪些？

我们来看一下 Oracle 官方文档中对反射的描述：

从 Oracle 官方文档中可以看出，反射主要应用在以下几方面：

- 反射让开发人员可以**通过外部类的全路径名创建对象**，并使用这些类，实现一些扩展的功能。
- 反射让开发人员可以**枚举出类的全部成员**，包括构造函数、属性、方法。以帮助开发者写出正确的代码。
- 测试时可以**利用反射 API 访问类的私有成员**，以保证测试代码覆盖率。

也就是说，Oracle 希望开发者将反射作为一个工具，用来帮助程序员实现本不可能实现的功能。

举两个最常见使用反射的例子，来说明反射机制的强大之处：

第一种：**JDBC 的数据库的连接**

在JDBC 的操作中，如果要想进行数据库的连接，则必须按照以上的几步完成

1. 通过Class.forName()加载数据库的驱动程序 （通过反射加载，前提是引入相关了Jar包）；
2. 通过 DriverManager 类进行数据库的连接，连接的时候要输入数据库的连接地址、用户名、密码；
3. 通过Connection 接口接收连接。

```java
public class ConnectionJDBC {  
  
    /** 
     * @param args 
     */  
    //驱动程序就是之前在classpath中配置的JDBC的驱动程序的JAR 包中  
    public static final String DBDRIVER = "com.mysql.jdbc.Driver";  
    //连接地址是由各个数据库生产商单独提供的，所以需要单独记住  
    public static final String DBURL = "jdbc:mysql://localhost:3306/test";  
    //连接数据库的用户名  
    public static final String DBUSER = "root";  
    //连接数据库的密码  
    public static final String DBPASS = "";  
      
      
    public static void main(String[] args) throws Exception {  
        Connection con = null; //表示数据库的连接对象  
        Class.forName(DBDRIVER); //1、使用CLASS 类加载驱动程序 ,反射机制的体现 
        con = DriverManager.getConnection(DBURL,DBUSER,DBPASS); //2、连接数据库  
        System.out.println(con);  
        con.close(); // 3、关闭数据库  
    }  

```

第二种：**Spring 框架的使用，最经典的就是xml的配置模式**。

Spring 通过 XML 配置模式装载 Bean 的过程：

1. 将程序内所有 `XML` 或 `Properties` 配置文件加载入内存中；
2. Java类里面解析`xml`或`properties`里面的内容，得到对应实体类的字节码字符串以及相关的属性信息；
3. 使用反射机制，根据这个字符串获得某个类的Class实例；
4. 动态配置实例的属性。

Spring这样做的好处是：

- 不用每一次都要在代码里面去new或者做其他的事情；
- 以后要改的话直接改配置文件，代码维护起来就很方便了；
- 有时为了适应某些需求，Java类里面不一定能直接调用另外的方法，可以通过反射机制来实现。

模拟 Spring 加载 XML 配置文件：

```java
public class BeanFactory {
       private Map<String, Object> beanMap = new HashMap<String, Object>();
       /**
       * bean工厂的初始化.
       * @param xml xml配置文件
       */
       public void init(String xml) {
              try {
                     //读取指定的配置文件
                     SAXReader reader = new SAXReader();
                     ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
                     //从class目录下获取指定的xml文件
                     InputStream ins = classLoader.getResourceAsStream(xml);
                     Document doc = reader.read(ins);
                     Element root = doc.getRootElement();  
                     Element foo;
                    
                     //遍历bean
                     for (Iterator i = root.elementIterator("bean"); i.hasNext();) {  
                            foo = (Element) i.next();
                            //获取bean的属性id和class
                            Attribute id = foo.attribute("id");  
                            Attribute cls = foo.attribute("class");
                           
                            //利用Java反射机制，通过class的名称获取Class对象
                            Class bean = Class.forName(cls.getText());
                           
                            //获取对应class的信息
                            java.beans.BeanInfo info = java.beans.Introspector.getBeanInfo(bean);
                            //获取其属性描述
                            java.beans.PropertyDescriptor pd[] = info.getPropertyDescriptors();
                            //设置值的方法
                            Method mSet = null;
                            //创建一个对象
                            Object obj = bean.newInstance();
                           
                            //遍历该bean的property属性
                            for (Iterator ite = foo.elementIterator("property"); ite.hasNext();) {  
                                   Element foo2 = (Element) ite.next();
                                   //获取该property的name属性
                                   Attribute name = foo2.attribute("name");
                                   String value = null;
                                  
                                   //获取该property的子元素value的值
                                   for(Iterator ite1 = foo2.elementIterator("value"); ite1.hasNext();) {
                                          Element node = (Element) ite1.next();
                                          value = node.getText();
                                          break;
                                   }
                                  
                                   for (int k = 0; k < pd.length; k++) {
                                          if (pd[k].getName().equalsIgnoreCase(name.getText())) {
                                                 mSet = pd[k].getWriteMethod();
                                                 //利用Java的反射极致调用对象的某个set方法，并将值设置进去
                                                 mSet.invoke(obj, value);
                                          }
                                   }
                            }
                           
                            //将对象放入beanMap中，其中key为id值，value为对象
                            beanMap.put(id.getText(), obj);
                     }
              } catch (Exception e) {
                     System.out.println(e.toString());
              }
       }
      
       //other codes
}
```


## 如何利用反射创建一个对象

1. 获得类的Class对象
2. 通过Class对象，获得类的构造器
3. 通过构造器的`newInstance`方法创建对象

## 反射机制的原理是什么？

```java
Class actionClass=Class.forName(“MyClass”);
Object action=actionClass.newInstance();
Method method = actionClass.getMethod(“myMethod”,null);
method.invoke(action,null);
```

https://flying-veal.notion.site/13893931261e4dcfa620988756a43803

上面就是最常见的反射使用的例子，前两行实现了**类的装载、链接和初始化****（newInstance方法实际上也是使用反射调用了<init>方法）**，后两行实现了从class对象中获取到method对象然后执行反射调用。

因反射原理较复杂，下面简要描述下流程，想要详细了解的小伙伴，可以看这篇文章：https://www.cnblogs.com/yougewe/p/10125073.html

1. 反射获取类实例 `Class.forName()`，并没有将实现留给了java,而是交给了jvm去加载！主要是先获取 `ClassLoader`, 然后调用` native 方法`，获取信息，加载类则是回调 `java.lang.ClassLoader`。最后，jvm又会回调 ClassLoader 进类加载！
2.  `newInstance()` 主要做了三件事：
   * 权限检测，如果不通过直接抛出异常；
   * 查找无参构造器，并将其缓存起来；
   *  调用具体方法的无参构造方法，生成实例并返回。

3. 获取Method对象，

   ![](http://blog-img.coolsen.cn/img/image-20210226195426092.png)

上面的Class对象是在加载类时由JVM构造的，JVM为每个类管理一个独一无二的Class对象，这份Class对象里维护着该类的所有Method，Field，Constructor的cache，这份cache也可以被称作根对象。

每次getMethod获取到的Method对象都持有对根对象的引用，因为一些重量级的Method的成员变量（主要是MethodAccessor），我们不希望每次创建Method对象都要重新初始化，于是所有代表同一个方法的Method对象都共享着根对象的MethodAccessor，每一次创建都会调用根对象的copy方法复制一份：

```java
 Method copy() { 

        Method res = new Method(clazz, name, parameterTypes, returnType,
            exceptionTypes, modifiers, slot, signature,
            annotations, parameterAnnotations, annotationDefault);

        res.root = this;

        res.methodAccessor = methodAccessor;

        return res;

    }
```

4. 调用invoke()方法。调用invoke方法的流程如下：

   ![](http://blog-img.coolsen.cn/img/image-20210226195531619.png)



调用Method.invoke之后，会直接去调MethodAccessor.invoke。MethodAccessor就是上面提到的所有同名method共享的一个实例，由ReflectionFactory创建。

创建机制采用了一种名为inflation的方式（JDK1.4之后）：如果该方法的累计调用次数<=15，会创建出NativeMethodAccessorImpl，它的实现就是直接调用native方法实现反射；如果该方法的累计调用次数>15，会由java代码创建出字节码组装而成的MethodAccessorImpl。（是否采用inflation和15这个数字都可以在jvm参数中调整）
以调用MyClass.myMethod(String s)为例，生成出的MethodAccessorImpl字节码翻译成Java代码大致如下：

```java
public class GeneratedMethodAccessor1 extends MethodAccessorImpl {    
    public Object invoke(Object obj, Object[] args)  throws Exception {
        try {
            MyClass target = (MyClass) obj;
            String arg0 = (String) args[0];
            target.myMethod(arg0);
        } catch (Throwable t) {
            throw new InvocationTargetException(t);
        }
    }
}
```

## 反射能获得private方法吗？怎么获得？

可以获得，但不能用`getMethod`，而是`getDeclaredMethod`
- 使用`getDeclaredMethod`，需要在`invoke`调用方法前，使用`method.setAccessible(true)`，来设置或取消访问权限检查


# 泛型

## Java中的泛型是什么 ? 

泛型是 JDK1.5 的一个新特性，**泛型就是将类型参数化，其在编译时才确定具体的参数**。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

## 使用泛型的好处是什么?

远在 JDK 1.4 版本的时候，那时候是没有泛型的概念的，如果使用 Object 来实现通用、不同类型的处理，有这么两个缺点：

1. 每次使用时都需要强制转换成想要的类型
2. 在编译时编译器并不知道类型转换是否正常，运行时才知道，不安全。

如这个例子：

```java
List list = new ArrayList();
list.add("www.cnblogs.com");
list.add(23);
String name = (String)list.get(0);
String number = (String)list.get(1);	//ClassCastException
```

上面的代码在运行时会发生**强制类型转换异常**。这是因为我们在存入的时候，第二个是一个 Integer 类型，但是取出来的时候却将其强制转换为 `String` 类型了。

Sun 公司为了使 Java 语言更加安全，减少运行时异常的发生。于是在 JDK 1.5 之后推出了泛型的概念。
根据《Java 编程思想》中的描述，泛型出现的动机在于：有许多原因促成了泛型的出现，而最引人注意的一个原因，就是**为了创建容器类**。

**使用泛型的好处有以下几点**：

1. 类型安全 
   * 泛型的主要目标是提高 Java 程序的类型安全
   * 编译时期就可以检查出因 Java 类型不正确导致的 `ClassCastException` 异常
   * 符合越早出错代价越小原则

2. 消除强制类型转换 
   * 泛型的一个附带好处是，使用时直接得到目标类型，消除许多强制类型转换
   * 所得即所需，这使得代码更加可读，并且减少了出错机会

3. 潜在的性能收益 
   * 由于泛型的实现方式，支持泛型（几乎）不需要 JVM 或类文件更改
   * 所有工作都在编译器中完成
   * 编译器生成的代码跟不使用泛型（和强制类型转换）时所写的代码几乎一致，只是更能确保类型安全而已

泛型的劣势：
1. 泛型集合无法像数组一样，实现多态：
```java
Object[] arr = new String[2];//可以通过编译
ArrayList<Obejct> list = new ArrayList<String>();//不能通过编译
```

2. 形式参数上的弊端：
如果方法的参数为`ArrayList<Integer>`，则传入的参数必须为`ArrayList<Integer>`，不能是`ArrayList<Object>`


## Java泛型的原理是什么 ? 什么是类型擦除 ?

https://www.cnblogs.com/robothy/p/13949788.html

> Java中的泛型基本上都是在编译器这个层次来实现的，也就是说： **泛型只存在于编译阶段，而不存在于运行阶段**。声明了泛型的 .java 源代码，在编译生成 .class 文件之后，泛型相关的信息就消失了。

泛型是一种语法糖，泛型的本质是将类型参数化，在代码逻辑不关注具体的数据类型时使用。从而可以实现具有通用性的算法，而不需要关注具体的数据类型。

泛型这种语法糖的基本原理是**类型擦除**。
类型擦除：**使用泛型的时候加上的类型参数，编译器在编译的时候去掉类型参数**。

Java 编译器通过如下方式实现擦除：

- 用 `Object` 或者`界定类型`替代泛型，产生的字节码中**只包含了原始的类，接口和方法**；
- 在恰当的位置插入**强制转换代码**来确保**类型安全**；
- 在继承了泛型类或接口的类中插入**桥接方法**来保留多态性。

大部分情况下，泛型类型都会以 `Object` 进行替换，而有一种情况则不是。那就是使用到了`extends`和`super`语法的有界类型，如：

```java
public class Caculate<T extends String> {
    private T num;
}
```

> 这种情况的泛型类型，`num` 会被替换为 `String` 而不再是 `Object`。
> 这是一个类型限定的语法，它限定 `T` 是 `String` 或者 `String 的子类`，也就是你构建 Caculate 实例的时候**只能限定 T 为 String 或者 String 的子类**。
> 所以**无论你限定 T 为什么类型，String 都是父类，不会出现类型不匹配的问题**，于是可以使用 String 进行类型擦除。

实际上编译器会正常的将使用泛型的地方编译并进行类型擦除，然后返回实例。
但是除此之外的是，如果构建泛型实例时使用了泛型语法，那么编译器将标记该实例并关注**该实例后续所有方法的调用，每次调用前都进行安全检查**，非指定类型的方法都不能调用成功。

实际上编译器不仅关注一个泛型方法的调用，它还会**为某些返回值为限定的泛型类型的方法进行强制类型转换，由于类型擦除，返回值为泛型类型的方法都会擦除成 Object 类型**，当这些方法被调用后，编译器会额外插入一行 checkcast 指令用于强制类型转换。这一个过程就叫做『泛型翻译』。

## 桥接方法是啥

```java
class User implements Comparable<User> {
    String name;
	
    public int compareTo(User other){
        return this.name.compareTo(other.name);
    }
}
```

对一个实现了`Comparable<User>`类，重写了`compareTo(User)`方法的User类，在编译后，`Comparable<User>`会被擦除为`Comparable`，且会被插入桥接方法:
```java
public int compareTo(User user)
{
    return name.compareTo(user.name);
}

public volatile int compareTo(Object obj)
{
    return compareTo((User)obj);
}
```

可见：多了一个`compareTo(Object)`方法，参数 `obj` 被强转成 `User` 再传入 `compareTo(User user)` 方法，体现了泛型擦除规则中的强制类型转换规则（保证类型安全）与桥接方法规则（保留多态性）

Java 中的多态特性允许我们使用一个父类或者接口的引用指向一个子类对象（即静态类型可以是实际类型的父类）`Comparable<User> user = new User();`，如果没有桥接方法，该`user`的`compareTo`方法就只能传入`Comparable<User>`类，无法传入`User类`。

## 什么是泛型中的限定通配符和非限定通配符 ?

限定通配符对类型进行了限制。有两种限定通配符：
1. `<? extends T>`，它通过**确保类型必须是T的子类**来设定类型的上界，
2. `<? super T>`，它通过**确保类型必须是T的父类**来设定类型的下界。

泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。

非限定通配符 **？**,可以用任意类型来替代。如`List<?>` 的意思是这个集合是一个可以持有任意类型的集合，它可以是`List<A>`，也可以是`List<B>`,或者`List<C>`等等。

## List<? extends T>和List <? super T>之间有什么区别 ?

这两个List的声明都是限定通配符的例子，`List<? extends T>`可以接受任何**继承自T的类型的List**，而`List<? super T>`可以接受任何**T的父类构成的List**。

## 可以把List`<String>`传递给一个接受List`<Object>`参数的方法吗？

不可以。真这样做的话会导致编译错误。因为List<Object>可以存储任何类型的对象包括String, Integer等等，而List<String>却只能用来存储String。　

```java
List<Object> objectList;
List<String> stringList;
objectList = stringList;  //compilation error incompatible types
```

## Array中可以用泛型吗?

不可以。这也是为什么 Joshua Bloch 在 《Effective Java》一书中建议使用 List 来代替 Array，因为 List 可以提供编译期的类型安全保证，而 Array 却不能。

## 判断`ArrayList<String>`与`ArrayList<Integer>`是否相等？

```java
ArrayList<String> a = new ArrayList<String>();
ArrayList<Integer> b = new ArrayList<Integer>();
Class c1 = a.getClass();
Class c2 = b.getClass();
System.out.println(c1 == c2); 
```

输出的结果是 `true`。因为无论对于 ArrayList 还是 ArrayList，它们的 `Class类型`都是`ArrayList.class`。

那它们声明时指定的 String 和 Integer 到底体现在哪里呢？

**答案是体现在类编译的时候。**当 JVM 进行类编译时，会进行泛型检查，如果一个集合被声明为 `String` 类型，那么它往该集合存取数据的时候就会对数据进行判断，从而避免存入或取出错误的数据。

# 序列化

## Java序列化与反序列化是什么？

Java序列化是指**把Java对象转换为字节序列**的过程，而Java反序列化是指**把字节序列恢复为Java对象**的过程：

- **序列化：**序列化是把对象转换成有序字节流，以便在网络上传输或者保存在本地文件中。核心作用是**对象状态的保存与重建**。
  - 我们都知道，Java对象是保存在JVM的堆内存中的，也就是说，如果JVM堆不存在了，那么对象也就跟着消失了。
    而序列化提供了一种方案，可以将对象持久化的存储。即把Java对象序列化成可存储或传输的形式（如二进制流），比如保存在文件中。这样，当再次需要这个对象的时候，从文件中读取出二进制流，再从二进制流中反序列化出对象。

- **反序列化：**客户端从文件中或网络上获得序列化后的对象字节流，根据字节流中所保存的对象状态及描述信息，通过反序列化重建对象。

## 为什么需要序列化与反序列化？

简要描述：**对内存中的对象进行持久化或网络传输, 这个时候都需要序列化和反序列化**

深入描述：

1. **对象序列化可以实现分布式对象。**

主要应用例如：`RMI`(即远程调用Remote Method Invocation)要利用对象序列化运行远程主机上的服务，就像在本地机上运行对象时一样。

2. **java对象序列化不仅保留一个对象的数据，而且递归保存对象引用的每个对象的数据。**

可以将整个对象层次写入字节流中，可以保存在文件中或在网络连接上传递。
利用对象序列化可以进行对象的**深复制**，即**复制对象本身及引用的对象本身**。序列化一个对象可能得到整个对象序列。

3. **序列化可以将内存中的类写入文件或数据库中。**

比如：将某个类序列化后存为文件，下次读取时只需将文件中的数据反序列化就可以将原先的类还原到内存中。也可以将类序列化为流数据进行传输。

总的来说就是将一个已经实例化的类转成文件存储，下次需要实例化的时候只要反序列化即可将类实例化到内存中并保留序列化时类中的所有变量和状态。

4. **对象、文件、数据，有许多不同的格式，很难统一传输和保存。**

序列化以后就都是字节流了，无论原来是什么东西，都能变成一样的东西，就可以进行通用的格式传输或保存，传输结束以后，要再次使用，就进行反序列化还原，这样对象还是对象，文件还是文件。

## 序列化实现的方式有哪些？

实现**Serializable**接口或者**Externalizable**接口。

### **Serializable**接口

**类通过实现 `java.io.Serializable` 接口以启用其序列化功能**。可序列化类的所有子类型本身都是可序列化的。
**序列化接口没有方法或字段，仅用于标识可序列化的语义。**

如以下例子：

```java
import java.io.Serializable;

public class User implements Serializable {
   private String name;
   private int age;
   public String getName() {
       return name;
   }
   public void setName(String name) {
       this.name = name;
   }

   @Override
   public String toString() {
       return "User{" +
               "name='" + name +
               '}';
   }
}
```

通过下面的代码进行序列化及反序列化：

```java
public class SerializableDemo {

   public static void main(String[] args) {
       //Initializes The Object
       User user = new User();
       user.setName("cosen");
       System.out.println(user);

       //Write Obj to File
       try (FileOutputStream fos = new FileOutputStream("tempFile"); ObjectOutputStream oos = new ObjectOutputStream(
           fos)) {
           oos.writeObject(user);
       } catch (IOException e) {
           e.printStackTrace();
       }

       //Read Obj from File
       File file = new File("tempFile");
       try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file))) {
           User newUser = (User)ois.readObject();
           System.out.println(newUser);
       } catch (IOException | ClassNotFoundException e) {
           e.printStackTrace();
       }
   }
}

//OutPut:
//User{name='cosen'}
//User{name='cosen'}
```



## **Externalizable**接口

`Externalizable`继承自`Serializable`，该接口中定义了两个抽象方法：`writeExternal()`与`readExternal()`。

当使用`Externalizable`接口来进行序列化与反序列化的时候需要开发人员重写`writeExternal()`与`readExternal()`方法。否则**所有变量的值都会变成默认值**。

```java
public class User implements Externalizable {

   private String name;
   private int age;

   public String getName() {
       return name;
   }
   public void setName(String name) {
       this.name = name;
   }
   public void writeExternal(ObjectOutput out) throws IOException {
       out.writeObject(name);
   }
   public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
       name = (String) in.readObject();
   }

   @Override
   public String toString() {
       return "User{" +
               "name='" + name +
               '}';
   }
}
```

通过下面的代码进行序列化及反序列化：

```java
public class ExternalizableDemo1 {

  public static void main(String[] args) {
      //Write Obj to file
      User user = new User();
      user.setName("cosen");
      try(ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile"))){
          oos.writeObject(user);
      } catch (IOException e) {
          e.printStackTrace();
      }

      //Read Obj from file
      File file = new File("tempFile");
      try(ObjectInputStream ois =  new ObjectInputStream(new FileInputStream(file))){
          User newInstance = (User) ois.readObject();
          //output
          System.out.println(newInstance);
      } catch (IOException | ClassNotFoundException e ) {
          e.printStackTrace();
      }
  }
}

//OutPut:
//User{name='cosen'}
```

## 两种序列化的对比

| 实现Serializable接口                                         | 实现Externalizable接口   |
| :------------------------------------------------------------:| :------------------------: |
| 系统自动存储必要的信息                                       | 程序员决定存储哪些信息   |
| Java内建支持，易于实现，只需要实现该接口即可，无需任何代码支持 | 必须实现接口内的两个方法 |
| 性能略差                                                     | 性能略好                 |

## 什么是serialVersionUID？

serialVersionUID 用来表明**类的不同版本间的兼容性**

Java的序列化机制是通过在运行时判断`类的serialVersionUID`来验证**版本一致性**的。
在进行反序列化时，JVM会把传来的`字节流中的serialVersionUID`与`本地相应实体（类）的serialVersionUID`进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常。

## 为什么还要显示指定serialVersionUID的值?

如果不显示指定serialVersionUID, JVM在序列化时会**根据属性自动生成一个serialVersionUID**, 然后与属性一起序列化, 再进行持久化或网络传输。

在反序列化时, JVM会再**根据属性自动生成一个新版serialVersionUID**, 然后将这个新版serialVersionUID与序列化时生成的旧版serialVersionUID进行比较, 如果相同则反序列化成功, 否则报错.

如果显示指定了, JVM在序列化和反序列化时仍然都会生成一个serialVersionUID, 但值为我们显示指定的值, 这样在反序列化时新旧版本的serialVersionUID就一致了.

在实际开发中, 不显示指定serialVersionUID的情况会导致什么问题? 如果我们的类写完后不再修改, 那当然不会有问题, 但这在实际开发中是不可能的, 我们的类会不断迭代, **一旦类被修改了, 那旧对象反序列化就会报错**. 

所以在实际开发中, 我们都会显示指定一个serialVersionUID, 值是多少无所谓, 只要不变就行。

## serialVersionUID什么时候修改？

《阿里巴巴Java开发手册》中有以下规定：

![](http://blog-img.coolsen.cn/img/image-20210226222339606.png)

想要深入了解的小伙伴，可以看这篇文章：https://juejin.cn/post/6844903746682486791

## Java 序列化中如果有些字段不想进行序列化，怎么办？

对于不想进行序列化的变量，使用 transient 关键字修饰。

`transient` 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中。
在被反序列化后，`transient` 变量的值被设为**初始值**，如 int 型的是 0，对象型的是 null。

> transient 只能修饰变量，不能修饰类和方法。

## 静态变量会被序列化吗?

不会。因为序列化是针对对象而言的, 而**静态变量优先于对象存在, 随着类的加载而加载, 所以不会被序列化**

看到这个结论, 是不是有人会问, serialVersionUID也被static修饰, 为什么serialVersionUID会被序列化? 
其实serialVersionUID属性并没有被序列化, JVM在序列化对象时会自动生成一个serialVersionUID, 然后**将我们显示指定的serialVersionUID属性值赋给自动生成的serialVersionUID**。

# 异常

## Error 和 Exception 区别是什么？

 Java 中，所有的异常都有一个共同的祖先 `java.lang` 包中的 `Throwable` 类。
 `Throwable` 类有两个重要的子类 `Exception`（异常）和 `Error`（错误）。

`Exception` 和 `Error` 二者都是 Java 异常处理的重要子类，各自都包含大量子类。

- **`Exception`** :程序本身可以处理的异常，可以通过 `catch` 来进行捕获，通常遇到这种错误，应对其进行处理，使应用程序可以继续正常运行。`Exception` 又可以分为**运行时异常**(RuntimeException, 又叫**非受检查异常**)和**非运行时异常**(又叫**检查型异常**) 。
- **`Error`** ：`Error` 属于程序无法处理的错误 ，我们没办法通过 `catch` 来进行捕获 。例如，系统崩溃，内存不足，堆栈溢出等，编译器不会对这类错误进行检测，一旦这类错误发生，通常应用程序会被终止，仅靠应用程序本身无法恢复。

![](http://blog-img.coolsen.cn/img/image-20210227103256234.png)

## 非受检查异常(运行时异常)和受检查异常(一般异常)区别是什么？

非受检查异常：包括 `RuntimeException` 类及其子类，表示 JVM 在运行期间可能出现的异常。 **Java 编译器不会检查运行时异常**。例如：`NullPointException(空指针)`、`NumberFormatException（字符串转换为数字）`、`IndexOutOfBoundsException(数组越界)`、`ClassCastException(类转换异常)`、`ArrayStoreException(数据存储异常，操作数组时类型不一致)`等。

受检查异常：是Exception 中除 `RuntimeException` 及其子类之外的异常。 **Java 编译器会检查受检查异常**。常见的受检查异常有： IO 相关的异常、`ClassNotFoundException` 、`SQLException`等。

**非受检查异常和受检查异常之间的区别**：是否强制要求调用者必须处理此异常：
- 检查型异常：调用者必须处理
- 非检查型异常：不强制处理

## throw 和 throws 的区别是什么？

Java 中的异常处理除了包括捕获异常和处理异常之外，还包括声明异常和拋出异常，可以通过 `throws` 关键字在**方法**上声明该方法要拋出的异常，或者在**方法内部**通过 `throw` 拋出异常对象。

throws 关键字和 throw 关键字在使用上的几点区别如下：

* `throw` 关键字用在**方法内部**，只能用于**抛出一种异常**，用来抛出方法或代码块中的异常，受查异常和非受查异常都可以被抛出。
* `throws` 关键字用在**方法声明**上，可以抛出多个异常，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了可能抛出的异常列表，调用该方法的方法中必须包含可处理异常的代码，否则也要在方法签名中用 throws 关键字声明相应的异常。

## NoClassDefFoundError 和 ClassNotFoundException 区别？

`NoClassDefFoundError` 是一个 Error 类型的异常，是由 JVM 引起的，不应该尝试捕获这个异常。
    引起该异常的原因是 `JVM` 或 `ClassLoader` **尝试加载某类时在内存中找不到该类的定义**，该动作发生在运行期间，即**编译时该类存在，但是在运行时却找不到了**，可能是编译后被删除了等原因导致。

`ClassNotFoundException` 是一个受检查异常，需要显式地使用 `try-catch` 对其进行捕获和处理，或在方法签名中用 throws 关键字进行声明。
    当使用 `Class.forName`, `ClassLoader.loadClass` 或 `ClassLoader.findSystemClass` **动态加载类到内存**的时候，通过传入的类路径参数没有找到该类，就会抛出该异常；另一种抛出该异常的可能原因是**某个类已经由一个类加载器加载至内存中，另一个加载器又尝试去加载它**。


## Java常见异常有哪些？
* java.lang.IllegalAccessError：违法访问错误。当一个应用试图访问、修改某个类的域（Field）或者调用其方法，但是又违反域或方法的可见性声明，则抛出该异常。
* java.lang.InstantiationError：实例化错误。当一个应用试图通过Java的new操作符构造一个抽象类或者接口时抛出该异常.
* java.lang.OutOfMemoryError：内存不足错误。当可用内存不足以让Java虚拟机分配给一个对象时抛出该错误。
* java.lang.StackOverflowError：堆栈溢出错误。当一个应用递归调用的层次太深而导致堆栈溢出或者陷入死循环时抛出该错误。
* java.lang.ClassCastException：类造型异常。假设有类A和B（A不是B的父类或子类），O是A的实例，那么当强制将O构造为类B的实例时抛出该异常。该异常经常被称为强制类型转换异常。
* java.lang.ClassNotFoundException：找不到类异常。当应用试图根据字符串形式的类名构造类，而在遍历CLASSPAH之后找不到对应名称的class文件时，抛出该异常。
* java.lang.ArithmeticException：算术条件异常。譬如：整数除零等。
* java.lang.ArrayIndexOutOfBoundsException：数组索引越界异常。当对数组的索引值为负数或大于等于数组大小时抛出。
* java.lang.IndexOutOfBoundsException：索引越界异常。当访问某个序列的索引值小于0或大于等于序列大小时，抛出该异常。
* java.lang.InstantiationException：实例化异常。当试图通过newInstance()方法创建某个类的实例，而该类是一个抽象类或接口时，抛出该异常。
* java.lang.NoSuchFieldException：属性不存在异常。当访问某个类的不存在的属性时抛出该异常。
* java.lang.NoSuchMethodException：方法不存在异常。当访问某个类的不存在的方法时抛出该异常。
* java.lang.NullPointerException：空指针异常。当应用试图在要求使用对象的地方使用了null时，抛出该异常。譬如：调用null对象的实例方法、访问null对象的属性、计算null对象的长度、使用throw语句抛出null等等。
* java.lang.NumberFormatException：数字格式异常。当试图将一个String转换为指定的数字类型，而该字符串确不满足数字类型要求的格式时，抛出该异常。
* java.lang.StringIndexOutOfBoundsException：字符串索引越界异常。当使用索引值访问某个字符串中的字符，而该索引值小于0或大于等于序列大小时，抛出该异常。

## try-catch-finally 中哪个部分可以省略？

catch 可以省略。更为严格的说法其实是：try只适合处理运行时异常，try+catch适合处理运行时异常+一般异常。
也就是说，如果你只用try去处理一般异常（非运行时异常，检查型异常）却不加以catch处理，编译是通不过的，因为编译器硬性规定，非运行时异常如果选择捕获，则必须用catch显示声明以便进一步处理。而运行时异常在编译时没有如此规定，所以catch可以省略，你加上catch编译器也觉得无可厚非。

理论上，编译器看任何代码都不顺眼，都觉得可能有潜在的问题，所以你即使对所有代码加上try，代码在运行期时也只不过是在正常运行的基础上加一层皮。但是你一旦对一段代码加上try，就等于显示地承诺编译器，对这段代码可能抛出的异常进行捕获而非向上抛出处理。如果是普通异常，编译器要求必须用catch捕获以便进一步处理；如果运行时异常，捕获然后丢弃并且+finally扫尾处理，或者加上catch捕获以便进一步处理。

至于加上finally，则是在不管有没捕获异常，都要进行的“扫尾”处理。

## try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

==会执行，在 return 前执行==

在 finally 中改变返回值的做法是不好的，因为如果存在 finally 代码块，==try中的 return 语句不会立马返回调用者==，而是记录下返回值待 finally 代码块执行完毕之后再向调用者返回其值，==然后如果在 finally 中修改了返回值，就会返回修改后的值==。
> finally中return的值优先


显然，在 finally 中返回或者修改返回值会对程序造成很大的困扰，Java 中也可以通过提升编译器的语法检查级别来产生警告或错误。
**代码示例1：**

```java
public static int getInt() {
    int a = 10;
    try {
        System.out.println(a / 0);
        a = 20;
    } catch (ArithmeticException e) {
        a = 30;
        return a;
        /*
         * return a 在程序执行到这一步的时候，这里不是return a 而是 return 30；这个返回路径就形成了
         * 但是呢，它发现后面还有finally，所以继续执行finally的内容，a=40
         * 再次回到以前的路径,继续走return 30，形成返回路径之后，这里的a就不是a变量了，而是常量30
         */
    } finally {
        a = 40;
    }
	return a;
}

//执行结果：30
```

**代码示例2：**

```java
public static int getInt() {
    int a = 10;
    try {
        System.out.println(a / 0);
        a = 20;
    } catch (ArithmeticException e) {
        a = 30;
        return a;
    } finally {
        a = 40;
        //如果这样，就又重新形成了一条返回路径，由于只能通过1个return返回，所以这里直接返回40
        return a; 
    }

}

// 执行结果：40
```

## JVM 是如何处理异常的？

在一个方法中如果发生异常，这个方法会创建一个**异常对象**，并转交给 JVM，该异常对象包含异常名称，异常描述以及异常发生时应用程序的状态。
**创建异常对象并转交给 JVM 的过程**称为抛出异常。
可能有一系列的方法调用，最终才进入抛出异常的方法，这一系列方法调用的有序列表叫做调用栈。

JVM 会顺着调用栈去查找看是否有可以处理异常的代码：
  - 如果有，则调用异常处理代码。当 JVM 发现可以处理异常的代码时，会把发生的异常传递给它。
  - 如果 JVM 没有找到可以处理该异常的代码块，JVM 就会将该异常转交给默认的异常处理器（默认处理器为 JVM 的一部分），默认异常处理器打印出异常信息并终止应用程序。


想要深入了解的小伙伴可以看这篇文章：https://www.cnblogs.com/qdhxhz/p/10765839.html

# IO

## Java的IO 流分为几种？

* 按照流的方向：
  * 输入流（inputStream）
  * 输出流（outputStream）
* 按照实现功能分：
  * 节点流（可以**从或向一个特定的地方读写数据**，如 FileReader）
  * 处理流（是**对一个已存在的流的连接和封装**，通过所封装的流的功能调用实现数据读写， BufferedReader）；
* 按照处理数据的单位： 
  * 字节流和字符流
    * 分别由四个抽象类来表示（每种流包括输入和输出两种所以一共四个）:InputStream，OutputStream，Reader，Writer。Java中其他多种多样变化的流均是由它们派生出来的。

## 字节流如何转为字符流？

字节输入流转字符输入流通过 `InputStreamReader` 实现，该类的构造函数可以传入 `InputStream` 对象。

字节输出流转字符输出流通过 `OutputStreamWriter` 实现，该类的构造函数可以传入 `OutputStream` 对象。

## 字符流与字节流的区别？

- 读写时，字节流是**按字节读写**，字符流**按字符读写**。
- 可处理的数据类型：
  - 字节流适合**所有类型文件**的数据传输，因为计算机字节（Byte）是电脑中表示信息含义的最小单位。
  - 字符流**只能够处理纯文本**数据，其他类型数据不行，但是字符流处理文本要比字节流处理文本要方便。
- 如何选择
  - 需要对内容按行处理，比如比较特定字符，处理某一行数据的时候一般会选择字符流。
  - 只是读写文件，和文件内容无关时，一般选择字节流。

## BIO、NIO、AIO的区别？

- BIO：**同步并阻塞**，在服务器中实现的模式为**一个连接一个线程**。
  - 客户端有连接请求的时候，服务器就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然这也可以通过线程池机制改善。
  - BIO**一般适用于连接数目小且固定的架构**，这种方式对于服务器资源要求比较高，而且并发局限于应用中，是JDK1.4之前的唯一选择，但好在程序直观简单，易理解。
- NIO：**同步并非阻塞**，在服务器中实现的模式为**一个请求一个线程**。
  - 客户端发送的连接请求都会**注册到多路复用器**上，**多路复用器轮询到有连接IO请求时才会启动一个线程进行处理**。
  - **NIO一般适用于连接数目多且连接比较短（轻操作）的架构**，并发局限于应用中，编程比较复杂，从JDK1.4开始支持。
- AIO：**异步非阻塞**，在服务器中实现的模式为**一个有效请求一个线程**。
  - 客户端的IO请求都是通过操作系统先完成之后，再通知服务器应用去启动线程进行处理。
  - AIO一般适用于连接数目多且连接比较长（重操作）的架构，充分调用操作系统参与并发操作，编程比较复杂，从JDK1.7开始支持。

## 介绍一下IO多路复用

## Java IO都有哪些设计模式？

使用了**适配器模式**和**装饰器模式**

**适配器模式**：

```java
Reader reader = new INputStreamReader(inputStream);
```

**把一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作**

- **类适配器**：Adapter类（适配器）继承Adaptee类（源角色）实现Target接口（目标角色）
- **对象适配器**：Adapter类（适配器）持有Adaptee类（源角色）对象实例，实现Target接口（目标角色）
  ![](http://blog-img.coolsen.cn/img/image-20210227114919307.png)

**装饰器模式**：

```java
new BufferedInputStream(new FileInputStream(inputStream));
```

**一种动态地往一个类中添加新的行为的设计模式。就功能而言，装饰器模式相比生成子类更为灵活，这样可以给某个对象而不是整个类添加一些功能。**

- ConcreteComponent（具体对象）和Decorator（抽象装饰器）实现相同的Conponent（接口）并且Decorator（抽象装饰器）里面持有Conponent（接口）对象，可以传递请求。
- ConcreteComponent（具体装饰器）覆盖Decorator（抽象装饰器）的方法并用super进行调用，传递请求。

![](http://blog-img.coolsen.cn/img/image-20210227115040999.png)

## 浅拷贝与深拷贝的实现方式

浅拷贝：调用`(User) super.clone()`，然后返回即可

深拷贝：在调用`(User) super.clone()`之后，还需要调用其中的每个域的clone()方法：例如`user.person = (Person) person.clone()`


# 注解

## 介绍一下注解

注解是JDK5引入的一种安全的类似注释的机制，用来将任何的**信息或元数据**（metadata）与程序元素（类、方法、成员变量等）进行关联，在编译时或运行时解析并处理这些元数据，为程序的元素（类、方法、成员变量）加上更直观更明了的说明。
这些由注解引入的说明信息是与程序的业务逻辑无关，并且供指定的工具或框架使用

**注解的作用：**

1. 作为特定标记，用于告诉编译器一些信息
2. 编译时动态处理，如动态生成代码
3. 运行时动态处理，作为额外信息的载体，如获取注解信息

**注解的分类：**
1. 元注解：java内置的注解，**标明该注解的使用范围、生命周期等**
2. 标准注解：Java提供的基础注解，**标明过期的元素/标明是复写父类方法的方法/标明抑制警告**
3. 自定义注解：第三方定义的注解，含义和功能由第三方来定义和实现

**注解的应用场景：**

1. 生成文档。
   - 这是最常见的，也是java 最早提供的注解。常用的有@param @return 等
2. 跟踪代码依赖性，实现替代配置文件功能。
   - 比如Dagger 2 依赖注入，未来java 开发，将大量注解配置，具有很大用处;
3. 在编译时进行格式检查。
   - 如@override 放在方法前，如果你这个方法并不是覆盖了超类方法，则编译时就能检查出。

## 注解的工作原理

注解本质是一个继承了`Annotation` 的特殊接口，注解本身是不会有行为的，它只是一个被动的元数据的载体。
想要处理注解，就需要借助额外的手段，而注解的处理方式分为两种：
1. 运行时处理：通过**反射**机制处理注解，
2. 编译时处理：通过**APT(注解处理器)1**处理注解
   - 编译时注解，是在 java 编译生成 .class 文件这一步进行的操作 

将注解按照生命周期来划分，包含以下三类，其运行时期取决于`@Retention`的取值：
- `RetentionPolicy.SOURCE`，注解只存在`.java`源文件中，Java文件编译为`.class`文件时遗弃，在**运行时**可以获得注解的信息，可在运行时处理
  - 只是做一些检查型的操作，例如`@Override`、`@SuppressWarnings`
- `RetentionPolicy.CLASS`，注解只存在于`.class`文件中，JVM加载`.class`文件时遗弃，所以**无法在运行时去获取注解的信息，只能在编译时处理**。（默认恶毒生命周期）
  - 用于在编译时进行一些预处理操作，生成一些辅助代码
- `RetentionPolicy.RUNTIME`，注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在，所以就可在**运行时**获取注解的信息
  - 用于在运行时动态获取注解信息

分别对应：Java源文件--->.class文件--->内存中的字节码

对**编译时注解**，需要使用`APT`技术进行处理，该技术提供了一套**编译期的注解处理流程**

`APT`是一种处理注解的工具，确切的说它是 `javac` 的一个工具，它用来**在编译时扫描和处理注解**，是针对编译时注解的注解处理器。
在编译期扫描`.java`文件的注解，并传递到注解处理器，注解处理器可根据注解生成新的`.java`文件，与原来的`.java`文件一起被javac编译。多用于对自定义注解的处理。

 注解处理器是一个在javac编译期处理注解的工具，你可以创建注解处理器并注册，在编译期你创建的处理器以Java代码作为输入，生成文件.java文件作为输出。
> 注意：注解处理器==不能修改已经存在的Java类（即不能向已有的类中添加方法），只能生成新的Java类==。

注解处理器需要继承`AbstractProcessor`，并实现如下方法：
- `synchronized void init(ProcessingEnvironment processingEnv)`：用于初始化注解处理器，其参数为APT框架的环境对象，可以获得很多工具类
  - 包括elementUtils与typeUtils等
- `Set<String> getSupportedAnnotationTypes()`：配置该注解处理器需要处理的注解类型
- `SourceVersion getSupportedSourceVersion()`：配置该注解处理器支持的最新版本
- `boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv)`：用于处理注解，并生成.java文件

当我们添加编译时注解后，在编译时，编译器会检查`AbstractProcessor`的子类，并且调用该类型的 process 函数，然后将添加了注解的所有元素都传递到 process 函数中，使得开发人员可以在编译器进行相应的处理。

> 注意：使用APT处理编译时注解前，首先需要将对应的注解处理器注册到/resources/META-INF/services中

但在JDK1.8之后，APT被移除了，取而代之的是插入式注解处理API

对**运行时注解**，需要在**注解处理器**通过**反射**对注解进行处理，反射也是唯一能够获得运行时注解内信息的手段。
然而，注解的本质是**特殊的接口**，而没有实现接口是无法调用其中的方法的，因此Java采用==动态代理==的技术实现了注解内方法的调用。

我们通过反射获取注解时，Java会在运行时生成动态代理对象`$Proxy1`，而这个动态代理对象实现了我们传入的注解(接口)，并实现了注解中的方法，同时继承了`Proxy`，而实现的方式是调用父类`Proxy`的`AnnotationInvocationHandler`的`invoke`方法。
而`invoke`方法会从`memberValues` 这个Map中获取需要的元数据，从而实现方法。而`memberValues` 的来源是Java 常量池

对于自定义的注解，只需要自定义注解处理器，然后通过注解处理器调用被注解的类、方法即可

## 编译时注解与运行时注解有哪些，分别有哪些应用场景

编译时注解：

运行时注解：

应用场景：
- RetentionPolicy.SOURCE（编译时注解）：
  1. APT 在编译期能够获取注解与注解声明的类,包括类中的 所有成员信息,一般用于生成额外的辅助类
  2. 可以提供IDE语法检查,取代枚举的使用 
- RetentionPolicy.CLASS（运行时注解）：
  1. 在编译出Class后，通过修改Class数据以实现修改代码逻辑目的
  2. 对于是否需要修改的区分或者修改为不同逻辑的判断可以使用注解 
- RetentionPolicy.RUNTIME（运行时注解）：
  1. 通过反射技术获得注解及其成员，从而完成不同的逻辑判断 

## 元注解是啥

元注解的作用就是负责注解其他注解。或者说元注解是一种基本注解，但是它能够应用到其它注解上面。

主要包含：
1. `@Target`：用于注定注解运用的地方。当一个注解被 @Target 注解时，这个注解就被限定了运用的场景。 
2. `@Rentention`：说明注解的存活时间
3. `@Document`：说明注解能否被文档化
4. `@Inhrited`：说明注解能否被继承
5. `@Repeatable`：JDK1.8引入，添加该注解后，当前注解就可以重复添加到容器中了。注解里面的值是容器注解的class对象（容器注解的一个域为当前注解的容器）

## 标准注解是啥

标准注解即内置注解，是Java提供的基础注解，标准注解主要包含以下三个：
1. `@Override`：表示当前的方法定义将覆盖父类中的方法
2. `@Deprecated`：表示代码被弃用，如果使用了被@Deprecated注解的代码则编译器将发出警告
3. `@SuppressWarnings`：表示关闭编译器警告信息


## 注解会体现在字节码文件（.class文件）中吗

这取决于注解的生命周期，即`@Rentention`的取值：
`RententionPolicy.SOURCE`只存在于源文件`.java`文件中，在编译到`.class`文件时就会被遗弃
而`RententionPolicy.CLASS`与`RententionPolicy.RUNTIME`会保留到`.class`文件中

## 自定义注解

`@interface`是声明注解的关键字，每个注解需要注明生命周期`@Rentention`以及作用范围`@Target`，而注解内部的定义也有规定:
- 注解方法不能带有参数。
- 注解方法返回值类型限定为：基本类型、String、Enums、Annotation或者这些类型的数组。
- 注解方法可以有默认值。
- 注解本身能够包含元注解，元注解被用来注解其他注解。



# JDK8的新特性

## 了解Lambda表达式吗



## 了解函数式编程吗



## 正则表达式



# 其他乱七八糟

## 怎么找到耗时较长的Java代码


## Unicode

https://mp.weixin.qq.com/s/QjU9lSekpbaF7fugZbyzkg




