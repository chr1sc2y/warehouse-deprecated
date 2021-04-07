---
title: "LeetCode 树（1）"
date: 2019-07-13T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 树（1）](https://leetcode-cn.com/tag/tree/)

## 题目

### 1. 树的遍历

#### [144 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

前序遍历一个二叉树。

前序遍历是按照根节点，左子节点，右子节点的顺序来遍历一个二叉树，有递归和迭代两种方法。对于迭代方法，先将节点加入结果数组，然后用一个栈保存右，左子节点，依次访问，重复此过程。

```c++
class Solution {
        vector<int> res;
public:
    vector<int> preorderTraversal(TreeNode *root) {
        res = vector<int>();
        Preorder(root);
        return res;
    }
    
    void Preorder(TreeNode *root) {
        if (!root)
            return;
        res.push_back(root->val);
        Preorder(root->left);
        Preorder(root->right);
    }
};
```

```c++
class Solution {
public:
    vector<int> preorderTraversal(TreeNode *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<TreeNode *> pre;
        pre.push(root);
        while (!pre.empty()) {
            TreeNode *node = pre.top();
            pre.pop();
            res.push_back(node->val);
            if (node->right)
                pre.push(node->right);
            if (node->left)
                pre.push(node->left);
        }
        return res;
    }
};
```

#### [589 N叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)

前序遍历一个 N 叉树。

跟二叉树的前序遍历类似，有递归和迭代两种做法。

```c++
class Solution {
    vector<int> res;
public:
    vector<int> preorder(Node *root) {
        res = vector<int>();
        Preorder(root);
        return res;
    }
    
    void Preorder(Node *root) {
        if (!root)
            return;
        res.push_back(root->val);
        for (auto &c:root->children)
            Preorder(c);
    }
};
```

```c++
class Solution {
public:
    vector<int> preorder(Node *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<Node *> pre;
        pre.push(root);
        while (!pre.empty()) {
            Node *node = pre.top();
            pre.pop();
            res.push_back(node->val);
            for (auto iter = node->children.rbegin(); iter < node->children.rend(); ++iter)
                pre.push(*iter);
        }
        return res;
    }
};
```

#### [94 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

中序遍历一个二叉树。

中序遍历是按照左子节点，根节点，右子节点的顺序来遍历一个二叉树，有递归和迭代两种方法。对于迭代方法，用一个栈保存父节点以便最后访问，先找到最左节点，将其加入结果数组，然后访问其右子节点，重复此过程。

```c++
class Solution {
    vector<int> res;
public:
    vector<int> inorderTraversal(TreeNode* root) {
        res = vector<int>();
        Inorder(root);
        return res;
    }
    
    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        res.push_back(root->val);
        Inorder(root->right);
    }
};
```

```c++
class Solution {
public:
    vector<int> inorderTraversal(TreeNode *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<TreeNode *> in;
        TreeNode *node = root;
        while (node || !in.empty()) {
            while (node) {
                in.push(node);
                node = node->left;
            }
            if (!in.empty()) {
                node = in.top();
                in.pop();
                res.push_back(node->val);
                node = node->right;
            }
        }
        return res;
    }
};
```

#### [145 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

后序遍历一个二叉树。

后序遍历是按照左子节点，右子节点，根节点的顺序来遍历一个二叉树，有递归和迭代两种方法。对于迭代方法，用一个栈保存父节点以便最后访问，先找到最左节点，如果其已经是叶子节点则将其加入结果数组，否则访问其右子节点，用 last 保存上一次访问过的右子节点防止再次访问，重复此过程。

```c++
class Solution {
    vector<int> res;
public:
    vector<int> postorderTraversal(TreeNode* root) {
        res = vector<int>();
        Postorder(root);
        return res;
    }
    
    void Postorder(TreeNode *root) {
        if (!root)
            return;
        Postorder(root->left);
        Postorder(root->right);
        res.push_back(root->val);
    }
};
```

```c++
class Solution {
public:
    vector<int> postorderTraversal(TreeNode *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<TreeNode *> post;
        TreeNode *node = root, *last = nullptr;
        while (node || !post.empty()) {
            while (node) {
                post.push(node);
                node = node->left;
            }
            if (!post.empty()) {
                node = post.top();
                if (node->right && node->right != last)
                    node = node->right;
                else {
                    res.push_back(node->val);
                    post.pop();
                    last = node;
                    node = nullptr;
                }
            }
        }
        return res;
    }
};
```

#### [590 N叉树的后序遍历](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)

跟二叉树的后序遍历类似，有递归和迭代两种做法。

```c++
class Solution {
    vector<int> res;
public:
    vector<int> postorder(Node *root) {
        res = vector<int>();
        Postorder(root);
        return res;
    }
    
    void Postorder(Node *root) {
        if (!root)
            return;
        for (auto &c:root->children)
            Postorder(c);
        res.push_back(root->val);
    }
};
```

```c++
class Solution {
public:
    vector<int> postorder(Node *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<Node *> post;
        post.push(root);
        while (!post.empty()) {
            Node *node = post.top();
            post.pop();
            res.push_back(node->val);
            for (auto &c:node->children)
                post.push(c);
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

#### [102 二叉树的层次遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

给一个二叉树，返回其按层次遍历的节点值。

用一个队列来保存该二叉树当前一层的所有节点，一边将这些节点 pop 并 push 进结果数组中，一边将这些节点的子节点 push 进队列。

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode *root) {
        vector<vector<int>> res;
        if (!root)
            return res;
        queue<TreeNode *> q;
        q.push(root);
        int n = 1;
        while (!q.empty()) {
            vector<int> lvl;
            for (int i = 0; i < n; ++i) {
                TreeNode *node = q.front();
                q.pop();
                lvl.push_back(node->val);
                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);
            }
            res.push_back(lvl);
            n = q.size();
        }
        return res;
    }
};
```

#### [107 二叉树的层次遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

给一个二叉树，返回其节点值自底向上的层次遍历。

跟上一题相同，只需要将结果数组倒置，或用一个栈保存结果，再 pop 进结果数组即可。

```c++
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode *root) {
        vector<vector<int>> res;
        if (!root)
            return res;
        queue<TreeNode *> q;
        int n = 1;
        q.push(root);
        while (!q.empty()) {
            vector<int> lvl;
            for (int i = 0; i < n; ++i) {
                TreeNode *node = q.front();
                q.pop();
                lvl.push_back(node->val);
                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);
            }
            res.push_back(lvl);
            n = q.size();
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

#### [429 N 叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

层序遍历一个 N 叉树。

用一个队列来保存该 N 叉树当前一层的所有节点，一边将这些节点 pop 并 push 进结果数组中，一边将这些节点的子节点 push 进队列。

```c++
class Solution {
public:
    vector<vector<int>> levelOrder(Node *root) {
        vector<vector<int>> res;
        if (!root)
            return res;
        queue<Node *> q;
        q.push(root);
        int n = 1;
        while (!q.empty()) {
            vector<int> lvl;
            for (int i = 0; i < n; ++i) {
                Node *node = q.front();
                q.pop();
                lvl.push_back(node->val);
                for (auto &c:node->children)
                    q.push(c);
            }
            res.push_back(lvl);
            n = q.size();
        }
        return res;
    }
};
```

#### [987 二叉树的垂序遍历](https://leetcode-cn.com/problems/vertical-order-traversal-of-a-binary-tree/)

垂序遍历一个二叉树。

用红黑树把二叉树中每个节点的位置保存下来，如果根节点的位置是 (x, y)，那么其左子节点和右子节点的位置分别是 (x - 1, y + 1) 和 (x + 1, y + 1)，再按顺序保存到结果数组即可。

```c++
class Solution {
    map<int, map<int, vector<int>>> matrix;
public:
    vector<vector<int>> verticalTraversal(TreeNode *root) {
        vector<vector<int>> res;
        matrix = map<int, map<int, vector<int>>>();
        Traverse(root, 0, 0);
        for (auto &m:matrix) {
            vector<int> temp;
            for (auto &n:m.second) {
                sort(n.second.begin(), n.second.end());
                temp.insert(temp.end(), n.second.begin(), n.second.end());
            }
            res.push_back(temp);
        }
        return res;
    }

    void Traverse(TreeNode *root, int x, int y) {
        if (!root)
            return;
        if (matrix.find(x) == matrix.end())
            matrix[x] = map<int, vector<int>>();
        if (matrix[x].find(y) == matrix[x].end())
            matrix[x][y] = vector<int>();
        matrix[x][y].push_back(root->val);
        Traverse(root->left, x - 1, y + 1);
        Traverse(root->right, x + 1, y + 1);
    }
};
```

#### [103 二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

锯齿形层次遍历一个二叉树。

用两个栈轮流从左往右和从右往左保存节点，再依次加入结果数组即可。

```c++
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode *root) {
        stack<TreeNode *> s1, s2;
        vector<vector<int>> res;
        if (!root)
            return res;
        s1.push(root);
        while (!s1.empty() || !s2.empty()) {
            vector<int> lvl;
            if (s2.empty()) {
                while (!s1.empty()) {
                    TreeNode *node = s1.top();
                    s1.pop();
                    lvl.push_back(node->val);
                    if (node->left)
                        s2.push(node->left);
                    if (node->right)
                        s2.push(node->right);
                }
            } else {
                while (!s2.empty()) {
                    TreeNode *node = s2.top();
                    s2.pop();
                    lvl.push_back(node->val);
                    if (node->right)
                        s1.push(node->right);
                    if (node->left)
                        s1.push(node->left);
                }
            }
            res.push_back(lvl);
        }
        return res;
    }
};
```

#### [124 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

用递归的方式，在每个节点值加上其左右子树的最大路径来更新结果，并以其左右子树中较大的一个加上其节点值返回即可。

```c++
class Solution {
    int res;
public:
    int maxPathSum(TreeNode *root) {
        res = INT_MIN;
        Traverse(root);
        return res;
    }

