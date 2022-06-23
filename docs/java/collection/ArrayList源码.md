## ArrayList 源码

### 简介

ArrayList底层使用数组来实现的集合，但是实现了动态扩容的功能，所以相当于是动态数组了，所以说比直接使用数组要方便很多。

ArrayList实现了一些接口和对应的功能。

- List：实现了List的功能，有序、可重复。
- RandomAccess：支持随机访问，也就是说可以根据下标快速获取到元素。
- Cloneable：重写 clone 方法，支持对象拷贝。
- Serializable：序列化接口，使用 readObject 和 writeObject 方法序列化反序列化对象。



### 核心源码解读

```java
import java.util.*;
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /** 默认初始容量 **/
    private static final int DEFAULT_CAPACITY = 10;

    /** 空数组 **/
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /** 初始化默认数组 **/
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /** 真正存放元素的数组 **/
    transient Object[] elementData; // non-private to simplify nested class access

    /** 元素个数 **/
    private int size;

    /**
     * 带初始值的有参构造方法
     * @param initialCapacity 初始大小
     */
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

    /**
     * 无参构造方法，将元素数组设置为默认数组
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 带初始集合的构造方法
     * @param c 源集合
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 修改为空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * 将数组内部数组压缩到元素个数大小
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }

    /**
     * 提前扩容
     * @param minCapacity 最小容量
     */
    public void ensureCapacity(int minCapacity) {
        /**
         * 这一步是校验扩容大小是否合理。
         * 如果数组初始化了，那么扩容后的大小 > 0 就好了，因为有可能是初始化了，但是又使用 trimToSize 收缩了。
         * 如果没有初始化，最少要比默认初始化大小大。
         */
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    private void ensureCapacityInternal(int minCapacity) {
        // 如果集合刚被创建，那么则会取传入值和默认大小之间最大的
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 判断扩容后大小是否大于当前数组长度，如果不是则不给扩容，因为将数组变小可能会导致元素丢失。
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /** 数组最大长度 **/
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * 真正改变数组长度的方法
     * @param minCapacity 最小容量
     */
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 扩容至原来的 1.5 倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 这条判断语句说明，扩容至 1.5倍后的容量如果小于 传入的 最小容量，那就会使用最小容量来作为扩容后容量
        // 可以理解为 newCapacity = Math.max(newCapacity, minCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 数组扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    /**
     * 获取最大容量
     * @param minCapacity 最小容量
     * @return
     */
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0)
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    /**
     * 传入元素，找到该元素下标
     * @param o 需要匹配的元素
     * @return 匹配的元素下标，找到了返回实际下标，没找到返回 -1
     */
    public int indexOf(Object o) {
        /**
         * 这里判断 null 的原因就是因为，null 不能使用 equals 方法，
         * 所以说为了结果的准确性使用了 equals 作为实际判断方法，
         * 所以说如果自定义类没有重写 equals 还是有可能匹配不到的。
         */
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 跟 indexOf 其实是一样的，只不过是从队尾向前查找，寻找最后一个匹配的元素下标。
     * @param o
     * @return
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 拷贝方法
     * @return
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }

    /**
     * 将集合转为数组，压缩后返回
     * @return
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 将集合转为数组，指定数组类型。
     * @param a
     * @param <T>
     * @return
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 如果参数数组的大小小于当前集合内数组大小，则只使用该数组的元素类型
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        /**
         * a.length >= size 将集合内数组复制到入参数组中，我不是很推荐这种方式。
         * 1.如果通过获取 new Object[list.size()] 新建数组传入，那其实跟不指定大小返回值是一样的。
         * 2.无法保证入参数组是否干净，万一里面有值给覆盖了，空余部分还有其他的值，就很麻烦。
         * 3.可以进行简单的业务拆分使得代码更加清晰。
         * System.arraycopy 方法解析
         * Object src: 源数组
         * int srcPos: 源数组开始位置
         * Object dest: 目标数组
         * int destPos: 目标数组开始位置
         * int length: 移动元素个数
         */
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 获取指定下标的元素
     * @param index
     * @return
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 设置指定下标值
     * @param index 下标
     * @param element 新值
     * @return 老值
     */
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    /**
     * 新增元素
     * @param e
     * @return
     */
    public boolean add(E e) {
        // 判断新增是否需要扩容
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }

    /**
     * 在指定下标增加元素
     * @param index
     * @param element
     */
    public void add(int index, E element) {
        // 判断下标是否越界
        rangeCheckForAdd(index);
        // 判断是否需要扩容
        ensureCapacityInternal(size + 1);
        // 从指定下标开始，所有元素后移一位
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        // 指定下标设置新值
        elementData[index] = element;
        size++;
    }

    /**
     * 删除指定下标的值
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 从指定下标后一个开始，所有元素向前移动一位
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        // 队尾设置为 null，顺便将 size 值 -1
        elementData[--size] = null;

        return oldValue;
    }

    /**
     * 写法跟 indexOf 一样，只不过加了一个 fastRemove
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /**
     * 内部方法，实际上跟 remove 是一样的，但是不暴露给外部使用，所以减少了判断下标是否越界以及返回值
     * 提高了使用效率
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null;
    }

    /**
     * 将数组所有元素设置为 null，但是数组容量并没有改变。
     */
    public void clear() {
        modCount++;

        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 添加所有元素，跟有参构造方法类似，就不具体解析了
     * @param c
     * @return
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 跟上面那个类似，但是指定了下标，从指定下标开始插入元素。
     * @param index
     * @param c
     * @return
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);

        int numMoved = size - index;
        if (numMoved > 0)
            // 指定下标后的所有元素，移动 size - index 位
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);
        // 插入到内部数组
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 删除指定范围内的所有元素，不会更改原数组大小
     * @param fromIndex
     * @param toIndex
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        // 将 toIndex 之后的数据挪到 fromIndex 的位置
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // 去除引用，帮助 GC
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 判断下标是否越界
     * @param index
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }


    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        // 记录版本号
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        // 遍历执行 action
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }
        // 执行过程中 elementData 发生改变直接抛异常
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    public boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        int removeCount = 0;
        // 存放需要删除元素的下标
        final BitSet removeSet = new BitSet(size);
        // 跟上一个方法同理，主要原因就是 ArrayList 类不是线程安全的
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            if (filter.test(element)) {
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // 判断是否需要删除
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                // 去除引用，帮助 GC
                elementData[k] = null;
            }
            this.size = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }

    @Override
    @SuppressWarnings("unchecked")
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        // 同上
        final int expectedModCount = modCount;
        final int size = this.size;
        // 遍历执行 operator
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            elementData[i] = operator.apply((E) elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }

    @Override
    @SuppressWarnings("unchecked")
    public void sort(Comparator<? super E> c) {
        // 同上
        final int expectedModCount = modCount;
        // 调用方法对 elementData 进行排序
        Arrays.sort((E[]) elementData, 0, size, c);
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
}
```



