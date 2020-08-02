# 面向对象三大特性  😏

**封装**

**继承**

如果多个类的某个部分的功能相同，那么可以抽象出一个类出来，把他们的相同部分都放到父类里，让他们都继承这个类。

实现

如果多个类处理的目标是一样的，但是处理的方法方式不同，那么就定义一个接口，也就是一个标准，让他们的实现这个接口，各自实现自己具体的处理方法来处理那个目标

**多态**



# RESTFUL-API设计规范  😏

是什么？
restful其实就是一套编写接口的协议，协议规定如何编写以及如何设置返回值、状态码等信息
（本身不是一门技术，只是一种规范一种架构设计而已）

怎么用？
restful: 给用户一个url，根据method不同在后端做不同的处理，比如：post 创建数据、get获取数据、put和patch修改数据、delete删除数据。
（感觉平常工作的时候还是用第一种比较多）
例子：
在Restful之前的操作：
http://127.0.0.1/user/query/1 GET  根据用户id查询用户数据
http://127.0.0.1/user/save POST 新增用户
http://127.0.0.1/user/update POST 修改用户信息
http://127.0.0.1/user/delete GET/POST 删除用户信息

RESTful用法：
http://127.0.0.1/user/1 GET  根据用户id查询用户数据
http://127.0.0.1/user  POST 新增用户
http://127.0.0.1/user  PUT 修改用户信息
http://127.0.0.1/user  DELETE 删除用户信息



# equals/hashCode  😏

## 为什么重写equal的时候要重写hashCode()方法

**equals方法和hashCode方法都是Object类中的方法，我们来看看他们的源码**

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

```java
public native int hashCode();
```

可知，equals方法在其内部是调用了"=="，所以说在不重写equals方法的情况下，equals方法是比较两个对象是否具有相同的引用，即是否指向了同一个内存地址。

而hashCode是一个本地方法，他返回的是这个对象的内存地址。

**hashCode的通用规定：**

- 在应用程序的执行期间，只要对象的equals方法的比较操作所用到的信息没有被修改，那么对同一个对象的多次调用，hashCode方法都必须始终返回同一个值。在一个应用程序与另一个应用程序的执行过程中，执行hashCode方法所返回的值可以不一致。
- 如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中的hashCode方法都必须产生同样的整数结果
- 如果两个对象根据equals(Object)方法比较是不相等的，那么调用这两个对象中的hashCode方法，则不一定要求hashCode方法必须产生不用的结果。但是程序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表的性能。

由上面三条规定可知，如果重写了equals方法而没有重写hashCode方法的话，就违反了第二条规定。***相等的对象必须拥有相等的hash code。\***

接下来，我用一个程序来演示一下不重写hashCode方法所带来的严重后果：

```java
public class Test {

    public static void main(String[] args) {
        Person person1 = new Person("TUCJVXCB");
        Person person2 = new Person("TUCJVXCB");


        Map<Person, Integer> hashMap = new HashMap<>();
        hashMap.put(person1, 1);


        System.out.println(person1.equals(person2));
        System.out.println(hashMap.containsKey(person2));
    }

    static class Person {
        private String name;

        public Person(String name) {
            this.name = name;
        }

        @Override
        public boolean equals(Object obj) {
            if (obj instanceof Person) {
                Person person = (Person) obj;

                return name.equals(person.name);
            }
            return false;
        }
    }
}
```

以下是输出结果

```
true
false
```

对于第一个输出true我们很容易知道，因为我们重写了equals方法，只要两个对象的name属性相同就会返回ture。但是为什么第二个为什么输出的是false呢？就是因为我们没有重写hashCode方法。所以我们得到一个结论：**如果一个类重写了equals方法但是没有重写hashCode方法，那么该类无法结合所有基于散列的集合（HashMap，HashSet）一起正常运作。**

------

# HashMap  😏

## **HashMap概述**

HashMap概述： HashMap是基于哈希表的Map接口的非同步实现。此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。

HashMap的数据结构： 在Java编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

HashMap 基于 Hash 算法实现的

当我们往Hashmap中put元素时，利用key的hashCode重新hash计算出当前对象的元素在数组中的下标
存储时，如果出现hash值相同的key，此时有两种情况。(1)如果key相同，则覆盖原始值；(2)如果key不同（出现冲突），则将当前的key-value放入链表中
获取时，直接找到hash值对应的下标，在进一步判断key是否相同，从而找到对应值。
理解了以上过程就不难明白HashMap是如何解决hash冲突的问题，核心就是使用了数组的存储方式，然后将冲突的key的对象放入链表中，一旦发现冲突就在链表中做进一步的对比。
需要注意Jdk 1.8中对HashMap的实现做了优化，当链表中的节点数据超过八个之后，该链表会转为红黑树来提高查询效率，从原来的O(n)到O(logn)

## **HashMap扩容**

①.在jdk1.8中，resize方法是在hashmap中的键值对大于阀值时或者初始化时，就调用resize方法进行扩容；

②.每次扩展的时候，都是扩展2倍；

③.扩展后Node对象的位置要么在原位置，要么移动到原偏移量两倍的位置。

在putVal()中，我们看到在这个函数里面使用到了2次resize()方法，resize()方法表示的在进行第一次初始化时会对其进行扩容，或者当该数组的实际大小大于其临界值值(第一次为12),这个时候在扩容的同时也会伴随的桶上面的元素进行重新分发，这也是JDK1.8版本的一个优化的地方，在1.7中，扩容之后需要重新去计算其Hash值，根据Hash值对其进行分发，但在1.8版本中，则是根据在同一个桶的位置中进行判断(e.hash & oldCap)是否为0，重新进行hash分配后，该元素的位置要么停留在原始位置，要么移动到原始位置+增加的数组大小这个位置上

## HashMap线程不安全(jdk1.7/1.8)

javadoc中关于hashMap的一段描述如下：

> 此实现不是同步的。如果多个线程同时访问一个哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须 保持外部同步。（结构上的修改是指添加或删除一个或多个映射关系的任何操作；仅改变与实例已经包含的键关联的值不是结构上的修改。）这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的非同步访问，如下所示：
>
> Map m = Collections.synchronizedMap(new HashMap(...));
> 

## 

`HashMap`的线程不安全主要体现在下面两个方面：
1.在JDK1.7中，当并发执行扩容操作时会造成环形链和数据丢失的情况。
2.在JDK1.8中，在并发执行put操作时会发生数据覆盖的情况。



**1.多线程调用put方法(addEntry)**

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
	Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
        if (size++ >= threshold)
            resize(2 * table.length);
    }
```

在hashmap做put操作的时候会调用到以上的方法。现在假如A线程和B线程同时对同一个数组位置调用addEntry，两个线程会同时得到现在的头结点，然后A写入新的头结点之后，B也写入新的头结点，那B的写入操作就会覆盖A的写入操作造成A的写入操作丢失

**2.多线程删除键值对(removeEntryForKey)**

```java
final Entry<K,V> removeEntryForKey(Object key) {
        int hash = (key == null) ? 0 : hash(key.hashCode());
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;
 
        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }
 
        return e;
    }
