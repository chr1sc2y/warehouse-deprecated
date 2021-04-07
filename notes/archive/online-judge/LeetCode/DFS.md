---
title: "LeetCode 深度优先搜索"
date: 2019-07-27T12:12:25+10:00
draft: false
categories: ["LeetCode"]
---

# [LeetCode DFS](https://leetcode-cn.com/tag/depth-first-search/)

## 题目

#### [78 子集](https://leetcode-cn.com/problems/subsets/)

典型的回溯，找出所有可能情况。

```c++
class Solution {
    vector<vector<int>> res;
public:
    vector<vector<int>> subsets(vector<int> &nums) {
        res = vector<vector<int>>(1, vector<int>());
        vector<int> curr;
        DFS(nums, 0, curr);
        return res;
    }

    void DFS(vector<int> &nums, int idx, vector<int> &curr) {
        for (int i = idx; i < nums.size(); ++i) {
            curr.push_back(nums[i]);
            res.push_back(curr);
            DFS(nums, i + 1, curr);
            curr.pop_back();
        }
    }
};
```

#### [733 图像渲染](https://leetcode-cn.com/problems/flood-fill/)

从给定的 image[sr][sc] 开始 DFS 或 BFS，将相邻的值相同的点的值全部修改为 newColor，注意要判断给定的 image[sr][sc] 是否等于 newColor，否则如果不使用额外空间的 visited 数组记录已经访问过的点的话会造成死循环栈溢出。

```c++
class Solution {
    int m, n;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
    vector<vector<bool>> visited;
public:
    vector<vector<int>> floodFill(vector<vector<int>> &image, int sr, int sc, int newColor) {
        m = image.size(), n = m ? image[0].size() : 0;
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        if (image[sr][sc] != newColor)
            DFS(image, sr, sc, newColor);
        return image;
    }

    void DFS(vector<vector<int>> &image, int r, int c, int val) {
        int ori = image[r][c];
        image[r][c] = val;
        for (int d = 0; d < 4; ++d) {
            int i = r + dir[d][0];
            int j = c + dir[d][1];
            if (i >= 0 && i < m && j >= 0 && j < n && image[i][j] == ori)
                DFS(image, i, j, val);
        }
    }
};
```

#### [463 岛屿的周长](https://leetcode-cn.com/problems/island-perimeter/)

对小岛进行 DFS，根据一个点周围有几个相邻的点来计算当前点的周长。

```c++
class Solution {
    int x, y, res;
    vector<vector<bool>> visited;
    int dir[4][2] = {{0,  1},
                     {1,  0},
                     {0,  -1},
                     {-1, 0}};
public:
    int islandPerimeter(vector<vector<int>> &grid) {
        x = grid.size(), y = x ? grid[0].size() : 0, res = 0;
        visited = vector<vector<bool>>(x, vector<bool>(y, false));
        if (x == 0)
            return 0;
        for (int i = 0; i < x; ++i)
            for (int j = 0; j < y; ++j)
                if (grid[i][j] == 1) {
                    DFS(grid, i, j);
                    return res;
                }
        return res;
    }

    void DFS(vector<vector<int>> &grid, int i, int j) {
        visited[i][j] = true;
        int edge = 4;
        for (int l = 0; l < 4; ++l) {
            int a = i + dir[l][0];
            int b = j + dir[l][1];
            if (a >= 0 && a < x && b >= 0 && b < y && grid[a][b] == 1) {
                --edge;
                if (!visited[a][b])
                    DFS(grid, a, b);
            }
        }
        res += edge;
    }
};
```

#### [200 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

每次进行 DFS 的全部节点即为一个岛屿，DFS 完整个数组即可。

```c++
class Solution {
    vector<vector<bool>> visited;
    int x, y, res;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
public:
    int numIslands(vector<vector<char>> &grid) {
        x = grid.size(), y = x ? grid[0].size() : 0, res = 0;
        visited = vector<vector<bool>>(x, vector<bool>(y, 0));
        if (x == 0)
            return 0;
        for (int i = 0; i < x; ++i)
            for (int j = 0; j < y; ++j)
                if (grid[i][j] == '1' && !visited[i][j]) {
                    ++res;
                    DFS(grid, i, j);
                }
        return res;
    }

    void DFS(vector<vector<char>> &grid, int i, int j) {
        visited[i][j] = true;
        for (int d = 0; d < 4; ++d) {
            int a = i + dir[d][0];
            int b = j + dir[d][1];
            if (a >= 0 && a < x && b >= 0 && b < y && grid[a][b] == '1' && !visited[a][b])
                DFS(grid, a, b);
        }
    }
};
```

