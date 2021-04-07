---
title: "LeetCode 动态规划（2）"
date: 2019-06-28T10:09:13+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 动态规划](https://leetcode-cn.com/problemset/all/?search=%E4%B8%91%E6%95%B0)

## 题目

### 3. 数组相关

#### [300 最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/submissions/)

在无序数组中找到最长上升子序列的长度。

用一个数组 dp[i] 表示到第 i 个数字为止的最长上升子序列，每次遍历 i 之前的每个数字 j，如果 nums[i] > nums[j]，那么 j 和 i 可以形成一个上升子序列，让 dp[i] = max(dp[i], dp[j] + 1) 就能得到最长的上升子序列了。

```c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size(), res = 2;
        if (n <= 1)
            return n;
        vector<int> dp(n, 1);
        for (int i = 1; i < n; ++i)
            for (int j = 0; j < i; ++j)
                if (nums[i] > nums[j]) {
                    dp[i] = max(dp[i], dp[j] + 1);
                    res = max(res, dp[i]);
                }
        return res;
    }
};
```

#### [53 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

找一个数组中具有最大和的连续子数组的和。

从头开始用一个变量 val 保存到当前数为止的连续子数组和，每一个数对于之前的连续子数组和只有加与不加两种选择，当之前的连续子数组和大于 0 时加上之前的连续子数组和，否则不加。

```c++
class Solution {
public:
    int maxSubArray(vector<int> &nums) {
        int res = INT_MIN, val = 0;
        for (auto &n:nums) {
            val = max(val, 0) + n;
            res = max(res, val);
        }
        return res;
    }
};
```

#### [718 最长重复子数组](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)

给两个数组，求两个数组中公共的长度最长的子数组的长度。

对于某两个字符 A[i] 和 B[j]，如果 A[i] == B[j]，则代表 A[i]，B[j] 与他们之前的子数组有可能是公共子数组，其最长长度 dp[i][j] = dp[i - 1][j - 1] + 1，否则他们不能组成公共子数组，dp[i][j] = 0。用两层循环遍历两个数组即可，时间复杂度是 O(m * n)，空间复杂度是 O(m * n)。

```c++
class Solution {
public:
    int findLength(vector<int> &A, vector<int> &B) {
        int m = A.size(), n = B.size(), res = 0;
        if (m == 0 || n == 0)
            return 0;
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for (int i = 1; i <= m; ++i)
            for (int j = 1; j <= n; ++j) {
                dp[i][j] = A[i - 1] == B[j - 1] ? dp[i - 1][j - 1] + 1 : 0;
                res = max(res, dp[i][j]);
            }
        return res;
    }
};
```

#### [983 最低票价](https://leetcode-cn.com/problems/minimum-cost-for-tickets/)

给出要旅行的所有日期，有三种通行证：一日票，七日票，三十日票。求最低消费。

对于第 i 天的最低消费，只需要选出一天前的最低消费加上一日票的消费，七天前的最低消费加上七日票，三十天前的最低消费加上三十日票这三种消费中最低的即可，因此有状态转移方程 dp[i] = min({dp[i - 1] + costs[0], dp[i - 7] + costs[1], dp[i - 30] + costs[2]})。

```c++
class Solution {
public:
    int mincostTickets(vector<int> &days, vector<int> &costs) {
        vector<int> dp(366, INT_MAX);
        dp[0] = 0;
        int j = 0, n = days.size();
        for (int i = 1; i <= 365 && j < n; ++i) {
            if (i == days[j]) {
                dp[i] = min({dp[i - 1] + costs[0], dp[max(0, i - 7)] + costs[1], dp[max(0, i - 30)] + costs[2]});
                ++j;
            } else
                dp[i] = dp[i - 1];
        }
        return dp[days[n - 1]];
    }
};
```

#### [813 最大平均值和的分组](https://leetcode-cn.com/problems/largest-sum-of-averages/)

将数组分为 K 个相邻的非空子数组，求所有子数组的平均值的和的最大值。

用二维数组 dp[n][K] 来表示前 i 个数分成 k 组得到的最优值，每次将从第 j 个数到第 n 个数分为一组，将前面 j - 1 个数分为 k - 1 组，求得 dp[i][k] 的最大值。为了快速地算出第 j 个数到第 n 个数的和以及前 j - 1 个数的和，可以用一个前缀和数组将前 m 个数的和保存下来，再用状态转移方程 dp[i][k] = max(dp[i][k], dp[j][k - 1] + (pre[i] - pre[j]) / (i - j)) 求得最优值。

```c++
class Solution {
public:
    double largestSumOfAverages(vector<int> &A, int K) {
        int n = A.size();
        if (n == 0)
            return 0;
        vector<double> pre(n + 1, 0);
        for (int i = 1; i <= n; ++i)
            pre[i] += pre[i - 1] + A[i - 1];
        vector<vector<double>> dp(n + 1, vector<double>(K + 1, 0));
        for (int i = 1; i <= n; ++i) {
            dp[i][1] = pre[i] / i;
            for (int k = 2; k <= K && k <= i; ++k)
                for (int j = 1; j < i; ++j)
                    dp[i][k] = max(dp[i][k], dp[j][k - 1] + (pre[i] - pre[j]) / (i - j));
        }
        return dp[n][K];
    }
};
```

#### [646 最长数对链](https://leetcode-cn.com/problems/maximum-length-of-pair-chain/)

按照每个数对的第一个元素从小到大排序，从第二个数对开始，依次判断其与其前面的所有数对是否符合题意，是的话则 dp[j] = max(dp[j], dp[i] + 1）。时间复杂度是 O(n^2)。

```c++
class Solution {
public:
    int findLongestChain(vector<vector<int>>& pairs) {
        sort(pairs.begin(), pairs.end());
        int n = pairs.size(), res = 1;
        vector<int> dp(n, 1);
        for (int j = 1; j < n; ++j)
            for (int i = 0; i < j; ++i)
                if (pairs[j][0] > pairs[i][1]) {
                    dp[j] = max(dp[j], dp[i] + 1);
                    res = max(res, dp[j]);
                }
        return res;
    }
};
```

### 4. 等差数列

#### [413 等差数列划分](https://leetcode-cn.com/problems/arithmetic-slices/)

给一个数组，计算数组中等差子数组的个数。

等差数列必须是相邻两个元素之差相等的长度大于 3 的子数组，因此只需要知道 nums[i] - nums[i - 1] == nums[i - 1] - nums[i - 1] 即可。用一个变量 cul 表示到目前为止等差数列的长度，diff 表示之前的公差，如果目前的差等于 diff 则加上 cul。时间复杂度是 O(n)，空间复杂度是 O(1)。

```c++
class Solution {
public:
    int numberOfArithmeticSlices(vector<int>& nums) {
        int res = 0, n = nums.size(), cul = 1, curr = 0;
        if (n < 3)
            return 0;
        int diff = nums[1] - nums[0];
        for (int i = 2; i < n; ++i) {
            if ((curr = nums[i] - nums[i - 1]) == diff) {
                res += cul;
                ++cul;
            }
            else {
                diff = curr;
                cul = 1;
            }
        }
        return res;
    }
};
```

#### [446 等差数列划分 II - 子序列](https://leetcode-cn.com/problems/arithmetic-slices-ii-subsequence/)

给一个数组，计算数组中等差子序列的个数。

对于某一个数 nums[i]，已知其与其之前某一个数的差 diff = nums[i] - nums[j]，需要知道在 nums[j] 之前公差为 diff 的等差子序列的最大个数，可以用一个哈希表数组来表示数 nums[j] 之前公差为 diff 的等差子序列的最大个数，如果 nums[i] - nums[j] == diff 则到位置 i 为止公差为 diff 的等差子序列的个数 dp[i][diff] += dp[j][diff]，注意要用 += 而不是 =，因为如果在 nums[i] 之前有多个相同的数字那么需要把每一个都算作一个独立的等差子序列。时间复杂度是 O(n ^ 2)，空间复杂度是 O(n ^ 2)。很搞笑的是这道题的动态规划解法在 LeetCode 上提交的时候 runtime 是 1000ms +-，而在LeetCode-CN 上提交的时候执行用时是 1400ms ~ 1800ms，而且偶尔还会超时，超时的 [case](https://leetcode-cn.com/submissions/detail/21590899/testcase/) 的公差超过了 int32 的表示范围，如果加上判断 diff < INT_MIN || diff > INT_MAX 直接 continue 就能正常通过了。

```c++
class Solution {
public:
    int numberOfArithmeticSlices(vector<int>& A) {
        int n = A.size(), res = 0;
        long long diff = 0;
        vector<unordered_map<long long, int>> dp(n);
        for (int i = 1; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                diff = static_cast<long long>(A[i]) - A[j];
                if (diff < INT_MIN || diff > INT_MAX)
                    continue;
                dp[i][diff] += dp[j][diff] + 1;
                res += dp[j][diff];
            }
        }
        return res;
    }
};
```

#### [1027 最长等差数列](https://leetcode-cn.com/problems/longest-arithmetic-sequence/submissions/)

给一个数组，计算数组中最长等差子序列的长度。

对于某一个数 nums[i]，已知其与其之前某一个数的差 diff = nums[i] - nums[j]，需要知道在 nums[j] 之前公差为 diff 的等差子序列有多长，可以用一个哈希表数组来表示每一个数之前公差为 diff 的等差子序列的最长长度，在此基础上 +1 即可得到最长的等差子序列的长度。时间复杂度是 O(n ^ 2)，空间复杂度是 O(n ^ 2)。

```c++
class Solution {
public:
    int longestArithSeqLength(vector<int> &A) {
        int n = A.size(), res = 2;
        vector<unordered_map<int, int>> dp(n);
        for (int i = 1; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                int diff = A[i] - A[j];
                if (dp[j].find(diff) == dp[j].end())
                    dp[i][diff] = 2;
                else {
                    dp[i][diff] = dp[j][diff] + 1;
                    res = max(res, dp[i][diff]);
                }
            }
        }
        return res;
    }
};
```

### 5. 斐波那契数列

#### [70 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

每次能爬 1 或 2 阶楼梯，求爬 n 阶楼梯有多少种方法。

爬到当前楼梯的方法等于爬到前两阶楼梯的方法之和，因此有状态转移方程 dp[i] = dp[i - 1] + dp[i - 2]，又因为当前状态只取决于前两个状态，因此可以只使用两个变量 s1 和 s2 来保存前两个状态的结果。时间复杂度是 O(n)，空间复杂度是 O(1)。

```c++
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 1)
            return 1;
        int s1 = 1, s2 = 1, temp;
        for (int i = 1; i < n; ++i) {
            temp = s2;
            s2 += s1;
            s1 = temp;
        }
        return s2;
    }
};
```

#### [746 使用最小花费爬楼梯](https://leetcode-cn.com/problems/min-cost-climbing-stairs/)

每次能爬 1 或 2 阶楼梯，每一阶楼梯有一个权值，求爬 n 阶楼梯的最小花费。

和上一题类似，不过每次只需要取前两阶楼梯中权值较小的就可以了。

```c++
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int n = cost.size();
        if (n == 0)
            return 0;
        else if (n == 1)
            return cost[0];
        else if (n == 2)
            return cost[0] + cost[1];
        int s1 = cost[0], s2 = cost[1];
        for (int i = 2; i < n; ++i) {
            int temp = s2;
            s2 = min(s1, s2) + cost[i];
            s1 = temp;
        }
        return min(s1, s2);
    }
};
```

#### [740 删除与获得点数](https://leetcode-cn.com/problems/delete-and-earn/)

给一个数组，每次任选一个 nums[i]，获得 nums[i] 的个数乘以 nums[i] 的点数，计算能获得的最大点数。

对于 nums[i]，能取到的最大点数只能是 nums[i - 1] 的最大点数或 nums[i - 2] 的最大点书加上 nums[i] 能获得的点数，因此有状态转移方程 dp[i] = max(dp[i - 1], dp[i - 2] + val[i])，时间复杂度是 O(n)。每个点只与其之前两个点有关，因此可以只使用两个变量 p1 和 p2 来保存前两个点的结果，空间复杂度是 O(1)。

```c++
class Solution {
public:
    int deleteAndEarn(vector<int>& nums) {
        vector<int> val(10001, 0);
        for (auto &m:nums)
            val[m] += m;
        int res = 0, prev = 0, curr = 0;
        for (int i = 1; i <= 10000; ++i) {
            res = max(curr, prev + val[i]);
            prev = curr;
            curr = res;
        }
        return res;
    }
};
```

#### [198 打家劫舍](https://leetcode-cn.com/problems/house-robber/)

给一个带权值的数组，取不相邻的数，求能取到的最大值。

对于某一点，如果到之前一点为止能取到的最大值大于取当前点与之前两点的最大值的和则不取，否则取当前点，当前点的值与之前两点的最大值之和就是当前点的最优值，因此有状态转移方程 dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])，时间复杂度是 O(n)。当前点只与之前两个点有关，因此可以只使用两个变量 p1 和 p2 来保存前两个点的结果，空间复杂度是 O(1)。

```c++
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if (n <= 2)
            return n == 0 ? 0 : n == 1 ? nums[0] : max(nums[0], nums[1]);
        int p1 = nums[0], p2 = max(nums[0], nums[1]), temp;
        for (int i = 2; i < n; ++i) {
            temp = p2;
            p2 = max(p1 + nums[i], p2);
            p1 = temp;
        }
        return p2;
    }
};
```

#### [213 打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii/)

给一个带权值的数组，数组首尾相邻，取不相邻的数，求能取到的最大值。

数组首尾相邻代表不能同时取第一个点和最后一个点，因此从第一个点到倒数第二个点进行动态规划得到的就是包含第一个点而不包含最后一个点能取到的最大值，从第二个点到最后一个点进行动态规划得到的就是包含最后一个点而不包含第一个点能取到的最大值，用于上一题同样的方法分别做两次就能得到结果。时间复杂度是 O(n)，空间复杂度是 O(1)。

```
class Solution {
public:
    int rob(vector<int>& nums) {
        int res = 0, n = nums.size();
        if (n <= 2)
            return n == 0 ? 0 : n == 1 ? nums[0] : max(nums[0], nums[1]);
        int p1 = nums[0], p2 = max(nums[0], nums[1]);
        for (int i = 2; i < n - 1; ++i) {
            int temp = p2;
            p2 = max(p2, p1 + nums[i]);
            p1 = temp;
        }
        res = p2;
        p1 = nums[1], p2 = max(nums[1], nums[2]);
        for (int i = 3; i < n; ++i) {
            int temp = p2;
            p2 = max(p2, p1 + nums[i]);
            p1 = temp;
        }
        return max(res, p2);
    }
};
```

#### [337 打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii/)

给一个带权值的二叉树，取不相邻的数，求能取到的最大值。

对于一个节点，如果取节点本身则不能取两个子节点，如果取两个子节点则不能去其本身和四个子节点，因此对比其两个子节点的和与其本身和四个孙子节点的和，取最大值返回，也就是 max(dp[node->left->left] + dp [node->left->right] + dp[node->right->left] + dp [node->right->right] + dp[node], dp[node->left] + dp[node->right])。因为递归会造成大量的重复计算，因此用一个哈希表把已经计算过的节点的最优值保存下来，递归到该节点的时候直接取值防止造成TLE。时间复杂度是 O(n)，空间复杂度是 O(n)。

```c++
class Solution {
    unordered_map<TreeNode *, int> val;
public:
    int rob(TreeNode *node) {
        if (!node)
            return 0;
        if (val.find(node) != val.end())
            return val[node];
        int p1 = 0, p2 = 0;
        if (node->left)
            p1 += rob(node->left->left) + rob(node->left->right);
        if (node->right)
            p1 += rob(node->right->left) + rob(node->right->right);
        p1 += node->val;
        p2 += rob(node->left) + rob(node->right);
        val[node] = max(p1, p2);
        return val[node];
    }
}
```

#### [96 不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

求有 n 个节点的二叉搜索树有多少种。

对于某一个数 i 作为根结点时，无论 i 是多少，其左子树左子树总是由 i - 1 个节点构成的，而其右节点总是由 n - i 个节点构成的，例如 n = 3 时，如果让 3 作为根节点，那么其左子树一定是由 1 和 2 两个节点构成的，那么我们只需要知道由两个节点构成的二叉搜索树有多少种，再用这个左子树的种类数 dp[2] 乘以右子树的种类数 dp[0] 就能知道由 3 个节点构成的，以 3 作为根节点的种类数，其次需要依次让 1 和 2 作为根节点，那么他们的左子树分别有 dp[0] 和 dp[1] 种构成的方法，因此得到状态转移方程 dp[i] += dp[j - 1] * dp[i - j]，时间复杂度是 O(n ^ 2)，空间复杂度是 O(n)。

```c++
class Solution {
public:
    int numTrees(int n) {
        vector<int> dp(n + 1, 0);
        dp[0] = dp[1] = 1;
        for (int i = 2; i <= n; ++i)
            for (int j = 1; j <= i; ++j)
                dp[i] += dp[j - 1] * dp[i - j];
        return dp[n];
    }
};
```

#### [873 最长的斐波那契子序列的长度](https://leetcode-cn.com/problems/length-of-longest-fibonacci-subsequence/)

给一个严格递增的数组，找到其中最长的斐波那契子序列的长度。

根据斐波那契数列的定义，要判断 A[i] 和 A[j] 能否在原数组中构成斐波那契数列，只需要知道 A[i - j] 是否在原数组中，并且 A[i] - A[j] < A[j] < A[i] 是否成立，于是我们可以用一个二维数组 dp[n][n] 来代表由 A[i] 和 A[j] 以及 A[i - j] 构成的斐波那契子序列的最长长度。为了查找 A[i - j] 是否在原数组中，我们可以用一个哈希表 pos 来保存 A[i - j] 在原数组中的下标，获取下标 k = pos[A[i] - A[j]] 后，用状态转移方程 dp[i][j] = dp[j][k] + 1 来更新最长长度。

注意在判断下标是否存在于哈希表中时要用 pos.find(A[i] - A[j]) == pos.end()，而不能直接用 pos[A[i] - A[j]] 来获取，这样虽然如果 A[i] - A[j] 不存在于哈希表中仍然能够得到结果 0，但效率非常低，会导致TLE。

```c++
class Solution {
public:
    int lenLongestFibSubseq(vector<int> &A) {
        int n = A.size(), res = 0;
        vector<vector<int>> dp(n, vector<int>(n, 0));
        unordered_map<int, int> pos;
        for (int i = 0; i < n; ++i) {
            pos[A[i]] = i;
            for (int j = 0; j < i; ++j) {
                auto k = pos.find(A[i] - A[j]) == pos.end() ? -1 : pos[A[i] - A[j]];
                dp[i][j] = A[i] - A[j] < A[j] && k != -1 ? dp[j][k] + 1 : 2;
                res = max(res, dp[i][j]);
            }
        }
        return res < 3 ? 0 : res;
    }
};
```
