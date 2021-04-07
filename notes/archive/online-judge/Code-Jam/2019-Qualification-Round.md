---
title: "Code Jam 2019 Qualification Round"
date: 2019-04-06T13:40:27+10:00
draft: false
categories: ["Code Jam"]
---

# [Code Jam 2019 Qualification Round](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705)

## [Foregone Solution (6pts, 10pts, 1pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/0000000000088231)

将一个带有数字4的数拆分为两个不带数字4的数。

### Solution: Construction

输入的数一定带有数字4，对于每一位上的数字4，我们可以将其拆分为2+2（或1+3）的两个数。输入数据最大是10的100次方，所以我们可以将其作为字符串处理。

- 时间复杂度：O(n)
- 空间复杂度：O(1)

```C++
// C++
#include <iostream>
#include <cmath>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;

int main() {
    int T;
    cin >> T;
    for (int t = 1; t <= T; ++t) {
        string N;
        cin >> N;
        string a, b;
        for (auto c:N) {
            a += c == '4' ? '2' : c;
            b += c == '4' ? '2' : '0';
        }
        while (a[0] == '0')
            a.erase(a.begin());
        while (b[0] == '0')
            b.erase(b.begin());
        cout << "Case #" << t << ": " << a << " " << b << endl;
    }
}
```

## [You Can Go Your Own Way (5pts, 9pts, 10pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/00000000000881da)

在n*n的矩阵里从(0,0)走到(n-1,n-1)，只能向右或向下走。矩阵里有一条已有的路径，不能与该路径有重合。最常规的做法是DFS/BFS，时间复杂度为O(n^2)。

### Solution: Mirror

因为只有一条已知路径，我们可以将其以对角线作镜像，得到的新路径一定与原路径没有重合。

- 时间复杂度：O(2 * n - 2)
- 空间复杂度：O(1)

```C++
// C++
#include <iostream>
#include <cmath>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;

int main() {
    int T;
    cin >> T;
    for (int t = 1; t <= T; ++t) {
        int N;
        string str;
        cin >> N >> str;
        for (int i = 0; i < N * 2 - 2; ++i)
            str[i] = str[i] == 'E' ? 'S' : 'E';
        cout << "Case #" << t << ": " << str << endl;
    }
}
```

## [Cryptopangrams (10pts, 15pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/000000000008830b)

输入上限N和一个长为L的数组product，数组里的每一个数都是一个长为L+1的质数数组res里的相邻质数的乘积。质数数组里一共只有26个质数，返回将这26个质数排序后分别映射为A-Z的结果。对于比较小的N，可以把小于等于N的所有质数保存下来，计算出product[0]是哪两个质数的乘积，再计算出product[1]是哪两个质数的乘积，得到res[1]，再依次计算质数数组res里的其他数。在N比较大的时候时间和空间占用都会很高。

### Solution: Greatest Common Denominator

相较于先找到所有质数再找到product[0]是哪两个质数的乘积，我们可以使用小学学过的辗转相除法来求出这个质数，这样可以非常有效地降低时间和空间消耗。

这道题测试用例的N最大值是10的100次方，用C++需要自己处理大数乘法（C++最大只支持128位的int型数）。

- 时间复杂度：O(L)
- 空间复杂度：O(L)

```Python
# Python 3
def GCD(a: int, b: int) -> int:
    if b == 0:
        return a
    return GCD(b, a % b)


T = int(input())
for t in range(1, T + 1):
    N, L = map(int, input().split())
    product = list(map(int, input().split()))
    res = [0 for _ in range(L + 1)]
    pos = -1
    for i in range(L - 1):
        if product[i] != product[i + 1]:
            res[i + 1] = GCD(product[i], product[i + 1])
            pos = i + 1
            break
    for i in range(pos + 1, L + 1):
        res[i] = product[i - 1] // res[i - 1]
    for i in range(pos - 1, -1, -1):
        res[i] = product[i] // res[i + 1]
    match = []
    for m in res:
        if m not in match:
            match.append(m)
    match.sort()
    ret = [chr(match.index(res[i]) + ord('A')) for i in range(len(res))]
    print("Case #{t}: {str}".format(t=t, str="".join(ret)))
```

## [Dat Bae (14pts, 20pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/00000000000881de)

// TODO