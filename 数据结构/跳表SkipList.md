---
tags:
  - 链表
  - 跳表
  - 数据结构
  - 面试
  - 算法
  - cpp
---
## 引入
在学习Redis时，发现其中有序集合对象的底层数据结构使用了跳表。因此本篇笔记对跳表的实现进行学习。

## 什么是跳表
顾名思义，跳表是一种类似于链表的数据结构。更加准确地说，跳表是对有序链表的改进。跳表可以在 **O(log(n))** 时间内完成增加、删除、搜索操作的数据结构。跳表相比于树堆与红黑树，其功能与性能相当，并且跳表的代码长度相较下更短。

为方便讨论，约定有序链表默认为 **升序** 排序。

![1702370216-mKQcTt-1506_skiplist.gif](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251027224221710.gif?imageSlim)

跳表的特点，可以概括如下。

- 跳表是多层（level）链表结构；
- 跳表中的每一层都是一个有序链表，并且按照元素升序（默认）排列；
- 跳表中的元素会在哪一层出现是随机决定的，但是只要元素出现在了第**k**层，那么**k**层以下的链表也会出现这个元素；
- 跳表的底层的链表包含所有元素；
- 跳表头节点和尾节点不存储元素，且头节点和尾节点的层数就是跳表的最大层数；
- 跳表中的节点包含两个指针，一个指针指向同层链表的后一节点，一个指针指向下层链表的同元素节点。
关于更具体的描述与介绍，可以参考[跳表 - OI Wiki](https://oi-wiki.org/ds/skiplist/)

下面直接上代码，跳表的实践操作可转至[1206. 设计跳表 - 力扣（LeetCode）](https://leetcode.cn/problems/design-skiplist/description/)

## 代码实现（cpp）
```cpp
#include <iostream>
#include <vector>

// 最大层高，取 8 足够了，Redis 中取为 32
static const int MAX_LEVEL = 8;

// 节点定义
struct Node
{
    int val;
    // 记录节点在每一层的 next，next[i] 表示当前节点第 i 层的 next
    std::vector<Node *> next;

    Node(int _val) : val(_val)
    {
        next.resize(MAX_LEVEL, NULL); // 初始化 next 数组
    }
};

// 跳表
class skiplist
{
private:
	// 头节点
    Node *head;
    // 工具函数：找到每一层 i 小于目标值 target 的最大节点 pre[i]，最后 pre 中存的就是每一层小于 target 的最大节点，其实就是找每层的前驱节点。
    void find(int target, std::vector<Node *> &pre);
	// 工具函数：随机化层数
    int randomLevel();
public:
    skiplist();
    ~skiplist();

	// 查找给定元素
    bool search(int target);
	// 插入元素
    void insert(int num);
	// 删除给定元素
    bool erase(int target);
    
	// 工具函数：可视化跳表
    void print() const;
};

skiplist::skiplist()
{
    head = new Node(-1);
}

// 回收内存
skiplist::~skiplist()
{
    auto p = head;
    while (p)
    {
        Node *nexNode = p->next[0];
        delete p;
        p = nexNode;
    }
}

void skiplist::find(int target, std::vector<Node *> &pre)
{
    Node *curr = head;
    for (int i = MAX_LEVEL - 1; i >= 0; --i) // 从顶层开始，由上至下，由左至右遍历每一层
    {
        while (curr->next[i] != NULL && curr->next[i]->val < target)
        {
            curr = curr->next[i]; // 前跳
        }
        // 此时 curr 已是目标节点的前驱节点，并且处于第 0 层
        // 记录当前层的前驱节点
        pre[i] = curr;
    }
}

int skiplist::randomLevel()
{
    int level = 1;
    while (std::rand() % 2) // 通常新增元素时，每多一层的概率为 50% ，数据量大时可接近二分
    {
        level++;
    }
    return std::min(MAX_LEVEL, level); // 不要超过最大层高
}

// 查找
bool skiplist::search(int target)
{
    std::vector<Node *> pre(MAX_LEVEL);
    find(target, pre);

	// 此时 pre[0] 即第 0 层的前驱，那么 pre[0] 下一个同行元素 next[0] 就是我们要找的目标的位置
    Node *n = pre[0]->next[0];
    return n != NULL && n->val == target;
}

// 插入
void skiplist::insert(int num)
{
    std::vector<Node *> pre(MAX_LEVEL);
    find(num, pre);

    Node *newNode = new Node(num);
    int level = randomLevel();
    // For debug
    // std::cout << "number: " << num << " level: " << level << std::endl;
    for (int i = 0; i < level; i++)
    {
	    // 和正常链表的插入操作一样，不过我们需要由低至高逐层插入
        newNode->next[i] = pre[i]->next[i];
        pre[i]->next[i] = newNode;
    }
}

// 删除
bool skiplist::erase(int target)
{
    std::vector<Node *> pre(MAX_LEVEL);
    find(target, pre);

    Node *n = pre[0]->next[0];
    if (n == NULL || n->val != target)
        return false;
        
    // 如果第 i 层的前驱节点的同层后继就是当前节点 n，那么说明第 i 层存在 n
    // 由低至高逐层删除，其实逻辑上和插入差不多 
    for (int i = 0; i < MAX_LEVEL && pre[i]->next[i] == n; i++)
    {
        pre[i]->next[i] = n->next[i];
    }
    delete n;

    return true;
}

void skiplist::print() const
{
    for (int i = MAX_LEVEL - 1; i >= 0; --i)
    {
        std::cout << "Level " << i << ": ";
        Node *p = head->next[i];
        while (p)
        {
            std::cout << p->val << " ";
            p = p->next[i];
        }
        std::cout << "\n";
    }
    std::cout.flush();
}


int main()
{
    skiplist *list = new skiplist();

    for (int i = 0; i < 300; i++)
    {
        list->insert(std::rand());
    }
    
    list->print();
}
```

结果展示（数据量为300）：
![image.png](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251027231652534.webp?imageSlim)

可以看到各层数据合理分布，且呈现二分结构。