#### [695 岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island/)

对每个岛屿进行 DFS，每次都更新最大面积即可。

```c++
class Solution {
    vector<vector<bool>> visited;
    int x, y, res;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
public:
    int maxAreaOfIsland(vector<vector<int>> &grid) {
        x = grid.size(), y = x ? grid[0].size() : 0, res = 0;
        visited = vector<vector<bool>>(x, vector<bool>(y, 0));
        if (x == 0)
            return 0;
        for (int i = 0; i < x; ++i)
            for (int j = 0; j < y; ++j)
                if (grid[i][j] == 1 && !visited[i][j]) {
                    int area = 1;
                    DFS(grid, i, j, area);
                }
        return res;
    }

    void DFS(vector<vector<int>> &grid, int i, int j, int &area) {
        visited[i][j] = true;
        res = max(res, area);
        for (int d = 0; d < 4; ++d) {
            int a = i + dir[d][0];
            int b = j + dir[d][1];
            if (a >= 0 && a < x && b >= 0 && b < y && grid[a][b] == 1 && !visited[a][b]) {
                ++area;
                DFS(grid, a, b, area);
            }
        }
    }
};
```

#### [841 钥匙和房间](https://leetcode-cn.com/problems/keys-and-rooms/)

对每个房间进行 DFS。

```c++
class Solution {
    vector<bool> visited;
    int m, n;
public:
    bool canVisitAllRooms(vector<vector<int>> &rooms) {
        m = n = rooms.size();
        visited = vector<bool>(n, false);
        return DFS(rooms, 0);
    }

    bool DFS(vector<vector<int>> &rooms, int room_num) {
        --m;
        visited[room_num] = true;
        if (m == 0)
            return true;
        for (auto &r:rooms[room_num])
            if (!visited[r] && DFS(rooms, r))
                return true;
        return false;
    }
};
```

#### [113 路径总和 II](https://leetcode-cn.com/problems/path-sum-ii/)

对整个树进行 DFS，在叶子节点进行判断。

```c++
class Solution {
    vector<vector<int>> res;
public:
    vector<vector<int>> pathSum(TreeNode *root, int sum) {
        res = vector<vector<int>>();
        vector<int> path;
        DFS(root, sum, path);
        return res;
    }

    void DFS(TreeNode *root, int sum, vector<int> &path) {
        if (!root)
            return;
        path.push_back(root->val);
        if (!root->left && !root->right) {
            if (sum - root->val == 0)
                res.push_back(path);
        } else {
            DFS(root->left, sum - root->val, path);
            DFS(root->right, sum - root->val, path);
        }
        path.pop_back();
    }
};
```

#### [130 被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions/)

对最外围的所有 'O' 进行 DFS 并进行标记，最后在遍历一遍整个矩阵，将所有未被标记的 'O' 改为 'X'。

```c++
class Solution {
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
    int m, n;
public:
    void solve(vector<vector<char>> &board) {
        m = board.size(), n = m ? board[0].size() : 0;
        for (int i = 0; i < m; ++i) {
            if (board[i][0] == 'O')
                DFS(board, i, 0);
            if (board[i][n - 1] == 'O')
                DFS(board, i, n - 1);
        }
        for (int j = 1; j < n - 1; ++j) {
            if (board[0][j] == 'O')
                DFS(board, 0, j);
            if (board[m - 1][j] == 'O')
                DFS(board, m - 1, j);
        }
        for (auto &bo:board)
            for (auto &b:bo)
                b = (b == 'O' ? 'X' : (b == 'M' ? 'O' : b));
    }

    void DFS(vector<vector<char>> &board, int x, int y) {
        board[x][y] = 'M';
        for (auto &d:dir) {
            int i = x + d[0], j = y + d[1];
            if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] == 'O')
                DFS(board, i, j);
        }
    }
};
```

