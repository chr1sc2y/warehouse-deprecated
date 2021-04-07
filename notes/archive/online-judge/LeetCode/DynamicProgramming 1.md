---
title: "LeetCode 动态规划（1）"
date: 2019-06-26T18:08:10+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 动态规划](https://leetcode-cn.com/problemset/all/?search=%E4%B8%91%E6%95%B0)

## 题目

### 1. 数字相关

#### [263 丑数](https://leetcode-cn.com/problems/ugly-number/)

判断一个数 num 是否是丑数。

通用的方法是自底向上求出大于等于 num 的第一个数来判断 num 是否是丑数。但这道题已经给出了数 num，直接通过模运算就能得到结果了。

```c++
class Solution {
public:
    bool isUgly(int num) {
        if (num < 1)
            return false;
        while (num % 2 == 0)
            num /= 2;
        while (num % 3 == 0)
            num /= 3;
        while (num % 5 == 0)
            num /= 5;
        return num == 1;
    }
};
```

#### [264 丑数 II](https://leetcode-cn.com/problems/ugly-number-ii/comments/)

求第 n 个丑数。

用一个数组 ugly 来保存前 m 个丑数，用三个质因数 2，3，5 乘以其当前系数对应的丑数，得到新的丑数，最小的一个就是第 m + 1 个丑数。时间复杂度是 O(m * n)，其中 m 是质因数的个数，n 是要找的第 n 个丑数。

```c++
class Solution {
public:
    int nthUglyNumber(int n) {
        vector<int> ugly(n, 1);
        int base_2 = 0, base_3 = 0, base_5 = 0;
        int m = INT_MAX;
        for (int i = 1; i < n; ++i) {
            ugly[i] = min({2 * ugly[base_2], 3 * ugly[base_3], 5 * ugly[base_5]});
            if (2 * ugly[base_2] == ugly[i])
                ++base_2;
            if (3 * ugly[base_3] == ugly[i])
                ++base_3;
            if (5 * ugly[base_5] == ugly[i])
                ++base_5;
            cout << ugly[i] << endl;
        }
        return ugly[n - 1];
    }
};
```

#### [313 超级丑数](https://leetcode-cn.com/problems/super-ugly-number/)

给定质因数数组 primes，求第 n 个丑数。

跟上一题完全相同，只是把原有的三个质因数 2，3，5 换成了一个数组。时间复杂度是 O(n * m)，其中 n 是第 n 个丑数，m 是数组的长度。

```c++
class Solution {
public:
    int nthSuperUglyNumber(int N, vector<int>& primes) {
        int n = primes.size(), m = INT_MAX;;
        vector<int> ugly(N, 1), base(n, 0);
        for (int i = 1; i < N; ++i) {
            m = INT_MAX;
            for (int j = 0; j < n; ++j)
                m = min(m, primes[j] * ugly[base[j]]);
            ugly[i] = m;
            for (int j = 0; j < n; ++j)
                if (primes[j] * ugly[base[j]] == m)
                    ++base[j];
        }
        return ugly[N - 1];
    }
};
```

#### [279 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

给一个数 n，其可以被表示为 m 个完全平方数的，找到最小的 m。

n 只能由比 n 小 1, 4, 9 等等的数的最优值加一得到，因此用一个数组 dp[n] 保存小于等于 n 的数被表示为 k 个完全平方数的和的最小的 k 的数量，根据状态转移方程 dp[n] = min({dp[n], dp[n - 1], dp[n - 4], dp[n - 9], ...}) + 1 计算得到结果。时间复杂度是 O(n * w)，w 是比 n 小的完全平方数的个数，空间复杂度是 O(n)。

```c++
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0, dp[1] = 1;
        for (int i = 2; i <= n; ++i)
            for (int j = 1; i - j * j >= 0; ++j)
                dp[i] = min(dp[i], dp[i - j * j] + 1);
        return dp[n];
    }
};
```

#### [343 整数拆分](https://leetcode-cn.com/problems/integer-break/)

给一个数 n，将其拆分为至少两个数的和，求这些整数的最大乘积。

n 可以被拆分为 2 个数的和，这两个数又可以被拆分为若干个数的和，因此只需要知道其被拆分为两个数时这两个数的最大乘积，自下往上地计算小于等于 n 的数被拆分时的最大乘积即可，状态转移方程是 dp[i] = max(dp[i], dp[j] * dp[i - j])。时间复杂度是 O(n ^ 2)，空间复杂度是 O(n)。

```c++
class Solution {
public:
    int integerBreak(int n) {
        vector<int> dp(n + 1, 0);
        if (n < 3)  return 1;
        if (n == 3) return 2;
        dp[2] = 2, dp[3] = 3;
        for (int i = 4; i <= n; ++i)
            for (int j = 1, k = i - j; j <= k; ++j, --k)
                dp[i] = max(dp[i], dp[j] * dp[k]);
        return dp[n];
    }
};
```

#### [1155 掷骰子的 N 种方法](https://leetcode-cn.com/problems/number-of-dice-rolls-with-target-sum/)

对于每一个骰子来说，它可以在之前的基础上有 f 种投掷的方法，它之后的状态是 dp[i + 1][j + k]，i 是投掷过的骰子的个数，k 是它投掷不同的 f 种方法，j 是到它为止投掷出和为 j 的种数，自下而上动态规划即可。

```c++
class Solution {
public:
    int numRollsToTarget(int d, int f, int target) {
        int dp[d + 1][target + f + 1];
        memset(dp, 0, sizeof(dp));
        dp[0][0] = 1;
        for (int i = 0; i < d; ++i)
            for (int j = 0; j < target; ++j)
                if (dp[i][j])
                    for (int k = 1; k <= f; ++k)
                        if (j + k <= target)
                            dp[i + 1][j + k] = (dp[i + 1][j + k] + dp[i][j]) % 1000000007;
        return dp[d][target];
    }
};
```

### 2. 买卖股票

#### [121 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

给一个股票数组，只能进行一次交易，求最大利润。

最大化当前值与之前的最小值之差。时间复杂度是 O(n)。

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int min_val = INT_MAX, res = 0;
        for (auto &p:prices) {
            min_val = min(min_val, p);
            res = max(res, p - min_val);
        }
        return res;
    }
};
```

#### [122 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

给一个股票数组，不限制交易次数，求最大利润。

每一个严格递增的区间都是交易的时机，所以将严格递增区间内的差值全部加上即可。时间复杂度是 O(n)。

```c++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int res = 0;
        for (int i = 1; i < prices.size(); ++i)
            res += max(prices[i] - prices[i - 1], 0);
        return res;
    }
};
```

#### [123 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

给一个股票数组，只能进行两笔交易，求最大利润。

因为要进行两笔交易，所以需要最大化两个值：一个是到第 i 天为止的最大收益，一个是第 i 天之后的最大收益。第一种方法是两次遍历，第一次计算到第 i 天为止的最大收益，第二次反向遍历计算第 i 天之后的最大收益，方法跟第一题相同，注意第二次买入操作必须在第一次卖出操作之后，不能发生在同一天。时间复杂度是 O(n)。

```c++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int n = prices.size(), pre_min = INT_MAX, post_max = 0, res = 0;
        vector<int> pre(n, 0), post(n, 0);
        for (int i = 0; i < n - 1; ++i) {
            pre_min = min(pre_min, prices[i]);
            pre[i] = max(pre[max(0, i - 1)], prices[i] - pre_min);
        }
        for (int i = n - 1; i >= 0; --i) {
            post_max = max(post_max, prices[i]);
            post[i] = max(post[min(n - 1, i + 1)], post_max - prices[i]);
        }
        for (int i = 0; i < n; ++i)
            res = max(res, pre[i] + post[i]);
        return res;
    }
};
```

第二种方法则是基于每天只有四种可能的操作：第一次买入，第一次卖出 res1，第二次买入，和第二次卖出 res2。第一次买入需要最大化之前买入股票的最小花费，第一次卖出需要最大化到第 i 天为止的股票价格与第一次买入的差值，第二次买入需要最大化在第 i 天买入股票并减去第一次卖出的收益，最后第二次卖出需要最大化到第 i 天为止的股票价格与第二次买入的差值。最后得到第二次卖出的最优值。时间复杂度是 O(n)。

```c++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int buy1 = INT_MIN, sell1 = 0, buy2 = INT_MIN, sell2 = 0;
        for (auto &p:prices) {
            buy1 = max(buy1, -p);
            sell1 = max(sell1, p + buy1);
            buy2 = max(buy2, sell1 - p);
            sell2 = max(sell2, p + buy2);
            cout << buy1 << ' ' << sell1 << ' ' << buy2 << ' ' << sell2 << ' ' << endl;
        }
        return sell2;
    }
};
```

#### [309 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

给一个股票数组，不限制交易次数，卖出和买入之间需要隔一天，求最大利润。

和上一题的第二种方法类似，我们可以用两个数组 buy 和 sell 分别表示买入和卖出操作，对于买入操作，需要最大化两天前卖出的最优值于今天买入的差值，对于卖出操作，需要最大化当天价格与前一天买入的最优值的差值。时间复杂度是 O(n)。

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n < 2)
            return 0;
        vector<int> buy(n, 0), sell(n, 0);
        buy[0] = -prices[0], sell[0] = 0;
        for (int i = 1; i < n; ++i) {
            buy[i] = max(buy[i - 1], sell[max(0, i - 2)] - prices[i]);
            sell[i] = max(sell[i - 1], buy[i - 1] + prices[i]);
        }
        return sell[n - 1];
    }
};
```

