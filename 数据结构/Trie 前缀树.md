---
tags:
  - 数据结构
  - 算法
  - cpp
  - 前缀树
  - 面试
  - 树
---
最近在搞一个[CodeCrafter](https://app.codecrafters.io/catalog)的小项目，实现一个简单的CMD程序。有一个具体的要求就是：如今主流的控制台框架基本都拥有命令补全的功能，即不需要输入完整的命令，打出命令的前几个字母后按下 tab 键就可以自动补全以此为前缀的命令。我需要实现这个功能，而通常的做法是可以使用一种叫做前缀树的数据结构来存放命令。
## 什么是前缀树
前缀树，也可以叫字典树，英文名 trie（[traɪ] 读音和 try相同）。
顾名思义，就是一个像字典一样的树。
![一颗前缀树](https://pic-1371809842.cos.ap-chengdu.myqcloud.com/PicGo/20251119195801448.webp?imageSlim)
通常，前缀树是多叉的，且节点取值于所匹配字符串的字符集之中。

## 节点
字典树最基础的应用就是查找一个字符串是否在「字典」中出现过。而有些情况下，所查询的字符串可能是一个更长字符串的前缀，因此有必要在节点中标注字符串的终结节点（并不一定是叶子结点）。
```cpp
// 树节点
class Node
{
private:
    bool end;
    Node *next[26];
  
public:
    Node();
    bool isEnd();
    void setEnd(bool end);
    Node *findNext(char ch);
    Node *setNext(char ch);
};
Node::Node()
{
    this->end = false;
    memset(next, NULL, sizeof(this->next));
}
bool Node::isEnd()
{
    return this->end;
}
void Node::setEnd(bool end)
{
    this->end = end;
}
Node *Node::findNext(char ch)
{
    return this->next[ch - 'a'];
}
Node *Node::setNext(char ch)
{
    int idx = ch - 'a';
    if (this->findNext(ch) == NULL)
    {
        this->next[idx] = new Node();
    }
    return next[idx];
}
```

## Trie 树
```cpp
class Trie
{
private:
    Node *root;

public:
    Trie();
    ~Trie();
    void insert(const std::string &word);
    bool search(const std::string &word);
    bool startsWith(const std::string &prefix);
};

Trie::Trie()
{
    root = new Node;
}
Trie::~Trie()
{
    std::function<void(Node *)> dfs = [&](Node *p)
    {
        if (!p)
            return;
        for (int i = 0; i < 26; ++i)
            dfs(p->findNext('a' + i));
        delete p;
    };
    dfs(root);
}
// 插入字符串
void Trie::insert(const std::string &word)
{
    Node *node = root;
    for (char ch : word)
    {
        node = node->setNext(ch);
    }
    // 单词末标记为 End，这样在此前缀上追加时不会将原单词覆盖
    node->setEnd(true);
}
// 严格匹配
bool Trie::search(const std::string &word)
{
    Node *node = root;
    for (char ch : word)
    {
        node = node->findNext(ch);
        if (node == NULL)
        {
            return false;
        }
    }
    return node->isEnd();
}
// 前缀匹配
bool Trie::startsWith(const std::string &prefix)
{
    Node *node = root;
    for (char ch : prefix)
    {
        node = node->findNext(ch);
        if (node == NULL)
        {
            return false;
        }
    }
    return true;
}
```

## 一些事项
- 查找或插入一个长度为 L 的单词，访问 next 数组的次数最多为 L+1，**和 Trie 中包含多少个单词无关**。
- Trie 的每个结点中都保留着一个字母表，这是很耗费空间的。如果 Trie 的高度为 n，字母表的大小为 m，最坏的情况是 Trie 中还不存在前缀相同的单词，那空间复杂度就为 $O(n^m)$。

关于前缀树的模板练习，可以转至[208. 实现 Trie (前缀树) - 力扣（LeetCode）](https://leetcode.cn/problems/implement-trie-prefix-tree/description/)