#### [529 扫雷游戏](https://leetcode-cn.com/problems/minesweeper/submissions/)

先计算出每个位置周围的 8 个位置的炸弹的数量，如果数量大于等于 1，那么标记出来并且结束搜索，如果数量为 0，那么继续向周围 8 个位置搜索。

```c++
class Solution {
    int m, n;
public:
    vector<vector<char>> updateBoard(vector<vector<char>> &board, vector<int> &click) {
        if (board[click[0]][click[1]] == 'M') {
            board[click[0]][click[1]] = 'X';
            return board;
        }
        m = board.size(), n = m ? board[0].size() : 0;
        DFS(board, click[0], click[1]);
        return board;
    }

    void DFS(vector<vector<char>> &board, int x, int y) {
        int b = 0;
        for (int i = x - 1; i <= x + 1; ++i)
            for (int j = y - 1; j <= y + 1; ++j)
                if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] == 'M')
                    ++b;
        if (b != 0) {
            board[x][y] = static_cast<char>(b + '0');
            return;
        }
        board[x][y] = 'B';
        for (int i = x - 1; i <= x + 1; ++i)
            for (int j = y - 1; j <= y + 1; ++j)
                if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] == 'E')
                    DFS(board, i, j);
    }
};
```

#### [473 火柴拼正方形](https://leetcode-cn.com/problems/matchsticks-to-square/submissions/)

因为要求用所有的火柴来拼成正方形，所以先判断所有的火柴组成的是否是 4 的倍数以及是否有数字大于 sum / 4 ，然后将数组从大到小排序，这样可以用贪心的策略减少搜索的次数，否则需要进行回溯，最后对整个数组进行 DFS。

```c++
class Solution {
    int match, n, sum;
    vector<bool> visited;
public:
    bool makesquare(vector<int> &nums) {
        sort(nums.begin(), nums.end(), [](int &a, int &b) { return a > b; });
        sum = accumulate(nums.begin(), nums.end(), 0), n = nums.size(), match = 4;
        visited = vector<bool>(n, false);
        if (n == 0 || sum % 4 != 0)
            return false;
        for (auto &m:nums)
            if (m > sum / 4)
                return false;
        for (int i = 0; i < n; ++i) {
            if (!visited[i] && DFS(nums, i, nums[i])) {
                visited[i] = true;
                --match;
            }
        }
        return match == 0;
    }

    bool DFS(vector<int> &nums, int m, int acc) {
        if (acc > sum / 4)
            return false;
        else if (acc == sum / 4)
            return true;
        for (int i = m + 1; i < n; ++i)
            if (!visited[i] && DFS(nums, i, acc + nums[i])) {
                visited[i] = true;
                return true;
            }
        return false;
    }
};
```

#### [980 不同路径 III](https://leetcode-cn.com/problems/unique-paths-iii/)

用一个变量 zeros 把矩阵中 0 的数量记录下来，每次遍历到 0 即 zeros - 1，直到 zeros == 0 且当前点的四个方向上有终点，那么结果 +1 并返回，继续下一步的 DFS。

```c++
class Solution {
    int m, n, zeros, res;
    vector<vector<bool>> visited;
    int dir[4][2] = {{0,  1},
                     {1,  0},
                     {0,  -1},
                     {-1, 0}};
public:
    int uniquePathsIII(vector<vector<int>> &grid) {
        m = grid.size(), n = m ? grid[0].size() : 0, zeros = m * n - 2, res = 0;
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        int sr, sc, er, ec;
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (grid[i][j] == 1)
                    sr = i, sc = j;
                else if (grid[i][j] == -1)
                    --zeros;
        DFS(grid, sr, sc, 0);
        return res;
    }

    void DFS(vector<vector<int>> &grid, int r, int c, int count) {
        visited[r][c] = true;
        for (auto &d:dir) {
            int i = r + d[0], j = c + d[1];
            if (i >= 0 && i < m && j >= 0 && j < n) {
                if (grid[i][j] == 2 && count == zeros) {
                    ++res;
                    break;
                }
                if (grid[i][j] == 0 && !visited[i][j])
                    DFS(grid, i, j, count + 1);
            }
        }
        visited[r][c] = false;
    }
};
```

