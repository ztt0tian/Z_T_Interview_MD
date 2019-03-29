# HashMap

参考：[](https://www.cnblogs.com/chengxiao/p/6059914.html)

HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对

HashMap可以接受null键值和值，Entry是HashMap中的一个静态内部类。

HashMap的整体结构如下

![img](https://images2015.cnblogs.com/blog/1024555/201611/1024555-20161113235348670-746615111.png)

**简单来说，HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好。**

### 我们能否让HashMap同步？

HashMap可以通过下面的语句进行同步：
Map m = Collections.synchronizeMap(hashMap);

### HashMap的get()方法的工作原理

HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于HashMap自己设定的一个hash()函数，以上hash函数计算出的值，通过indexFor进一步处理【h & (length-1)】来获取实际的存储位置即要找的entry数组索引来找到bucket位置来储存的Entry对象。

### 当两个对象的hashcode相同会发生什么？

因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中。

一般会使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。

### 如果两个键的hashcode相同，你如何获取值对象？

找到数组位置之后，会调用keys.equals()方法去找到存储着相同哈希码（经过hash()函数处理过的key哈希码）的键值对的链表中正确的节点，最终找到要找的值对象

### 重新调整HashMap大小存在什么问题吗

默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

### 为何HashMap的数组长度一定是2的次幂

 hashMap的数组长度一定保持2的次幂，比如16的二进制表示为 10000，那么length-1就是15，二进制为01111，同理扩容后的数组长度为32，二进制表示为100000，length-1为31，二进制表示为011111。从下图可以我们也能看到这样会保证低位全为1，而扩容后只有一位差异，也就是多出了最左位的1，这样在通过 h&(length-1)的时候，只要h对应的最左边的那一个差异位为0，就能保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)，个人理解.

h&(length-1)这个算法实际就是取模，hash%length，计算机中直接求余效率不如位移运算，源码中做了优化hash&(length-1)，hash%length==hash&(length-1)的前提是length是2的n次方；

### 重写equals方法需同时重写hashCode方法

get方法的实现相对简单，key(hashcode)-->hash-->indexFor-->最终索引位置，找到对应位置table[i]，再查看是否有链表，遍历链表，通过key的equals方法比对查找对应的记录。要注意的是，有人觉得上面在定位到数组位置之后然后遍历链表的时候，e.hash == hash这个判断没必要，仅通过equals判断就可以。其实不然，试想一下，如果传入的key对象重写了equals方法却没有重写hashCode，而恰巧此对象定位到这个数组位置，如果仅仅用equals判断可能是相等的，但其hashCode和当前对象不一致，这种情况，根据Object的hashCode的约定，不能返回当前对象，而应该返回null，后面的例子会做出进一步解释。

### JDK1.8版本的HashMap

参考<https://blog.csdn.net/v123411739/article/details/78996181>

HashMap在JDK1.8及以后的版本中引入了红黑树结构，若桶中链表元素个数大于等于8时，链表转换成树结构；若桶中链表元素个数小于等于6时，树结构还原成链表。因为红黑树的平均查找长度是log(n)，长度为8的时候，平均查找长度为3，如果继续使用链表，平均查找长度为8/2=4，这才有转换为树的必要。链表长度如果是小于等于6，6/2=3，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短。

还有选择6和8，中间有个差值7可以有效防止链表和树频繁转换。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低.

在JDK1.8的实现中，还优化了高位运算的算法，将hashCode的高16位与hashCode进行异或运算，主要是为了在table的length较小的时候，让高位也参与运算，并且不会有太大的开销。

get()方法

先对table进行校验，校验是否为空，length是否大于0
使用table.length - 1和hash值进行位与运算，得出在table上的索引位置，将该索引位置的节点赋值给first节点，校验该索引位置是否为空
检查first节点的hash值和key是否和入参的一样，如果一样则first即为目标节点，直接返回first节点
如果first的next节点不为空则继续遍历
如果first节点为TreeNode，则调用getTreeNode方法（见下文代码块1）查找目标节点
如果first节点不为TreeNode，则调用普通的遍历链表方法查找目标节点

如果查找不到目标节点则返回空

put()方法

校验table是否为空或者length等于0，如果是则调用resize方法（见下文resize方法）进行初始化
通过hash值计算索引位置，将该索引位置的头节点赋值给p节点，如果该索引位置节点为空则使用传入的参数新增一个节点并放在该索引位置
判断p节点的key和hash值是否跟传入的相等，如果相等, 则p节点即为要查找的目标节点，将p节点赋值给e节点
如果p节点不是目标节点，则判断p节点是否为TreeNode，如果是则调用红黑树的putTreeVal方法（见下文代码块4）查找目标节点
走到这代表p节点为普通链表节点，则调用普通的链表方法进行查找，并定义变量binCount来统计该链表的节点数
如果p的next节点为空时，则代表找不到目标节点，则新增一个节点并插入链表尾部，并校验节点数是否超过8个，如果超过则调用treeifyBin方法（见下文代码块6）将链表节点转为红黑树节点
如果遍历的e节点存在hash值和key值都与传入的相同，则e节点即为目标节点，跳出循环
如果e节点不为空，则代表目标节点存在，使用传入的value覆盖该节点的value，并返回oldValue如果插入节点后节点数超过阈值，则调用resize方法（见下文resize方法）进行扩容

Jdk 1.8以前，导致死循环的主要原因是扩容后，节点的顺序会反掉。

即使此时有多个线程并发扩容，也不会出现死循环的情况。当然，这仍然改变不了HashMap仍是非并发安全，在并发下，还是要使用ConcurrentHashMap来代替

### 扩容后，节点重hash为什么只可能分布在原索引位置与原索引+oldCap位置？

扩容代码中，使用e节点的hash值跟oldCap进行位与运算，以此决定将节点分布到原索引位置或者原索引+oldCap位置上，这是为什么了？假设老表的容量为16，即oldCap=16，则新表容量为16*2=32，假设节点1的hash值为0000 0000 0000 0000 0000 1111 0000 1010，节点2的hash值为0000 0000 0000 0000 0000 1111 0001 1010，则节点1和节点2在老表的索引位置计算如下图计算1，由于老表的长度限制，节点1和节点2的索引位置只取决于节点hash值的最后4位。再看计算2，计算2为新表的索引计算，可以知道如果两个节点在老表的索引位置相同，则新表的索引位置只取决于节点hash值倒数第5位的值，而此位置的值刚好为老表的容量值16，此时节点在新表的索引位置只有两种情况：原索引位置和原索引+oldCap位置（在此例中即为10和10+16=26）。由于结果只取决于节点hash值的倒数第5位，而此位置的值刚好为老表的容量值16，因此此时新表的索引位置的计算可以替换为计算3，直接使用节点的hash值与老表的容量16进行位于运算，如果结果为0则该节点在新表的索引位置为原索引位置，否则该节点在新表的索引位置为原索引+oldCap位置。

![img](https://img-blog.csdn.net/20180107234931776?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdjEyMzQxMTczOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### HashMap和Hashtable的区别：

HashMap允许key和value为null，Hashtable不允许。
HashMap的默认初始容量为16，Hashtable为11。
HashMap的扩容为原来的2倍，Hashtable的扩容为原来的2倍加1。
HashMap是非线程安全的，Hashtable是线程安全的。
HashMap的hash值重新计算过，Hashtable直接使用hashCode。
HashMap去掉了Hashtable中的contains方法。

HashMap继承自AbstractMap类，Hashtable继承自Dictionary类。

### 1.8版本HashMap总结：

HashMap的底层是个Node数组（Node<K,V>[] table），在数组的具体索引位置，如果存在多个节点，则可能是以链表或红黑树的形式存在。
增加、删除、查找键值对时，定位到哈希桶数组的位置是很关键的一步，源码中是通过下面3个操作来完成这一步：

​	1）拿到key的hashCode值；

​	2）将hashCode的高位参与运算，重新计算hash值；

​	3）将计算出来的hash值与(table.length - 1)进行&运算。

HashMap的默认初始容量（capacity）是16，capacity必须为2的幂次方；默认负载因子（load factor）是0.75；实际能存放的节点个数（threshold，即触发扩容的阈值）= capacity * load factor。
HashMap在触发扩容后，阈值会变为原来的2倍，并且会进行重hash，重hash后索引位置index的节点的新分布位置最多只有两个：原索引位置或原索引+oldCap位置。例如capacity为16，索引位置5的节点扩容后，只可能分布在新报索引位置5和索引位置21（5+16）。

导致HashMap扩容后，同一个索引位置的节点重hash最多分布在两个位置的根本原因是：

1）table的长度始终为2的n次方；

2）索引位置的计算方法为“(table.length - 1) & hash”。HashMap扩容是一个比较耗时的操作，定义HashMap时尽量给个接近的初始容量值。

HashMap有threshold属性和loadFactor属性，**但是没有capacity属性**。初始化时，如果传了初始化容量值，该值是存在threshold变量，并且Node数组是在第一次put时才会进行初始化，初始化时会将此时的threshold值作为新表的capacity值，然后用capacity和loadFactor计算新表的真正threshold值。
当同一个索引位置的节点在增加后达到**9**个时，会触发链表节点（Node）转红黑树节点（TreeNode，间接继承Node），转成红黑树节点后，其实链表的结构还存在，通过next属性维持。**链表节点转红黑树节点**的具体方法为源码中的**treeifyBin(Node<K,V>[] tab, int hash)**方法。当同一个索引位置的节点在移除后达到**6**个时，并且该索引位置的节点为红黑树节点，会触发红黑树节点转链表节点。

**红黑树节点转链表节点**的具体方法为源码中的**untreeify(HashMap<K,V> map)**方法。
HashMap在JDK1.8之后不再有死循环的问题，**JDK1.8之前存在死循环的根本原因是在扩容后同一索引位置的节点顺序会反掉**。HashMap是非线程安全的，在并发场景下使用ConcurrentHashMap来代替。