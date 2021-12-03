# 阅读 JDK 源码步骤

1.先读接口代码。包括接口的定义、各个方法的定义说明。

2.读实现类的主要方法，有两个主线：构造器方法、常用方法

在Java里，接口通常意味着一种标准或者是协议。一个接口有多种实现类，它们按照接口的标准来实现。因此先理解接口的方法，再看实现会更容易理解

# ArrayList

## 类图

![image-20211203110644198](JDK1.8util.assets/image-20211203110644198.png)

从类图可以看到 ArrayList 实现了很多接口，其中 Cloneabled、RandomAccess、Serializable是空的，可以略过。主要看List接口

## 构造函数

ArrayList 有 3 个构造函数。

```java
    //构造一个具有指定初始容量的空数组
    //当指定容量大于0，则按指定容量构造Object;如果指定容量等于0，则构造初始容量为10的Object；如果指定容量小于0，则抛IllegalArgumentException异常
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

    //未指定初始容量时，构造一个初始容量为 10 的空数组
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    //按照集合的迭代器返回的顺序构造一个包含指定集合元素的数组
    //当传入的Collection不为空时，复制一个新数组；若为空，则构造一个初始容量10的空数组
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

## 扩容

 扩容方法：grow() ，得到一个新数组，把旧数组复制到新数组里。

新数组的容量怎么得到呢？

```java
int newCapacity = oldCapacity + (oldCapacity >> 1); 
```

oldCapacity >> 1 意思是右移一位，高位补 0，效果等同为 oldCapacity/2，因此新数组长度为旧数组长度的 1.5 倍。

## add

add将元素添加到列表的末尾，如果指定位置，那么该位置上的元素和后面的元素全部后移一位。添加元素先进行容量检查，如果超出容量，先扩容，再添加元素。

详解

```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }
```

为确保元素能添加，调用 ensureCapacityInternal（）对容量进行检查，size+1 为添加本元素后的容量

```java
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
```

在这个函数里调用了 ensureExplicitCapacity()

```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

modCount的作用是记录list被修改的次数，每次增加或删除都会更新。

如果minCapacity大于数组的容量，就扩容。

它将calculateCapacity（）的返回值作为入参

```java
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

## remove

移除指定位置的元素，将该位置后面的所有元素全部前移一位。