```

当多个线程同时操作同一个数组位置的时候，也都会先取得现在状态下该位置存储的头结点，然后各自去进行计算操作，之后再把结果写会到该数组位置去，其实写回的时候可能其他的线程已经就把这个位置给修改过了，就会覆盖其他线程的修改

**3.扩容(resize)**

在此分别对jdk1.7与jdk1.8扩容进行详解

**JDK1.7**

```java
void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
 
        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable);
        table = newTable;
        threshold = (int)(newCapacity * loadFactor);
    }
```

一般我们声明HashMap时，使用的都是默认的构造方法：HashMap<K,V>，看了代码你会发现，它还有其它的构造方法：

```java
HashMap(int initialCapacity, float loadFactor)
```

其中参数initialCapacity为初始容量，loadFactor为加载因子，扩容就是在put加入元素的个数超过initialCapacity * loadFactor的时候就会将内部Entry数组大小扩大至原来的2倍，然后将数组元素按照新的数组大小重新计算索引，放在新的数组中，同时修改每个节点的链表关系（主要是next和节点在链表中的位置）

假设这里有两个线程同时执行了put()操作，并进入了transfer()环节：





**JDK1.8**

根据上面JDK1.7出现的问题，在JDK1.8中已经得到了很好的解决，如果你去阅读1.8的源码会发现找不到`transfer`函数，因为JDK1.8直接在`resize`函数中完成了数据迁移。另外说一句，JDK1.8在进行元素插入时使用的是尾插法。

为什么说JDK1.8会出现**数据覆盖**的情况呢，我们来看一下下面这段JDK1.8中的put操作代码

```java
 /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        /** 其中第六行代码是判断是否出现hash碰撞，假设两个线程A、B都在进行put操作，并且hash函数计算出的插入下标是相同的，当线程A执行完第六行代码后由于时间片耗尽导致被挂起，而线程B得到时间片后在该下标处插入了元素，完成了正常的插入，然后线程A获得时间片，由于之前已经进行了hash碰撞的判断，所有此时不会再进行判断，而是直接进行插入，这就导致了线程B插入的数据被线程A覆盖了，从而线程不安全 */
        
        // (此为第6行代码)
        // 如果没有hash碰撞则直接插入元素
        if ((p = tab[i = (n - 1) & hash]) == null) 
            tab[i] = newNode(hash, key, value, null);
        
        
        else {
            Node<K,V> e; K k;
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
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        
        // (此为第38行代码)
        /**我们这样想，还是线程A、B，这两个线程同时进行put操作时，假设当前HashMap的zise大小为10，当线程A执行到第38行代码时，从主内存中获得size的值为10后准备进行+1操作，但是由于时间片耗尽只好让出CPU，线程B快乐的拿到CPU还是从主内存中拿到size的值10进行+1操作，完成了put操作并将size=11写回主内存，然后线程A再次拿到CPU并继续执行(此时size的值仍为10)，当执行完put操作后，还是将size=11写回内存，此时，线程A、B都执行了一次put操作，但是size的值只增加了1，所有说还是由于数据覆盖又导致了线程不安全。*/
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```







**多线程下HashMap扩容**







# HashTable  😏

## HashTable概述

HashTable在竞争激烈的并发环境下





# ConcurrentHashMap  😏

## ConcurrentHashMap概述

concurrentHashMap是线程安全且高效的HashMap

## 为什么使用concurrentHashMap

**1.并发编程中使用HashMap可能导致程序死循环(HashMap部分中有作解释)**

**2.使用HashTable效率又非常低下(HashTable部分中有作解释)**

基于以上两个原因，便有了ConcurrentHashMap的登场机会

(3) ConcurrentHashMap的锁分段技术可以有效提升并发访问率



# ArrayList  😏



# LinkedList  😏





# ConcurrentLinkedQueue 😏











# 2.经常被BB到的

## Collection/Collections

Collections是个java.util下的类，它包含有各种有关集合操作的静态方法。 

Collection是个java.util下的接口，它是各种集合结构的父接口。 

## String/StringBuilder/StringBuffer

String
String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）
在 Java 8 中，String 内部使用 char 数组存储数据。

不可变的好处：
1. 可以缓存 hash 值
因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。
不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

2. String Pool（字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定）的需要
如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。
只有 String 是不可变的，才可能使用 String Pool

3. 安全性
String 经常作为参数，String 不可变性可以保证参数不可变。
例如在作为网络连接参数的情况下如果 String 是可变的，
那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，
而实际情况却不一定是。

4. 线程安全
String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

1. 可变性

String 不可变
StringBuffer 和 StringBuilder 可变

2. 线程安全

String 不可变，因此是线程安全的
StringBuilder 不是线程安全的
StringBuffer 是线程安全的，内部使用 synchronized 进行同步

## HashMap/ConcurrentHashMap/HashTable/Collections的synchronizedMap

HashMap线程不安全1:resize / 2:fail-fast (Fail-safe和iterator迭代器相关。如果某个集合对象创建了Iterator或者ListIterator，然后其它的线程试图“结构上”更改集合对象，将会抛出ConcurrentModificationException异常。但其它线程可以通过set()方法更改集合对象是允许的，因为这并没有从“结构上”更改集合。但是假如已经从结构上进行了更改，再调用set()方法，将会抛出IllegalArgumentException异常。结构上的更改指的是删除或者插入一个元素，这样会影响到map的结构。

解决办法：可以使用Collections的synchronizedMap方法构造一个同步的map，或者直接使用线程安全的ConcurrentHashMap来保证不会出现fail-fast策略。)

**HashMap和ConcurrentHashMap对比：**

- HashMap非线程安全、ConcurrentHashMap线程安全
- HashMap允许Key与Value为空，ConcurrentHashMap不允许
- HashMap不允许通过迭代器遍历的同时修改，ConcurrentHashMap允许。并且更新可见

 **HashMap和HashTable的对比：**

（1）HashMap是非线程安全的，HashTable是线程安全的。

（2）HashMap的键和值都允许有null存在，而HashTable则都不行。

（3）因为线程安全、哈希效率的问题，HashMap效率比HashTable的要高。

**HashTable和ConcurrentHashMap对比：**

HashTable里使用的是synchronized关键字，这其实是对对象加锁，锁住的都是对象整体，当Hashtable的大小增加到一定的时候，性能会急剧下降，因为迭代时需要被锁定很长的时间。ConcurrentHashMap相对于HashTable的syn关键字锁的粒度更精细了一些，并发性能更好。

 问题：在put的时候是放在链表头部还是尾部？

jdk1.7之前是放在链表头部在jdk1.8之后是放在尾部。

concurrentHashMap->JDK1.8的实现已经抛弃了Segment分段锁机制，利用CAS+Synchronized来保证并发更新的安全。数据结构采用：数组+链表+红黑树。



## HashMap/TreeMap/LinkedHashMap

HashMap：数组方式存储key/value，线程非安全，允许null作为key和value，key不可以重复，value允许重复，不保证元素迭代顺序是按照插入时的顺序，key的hash值是先计算key的hashcode值，然后再进行计算，每次容量扩容会重新计算所以key的hash值，会消耗资源，要求key必须重写equals和hashcode方法

默认初始容量16，加载因子0.75，扩容为旧容量乘2，查找元素快，如果key一样则比较value，如果value不一样，则按照链表结构存储value，就是一个key后面有多个value；

TreeMap：基于红黑二叉树的NavigableMap的实现，线程非安全，不允许null，key不可以重复，value允许重复，存入TreeMap的元素应当实现Comparable接口或者实现Comparator接口，会按照排序后的顺序迭代元素，两个相比较的key不得抛出classCastException。主要用于存入元素的时候对元素进行自动排序，迭代输出的时候就按排序顺序输出

### 1. newNode方法

首先：LinkedHashMap重写了newNode()方法，通过此方法保证了插入的顺序性。

```java
/**
 * 使用LinkedHashMap中内部类Entry
 */
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p = new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}
/**
 * 将新创建的节点p作为尾结点tail，
 * 当然如果存储的第一个节点，那么它即是head节点，也是tail节点，此时节点p的before和after都为null
 * 否则，建立与上一次尾结点的链表关系，将当前尾节点p的前一个节点（before）设置为上一次的尾结点last,
 * 将上一次尾节点last的后一个节点（after）设置为当前尾结点p
 * 通过此方法实现了双向链表功能，完成before,after,head,tail的值设置
 */