#### [714 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

给一个股票数组，不限制交易次数，每次卖出有一定手续费，求最大利润。

和上一题类似，区别在于没有了交易间隔，以及每次进行 sell 操作的时候需要减去手续费。时间复杂度是 O(n)。

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();
        if (n < 2)
            return 0;
        vector<int> buy(n, 0), sell(n, 0);
        buy[0] = -prices[0];
        for (int i = 1; i < n; ++i) {
            buy[i] = max(buy[i - 1], sell[i - 1] - prices[i]);
            sell[i] = max(sell[i - 1], buy[i - 1] + prices[i] - fee);
        }
        return sell[n - 1];
    }
};
```

#### [188 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

给一个股票数组，最多能进行 k 笔交易，求最大利润。

把上面五道题都做完之后这道题就很简单了。相比于第二题，因为这道题的 k 是未知的，所以要用一个循环将所有可能的 k 笔交易的最优值都计算出来，因此用一个三维数组 dp[n][k][2]，或是分开成两个二维数组 buy[n][k] 和 sell[n][k]，来表示前 n 天进行 k 笔交易的最优的买入和卖出值。同样的，买入和卖出操作都是在之前一次的卖出和买入的基础上进行的，使用 buy[i][j] = max({buy[i][j - 1], buy[i - 1][j], sell[i - 1][j - 1] - prices[i]}) 来表示前 i 天进行了 j 次交易的最优的买入值，第一项在 j <= i / 2 时会填充为 buy[i][j - 1]的值，防止后面操作时不会取到空值或默认值造成错误，第二项是当前买入操作不能取得最优值的结果，第三项则是当前买入操作能取得最优值的结果；对应的卖出操作则是 sell[i][j] = max({sell[i][j - 1], sell[i - 1][j], buy[i - 1][j] + prices[i]})。

值得注意的是，当 k 远大于数组长度的两倍，或 k 非常大时，构造二维数组会造成MLE，此时可以直接用第二题的思路解决。时间复杂度是 O(n * k)，空间复杂度是 O(n * k)。

```c++
class Solution {
public:
    int maxProfit(int k, vector<int> &prices) {
        int n = prices.size(), res = 0;
        if (n < 2 || k == 0)
            return 0;
        if (k >= n * 2) {
            for (int i = 1; i < n; ++i)
                res += max(0, prices[i] - prices[i - 1]);
            return res;
        }
        vector<vector<int>> buy(n, vector<int>(k, INT_MIN)), sell(n, vector<int>(k, 0));
        buy[0][0] = -prices[0], sell[0][0] = 0;
        for (int i = 1; i < n; ++i) {
            buy[i][0] = max(buy[i - 1][0], -prices[i]);
            sell[i][0] = max(sell[i - 1][0], buy[i - 1][0] + prices[i]);
            for (int j = 1; j < k; ++j) {
                buy[i][j] = max({buy[i][j - 1], buy[i - 1][j], sell[i - 1][j - 1] - prices[i]});
                sell[i][j] = max({sell[i][j - 1], sell[i - 1][j], buy[i - 1][j] + prices[i]});
            }
        }
        return sell[n - 1][k - 1];
    }
};
```
