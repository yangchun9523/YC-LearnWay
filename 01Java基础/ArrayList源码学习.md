## Java之ArrayList
### ArrayList简介
ArrayList是我们经常使用的一个数据结构，我们通常把其用作一个可变长度的动态数组使用。
#### 特点

1. ArrayList 底层是一个动态扩容的数组结构
2. 允许存放（不止一个） null 元素
3. 允许存放重复数据，存储顺序按照元素的添加顺序
4. ArrayList 并不是一个线程安全的集合

### 源码学习
#### ArrayList继承关系
```java
public class ArrayList<E> extends AbstractList<E> //抽象集合框架
        implements 
        List<E>, // 集合框架（约定必须的方法及功能）
        RandomAccess, // 标识具有随机访问能力
        Cloneable,  // 能够被克隆。注意覆写的clone方法只是浅克隆，原始值更改克隆值也会更改
        java.io.Serializable // 支持序列化
```
#### ArrayList变量
```java
	/**
	 * ArrayList 默认的数组容量
	 */
	 private static final int DEFAULT_CAPACITY = 10;
	
	/**
	 * 这是一个共享的空的数组实例，当使用 ArrayList(0) 或者 ArrayList(Collection<? extends E> c) 
	 * 并且 c.size() = 0 的时候讲 elementData 数组讲指向这个实例对象。
	 */
	 private static final Object[] EMPTY_ELEMENTDATA = {};
	
	/**
	 * 另一个共享空数组实例，在第一次 add 元素的时候将使用它来判断数组大小是否设置为 DEFAULT_CAPACITY
	 */
	 private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	
	/**
	 * 真正装载集合元素的底层数组 
	 * 至于 transient 关键字，被它修饰的成员变量无法被 Serializable 序列化 
	 */
	transient Object[] elementData; // non-private to simplify nested class access
	
	/**
	 * ArrayList 数组的大小
	 */
	private int size;

```

#### ArrayList构造方法

```java
	// 无参构造方法，在第一次Add时初始化容量默认为10
	public ArrayList() {
	    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
	}
	// 指定容量的构造方法
	public ArrayList(int initialCapacity) {
	    if (initialCapacity > 0) {
	        this.elementData = new Object[initialCapacity];
	    } else if (initialCapacity == 0) {
	        this.elementData = EMPTY_ELEMENTDATA;
	    } else {
	        throw new IllegalArgumentException("Illegal Capacity: "+
	                                           initialCapacity);
	    }
	}
	// 构造一个包含指定集合元素的列表，元素的顺序由集合的迭代器返回
	public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

#### ArrayList动态增长
```java 
	// 添加元素先判断是否需要扩容
	public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    // 进行容量判断 DEFAULT_CAPACITY 值为10，所以第一次添加时扩展为10.
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
    // 实际的扩容操作，使用Arrays.copyOf方式。多次扩容较为消耗内存，指定初始大小容量构造方法存在原因。
    // 扩展因素 oldCapacity + (oldCapacity >> 1) 原始大小的1.5倍
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
#### ArrayList添加删除元素(内存消耗)
```java
	// 指定位置添加，先判断是否需要扩容，然后System.arraycopy复制，在指定选择位置的值
	public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    // 先System.arraycopy复制需要的元素，然后最后一位置空并减少size。置空后GC触发时会回收
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
#### 数组复制的绝招
```java
/**
*src	原数组
*srcPos	原数组其实位置
*dest	目标数组
*destPos	目标数组的起始位置
*length	要复制的数组元素的数目
*/
public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

### ArrayList使用
#### 循环方式

```text
before jdk1.8
	for 循环遍历， foreach 遍历，迭代器遍历
	// 迭代器 循环
	ListIterator<String> listIt = list2.listIterator();
	while (listIt.hasNext()){
	}
	
after jdk1.8
	forEach(支持lambda表达式)
```

#### 错误用法
程序抛出了角标越界的异常，因为这样每次 fori 的时候我们不去拿更新后的 list 元素的 size 大小，所以当我们删除一个元素后，size = 3 当我们 for 循环去list2.get(3)的时候就会被 **rangeCheck**方法抛出异常[ConcurrentModificationException],**在多线程使用情况下，比较常见**。

```java
int size = list2.size();
for (int i = 0; i < size; i++) {
  if (list2.get(i).test == 3) {
      list2.remove(i);//remove 以后 list 内部将 size 重新改变了 for 循环下次调用的时候可能就不进去了
  }
}
System.out.println(list2);

//Exception in thread "main" java.lang.IndexOutOfBoundsException: Index: 3, Size: 3
```