#### [37 解数独](https://leetcode-cn.com/problems/sudoku-solver/)

对每个 '.' 格子进行从 '1' 到 '9' 的回溯，判断当前行，列，以及 3 * 3 的格子中是否有相同的值，直到到达矩阵的最后。

```c++
class Solution {
    int m, n;
public:
    void solveSudoku(vector<vector<char>> &board) {
        m = board.size(), n = board[0].size();
        DFS(board, 0, 0);
    }

    bool DFS(vector<vector<char>> &board, int i, int j) {
        if (j >= n)
            return DFS(board, i + 1, 0);
        else if (i >= m)
            return true;
        else if (board[i][j] != '.')
            return DFS(board, i, j + 1);
        for (char c = '1'; c <= '9'; ++c) {
            if (CheckNum(board, i, j, c)) {
                board[i][j] = c;
                if (DFS(board, i, j + 1))
                    return true;
                board[i][j] = '.';
            }
        }
        return false;
    }

    bool CheckNum(vector<vector<char>> &board, const int &i, const int &j, const char &c) {
        for (int k = 0; k < 9; ++k)
            if (board[k][j] == c || board[i][k] == c)
                return false;
        for (int a = 0; a < 3; ++a)
            for (int b = 0; b < 3; ++b)
                if (board[a + i / 3 * 3][b + j / 3 * 3] == c)
                    return false;
        return true;
    }
};
```

#### [79 单词搜索](https://leetcode-cn.com/problems/word-search/)

在矩阵里进行一次 DFS 即可。

```c++
class Solution {
    int m, n;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
    vector<vector<bool>> visited;
public:
    bool exist(vector<vector<char>> &board, string word) {
        m = board.size(), n = m ? board[0].size() : 0;
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (board[i][j] == word[0] && DFS(board, i, j, word.substr(1)))
                    return true;
        return false;
    }

    bool DFS(vector<vector<char>> &board, int i, int j, string word) {
        if (word == "")
            return true;
        visited[i][j] = true;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n && board[a][b] == word[0] && !visited[a][b] &&
                DFS(board, a, b, word.substr(1)))
                return true;
        }
        visited[i][j] = false;
        return false;
    }
};
```

#### [212 单词搜索 II](https://leetcode-cn.com/problems/word-search-ii/)

最简单的方法是对每一个单词在矩阵里进行一次 DFS，这样的话时间复杂度是 O(m * n * k * l)，其中 m 是矩阵的长，n 是矩阵的宽，l是单词的数量，k 是所有单词的最长长度。我们可以为所有单词建立一个字典树，然后再在矩阵里进行一次 DFS，在矩阵的每个点处判断当前的字母是否在字典树的根节点的 next 数组中，如果是的话搜索其周围的字母以及继续遍历字典树，这样做的时间复杂度是 O(m * n * k)。

```c++
class Solution {
    struct TrieNode {
        vector<TrieNode *> next;
        bool end;

        TrieNode() {
            next = vector<TrieNode *>(26, nullptr);
            end = false;
        }
    };

    TrieNode *root;
    int m, n;
    unordered_set<string> res;
    vector<string> ret;
    vector<vector<bool>> visited;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
public:
    vector<string> findWords(vector<vector<char>> &board, vector<string> &words) {
        res = unordered_set<string>();
        m = board.size(), n = board[0].size();
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        BuildTrie(words);
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (root->next[board[i][j] - 'a'])
                    DFS(board, i, j, root->next[board[i][j] - 'a'], string(1, board[i][j]));
        ret = vector<string>(res.begin(), res.end());
        return ret;
    }

    void BuildTrie(vector<string> &words) {
        root = new TrieNode();
        TrieNode *node;
        for (auto &s:words) {
            node = root;
            for (auto c:s) {
                if (!node->next[c - 'a'])
                    node->next[c - 'a'] = new TrieNode();
                node = node->next[c - 'a'];
            }
            node->end = true;
        }
    }

    void DFS(vector<vector<char>> &board, int i, int j, TrieNode *node, string word) {
        if (!node)
            return;
        if (node->end)
            res.insert(word);
        visited[i][j] = true;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n && node->next[board[a][b] - 'a'] && !visited[a][b])
                DFS(board, a, b, node->next[board[a][b] - 'a'], word + board[a][b]);
        }
        visited[i][j] = false;
    }
};
```