```



## ArrayList与LinkedList的线程不安全问题以级解决方法

方法1：Collections.synchronizedList(new LinkedList<String>())

方法2：LinkedList和ArrayList换成线程安全的集合，如CopyOnWriteArrayList，ConcurrentLinkedQueue......

方法3：Vector(内部主要使用synchronized关键字实现同步)



## ArrayList/Vetcor

| ArrayList                                                    | Vector                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1、实现原理：采用动态对象数组实现，默认构造方法创建了一个空数组 | 1：实现原理：采用动态数组对象实现，默认构造方法创建了一个大小为10的对象数组 |
| 2、第一次添加元素，扩展容量为10，之后的扩充算法：原来数组大小+原来数组的一半 | 2、扩充的算法：当增量为0时，扩充为原来大小的2倍，当增量大于0时，扩充为原来大小+增量 |
| 3、不适合进行删除或插入操作                                  | 3、不适合删除或插入操作                                      |
| 4、为了防止数组动态扩充次数过多，建议创建ArrayList时，给定初始容量。 | 4、为了防止数组动态扩充次数过多，建议创建Vector时，给定初始容量 |
| 5、多线程中使用不安全，适合在单线程访问时使用，效率较高。    | 5、线程安全，适合在多线程访问时使用，效率较低(synchronized)  |

## HashSet实现原理

HashSet实现Set接口，由哈希表（实际上是一个HashMap实例）支持。它不保证set 的迭代顺序；特别是它不保证该顺序恒久不变。此类允许使用null元素。

对于HashSet而言，它是基于HashMap实现的，HashSet底层使用HashMap来保存所有元素，因此HashSet 的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成，HashSet的源代码如下：

map是一个HashMap的实例，Set偷偷的用了HashMap的put方法，然而HashMap并没有去重的功能呀，那么Set是如何做到去重的呢？

可以看到，E是我们要存储的值，而到了HashMap里面却变成了Key，PRESENT就是个空对象。

在HashMap中Key的HashCode是决定底层数组的下标，进一步使用equals进行遍历对象链表中的Key进而覆盖原来的Value。

那么对于HashSet，如果e已经存在（先HashCode相同定位到链表，然后equals比较定位到具体的Node），那么覆盖oldValue（value其实就是个傀儡，没啥用），Key不变；如果不存在，就添加一个新的节点（即加了一个新的Key）。

HashMap的返回值是oldValue，oldValue==null说明节点之前不存在；反之说明节点存在，虽然返回false但实际上还是对底层数据进行了改变（即旧的空对象变成了新的空对象）。

总而言之，HashSet确定相同的方式其实就是HashCode相同（才能找到同一链表），然后equals的返回值（才能比较具体节点进行覆盖）

## Queue中poll()和remove有什么区别

remove() ，如果队列为空的时候，则会抛出异常，而poll（）只会返回null

## 哪些集合是线程安全的

Vector、HashTable、Properties是线程安全的；

ArrayList、LinkedList、HashSet、TreeSet、HashMap、TreeMap等都是线程不安全的。

值得注意的是：为了保证集合是线程安全的，相应的效率也比较低；线程不安全的集合效率相对会高一些。

## Iterator和ListIterator的区别

- Iterator可用来遍历Set和List集合，但是ListIterator只能用来遍历List。
- Iterator对集合只能是前向遍历，ListIterator既可以前向也可以后向。
- ListIterator实现了Iterator接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。

## 怎么确保一个集合不被修改

我们很容易想到用final关键字进行修饰，我们都知道

final关键字可以修饰类，方法，成员变量，final修饰的类不能被继承，final修饰的方法不能被重写，final修饰的成员变量必须初始化值，如果这个成员变量是基本数据类型，表示这个变量的值是不可改变的，如果说这个成员变量是引用类型，则表示这个引用的地址值是不能改变的，但是这个引用所指向的对象里面的内容还是可以改变的

那么，我们怎么确保一个集合不能被修改？首先我们要清楚，集合（map,set,list…）都是引用类型，所以我们如果用final修饰的话，集合里面的内容还是可以修改的。

我们可以做一个实验：

可以看到：我们用final关键字定义了一个map集合，这时候我们往集合里面传值，第一个键值对1,1；我们再修改后，可以把键为1的值改为100，说明我们是可以修改map集合的值的。

那我们应该怎么做才能确保集合不被修改呢？
我们可以采用Collections包下的unmodifiableMap方法，通过这个方法返回的map,是不可以修改的。他会报 java.lang.UnsupportedOperationException错。

同理：Collections包也提供了对list和set集合的方法。
Collections.unmodifiableList(List)
Collections.unmodifiableSet(Set)



## 



# 接口/抽象类

**抽象类**
抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。
抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承。

**接口**
接口是抽象类的延伸
接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected。
接口的字段默认都是 static 和 final 的。

## 接口与抽象类的区别



## 接口是否可以继承接口

java接口可以多继承。Interface3 Extends Interface0, Interface1, interface……

<img src="https://pic4.zhimg.com/80/v2-0f96eaa81fcb5d6b127bda60864a280d_1440w.jpg?source=1940ef5c" alt="img" style="zoom:67%;" />

## 抽象类是否可以实现接口

抽象类肯定可以实现接口，这是一种思想，当你自己写的类想用接口中个别方法的时候（注意不是所有的方法），那么你就可以用一个抽象类先实现这个接口（方法体中为空），然后再用你的类继承这个抽象类，这样就可以达到你的目的了，如果你直接用类实现接口，那是所有方法都必须实现的

这种思想在java.swing.event包中运用的非常多，里面一般以Adapter为后缀的都是抽象类，它们都实现了特定的事件接口

抽象类定义在接口与真正的实现类之间有一个重要的作用，就是过滤掉一些不需由真正的实现类重写的方法。举一个例子，譬如说HttpServlet这个抽象类类中有init(),doGet(),doPost(),destroy()等方法，但是真正要让程序员实现的只有doGet(),doPost()，就是因为HttpServlet中定义了对一些方法的默认实现，这样一个类在扩展它时，就不必重写所有HttpServlet或者其父类所实现的所有接口的所有方法。经常是这样，即使我们重写init()方法，也只是调用super.init()

举个例子 

> “桌子”是一个 interface，它要有一个桌面能用来摆放东西。 
> “圆桌”是“桌子”的 abstract class，它定义了桌面是圆的。 
> “正圆桌”继承自“圆桌”，它的桌面是圆形中的正圆。 
> “椭圆桌”继承自“圆桌”，它的桌面是圆形中的椭圆。 

在Java中，如果抛开 Object 类，那么 Interface 就是最高层次的抽象，其次是 abstract class，接下来是 class，最后是程序运行的时候产生的对象。

## 抽象类是否可以继承实体类

抽象类是可以继承实体类，但前提是实体类必须有明确的构造函数

其实从Object就是个实体类，java的API文档里，每个抽象类的条目里都明确写着直接或间接继承自Object



# 重构/重载/重写



## 构造器是否可以被override



# jdk1.8特性

## 1.接口的默认方法

Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 default关键字即可，这个特征又叫做扩展方法。

## 2.Lambda 表达式

在Java 8 中你就没必要使用这种传统的匿名对象的方式了，Java 8提供了更简洁的语法，[lambda表达式](https://www.baidu.com/s?wd=lambda表达式&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3mynkn1-huADkmHmsm1K90ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHm4nHcLnjc3nWTznHbsP1DYr0)：

Collections.sort(names, (String a, String b) -> {

return b.compareTo(a);

});

## 3.函数式接口

Lambda表达式是如何在java的类型系统中表示的呢？每一个lambda表达式都对应一个类型，通常是接口类型。而“函数式接口”是指仅仅只包含一个抽象方法的接口，每一个该类型的lambda表达式都会被匹配到这个抽象方法。因为 默认方法 不算抽象方法，所以你也可以给你的函数式接口添加默认方法。 

## 4.方法与构造函数引用

Java 8 允许你使用 :: 关键字来传递方法或者[构造函数]引用，上面的代码展示了如何引用一个静态方法，我们也可以引用一个对象的方法：

converter = something::startsWith;

String converted = converter.convert("Java");

System.out.println(converted);

## 5.Lambda 作用域

在lambda表达式中访问外层作用域和老版本的匿名对象中的方式很相似。你可以直接访问标记了final的外层局部变量，或者实例的字段以及静态变量。

## 6.访问局部变量

可以直接在lambda表达式中访问外层的局部变量：

## 7.访问对象字段与静态变量

和本地变量不同的是，lambda内部对于实例的字段以及[静态变量](https://www.baidu.com/s?wd=静态变量&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3mynkn1-huADkmHmsm1K90ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHm4nHcLnjc3nWTznHbsP1DYr0)是即可读又可写。该行为和匿名对象是一致的：

八、访问接口的默认方法

JDK 1.8 API包含了很多内建的函数式接口，在老Java中常用到的比如Comparator或者Runnable接口，这些接口都增加了@FunctionalInterface注解以便能用在lambda上。

Java 8 API同样还提供了很多全新的函数式接口来让工作更加方便，有一些接口是来自Google Guava[库里](https://www.baidu.com/s?wd=库里&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1Y3mynkn1-huADkmHmsm1K90ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHm4nHcLnjc3nWTznHbsP1DYr0)的，即便你对这些很熟悉了，还是有必要看看这些是如何扩展到lambda上使用的。

## 8.流(stream)

![img](https://img-blog.csdn.net/20180903231649588?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FsX2Fzc2Fk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

①中间操作 

- 当数据源中的数据上了流水线后，这个过程对数据进行的所有操作都称为“中间操作”；
- 中间操作仍然会返回一个流对象，因此多个中间操作可以串连起来形成一个流水线；
- stream 提供了多种类型的中间操作，如 filter、distinct、map、sorted 等等；

②终端操作 

- 当所有的中间操作完成后，若要将数据从流水线上拿下来，则需要执行终端操作；
- stream 对于终端操作，可以直接提供一个中间操作的结果，或者将结果转换为特定的 collection、array、String 等；

**stream 的特点**

①只能遍历一次：

数据流的从一头获取数据源，在流水线上依次对元素进行操作，当元素通过流水线，便无法再对其进行操作，可以重新在数据源获取一个新的数据流进行操作；

②采用内部迭代的方式：

对Collection进行处理，一般会使用 Iterator 遍历器的遍历方式，这是一种外部迭代；

而对于处理Stream，只要申明处理方式，处理过程由流对象自行完成，这是一种内部迭代，**对于大量数据的迭代处理中，内部迭代比外部迭代要更加高效；**


**stream 和 iterator 迭代的效率比较**

好了，上面 stream 的优点吹了那么多，stream 函数式的写法是很舒服，那么 steam 的效率到底怎样呢？

先说结论：

- 传统 iterator (for-loop) 比 stream(JDK8) 迭代性能要高，尤其在小数据量的情况下；

- 在多核情景下，对于大数据量的处理，parallel stream 可以有比 iterator 更高的迭代处理效率；

我分别对一个随机数列 List （数量从 10 到 10000000）进行映射、过滤、排序、规约统计、字符串转化场景下，对使用 stream 和 iterator 实现的运行效率进行了统计，测试代码 基准测试代码链接

xxxxx这里很多实验结果，后面再补

实验结果总结

从以上的实验来看，可以总结处以下几点：

- 在少低数据量的处理场景中（size<=1000），stream 的处理效率是不如传统的 iterator 外部迭代器处理速度快的，但是实际上这些处理任务本身运行时间都低于毫秒，这点效率的差距对普通业务几乎没有影响，反而 stream 可以使得代码更加简洁；

- 在大数据量（szie>10000）时，stream 的处理效率会高于 iterator，特别是使用了并行流，在cpu恰好将线程分配到多个核心的条件下（当然parallel stream 底层使用的是 JVM 的 ForkJoinPool，这东西分配线程本身就很玄学），可以达到一个很高的运行效率，然而实际普通业务一般不会有需要迭代高于10000次的计算；

- Parallel Stream 受引 CPU 环境影响很大，当没分配到多个cpu核心时，加上引用 forkJoinPool 的开销，运行效率可能还不如普通的 Stream；



## 9.localdate

## 10.Option



## 常用的Hash算法

常用的hash算法有哪些？

1. 加法Hash；把输入元素一个一个的加起来构成最后的结果
   位运算Hash；这类型Hash函数通过利用各种位运算（常见的是移位和异或）来充分的混合输入元素
2. 乘法Hash；这种类型的Hash函数利用了乘法的不相关性（乘法的这种性质，最有名的莫过于平方取头尾的随机数生成算
   法，虽然这种算法效果并不好）；jdk5.0里面的String类的hashCode()方法也使用乘法Hash；32位FNV算法
3. 除法Hash；除法和乘法一样，同样具有表面上看起来的不相关性。不过，因为除法太慢，这种方式几乎找不到真正的应用
4. 查表Hash；查表Hash最有名的例子莫过于CRC系列算法。虽然CRC系列算法本身并不是查表，但是，查表是它的一种最快
   的实现方式。查表Hash中有名的例子有：Universal Hashing和Zobrist Hashing。他们的表格都是随机生成的。
5. 混合Hash；混合Hash算法利用了以上各种方式。各种常见的Hash算法，比如MD5、Tiger都属于这个范围。它们一般很少
   在面向查找的Hash函数里面使用



## Nginx

### 负载均衡策略

| 轮询               | 默认方式        |
| ------------------ | --------------- |
| weight             | 权重方式        |
| ip_hash            | 依据ip分配方式  |
| least_conn         | 最少连接方式    |
| fair（第三方）     | 响应时间方式    |
| url_hash（第三方） | 依据URL分配方式 |



## 设计模式

(实际项目代码里用到的地方)

### 1.单例模式

#### 1.1.实现方式

a） 将被实现的类的构造方法设计成private的。

b） 添加此类引用的静态成员变量，并为其实例化。

c） 在被实现的类中提供公共的CreateInstance函数，返回实例化的此类,就是b中的静态成员变量。

#### 1.2.五种写法

##### 1.2.1.懒汉

懒汉式，顾名思义就是实例在用到的时候才去创建，“比较懒”，用的时候才去检查有没有实例，如果有则返回，没有则新建。有线程安全和线程不安全两种写法，区别就是synchronized关键字。

![img](https://img-blog.csdnimg.cn/20190623135131582.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fic29sdXRlX2NoZW4=,size_16,color_FFFFFF,t_70)

##### 1.2.2.饿汉式

饿汉式，从名字上也很好理解，就是“比较勤”，实例在初始化的时候就已经建好了，不管你有没有用到，都先建好了再说。好处是没有线程安全的问题，坏处是浪费内存空间。

![img](https://img-blog.csdnimg.cn/20190623135149892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fic29sdXRlX2NoZW4=,size_16,color_FFFFFF,t_70)

##### 1.2.3.双检锁

双检锁，又叫双重校验锁，综合了懒汉式和饿汉式两者的优缺点整合而成。看上面代码实现中，特点是在synchronized关键字内外都加了一层 if 条件判断，这样既保证了线程安全，又比直接上锁提高了执行效率，还节省了内存空间。

<!--文章这里不规范，需要加上volatile关键字，不然高并发存在指令重排问题-->

```
/**
 * error部分，JVM会分成三步：
 * 1.分配内存空间
 * 2.初始化对象
 * 3.将对象指向刚分配的内存空间
 * <p>
 * 2,3步可能发生指令重排
 * 两个线程，A线程分配内存空间，并将对象指向刚分配的内存空间，尚未初始化
 * B线程判断不为空，拿到未初始化的对象
 * <p>
 * 解决方法 volatile关键字
 * //private volatile static SingletonDoubleSynchronized singleton = null;
 * 禁止重排序
 */
