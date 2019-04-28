## Java LinkedHashMap类
### 一、LinkedHashMap简介
LinkedHashMap继承于HashMap，通过维护一个双向链表，来保证数据的访问顺序为插入顺序。一定程度增加时间及空间上的开销。  
LinkedHashMap的实现就是HashMap+LinkedList的实现方式，以HashMap维护数据结构，以LinkList的方式维护数据插入顺序 

#### 特点
- LinkedHashMap的Key和Value都允许空
- LinkedHashMap允许重复数据
- LinkedHashMap是有序的
- LinkedHashMap非线程安全

### 二、源码学习
#### 2.1维护链表的基础参数

循环双向链表的头部存放的是`最久访问的节点或最先插入的节点`，尾部为`最近访问的或最近插入的节点`，迭代器遍历方向是从链表的头部开始到链表尾部结束，在链表尾部有一个空的header节点，该节点不存放key-value内容，为LinkedHashMap类的成员属性，循环双向链表的入口(他的下个节点为最老数据)。

```java
	// 首节点（最老的节点）
	transient LinkedHashMap.Entry<K,V> head;
	// 后面的节点(最老的节点)
	transient LinkedHashMap.Entry<K,V> tail;
	// true 时是访问顺序， false时为插入顺序
	final boolean accessOrder;

	static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

```

#### 2.2 使用场景

通过重写`removeEldestEntry`方法维护数据集合, 通过一定条件删除最老的元素。  
LRUCache就是基于LRU算法的Cache（缓存），这个类继承自LinkedHashMap。LRU即`Least Recently Used`最近使用，也就是说，当缓存满了，会优先淘汰那些最近最不常访问的数据。

```java 
	protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
       return false;
   }
  
   \\ LRU 算法应用
   	public class LRUCache extends LinkedHashMap{
	    public LRUCache(int maxSize){
	        super(maxSize, 0.75F, true);
	        maxElements = maxSize;
	    }
	
	    protected boolean removeEldestEntry(java.util.Map.Entry eldest){
	        return size() > maxElements;
	    }
	
	    private static final long serialVersionUID = 1L;
	    protected int maxElements;
	}
   
```