### 简介
[并查集](https://oi-wiki.org/ds/dsu/)是一种用于管理元素所属集合的数据结构，实现为一个森林，其中每棵树表示一个集合，树中的节点表示对应集合中的元素．

顾名思义，并查集支持两种操作：

- 合并（Unite）：合并两个元素所属集合（合并对应的树）．
- 查询（Find）：查询某个元素所属集合（查询对应的树的根节点），这可以用于判断两个元素是否属于同一集合．

---
### 代码实现
```cpp
#include <vector>
#include <numeric>

class DSU {
private:
    std::vector<int> parent;
    std::vector<int> rank;
    int num_sets;

public:
    // Initialize DSU with 'n' elements
    DSU(int n) {
        parent.resize(n);
        // Initially, every element is its own parent (a set of one)
        std::iota(parent.begin(), parent.end(), 0);
        rank.assign(n, 0);
        num_sets = n;
    }

    // Find the representative (root) of the set containing 'i'
    // Uses Path Compression for O(alpha(n)) amortized time
    int find(int i) {
        if (parent[i] == i)
            return i;
        return parent[i] = find(parent[i]);
    }

    // Unites the set containing 'i' and the set containing 'j'
    // Uses Union by Rank to keep the tree flat
    void unite(int i, int j) {
        int root_i = find(i);
        int root_j = find(j);

        if (root_i != root_j) {
            if (rank[root_i] < rank[root_j]) {
                parent[root_i] = root_j;
            } else if (rank[root_i] > rank[root_j]) {
                parent[root_j] = root_i;
            } else {
                parent[root_i] = root_j;
                rank[root_j]++;
            }
            num_sets--;
        }
    }

    // Check if i and j belong to the same set
    bool connected(int i, int j) {
        return find(i) == find(j);
    }

    // Return the number of disjoint sets
    int count() const {
        return num_sets;
    }
};
```

#### 时间复杂度
$$
O(\alpha(N))
$$
可以认为是 $O(1)$ 常数级复杂度。

#### 空间复杂度
$$
O(N)
$$

### 启发式合并
当合并 $A$  $B$ 两颗树时，无非两种选项：
- 将树 $A$ 连接到树 $B$ 的根节点。
- 将树 $B$ 附加到树 $A$ 的根节点。

如果采取随机连接方式，运气差时树的高度会不断增加，甚至退化为类似链表的结构，此时 `find()` 操作将花费接近 $O(N)$ 的时间。为了避免这种情况，可以采取启发式合并，即：总是把**较矮的树**接在**高树**下面。这样合并后的树高度为

```cpp
rank = (rank(A) == rank(B) ? max(rank(A), rank(B)) : rank(A) + 1)
```

只有在两棵树高度相同时，合并后树的高度才会增加。