---
title: "LeetCode 二分查找"
date: 2019-06-23T19:26:39+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode 二分查找](https://leetcode-cn.com/tag/binary-search/)

二分查找可以在有序数组中以较高的效率查找符合条件的值，时间复杂度是O(logN)，空间复杂度是O(1)。

## 易错点

1. 计算中间值的方法

    - k = i + (j - i) / 2
    - k = (i + j) / 2

    第二种方法一般都会造成整型数据溢出，所以只用第一种方法。

2. 循环条件

    - 如果要找唯一的值，且下限i和上限j都会在更新时在中间值k的基础上+1或-1，那么循环条件是 i <= j，j能被取到，j = nums.size() - 1，计算中间值用 k = i + (j - i + 1) / 2

    - 如果要找大于等于或小于等于某条件的值，且下限i和上限j其中之一不会在k的基础上+1或-1，那么循环条件是 i < j，j不能被取到，j = nums.size()，计算中间值用 k = i + (j - i) / 2

    两种方法有各自的应用场景，没用对的话会出现边界值的问题，或是死循环导致TLE。

## 题目

### 1. 查找数

#### [704 二分查找](https://leetcode-cn.com/problems/binary-search/)

在有序数组中搜索目标值，如果存在则返回下标，否则返回 -1。

最标准的二分查找。因为下限i和上限j都会在更新时+1和-1，所以让j = nums.size() - 1，循环条件是 i <= j。

```c++
class Solution {
public:
    int search(vector<int> &nums, int target) {
        int i = 0, j = nums.size() - 1, k = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            if (nums[k] > target)
                j = k - 1;
            else if (nums[k] < target)
                i = k + 1;
            else
                return k;
        }
        return -1;
    }
};
```

#### [374 猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower/)

从 1 到 n 选择一个数字，通过一个预定义的接口 guess(int num)得到-1：数字打了；1：数字小了；0：猜对了。

跟上一题几乎一模一样，只是把target换成了一个接口。

```c++
int guess(int num);

class Solution {
public:
    int guessNumber(int n) {
        int i = 1, j = n, k = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            auto res = guess(k);
            if (res == 1)
                i = k + 1;
            else if (res == -1)
                j = k - 1;
            else
                return k;
        }
        return 0;
    }
};
```

#### [367 有效的完全平方数](https://leetcode-cn.com/problems/valid-perfect-square/)

判断一个正整数是否是一个完全平方数。

判断一个数是否是完全平方数。因为在二分的过程中平方得到的结果可能会超过32位int型的上限，所以用long long。

```c++
class Solution {
public:
    bool isPerfectSquare(int num) {
        long long i = 0, j = num, k = 0, res = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            res = k * k;
            if (res < num)
                i = k + 1;
            else if (res > num)
                j = k - 1;
            else
                return true;
        }
        return j * j == num;
    }
};
```

#### [69 x的平方根](https://leetcode-cn.com/problems/sqrtx/)

计算一个数的平方根，只保留整数部分。

实现int sqrt(int x)函数。用商来判断比用乘积来判断更直观。

```c++
class Solution {
public:
    int mySqrt(int x) {
        if (x <= 1)
            return x;
        int i = 1, j = x, k = 0, sqrt = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            sqrt = x / k;
            if (sqrt < k)
                j = k - 1;
            else if (sqrt > k)
                i = k + 1;
            else
                return k;
        }
        return j;
    }
};
```

### 2. 查找上界/下界

#### [35 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)

在有序数组中找到目标值，如果不存在则返回它将会被按顺序插入的位置，数组中无重复元素。

找到大于等于target的第一个数。因为上限j在更新时会直接被赋给k的值，所以让j = nums.size()，循环条件是i < j。

```c++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int i = 0, j = nums.size(), k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            if (nums[k] < target)
                i = k + 1;
            else if (nums[k] > target)
                j = k;
            else
                return k;
        }
        return j;
    }
};
```

#### [744 寻找比目标字母大的最小字母](https://leetcode-cn.com/problems/find-smallest-letter-greater-than-target/)

在有序数组中找到比目标字母大的最小字母，数组里的字母是循环的。

找到大于target的第一个数，相比于上一题只是少了在循环内判断是否等于的情况。从int类型数组变成了char类型数组，不过这一点并没有任何影响。

```c++
class Solution {
public:
    char nextGreatestLetter(vector<char>& letters, char target) {
        int n = letters.size(), i = 0, j = n, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            if (letters[k] <= target)
                i = k + 1;
            else if (letters[k] > target)
                j = k;
        }
        return j < n ? letters[j] : letters[0];
    }
};
```

#### [278 第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version/)

产品都是基于之前的版本开发的，所有错误的版本之后的所有版本都是错的，找到出错的第一个版本。通过一个接口 bool isBadVersion(version) 来判断版本是否出错。

很标准的找下界。

```c++
bool isBadVersion(int version);

class Solution {
public:
    int firstBadVersion(long long n) {
        long long i = 0, j = n + 1, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            if (!isBadVersion(k))
                i = k + 1;
            else
                j = k;
        }
        return j;
    }
};
```

#### [875 爱吃香蕉的珂珂](https://leetcode-cn.com/problems/koko-eating-bananas/)

有 N 堆香蕉，每小时内吃完一堆则不吃另外一堆，计算能在 H 小时内吃完的最慢速度。

将速度作为二分查找的变量，每次判断以当前速度是否能吃完所有香蕉，如果能则 j = k，k 有可能是最后的结果，否则 i = k + 1，此时的 k 一定比结果小。

```c++
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int H) {
        int i = 1, j = INT_MAX, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            int res = CanEatAll(k, H, piles);
            if (res)
                j = k;
            else
                i = k + 1;
        }
        return j;
    }
    
    bool CanEatAll(const int &speed, int hour, vector<int>& piles) {
        for (auto &p:piles)
            hour -= p / speed + (p % speed > 0);
        return hour >= 0;
    }
};
```

### 3. 根据位置关系查找

#### [378 有序矩阵中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix/)

n x n 的矩阵中每行和每列均按升序排序，找到矩阵中第k小的元素。

这道题可以用跟剑指offer里二维数组中的查找这道题的思路结合二分查找来做，二分的时候每次计算矩阵里小于等于中间值的数的个数就能得到结果了。时间复杂度是 O(logm * n)，m 是 matrix[n - 1][n - 1]，n 是 matrix.size()。

```c++
class Solution {
public:
    int kthSmallest(vector<vector<int>>& matrix, int m) {
        int n = matrix.size(), i = matrix[0][0], j = matrix[n - 1][n - 1] + 1, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            auto res = CountLess(matrix, k);
            if (res < m)
                i = k + 1;
            else
                j = k;
        }
        return i;
    }
    
    int CountLess(vector<vector<int>>& matrix, const int &target) {
        int n = matrix.size(), i = 0, j = n - 1, res = 0;
        while (i < n && j >= 0) {
            if (matrix[i][j] <= target)
                res += j + 1, ++i;
            else
                --j;
        }
        return res;
    }
};
```

#### [153 寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

一个有序数组在某个点上进行了旋转，找出其中最小的元素。

根据中间值与左右边值和其左右边一位的大小关系来判断，如果中间值k比左边值小，那么k有可能是结果，让 j = k ；否则比较中间值与其右边一位，如果中间值大于右边一位的值，那么右边一位的值就是结果，否则让 i = k + 1。

```c++
class Solution {
public:
    int findMin(vector<int>& nums) {
        int n = nums.size(), i = 0, j = nums.size(), k = 0;
        if (n == 1 || nums[0] < nums[n - 1])
            return nums[0];
        else if (n == 2)
            return min(nums[0], nums[1]);
        while (i < j) {
            k = i + (j - i) / 2;
            if (nums[i] > nums[k])
                j = k;
            else if (k + 1 < j && nums[k + 1] > nums[i])
                i = k + 1;
            else
                return nums[k + 1];
        }
        return j >= n ? nums[i] : min(nums[i], nums[j]);
    }
};
```

#### [540 有序数组中的单一元素](https://leetcode-cn.com/problems/single-element-in-a-sorted-array/)

一个有序数组中每个元素都会出现了两次，只有一个数出现了一次，找出这个数。

有序数组里其他每个元素都出现两次，找到唯一只出现一次的数。在唯一的数出现之后奇偶位的相等关系会发生变化，利用这一点来做判断。比较直观的写法是分别判断 k % 2 == 0 和 k % 2 == 1 的情况，这样写起来比较复杂，可以直接在 k % 2 == 1 的时候 --k，再直接判断 nums[k] 和 nums[k + 1] 的关系。

```c++
class Solution {
public:
    int singleNonDuplicate(vector<int> &nums) {
        int i = 0, j = nums.size() - 1, k = 0, n = nums.size();
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            cout << k << endl;
            if (k % 2 == 0) {
                if (k + 1 < n && nums[k] == nums[k + 1])
                    i = k + 1;
                else if (k - 1 >= 0 && nums[k] == nums[k - 1])
                    j = k - 1;
                else
                    return nums[k];
            } else {
                if (k + 1 < n && nums[k] == nums[k + 1])
                    j = k - 1;
                else if (k - 1 >= 0 && nums[k] == nums[k - 1])
                    i = k + 1;
                else
                    return nums[k];
            }
        }
        return 0;
    }
};
```

```c++
class Solution {
public:
    int singleNonDuplicate(vector<int> &nums) {
        int i = 0, j = nums.size(), k = 0, n = nums.size();
        while (i < j) {
            k = i + (j - i) / 2;
            if (k % 2 == 1)
                --k;
            if (k + 1 < n && nums[k] == nums[k + 1])
                i = k + 2;
            else
                j = k;
        }
        return nums[j];
    }
};
```

### 4. 综合

#### [1095 山脉数组中查找目标值](https://leetcode-cn.com/problems/find-in-mountain-array/)

在一个山脉数组中找到等于目标值的最小下标。

因为已知数组一定是一个山脉数组，所以一定有唯一的山顶，用二分查找找到山顶的下标。如果山顶小于目标值那么数组中一定没有目标值存在，返回 -1，否则在山顶左边使用二分查找找目标值，没有的话则在山顶右边使用二分查找找目标值。

```c++
class Solution {
public:
    int findInMountainArray(int target, MountainArray &mountainArr) {
        int n = mountainArr.length(), i = 0, j = n - 1, k = 0, peak = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            if (mountainArr.get(k) > mountainArr.get(k - 1) && mountainArr.get(k) > mountainArr.get(k + 1))
                break;
            else if (mountainArr.get(k) < mountainArr.get(k - 1))
                j = k - 1;
            else
                i = k + 1;
        }
        if (mountainArr.get(k) < target)
            return -1;
        else if (mountainArr.get(k) == target)
            return k;
        
        peak = k;
        i = 0, j = peak;
        while (i < j) {
            k = i + (j - i) / 2;
            if (mountainArr.get(k) < target)
                i = k + 1;
            else
                j = k;
        }
        if (mountainArr.get(j) == target)
            return j;
        
        i = peak + 1, j = n;
        while (i < j) {
            k = i + (j - i) / 2;
            if (mountainArr.get(k) < target)
                j = k - 1;
            else if (mountainArr.get(k) > target)
                i = k + 1;
            else
                j = k;
        }
        if (i < n && mountainArr.get(i) == target)
            return i;
        return -1;
    }
};
```

### 5. 猜数

#### [719 找出第 k 小的距离对](https://leetcode-cn.com/problems/find-k-th-smallest-pair-distance/)

最简单的做法遍历两遍算出所有数对的差值保存在一个小根堆中，然后依次找到小于等于 k 的最小距离，但这样做时间复杂度是 O(n ^ 2) 会超时。我们可以先将数组排序，然后用 low = 0, high = nums[n - 1] - nums[0] 表示可能结果的最小值和最大值，用二分法来判断中间值 mid 是否满足小于等于 mid 的数对差值的数是否小于等于 k，判断时因为数组是有序的，可以使用双指针让中间的差值维持在小于等于 mid，从而计算出小于等于 mid 的数对差值的数，时间复杂度是 O(nlogn + nlogm)，其中 n 是数组的长度，m 是数组最大值与最小值之差，nlogn 是排序的平均时间复杂度，而 nlogm 中 n 是使用双指针判断的时间复杂度，logm 是二分法的时间复杂度。

```c++
class Solution {
public:
    int smallestDistancePair(vector<int> &nums, int k) {
        sort(nums.begin(), nums.end());
        int n = nums.size(), low = 0, high = nums[n - 1] - nums[0], mid = 0;
        while (low < high) {
            mid = low + (high - low) / 2;
            if (IsDistMoreThanK(nums, mid, k))
                high = mid;
            else
                low = mid + 1;
        }
        return high;
    }

    bool IsDistMoreThanK(const vector<int> &nums, int m, const int &k) {
        int left = 0, right = 0, count = 0;
        while (right < nums.size()) {
            while (nums[right] - nums[left] > m)
                ++left;
            count += right - left;
            ++right;
        }
        return count >= k;
    }
};
```