```

![img](https://img-blog.csdnimg.cn/20190623135205866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fic29sdXRlX2NoZW4=,size_16,color_FFFFFF,t_70)

##### 1.2.4.静态内部类

静态内部类的方式效果类似双检锁，但实现更简单。但这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。

![img](https://img-blog.csdnimg.cn/20190623135218977.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fic29sdXRlX2NoZW4=,size_16,color_FFFFFF,t_70)

##### 1.2.5.枚举

枚举的方式是比较少见的一种实现方式，但是看上面的代码实现，却更简洁清晰。并且她还自动支持序列化机制，绝对防止多次实例化

![img](https://img-blog.csdnimg.cn/20190623135234518.png)

#### 1.3.项目中运用场景

1. 网站的计数器，一般也是采用单例模式实现，否则难以同步。 

2. 应用程序的日志应用，一般都何用单例模式实现，这一般是由于共享的日志文件一直处于打开状态，因为只能有一个实例去操作，否则内容不好追加。 

3. 数据库连接池的设计一般也是采用单例模式，因为数据库连接是一种数据库资源。数据库软件系统中使用数据库连接池，主要是节省打开或者关闭数据库连接所引起的效率损耗，这种效率上的损耗还是非常昂贵的，因为何用单例模式来维护，就可以大大降低这种损耗。 

4. 多线程的线程池的设计一般也是采用单例模式，这是由于线程池要方便对池中的线程进行控制。 

5. SimpleDateFormat

   

#### 1.4可能问的问题

可能的问题： 
1.为何要检测2次？ 
有可能延迟或者缓存原因，造成构造多个实例，违反了单例的初衷。 
2.构造函数能否公有化 
不，单例类的构造函数必须私有化，单例类不能被实例化，只能被静态调用。 
3.lock住的对象为什么要是object对象，可以是int型吗？ 
不行，锁住的必须是个引用类型，如果锁值类型，每个不同的线程在声明的时候值类型变量的地址都不一样，那么上个线程锁住的东西，下个线程进来会认为根本没有锁，相当于每次都锁了不同的门。而引用类型的变量地址是相同的，每个线程进来判断锁都是判断同一个地址，相当于锁在同一扇门，起到了锁的作用。



### 2.工厂模式

工厂模式其实是三种模式
关键点如下： 
一、实现越来越复杂 
二、简单工厂通过构造时传入的标识来生产产品，不同产品都在同一个工厂中生产，这种判断会随着产品的增加而增加，给扩展和维护带来麻烦。 
三、工厂模式无法解决产品族和产品等级结构的问题 
四、抽象工厂模式中，一个工厂生产多个产品，它们是一个产品族，不同的产品族的产品派生于不同的抽象产品（或产品接口）。 

#### 2.1.简单工厂模式

**简单工厂模式：**一个抽象的接口，多个抽象接口的实现类，一个工厂类，用来实例化抽象的接口

```java
// 抽象产品类
abstract class Car {
    public void run();
 
