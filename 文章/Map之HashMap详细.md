
## Java集合框架之HashMap详细

希望能通过了解以下几个方面来完整的掌握理解HashMap实现的原理。

> 关注的点：
> 1. 实现本质
> 2. 负载因子的作用
> 3. 如何扩容？
> 4. 如何决定存储的位置以及解决hash冲突
> 5. 如何取出一个元素



### 1. HashMap实现的原理

```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * An empty table instance to share when the table is not inflated.
 */
static final Entry<?,?>[] EMPTY_TABLE = {};

/**
 * The table, resized as necessary. Length MUST Always be a power of two.
 */
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```
由这段代码可知HashMap的本质是维护了一个Entry<K,V>的数组，默认数组的大小为16，负载因子默认为0.75。使用默认情况下，我们可以计算当数组存储的元素超过16*0.75=12时，数组就会扩容第一次。

Entry<K,V>的结构由它的部分代码可以看出是一个单向的链表,每一个节点包含了key以及所对应的value.这种结构为HashMap集合提供了快速定位元素，并解决了通过hash计算数组下标地址所带来的地址冲突（可能不同的key所计算出来的hash值相同）。

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next; //指向下一个节点
    int hash;

    /**
     * Creates new entry.
     */
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
}
```
结构如下图：

![](https://raw.githubusercontent.com/callme001/my-images/master/imgs/2016-09-06%2019-16-40%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

与ArrayList不同，HashMap内部的数组是通过计算hash值来确定存储的位置，同时也是通过计算hash值确定数组下标来快速定位所访问的元素。比起数组遍历，这种操作具有很高的访问效率。

#### 2. 负载因子的作用

学习HashMap的知识点，可能会经常听见负载因子这个名词。负载因子是用于确定数组达到何种状态的时候扩容，

当： 数组存储的元素个数  > 数组容量 * 负载因子 的时候，数组会自动扩容。

如果负载因子设置得过小，会造成数组空间资源的浪费，并且数组在扩容的时候会增加额外的开销。

如果负载因子设置得过大，会造成查询的时候增加额外的时间成本。原理是负载因子设置得过大，数组元素的单链表结构会增加，使得类似上图数组下标为16的这种情况增加，甚至一个数组元素下面的'链'会更长，在获取元素的时候需要遍历这条单向的链表，增加了查找时的时间成本。

#### 3. 如何扩容

扩容的主要函数：

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) { //检车是否达到扩容条件即：数组存储的元素个数  > 数组容量 * 负载因子 
        resize(2 * table.length); //如果达到扩容条件，则执行扩容 每次扩容是上一次容量的2倍
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}

void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1); //说明threshold变量的值是等于 数组容量 * 负载因子
}
```
在存储一个key-value时，会检测是否达到扩容的条件，如果达到扩容条件，执行扩容。每次扩容的总容量为上一次容量的**2倍**。

数组扩容实际上就是 **创建一个新的数组，并且把以前的数组复制到新的数组**

#### 4. 如何决定存储的位置以及解决hash冲突

从put函数说起：
（由下面的代码也可以看出，HashMap是允许null作为key和值的，作为key的时候null只有一个）
```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) { //判断table数组是不是空，如果是空则初始化table
        inflateTable(threshold);
    }
    if (key == null) //如果Key为空 执行hashmap空key增加的操作
        return putForNullKey(value);
    int hash = hash(key); //hash是通过key来计算获得的
    int i = indexFor(hash, table.length); // 这是计算存储位置的，这个存储位置就是table数组的下标位置
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

如上代码所示：

我们假设一种极端情况，两个不同的key产生了相同的hash值，hashmap是怎么解决冲突的呢？

首先根据key计算出了hash值，并且通过hash值和数组长度计算出了存储在数组的下标值i，i的值就是Entry所在数组的下标地址。（如何保证计算的值不会超过数组长度，也是HashMap设计的一个精巧的点`h & (length-1)`传入一个任意大小的数与length-1做与运算，得到的数永远小于length）

我们假设这里key计算出来的hash和另一个不同的key计算出来的hash值一样，即产生了冲突。于是代码
```java
e.hash == hash && ((k = e.key) == key // hash值相等 但是key值不等
```
条件不成立，跳出循环进入addEntry()函数。继续把hash值key,value以及数组下标传进去。

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {  //判断是否满足扩容条件
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}

Entry(int h, K k, V v, Entry<K,V> n) {
    value = v;
    next = n;
    key = k;
    hash = h;
}
```

从createEntry函数可以看出，首先是把由hash计算出的数组下标的元素取出来，然后赋值了一个新的Entry<>，由Entry的初始化函数可以得知，以前的被放进了第二个位置。

![](https://raw.githubusercontent.com/callme001/my-images/master/imgs/2016-09-06%2019-16-40%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

即First被放置到了单链表的第二个元素，新的元素成为了单链表的第一个元素。

这就是HashMap处理hash相等但是key不等的情况。如果出现了较多的这种情况，会导致HashMap在获取元素时性能降低。理想的状态就是一个数组节点只存储一个元素，通过计算hash值直接获取元素，这是最高效的方式。


#### 5. 如何取出一个元素

如果对于如何解决冲突还不是很明白，通过查看源码HashMap如何取出一个元素，会加强理解HashMap解决冲突的方式。

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}

final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)]; //遍历单链表
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) //查找到hash值相等并且key值也相等的元素，然后返回
            return e;
    }
    return null;
}
```
for循环中，首先通过key以及数组长度计算了该元素的数组下标，然后遍历单链表。当且仅当hash值相等,key值相等的时候返回元素。

所以在取出元素的过程中，不仅仅是通过计算hash值取得下标就获取元素，而是满足hash值和key值都相等才可以。这就是为什么hash冲突了依旧能正确取出元素的原因。

以上即为学习HashMap的全部内容，关于计算数组下标永远不会超出数组边界。做了以下实验：

```java
    int length = 16;
    System.out.println(1&(length-1)); // 1
    System.out.println(6&(length-1)); // 6
    System.out.println(8&(length-1)); // 8
    System.out.println(79464646&(length-1));  //6
```

无论值多大，计算的结果总是比16小