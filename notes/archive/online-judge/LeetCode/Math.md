---
title: "LeetCode 数学"
date: 2019-07-24T19:12:25+10:00
draft: true
categories: ["LeetCode"]
---

# [LeetCode 数学](https://leetcode-cn.com/tag/math/)

## 题目

### 1. 

#### [36 有效的数独](https://leetcode-cn.com/problems/valid-sudoku/submissions/)

用 9 * 3 个哈希表分别将当前行，列和 9 个格子内已经出现过的数字记录下来，并在之后在哈希表中进行判断是否出现过即可。

```c++
class Solution {
public:
    bool isValidSudoku(vector<vector<char>> &board) {
        unordered_set<int> row[9], col[9], box[9];
        for (int i = 0; i < 9; ++i) {
            row[i] = unordered_set<int>();
            col[i] = unordered_set<int>();
            box[i] = unordered_set<int>();
        }
        for (int i = 0; i < 9; ++i)
            for (int j = 0; j < 9; ++j) {
                if (board[i][j] == '.')
                    continue;
                int m = board[i][j] - '0';
                int box_index = i / 3 * 3 + j / 3;
                if (row[i].find(m) != row[i].end() || col[j].find(m) != col[j].end() ||
                    box[box_index].find(m) != box[box_index].end())
                    return false;
                row[i].insert(m);
                col[j].insert(m);
                box[box_index].insert(m);
            }
        return true;
    }
};
```