### 动态扩容解读

简而言之就是会自动扩建内部数组大小，外部使用无感知。

我们一般在使用的时候并不会考虑这些问题，就是新建一个 list ，add 就完事了，没有考虑为啥我一直 add 却不会下标越界，初始化容量是多少？下面就从构造方法到add执行之后发生了多少事情。



#### 构造方法

```java
/**
 * 无参构造方法，将元素数组设置为默认数组
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
* 带初始值的有参构造方法
* @param initialCapacity 初始大小
*/
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
```

主要就说这两个构造方法。

第一个无参构造方法只做了一件事，那就是将对象中的 elementData 设置为常量 DEFAULTCAPACITY_EMPTY_ELEMENTDATA，而 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 是一个空数组，也就是说 ArrayList 并没有在无参构造方法的时候初始化容量，而是在第一次 add 的时候初始化的。这个时候其实就可以看出来 elementData  是实际存放元素的数组，所以的集合操作实际上都是操作 elementData  数组。

第二个构造方法看起来就比较清洗了，根据传入的参数来决定初始化容量，如果为正数则直接当做数组的容量，如果为 0 则将 elementData 属性设置为 EMPTY_ELEMENTDATA。小于 0 则直接抛出异常，无法初始化。



#### add 方法

```java
/**
 * 新增元素
 * @param e
 * @return
 */
public boolean add(E e) {
    // 判断新增是否需要扩容
    ensureCapacityInternal(size + 1);
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    // 如果集合刚被创建，那么则会取传入值和默认大小之间最大的
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 判断扩容后大小是否大于当前数组长度，如果不是则不给扩容，因为将数组变小可能会导致元素丢失。
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    // 扩容至原来的 1.5 倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 这条判断语句说明，扩容至 1.5倍后的容量如果小于 传入的 最小容量，那就会使用最小容量来作为扩容后容量
    // 可以理解为 newCapacity = Math.max(newCapacity, minCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 数组扩容
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

跟着代码和注释看一遍其实就是知道扩容的实现方式了，总结一下就是，每次对集合新增元素的时候就会校验一下新增后的元素是否超出数组大小，如果超出了就需要扩容，扩容时候会先校验一下是否是无参构造方法创建的，如果是则将扩容后的大小和默认值 DEFAULT_CAPACITY `10` 比较，选取最大的，之后再判断这个值是否大于当前数组长度，大于的话进入实际扩容方法 grow ，grow 方法会对入参和 数组长度 * 1.5 后的值进行比较，选取最大的对数组进行扩容。



#### ensureCapacity 方法

```java
/**
 * 提前扩容
 * @param minCapacity 最小容量
 */
public void ensureCapacity(int minCapacity) {
    /**
     * 这一步是校验扩容大小是否合理。
     * 如果数组初始化了，那么扩容后的大小 > 0 就好了，因为有可能是初始化了，但是又使用 trimToSize 收缩了。
     * 如果没有初始化，最少要比默认初始化大小大。
     */
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        ? 0
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

这个方法我们理解是将数组扩容到指定大小，但是本质上调用的还是 grow 方法，所以他还会对指定大小和数组长度*1.5进行判断，选取最大的扩容。

具体可以Debug查看下面这段代码

```java
ArrayList<String> arrayList = new ArrayList<>(10);
arrayList.ensureCapacity(11);
```



### 总结

对于源码解读上面注释了一部分方法，还有一些不是很常用的就没有写，如果想要更深入学习可以自己去翻阅源码。自己查看源码的时候最好 Debug 一步步查看，多看多记多理解。源码中细节其实还有很多，比如 grow 方法还会跟 数组长度*1.5 比对去取最大值，所以说有些时候自己理解但是没有仔细查看还是会有问题的。

下面这段程序比较有意思，也算是解释了为什么有了 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 还需要 EMPTY_ELEMENTDATA，自己去运行查看顺序。

```java
ArrayList<String> arrayList = new ArrayList<>(0);
for (int i = 0; i < 10; i++) {
    arrayList.add("as");
}
```