#### [749 隔离病毒](https://leetcode-cn.com/problems/contain-virus/)

矩阵会持续地变化，每一轮 DFS 结束后需要进行两个操作，一是将已经隔离的病毒进行标记，二是将未隔离的病毒进行感染（延伸），可以先将所有的未隔离的病毒先保存下来再依次进行延伸，写起来比较复杂。

```c++
class Solution {
    int m, n;
    int dir[4][2] = {{0,  1},
                     {1,  0},
                     {0,  -1},
                     {-1, 0}};
    vector<vector<bool>> visited;
public:
    int containVirus(vector<vector<int>> &grid) {
        m = grid.size(), n = m ? grid[0].size() : 0;
        int res = 0;
        bool exist = true;
        while (exist) {
            exist = false;
            int perimeter = 0, co_x = 0, co_y = 0;
            visited = vector<vector<bool>>(m, vector<bool>(n, false));
            for (int i = 0; i < m; ++i) {
                for (int j = 0; j < n; ++j) {
                    if (grid[i][j] == 1 && !visited[i][j]) {
                        exist = true;
                        int peri = CalcPeri(grid, i, j);
                        if (peri > perimeter) {
                            perimeter = peri;
                            co_x = i, co_y = j;
                        }
                    }
                }
            }
            res += perimeter;
            if (exist) {
                Contain(grid, co_x, co_y);
                Infect(grid);
            }
        }
        return res;
    }

    int CalcPeri(vector<vector<int>> &grid, int i, int j) {
        int peri = 4, res = 0;
        visited[i][j] = true;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n) {
                if (grid[a][b] != 0)
                    --peri;
                if (grid[a][b] == 1 && !visited[a][b])
                    res += CalcPeri(grid, a, b);
            } else
                --peri;
        }
        return res + peri;
    }

    void Contain(vector<vector<int>> &grid, int i, int j) {
        grid[i][j] = 2;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n && grid[a][b] == 1)
                Contain(grid, a, b);
        }
    }

    void Infect(vector<vector<int>> &grid) {
        vector<pair<int, int>> infect;
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (grid[i][j] == 1)
                    infect.push_back(pair<int, int>(i, j));
        for (auto &f:infect) {
            for (auto &d:dir) {
                int a = f.first + d[0], b = f.second + d[1];
                if (a >= 0 && a < m && b >= 0 && b < n && grid[a][b] == 0)
                    grid[a][b] = 1;
            }
        }
    }
};
```

#### [51 N皇后](https://leetcode-cn.com/problems/n-queens/)

很经典的回溯问题，用 DFS 搜索每一种可能直到搜索完最后一行，用当前位置的横纵坐标的和和差分别判断两个对角线上是否有皇后即可。

```c++
class Solution {
public:
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> res;
        string temp = "";
        for (int i = 0; i < n; ++i)
            temp += ".";
        vector<string> board(n, temp);
        unordered_map<int, bool> left_diagonal, right_diagonal;
        vector<bool> row(n, false), col(n, false);
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                left_diagonal[i + j] = false;
                right_diagonal[i - j] = false;
            }
        }
        Backtrack(0, n, board, res, col, left_diagonal, right_diagonal);
        return res;

    }

    void Backtrack(int i, int &n, vector<string> &board, vector<vector<string>> &res,
                   vector<bool> &col, unordered_map<int, bool> &left_diagonal,
                   unordered_map<int, bool> &right_diagonal) {
        if (i == n) {
            res.push_back(board);
            return;
        }
        for (int j = 0; j < n; ++j) {
            if (!col[j] && !left_diagonal[i + j] && !right_diagonal[i - j]) {
                col[j] = true;
                left_diagonal[i + j] = true;
                right_diagonal[i - j] = true;
                board[i][j] = 'Q';
                Backtrack(i + 1, n, board, res, col, left_diagonal, right_diagonal);
                board[i][j] = '.';
                col[j] = false;
                left_diagonal[i + j] = false;
                right_diagonal[i - j] = false;
            }
        }
    }
};
```