# 1.List
- ArrayList：基于动态数组实现，支持随机访问
- LinkedList：基于双向循环链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList还可以用作栈、队列和双端队列

## 源码分析
ArrayList.java
实现了RandomAccess接口，因此支持随机访问
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

基于数组实现，保存元素的数组使用transient进行修饰，用transient关键字标记的成员变量不参与序列化过程
这是因为该数组不一定所有的位置都占满了元素，因此没有必要全部都进行序列化
```java
transient Object[] elementData;
```

数组的默认大小为10
```java
private static final int DEFAULT_CAPACITY = 10;
```

删除元素的时候调用System.arraycopy()对元素进行复制，因此在删除元素的时候成本很高
```java
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

添加元素的时候，使用ensureCapacity()方法来保证容量的足够，如果不够时就会进行扩容，使得新容量为旧容量的1.5倍
modCount用来记录ArrayList发生变化的次数，因为每次在进行add()和addAll()时都需要调用ensureCapacity()，因此直接在ensureCapacity() 中对 modCount 进行修改。
```java
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

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

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

在进行序列化或者迭代等操作时，需要比较前后modCount是否改变，如果改变了需要抛出ConcurrentModificationException异常
```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

## ArrayList和Vector的区别
1. Vector和ArrayList几乎是完全相同的，唯一的区别在于Vector是同步的，因此开销就比ArrayList要大，访问要慢。因此最好使用ArrayList而不是Vector，因为同步完全可以由自己来进行控制
2. Vector每次扩容请求其大小的2倍，而ArrayList是1.5倍

## ArrayList和LinkedList的区别
1. ArrayList是基于动态数组实现的，LinkedList是基于双向循环链表实现的
2. ArrayList支持随机访问，LinkedList不支持
3. LinkedList在任意位置添加与删除元素更快

