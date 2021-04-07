---
title: "LeetCode 链表（1）"
date: 2019-07-04T19:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 链表（1）](https://leetcode-cn.com/tag/linked-list/)

## 题目

### 1. 常规题

#### [2 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

给两个链表分别代表两个正数的逆序表示，计算两个链表之和。

依次按位进行相加。

```c++
class Solution {
public:
    ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
        int acc = 0, val = 0;
        auto head = l1, tail = l1;
        while (l1 && l2) {
            val = l1->val + l2->val + acc;
            acc = val / 10;
            l1->val = val % 10;
            if (!l1->next)
                l1->next = l2->next, l2->next = nullptr;
            tail = l1;
            l1 = l1->next, l2 = l2->next;
        }
        while (l1) {
            val = l1->val + acc;
            acc = val / 10;
            l1->val = val % 10;
            tail = l1;
            l1 = l1->next;
        }
        if (acc)
            tail->next = new ListNode(1);
        return head;
    }
};
```

#### [21 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/submissions/)

合并两个有序链表。

逐个比较大小并添加到当前节点后面，并移动对应的链表节点。

```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode *head = new ListNode(0), *node = head;
        while (l1 && l2) {
            if (l1->val < l2->val)
                head->next = l1, l1 = l1->next;
            else
                head->next = l2, l2 = l2->next;
            head = head->next;
        }
        while (l1)
            head->next = l1, l1 = l1->next, head = head->next;
        while (l2)
            head->next = l2, l2 = l2->next, head = head->next;
        return node->next;
    }
};
```

#### [83 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/submissions/)

删除链表中所有重复的节点。

将每个节点与其后面的节点的值做对比，如果相同则删除后面的节点。

```c++
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        auto ret = head;
        while (head) {
            while (head->next && head->next->val == head->val) {
                auto next = head->next;
                head->next = next->next;
                delete next;
            }
            head = head->next;
        }
        return ret;
    }
};
```

#### [82 删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/submissions/)

删除链表中所有重复的节点，只保留原始链表中没有重复出现的数字。

为了删除所有重复的节点并只保留所有没有出现过的数字，需要提前两个节点检查接下来的两个节点的值是否相同，如果相同的话需要将这两个节点及其后的所有重复节点都删除。

```c++
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode *node = new ListNode(0), *ret = node, *next = nullptr, *temp = nullptr;
        node->next = head;
        while (node && node->next) {
            next = node->next->next;
            while (next && node->next->val == next->val) {
                temp = next;
                next = next->next;
                delete temp;
            }
            if (next != node->next->next) {
                temp = node->next;
                node->next = next;
                delete temp;
            } else
                node = node->next;
        }
        return ret->next;
    }
};
```

#### [203 移除链表元素](https://leetcode-cn.com/problems/remove-linked-list-elements/submissions/)

删除链表中等于给定值的所有节点。

先判断头节点是否等于给定值，再判断其后面的节点是否等于给定值。

```c++
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        while (head && head->val == val) {
            auto prev = head;
            head = head->next;
            delete prev;
        }
        auto ret = head;
        while (head) {
            while (head->next && head->next->val == val) {
                auto next = head->next;
                head->next = next->next;
                delete next;
            }
            head = head->next;
        }
        return ret;
    }
};
```

#### [817 链表组件](https://leetcode-cn.com/problems/linked-list-components/submissions/)

给一个链表和一个数组，找到链表中一段子链表的值都在数组中的子链表的个数。

先将数组转换成哈希表方便查询，再依次遍历整个链表，判断一段子链表结束时或遍历结束时子链表是否是符合条件。

```c++
class Solution {
public:
    int numComponents(ListNode* head, vector<int>& G) {
        unordered_set<int> exist(G.begin(), G.end());
        bool cont = false, curr = false;
        int res = 0;
        while (head) {
            curr = exist.count(head->val);
            res += cont && !curr;
            cont = curr;
            head = head->next;
        }
        return res + cont;
    }
};
```

#### [24 两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

给一个链表，返回两两交换其中相邻的节点后的结果。

用三个指针把要交换的两个节点和他们的前驱节点保存下来，交换后再更新三个指针，按顺序交换即可。

```c++
class Solution {
public:
    ListNode *swapPairs(ListNode *head) {
        if (!head || !head->next)
            return head;
        ListNode *root = new ListNode(0), *prev = root;
        root->next = head;
        auto first = head, second = head->next;
        while (first && second) {
            auto temp = second->next;
            prev->next = second;
            second->next = first;
            first->next = temp;
            prev = first;
            first = temp;
            second = temp ? temp->next : nullptr;
        }
        return root->next;
    }
};
```

#### [430 扁平化多级双向链表](https://leetcode-cn.com/problems/flatten-a-multilevel-doubly-linked-list/submissions/)

给一个带子节点的双向链表，将其扁平化并使所有结点出现在单级双链表中。

对于某一个节点，如果它有 child 节点，那么需要将其 child 节点作为其新的 next 节点，将其 child 链表上的最后一个节点作为其原本 next 节点的 prev 节点，递归调用整个过程即可。

```c++
class Solution {
public:
    Node *flatten(Node *head) {
        if (!head)
            return nullptr;
        Node *node = head, *next = nullptr;
        while (node) {
            if (node->child) {
                next = node->next;
                auto child = flatten(node->child);
                node->next = child;
                child->prev = node;
                node->child = nullptr;
                while (node->next)
                    node = node->next;
                node->next = next;
                if (next)
                    next->prev = node;
            }
            node = node->next;
        }
        return head;
    }
};
```

### 2. 链表反转

#### [206 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/submissions/)

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head)
            return nullptr;
        ListNode *prev = nullptr, *curr = head, *next = head->next;
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

