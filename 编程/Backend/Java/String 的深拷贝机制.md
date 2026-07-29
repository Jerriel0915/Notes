
> [!NOTE] 省流
> String 既不需要深拷贝，也不存在传统意义上的浅拷贝问题。它的**不可变性**从根本上消解了拷贝的必要性，而 JVM 在底层大量使用的**引用共享**，恰恰是其高性能与线程安全的设计精髓。

## 引入
去面试的时候被问到了这个问题：
> "String 的赋值是深拷贝还是浅拷贝？`new String("abc")` 会创建新的字符数组吗？"

一开始很简单地认为 String 是引用类型，因此回答浅拷贝，回来后仔细一想发现并不准确。这个问题精妙在设定了一个坑——**String 并不属于需要讨论深/浅拷贝的对象范畴**。

想理解这点需要从几个点展开：

## 深拷贝 VS 浅拷贝
在面向对象编程中，深/浅拷贝的概念只对一个对象有意义——**可变对象（Mutable Object）**。

| 类型      | 核心特征            | 风险             |
| ------- | --------------- | -------------- |
| **浅拷贝** | 复制引用，新旧对象共享内部数据 | 修改一方，另一方"被动"改变 |
| **深拷贝** | 递归复制对象及其所有引用成员  | 内存开销大，但完全独立    |

浅拷贝的风险建立在内部数据的可变性上，但如果对象数据本身不可变，共享引用就是**安全**的。

## String 的不可变性

String 的不可变性**（Immutable）**主要体现在三个方面：

### 1. 类层面：`final` 修饰，不可继承
```Java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    // ...
}
```

### 2. 数据层面：字符数组被 `private final` 锁定
```Java
// Java 8 及之前
public final class String {
    private final char[] value;  // 字符数组
    private final int offset;    // 起始偏移
    private final int count;     // 字符长度
    // ...
}
```

在 JDK 8 及以前，`String` 内部维护了三个字段：`value`（字符数组）、`offset`（偏移量）、`count`（长度）。这种设计允许多个 `String` 对象**共享同一个字符数组**，通过 `offset` 和 `count` 来标记不同的子串范围。

```Java
public final class String {
    private final byte[] value;  // 字节数组
    private final byte coder;    // 编码标识（LATIN1或UTF16）
    // 移除了 offset 和 count
}
```

JDK 9 引入了 Compact Strings 优化[^1]，用 `byte[]` 替代 `char[]` 以节省内存，同时移除了 `offset` 和 `count` 字段[^1]。这一变化深刻影响了子串等操作的拷贝行为。但始终不变的是，内部字节数组总是被 `private final` 保护的。

### 3. 行为层面：所有修改操作都返回新对象
比如： `concat()`、`substring()`、`replace()`、`toUpperCase()` ……没有任何方法会修改原对象的内部状态。

以 `concact()` 源码为例：
```Java
private final char value[]; 

public String concat(String str) {
	int otherLen = str.length(); 
	if (otherLen == 0) {
		//长度为0，返回原字符串
		return this; 
	} 
	int len = value.length; 
	//复制原字符串到长度为（len + otherLen）字符数组 
	char buf[] = Arrays.copyOf(value, len + otherLen); 
	str.getChars(buf, len);
	//复制目标字符串 
	return new String(buf, true); 
}
```
可以看到最终返回的是一个新 String 对象（`return new String()`）

再看构造器：
```Java
public String(String original) {
    this.value = original.value;
    this.coder = original.coder;
    this.hash = original.hash;
}
```

**所有 JDK 版本中，这个构造器都是浅拷贝**——新 `String` 对象直接引用了原对象的内部字符/字节数组。它只创建了一个新的 `String` 包装对象，但数据是共享的[^1]。

```Java
String str1 = "hello";
String str2 = new String(str1);
System.out.println(str1 == str2);        // false，不同对象
// 但内部共享同一个 char[]/byte[]
```

```Java
String s1 = "hello";
String s2 = s1;  // 引用赋值（看起来像"浅拷贝"）

s1 = "world";    // 注意：这里不是修改原对象，而是让 s1 指向新的常量池对象
System.out.println(s2); // 仍然是 "hello"
```
s1 和 s2 即使指向同一个堆内对象，也没有任何 API 可以修改它。既然无法修改，共享引用就不会产生副作用。因此，传统深/浅拷贝的区分对 String 而言失去了意义。

同时，由于 String 未实现 `Cloneable` 接口，其自身的 `clone()` 方法为 `protected`，无法在外部直接调用，即使通过反射强行调用，也因为内部数组的 `final` 字段而无法复制。

## 回答为什么？
理解了“是什么”之后，我们需要追问“为什么”。

### 1. 不可变性的核心约束
`String` 被设计为**不可变（Immutable）** 类。不可变性带来了诸多好处：
- **线程安全**：无需同步
- **哈希缓存**：`hashCode` 只需计算一次
- **安全**：作为参数传递时不会被意外修改
- **字符串常量池**：可以安全地复用

不可变性意味着：**一旦 `String` 对象创建，其内部字符数组永远不会改变**。因此，多个 `String` 对象共享同一个内部数组是**安全的**——没有人能修改这个数组的内容。这正是浅拷贝在 `String` 语境下是安全的核心原因。

### 2. 性能考量
- **浅拷贝（如 `new String(str)`）** ：O(1)时间复杂度，只复制引用
- **深拷贝（如复制数组）** ：O(n)时间复杂度，需要遍历所有字符

对于频繁的字符串操作，浅拷贝的性能优势巨大。而不可变性保证了这种优化的安全性。

### 3. JDK 版本演进的权衡
JDK 7 将 `substring` 从浅拷贝改为深拷贝，是为了**解决内存泄漏问题**——不再让一个小子串“拖住”一个大数组无法回收。但代价是 `substring` 的性能从 O(1)变成了 O(n)。这是一种**正确性优先于性能**的设计权衡。

## 最后
回到最初的问题：**Java 的 String 采用的是深拷贝吗还是浅拷贝？**

答案是：**没有一个统一的答案。** `String` 的拷贝行为取决于你使用的具体方法以及 JDK 版本。`new String(str)` 在所有版本中都是浅拷贝；`substring` 在 JDK 7 前后行为完全不同；而 `concat` 和 `+` 则是深拷贝。

更深层的启示是：**理解一个类的拷贝行为，必须深入其底层实现和设计哲学**。`String` 的不可变性使得浅拷贝安全可行，而 JDK 版本的演进则反映了工程实践中对性能与内存的持续权衡。

作为开发者，与其纠结于“String 是深拷贝还是浅拷贝”这个二元问题，不如记住：

> **对于 `String`，永远不要假设某种拷贝行为——查看源码，确认版本，按需选择。**


---
_本文基于 OpenJDK 源码及 Java 官方文档编写，如有不同 JDK 版本的行为差异，欢迎讨论指正。_

[^1]: [Java官方文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/String.html)
