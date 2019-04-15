## java LinkedList类
### LinkedList简介
LinkedList是一个常用的集合类，用于顺序存储元素。LinkedList经常和ArrayList一起被提及。大部分人应该都知道ArrayList内部采用数组保存元素，适合用于随机访问比较多的场景，而随机插入、删除等操作因为要移动元素而比较慢。LinkedList内部采用链表的形式存储元素，随机访问比较慢，但是插入、删除元素比较快，一般认为时间复杂都是O(1)。  
关于栈或队列，现在的首选是**ArrayDeque**，它有着比LinkedList（当作栈或队列使用时）有着更好的性能。

#### 数据结构特点
- LinkedList允许为空
- LinkedList允许重复数据
- LinkedList存储数据为有序
- LinkedList非线程安全

### 源码学习
#### LinkedList继承关系
```java
// 实现 Deque 接口，即能将LinkedList当作双端队列使用
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

#### LinkedList变量
```java
	// 当前节点数
	transient int size = 0;
	// 首节点
	transient Node<E> first;
	// 尾节点
	transient Node<E> last;
	// transient 关键词，修饰的关键词不被序列化
```

#### 内部类
LinkedList的Entry中的”E element”，就是它真正存储的数据。”Entry<E> next”和”Entry<E> previous”表示的就是这个存储单元的前一个存储单元的引用地址和后一个存储单元的引用地址。

```java
	private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

#### ADD(E e)方法

该方法直接将新增的元素放置链表的最后面，然后链表的长度（size）加1，修改的次数（modCount）加1。

```java
	public boolean add(E e) {
        linkLast(e);
        return true;
    }
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

#### ADD(int index, E e)方法
检查添加的位置index 有没有小于等于当前的长度链表size，并且要求大于等于0。  
如果是index是等于size，那么直接往链表的最后面添加元素，相当于调用add(E e)方法。  
如果index不等于size，则先是索引到处于index位置的元素(`node(index)方法`)，然后在index的位置前面添加新增的元素。  
把索引到的元素放到新增的元素之后。


```java
	public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```
#### GET方法

`首先是判断索引位置有没有越界`，确定完成之后开始遍历链表的元素，那么从头开始遍历还是从结尾开始遍历呢，这里其实是要索引的位置与`当前链表长度的一半`去做对比，如果索引位置小于当前链表长度的一半，否则从结尾开始遍历。(意想不到的加速动作)

```java
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
    Node<E> node(int index) {
	    // assert isElementIndex(index);
	    if (index < (size >> 1)) {
	        Node<E> x = first;
	        for (int i = 0; i < index; i++)
	            x = x.next;
	        return x;
	    } else {
	        Node<E> x = last;
	        for (int i = size - 1; i > index; i--)
	            x = x.prev;
	        return x;
	    }
    }
```

#### REMOVE()方法

remove方法本质调用的还是removeFirst方法  
移除第一个节点，将第一个节点置空，让下一个节点变成第一个节点，链表长度减1，修改次数加1，返回移除的第一个节点。  

```java
    public E remove() {
        return removeFirst();
    }
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }

```

#### REMOVE(int index)方法

这个问题我稍微扩展深入一点：  
按照Java虚拟机HotSpot采用的垃圾回收检测算法—-`根节点搜索算法`来说，即使previous、element、next不设置为null也是可以回收这个Entry的，因为此时这个Entry已经没有任何地方会指向它了，tail的previous与header的next都已经变掉了，所以这块Entry会被当做”垃圾”对待。  
之所以还要将previous、element、next设置为null，我认为可能是为了兼容另外一种垃圾回收检测算法—-`引用计数法`，这种垃圾回收检测算法，只要对象之间存在相互引用，那么这块内存就不会被当作”垃圾”对待。

```java
	 // 根据对象删除
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    // 根据索引删除
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
    // 链表删除既取消当前元素前应用及后引用。当前元素前元素指向当前元素后元素
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;
		  // 删除为首节点时
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
		  // 删除为尾节点时
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
        x.item = null;
        size--;
        modCount++;
        return element;
    }
```


#### push和pop方法(队列方法)
push其实就是调用addFirst(e)方法，pop调用的就是removeFirst()方法。

### LinkedList和ArrayList的对比

有些说法认为LinkedList做插入和删除更快，这种说法其实是不准确的：  
（1）LinkedList做插入、删除的时候，慢在寻址，快在只需要改变前后Entry的引用地址  
（2）ArrayList做插入、删除的时候，慢在数组元素的批量copy，快在寻址