    int Traverse(TreeNode *node) {
        if (!node)
            return 0;
        int left = max(0, Traverse(node->left));
        int right = max(0, Traverse(node->right));
        res = max(res, left + right + node->val);
        return max(left, right) + node->val;
    }
};
```

#### [968 监控二叉树](https://leetcode-cn.com/problems/binary-tree-cameras/)

贪心法，分情况讨论，三种状态分别是：0 表示节点未被监控，1 表示节点自带监控，2 表示节点被子节点监控；一个节点的两个字子节点组合起来分别由 6 种情况，分别是：若两个子节点都为 2（22），那么当前节点未被监控，需要被父节点监控，返回 0；若两个子节点至少有一个为 0（00，01，02，未被监控），那么当前节点需要装上监控以监控子节点，返回 1；剩下的 2 种情况是至少有一个子节点自带监控（11，12），那么当前节点被子节点监控，且其子节点都已被监控或自带监控，返回 2；最后需要单独判断树的根节点是否是 0 的状态，因为已经没有父节点可以进行监控。

```c++
class Solution {
    int res;
public:
    int minCameraCover(TreeNode *root) {
        res = 0;
        if (Traverse(root) == 0)
            ++res;
        return res;
    }

    int Traverse(TreeNode *node) {
        if (!node)
            return 2;
        int left = Traverse(node->left);
        int right = Traverse(node->right);
        if (left == 2 && right == 2)
            return 0;
        else if (left == 0 || right == 0) {
            ++res;
            return 1;
        }
        return 2;
    }
};
```

### 2. 构造二叉树

#### [105 从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

根据前序遍历与中序遍历的结果构造二叉树。

前序遍历结果中的第一个元素一定是二叉树的根节点的值，因此在中序遍历结果中找到这个值，那么中序遍历结果中这个值左边的所有元素一定都在这个根节点的左子树上，右边的所有元素一定都在这个根节点的右子树上，假设左边的长度为 m，那么在前序遍历结果的第一个元素后的 m 个元素也都对应左子树上的这些元素，分别把这两部分递归调用构造新的子树即可。

```c++
class Solution {
public:
    TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder) {
        return Build(preorder, 0, preorder.size(), inorder, 0, inorder.size());
    }

    TreeNode *Build(vector<int> &preorder, int x, int y, vector<int> &inorder, int m, int n) {
        if (x >= y || m >= n)
            return nullptr;
        TreeNode *node = new TreeNode(preorder[x]);
        int pos = m;
        while (pos < n && inorder[pos] != preorder[x])
            ++pos;
        node->left = Build(preorder, x + 1, x + 1 + pos - m, inorder, m, pos);
        node->right = Build(preorder, x + 1 + pos - m, y, inorder, pos + 1, n);
        return node;
    }
};
```

#### [106 从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

根据后序遍历与中序遍历的结果构造二叉树。

后序遍历结果中的最后一个元素一定是二叉树的根节点的值，因此在中序遍历结果中找到这个值，那么中序遍历结果中这个值左边的所有元素一定都在这个根节点的左子树上，右边的所有元素一定都在这个根节点的右子树上，假设左边的长度为 m，那么在后序遍历结果中从第一个元素往后 m 个元素也都对应左子树上的这些元素，分别把这两部分递归调用构造新的子树即可。

```c++
class Solution {
public:
    TreeNode *buildTree(vector<int> &inorder, vector<int> &postorder) {
        return Build(inorder, 0, inorder.size(), postorder, 0, postorder.size());
    }

