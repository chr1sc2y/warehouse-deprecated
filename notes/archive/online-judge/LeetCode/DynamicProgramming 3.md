---
title: "LeetCode 动态规划（3）"
date: 2019-07-01T18:22:45+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 动态规划](https://leetcode-cn.com/problemset/all/?search=%E4%B8%91%E6%95%B0)

## 题目

### 6. 字符串相关

#### [712 两个字符串的最小ASCII删除和](https://leetcode-cn.com/problems/minimum-ascii-delete-sum-for-two-strings/)

给定两个字符串，计算使两个字符串相同所需要删除的字符的ASCII值的和的最小值。

对于两个字符串中的字符 s1[i] 和 s2[j]，如果s1[i] == s2[j]，那么这两个字符都不需要被删除，所以 dp[i][j] = dp[i - 1][j - 1]，否则至少有一个应该被删除，取两个中的最小值，状态转移方程是dp[i][j] = min(dp[i - 1][j] + s1[i], dp[i][j - 1] + s2[j])。时间复杂度是 O(m * n)，空间复杂度是 O(m * n)。

```c++
class Solution {
public:
    int minimumDeleteSum(string s1, string s2) {
        int n1 = s1.size(), n2 = s2.size();
        vector<vector<int>> dp(n1 + 1, vector<int>(n2 + 1, 0));
        for (int i = 1; i <= n1; ++i)
            dp[i][0] = dp[i - 1][0] + s1[i - 1];
        for (int j = 1; j <= n2; ++j)
            dp[0][j] = dp[0][j - 1] + s2[j - 1];
        for (int i = 0; i < n1; ++i)
            for (int j = 0; j < n2; ++j)
                if (s1[i] == s2[j])
                    dp[i + 1][j + 1] = dp[i][j];
                else
                    dp[i + 1][j + 1] = min(dp[i][j] + s1[i] + s2[j], min(dp[i][j + 1] + s1[i], dp[i + 1][j] + s2[j]));
        return dp[n1][n2];
    }
};
```

#### [5 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

找到一个字符串中的最长回文子串。

最简单的方法是从一个字符与其前一个/两个字符分别往两边遍历。也可以按照自下而上的动态规划思想，用一个二维数组 dp[i][j]，判断两个字符 s[i] 与 s[j] 是否相等，以及他们内侧的子字符串是否是一个回文串，状态转移方程是 dp[i][j] = s[i] == s[j] && (dp[i + 1][j - 1] || j - i < 3)。

```c++
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();
        string res;
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        for (int i = 0; i < n; ++i)
            for (int j = i; j >= 0; --j)
                if (s[i] == s[j] && (i - j < 3 || dp[j + 1][i - 1])) {
                    dp[j][i] = true;
                    if (i - j + 1 > res.size())
                        res = s.substr(j, i - j + 1);
                }
        return res;
    }
};
```

#### [647 回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)

找到一个字符串中的回文子串的个数。

与上一题类似，用一个二维数组 dp[i][j]，判断两个字符 s[i] 与 s[j] 是否相等，以及他们内侧的子字符串是否是一个回文串，如果是那么 s.substr(j, i - j + 1)，将结果 +1 即可。时间复杂度是 O(n ^ 2)，空间复杂度是 O(n ^ 2)。

```c++
class Solution {
public:
    int countSubstrings(string s) {
        int res = 0, n = s.size();
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        for (int i = 0; i < n; ++i)
            for (int j = i; j >= 0; --j)
                if (s[i] == s[j] && (i - j < 3 || dp[j + 1][i - 1]))
                    dp[j][i] = true, ++res;
        return res;
    }
};
```

#### [516 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

给定一个字符串，找最长的回文子序列。

只有当两个字符相等时，他们才有可能和他们之间的子序列形成回文子序列，因此只需要知道他们之间的最长回文子序列的长度即可，否则他们之间的最长回文子序列只能是其中一个字符的左边或右边到另一个字符之间的最大回文子序列长度，状态转移方程是 dp[j][i] = s[i] == s[j] ? dp[j + 1][i - 1] + 2 : max(dp[j + 1][i], dp[j][i - 1])。时间复杂度是 O(n ^ 2)，空间复杂度是 O(n ^ 2)。

```c++
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        int n = s.size();
        if (n <= 1)
            return n;
        vector<vector<int>> dp(n, vector<int>(n, 0));
        for (int i = 1; i < n; ++i) {
            dp[i][i] = 1;
            for (int j = i - 1; j >= 0; --j) {
                if (s[i] == s[j])
                    dp[j][i] = dp[j + 1][i - 1] + 2;
                else
                    dp[j][i] = max(dp[j + 1][i], dp[j][i - 1]);
            }
        }
        return dp[0][n - 1];
    }
};
```

### 7. 路径和

#### [62 不同路径](https://leetcode-cn.com/problems/unique-paths/)

给一个矩阵，求从左上角走到右下角有多少种方法。

走到第一列和第一行的每一格都只有一种方法，其余的格子均可以从其上方和左方走一格到达，因此有状态转移方程 dp[i][j] = dp[i][j - 1] + dp[i - 1][j]。时间复杂度是 O(m * n)，空间复杂度是 O(m * n)。

```c++
class Solution {
public:
    int uniquePaths(int m, int n) {
        if (m == 0 || n == 0)
            return 0;
        vector<vector<int>> dp(m, vector<int>(n, 1));
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                    dp[i][j] = dp[i][j - 1] + dp[i - 1][j];
        return dp[m - 1][n - 1];
    }
};
```

对于每一个格子来说，它的值都等于到达上方和左方格子的方法数量之和，也就相当于在遍历完一行之后，把上一行的值全部赋值给下一行，在下一行遍历时使其加上左方格子的方法数量，由此可以将赋值的过程简化为一个一维数组，空间复杂度降低为 O(min(m, n))。

```c++
class Solution {
public:
    int uniquePaths(int m, int n) {
        if (m == 0 || n == 0)
            return 0;
        vector<int> dp(n, 1);
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                dp[j] += dp[j - 1];
        return dp[n - 1];
    }
};
```

#### [63 不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii/)

给一个矩阵，部分位置有障碍物，求从左上角走到右下角有多少种方法。

和上一题相比在部分位置增加了障碍物，首先要处理第一列和第一行，如果有一个位置有障碍物那么接下来的位置都不能到达了，然后对于其他格子，如果本身是障碍物那么也无法到达，否则仍然等于其上方和左方之和。时间复杂度是 O(m * n)，空间复杂度是 O(m * n)。

```c++
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& grid) {
        int m = grid.size(), n = m != 0 ? grid[0].size() : 0;
        if (m == 0 || n == 0)
            return 0;
        vector<vector<long long>> dp(m, vector<long long>(n, 0));
        dp[0][0] = grid[0][0] ^ 1;
        for (int i = 1; i < m; ++i)
            dp[i][0] = (grid[i][0] ^ 1) & dp[i - 1][0];
        for (int j = 1; j < n; ++j)
            dp[0][j] = (grid[0][j] ^ 1) & dp[0][j - 1];
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                dp[i][j] = grid[i][j] ? 0 : dp[i - 1][j] + dp[i][j - 1];
        return dp[m - 1][n - 1];
    }
};
```

#### [64 最小路径和](https://leetcode-cn.com/problems/minimum-path-sum/)

给一个带权值的矩阵，求从左上角走到右下角的最小权值之和。

到达第一列和第一行的每一格都只有一种方法，因此先将其初始化。因为每一格只能从其上方和左方到达，因此有状态转移方程 grid[i][j] += min(grid[i - 1][j], grid[i][j - 1])。时间复杂度是 O(m * n)。可以直接在给的矩阵中操作，因此空间复杂度是 O(1)。

```c++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = m != 0 ? grid[0].size() : 0;
        if (m == 0)
            return 0;
        for (int i = 1; i < m; ++i)
            grid[i][0] += grid[i - 1][0];
        for (int j = 1; j < n; ++j)
            grid[0][j] += grid[0][j - 1];
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                grid[i][j] += min(grid[i - 1][j], grid[i][j - 1]);
        return grid[m - 1][n - 1];
    }
};
```

#### [120 三角形最小路径和](https://leetcode-cn.com/problems/triangle/)

给一个带权值的三角形，求自顶向下的最小路径和。每一步可以移动到左下方或右下方。

因为每一格只能从其左上方和右上方到达，因此有状态转移方程 tri[i][j] += min(tri[i - 1][j], tri[i - 1][j - 1])。时间复杂度是 O(m * n)。可以直接在给的矩阵中操作，因此空间复杂度是 O(1)。

```c++
class Solution {
public:
    int minimumTotal(vector<vector<int>> &tri) {
        int n = tri.size(), res = INT_MAX;
        if (n <= 1)
            return n == 0 ? 0 : tri[0][0];
        for (int i = 1; i < n; ++i) {
            for (int j = 0; j < tri[i].size(); ++j) {
                if (j == 0)
                    tri[i][j] += tri[i - 1][j];
                else if (j >= tri[i - 1].size())
                    tri[i][j] += tri[i - 1][j - 1];
                else
                    tri[i][j] += min(tri[i - 1][j], tri[i - 1][j - 1]);
                if (i == n - 1)
                    res = min(res, tri[i][j]);
            }
        }
        return res;
    }
};
```

#### [931 下降路径最小和](https://leetcode-cn.com/problems/minimum-falling-path-sum/)

给一个带权值的方形，求自顶向下的最小路径和。每一步可以移动到左下方，下方或右下方。

每一格可以从其左上方，上方和右上方到达，因此有状态转移方程 A[i][j] += min({A[i - 1][j - 1], A[i - 1][j], A[i - 1][j + 1]})。

```c++
class Solution {
public:
    int minFallingPathSum(vector<vector<int>>& A) {
        int m = A.size(), n = m != 0 ? A[0].size() : 0, val = INT_MAX, res = INT_MAX;
        if (m == 0)
            return 0;
        for (int i = 1; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                val = A[i - 1][j];
                if (j < n - 1)
                    val = min(val, A[i - 1][j + 1]);
                if (j > 0)
                    val = min(val, A[i - 1][j - 1]);
                A[i][j] += val;
            }
        }
        return *min_element(A[m - 1].begin(), A[m - 1].end());
    }
};
```

### 8. 其他

#### [650 只有两个键的键盘](https://leetcode-cn.com/problems/2-keys-keyboard/)

有一个字符 'A'，只能进行复制和粘贴操作，求得到 n 个 'A' 的最小操作次数。

m 个 'A' 只能通过粘贴的操作得到，求出所有能整除 m 的数里通过复制粘贴操作得到 m 的最小次数即可。

```c++
class Solution {
public:
    int minSteps(int n) {
        vector<int> dp(n + 1, n);
        dp[1] = 0;
        for (int i = 1; i <= n; ++i) {
            int res = dp[i] + 1;
            for (int j = i; j <= n; j += i) {
                dp[j] = min(dp[j], res);
                ++res;
            }
        }
        return dp[n];
    }
};
```

#### [651 4键键盘](https://leetcode-cn.com/problems/4-keys-keyboard/submissions/)

一个键盘上有四个键：输入 'A'，选中全部，复制，和粘贴。可以按 N 次键盘，求最多能显示多少个 'A'。

因为 N 是最后一次操作，所以只能进行输入 'A' 和粘贴两种操作，只需要求出每一步在之前一步基础上输入 'A'，以及在往前三步的每一步基础上选中，复制，粘贴能得到的最2优解。

```c++
class Solution {
public:
    int maxA(int N) {
        vector<int> dp(N + 1, 0);
        for (int i = 1; i <= N; ++i) {
            dp[i] = dp[i - 1] + 1;
            for (int j = i - 1; j >= 2; --j)
                dp[i] = max(dp[i], dp[j - 2] * (i - j + 1));
        }
        return dp[N];
    }
};
```
