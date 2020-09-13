---
title: 聊一聊String，StringBuilder，StringBuffer
date: 2020-08-24 20:16
tags: 
categories: Java
thumbnail: /thumbnail/florian-schneider-446378-unsplash.jpg
---

`String`和`StringBuilder`以及`StringBuffer`的区别应该是我经历的面试中被问到的频率最高的问题之一。这里对这三者相关的东西做个总结吧。

<!-- more -->

一般遇到上面的问题，都会脱口而出：`String`是不可变的，而`StringBuilder`和`StringBuffer`是可变的。至于`StringBuilder`和`StringBuffer`的区别则是前者是线程不安全的，而后者是线程安全的。

听了这个答案，大部分面试官也就进入下一个问题了，深入一点的面试官可能还会问到如下问题：

1. 为什么说`String`是不可变的？

2. 为什么`String`要设计成不可变的？有什么好处？
3. 为什么说`StringBuilder`和`StringBuffer`是可变的？
4. 为什么说`StringBuilder`是线程不安全的？
5. `StringBuffer`如何保证了线程安全？

首先解释下这里说的不可变的意思，不可变是指`String`的内容不可变，也就是我们如果对`String`内容进行修改，那么必然会产生一个新的`String`对象(不考虑字符串常量池)。

#### 问题一，为什么说`String`是不可变的？

查看`String`类的源码，如下：

```Java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    .............
    }
```

首先该类被`final`关键字修饰，导致该类不可被继承；其次，存放内容的`char[]`被`final`关键字修饰，那么一旦构造函数执行后，该数组引用不可指向其他数组；再者，该类中除了构造方法之外，没有其他地方直接对`value []`的值进行修改。这三点保证了`String`类是不可变的。

#### 问题二，为什么`String`要设计成不可变的？有什么好处？

这种设计上的问题可以去`so`找一找答案。我找到了一个高赞答案总结下：

1. 安全性。`String`变量常代表网络地址，用户名和密码等等作为参数传递。如果设计为可变的，这些参数很容易改变，容易导致代码安全问题。

2. 同步和并发。将`String`作为不变的，天然解决了并发问题。

3. 字符串常量池考虑。如果`String`可变，那么常量池就没了意义。这里举个例子：

   ```java
   String fooOne = "foo";
   String fooTwo = "foo";
   // 上面定义了两个变量。因为字符串常量池的设计，fooOne和fooTwo将指向同一内存地址。
   此时，我们进行如下操作：
   fooTwo = "fooTwo";
   // 此时，假如不存在String不可变的特性，那么会出现fooOne的内容也会变成"fooTwo"的情况，这显然是会出问题的，而String的不变性则解决了这个问题。
   ```

4. 类加载。`String`被作为类加载的参数，如果它是可变的话，很容易导致错误的类被加载。

#### 问题三，为什么说`StringBuilder`和`StringBuffer`是可变的？

因为这两者源码几乎一模一样，所以这里拿`StringBuilder`举例子。查看`StringBuilder`源码，发现其继承于`AbstractStringBuilder`:

```Java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;
    ......
}
```

可以发现，`char[]`没有被`final`修饰，并且多了个`count`属性用来表示已经使用的`char`字符的数量，很容易让人想到扩容机制。

再看`StringBuilder`的方法，多了一些诸如：`append()`，`insert()`，`delete()`等等方法，这也是支撑`StringBuilder`和`StringBuffer`可变的方法。

看看`append`方法的源码，其余方法原理都差不多：

```java
    @Override
    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
```

内部调用了父类的方法：

```java
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
```

首先调用 `ensureCapacityInternal(count + len)`确保`char[]`的长度足够容纳所有字符，看一下内部实现:

```Java
    private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        if (minimumCapacity - value.length > 0) {
            value = Arrays.copyOf(value,
                    newCapacity(minimumCapacity));
        }
    }

 private int newCapacity(int minCapacity) {
        // overflow-conscious code
        // 首先将现有的容量*2再加上2得到一个新的容量
        int newCapacity = (value.length << 1) + 2;
  
       // 如果新容量还是不够容纳 则直接使用minCapacity
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
   
   // 当newCapacity越界或者大于了MAX_ARRAY_SIZE(Integer.MAX_VALUE - 8) 
        return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }

    private int hugeCapacity(int minCapacity) {
      // 这个判断其实我有疑问，Inter.MAX_VALUE已经是最大的Int值，怎么还存在<0 的情况，实际上当minCapacity越界时，这个等式是成立的
        if (Integer.MAX_VALUE - minCapacity < 0) { // overflow
            throw new OutOfMemoryError();
        }
        return (minCapacity > MAX_ARRAY_SIZE)
            ? minCapacity : MAX_ARRAY_SIZE;
    }
```

确保`char[]`长度足够之后，调用了`String`的`getChars`方法。该方法注释为，

> Copies characters from this string into the destination character array.

也就是将自身的部分字符拷贝到指定`char`数组中，很好理解。

拷贝完成之后，`char`数组中的值已经得到了修改。

之后，更新`count`的值，`append`方法结束。

分析完部分源码，可以回答第三个问题，为什么`StringBuilder`和`StringBuffer`是可变的。因为在他们内部，`char[]`并没有被`final`修饰，因此可以在扩容之后，指向新的char数组 ；并且在追加新的字符时，会直接修改`char[]`的值。因此是可变的。

#### 问题四，为什么说`StringBuilder`是线程不安全的？

先写个代码测试一下：

```Kotlin
    fun main(args: Array<String>) {
        val sb = StringBuilder()

        for (i in 0 until 10){
            thread {
                for (j in 0 until 1000){
                    sb.append("1")
                }
            }
        }

        Thread.sleep(2000)
        println(sb.length)
    }
```

分别开`10`个线程，对`StringBuilder`进行`1000`次的`append`操作。如果`StringBuilder`是线程安全的，那么打印出来的长度应该是`10000`.

可是实际上的结果是：

1. 每次打印的结果均小于`10000`.
2. 部分测试用例，还会存在 爆出`java.lang.ArrayIndexOutOfBoundsException`这个错误的情况。

先分析**1**，为什么每次打印的结果均小于`10000`.

试想这样一种情况，`AB`两个线程刚好同时对`value[]`进行了`append`操作，此时双方都到了更新`count`的代码处，假如此时，`count`的值为`100`，那么`count`最终的结果理应为`102`。可是，由于没有进行同步，两个线程几乎同时拿到`100`的值，然后分别对其`+1`操作，然后再设置`count`的值为`101`，因此，本身`append`了`2`次的值却只有一次生效了。也就导致最终使`StringBuilder`的`length`小于预期情况。

再分析**2**，为什么会出现爆出`java.lang.ArrayIndexOutOfBoundsException`的情况。

看了下，这个报错出现在`str.getChars(0, len, value, count)`方法内部，具体在`System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin)`这一行代码中。

先画个图理解下`System.arraycopy`方法：

![](https://i.loli.net/2020/08/25/9GVPnfJum7EsDCy.png)

根据图片概括就是，`src`从`srcPos`开始往`des`的`desPos`处复制`length`长度的数据。而这个`desPos`对应我们的`count`。因为处于多线程环境中，可能另一个线程刚好完成了`append`，使得`count+1`，因此，拿到的`count`就预期的多了`1`，导致了这个`Exception`。

#### 问题五，`StringBuffer`如何保证了线程安全？

这个看了源码就很好回答了。

`StringBuffer`通过对方法使用`synchronized`关键字加锁来保证线程安全。

### 结尾

`StringBuild`看似源码比较简单，但是源码中对`overflow`的处理以及对`System.copy`函数的灵活使用，还是很值得学习和借鉴的。