    TreeNode *Build(vector<int> &inorder, int x, int y, vector<int> &postorder, int m, int n) {
        if (x >= y || m >= n)
            return nullptr;
        TreeNode *node = new TreeNode(postorder[n - 1]);
        int pos = x;
        while (pos < y && inorder[pos] != postorder[n - 1])
            ++pos;
        node->left = Build(inorder, x, pos, postorder, m, m + pos - x);
        node->right = Build(inorder, pos + 1, y, postorder, m + pos - x, n - 1);
        return node;
    }
};
```

#### [889 根据前序和后序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

根据前序遍历与后序遍历的结果构造二叉树。

前序遍历结果中的第一个元素对应后序遍历结果中的最后一个元素，都是二叉树的根节点的值，以此构建根节点；前序遍历结果中的第二个元素一定是根节点的左子节点，而后序遍历结果中的这个值一定是左子树上的最后一个值，那么只需要找到这个值在后序遍历结果中的位置，就可以确定左子树和右子树的长度，分别递归调用构造新的子树即可。

```c++
class Solution {
public:
    TreeNode *constructFromPrePost(vector<int> &pre, vector<int> &post) {
        return Build(pre, 0, pre.size(), post, 0, post.size());
    }

    TreeNode *Build(vector<int> &pre, int x, int y, vector<int> &post, int m, int n) {
        if (x >= y || m >= n)
            return nullptr;
        TreeNode *node = new TreeNode(pre[x]);
        if (x == y - 1)
            return node;
        int pos = m;
        while (pos < n && post[pos] != pre[x + 1])
            ++pos;
        node->left = Build(pre, x + 1, x + pos - m + 2, post, m, pos + 1);
        node->right = Build(pre, x + pos - m + 2, y, post, pos + 1, n - 1);
        return node;
    }
};
```

#### [1008 先序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/)

给一个先序遍历的结果，构造其对应的二叉搜索树。

根据先序遍历和二叉搜索树的定义，数组的第一个元素即是根节点，其后所有小于它的元素都在它的左子树上，所有大于它的元素都在它的右子树上，递归求解即可。

```c++
class Solution {
public:
    TreeNode *bstFromPreorder(vector<int> &preorder) {
        return Build(preorder, 0, preorder.size());
    }