    public void stop();
}
 
// 具体实现类
class Benz implements Car {
    public void run() {
        System.out.println("Benz开始启动了。。。。。");
    }
 
    public void stop() {
        System.out.println("Benz停车了。。。。。");
    }
}
 
class Ford implements Car {
    public void run() {
        System.out.println("Ford开始启动了。。。");
    }
 
    public void stop() {
        System.out.println("Ford停车了。。。。");
    }
}
 
// 工厂类
class Factory {
    public static Car getCarInstance(String type) {
        Car c = null;
        if ("Benz".equals(type)) {
            c = new Benz();
        }
        if ("Ford".equals(type)) {
            c = new Ford();
        }
        return c;
    }
}
 
public class Test {
    public static void main(String[] args) {
        Car c = Factory.getCarInstance("Benz");
        if (c != null) {
            c.run();
            c.stop();
        } else {
            System.out.println("造不了这种汽车。。。");
        }
    }
}
```

#### 2.2.工厂方法模式

有四个角色，抽象工厂模式，具体工厂模式，抽象产品模式，具体产品模式。不再是由一个工厂类去实例化具体的产品，而是由抽象工厂的子类去实例化产品

```java
// 抽象产品角色
public interface Moveable {
    void run();
}
 
// 具体产品角色
public class Plane implements Moveable {
    @Override
    public void run() {
        System.out.println("plane....");
    }
}
 