#### [92 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/submissions/)

将链表中从 m 到 n 位置的节点反转。

先找到位置在 m - 1 的节点，将其后的节点截断，再找到位置在 n 的节点，将其后的节点截断，将中间的一段链表反转后再链接到愿链表上。

```c++
class Solution {
public:
    ListNode *reverseBetween(ListNode *head, int m, int n) {
        ListNode *root = new ListNode(0), *r = root, *prev, *last;
        root->next = head;
        int cnt = 1;
        while (cnt < m && r)
            r = r->next, ++cnt;
        prev = r;
        while (cnt <= n && r && r->next)
            r = r->next, ++cnt;
        last = r->next;
        r->next = nullptr;
        prev->next = Reverse(prev->next);
        while (prev && prev->next)
            prev = prev->next;
        prev->next = last;
        return root->next;
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

#### [369 给单链表加一](https://leetcode-cn.com/problems/plus-one-linked-list/submissions/)

用一个单链表表示一个整数，计算将其加一的结果。

先将链表反转方便进位，然后进行加一和进位的操作，最后再反转一次链表。

```c++
class Solution {
public:
    ListNode* plusOne(ListNode* head) {
        if (!head)
            return nullptr;
        head = Reverse(head);
        auto root = head;
        while (head) {
            if (head->val == 9) {
                head->val = 0;
                if (!head->next) {
                    head->next = new ListNode(1);
                    break;
                }
                head = head->next;
            }
            else {
                ++head->val;
                break;
            }
        }
        return Reverse(root);
    }
    
