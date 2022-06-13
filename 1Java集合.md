# 基础

## 常见的集合有哪些？

Java集合类主要由两个根接口**Collection**和**Map**派生出来的，Collection派生出了三个子接口：`List`、`Set`、`Queue`（Java5新增的队列），因此Java集合大致也可分成List、Set、Queue、Map四种接口体系。

**注意：Collection是一个接口，Collections是一个工具类，Map不是Collection的子接口**。


- List代表了有序可重复集合，可直接根据元素的索引来访问；
- Set代表无序不可重复集合，只能根据元素本身来访问；
- Queue是队列集合。
- Map代表的是存储key-value对的集合，可根据元素的key来访问value。

Java集合框架图如下：

![](http://blog-img.coolsen.cn/img/image-20210403163733569.png)

![](http://blog-img.coolsen.cn/img/image-20210403163751501.png)

上图中淡绿色背景覆盖的是集合体系中常用的实现类，分别是ArrayList、LinkedList、ArrayQueue、HashSet、TreeSet、HashMap、TreeMap等实现类。

## 线程安全的集合有哪些？线程不安全的呢？

线程安全的：

- Hashtable：比HashMap多了个线程安全。
- ConcurrentHashMap:是一种高效但是线程安全的集合。
- Vector：比Arraylist多了个同步化机制。
- Stack：栈，也是线程安全的，继承于Vector。

线性不安全的：

- HashMap
- Arraylist
- LinkedList
- HashSet
- TreeSet
- TreeMap

## transient的作用是什么

- 一旦加上`trainsient`，变量就不再是对象持有化的一部分，即不被序列化
    > 静态变量都不能被序列化
    反序列化时是通过JVM得到类的变量赋值的

- `trainsient`只能修饰变量。本地变量不能被`trainsient`修饰。如果是用户自定义类变量，需要实现`Serializable`接口

> 若实现的是`Serializable`接口，则所有的序列化将会自动进行，符合`transient`的规则
> 若实现的是`Externalizable`接口，则没有任何东西可以自动序列化，需要在`writeExternal`方法中进行手工指定所要序列化的变量，这与是否被`transient`修饰无关

# List


## ArrayList的细节

- ArrayList是基于Object数组实现的，增删的时候需要调用**数组的拷贝复制**
- ArrayList的默认初始化容量为10，每次扩容变为原来的**1.5倍**
- 删除元素时不会减少内部数组的容量，而是直接移动数组中的元素，空出来的null让GC回收
  - >如果希望减少容量，应调用`trimToSize()`


## Arraylist与 LinkedList 异同点？

- **是否保证线程安全：** ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；
- **底层数据结构：** Arraylist 底层使用的是Object数组；LinkedList 底层使用的是双向循环链表数据结构；
- **插入和删除是否受元素位置的影响：** **ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 
  - 比如：执行`add(E e)`方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O(1)。
  - 但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。  
  - **LinkedList 采用链表存储，所以插入，删除元素时间复杂度不受元素位置的影响，都是近似 O（1）而数组为近似 O（n）。**
- **是否支持快速随机访问：** 
  - `ArrayList` 实现了`RandmoAccess` 接口，所以有随机访问功能。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
  - `LinkedList` 不支持高效的随机元素访问，
- **内存空间占用：** 
  - `ArrayList`的空间浪费体现在：list列表的结尾会预留一定的容量空间，
  - `LinkedList`的空间花费体现在：它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。

##  ArrayList 与 Vector 区别？

* Vector是线程安全的，ArrayList不是线程安全的。
  * `Vector`在关键性的方法前面都加了`synchronized`关键字，来保证线程的安全性。
* `ArrayList`在底层数组不够用时在原来的基础上扩展`0.5`倍，`Vector`是扩展`1`倍，这样ArrayList就有利于节约内存空间。

##  说一说ArrayList 的扩容机制？

ArrayList扩容的本质就是计算出新的扩容数组的size后实例化，并将原有数组内容复制到新数组中去。**默认情况下，新的容量会是原容量的1.5倍**。

以JDK1.8为例说明:
1. 在`add(E e)`时，判断是否可以容纳`e`，若能，则直接添加在末尾；若不能，则在`grow()`方法中**进行扩容**，然后再把e添加在末尾
   - 通过`ensureCapacityInternal()`方法判断能否容纳
2. 首先获得ArrayList内部的`elementData`数组的长度，将其扩大为`1.5`倍，并在保证不溢出（不大于默认的最大值）的情况下，申请新的内存空间，将旧的数据复制到新的`elementData`中

## Array 和 ArrayList 有什么区别？什么时候该应 Array 而不是 ArrayList 呢？

* Array 可以包含**基本类型和对象类型**，ArrayList 只能包含**对象类型**。

* Array 大小是固定的，ArrayList 的大小是动态变化的。

* ArrayList 提供了更多的方法和特性，比如：addAll()，removeAll()，iterator() 等等。



## arraylist扩容，为什么是1.5倍

在Vector中，扩容因子的大小为2，而扩容因子的最佳范围为(1,2)
- 由于ArrayList是需要分配连续的内存空间的，如果扩容因子为1.5，可以使得ArrayList使用的连续内存空间的扩张速度减慢，避免了没有足够的内存空间，导致垃圾回收的频率增高
- 其次，当扩容因子为1.5，多次扩容后，还可能可以利用之前的连续内存空间，而如果扩容因子>2，就刚好大于之前的所有内存空间的和
- 而扩容因子是1.5，而不是1.25、1.75等值的原因，是因为这样的计算可以利用移位操作减少浮点数或者运算时间和运算次数。
  `int newCapacity = oldCapacity + (oldCapacity >> 1);`

## ArrayList里面有100个元素怎么删除中间的30到40；

如果直接在循环中删除，要注意删除后容量的变化：
```java
for(int i = 0 , len= list.size();i<len;++i){
  if(list.get(i)==XXX){
       list.remove(i);
  }
}
```

在上述方式中，由于`list.remove(i)`改变了len，如果最后还是遍历到len-1，就会越界。

也可以考虑通过`iterator()`实现数据的删除

## ArrayList和LinkedList的查找复杂度，怎么优化LinkedList的查找复杂度（方式1hashmap但会增加空间复杂度，方式2跳表，跳表怎么实现怎么存储）

由于ArrayList实现了RandomAccess，访问元素的时间复杂度为O(1)，但LinkedList为链式存储，只能一个一个遍历过去，访问的时间复杂度为O(n)，但查找的时间复杂度，二者均为O(n)；

LinkedList的查找时间复杂度优化：
1. 使用HashMap存储LinkedList中的数据
2. 使用跳表skip-list

## ArrayList和LinkedList的效率差别




## 除了ArrayList和LinkedList之外，还有什么List


## 知道迭代器么，用迭代器遍历和普通 for 循环有什么区别

使用迭代器遍历的速度要快于for循环（当然，这里指的是普通的for(int i = 1)这样的循环）

foreach循环底层用的也是迭代器



## 用迭代器删除元素，会有什么问题吗

迭代器可以通过`remove()`方法删除元素，且不会抛出并发修改异常

## 介绍一下跳表

![](./../images/集合框架/跳表.jpeg)

跳表的大致结构如上图所述，即通过建立索引的方式，将链表的顺序查询O(n)变为自底向上的二分查找`O(log(n))`

跳表的空间复杂度为`O(n)`，是典型空间换时间。但和B+树索引一样，跳表的索引节点也是只需要存储几个关键的值与指针，不需要存储完整的记录，所以占的空间不大。

### 跳表的查询：

从最高级别的节点开始，比较当前级的下一节点是否等于要查找的节点：
- 如果下一节点的值大于要查找的节点，说明当前级别的跨度太大了，则到当前节点所在的下一级继续寻找
- 如果下一节点小于要查找的节点，则从下一节点的当前级继续查找。

若以每两个节点建立一个上级节点的规则，则跳表的高度为$log_2n$，每一级索引 最多需要遍历3个节点，因此查询次数的期望值为$3*log_2n$，时间复杂度为`O(logn)`
此时，空间复杂度为O(n)，但节点数是链表形式的2倍


### 跳表的删除

删除需要改变链表结构所以需要处理好节点之间的联系。有以下几点原则：
1. 删除当前节点和这个节点的**前后节点**都有关系
2. 删除当前层节点之后，**下一层该key的节点也要删除，一直删除到最底层**

当删除一个节点时，首先需要找到其前驱节点，并通过前驱逐层删除：

设置`team = head`，`while(team != null)`：
1. 如果team右侧为null，那么`team=team.down`(之所以敢直接这么判断是因为左侧有头结点在左侧，不用担心特殊情况)

2. 如果team右侧不为null，并且右侧的key等于待删除的key，那么先删除节点，再`team=team=team.down`，继续删除target的下层节点。

3. 如果team右侧不为null，并且右侧key小于待删除的key，那么team向右`team=team.right`。

4. 如果team右侧不为null，并且**右侧key大于待删除的key**，那么team向下team=team.down，在下层继续查找删除节点。

总而言之，就是在每一级找到`target`的前驱节点，删除完当前层的前驱后，到下一层继续找`target`的前驱。

### 跳表的增加

假如一直往原始列表中添加数据，但是不更新索引，就可能出现两个索引节点之间数据非常多的情况，极端情况下会退化为单链表。

因此，在插入数据时，也需要对索引进行维护，包含以下常用方法：
- 完全重建索引
  - 我们每次插入数据后，都把这个跳表的索引删掉全部重建
  - 建立索引的时间复杂度为O(n)，导致插入数据的时间复杂度变为O(n)
- 通过随机选取节点的方式建立索引
  > 在每一级索引，都维护`n/2`个索引节点，n为上一级的节点数。即原始链表有`n`个节点，第一级索引有`n/2`个，第二级有`n/4`个...这样一来，就可以避免节点数量过多导致的效率降低
  - 有数据要插入时，先通过随机函数计算该元素应该**插入到几级索引**中
    - 而当插入高级索引时，需要依次在低级索引与原始链表插入节点，这样就实现了一级索引的节点数是原始链表的`1/2`，二级链表是一级链表的`1/4`...
```java
 int rand_level()
{
    int ret = 1;
    while (rand() % 2 && ret <= MAX_LEVEL)
        ++ret;
    return ret;
}
```

# HashMap

  
## HashMap的底层数据结构是什么？

在JDK1.7 和JDK1.8 中有所差别：

在JDK1.7 中，由**数组+链表**组成，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的。

在JDK1.8 中，由**数组+链表+红黑树**组成。当链表过长，则会严重影响 HashMap 的性能，红黑树搜索时间复杂度是 O(logn)，而链表是糟糕的 O(n)。因此，JDK1.8 对数据结构做了进一步的优化，引入了红黑树，链表和红黑树在达到一定条件会进行转换：

* 当==链表超过 8 且数据总量超过 64==才会转红黑树。

* 将链表转换成红黑树前会判断，如果**当前数组的长度小于 64，那么会选择先进行数组扩容**，而不是转换为红黑树，以减少搜索时间。

![Jdk1.8 HashMap结构](http://blog-img.coolsen.cn/img/image-20210112185830788.png)

## 解决hash冲突的办法有哪些？HashMap用的哪种？

解决Hash冲突方法有:
- 开放定址法
  - 如果`p=H(key)`出现冲突时，则以`p`为基础，再次查找，直到不冲突位置，具体查找方法如下：
    1. 线性探测再散列，冲突发生时，顺序查看表中下一单元，直到找出一个空单元或查遍全表
    2. 二次探测再散列，冲突发生时，在表的左右进行跳跃式探测，比较灵活$p\pm i^2$
    3. 伪随机探测再散列，以随机数序列作为冲突时累加的值
       - 例如序列为`2,5,9 `，则哈希函数分别为`p1 = (p + 2) % len`;`p2 = (p1 + 5) % len`...
  - 开放定址法所需要的hash表的长度要**大于等于所需要存放的元素**，而且因为存在再次hash，所以**只能在删除的节点上做标记，而不能真正删除节点**
- 再哈希法（双重散列、多重散列）
  - 提供多个不同的`hash`函数，当一个函数冲突时，则**重新计算**其他哈希函数，直到没有冲突
  - 不易产生堆积，但增加了计算时间
- 链地址法
  - 将哈希值相同的元素构成一个同义词的**单链表**，并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。链表法适用于经常进行插入和删除的情况。
- 建立公共溢出区
  - 将哈希表分为公共表与溢出表，溢出发生时，将所有溢出数据统一放到溢出区
 
HashMap中采用的是 链地址法 。

## 四种解决哈希冲突方法有什么优缺点：
**开放散列/拉链法（针对桶链结构）**

优点：
1. 对于**记录总数频繁变化**的情况，处理的比较好（也就是避免了动态调整的开销） 
2. 由于记录存储在结点中，而结点是动态分配，不会造成内存的浪费，所以尤其适合那种记录本身尺寸（size）很大的情况，因为此时指针的开销可以忽略不计了 
3. 删除记录时，比较方便，直接通过指针操作即可


缺点：
1. 存储的记录是**随机分布在内存中**的，这样在查询记录时，相比结构紧凑的数据类型（比如数组），哈希表的跳转访问会带来额外的时间开销 
2. 由于使用指针，记录不容易进行序列化（serialize）操作


**封闭散列（closed hashing）/ 开放定址法**

优点： 
1. 记录更容易进行序列化（`serialize`）操作
2. 如果记录总数可以预知，可以创建**完美哈希函数**，此时处理数据的效率是非常高的

缺点：
1. 存储记录的数目不能超过桶数组的长度，如果**超过就需要扩容**，而扩容会导致某次操作的时间成本飙升，这在实时或者交互式应用中可能会是一个严重的缺陷
2. 使用探测序列，有可能其计算的时间成本过高，导致哈希表的处理性能降低
3. 由于记录是存放在桶数组中的，而桶数组必然存在**空槽**，所以当记录本身尺寸（size）很大并且记录总数规模很大时，空槽占用的空间会导致明显的内存浪费
4. 删除记录时，比较麻烦。
   - 比如需要删除记录a，记录b是在a之后插入桶数组的，但是和记录a有冲突，是通过探测序列再次跳转找到的地址，所以如果直接删除a，a的位置变为空槽，而空槽是查询记录失败的终止条件，这样会导致记录b在a的位置重新插入数据前不可见，所以不能直接删除a，而是设置**删除标记**。这就需要额外的空间和操作。

## 为什么在解决 hash 冲突的时候，不直接用红黑树？而选择先用链表，再转红黑树?

因为红黑树需要进行左旋，右旋，变色这些操作来保持平衡，而单链表不需要。当元素小于 8 个的时候，此时做查询操作，链表结构已经能保证查询性能。当元素大于 8 个的时候， 红黑树搜索时间复杂度是 O(logn)，而链表是 O(n)，此时需要红黑树来加快查询速度，但是新增节点的效率变慢了。

因此，如果一开始就用红黑树结构，元素太少，新增效率又比较慢，无疑这是浪费性能的。

## 不用红黑树，用二叉查找树可以么?

可以。但是二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。

##  为什么链表改为红黑树的阈值是 8?

理想情况下使用随机的哈希码，容器中节点分布在 hash 桶中的频率遵循[泊松分布](http://en.wikipedia.org/wiki/Poisson_distribution)，按照泊松分布的计算公式计算出了桶中元素个数和概率的对照表，可以看到链表中元素个数为 8 时的概率已经非常小，再多的就更少了，所以原作者在选择链表元素个数时选择了 8，是根据概率统计而选择的。

## HashMap默认加载因子是多少？为什么是 0.75，不是 0.6 或者 0.8 ？

回答这个问题前，我们来先看下HashMap的默认构造函数：

```java
     int threshold;             // 容纳键值对的最大值
     final float loadFactor;    // 负载因子
     int modCount;  
     int size;  
```

- `Node[] table`的初始化长度`length`(默认值是16)
- `Load factor`为负载因子(默认值是0.75)，
- `threshold`是HashMap所能容纳键值对的最大值。

`threshold = length * Load factor`。也就是说，在数组定义好长度之后，负载因子越大，所能容纳的键值对个数越多。

默认的loadFactor是0.75，0.75是对空间和时间效率的一个平衡选择，一般不要修改，除非在时间和空间比较特殊的情况下 ：

* 如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值 。

* 相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。

设置初始大小时，应该考虑预计的entry数在map及其负载系数，并且尽量减少rehash操作的次数。如果初始容量大于最大条目数除以负载因子，rehash操作将不会发生。

## HashMap是怎样初始化的

HashMap主要包含四个构造器，暂时不考虑复制构造器：
- 提供初始容量initCapacity、负载因子loadFactor的构造器
  - 该构造器首先确保参数的有效性，且`initCapacity`不能大于`MAXIMUM_CAPACITY（2 ^ 30）`，然后初始化了`loadFactor`和`threshold`
  - **容量为0，但threshold均>0**，且`threshold`为$2^n$
    > 如果容量大于最大容量，则可以直接对容量赋值
  
- 提供初始容量initCapacity的构造器
  - 调用上一个构造器，但传入默认的负载因子（0.75）
  - **容量为0，但threshold>0**
- 无参构造器
  - **只是将loadFactor设为默认的0.75**
  - 注意：**使用无参构造器后，`capacity`与`threshold`均为0**

> 在上述三种构造器中，均没有为`table数组`分配内存空间，是等到==第一次调用`put()`方法时==，才分配的
> 因此，==三种构造器的容量均为0==

|构造器	|this.loadFactor|	this. threshold|	this.table|	this.size|
|:--:|:--:|:--:|:--:|:--:|
|HashMap(容量, 负载因子)	|loadFactor	|`tableSizeFor(initialCapacity)	`|null	|0|
|HashMap(容量)|	DEFAULT_LOAD_FACTOR	|`tableSizeFor(initialCapacity)`|	null|	0|
|HashMap( )	|DEFAULT_LOAD_FACTOR|	`0`|	null|	0|

如果使用的是传入同类Map的复制构造器，则`threShold`的值取决于`oldMap`的节点数：
```java
t = ((float)oldMap.size() / loadFactor) + 1.0
threShold = tableSizeFor(t); 
```
而其他参数与无参构造相同，只设置默认的`loadFactor`，因此`table`为null，容量为0
之后，遍历`oldMap`的节点，将其`put`入`newMap`中

## HashMap 中 key 的存储索引是怎么计算的？

1. 根据key的值计算出hashcode的值
2. 根据hashcode计算出hash值
3. 通过`hash &（length-1）`计算得到存储的位置。

> Object.hashCode()是一个native方法，即根据对象在内存中的地址，将其转换为一个整数，不同的Object可能有hashCode相同的情况，

```java
// jdk1.7
方法一：
static int hash(int h) {
    int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

    h ^= k.hashCode(); // 为第一步：取hashCode值
    h ^= (h >>> 20) ^ (h >>> 12); 
    return h ^ (h >>> 7) ^ (h >>> 4);
}
方法二：
//jdk1.7的源码，jdk1.8没有这个方法，但实现原理一样
static int indexFor(int h, int length) {  
     return h & (length-1);  //第三步：取模运算
}
```

```java
// jdk1.8
static final int hash(Object key) {   
     int h;
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    /* 
     h = key.hashCode() 为第一步：取hashCode值
     h ^ (h >>> 16)  为第二步：将hashCode的高16位参与运算
    */
}

// 代码2
int n = tab.length;
// 将(tab.length - 1) 与 hash值进行&运算
int index = (n - 1) & hash;
// 之后，根据index直到链表的头结点，到链表中/红黑树寻找
```

这里的 Hash 算法本质上就是三步：**取key的 hashCode 值、根据 hashcode 计算出hash值、通过取模计算下标**。
其中，JDK1.7和1.8的不同之处，就在于第二步。我们来看下详细过程，以JDK1.8为例，n为table的长度。

 ![](https://images-shf.oss-cn-beijing.aliyuncs.com/集合框架/HashMap-hash.png?versionId=CAEQHhiCgMDYkaCE.RciIDY3NWI0OGE1MDRkNzRjNTA4N2I0NTdhZDg3MGJkNzY3)

> **将hashCode的高16位参与计算的原因：**                        
> 在计算index时，需要计算`(n - 1) & hash`，这实际上是HashMap对取模操作的优化
> 取模运算为`hash mod tab.length`，如果`tab.length`为$2^N$，此时有$ x \mod  2^n = x \And (2^n - 1) $，这样一来，就将复杂的取模运算转化为了高效的与运算，实现了更高的效率，也因此，==table的长度应当为`2^n`==
> 当`n`，即`tab.length==16`时，`n-1`在二进制上为**4位全1，其他全0**，会导致`hash & (tab.length - 1)`的结果**仅由hashCode的低4位决定，高28位没有作用**，严重增加了冲突的概率。
> 所以通过将key的`hashCode()`右移16位的方式，在不增加开销的前提下，让hashCode的高16位也参与计算
> 
> 除此之外，JDK1.7中通过位运算扰动了4次，性能会差一些


## HashMap 的put方法流程？

简要流程如下：

1. 首先根据 key 的值计算 `hash` 值，找到该元素在数组中存储的下标；

2. 如果`Node数组`是空的，则调用 `resize` 进行初始化；

3. 如果没有哈希冲突，直接新建节点，放在对应的数组下标里；

4. 如果冲突了：
   1. 且 `key` 已经存在，就覆盖掉 `value`；
   2. 无重复`key`，若节点为红黑树，则将其插入到树中适当的位置
   3. 无重复`key`，节点为链表，判断该链表是否大于 8 ：
      - 如果**链表节点数大于 8 并且数组的容量大于 64**，则将这个结构转换为红黑树；
      - 否则，新建一个节点，插入链表尾部；若 key 存在，就覆盖掉 value。
5. 插入之后，判断当前是否需要扩容（**当前链表的节点数大于 8 ，且数组容量小于 64**）
   ![hashmap之put方法(JDK1.8)](http://blog-img.coolsen.cn/img/hashmap之put方法.jpg)

## HashMap的get方法的流程

简要流程如下：
1. 对table进行校验
2. 计算`hash & (tab.length - 1)`，找到table中对应的节点，判断是否直接返回
3. 判断是链表还是红黑树，根据hash和key，用对应的方法查找，知道当前节点的hash和key都与入参相同

## HashMap 的扩容方式？

HashMap 在容量超过**负载因子所定义的容量**之后，就会扩容。
Java 里的数组是无法自动扩容的，方法是**将 HashMap 的大小扩大为原来数组的两倍，并将原来的对象放入新的数组中**

扩容步骤：
1. 如果旧表的容量不为0，则需要判断旧表的容量是否达到最大容量，
   - 如果达到则无法扩容，直接返回旧表；
   - 若未达到最大容量，则令`newCap = 2*oldCap`，若`oldCap>=16`，则令`newThr = 2*oldThr`


2. 若为在**第一次调用`put`方法时，调用了`resize()`方法**，将新表的容量设置为旧表的阈值即可
> resize()方法对第一次put()的处理：
> - 若使用了带有初始容量参数的两个构造器，则令`newCap=oldThr`，并令`newThr = (int)((float)newCap * loadFactor)`
>   > `oldThr`为$2^n$
> - 若使用了无参构造器，则令`newCap=默认初始容量`，`newThr`计算同理

3. 利用新计算得到的`newCap`与`newThr`，创建新的`Node数组`
4. 对旧表中的每一个节点`e=oldTab[j]`，将其与旧表断开连接`oldTab[j]=null`，并将其移动到新表中去，
   1. 若`e.next==null`，说明该位置只有一个节点，直接放到新表中
      - 新表的索引与旧表类似：`e.hash & (newCap - 1)`，只是将`oldCap`替换为`newCap`
   2. 若`e`为链表节点，则进行普通的重hash分布。
      1. 分别创建`loHead`、`hiHead`，对应将原始链表划分得到的两条新链表；
      2. 根据`(e.hash & oldCap) == 0`的结构，判断将节点加入哪条链表
         > `(e.hash & oldCap) == 0 `，说明==扩容后的索引位置与旧表中的索引位置相同==
      3. 将`loHead`放置到`newTab[j]`，`hiHead`放置到`newTab[j + oldCap]`
   3. 若`e`为红黑树节点，使用红黑树的`split`方法，进行重hash分布


> **扩容后，节点重hash为什么只可能分布在`原索引位置`与`原索引+oldCap位置`**
> 扩容代码中，使用`e.hash & oldCap`确定节点分布的位置
> 一般来说，`newCap=oldCap*2`，而索引计算方式`index = (tab.length - 1) & hash`，**`newCap-1`相当于在`oldCap-1`的第一个`1`之前加了一个`1`**
> 如果两个节点在旧表的索引位置相同，则在新表的索引位置值**取决于多出来的这么一位`1`**，而 ==***oldCap恰好是除了这一位为1，其他位全0的***==，因此可以用`oldCap`对这两个位置进行划分。
> 如果`e.hash & oldCap == 0`，说明`e.hash`在多出来的这一位为0，则其在新表中的位置也将为旧表中的`原索引位置`，否则一定为`原索引+oldCap`（因为多了这一位1，恰好等于oldCap）


## 为什么 hash 值要与length-1相与？

- 把 hash 值对数组长度取模运算，模运算的消耗很大，没有位运算快。
- 当 length 总是 2 的n次方时，h& (length-1) 运算等价于对length取模，也就是 h%length，但是 & 比 % 具有更高的效率。

## HashMap数组的长度为什么是 2 的幂次方？

这样做效果上等同于取模，在速度、效率上比直接取模要快得多。
除此之外，2 的 N 次幂有助于**减少碰撞的几率**。如果 length 为2的幂次方，则 length-1 转化为二进制必定是11111……的形式，在与h的二进制与操作效率会非常的快，而且空间不浪费。我们来举个例子，看下图：

![map-幂次-1](http://blog-img.coolsen.cn/img/map-幂次-1.jpg)

当 length = 15 时，6 和 7 的结果一样，这样表示他们在 table 存储的位置是相同的，也就是产生了碰撞，6、7就会在一个位置形成链表，4和5的结果也是一样，这样就会导致查询速度降低。

如果我们进一步分析，还会发现空间浪费非常大，以 length=15 为例，在 1、3、5、7、9、11、13、15 这八处没有存放数据。因为hash值在与14（即 1110）进行&运算时，得到的结果最后一位永远都是0，即 0001、0011、0101、0111、1001、1011、1101、1111位置处是不可能存储数据的。

## HashMap如何得到2的n次幂

HashMap 构造函数允许用户传入的容量不是  2  的  n  次方，因为它可以自动地将传入的容量转换为  2  的  n 次方。会取大于或等于这个数的 且最近的2次幂作为 table 数组的初始容量，使用` tableSizeFor(int) `方法，如 tableSizeFor(10) = 16（2 的 4 次幂），tableSizeFor(20) = 32（2 的 5 次幂），也就是说 table 数组的长度总是 2 的次幂。JDK1.8 源码如下：

```java
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

解释：位或( | )
`int n = cap - 1;`　让`cap-1`再赋值给`n`的目的是：令找到的目标值大于或等于原值。
例如二进制1000，十进制数值为8。如果不对它减1而直接操作，将得到答案10000，即16。显然不是结果。减1后二进制为111，再进行操作则会得到原来的数值1000，即8。

## 一般用什么作为HashMap的key?

一般用`Integer`、`String` 这种不可变类作为 key，而且 String 最为常用。

- 因为字符串是不可变的，所以在它创建的时候 `hashcode` 就被缓存了，不需要重新计算。
- 因为获取对象的时候要用到 `equals()` 和 `hashCode()` 方法，那么键对象正确的重写这两个方法是非常重要的,这些类已经很规范的重写了 hashCode() 以及 equals() 方法。

## HashMap为什么线程不安全？

![](http://blog-img.coolsen.cn/img/HashMap为什么线程不安全.png)

* 多线程下扩容死循环。
  * JDK1.7中的 HashMap 使用**头插法**插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环。
  * 因此，JDK1.8使用**尾插法**插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。
* 多线程的put可能导致元素的丢失。
  * 多线程同时执行 put 操作，如果计算出来的**索引位置是相同的**，那会造成前一个 key 被后一个 key 覆盖，从而导致元素的丢失。
    > 此问题在JDK 1.7和 JDK 1.8 中都存在。
* put和get并发时，可能导致get为null。
  * 线程1执行`put`时，因为元素个数超出`threshold`而导致`rehash`，线程2此时执行get，有可能导致这个问题。
    > 此问题在JDK 1.7和 JDK 1.8 中都存在。

具体分析可见我的这篇文章：[面试官：HashMap 为什么线程不安全？](https://mp.weixin.qq.com/s?__biz=Mzg4MjUxMTI4NA==&mid=2247484436&idx=1&sn=eb677611e2ba1d10e3eb3ceb825bef02&chksm=cf54d8cff82351d9cb1c6ad49b6df8b7f0eaa7b965e3be5546b449e71ce1ffccf47ae68f7bf7&token=1920060057&lang=zh_CN#rd)


## HashMap 查找元素的时间复杂度为什么是 O(1)

在查询时，我们首先需要从Hash桶数组找到元素所在的桶，这是通过Key的hash值计算的，且数组是支持随机访问的，所以时间复杂度为O(1)

而在找到元素所在的桶后，存在三种情况：
1. 元素所在桶只有一个节点，直接返回，O(1)
2. 元素所在桶是一个链表，需要在链表中进行搜索，O(N)
3. 元素所在桶是一个红黑树，O(lgN)

然而，根据泊松分布，链表中的节点数超过8的概率极低，因此基本可以忽略，可以将在链表中查询的时间复杂度视作O(1)

## HashMap 的 key 如何做到唯一的

HashMap在put()时，会通过key的hashcode计算出hash值，并通过hash值判断应当将对应的数据放到哪个Hash桶中，并通过hash与key的equals()方法判断是否存在重复key，如果存在则覆盖

## HashMap 什么情况下会出现 CPU 100% 的问题？如何定位？

在JDK7中，出现环形链表会导致CPU100%的问题

出现这种问题，应该去查看是否在多线程环境中不恰当的进行put()

## hashmap 和 hashtable 的区别


|  项目 | HashMap | Hashtable |
|:--:|:--:|:--:|
| 父类 | AbstractMap | Dictionary |
| 线程安全 | 非线程安全 | 线程安全，通过synchronized实现 |
| 迭代器 | Iterator，是`fast-fail`的 | enumerator，不是`fast-fail`的 |
| contains方法 | 没有contains方法，只有ContainsKey与ContainsValue | 包含contains方法，效果同ContainsKey |
| Hash值的计算 | 支持对null的key的计算，通过hashcode与右移16位的hashcode进行异或，优化了只有低位参与运算的问题 | 直接将hashCode作为hash值 |
| 索引的计算 | 通过`index = (tab.length - 1) & hash`计算索引 | 将hashCode转换为正值后，直接与`tab.length`求模，作为hash值 |
| Null值 | key与value都允许为null | **key与value均不能为null** |
| 扩容方式 | 新容量为`2 * oldCap` | 新容量为`2 * oldCap + 1` |
| 解决Hash冲突的方式 | 链地址法，包含扩容、转换为红黑树的操作 | 只使用链表 |
| 默认容量 | 16 | 11 |

## HashMap与TreeMap的区别

|  项目 | HashMap | TreeMap |
|:--:|:--:|:--:|
| 线程安全 | 非线程安全 | 非线程安全 |
| 数据结构 | 数组+链表+红黑树 | 红黑树 |
| 用途 | 适用于快速定位元素 | 实现了SortMap接口，可以根据Key排序，使用Iterator遍历时，得到的记录是有序的 |
| Null值 | key与value都可以为null | ==key不能为null==，value可以为null |
| 节点定位 | 通过计算hash值，定位节点 | 在红黑树中利用比较算法定位 | 

## 扩容在JDK1.8中有什么不一样？

JDK1.8做了两处优化：

1. `resize` 之后，元素的位置在`原索引位置`，或`原索引位置+oldCap`。不需要像 JDK1.7 的实现那样重新计算hash ，只需要看看原来的 hash 值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引 + oldCap ”。这个设计非常的巧妙，省去了重新计算 hash 值的时间。

2. JDK1.7 中 `rehash` 的时候，旧链表迁移新链表的时候，如果在新表的数组索引位置相同，则链表元素会倒置（头插法）。JDK1.8 不会倒置，使用尾插法。

## 其他

### 还知道哪些hash算法？

Hash函数是指把一个大范围映射到一个小范围，目的往往是为了节省空间，使得数据容易保存。 比较出名的有`MurmurHash`、`MD4`、`MD5`等等。

### 用可变类当 HashMap 的 key 有什么问题?

可变类的hashcode 可能发生改变，导致 put 进去的值，无法 get 出。如下所示

```java
HashMap<List<String>, Object> changeMap = new HashMap<>();
List<String> list = new ArrayList<>();
list.add("hello");
Object objectValue = new Object();
changeMap.put(list, objectValue);
System.out.println(changeMap.get(list));
list.add("hello world");//hashcode发生了改变，导致后面找不到list了
System.out.println(changeMap.get(list));
```

输出值如下

```java
java.lang.Object@74a14482
null
```

## 多线程下扩容死循环

JDK1.7中的 HashMap 使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环。因此，JDK1.8使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。

下面看看多线程情况下， JDK1.7 扩容死循环问题的分析。

![](http://blog-img.coolsen.cn/img/resize1.png)

新建一个更大尺寸的hash表，然后把数据从老的hash表中迁移到新的hash表中。重点看下transfer方法：

![](http://blog-img.coolsen.cn/img/resize2.png)

假设HashMap初始化大小为2，hash算法就是用key mod 表的长度，在mod 2以后都冲突在table[1]这里了。负载因子是 1.5 (默认为 0.75 )，由公式` threshold=负载因子 *  hash表长度`可得，`threshold=1.5 * 2 =3`，size=3，而 size>=threshold 就要扩容，所以 hash表要 resize 成 4。

未resize前的table如下图：

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第10页.png)

正常的ReHash，得到的结果如下图所示：

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第9页.png)

我们来看看多线程下的ReHash，假设现在有两个线程同时进行，线程1和线程2，两个线程都会新建新的数组，下面是resize 的过程。

**Step1:**

![](http://blog-img.coolsen.cn/img/carbon.png)

假设 **线程1** 在执行到`Entry<K,V> next = e.next;`之后，cpu 时间片用完了，被调度挂起，这时**线程1的 e** 指向 节点A，**线程1的 next** 指向节点B。

**线程2**继续执行，

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第1页.png)

**Step2:**

线程 1 被调度回来执行。

- 先是执行 newTalbe[i] = e;
- 然后是e = next，导致了e指向了节点B，
- 而下一次循环的next = e.next导致了next指向了节点A。

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第2页.png)

**Step3:**

线程1 接着工作。**把节点B摘下来，放到newTable[i]的第一个，然后把e和next往下移**。

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第3页.png)

**Step4:** 出现环形链表

e.next = newTable[i] 导致A.next 指向了 节点B，此时的B.next 已经指向了节点A，出现**环形链表**。

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第4页.png)

如果get一个在此链表中不存在的key时，就会出现死循环了。如 get(11)时，就发生了死循环。

分析见get方法的源码：

![](http://blog-img.coolsen.cn/img/carbon1.png)

for循环中的`e = e.next`永远不会为空，那么，如果get一个在这个链表中不存在的key时，就会出现死循环了。

## 多线程的put可能导致元素的丢失

多线程同时执行 put 操作，如果计算出来的索引位置是相同的，那会造成**前一个 key 被后一个 key 覆盖**，从而导致元素的丢失。此问题在JDK 1.7和 JDK 1.8 中都存在。

## put和get并发时，可能导致get为null

线程1执行`put`时，因为元素个数超出`threshold`而导致`rehash`，线程2此时执行`get`，有可能导致这个问题。此问题在JDK 1.7和 JDK 1.8 中都存在。

我们来看下JDK 1.8 中 resize 方法的部分源码，重点看黄色部分：

![](http://blog-img.coolsen.cn/img/carbon3.png)

在代码`#1`位置，用新计算的容量new了一个新的hash表，`#2`将新创建的空hash表赋值给实例变量table。并在后续语句中，将旧表的数据移动到新表

注意**此时实例变量table是空的**，如果此时另一个线程执行get，就会get出null。

# ConcurrentHashMap

## ConcurrentHashMap 的实现原理是什么？ 

ConcurrentHashMap  在 JDK1.7 和 JDK1.8  的实现方式是不同的。

**先来看下JDK1.7**

JDK1.7中的ConcurrentHashMap  是由 `Segment` 数组结构和 `HashEntry` 数组结构组成，即ConcurrentHashMap 把**哈希桶切分成小数组（Segment ），每个小数组包含 n 个 HashEntry**。

> Segment 继承了 `ReentrantLock`，所以 Segment 是一种可重入锁，扮演锁的角色；HashEntry 用于存储键值对数据。

ConcurrentHashMap首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问，能够实现真正的并发访问。

**再来看下JDK1.8**

在数据结构上， JDK1.8  中的ConcurrentHashMap  选择了与 HashMap 相同的**数组+链表+红黑树**结构；
在锁的实现上，抛弃了原有的 `Segment 分段锁`，采用` CAS + synchronized `实现==更低粒度的锁==。

将锁的级别控制在了更细粒度的**哈希桶元素级别**，也就是说**只需要锁住这个链表头结点（红黑树的根节点）**，不会影响其他的哈希桶元素的读写，大大提高了并发度。

![](http://blog-img.coolsen.cn/img/ConcurrentHashMap-jdk1.8.png)

## ConcurrentHashMap初始化的逻辑是怎样的

首先，ConcurrentHashMap中有几个重要的变量：
```java
// 默认为0
// 初始化时，为-1
// 扩容时，为-(1+扩容线程数)
// 初始化或扩容完成后，为 下一次扩容的阈值大小
// sizeCtl = 1.5* initialCapacity + 1
private transient volatile int sizeCtl;

// 扩容时，如果某个bin迁移完毕，用 ForwardingNode 作为 旧table bin的头结点，表示当前bin已经迁移完成了
static final class ForwardingNode<K, V> extends Node<K, V> {}

// 用于compute 与 computeIfAbsent时，用来占位，计算完成后替换为普通Node
static final class ReservationNode<K, V> extends Node<K, V>{}

// 扩容索引，表示已经分配给扩容线程的table数组索引位置。主要用来协调多个线程，并发安全地找到需要迁移的hash桶
private transient volatile int transferIndex;

//扩容线程每次最少需要扩容16个桶
private static final int MIN_TRANSFER_STRIDE = 16;
```

## ConcurrentHashMap的sizeCtl是什么？

sizeCtl是一个volatile的变量，通过CAS进行修改，用于控制数组初始化与扩容操作。
sizeCtl的取值与意义如下：
- -1：正在初始化；
- -N：表示有N-1个线程正在进行扩容操作
- 0：未初始化
- 正数：下一次扩容的大小

> sizeCtl = 1.5* initialCapacity + 1




## ConcurrentHashMap的节点的Hash值有什么特殊含义吗

令节点`e`的Hash值为`eh`
- `eh == -1`：表示节点为forwardingNode，表示Map正在扩容，且该节点已完成扩容
- `eh == -2`：表示节点为treebin，即红黑树的节点
- `eh > 0`：说明是正常的链表节点


## ConcurrentHashMap  的 put 方法执行逻辑是什么？

**JDK1.7：**

首先，会尝试获取锁，如果获取失败，利用自旋获取锁；如果自旋重试的次数超过 64 次，则改为阻塞获取锁。

获取到锁后：

1. 将当前 Segment 中的 table 通过 `key 的 hashcode` 定位到 `HashEntry`。
2. 遍历该 `HashEntry`，如果不为空则判断传入的 key 和当前遍历的 key 是否相等：
   - 相等则覆盖旧的 value。
   - 不相等则需要新建一个 `HashEntry` 并加入到 `Segment` 中，同时会先判断是否需要扩容。
3. 释放 Segment 的锁。

**再来看JDK1.8**

大致可以分为以下步骤：

1. 根据 key 计算出 hash值。
2. 判断是否需要进行初始化。
3. 定位到 `Node`，拿到首节点 f，判断首节点 f：
   * 如果为  `null`  ，则通过`cas`的方式尝试添加。
   * 如果为 `f.hash = MOVED = -1` ，说明**其他线程在扩容，参与一起扩容**。
   * 如果都不满足 ，利用`synchronized` 锁住 f 节点，判断是链表还是红黑树，遍历插入。
4. 当在链表长度达到8的时候，数组扩容或者将链表转换为红黑树。
5. 通过`CAS`增加节点数

源码分析可看这篇文章：[面试 ConcurrentHashMap ，看这一篇就够了！](https://mp.weixin.qq.com/s?__biz=Mzg4MjUxMTI4NA==&mid=2247484715&idx=1&sn=f5c3ad8e66122531a1c77efcb9cb50b7&chksm=cf54d9f0f82350e637a51fa8bc679f6197d15e4c9703aac971150bfcc5437e867c3bcf3f409c&token=1920060057&lang=zh_CN#rd)



## 介绍一下initTable方法

在第一次`put()`时，table数组尚未初始化，需要调用initTable方法进行初始化。

在`initTable`方法中：
1. 在没有竞争的情况(`sizeCtl >= 0`)下，将`sizeCtl`通过`CAS`修改为`-1`
2. 然后根据`默认容量DEFAULT_CAPACITY`或`sizeCtl`修改前的值进行`Node[]`的初始化
3. 并令`sizeCtl = newCap - (newCAP >>> 2)`， 即`0.75 *newCap`

> sizeCtl类似ThreShold，上述`sizeCtl = 0.75 * newCap`即扩容阈值的计算



## ConcurrentHashMap的扩容流程是怎样的？

https://blog.csdn.net/zzu_seu/article/details/106698150
[扩容讲解，很不错！](https://www.sohu.com/a/320372210_120176035)

在put的流程中，在定位`Node`后，会通过`CAS`或`synchronized`的方式，将节点加入其中，并进行`treeifyBin`等操作，转换为红黑树。
在完成了上述操作后，需要通过`CAS`增加节点数，并判断是否进行扩容。
而JDK8采用的是多线程扩容的机制，在整个扩容过程中，通过`CAS`设置`sizeCtl`，`transferIndex`等变量协调多个线程进行并发扩容，对table数组中的Node进行迁移

1. 在完成节点插入的工作后，调用`addCount(1, binCount)`，尝试使用`CAS`，将`BASECOUNT`+1
   - binCount：链表节点数 
2. 首先判断Map的`CounterCell[]`是否已经被创建，被创建说明存在竞争，而没创建则进行初始化
   > `CounterCell[]`也是延迟初始化的 
   > `CounterCell[]`采用了`LongAdder`的思想，**设置了多个累加单元，减少cas冲突，提高性能；可以通过`sumCount()`累加每个单元，作为最终的元素数量。**
   > > 使用`CounterCell[]`数组，而不是一个**成员变量**来统计元素数量的原因，是为了保证并发环境的下的安全性，如果是使用成员变量，势必会使用加锁或自旋来保证，影响性能
3. 从`CounterCell[]`数组中随机取出一个`CounterCell`，如果为空或对该`CounterCell`的CAS增加失败，则调用`fullAddCount`
   > 取`CounterCell`的方法为`as[ThreadLocalRandom.getProbe() & m]`，其中，`ThreadLocalRandom.getProbe()`可以保证每个线程取出来的随机值都不是相同的
4. `fullAddCount(x,...)`主要是用来初始化CounterCell，来记录元素个数，里面包含扩容，初始化等操作
   > 在fullAddCount中，通过**新建值为x的CounterCell**，或**随机选取一个CounterCell，通过CAS使其值增加**，使得在后续的`sumCount()`中，能够感知到元素数量的增加
5. **通过sumCount()计算CHM中的元素数量，若元素个数大于`阈值sizeCtl`，且table不为空，则进行扩容**
   1. 若`sizeCtl < 0`，说明与其他线程正在扩容，则**帮助扩容**
      - 以下5个条件，只要有一个为true，则不协助扩容：
        a. `sc >>> RESIZE_STAMP_SHIFT != rs`，表示比较`sizeCtl`的高`32 - RESIZE_STAMP_SHIFT位`是否和`resizeStamp`相等
        b. `sizeCtl == resizeStamp + MAX_RESIZERS`，表示扩容线程已经达到最大扩容线程数 
        c. `sizeCtl == resizeStamp + 1`，表示扩容结束
        d. `nextTalbe == null`，表示扩容已结束
        e. `transferIndex==0`，即所有node均已分配
   2. 若无其他线程在扩容，则`sizeCtl一定是正数(表示扩容后的容量)`，则通过CAS，将`sizeCtl`设置为`resizeStamp << RESIZE_STAMP_SHIFT) + 2`，即**一个负数(表示有线程在扩容了)**，并通过`transfer`方法进行扩容
6. 重新使用`sumCount()`计数，判断是否需要进行下一轮扩容


ConcurrentHashMap的扩容是并发扩容的，它把Node数组当作多个线程之间共享的任务队列，并给多个线程分配各自负责的区域，把旧table中的链表一个个复制到新table中。
> 分配时是从后向前分配，假设table原先长度是64，有四个线程，则第一个到达的线程负责48-63这块内容的复制，第二个线程负责32-47，第三个负责16-31，第四个负责0-15。
> 同理，复制也是从后向前复制的，假如一个线程完成了下标为63的节点的扩容，才会进入62的扩容，一直向前，直到完成
> 在当前线程完成后，还会判断其他线程是否完成任务，若为完成，会去帮助其他线程进行扩容

那么线程之间相互帮忙会导致混乱吗？
1. 线程扩容之前，会将`sizeCtl`通过`CAS`设置为`(rs << RESIZE_STAMP_SHIFT) + 2`，即将`sizeCtl`由正数变为负数，表示有线程在扩容了
2. 扩容已完成的位置，会将节点替换为`ForwardingNode`，即Hash值为-1的Node，表示当前节点已完成
3. 对于正在执行的复制任务的位置，线程会直接锁住该桶

复制结束的标志：
每个线程参加复制前会将标记位`sizeCtl加1`，同样退出时会将`sizeCtl减1`，这样每个线程退出时，只要检查一下`sizeCtl是否等于进入前的状态`就知道是否全都退出了。
最后一个退出的线程，则将就table的地址更新指向新table的地址，这样后面的操作就是新table的操作了。


`transfer()`方法的主要流程如下

1. 在扩容之前，`transferIndex` 在数组的最右边。此时有一个线程发现已经到达扩容阈值，准备开始扩容。
2. 首先，分配**每个线程需要处理的桶数**，即`stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < 16`
   - 如果是多核CPU，则为每个核平均分配需要处理的桶数，最小为16
   - 如果是单核，则直接分配所有桶
3. 首先，如果`nextTab`没有初始化，则使用`2 * oldCap`对其进行初始化 
4. 在for循环中进行任务分配，用一个变量advance标志是否需要对当前线程进行任务分配（即划分当前线程需要处理的桶的范围），并通过CAS修改`transferIndex`，令其为`Math.max(transferIndex - stride, 0)`，自旋CAS可以保证划分范围时的线程安全。
5. 若当前线程完成了扩容(`i<0 || i>=n || i+n>=nextn`)
   1. 如果**所有遍历任务已完成**，则更新`table数组`与扩容的阈值`sizeCtl = (n << 1) - (n >>> 1)`，即`sizeCtl = 0.75 * newCap`；
   2. 并不是所有线程都完成了，则仅仅通过CAS令`sizeCtl - 1`，即令其低16位-1，表明**当前线程退出扩容任务**。
      - 此时，如果`(sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT`，则说明还有线程在扩容，则直接返回
      - 否则说明在当前线程完成后，所有线程已完成，设置`finishing`标志位，并令遍历标志`i=n`，即从最右端开始重新检查一遍（在下轮循环中遍历，但下一轮循环不会走到这里，会全部检查一遍）
6. 未完成扩容，则继续处理，包含一下几种情况：
   1. 当前节点为`null`，则将链表头替换为`ForwardingNode`，在下一轮循环中变为`情况2`；
   2. 当前节点的`hash值`为`-1`，说明已经是`ForwardingNode了`，修改`advance`，表示需要**在下一轮循环中调整当前线程处理的桶的范围**，即在下一轮循环处理新一批的桶；
   3. 若均不满足，则说明需要进行迁移：
      1. 首先对节点用`synchronized`加锁。由于n是2的幂次方，与HashMap一样，根据`hash & tab.length`的结果将所有结点分为两部分，分别初始化高位节点与低位节点
      2. 先遍历一次整张链表，找到最后一段颜色相同的链表，并将其开头的节点当做低位or高位的起始节点，另一位的起始节点为null；
      3. 再次遍历，将每一个节点以==头插法==的方式，连接入低位与高位链表中
      4. 完成遍历后，通过`setTabAt`方法，将其放入`nextTable`中的合适的位置（低位与高位）
> 树节点的处理方式类似，就省略了

## 使用tabAt方法，而不是直接使用tab[i]的原因

该方法获取对象中offset偏移地址对应的对象field的值。实际上这段代码的含义等价于tab[i]

在执行`tabAt()`前，首先要执行如下的代码，为`tabAt()`的计算做前期工作。

```java
Class<?> ak = Node[].class;
// 获取数组的起始位置：16：12Byte为对象头+4Byte的数组长度
ABASE = U.arrayBaseOffset(ak);
// 获取数组的数据的大小，比如byte类型的数据为1，int类型的为4，long为8
int scale = U.arrayIndexScale(ak);
if ((scale & (scale - 1)) != 0)
    throw new Error("data type scale not a power of two");
// 获得指定二进制数字前导0个数
ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
```

这个方法实际调用的是`Unsafe.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE)`，其中，`((long)i << ASHIFT) + ABASE`是表示在数组开头地址的基础上，寻找指定数组在内存中i位置的数据。

数组的寻址方式为`a[i]_address = base_address + i * data_type_size`
当保存的数据为int，`ASHIFT == 2`，则上式为`i << 2 + ABASE == i * 4 + ABASE`，与寻址公式一致。

而通过Unsafe类的`getObjectVolatile`方法，可以直接操作数组内存地址来实现访问，且使用了`volatile`字段，保证了其他线程的修改对当前线程的读的可见性

> 为啥不令table数组为volatile，再从里面直接用tab[i]取

因为虽然table数组本身是增加了volatile属性，但是`volatile的数组`**只针对数组的引用具有volatile的语义，而不是它的元素**。
所以如果有其他线程对这个数组的元素进行写操作，那么当前线程来读的时候不一定能读到最新的值。


## 介绍一下resizeStamp

`resizeStamp`用来生成一个和扩容有关的扩容戳，主要用于修改`sizeCtl`

```java
static final int resizeStamp(int n) { 
	return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1)); 
} 
```
`Integer.numberOfLeadingZeros` 这个方法是返回**无符号整数n最高位非0位前面的0的个数**
例如16的二进制为：`0000 0000 0000 0000 0000 0000 0001 0000`
那么`Integer.numberOfLeadingZeros`的返回值就是27，即`0000 0000 0000 0000 0000 0000 0001 1011`

假设`RESIZE_STAMP_BITS = 16`，则`1 << (RESIZE_STAMP_BITS - 1)`的值为`0000 0000 0000 0000 1000 0000 0000 0000`，即高16位为0，第16位为1，低15位存放当前容量n。

==因此，resizeStamp的值为`0000 0000 0000 0000 1000 0000 0001 1011`==

> 为什么要计算有n有多少个前导0？
> ConcurrentHashMap的容量n必定是2的幂次方，所以不同的容量n前面0的个数必然不同，这样可以保证是**在原容量为n的情况下进行扩容**。

而当第一个线程尝试扩容时，`sizeCtl >= 0`，表示没有线程在扩容，会执行：
```java
U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2) 
// private static int RESIZE_STAMP_BITS = 16;
// private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;
```
rs左移16位，相当于原本的二进制低位变成了高位：
`1000 0000 0001 1011 0000 0000 0000 0000`
这样一来，`resizeStamp << RESIZE_STAMP_SHIFT`就变为负数

然后再+2，得到：
`1000 0000 0001 1011 0000 0000 0000 0010`

将`sizeCtl`通过`CAS`设置为这个值，就确保`sizeCtl`为负值，且低`16`位表示的为当前并行扩容线程数+1

其中，==高16位代表扩容的标记，将用于与`resizeStamp`进行比较，进行校验；低16位代表并行扩容的线程数+1==
> 低RESIZE_STAMP_SHIFT位：并行扩容线程数+1

这样的设计的好处是：
1. 可以保证每次扩容都生成唯一的生成戳，每次新的扩容，都有一个不同的`sizeCtl`，用于唯一地标识这次扩容过程
2. 将扩容标记与扩容线程数放到一个变量`sizeCtl`里，有助于并发编程的实施

具体使用场景：

```java
addCount()中：
if(sizeCtl < 0){
    //sizeCtl的高16位不等于resizeStamp，说明sizeCtl变化了，出现异常
    //      resizeStamp = n的前导0数量 | (1 << (RESIZE_STAMP_BITS - 1));
    //                  =  0000 0000 0000 0000 1000 0000 0001 1011
    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || 
    // sizeCtl == resizeStamp + 1，表示扩容结束了
    //  此时sizeCtl < 0，表示并行扩容的线程数，默认第一个线程设置sc = (rs << 16) + 2，当第一个线程扩容结束，将sc - 1，则sc = rs + 1
        sc == rs + 1 ||
    // sizeCtl == resizeStamp + MAX_RESIZERS，说明线程数达到了最大值
        sc == rs + MAX_RESIZERS ||
        (nt = nextTable) == null ||
        transferIndex <= 0)
            break;
    //帮助扩容
    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
        transfer(tab, nt);
}
//扩容前，将sizeCtl变为(rs << RESIZE_STAMP_SHIFT) + 2，即正数变负数
else if (U.compareAndSwapInt(this, SIZECTL, sc, (rs << RESIZE_STAMP_SHIFT) + 2))
    transfer(tab, null);
```

> 注：当sizeCtl为正时，表示下次扩容需要的容量，为负数时，`-N：表示有N-1个线程正在进行扩容操作`
> 而resizeStamp的高RESIZE_STAMP_BITS位表示扩容容量，低RESIZE_STAMP_SHIFT位表示扩容线程数+1
> 因此，若(sc >>> RESIZE_STAMP_SHIFT) != rs，就表明
> 若sc == rs + MAX_RESIZERS，就说明


## ConcurrentHashMap的get方法逻辑是怎样的

**JDK1.7：**
首先，根据 key 计算出 hash 值定位到具体的 Segment ，再根据 hash 值获取定位 HashEntry 对象，并对 HashEntry 对象进行链表遍历，找到对应元素。

> 由于 HashEntry 涉及到的共享变量都使用 volatile 修饰，volatile 可以保证内存可见性，所以每次获取时都是最新值。

**JDK1.8：**

大致可以分为以下步骤：

1. 根据 key 计算出 hash 值，判断数组是否为空；
2. 如果是首节点，就直接返回；
3. 如果是红黑树结构，就从红黑树里面查询；
4. 如果是链表结构，循环遍历判断。

> 可见，ConcurrentHashMap的get()方法是不需要加锁的

## ConcurrentHashMap 的 get 方法是否要加锁，为什么？

**get 方法不需要加锁**。因为 Node 的元素 `val` 和指针 `next` 是用 `volatile`修饰的，在多线程环境下，线程A修改结点的val或者新增节点的时候是对线程B可见的。

这也是它比其他并发集合比如 Hashtable、用 Collections.synchronizedMap()包装的 HashMap 效率高的原因之一。


```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    //可以看到这些都用了volatile修饰
    volatile V val;
    volatile Node<K,V> next;
}
```

## get方法不需要加锁与volatile修饰的哈希桶有关吗？

没有关系。
哈希桶`table`用`volatile`修饰主要是**保证在数组扩容的时候保证可见性**。

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

    // 存放数据的桶
    transient volatile HashEntry<K,V>[] table;
```



## ConcurrentHashMap  不支持 key 或者 value 为  null  的原因？

我们先来说`value` 为什么不能为 `null` ：
因为`ConcurrentHashMap `是用于多线程的 ，如果`map.get(key)`得到了 null ，==无法判断是key的value是 null ，还是没有找到对应的key而返回null== ，这就有了二义性。

而用于单线程状态的`HashMap`却可以用`containsKey(key)` 去判断到底是否包含了这个 null 。

> 我们用**反证法**来推理：
> 
> 假设ConcurrentHashMap 允许存放值为 `null` 的`value`，这时有A、B两个线程，线程A调用ConcurrentHashMap .get(key)方法，返回为 null ，我们不知道这个 null 是没有映射的 null ，还是存的值就是 null 。
> 
> 假设此时，返回为 null 的真实情况是没有找到对应的key。那么，我们可以用`ConcurrentHashMap.containsKey(key)`来验证我们的假设是否成立，我们期望的结果是返回false。
> 
> 但是在我们调用`ConcurrentHashMap.get(key)`方法之后，`containsKey()`方法之前，线程B执行了`ConcurrentHashMap.put(key, null)`的操作。那么我们调用containsKey方法返回的就是true了，这就与我们的假设的真实情况不符合了，这就有了二义性。

ConcurrentHashMap 中的`key`不能为 null 的原因：在多线程环境下，如果允许`key`为null，其实不会有什么问题，但为了设计的一致性，还是不允许`key`为null


[这道面试题我真不知道面试官想要的回答是什么](https://mp.weixin.qq.com/s?__biz=MzIxNTQ4MzE1NA==&mid=2247484354&idx=1&sn=80c92881b47a586eba9c633eb78d36f6&chksm=9796d5bfa0e15ca9713ff9dc6e100593e0ef06ed7ea2f60cb984e492c4ed438d2405fbb2c4ff&scene=21#wechat_redirect)





## ConcurrentHashMap 的并发度是多少？

在JDK1.7中，并发度默认是`16`，这个值可以在构造函数中设置。如果自己设置了并发度，ConcurrentHashMap 会使用**大于等于该值的最小的2的幂指数**作为实际并发度，也就是比如你设置的值是17，那么实际并发度是32。

> 并发度可以理解为程序运行时能够**同时更新 ConccurentHashMap且不产生锁竞争的最大线程数**
> 在JDK1.7中，反映为`Segment[]`数组的长度，即分段锁的个数
> 在JDK1.8中，反映为`Node数组`的大小。

如果并发度设置的过小，会带来严重的锁竞争问题；
如果并发度设置的过大，原本位于同一个Segment内的访问会扩散到不同的Segment中，CPU cache命中率会下降，从而引起程序性能下降。

## ConcurrentHashMap 迭代器是强一致性还是弱一致性？

与HashMap迭代器是强一致性不同，**ConcurrentHashMap 迭代器是弱一致性**。

ConcurrentHashMap 的迭代器创建后，就会按照哈希表结构遍历每个元素，但在遍历过程中，内部元素可能会发生变化，
- 如果变化发生在已遍历过的部分，迭代器就不会反映出来
- 如果变化发生在未遍历过的部分，迭代器就会发现并反映出来

这就是弱一致性。

这样迭代器线程可以使用原来老的数据，而写线程也可以并发的完成改变，更重要的，这保证了多个线程并发执行的连续性和扩展性，是性能提升的关键。想要深入了解的小伙伴，可以看这篇文章[为什么ConcurrentHashMap 是弱一致的](http://ifeve.com/ConcurrentHashMap-weakly-consistent/)

## JDK1.7与JDK1.8 中ConcurrentHashMap 的区别？

|  项目 | 1.7 | 1.8 |
|:--:|:--:|:--:|
| 线程安全机制 | 采用Segment的分段锁机制 | 采用CAS+Synchronized |
| 锁的粒度 | 对Segment加锁 | 对每个Node数组的元素加锁 |
| 存储优化 | 无 | 利用红黑树加快搜索 |
| 查询时间复杂度 | 遍历链表O(N) | 红黑树查找O(logN) | 

## ConcurrentHashMap 和Hashtable的效率哪个更高？为什么？

ConcurrentHashMap 的效率要高于Hashtable
- `Hashtable`在许多关键方法上使用synchronized加锁，即给**整个哈希表加锁**来实现线程安全。
- `ConcurrentHashMap` 的锁粒度更低，在JDK1.7中采用分段锁实现线程安全，在JDK1.8 中采用`CAS+Synchronized`实现线程安全。


## JDK1.7 与 JDK1.8 中ConcurrentHashMap 的区别

- 数据结构：取消了 Segment 分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。
- 保证线程安全机制：JDK1.7 采用 Segment 的分段锁机制实现线程安全，其中 Segment 继承自 ReentrantLock 。JDK1.8 采用`CAS+synchronized `保证线程安全。
- 锁的粒度：JDK1.7 是对需要进行数据操作的 Segment 加锁，JDK1.8 调整为对每个数组元素加锁（Node）。
- 链表转化为红黑树：定位节点的 hash 算法简化会带来弊端，hash 冲突加剧，因此在链表节点数量大于 8（且数据总量大于等于 64）时，会将链表转化为红黑树进行存储。
- 查询时间复杂度：从 JDK1.7的遍历链表O(n)， JDK1.8 变成遍历红黑树O(logN)。


## 多线程下安全的操作 map还有其他方法吗？

还可以使用`Collections.synchronizedMap`方法，对方法进行加同步锁。

![](http://blog-img.coolsen.cn/img/ConcurrentHashMap-code8.png)

如果传入的是 HashMap 对象，其实也是对 HashMap 做的方法做了一层包装，里面使用对象锁来保证多线程场景下，线程安全，本质也是对 HashMap 进行全表锁。**在竞争激烈的多线程环境下性能依然也非常差，不推荐使用！**

## 分段锁依据什么分段

分段锁实际上是一个`Segment[]`数组，`Segment`是`ReentrantLock`的子类，通过将CHM的`Node[]`数组进行均分，确定每个锁负责的区域，例如有16个Segment，则每个锁保护所有Hash桶的`1/16`，第N个桶由第`N mod 16`个锁保护

## 当CHM扩容了之后分段锁数量会改变吗？如果有15个线程同时对一个segment里边的用户信息进行修改会发生什么？

不会改变，只是每个Segment负责的区域更多了，Segment[]数组一旦初始化好了，就不能再改变

会有一个线程获得锁，进行修改，而另外14个线程进入Segment的EntryList，陷入阻塞，等待被`notify()`

##  JDK1.8  中为什么使用内置锁 synchronized替换 可重入锁 ReentrantLock？

* 在 JDK1.6 中，对 synchronized 锁的实现引入了大量的优化，并且 synchronized 有多种锁状态，会从无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁一步步转换。
* 减少内存开销 。假设使用可重入锁来获得同步支持，那么**每个节点都需要通过继承 AQS 来获得同步支持**。但并不是每个节点都需要获得同步支持的，只有链表的头节点（红黑树的根节点）需要同步，这无疑带来了巨大内存浪费。 
## 利用hash实现分布式存储有什么弊端



## 一致性哈希问题






# 其他乱七八糟


## 有哪些线程安全的集合，讲一讲原理



## 说一下Hashtable的锁机制 ?

`Hashtable`是使用`Synchronized`来实现线程安全的，给整个哈希表加了一把大锁，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞等待需要的锁被释放，在竞争激烈的多线程场景中性能就会非常差！

![](http://blog-img.coolsen.cn/img/ConcurrentHashMap-hashtable.png)

## HashSet 和 HashMap 区别?

![](http://blog-img.coolsen.cn/img/image-20210403193010949.png)

补充HashSet的实现：`HashSet`的底层其实就是`HashMap`，只不过我们**HashSet是实现了Set接口并且把数据作为K值，而V值一直使用一个相同的虚值来保存**。

如源码所示：

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;// 调用HashMap的put方法,PRESENT是一个至始至终都相同的虚值
}
```

由于HashMap的K值本身就不允许重复，并且在HashMap中如果K/V相同时，会用新的V覆盖掉旧的V，然后返回旧的V，那么在HashSet中执行这一句话始终会返回一个false，导致插入失败，这样就保证了数据的不可重复性。

## Collection框架中实现比较要怎么做？

第一种，实体类实现Comparable接口，并实现 compareTo(T t) 方法，称为内部比较器。

第二种，创建一个外部比较器，这个外部比较器要实现Comparator接口的 compare(T t1, T t2)方法。

## Iterator 和 ListIterator 有什么区别？

* 遍历。使用Iterator，可以遍历所有集合，如Map，List，Set；但只能在向前方向上遍历集合中的元素。

使用ListIterator，只能遍历List实现的对象，但可以向前和向后遍历集合中的元素。

* 添加元素。Iterator无法向集合中添加元素；而，ListIteror可以向集合添加元素。

* 修改元素。Iterator无法修改集合中的元素；而，ListIterator可以使用set()修改集合中的元素。

* 索引。Iterator无法获取集合中元素的索引；而，使用ListIterator，可以获取集合中元素的索引。

## 讲一讲快速失败(fail-fast)和安全失败(fail-safe)

**快速失败（fail—fast）** 

* 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。
* 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个        modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。      

* 注意：这里异常的抛出条件是检测到 modCount！=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。      

* 场景：java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改），比如HashMap、ArrayList 这些集合类。      

**安全失败（fail—safe）**  

* 采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。      

* 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。      

*  缺点：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。      

* 场景：java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改，比如：ConcurrentHashMap。




## HashSet 如何去重？