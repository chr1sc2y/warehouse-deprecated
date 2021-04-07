---
title: "LeetCode 树（3）"
date: 2019-08-24T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 树（3）](https://leetcode-cn.com/tag/tree/)

## 题目

### 4. 递归求解

#### [617 合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/)

合并两个二叉树。

判断各个节点是否存在，全部合并到一棵树上即可。

```c++
class Solution {
public:
    TreeNode *mergeTrees(TreeNode *t1, TreeNode *t2) {
        if (!t1 && !t2)
            return nullptr;
        else if (!t1)
            return t2;
        else if (!t2)
            return t1;
        t1->val += t2->val;
        t1->left = mergeTrees(t1->left, t2->left);
        t1->right = mergeTrees(t1->right, t2->right);
        return t1;
    }
};
```

#### [226 翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

翻转一个二叉树。

先将左右子树分别翻转，再交换两者的位置。

```c++
class Solution {
public:
    TreeNode *invertTree(TreeNode *root) {
        if (!root)
            return nullptr;
        TreeNode *left = invertTree(root->left), *right = invertTree(root->right);
        root->right = left;
        root->left = right;
        return root;
    }
};
```

#### [104 二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

找出一个二叉树的最大深度。

每层深度为 1，加上左右子树中更大的深度即为最大深度。

```c++
class Solution {
public:
    int maxDepth(TreeNode *root) {
        if (!root)
            return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};
```

#### [965 单值二叉树](https://leetcode-cn.com/problems/univalued-binary-tree/)

判断一个二叉树是否是一个单值二叉树。

判断每个节点与其左右节点的值是否相同即可。

```c++
class Solution {
public:
    bool isUnivalTree(TreeNode *root) {
        if (!root)
            return true;
        return (root->left ? root->val == root->left->val : true) && (root->right ? root->val == root->right->val : true) && isUnivalTree(root->left) && isUnivalTree(root->right);
    }
};
```

#### [559 N叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)

找到一个 N 叉树的最大深度。

每层深度为 1，加上其所有子树中最大的深度即为最大深度。

```c++
class Solution {
public:
    int maxDepth(Node* root) {
        if (!root)
            return 0;
        int depth = 0;
        for (auto &c:root->children)
            depth = max(depth, maxDepth(c));
        return 1 + depth;
    }
};
```

#### [563 二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt/)

计算一个二叉树的坡度。

对于每个节点，计算其左子树和右子树的和，将其差的绝对值加到总的坡度上，再返回左子树，右子树，与自己的值的和，递归调用即可。

```c++
class Solution {
    int res;
public:
    int findTilt(TreeNode *root) {
        res = 0;
        CalcTilt(root);
        return res;
    }

    int CalcTilt(TreeNode *node) {
        if (!node)
            return 0;
        int left = CalcTilt(node->left);
        int right = CalcTilt(node->right);
        res += abs(left - right);
        return node->val + left + right;
    }
};
```

#### [508 出现次数最多的子树元素和](https://leetcode-cn.com/problems/most-frequent-subtree-sum/submissions/)

找出一个二叉树中出现次数最多的子树元素和。

计算出一个节点的左子树和右子树的子树元素和，加上自身的值就是一个完整的子树元素和，递归调用计算所有的节点并计数即可。

```c++
class Solution {
    unordered_map<int, int> count;
public:
    vector<int> findFrequentTreeSum(TreeNode *root) {
        vector<int> res;
        count = unordered_map<int, int>();
        Traverse(root);
        int n = 0;
        for (auto &c:count) {
            if (c.second > n) {
                n = c.second;
                res.clear();
                res.push_back(c.first);
            } else if (c.second == n)
                res.push_back(c.first);
        }
        return res;
    }

    int Traverse(TreeNode *root) {
        if (!root)
            return 0;
        int val = Traverse(root->left) + Traverse(root->right) + root->val;
        ++count[val];
        return val;
    }
};
```

### 5. 栈求解

#### [623 在二叉树中增加一行](https://leetcode-cn.com/problems/add-one-row-to-tree/)

给一个二叉树，在第 d 层追加一行值为 v 的节点。

用一个栈保存一层的所有节点，逐层遍历即可。注意 d = 1 时要单独处理。

```c++
class Solution {
public:
    TreeNode *addOneRow(TreeNode *root, int v, int d) {
        if (!root)
            return nullptr;
        if (d == 1) {
            TreeNode *new_root = new TreeNode(v);
            new_root->left = root;
            return new_root;
        }
        queue<TreeNode *> q;
        q.push(root);
        int depth = 1, n = 1;
        while (!q.empty() && depth < d) {
            for (int i = 0; i < n; ++i) {
                TreeNode *node = q.front();
                q.pop();
                if (depth == d - 1) {
                    TreeNode *left = node->left, *right = node->right;
                    node->left = new TreeNode(v);
                    node->right = new TreeNode(v);
                    node->left->left = left;
                    node->right->right = right;
                }
                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);
            }
            ++depth;
            n = q.size();
        }
        return root;
    }
};
```

### 6. 找节点

#### [1123. 最深叶节点的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-deepest-leaves/)

找到一个二叉树最深的叶节点的最近公共祖先。

可以先用层序遍历找到二叉树的深度，再通过一次递归找到所有叶节点的公共祖先。

```c++
class Solution {
    TreeNode *res;
    int lvl;
public:
    TreeNode *lcaDeepestLeaves(TreeNode *root) {
        if (!root)
            return nullptr;
        res = nullptr;
        lvl = 0;
        queue<TreeNode *> nodes;
        nodes.push(root);
        int n = 1;
        while (!nodes.empty()) {
            for (int i = 0; i < n; ++i) {
                TreeNode *node = nodes.front();
                nodes.pop();
                if (node->left) nodes.push(node->left);
                if (node->right) nodes.push(node->right);
            }
            ++lvl;
            n = nodes.size();
        }
        FindLCA(root, 1);
        return res;
    }

    bool FindLCA(TreeNode *root, int l) {
        if (root && l == lvl) {
            res = root;
            return true;
        } else if (!root)
            return false;
        bool left = FindLCA(root->left, l + 1), right = FindLCA(root->right, l + 1);
        if (left && right)
            res = root;
        return left || right;
    }
};
```

但实际上我们并不需要知道这棵树的深度，只需要知道最深的节点即是叶节点，并且如果一个节点的左子树和右子树的最深节点的深度相同，那么这个节点就是他们的最近公共祖先，返回这个节点即可。

```c++
class Solution {
public:
    TreeNode *lcaDeepestLeaves(TreeNode *root) {
        return FindLCA(root).first;
    }

    pair<TreeNode *, int> FindLCA(TreeNode *root) {
        if (!root)
            return pair<TreeNode *, int>(nullptr, 0);
        auto left = FindLCA(root->left), right = FindLCA(root->right);
        if (left.second > right.second)
            return pair<TreeNode *, int>(left.first, left.second + 1);
        if (left.second < right.second)
            return pair<TreeNode *, int>(right.first, right.second + 1);
        return pair<TreeNode *, int>(root, left.second + 1);
    }
}
```
