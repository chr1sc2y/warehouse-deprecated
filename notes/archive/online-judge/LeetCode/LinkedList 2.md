---
title: "LeetCode 链表（2）"
date: 2019-07-09T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 链表（2）](https://leetcode-cn.com/tag/linked-list/)

## 题目

### 4. 双指针

#### [19 删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/submissions/)

删除链表的倒数第 n 个节点。

在链表中不易直接取到倒数第 n 个位置，所以用两个指针 prev 和 tail，tail 先往前走 n 步，然后两个指针一起往前走直到 tail 没有后继指针，此时 prev 的后继指针就是倒数第 n 个位置，删除其即可。注意如果要删除的指针是头指针的话要单独处理。

```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode *prev = head, *tail = head;
        for (int i = 0; i < n; ++i)
            tail = tail->next;
        if (!tail) {
            head = head->next;
            delete prev;
            return head;
        }
        while (tail->next)
            tail = tail->next, prev = prev->next;
        ListNode *next = prev->next;
        prev->next = next->next;
        delete next;
        return head;
    }
};
```

#### [61 旋转链表](https://leetcode-cn.com/problems/rotate-list/submissions/)

给一个链表，将其每个节点向右移动 k 个位置。

在链表中不易直接取到前 k 个位置，所以用两个指针，第一个先往前走 k 步，然后两个指针一起往前走直到第一个指针没有后继指针，就可以将头节点链接到第一个指针后，将第二个指针后置为空。注意 k 可能非常大，要先计算一次链表的长度 len 然后用 k % len 进行计算。

```c++
class Solution {
public:
    ListNode *rotateRight(ListNode *head, int k) {
        if (!head)
            return nullptr;
        ListNode *first = head, *second = head, *traverse = head;
        int len = 0;
        while (traverse) {
            traverse = traverse->next;
            ++len;
        }
        k %= len;
        for (int i = 0; i < k; i++)
            first = first->next;
        while (first && first->next)
            first = first->next, second = second->next;
        first->next = head;
        auto ret = second->next;
        second->next = nullptr;
        return ret;
    }
};
```

#### [876 链表的中间结点](https://leetcode-cn.com/problems/middle-of-the-linked-list/solution/lian-biao-de-zhong-jian-jie-dian-by-leetcode/)

找链表的中间节点。

用两个指针 slow 和 fast，fast 每次移动两步，slow 每次移动一步，当 fast 走到末尾的时候 slow 恰好到指针的中间。

```c++
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        auto slow = head, fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow;
    }
};
```

#### [160 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/submissions/)

找到两个链表相交的起始节点。

假设链表 A 未相交的长度为 l1，链表 B 未相交的长度为 l2，相交部分的长度为 lc，为了让两个指针走相同的长度，只需要让两个指针在走到尾部的时候重新回到另一个链表的头部再继续走，最后当两个指针走过的长度都是 l1 + l2 + l3 时即相交。

```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        auto a = headA, b = headB;
        if (!a || !b)
            return nullptr;
        while (a != b) {
            a = a ? a->next : headB;
            b = b ? b->next : headA;
        }
        return a;
    }
};
```

#### [141 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

判断一个链表中是否有环。

用两个指针 slow 和 fast，fast 每次移动两步，slow 每次移动一步，如果链表中有环那么这两个指针一定会最终相遇，否则 fast 将先到达末尾，当 fast == nullptr 或 fast->next == nullptr 即代表链表没有换并且 fast 已经到达末尾。

```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        auto slow = head, fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
            if (slow == fast)
                return true;
        }
        return false;
    }
};
```

#### [142 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

给一个链表，找到链表中入环的第一个节点。

跟上一题相同，用两个指针 slow 和 fast，fast 每次移动两步，slow 每次移动一步，如果链表中有环那么这两个指针一定会最终相遇，此时 fast 指针移动的距离是 l1 + l2 + c，其中 l1 是链表中环外部分的长度，l2 是环内两个指针走过的共同部分的长度，c 是环的长度，而 slow 指针移动的距离则是 l1 + l2，因为 fast 指针的移动速度是 slow 的两倍，所以有 l1 + l2 + c = 2 * (l1 + l2)，因此 c = l1 + l2，又因为 slow 指针已经在环中走过了长度为 l2 的部分，只剩下长度为 l1 的部分，只需要用一个新的指针从链表的头部开始每次移动一步，同时让 slow 指针每次移动一步，最终他们都会移动 l1 的距离并在入环处相遇。

```c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if (!head || !head->next)
            return nullptr;
        auto slow = head->next, fast = head->next->next;
        while (fast && fast->next && fast != slow) {
            fast = fast->next->next;
            slow = slow->next;
        }
        if (!fast || !fast->next)
            return nullptr;
        auto lin = head;
        while (lin != slow)
            lin = lin->next, slow = slow->next;
        return lin;
    }
};
```

### 5. 综合

#### [234 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/submissions/)

判断一个链表是否为回文链表。

先找到链表的中间节点，然后将右边的链表翻转，将左边的链表从中间截断，再一次对比两边的每一个节点是否相等。

```c++
class Solution {
public:
    bool isPalindrome(ListNode *head) {
        if (!head || !head->next)
            return true;
        auto prev = head, slow = head, fast = head;
        while (fast && fast->next) {
            prev = slow;
            slow = slow->next;
            fast = fast->next->next;
        }
        auto node = Reverse(fast ? slow->next : slow);
        prev->next = nullptr;
        while (head && node && head->val == node->val)
            head = head->next, node = node->next;
        return !head && !node;
    }
    
    ListNode *Reverse(ListNode *head) {
        if (!head)
            return head;
        ListNode *prev = nullptr, *curr = head, *next = head->next;
        while(curr) {
            next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
};
```

#### [109 有序链表转换二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)

给一个有序链表，将其转换为一个平衡二叉搜索树。

先找到链表的中间节点，将其作为树的根节点，将中间节点的左边部分链表作为其左子树，右边部分链表作为其右子树。

```c++
class Solution {
public:
    TreeNode *sortedListToBST(ListNode *head) {
        if (!head)
            return nullptr;
        ListNode *prev = nullptr, *slow = head, *fast = head;
        while (fast && fast->next)
            prev = slow, slow = slow->next, fast = fast->next->next;
        if (prev)
            prev->next = nullptr;
        auto root = new TreeNode(slow->val);
        root->left = sortedListToBST(prev ? head : nullptr);
        root->right = sortedListToBST(slow->next);
        return root;
    }
};
```

#### [426 将二叉搜索树转化为排序的双向链表](https://leetcode-cn.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/submissions/)

将一个二叉搜索树转换为一个双向循环链表。

对于一个根节点，其被转换为双向链表之后的前驱节点应该是其左子树中值最大的节点，也就是其左子节点的最右子节点，因此只需要先找到这个节点，然后将根节点与其首尾相连即可，在此之前应该先对根节点的左子树左对应的操作，右子树亦然。除此之外还要把最小的和最大的两个节点保存下来，将这两个节点首尾相连，形成循环链表。

```c++
class Solution {
    Node *first, *last;
public:
    Node *treeToDoublyList(Node *root) {
        first = last = nullptr;
        TreeToList(root);
        if (first)
            first->left = last;
        if (last)
            last->right = first;
        return first;
    }

    void TreeToList(Node *root) {
        if (!root)
            return;
        if (!first || first->val > root->val)
            first = root;
        if (!last || last->val < root->val)
            last = root;
        auto left = root->left, right = root->right;
        TreeToList(left);
        TreeToList(right);
        while (left && left->right)
            left = left->right;
        if (left)
            left->right = root;
        root->left = left;
        while (right && right->left)
            right = right->left;
        if (right)
            right->left = root;
        root->right = right;
    }
};
```

#### [143 重排链表](https://leetcode-cn.com/problems/reorder-list/submissions/)

将一个链表从 L0→L1→…→Ln-1→Ln 重新排列为 L0→Ln→L1→Ln-1→L2→Ln-2→...。

先将链表从中间分为两部分，将后半部分反转，再将对应位置的节点依次链接上。

```c++
class Solution {
public:
    void reorderList(ListNode* head) {
        if (!head)
            return;
        auto prev = head, slow = head, fast = head, first = head, second = head;
        while (fast && fast->next)
            prev = slow, slow = slow->next, fast = fast->next->next;
        if (fast) {
            second = slow->next;
            slow->next = nullptr;
        } else {
            second = slow;
            prev->next = nullptr;
        }
        second = Reverse(second);
        while (second) {
            auto n1 = first->next, n2 = second->next;
            first->next = second;
            second->next = n1;
            first = n1;
            second = n2;
        }
    }
    
    ListNode *Reverse(ListNode *head) {
        ListNode *prev = nullptr, *curr = head, *next = nullptr;
        while (curr) {
            next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
};
```

#### [138 复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/submissions/)

给一个链表，每个节点包含一个 random 指针指向链表的某一个节点，返回这个链表的深拷贝。

因为在第一次遍历时 random 指针指向的节点可能并未被拷贝，可以先将所有节点先拷贝一次，用一个哈希表将节点的对应关系保存下来，再遍历一次将 random 指针指向的节点依次链接上，这样做空间复杂度是 O(n)。也可以先将每一个节点都拷贝一次，并将拷贝的节点链接到该节点后，再遍历两次分别修改 random 指针指向的节点以及将原链表及其拷贝分开，这样做空间复杂度是 O(1)。

```c++
class Solution {
public:
    Node *copyRandomList(Node *head) {
        if (!head)
            return head;
        Node *node = head;
        while (node) {
            Node *cop = new Node(node->val, node->next, node->random);
            node->next = cop;
            node = node->next->next;
        }
        node = head;
        while (node && node->next) {
            if (node->random)
                node->next->random = node->random->next;
            node = node->next->next;
        }
        node = head;
        Node *res = head->next;
        while (node) {
            Node *next = node->next;
            node->next = next->next;
            next->next = next->next ? next->next->next : nullptr;
            node = node->next;
        }
        return res;
    }
};
```

#### [23 合并K个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/submissions/)

合并 k 个有序链表。

最简单的做法是每次遍历所有头节点，取出值最小的一个，将其加到要返回的链表中。时间复杂度是 O(m * n)，其中 m 是链表的个数，n 是节点的总个数，空间复杂度是 O(1)。

```c++
class Solution {
public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        ListNode *root = new ListNode(0), *node = root;
        while (node) {
            ListNode *min_ptr = nullptr;
            for (auto &l :lists)
                if (l && (!min_ptr || l->val < min_ptr->val))
                    min_ptr = l;
            node->next = min_ptr;
            for (auto &l :lists)
                if (l && l == min_ptr)
                    l = l->next;
            node = node->next;
        }
        return root->next;
    }
};
```

在此基础上可以用一个小根堆来保存所有头节点，每次取出堆顶的节点，将该节点加入要返回的链表中，判断该节点是否有后继节点，如果有的话则将其加入堆中并维护，注意需要重载优先队列的比较函数。时间复杂度是 O(n * logm)，空间复杂度是 O(m)。

```c++
class Solution {
    struct compare {
        bool operator()(ListNode *l1, ListNode *l2) {
            return l1->val > l2->val;
        }
    };

public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        ListNode *root = new ListNode(0), *node = root;
        priority_queue<ListNode *, vector<ListNode *>, compare> heap;
        for (auto &l:lists)
            if (l)
                heap.push(l);
        while (!heap.empty()) {
            ListNode *curr = heap.top();
            node->next = curr;
            node = node->next;
            heap.pop();
            if (curr->next)
                heap.push(curr->next);
        }
        return root->next;
    }
};
```

还可以用分治法两两合并链表，省去维护堆所花费的空间和时间，因为链表的数量是 m，所以用分治法需要花费 logm 的时间来合并所有的链表。时间复杂度是 O(n * logm)，空间复杂度是 O(1)。

```c++
class Solution {
    ListNode *MergeLists(ListNode *l1, ListNode *l2) {
        ListNode *root = new ListNode(0), *node = root;
        while (l1 && l2) {
            if (l1->val < l2->val) {
                node->next = l1;
                l1 = l1->next;
            } else {
                node->next = l2;
                l2 = l2->next;
            }
            node = node->next;
        }
        node->next = l1 ? l1 : l2;
        return root->next;
    }

public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        int n = lists.size();
        for (int interval = 1; interval < n; interval *= 2) {
            for (int i = 0; i < n; i += interval * 2)
                if (i + interval < n)
                    lists[i] = MergeLists(lists[i], lists[i + interval]);
        }
        return n == 0 ? nullptr : lists[0];
    }
};
```