    ListNode *Reverse(ListNode* head) {
        if (!head)
            return nullptr;
        ListNode *prev = nullptr, *curr = head, *next = head->next;
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

#### [445 两数相加 II](https://leetcode-cn.com/problems/add-two-numbers-ii/submissions/)

给两个链表分别代表两个正数，计算两个链表之和。

先将两个链表反转，再依次按位进行相加，最后再将得到的结果反转并返回。

```c++
class Solution {
public:
    ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
        if (!l1 || !l2)
            return !l1 ? l2 : l1;
        l1 = Reverse(l1);
        l2 = Reverse(l2);
        int carry = 0, sum = 0;
        ListNode *head = l1, *prev = l1;
        while (l1 && l2) {
            prev = l1;
            sum = l1->val + l2->val + carry;
            carry = sum / 10;
            l1->val = sum - carry * 10;
            if (l2->next && !l1->next)
                l1->next = l2->next, l2->next = nullptr;
            l1 = l1->next;
            l2 = l2->next;
        }
        while (l1) {
            prev = l1;
            sum = l1->val + carry;
            carry = sum / 10;
            l1->val = sum - carry * 10;
            l1 = l1->next;
        }
        if (carry && prev)
            prev->next = new ListNode(1);
        return Reverse(head);
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

#### [25 K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

给一个链表，每 k 个节点一组进行翻转。

用几个指针记录下需要反转的部分的起始，终止位置，依次反转即可。

```c++
class Solution {
    ListNode *ReverseLinkedList(ListNode *head) {
        ListNode *prev = nullptr, *curr = head, *next = nullptr;
        while (curr) {
            next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }

public:
    ListNode *reverseKGroup(ListNode *head, int k) {
        ListNode *root = new ListNode(0), *prev = root;
        prev->next = head;
        while (prev) {
            int i = 1;
            ListNode *start = prev->next, *end = prev->next, *curr = end, *next = nullptr;
            while (curr && i < k)
                curr = curr->next, ++i;
            if (i < k || !curr)
                break;
            next = curr->next;
            curr->next = nullptr;
            start = ReverseLinkedList(start);
            prev->next = start;
            end->next = next;
            prev = end;
        }
        return root->next;
    }
};
```

### 3. 双链表

#### [328 奇偶链表](https://leetcode-cn.com/problems/odd-even-linked-list/submissions/)

把一个链表中的奇数位节点和偶数位节点分别排在一起。

用两个头节点分别表示奇数位和偶数位节点的起始位置，遍历整个链表，将奇数位节点链接在奇数位起始节点后，将偶数位节点链接在偶数位起始节点后，最后将偶数位起始节点链接在奇数位最后节点后即可。

```c++
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if (!head || !head->next)
            return head;
        auto odd = head, even = head->next, node = even->next, even_head = even;
        bool flag = true;
        while (node) {
            if (flag) {
                odd->next = node;
                odd = odd->next;
                flag = false;
            } else {
                even->next = node;
                even = even->next;
                flag = true;
            }
            node = node->next;
        }
        odd->next = even_head;
        even->next = nullptr;
        return head;
    }
};
```

#### [86 分隔链表](https://leetcode-cn.com/problems/partition-list/)

给一个链表和一个值 x，重新排列链表使得所有小于 x 的节点都在大于等于 x 的节点之前。

用两个头节点 sth 和 geq 分别表示小于 x 和大于等于 x 的节点的起始位置，遍历整个链表，分别将各个节点链接到两个头节点之后，最后将 geq 链接到 sth 之后，再将 geq 的末端设为空指针。

```c++
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        auto sth = new ListNode(0), sth_head = sth, geq = new ListNode(0), geq_head = geq;
        while (head) {
            if (head->val < x) {
                sth->next = head;
                sth = sth->next;
            } else {
                geq->next = head;
                geq = geq->next;
            }
            head = head->next;
        }
        sth->next = geq_head->next;
        geq->next = nullptr;
        return sth_head->next;
    }
};
```

#### [725 分隔链表](https://leetcode-cn.com/problems/split-linked-list-in-parts/submissions/)

给一个链表, 将其分隔为 k 个连续的部分。

先计算出链表的长度和 k 个连续部分中每个部分的长度，依次将每个部分的头节点放入数组中。

```c++
class Solution {
public:
    vector<ListNode *> splitListToParts(ListNode *root, int k) {
        vector<ListNode *> res(k, nullptr);
        int len = 0;
        ListNode *head = root, *temp = nullptr;
        while (root)
            root = root->next, ++len;
        int n = len / k, m = len % k, i = 0;
        while (head) {
            res[i] = head;
            for (int j = 0; j < n + (m > 0) - 1; ++j)
                head = head->next;
            temp = head;
            head = head->next;
            temp->next = nullptr;
            ++i;
            --m;
        }
        return res;
    }
};
```