    TreeNode *Build(vector<int> &preorder, int x, int y) {
        if (x >= y)
            return nullptr;
        TreeNode *root = new TreeNode(preorder[x]);
        int i = x + 1;
        while (i < y && preorder[i] < preorder[x])
            ++i;
        root->left = Build(preorder, x + 1, i);
        root->right = Build(preorder, i, y);
        return root;
    }
};
```

#### [1028 从先序遍历还原二叉树](https://leetcode-cn.com/problems/recover-a-tree-from-preorder-traversal/)

给一个先序遍历的结果和用不同长度的 '-' 相连的字符串，构造其对应的二叉树。

'-' 的长度代表当前的层次，如果当前节点之后的 '-' 的长度等于当前的层次加一，那么其后的数构成当前节点的左节点；如果左节点之后的 '-' 的长度等于当前的层次加一，那么其后的数构成当前节点的右节点；否则如果当前节点之后或左节点之后的 '-' 的长度小于等于等钱层次则代表当前节点已经是叶子节点，其没有左右子节点，直接返回。用一个变量 pos 保存当前遍历到的字符串位置，递归调用该过程即可。

```c++
class Solution {
public:
    TreeNode *recoverFromPreorder(string S) {
        int pos = 0;
        return Build(S, 0, pos, 0);
    }

    TreeNode *Build(const string &S, int x, int &pos, int lvl) {
        int num = 0, i = x;
        while (S[i] >= '0' && S[i] <= '9')
            ++i;
        TreeNode *node = new TreeNode(stoi(S.substr(x, i - x + 1)));
        pos = i;
        while (S[i] == '-')
            ++i;
        if (i - pos == lvl + 1)
            node->left = Build(S, i, pos, lvl + 1);
        else
            return node;
        i = pos;
        while (S[i] == '-')
            ++i;
        if (i - pos == lvl + 1)
            node->right = Build(S, i, y, pos, lvl + 1);
        return node;
    }
};
```
