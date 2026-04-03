原题链接：[236. 二叉树的最近公共祖先 - 力扣]( [236. 二叉树的最近公共祖先 - 力扣（LeetCode）](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/description/?envType=study-plan-v2&envId=top-interview-150))

---
给定一个二叉树, 找到该树中两个指定节点（$p$ 和 $q$）的最近公共祖先。

---
## 递归法
从根节点开始向下搜索。对于当前的节点 `root`：
- **边界条件**：如果 `root` 为空，或者 `root` 本身就是 $p$ 或 $q$ 其中的一个，直接返回 `root`。
- **递归搜索**：去左子树找 $p, q$，去右子树找 $p, q$。
- **判断结果**：
    - **左右都有返回**：说明 $p$ 和 $q$ 分别分布在当前节点的左右子树中，那么当前节点 `root` 就是它们的最近公共祖先。
    - **只有一边有返回**：说明 $p$ 和 $q$ 都在同一侧子树中，先返回的那个节点就是（或者其上方有）最近公共祖先。
    - **两边都为空**：说明这一侧没找到。
```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == nullptr || root == p || root == q) {
            return root;
        }
        TreeNode* l = lowestCommonAncestor(root->left, p, q);
        TreeNode* r = lowestCommonAncestor(root->right, p, q);
        if (l && r) {
            return root;
        }
        return l ? l : r;
    }
};
```

---
## 迭代法
如果我们知道每个节点的父节点是谁，那么寻找 LCA 就变成了寻找**两个链表的首个公共节点**。
**实现步骤：**
1. **遍历整棵树**：利用 BFS 或 DFS 遍历一遍树，用一个哈希表（Map）记录每个节点的父节点，即 `parent[child] = parent_node`。
2. **记录路径**：从节点 $p$ 开始，利用哈希表不断向上寻找父节点，并将路径上的所有节点存入一个集合（Set）。
3. **寻找交点**：从节点 $q$ 开始向上寻找父节点，第一个出现在集合中的节点，就是它们的最近公共祖先。
```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */

class Solution {
public:
    // 用来记录每个节点的父节点
    unordered_map<TreeNode*, TreeNode*> parent;
    // 用来记录访问过的节点路径
    unordered_set<int> visited;

    // 1. 先进行一次 DFS 或 BFS，建立子节点到父节点的映射
    void dfs(TreeNode* root) {
        if (root->left) {
            parent[root->left] = root;
            dfs(root->left);
        }
        if (root->right) {
            parent[root->right] = root;
            dfs(root->right);
        }
    }

    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        // 根节点没有父节点，设为空
        parent[root] = nullptr;
        dfs(root);

        // 2. 从 p 开始，不断往上爬，记录 p 经过的所有祖先节点
        while (p != nullptr) {
            visited.insert(p->val);
            p = parent[p]; // 向上移动
        }

        // 3. 从 q 开始往上爬，遇到的第一个在 visited 集合里的节点就是 LCA
        while (q != nullptr) {
            if (visited.count(q->val)) {
                return q;
            }
            q = parent[q];
        }

        return nullptr;
    }
};
```