---
title: "LeetCode 字典树"
date: 2019-07-24T19:12:25+10:00
draft: true
categories: ["LeetCode"]
---

# [LeetCode Trie](https://leetcode-cn.com/tag/trie/)

Trie前缀树是一种树形数据结构，用于检索字符串数据集中的键，常用于自然语言处理。Trie对比于Hash的优势在于，Hash冲突会随着Hash的增加而增加，其搜索时间复杂度在最坏情况下可能为O(n)，其中n是key的数量，而trie的时间复杂度仅为O(m)，其中m是key的最大长度，并且Trie可以使用比Hash更少的空间。而在平衡树中搜索的时间复杂度为O(mlogn)。

## 题目

### 1. 

#### [208 实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)

一个最基础的 Trie 包含插入和查询（包括整个字符串和包含字符串作为前缀）两个功能，每个 Trie 的节点包含 26 个子节点分别表示 26 个字符，以及一个 bool 变量标示其是否是一个字符串的结尾。

```c++
class Trie {
    struct TrieNode {
        TrieNode *next[26];
        bool is_end;

        TrieNode() : is_end(false) {
            for (int i = 0; i < 26; ++i)
                next[i] = nullptr;
        }
    };

    TrieNode *root;
public:
    /** Initialize your data structure here. */
    Trie() {
        root = new TrieNode();
    }

    /** Inserts a word into the trie. */
    void insert(string word) {
        TrieNode *node = root;
        for (auto c:word) {
            if (!node->next[c - 'a'])
                node->next[c - 'a'] = new TrieNode();
            node = node->next[c - 'a'];
        }
        node->is_end = true;
    }

    /** Returns if the word is in the trie. */
    bool search(string word) {
        TrieNode *node = root;
        for (auto c:word) {
            if (!node->next[c - 'a'])
                return false;
            node = node->next[c - 'a'];
        }
        if (node->is_end)
            return true;
        return false;
    }

    /** Returns if there is any word in the trie that starts with the given prefix. */
    bool startsWith(string prefix) {
        TrieNode *node = root;
        for (auto c:prefix) {
            if (!node->next[c - 'a'])
                return false;
            node = node->next[c - 'a'];
        }
        return true;
    }
};
```