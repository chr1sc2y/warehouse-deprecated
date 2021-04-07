---
title: "LeetCode 位运算"
date: 2019-06-19T19:26:39+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 位运算](https://leetcode-cn.com/tag/bit-manipulation/)

位运算包括：

1. 与 &
2. 或 |
3. 异或 ^
4. 取反 ~
5. 左移 <<
6. 右移 >>

## 技巧

1. 移位运算

    - x << 1：算数左移
        - 数字的二进制表示的所有位向左移动一位，相当于乘以 2
        - 在右边补 0
    - x >> 1：算数右移
        - 数字的二进制表示的所有位向右移动一位，相当于除以 2
        - 在左边补符号位，即正数补 0，负数（在补码的基础上）补 1
    - 负数移位
        - 负数是以补码的形式存储的，负数进行右移运算时需要将其取反转换成反码，加一转换成补码，再将其向右移动一位得到新的补码，再将其减一得到新的反码，再取反转换成原码才能得到结果。例如 -7 的二进制表示是 10000111（因为 32 位太长所以这里用 8 位 int 型表示），其反码是 11111000，补码是11111001，向右移动一位是 11111100，减一得到新的反码 11111011，原码是10000100，也就是 -4；补码向左移一位是 11110010，减一得到新的反码 11110001，原码是 10001110，也就是 -14
        - 比较简单的理解方式是左移乘以 2，右移除以 2。例如 -7 >> 1 = -7 / 2 = -4，-7 << 1 = -14

## 题目

### 1. 单个数字

#### [693 交替位二进制数](https://leetcode-cn.com/problems/binary-number-with-alternating-bits/)

检查一个二进制数相邻的两个位数是否均不相等。

逐位 & 1 进行判断即可。

```c++
class Solution {
public:
    bool hasAlternatingBits(int n) {
        bool rel = n & 1;
        while (n > 0 && (n & 1) == rel) {
            rel = !rel;
            n >>= 1;
        }
        return n <= 0;
    }
};
```

#### [476 数字的补数](https://leetcode-cn.com/problems/number-complement/)

给一个正整数，求二进制表示取反的结果，取反不包括前导 0。

对每一位异或 1 即可。将大于 num 的第一个 2 ^ n 的数减一即可得到二进制表示全是 1 的，位数等于 num 的位数的数。

```c++
class Solution {
public:
    int findComplement(int num) {
        long long util = 1;
        while (util <= num)
            util <<= 1;
        return num ^ (util - 1);
    }
};
```

#### [461 汉明距离](https://leetcode-cn.com/problems/hamming-distance/)

计算两个整数的二进制表示在各位上不同的数目。

逐位进行异或判断即可。

```c++
class Solution {
public:
    int hammingDistance(int x, int y) {
        int res = 0;
        while (x > 0 || y > 0) {
            res += ((x & 1) ^ (y & 1));
            x >>= 1, y >>= 1;
        }
        return res;
    }
};
```

#### [762 二进制表示中质数个计算置位](https://leetcode-cn.com/problems/prime-number-of-set-bits-in-binary-representation/)

计算 [L, R] 中置位位数为质数的个数。

先逐位判断数 i 在当前位上是否等一 1，得到置位位数后判断是否是质数。判断质数的时候可以先排除掉模以 6 等于 0, 2, 3, 4 的情况，因为这几种情况分别可以被 6, 2, 3, 2 整除，剩下的再判断其模以 6n - 1 和 6n + 1 是否等于 0 即可。还可以先把小于等于 32 的质数存储在哈希表或数组中，这样的话查询时间会降低一些。时间复杂度是 O(n)，n 是 [L, R] 的个数。

```c++
class Solution {
public:
    int countPrimeSetBits(int L, int R) {
        int res = 0;
        for (int i = L; i <= R; ++i) {
            int m = i;
            int cnt = 0;
            while (m > 0) {
                if (m & 1)
                    ++cnt;
                m >>= 1;
            }
            if (IsPrime(cnt))
                ++res;
        }
        return res;
    }
    
    bool IsPrime(int num) {
        if (num < 4)
            return num > 1;
        else if (num % 6 != 1 && num % 6 != 5)
            return false;
        for (int i = 5; i <= sqrt(num); i += 6)
            if (num % i == 0 || num % (i + 2) == 0)
                return false;
        return true;
    }
};
```

#### [136 只出现一次的数字](https://leetcode-cn.com/problems/single-number/)

给一个数组，其他数都出现了两次，只有一个数出现了一次，找到这个数。

因为 a ^ a = 0，所以可以用 0 异或这个数组中的所有数，最后得到的就是唯一的一个数。

```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        for (auto &m:nums)
            res ^= m;
        return res;
    }
};
```