public class Broom implements Moveable {
    @Override
    public void run() {
        System.out.println("broom.....");
    }
}
 
// 抽象工厂
public abstract class VehicleFactory {
    abstract Moveable create();
}
 
// 具体工厂
public class PlaneFactory extends VehicleFactory {
    public Moveable create() {
        return new Plane();
    }
}
 
public class BroomFactory extends VehicleFactory {
    public Moveable create() {
        return new Broom();
    }
}
 
// 测试类
public class Test {
    public static void main(String[] args) {
        VehicleFactory factory = new BroomFactory();
        Moveable m = factory.create();
        m.run();
    }
}
```

#### 2.3.项目中运用场景

我们做报表的时候其实就用到了工厂模式，有个报表模板的基类，然后运输清单，配送清单等实现了这个报表基类，然后有个报表工厂方法，通过传入的报表的类型，生成不同的报表。

#### 2.4.可能问的问题

1.何时使用工厂模式 
根据具体业务需求，不要认为简单工厂是用switch case就一无是处，用设计模式是为了解决问题，根据三种模式的特质，以及对未来扩展的预期，来确定使用哪种工厂模式。 
2.你在项目中工厂模式的应用



### 3.建造者模式

#### 3.1.实现

将一个复杂对象的构建和它的表示分离，使得同样的构建过程可以创建不同的表示。

下面以生产汽车为例子：

建造的复杂对象Car，提供基本方法。

```java
public class Car {
 
	private String chassis; //汽车底盘
	private String tyre;  //汽车轮胎
	private String steeringWheel; //汽车方向盘
	public String getChassis() {
		return chassis;
	}
	public void setChassis(String chassis) {
		this.chassis = chassis;
	}
	public String getTyre() {
		return tyre;
	}
	public void setTyre(String tyre) {
		this.tyre = tyre;
	}
	public String getSteeringWheel() {
		return steeringWheel;
	}
	public void setSteeringWheel(String steeringWheel) {
		this.steeringWheel = steeringWheel;
	}
}
```

抽象的建造者接口，主要是实现所有声明的方法以及返回建造好的产品实例。

```java
public interface CarBuilder {
	public void createChassis();
	public void createTyre();
	public void createsSteeringWheel();
	public Car builder();
}
```

具体建造者实现了抽象建造者接口，主要是实现所有声明的方法以及返回建造好的产品实例。

```java
public class SagitarCarBuilder implements CarBuilder{
	private Car car=new Car();
	public void createChassis() {
		car.setChassis("速腾的底盘，嗷嗷叫，断轴很牛逼！");
	}
	public void createTyre() {
		car.setTyre("固特异的轮胎，还可以");
	}
	public void createsSteeringWheel() {
		car.setSteeringWheel("真皮方向盘！就问你牛逼不？");	
	}
	public Car builder() {
		return car;
	}
}
```

负责调用具体建造者按照顺序建造产品。导演者只负责调度，真正执行的是具体建造者角色。

```java
public class CarDirector {
	private CarBuilder carBuilder;
	public CarDirector(CarBuilder carBuilder){
		this.carBuilder = carBuilder;
	}
	public Car construct(){
		carBuilder.createChassis();
		carBuilder.createTyre();
		carBuilder.createsSteeringWheel();
		return carBuilder.builder();
	}
}
```

**跟工厂方法模式对比：**建造者模式和工厂模式同样是创建一个产品，工厂模式就是一个方法，而建造者模式有多个方法，并且建造者模式是有**顺序**的执行方法。就是说建造者模式强调的是顺序，而工厂模式没有顺序一说。

#### 3.2.项目中运用场景





### 4.策略模式

#### 4.1实现方式

策略模式的重心不是如何实现算法，而是如何组织、调用这些算法，从而让程序结构更灵活，具有更好的维护性和扩展性

假设现在要设计一个贩卖各类书籍的电子商务网站的购物车系统。一个最简单的情况就是把所有货品的单价乘上数量，但是实际情况肯定比这要复杂。比如，本网站可能对所有的高级会员提供每本20%的促销折扣；对中级会员提供每本10%的促销折扣；对初级会员没有折扣。

　　根据描述，折扣是根据以下的几个算法中的一个进行的：

　　算法一：对初级会员没有折扣。

　　算法二：对中级会员提供10%的促销折扣。

　　算法三：对高级会员提供20%的促销折扣。

　　使用策略模式来实现的结构图如下：

![img](data:image/png;base64,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)

价格接口

```java
public interface MemberStrategy {
    /**
     * 计算图书的价格
     * @param booksPrice    图书的原价
     * @return    计算出打折后的价格
     */
    public double calcPrice(double booksPrice);
}
```

初级会员折扣类

```java
public class PrimaryMemberStrategy implements MemberStrategy {
    @Override
    public double calcPrice(double booksPrice) {
        System.out.println("对于初级会员的没有折扣");
        return booksPrice;
    }
}
```

中级会员折扣类

```java
public class IntermediateMemberStrategy implements MemberStrategy {
    @Override
    public double calcPrice(double booksPrice) {
        System.out.println("对于中级会员的折扣为10%");
        return booksPrice * 0.9;
    }
}
```

高级会员折扣类

```java
public class AdvancedMemberStrategy implements MemberStrategy {
    @Override
    public double calcPrice(double booksPrice) {
        System.out.println("对于高级会员的折扣为20%");
        return booksPrice * 0.8;
    }
}
```

　价格类

```java
public class Price {
    //持有一个具体的策略对象
    private MemberStrategy strategy;
    /**
     * 构造函数，传入一个具体的策略对象
     * @param strategy    具体的策略对象
     */
    public Price(MemberStrategy strategy){
        this.strategy = strategy;
    }
    
