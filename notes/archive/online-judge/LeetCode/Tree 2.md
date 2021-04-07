---
title: "LeetCode 树（2）"
date: 2019-07-18T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 树（2）](https://leetcode-cn.com/tag/tree/)

## 题目

### 3. 二叉搜索树

#### [95 不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

生成由 1 ... n 为节点所组成的二叉搜索树。

为了构造以 i 为根节点的二叉搜索树，我们需要先构造以 1 ... i - 1 为左子树的所有二叉搜索树与以 i + 1 ... n 为右子树的所有二叉搜索树，再将这些子树排列组合得到以 i 为根节点的所有二叉搜索树。

```c++
class Solution {
public:
    vector<TreeNode *> generateTrees(int n) {
        if (n == 0)
            return {};
        return Generate(1, n);
    }

    vector<TreeNode *> Generate(int m, int n) {
        vector<TreeNode *> nodes;
        if (m == n)
            nodes.push_back(new TreeNode(n));
        if (m > n)
            nodes.push_back(nullptr);
        if (m >= n)
            return nodes;
        for (int i = m; i <= n; ++i) {
            vector<TreeNode *> left = Generate(m, i - 1);
            vector<TreeNode *> right = Generate(i + 1, n);
            for (auto &l:left)
                for (auto &r:right) {
                    TreeNode *node = new TreeNode(i);
                    node->left = l, node->right = r;
                    nodes.push_back(node);
                }
        }
        return nodes;
    }
};
```

#### [98 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

因为二叉搜索树的中序遍历结果是一个有序数组，所以一种方法是将中序遍历的结果保存下来进行判断，也可以根据二叉搜索树的定义判断子节点和根节点的大小关系。

```c++
class Solution {
public:
    bool isValidBST(TreeNode *root) {
        return IsValidSubtree(root, nullptr, nullptr);
    }

    bool IsValidSubtree(TreeNode *node, const int *min_val, const int *max_val) {
        if (!node)
            return true;
        if ((min_val && *min_val >= node->val) || (max_val && *max_val <= node->val))
            return false;
        return IsValidSubtree(node->left, min_val, &(node->val)) && IsValidSubtree(node->right, &(node->val), max_val);
    }
};
```

#### [108 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

给一个有序数组，将其转换为一棵平衡二叉搜索树。

二叉搜索树的中序遍历结果即为有序数组，所以只需要每次找到中间的元素作为根节点，左边的子数组作为左子树，右边的子数组作为右子树，递归构造即可。

```c++
class Solution {
public:
    TreeNode *sortedArrayToBST(vector<int> &nums) {
        return Build(nums, 0, nums.size());
    }

    TreeNode *Build(vector<int> &nums, int x, int y) {
        if (x >= y)
            return nullptr;
        int pos = x + (y - x) / 2;
        TreeNode *node = new TreeNode(nums[pos]);
        node->left = Build(nums, x, pos);
        node->right = Build(nums, pos + 1, y);
        return node;
    }
};
```

#### [235 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

找到一个二叉搜索树中两个指定节点的最近公共祖先。

由二叉搜索树可知，如果两个节点的值都大于根节点，那么他们都应该在根节点的右子树上；如果两个节点的值都小于根节点，那么他们都应该在根节点的左子树上；否则他们可能在根节点及其子树上的任意位置，那么根节点即是他们的最近公共祖先。

```c++
class Solution {
public:
    TreeNode *lowestCommonAncestor(TreeNode *root, TreeNode *p, TreeNode *q) {
        if (!root)
            return nullptr;
        else if (root->val > p->val && root->val > q->val)
            return lowestCommonAncestor(root->left, p, q);
        else if (root->val < p->val && root->val < q->val)
            return lowestCommonAncestor(root->right, p, q);
        return root;
    }
};
```

#### [671 二叉树中第二小的节点](https://leetcode-cn.com/problems/second-minimum-node-in-a-binary-tree/)

给一个二叉树，其子节点数量只为 0 或 2，并且根节点的值一定小于等于子节点的值，找到所有节点中的第二小的值。

因为二叉树上根节点一定小于等于子节点，所以整个树的根节点的值一定是最小值，只需要遍历整个树，找到除根节点之外的最小值即可。

```c++
class Solution {
public:
    int findSecondMinimumValue(TreeNode *root) {
        if (!root)
            return -1;
        int first = root->val, *res = nullptr;
        queue<TreeNode *> q;
        q.push(root);
        while (!q.empty()) {
            TreeNode *node = q.front();
            q.pop();
            if (first < node->val)
                if (!res)
                    res = new int(node->val);
                else
                    *res = min(*res, node->val);
            if (node->left && node->right)
                q.push(node->left), q.push(node->right);
        }
        return res ? *res : -1;
    }
};
```

#### [230 二叉搜索树中第 K 小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

找到一个二叉搜索树中第 k 小的元素。

因为二叉搜索树的中序遍历结果是有序的，我们可以使用中序遍历并在找到第 k 小的元素时提前终止并返回结果。

```c++
class Solution {
    int res;
public:
    int kthSmallest(TreeNode *root, int k) {
        res = 0;
        Inorder(root, k);
        return res;
    }

    void Inorder(TreeNode *root, int &k) {
        if (!root)
            return;
        Inorder(root->left, k);
        if (k <= 0)
            return;
        --k;
        if (k == 0) {
            res = root->val;
            return;
        }
        Inorder(root->right, k);
    }
};
```

#### [450 删除二叉搜索树中的节点](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

给一个二叉搜索树和一个值，删除二叉搜索树中的对应节点。

根据二叉搜素树的定义，很容易通过大小关系找到对应节点，找到之后只需要将原先的节点替换为左子树上的最大节点，也就是左子节点的最右子节点即可，注意要将左子节点的最右子节点的左子树接到其父节点的右子节点上。

```c++
class Solution {
public:
    TreeNode *deleteNode(TreeNode *root, int key) {
        if (!root)
            return nullptr;
        if (root->val == key) {
            if (!root->left)
                return root->right;
            auto node = root->left, head = node;
            if (!node->right) {
                node->right = root->right;
                return node;
            }
            while (node->right)
                head = node, node = node->right;
            head->right = node->left;
            node->left = root->left, node->right = root->right;
            return node;
        } else if (root->val < key)
            root->right = deleteNode(root->right, key);
        else if (root->val > key)
            root->left = deleteNode(root->left, key);
        return root;
    }
};
```

#### [669 修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)

给一个二叉搜索树，以及最小边界 L 和最大边界 R，修剪二叉搜索树使得所有节点值都在 [L, R] 的范围内。

如果一个节点的值在范围外，根据二叉搜索树的定义，返回对应方向的节点修剪后的结果即可；如果一个节点的值在范围内，对其左右子树分别进行修建即可。

```c++
class Solution {
public:
    TreeNode *trimBST(TreeNode *root, const int &L, const int &R) {
        if (!root)
            return nullptr;
        if (root->val < L)
            return trimBST(root->right, L, R);
        if (root->val > R)
            return trimBST(root->left, L, R);
        root->left = trimBST(root->left, L, R);
        root->right = trimBST(root->right, L, R);
        return root;
    }
};
```

#### [530 二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)

求一个二叉搜索树树中任意两节点的差的绝对值的最小值。

因为二叉搜索树的中序遍历结果是有序的，任意两节点的差的绝对值的最小值一定产生在相邻的两个值之间，因此做一次中序遍历，同时更新两节点的差的绝对值的最小值即可。

class Solution {
    TreeNode *node;
    int res;
public:
    int getMinimumDifference(TreeNode *root) {
        node = nullptr;
        res = INT_MAX;
        Inorder(root);
        return res;
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (!node)
            node = root;
        else
            res = min(res, abs(node->val - root->val));
        node = root;
        Inorder(root->right);
    }
};

#### [783 二叉搜索树结点最小距离](https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes/)

求一个二叉搜索树树中任意两节点的差的绝对值的最小值。

同上。

class Solution {
    TreeNode *node;
    int res;
public:
    int minDiffInBST(TreeNode *root) {
        node = nullptr;
        res = INT_MAX;
        Inorder(root);
        return res;
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (!node)
            node = root;
        else
            res = min(res, abs(node->val - root->val));
        node = root;
        Inorder(root->right);
    }
};

#### [501 二叉搜索树中的众数](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/)

找出一个二叉搜索树中的所有众数。

因为二叉搜索树的中序遍历结果是有序的，可以直接进行一次中序遍历，同时更新结果数组。

```c++
class Solution {
    vector<int> res;
    TreeNode *node;
    int n, m;
public:
    vector<int> findMode(TreeNode *root) {
        res = vector<int>();
        node = nullptr;
        n = m = 0;
        Inorder(root);
        return res;
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (!node || node->val != root->val)
            m = 1;
        else
            ++m;
        node = root;
        if (m > n) {
            n = m;
            res.clear();
        }
        if (m == n)
            res.push_back(node->val);
        Inorder(root->right);
    }
};
```

#### [538 把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)

把一个二叉搜索树转换成为累加树。

按照二叉搜索树的定义，每个节点值一定比右子树上的节点值小，所以按照右中左的顺序遍历整个树，同时在根节点处加上右边的累积值。

```c++
class Solution {
    int val;
public:
    TreeNode *convertBST(TreeNode *root) {
        val = 0;
        Accumulate(root);
        return root;
    }

    void Accumulate(TreeNode *root) {
        if (!root)
            return;
        Accumulate(root->right);
        root->val += val;
        val = root->val;
        Accumulate(root->left);
    }
};
```

#### [700 二叉搜索树中的搜索](https://leetcode-cn.com/problems/search-in-a-binary-search-tree/)

在二叉搜索树中搜索一个特定值。

按照二叉搜索树的特性搜索即可。

```c++
class Solution {
public:
    TreeNode *searchBST(TreeNode *root, const int &val) {
        if (!root)
            return nullptr;
        if (root->val < val)
            return searchBST(root->right, val);
        else if (root->val > val)
            return searchBST(root->left, val);
        else
            return root;
    }
};
```

#### [701 二叉搜索树中的插入操作](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

在一个二叉搜索树中插入一个值。

按照给定值与节点的值的大小关系依次往下搜索直到找到空节点，新建一个节点并返回即可。

```c++
class Solution {
public:
    TreeNode *insertIntoBST(TreeNode *root, const int &val) {
        if (!root)
            return new TreeNode(val);
        else if (val < root->val)
            root->left = insertIntoBST(root->left, val);
        else if (val > root->val)
            root->right = insertIntoBST(root->right, val);
        return root;
    }
};
```

#### [938 二叉搜索树的范围和](https://leetcode-cn.com/problems/range-sum-of-bst/)

给一个二叉搜索树，计算 L 和 R 之间的所有结点的值的和。

判断根节点的值 L <= val <= R 即可。

```c++
class Solution {
public:
    int rangeSumBST(TreeNode* root, const int &L, const int &R) {
        if (!root)
            return 0;
        int left = 0, right = 0;
        if (root->val >= L)
            left = rangeSumBST(root->left, L, R);
        if (root->val <= R)
            right = rangeSumBST(root->right, L, R);
        return left + right + (root->val >= L && root->val <= R ? root->val : 0);
    }
};
```

#### [99 恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree/)

恢复一个有两个节点被错误地交换的二叉搜索树。

因为只有两个节点被错误地交换，只需要做一次中序遍历就能从相邻节点的大小关系找到这两个节点，将第一次出现错误位置关系的前一个节点与第二次出现错误位置关系的后一个节点交换即可。

```c++
class Solution {
    TreeNode *first, *second, *prev;
public:
    void recoverTree(TreeNode *root) {
        first = second = prev = nullptr;
        Inorder(root);
        swap(first->val, second->val);
    }
    
    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (prev && prev->val >= root->val && !first)
            first = prev;
        if (prev && prev->val >= root->val && first)
            second = root;
        prev = root;
        Inorder(root->right);
    }
};
```