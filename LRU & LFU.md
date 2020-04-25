为了保证缓存中的数据都是热点数据，我们一般会采用淘汰策略淘汰掉不经常访问的数据。那么这种淘汰策略是如何实现的呢？

接下来让我们以LRU和LFU缓存机制为例，了解一下它们的相关原理及底层代码实现。

**LRU（Least Recently Used）缓存机制**

> 从数据集中，挑选最近最少使用的数据淘汰。

leetcode中就有这道题([前往原题](https://leetcode-cn.com/problems/lru-cache/))：
```markdown
运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。
获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果密钥已经存在，则变更其数据值；如果密钥不存在，则插入该组「密钥/数据值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。
```
注：get()和put()方法都算是对数据的一次访问，我们要淘汰的是最长时间没有被访问过的数据。

所以，想要实现LRU，需保证以下几点：

- 支持插入key-value形式的数据。
- 可根据键值获取数据。
- 可删除最久未访问的数据。

**方法一（直接使用LinkedHashMap）**

Java中的LinkedHashMap不仅满足前两点要求，还提供了一个removeEldestEntry(Map.Entry<K,V> eldest)方法，可以删除最久未访问的结点，非常适合用来解这道题。

代码：

```java
class LRUCache extends LinkedHashMap<Integer,Integer>{
    private int capacity;
    public LRUCache(int capacity) {
        //调用父类中的构造方法创建一个LinkedHashMap，设置其容量为capacity，loadFactor为0.75，并开启accessOrder为true
        super(capacity, 0.75F, true);
        this.capacity = capacity;
    }

    public int get(int key) {
        //若key存在,返回对应value值;若key不存在,返回-1
        return super.getOrDefault(key,-1);
    }
    
    public void put(int key, int value) {
        super.put(key,value);
    }
    protected boolean removeEldestEntry(Map.Entry<Integer,Integer> eldest){
        //若返回的结果为true，则执行removeEntryForKey方法删除eldest entry
        return size() > capacity;
    }
}
```

说明：

创建LinkedHashMap时，用到了loadFactor和accessOrder。loadFactor（负载因子），是用来控制数组存放数据的疏密程度的参数。

loadFactor越趋近于1，数组中存放的数据就越多、越密，会导致查找元素效率降低。而loadFactor越趋近于0，数组中存放的数据就越少、越疏，会导致数组的利用率降低。loadFactor的默认值为0.75f，是官方给出的一个比较好的临界值。

至于accessOrder，介绍它之前，先来看一下LinkedHashMap的底层构造：

![image](https://raw.githubusercontent.com/chenyiwei00/picture/master/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-04-02%2014.23.20.png)

linkedHashMap的底层结构是数组+单向链表，此外还维护一条逻辑上的双向链表

LinkedHashMap的底层数据结构和HashMap一样，采用数组+单向链表（JDK1.8之前）的形式，只是在节点Entry中增加了before和after变量，用于维护一个双向链表来保存LinkedHashMap的存储顺序。

而这个双向链表又提供了两种排序方法：插入顺序和访问顺序。

当accessOrder为false时（默认情况），linkedHashMap只会按插入顺序维护双向链表。而开启了accessOrder之后，linkedHashMap就会把每一次对结点的访问也作为标准来进行排序。也就是说，在每次插入结点/访问结点的时候，都会将相应结点移动到双向链表的尾部，从而达到按访问顺序进行排序的目的。

所以这里需将accessOrder参数开启为true。

可以看到，用LinkedHashMap实现LRU非常方便，它内部提供了我们需要的所有操作。但走这条捷径并不是很有利于我们深刻理解LRU的缓存机制，接下来我们就尝试使用最简单的数据结构来实现。

**方法二（哈希表+双向链表）**

思路：不依赖LinkedHashMap，只使用最基础的哈希表，再维护一个双向链表，提供以下几种方法即可：

- 插入结点到头部
- 移动结点到头部
- 删除结点
- 删除尾部结点

代码：

```java
import java.util.Hashtable;
class LRUCache {
    class DLinkedNode{
        int key;
        int value;
        DLinkedNode prev;
        DLinkedNode next;
    }
    //插入结点到头部
    private void addNode(DLinkedNode node){
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }
    //删除结点
    private void removeNode(DLinkedNode node){
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    //删除尾部结点
    private void removeLastNode(){
        removeNode(tail.prev);
    }
    //移动结点到头部
    private void removeToHead(DLinkedNode node){
        removeNode(node);
        addNode(node);
    }
    private Hashtable<Integer,DLinkedNode> cache = new Hashtable<Integer,DLinkedNode>();
    private int capacity,size;
    private DLinkedNode head,tail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.size = 0;
        head = new DLinkedNode();
        tail = new DLinkedNode();
        //将头结点和尾结点相连
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        DLinkedNode node = cache.get(key);
        if(node==null){
            return -1;
        }
        //若key值存在，移动该结点到头部
        removeToHead(node);
        return node.value;
    }
    
    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);
        //若key值不存在，直接插入结点到头部，再判断当前容量是否大于capacity，如果是，就删除尾部结点
        if(node==null){
            DLinkedNode newNode = new DLinkedNode();
            newNode.key = key;
            newNode.value = value;
            cache.put(key,newNode);
            addNode(newNode);
            size++;
            if(size>capacity){
                DLinkedNode last = tail.prev;
                cache.remove(last.key);
                removeLastNode();
                size--;
            }
        }else{
            //若key值存在，则更新value值，并移动该结点到链表头部
            node.value = value;
            removeToHead(node);
        }
    }
}
```

总结：这就是LRU的底层实现了，get和put方法的时间复杂度都是O(1)，执行效率非常高。

**LFU（Least Frequently Used）缓存机制**

> 从数据集中，挑选最不经常使用的数据淘汰。

在了解了LRU的原理后，想要实现LFU也不难了。LFU和LRU的区别在于，LRU淘汰的是最久未访问到的数据，而LFU是淘汰的是最不经常使用的数据（若两个或多个数据的使用频率相同时，LFU会再选择最久未访问到的数据淘汰）。

所以我们可以在结点中新增一个代表访问频率的参数，在每次结点被访问时，令该结点的访问频率+1即可。

代码：

```java
import java.util.Hashtable;
class LFUCache {
    class DLinkedNode{
        int key;
        int value;
        int count = 0;
        DLinkedNode prev;
        DLinkedNode next;
    }
    //添加结点到头部
    private void addNode(DLinkedNode node){
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
        node.count++;
    }
    //删除结点
    private void removeNode(DLinkedNode node){
        node.next.prev = node.prev;
        node.prev.next = node.next;
    }
    //移动结点到头部
    private void removeToHead(DLinkedNode node){
        removeNode(node);
        addNode(node);
    }
    //删除最不经常访问的结点
    private DLinkedNode removeLastNode(){
        //从head.next.next结点开始查找，因为head.next是我们刚插入的结点，不参与这次筛选
        DLinkedNode pNode = head.next.next;
        int temp = tail.prev.count;
        //找到链表中的结点最低的访问频率
        while(pNode!=null && pNode.count>0){
            if(temp>pNode.count){
                temp = pNode.count;
            }
            pNode = pNode.next;
        }
        pNode = tail.prev;
        //从链表尾部开始查找具有最低访问频率的结点
        while(pNode.count>0 && pNode.count!=temp){
            pNode = pNode.prev;
        }
        removeNode(pNode);
        return pNode;
    }
    private Hashtable<Integer,DLinkedNode> cache = new Hashtable<Integer,DLinkedNode>();
    private int capacity,size;
    private DLinkedNode head,tail;
    public LFUCache(int capacity) {
        this.capacity = capacity;
        size = 0;
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        DLinkedNode node = cache.get(key);
        if(node==null){
            return -1;
        }
        removeToHead(node);
        return node.value;
    }
    
    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);
        //若key值不存在，直接插入结点到头部，再判断当前容量是否大于capacity，如果是，就删除最不经常访问的结点
        if(node==null){
            DLinkedNode newNode = new DLinkedNode();
            newNode.key = key;
            newNode.value = value;
            cache.put(key,newNode);
            addNode(newNode);
            size++;
            if(size>capacity){
                DLinkedNode res = removeLastNode();
                cache.remove(res.key);
                size--;
            }
        }else{
            //若key值存在，则更新value值，并移动该结点到链表头部
            node.value = value;
            removeToHead(node);
        }
    }
}
```

总结：LFU的get和put方法时间复杂度也都是O(1)，但由于它无法对一个拥有最初高访问率之后长时间没有被访问的条目缓存负责，所以这个策略很少被使用。

到这里，LRU和LFU缓存机制就讲完啦，这篇文章也只是我结合leetcode官方题解和相关资料所做出的粗浅理解，欢迎指正^__^。