    /**
     * 计算图书的价格
     * @param booksPrice    图书的原价
     * @return    计算出打折后的价格
     */
    public double quote(double booksPrice){
        return this.strategy.calcPrice(booksPrice);
    }
}
```

#### 4.2.项目中运用场景

策略模式:我们app也有会员制，就是开单的时候根据是否是会员有一定的折扣（我们开一单是默认1分钱）,开了会员按照会员的等级有9折、8折这样的优惠，然后我就有一个开单价格的接口，然后不同等级的会员类实现了这个接口写了自己的一些付费逻辑，然后开单的时候通过传入的不同的会员类，实现不同的开单场景



### 5.观察者模式

#### 5.1.实现方式

对象间一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。Android中的各种Listener就使用到了这一设计模式，只要用户对手机进行操作，对应的listener就会被通知，并作出响应的处理。 

![è§å¯èæ¨¡å¼UMLå¾](https://img-blog.csdn.net/20160704140407554)

看不懂图的人端着小板凳到这里来，给你举个栗子：假设有三个人，小美（女，28），老王和老李。小美很漂亮，很风骚，老王和老李是两个中年男屌丝，时刻关注着小美的一举一动。有一天，小美说了一句：我老公今天不在家，一个人好无聊啊～～～，这句话被老王和老李听到了，结果乐坏了，蹭蹭蹭，没一会儿，老王就冲到小美家门口了，于是进门了………
在这里，小美是被观察者，老王和老李是观察者，被观察者发出一条信息，然后被观察者进行相应的处理，看代码：

```java
public interface Person {
    //老王和老李通过这个接口可以接收到小美发过来的消息
    void getMessage(String s);
}
```

这个接口相当于老王和老李的电话号码，小美发送通知的时候就会拨打getMessage这个电话，拨打电话就是调用接口，看不懂没关系，先往下看

```java
public class LaoWang implements Person {

    private String name = "老王";

    public LaoWang() {
    }
 
    @Override
    public void getMessage(String s) {
        System.out.println(name + "接到了小美打过来的电话，电话内容是：" + s);
    }
 
}
 
public class LaoLi implements Person {
 
    private String name = "老李";
 
    public LaoLi() {
    }
 
    @Override
    public void getMessage(String s) {
        System.out.println(name + "接到了小美打过来的电话，电话内容是：->" + s);
    }
}
```

代码很简单，我们再看看小美的代码：

```java
public class XiaoMei {
    List<Person> list = new ArrayList<Person>();
     public XiaoMei(){
     }
 
     public void addPerson(Person person){
         list.add(person);
     }
 
     //遍历list，把自己的通知发送给所有暗恋自己的人
     public void notifyPerson() {
         for(Person person:list){
             person.getMessage("今天家里就我一个人，你们过来吧，谁先过来谁就能得到我!");
         }
     }
}
```

我们写一个测试类来看一下结果对不对

```java
public class Test {
    public static void main(String[] args) {
 
        XiaoMei xiao_mei = new XiaoMei();
        LaoWang lao_wang = new LaoWang();
        LaoLi lao_li = new LaoLi();
 
        //老王和老李在小美那里都注册了一下
        xiao_mei.addPerson(lao_wang);
        xiao_mei.addPerson(lao_li);
 
        //小美向老王和老李发送通知
        xiao_mei.notifyPerson();
    }
}
```

运行结果我截图了 

![è¿è¡ç»æ](https://img-blog.csdn.net/20160929215816074)

完美～～～



#### 5.2.项目中运用场景

我理解的观察者模式其实跟通知有点类似，在项目中我们抢单中其实有用到，司机作为观察者，被司机关注的的货运站作为被观察者，货运站有用户开单，单子进入到待发件的状态时，有关注司机就会收到这个发单的消息，这个时候从而进行抢单



### 6.装饰器模式

#### 6.1.实现方法

对已有的业务逻辑进一步的封装，使其增加额外的功能，如java中的IO流就使用了装饰者模式，用户在使用的时候，可以任意组装，达到自己想要的效果。 
举个栗子，我想吃三明治，首先我需要一根大大的香肠，我喜欢吃奶油，在香肠上面加一点奶油，再放一点蔬菜，最后再用两片面包加一下，很丰盛的一顿午饭，营养又健康，那我们应该怎么来写代码呢？ 
首先，我们需要写一个Food类，让其他所有食物都来继承这个类，看代码：

```java
public class Food {
    private String food_name;
 
    public Food() {
    }
 
    public Food(String food_name) {
        this.food_name = food_name;
    }
 
    public String make() {
        return food_name;
    };
}
```

代码很简单，我就不解释了，然后我们写几个子类继承它：

```java
//面包类
public class Bread extends Food {
 
    private Food basic_food;
 
    public Bread(Food basic_food) {
        this.basic_food = basic_food;
    }
 
    public String make() {
        return basic_food.make()+"+面包";
    }
}
 
//奶油类
public class Cream extends Food {
 
    private Food basic_food;
 
    public Cream(Food basic_food) {
        this.basic_food = basic_food;
    }
 
    public String make() {
        return basic_food.make()+"+奶油";
    }
}
 
//蔬菜类
public class Vegetable extends Food {
 
    private Food basic_food;
 
    public Vegetable(Food basic_food) {
        this.basic_food = basic_food;
    }
 
