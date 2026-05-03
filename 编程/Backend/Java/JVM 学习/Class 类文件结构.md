Class 文件是一组以 8 个字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在文件之中，中间无任何分隔符。

Class  文件格式采用一种类似于 C 语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：**无符号数**和**表**。
- 无符号数是基本数据类型，以 $u1$、$u2$、$u4$、$u8$ 来分别代表 1 个字节、2 个字节、4 个字节和 8 个 字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照 UTF-8 编码构成字符串 值。
- 表是由多个无符号数或者其他表作为数据项构成的复合数据类型，为了便于区分，所有表的命名都习惯性地以“\_info”结尾。表用于描述有层次关系的复合结构的数据，整个 Class 文件本质上也可以视作是一张表，这张表由下表所示的数据项按严格顺序排列构成。

| 类型                | 名称                        | 数量                          |
| ----------------- | ------------------------- | --------------------------- |
| $u4$              | [[#1. 魔数\|magic]]         | $1$                         |
| $u2$              | [[#2. 版本\|minor_version]] | $1$                         |
| $u2$              | [[#2. 版本\|major_version]] | $1$                         |
| $u2$              | constant_pool_count       | $1$                         |
| $cp\_info$        | constant_pool             | $constant\_pool\_count - 1$ |
| $u2$              | access_flags              | $1$                         |
| $u2$              | this_class                | $1$                         |
| $u2$              | super_class               | $1$                         |
| $u2$              | interface_count           | $1$                         |
| $u2$              | interfaces                | $interfaces\_count$         |
| $u2$              | fields_count              | $1$                         |
| $field\_info$     | fields                    | $fields\_count$             |
| $u2$              | methods_count             | $1$                         |
| $method\_info$    | methods                   | $methods\_count$            |
| $u2$              | attributes_count          | $1$                         |
| $attribute\_info$ | attributes                | $attributes\_count$         |

### 1. 魔数
每个 Class 文件的头 4 个字节被称为**魔数（Magic Number）**，它的唯一作用是确定这个文件是否为 一个能被虚拟机接受的 Class 文件[^1]，而不是依据可以随意被更改的文件扩展名。[^2]

### 2. 版本
紧接着魔数的 4 个字节存储的是 Class 文件的**版本号**：第 5 和第 6 个字节是**次版本号（Minor Version）**，第 7 和第 8 个字节是**主版本号（Major Version）**。Java 版本号开始于 45，从 JDK 1.1 之后每个大版本将其加 1。

高版本的 JDK 能向下兼容以前版本的 Class  文件，但不能运行以后版本的 Class 文件，即便文件结构没有任何变化，虚拟机也会拒绝运行。

### 3. 常量池





[^1]: 很多文件格式标准中都有使用魔数来进行身份识 别的习惯，譬如图片格式，如 GIF 或者 JPEG 等在文件头中都存有魔数。

[^2]: Java 的 Class 文件魔数值为 0xCAFEBABE（咖啡宝贝），还挺有意思的。
