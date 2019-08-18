#  
# 快速导航
* [1.HashMap的源码，实现原理，JDK8中对HashMap做了怎样的优化。](#_1hashmap的源码，实现原理，jdk8中对hashmap做了怎样的优化。)
* [2.HaspMap扩容是怎样扩容的，为什么都是2的N次幂的大小。](#_2haspmap扩容是怎样扩容的，为什么都是2的n次幂的大小。)
* [3.HashMap，HashTable，ConcurrentHashMap的区别。](#_3hashmap，hashtable，concurrenthashmap的区别。)
* [4.极高并发下hashtable和concurrenthashmap哪个性能更好，为什么，如何实现的。](#_4极高并发下hashtable和concurrenthashmap哪个性能更好，为什么，如何实现的。)
* [5.hashmap在高并发下如果没有处理线程安全会有怎样的安全隐患，具体表现是什么？](#_5hashmap在高并发下如果没有处理线程安全会有怎样的安全隐患，具体表现是什么？)
* [6.java中四种修饰符的限制范围。](#_6java中四种修饰符的限制范围。)
* [7.object-类中的方法](#_7object-类中的方法)
* [8.接口和抽象类的区别，注意jdk8的接口可以有实现。](#_8接口和抽象类的区别，注意jdk8的接口可以有实现。)
* [9.动态代理的两种方式，以及区别。](#_9动态代理的两种方式，以及区别。)


# 1.HashMap的源码，实现原理，JDK8中对HashMap做了怎样的优化。

[深入理解](https://www.cnblogs.com/wskwbog/p/10907457.html)

* #### HashMap的源码，实现原理：
HashMap的链表元素对应的是一个静态内部类Entry，Entry主要包含key，value，next三个元素

主要有put和get方法，

put的原理是，通过hash%Entry.length计算index，此时记作Entry[index]=该元素。如果index相同

就是新入的元素放置到Entry[index]，原先的元素记作Entry[index].next

get就比较简单了，先遍历数组，再遍历链表元素。

==null key总是放在Entry数组的第一个元素==

解决hash冲突的方法：链地址法

再散列rehash的过程：确定容量超过目前哈希表的容量，重新调整table 的容量大小，当超过容量的最大值时，取 Integer.Maxvalue



#### 数组和链表的区别：
1 链表在内存是是不连续的，数组是连续的

2 数组内存分配是一次性的全部分配，链表不是

3 数组是同级别的，链表是一个连接一个，查找效率链表不如数组

4 链表的增删比数组方便

5 链表的扩展潜力大于数组 

#### JDK8中对HashMap做了怎样的优化：
1.生成一个entry初始容量16的数组+链表结构,使用容量大于0.75f时,自动扩容2^n;

扩容时，利用 2 的次幂数值的二进制特点，既省去重新计算 hash 的时间，又把之前冲突的节点散列到了其他位置

2.当链表长度大于8时,转化为红黑树结构.在冲突比较严重的情况下，将 get 操作的时间复杂从 O(n) 降为了 O(logn)

3.优化 hash 算法只进行一次位移操作



# 2.HaspMap扩容是怎样扩容的，为什么都是2的N次幂的大小。

默认情况下，初始容量是 16，负载因子是 0.75f，threshold 是 12，也就是说，插入 12 个键值对就会扩容

在扩容时，会扩大到原来的两倍，因为使用的是2的次幂扩展，那么元素的位置要么保持不变，要么在原位置上偏移2的次幂。

根据子链表的长度做以下处理:
* 长度小于 6，返回一个不包含 TreeNode 的普通链表
* 否则，把子链表转为红黑树


# 3.HashMap，HashTable，ConcurrentHashMap的区别。

[深入理解](https://www.cnblogs.com/heyonggang/p/9112731.html)

### HashTable
* 底层数组+链表实现，无论key还是value都不能为null，线程安全，实现线程安全的方式是在修改数据时锁住整个HashTable，效率低，ConcurrentHashMap做了相关优化
* 初始size为11，扩容：newsize = olesize*2+1
* 计算index的方法：index = (hash & 0x7FFFFFFF) % tab.length

### HashMap
* 底层数组+链表实现，可以存储null键和null值，线程不安全
* 初始size为16，扩容：newsize = oldsize*2，size一定为2的n次幂
* 扩容针对整个Map，每次扩容时，原来数组中的元素依次重新计算存放位置，并重新插入
* 插入元素后才判断该不该扩容，有可能无效扩容（插入后如果扩容，如果没有再次插入，就会产生无效扩容）
* 当Map中元素总数超过Entry数组的75%，触发扩容操作，为了减少链表长度，元素分配更均匀
* 计算index方法：index = hash & (tab.length – 1)
 

**HashMap的初始值还要考虑加载因子:**
* **哈希冲突**：若干Key的哈希值按数组大小取模后，如果落在同一个数组下标上，将组成一条Entry链，对Key的查找需要遍历Entry链上的每个元素执行equals()比较。
* **加载因子**：为了降低哈希冲突的概率，默认当HashMap中的键值对达到数组大小的75%时，即会触发扩容。因此，如果预估容量是100，即需要设定100/0.75＝134的数组大小。
* **空间换时间**：如果希望加快Key查找的时间，还可以进一步降低加载因子，加大初始大小，以降低哈希冲突的概率。

HashMap和Hashtable都是用hash算法来决定其元素的存储，因此HashMap和Hashtable的hash表包含如下属性：

* 容量（capacity）：hash表中桶的数量
* 初始化容量（initial capacity）：创建hash表时桶的数量，HashMap允许在构造器中指定初始化容量
* 尺寸（size）：当前hash表中记录的数量
* 负载因子（load factor）：负载因子等于“size/capacity”。负载因子为0，表示空的hash表，0.5表示半满的散列表，依此类推。轻负载的散列表具有冲突少、适宜插入与查询的特点（但是使用Iterator迭代元素时比较慢）

除此之外，hash表里还有一个“负载极限”，“负载极限”是一个0～1的数值，“负载极限”决定了hash表的最大填满程度。当hash表中的负载因子达到指定的“负载极限”时，hash表会自动成倍地增加容量（桶的数量），并将原有的对象重新分配，放入新的桶内，这称为rehashing。

HashMap和Hashtable的构造器允许指定一个负载极限，HashMap和Hashtable默认的“负载极限”为0.75，这表明当该hash表的3/4已经被填满时，hash表会发生rehashing。

“负载极限”的默认值（0.75）是时间和空间成本上的一种折中：

* 较高的“负载极限”可以降低hash表所占用的内存空间，但会增加查询数据的时间开销，而查询是最频繁的操作（HashMap的get()与put()方法都要用到查询）
* 较低的“负载极限”会提高查询数据的性能，但会增加hash表所占用的内存开销
程序猿可以根据实际情况来调整“负载极限”值。

### ConcurrentHashMap
* 底层采用分段的数组+链表实现，线程安全
* 通过把整个Map分为N个Segment，可以提供相同的线程安全，但是效率提升N倍，默认提升16倍。(读操作不加锁，由于HashEntry的value变量是 volatile的，也能保证读取到最新的值。)
* Hashtable的synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术
* 有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁
* 扩容：段内扩容（段内元素超过该段对应Entry数组长度的75%触发扩容，不会对整个Map进行扩容），插入前检测需不需要扩容，有效避免无效扩容
 

Hashtable和HashMap都实现了Map接口，但是Hashtable的实现是基于Dictionary抽象类的。Java5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。

HashMap基于哈希思想，实现对数据的读写。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计算hashcode，然后找到bucket位置来存储值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞时，对象将会储存在链表的下一个节点中。HashMap在每个链表节点中储存键值对对象。当两个不同的键对象的hashcode相同时，它们会储存在同一个bucket位置的链表中，可通过键对象的equals()方法来找到键值对。如果链表大小超过阈值（TREEIFY_THRESHOLD,8），链表就会被改造为树形结构。

在HashMap中，null可以作为键，这样的键只有一个，但可以有一个或多个键所对应的值为null。当get()方法返回null值时，即可以表示HashMap中没有该key，也可以表示该key所对应的value为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个key，应该用containsKey()方法来判断。而在Hashtable中，无论是key还是value都不能为null。

Hashtable是线程安全的，它的方法是同步的，可以直接用在多线程环境中。而HashMap则不是线程安全的，在多线程环境中，需要手动实现同步机制。

Hashtable与HashMap另一个区别是HashMap的迭代器（Iterator）是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。

先看一下简单的类图：
![image](http://dl2.iteye.com/upload/attachment/0087/3840/1a20c1a7-c422-374b-9acf-8c33479586cb.jpg)

从类图中可以看出来在存储结构中ConcurrentHashMap比HashMap多出了一个类Segment，而Segment是一个可重入锁。

ConcurrentHashMap是使用了锁分段技术来保证线程安全的。

**锁分段技术**：首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。 

ConcurrentHashMap提供了与Hashtable和SynchronizedMap不同的锁机制。Hashtable中采用的锁机制是一次锁住整个hash表，从而在同一时刻只能由一个线程对其进行操作；而ConcurrentHashMap中则是一次锁住一个桶。

ConcurrentHashMap默认将hash表分为16个桶，诸如get、put、remove等常用操作只锁住当前需要用到的桶。这样，原来只能一个线程进入，现在却能同时有16个写线程执行，并发性能的提升是显而易见的。


# 4.极高并发下HashTable和ConcurrentHashMap哪个性能更好，为什么，如何实现的。

  多线程下 ConcurrentHashMap 性能更好.
  * HashTable使用一把锁处理并发问题，当有多个线程访问时，需要多个线程竞争一把锁，导致阻塞
  * ConcurrentHashMap则使用分段，相当于把一个HashMap分成多个，然后每个部分分配一把锁，这样就可以支持多线程访问


# 5.hashmap在高并发下如果没有处理线程安全会有怎样的安全隐患，具体表现是什么？

  * 多线程put时可能会导致get无限循环，具体表现为CPU使用率100%
    * 原因：在向HashMap put元素时，会检查HashMap的容量是否足够，如果不足，则会新建一个比原来容量大两倍的Hash表，然后把数组从老的Hash表中迁移到新的Hash表中，迁移的过程就是一个rehash()的过程，多个线程同时操作就有可能会形成循环链表，所以在使用get()时，就会出现Infinite Loop的情况。
    ```
    // tranfer()片段
    // 这是在resize()中调用的方法,resize()就是HashMap扩容的方法     
    for (int j = 0; j < src.length; j++) {
        Entry e = src[j];
        if (e != null) {
        src[j] = null;
        do {
        Entry next = e.next;  //假设线程1停留在这里就挂起了,线程2登场
        int i = indexFor(e.hash, newCapacity);
        e.next = newTable[i];
        newTable[i] = e;
        e = next;
        } while (e != null);
      }
    }
    ```
  * 多线程put时可能导致元素丢失
    * 原因：当多个线程同时执行addEntry(hash,key ,value,i)时，如果产生哈希碰撞，导致两个线程得到同样的bucketIndex去存储，就可能会发生元素覆盖丢失的情况。
    ```
    void addEntry(int hash, K key, V value, int bucketIndex) {
        //多个线程操作数组的同一个位置
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
        if (size++ >= threshold)
            resize(2 * table.length);
    }
    ```

# 6.java中四种修饰符的限制范围。
  
  访问权限 | 本类 | 本包 | 不同包子类 | 不同包非子类
  ---|---|---|---|---
  public | √ | √ | √ | √
  protected | √ | √ | √	 
  default | √ | √	 	 
  private | √	 	 	 


# 7.Object 类中的方法
  
  [深入了解](https://blog.csdn.net/u012156116/article/details/79550497)

  ![image](https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=3213190560,3045064029&fm=173&app=49&f=JPEG?w=542&h=373&s=6342FC1A171C55CC487D04DA000080B3)



# 8.接口和抽象类的区别，注意JDK8的接口可以有实现。

  * 首先是相同的地方：
    * 1. 接口和抽象类都能定义方法和属性。
  
    * 2. 接口和抽象类都是看作是一种特殊的类。大部分的时候，定义的方法要子类来实现

    * 3. 抽象类和接口都可以不含有抽象方法。接口没有方法就可以作为一个标志。比如可序列化的接口Serializable，没有方法的接口称为空接口。没有抽象方法的抽象类，小编不知道有什么作用，总之是可以通过编译的。

    * 4. 抽象类和接口都不能创建对象。

    * 5. 抽象类和接口都能利用多态性原理来使用抽象类引用指向子类对象。

    * 6. 继承和实现接口或抽象类的子类必须实现接口或抽象类的所有的方法，抽象类若有没有实现的方法就继续作为抽象类，要加abstract修饰。若接口的子类没有实现的方法，也要变为抽象类。

  * 下面是接口和抽象类的不同点：
    * 1. 接口能够多实现，而抽象类只能单独被继承，其本质就是，一个类能继承多个接口，而只能继承一个抽象类。

    * 2. 方法上，抽象类的方法可以用abstract 和public或者protect修饰。而接口默认为public abttact 修饰。

    * 3. 抽象类的方法可以有需要子类实现的抽象方法，也可以有具体的方法。而接口在老版本的jdk中，只能有抽象方法，但是Java8版本的接口中，接口可以带有默认方法。

    * 4. 属性上，抽象类可以用各种各样的修饰符修饰。而接口的属性是默认的public static final

    * 5. 抽象类中可以含有静态代码块和静态方法，而接口不能含有静态方法和静态代码块。

    * 6. 抽象类可以含有构造方法，接口不能含有构造方法。

    * 7. 设计层面上，抽象类表示的是子类“是不是”属于某一类的子类，接口则表示“有没有”特性“能不能”做这种事。如飞机和鸟都能飞，但是他们在设计上实现一个Fly接口，实现fly()方法。远比两个类继承飞行物抽象类好得多。因为，飞机和鸟有太多的属性不一样。

    * 8. 设计层面上，另外一点，抽象类可以是一个模板，因为可以自己带集体方法，所以要加一个实现类都能有的方法，直接在抽象类中写出并实现就好，接口在以前的版本则不行。新版本Java8才有默认方法。

    * 9. 既然说到Java 8 那么就来说明，Java8中的接口中的默认方法是可以被多重继承的。而抽象类不行。

    * 10. 另外，接口只能继承接口。而抽象类可以继承普通的类，也能继承接口和抽象类。


# 9.动态代理的两种方式，以及区别。
  
  [深入理解](https://blog.csdn.net/weixin_36759405/article/details/82770422)
  * 1.JDK代理使用的是反射机制实现aop的动态代理，CGLIB代理使用字节码处理框架asm，通过修改字节码生成子类。所以jdk动态代理的方式创建代理对象效率较高，执行效率较低，cglib创建效率较低，执行效率高；
  * 2.JDK动态代理机制是委托机制，具体说动态实现接口类，在动态生成的实现类里面委托hanlder去调用原始实现类方法，CGLIB则使用的继承机制，具体说被代理类和代理类是继承关系，所以代理类是可以赋值给被代理类的，如果被代理类有接口，那么代理类也可以赋值给接口。


# 10.Java序列化的方式。

  [Java序列化](https://www.jianshu.com/p/35ddcf46aa2d)

  [Java序列化的几种方式以及序列化的作用](https://www.cnblogs.com/zhengshao/p/8882613.html)


# 11.传值和传引用的区别，Java是怎么样的，有没有传值引用。

  * 传值：传递的是值的副本。方法中对副本的修改，不会影响到调用方。
  * 传引用：传递的是引用的副本，共用一个内存，会影响到调用方。此时，形参和实参指向同一个内存地址。对引用副本本身（对象地址）的修改，如设置为null，重新指向其他对象，不会影响到调用方。
  * 基本类型（byte,short,int,long,double,float,char,boolean）为传值；对象类型（Object，数组，容器）为传引用；String、Integer、Double等immutable类型因为类的变量设为final属性，无法被修改，只能重新赋值或生成对象。当Integer作为方法参数传递时，对其赋值会导致原有的引用被指向了方法内的栈地址，失去原有的的地址指向，所以对赋值后的Integer做任何操作都不会影响原有值。


# 12.一个ArrayList在循环过程中删除，会不会出问题，为什么。

  [link](https://www.cnblogs.com/androidsuperman/p/9012320.html)



# 13.@transactional注解在什么情况下会失效，为什么。

  [link](https://www.cnblogs.com/hunrry/p/9183209.html)