    public String make() {
        return basic_food.make()+"+蔬菜";
    }
 
}
```

这几个类都是差不多的，构造方法传入一个Food类型的参数，然后在make方法中加入一些自己的逻辑，如果你还是看不懂为什么这么写，不急，你看看我的Test类是怎么写的，一看你就明白了

```java
public class Test {
    public static void main(String[] args) {
        Food food = new Bread(new Vegetable(new Cream(new Food("香肠"))));
        System.out.println(food.make());
    }
}
```

看到没有，一层一层封装，我没从里往外看：最里面我new了一个香肠，在香肠的外面我包裹了一层奶油，在奶油的外面我又加了一层蔬菜，最外面我放的是面包，是不是很形象，哈哈 ~ 这个设计模式简直跟现实生活中一摸一样，看懂了吗？ 
我们看看运行结果吧

![è¿è¡ç»æ](https://img-blog.csdn.net/20160929223316699) 

#### 6.2.项目中运用场景



等23种设计模式

> 总体来说设计模式分为三大类：
>
> 创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。
>
> 结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。
>
> 行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。



### 7.框架下使用到的设计模式

#### 7.1.Spring

1.工厂模式，这个很明显，在各种BeanFactory以及ApplicationContext创建中都用到了；
2.模版模式，这个也很明显，在各种BeanFactory以及ApplicationContext实现中也都用到了；
3.代理模式，在Aop实现中用到了JDK的动态代理；
4.策略模式，第一个地方，加载资源文件的方式，使用了不同的方法，比如：ClassPathResourece，FileSystemResource，ServletContextResource，UrlResource但他们都有共同的借口Resource；第二个地方就是在Aop的实现中，采用了两种不同的方式，JDK动态代理和CGLIB代理；
5.单例模式，这个比如在创建bean的时候。

#### 7.2.Mybatis

Mybatis中用到至少用到以下设计模式，

1、策略模式

例如SqlSessionFactoryBuilder、XMLConfigBuilder、XMLMapperBuilder、XMLStatementBuilder、CacheBuilder；

2、 工厂模式

例如SqlSessionFactory、ObjectFactory、MapperProxyFactory；

3、单例模式

例如ErrorContext和LogFactory

4、代理模式

Mybatis实现的核心，比如MapperProxy、ConnectionLogger，用的jdk的动态代理；还有executor.loader包使用了cglib或者javassist达到延迟加载的效果；

5、组合模式

例如SqlNode和各个子类ChooseSqlNode等

6、模板方法模式

例如BaseExecutor和SimpleExecutor，还有BaseTypeHandler和所有的子类例如IntegerTypeHandler；

7、适配器模式

例如Log的Mybatis接口和它对jdbc、log4j等各种日志框架的适配实现；

8、装饰者模式

例如Cache包中的cache.decorators子包中等各个装饰者的实现；

9、迭代器模式

例如迭代器模式PropertyTokenizer；

#### 7.3.Tomcat

算了把兄弟，别搞我了

### 





秒杀系统设计
什么是秒杀

通俗一点讲就是网络商家为促销等目的组织的网上限时抢购活动

业务特点

高并发：秒杀的特点就是这样时间极短、 瞬间用户量大。

库存量少：一般秒杀活动商品量很少，这就导致了只有极少量用户能成功购买到。

业务简单：流程比较简单，一般都是下订单、扣库存、支付订单

恶意请求，数据库压力大

解决方案

前端：页面资源静态化，按钮控制，使用答题校验码可以防止秒杀器的干扰，让更多用户有机会抢到

nginx：校验恶意请求，转发请求，负载均衡；动静分离，不走tomcat获取静态资源；gzip压缩，减少静态文件传输的体积，节省带宽，提高渲染速度

业务层：集群，多台机器处理，提高并发能力

redis：集群保证高可用，持久化数据；分布式锁（悲观锁）；缓存热点数据（库存）

mq：削峰限流，MQ堆积订单，保护订单处理层的负载，Consumer根据自己的消费能力来取Task，实际上下游的压力就可控了。重点做好路由层和MQ的安全

数据库：读写分离，拆分事务提高并发度

秒杀系统设计小结

秒杀系统就是一个“三高”系统，即高并发、高性能和高可用的分布式系统
秒杀设计原则：前台请求尽量少，后台数据尽量少，调用链路尽量短，尽量不要有单点
秒杀高并发方法：访问拦截、分流、动静分离
秒杀数据方法：减库存策略、热点、异步、限流降级
访问拦截主要思路：通过CDN和缓存技术，尽量把访问拦截在离用户更近的层，尽可能地过滤掉无效请求。
分流主要思路：通过分布式集群技术，多台机器处理，提高并发能力。



## 异常

**所有异常的爸爸：java.lang.Throwable**

Java是一门万物皆为对象的语言，其中异常也不例外。Throwable是所有异常的父类，我们平时遇到的所有异常都是这个类的子类。

下图便是一些基本常见的异常子类与其关系图。

![img](http://i1167.photobucket.com/albums/q637/CodeAi/1_zps1pcwkraq.jpg)

  **2.异常类的三种主要类型:System error(系统错误),Exception(异常),RuntimeException(运行时异常)**

### 1.system error

系统错误是由 Java Virtual Machine(java虚拟机)抛出的,用Error类来代表。这类错误一般代表系统的内部错误（jvm自己的错误），但是这类问题很少发生，反正我还没有遇到过。如果遇到，可能是你所搭建的开发环境有问题，好好检察一下环境。

### 2.Exception

异常描述的是由程序和外部环境所引起的一些错误，但是能被程序捕获和处理。也就是说这种异常是可以被程序员处理的！

### 3.RuntimeException

运行时错误是java程序员所会遇到次数最多的错误，因为这种错误一般是代表着是程序员在设计程序中所犯的错误。常见的错误有：空指针错误（NullPointer），数组越界错误(IndexOutOfBounds)，算数错误(Arithmetic)，非法传参(IleagleArgument)错误...等等



 两大分类：

检查异常（checked exception）和免检查异常 (unchecked exception)

（1）checked exception
顾名思义，检查异常就是jvm会帮你检查的异常。RuntimeException，error和它们的子类都称为免检查异常，这就是说当操作存在抛出免检查异常的可能的时候，我们不强制需要对我们的代码做更多处理（例如添加 try catch语句块或者是声明异常）。jvm会在程序运行时主动抛出这些异常。

  但是个人在实际开发中的体会是即使是检查类型的异常，也需要我们去try  catch语句块去捕获和处理以避免jvm停止工作。以android开发为例，常常因为应用中有一个空指针错误（没有成功从后台获取所需信息）而突然闪退出应用程序，这对于用户肯定不是很好的一个体验。所以平时可以添加一个try catch语句块在中央调度程序的运行块中，然后把所有的非检查错误自定封装起来。在发生这类错误的时候直接弹出一个dialog告诉用户（哎呀，我不小心崩溃了！），这样的处理是不是很萌呢？

（2）unchecked exception

​      免检查异常就是需要程序员自己去捕获处理的异常，例如程序中总会有一些虽然符合机器逻辑但是并不符合我们业务逻辑的错误。这些错误虽然对于程序来说可以继续运行，但是并不是向我们所期望的方向所运行。这时候自定义异常继承Exception然后再在catch语句块中去添加处理措施。



​                   

# 生产问题

## 遇到cpu 100%怎么排查问题

处理流程：

1. 登陆线上机器top命令，查看耗费cpu的进程号，举例来说发现进程3997持续耗费资源
2. top -H -p 3997去查看持续耗费cpu的线程号30437
3. printf "%x\n" 30437将线程号转为16进制，转换为76e5
4. jstack 3997 > jstack.txt将进程3997的线程堆栈打印出来
5. 在jstack.txt中搜索76e5就可以看到这个持续耗费cpu的线程的堆栈信息，进而分析出程序在做什么

## 遇到oom怎么排查问题

