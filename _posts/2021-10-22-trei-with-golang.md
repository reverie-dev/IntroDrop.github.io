---
layout:     post
title:      "golang实现trie "
subtitle:   "trie implement with golang"
date:       2021-10-22 15:28:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- leetcode
- data structure
---


## 数据结构Trie(前缀树)的golang版本实现

### 用数组存储child版本
```golang
type Trie struct {
	children [26]*Trie
	isEnd    bool
}

func Constructor() Trie {
	return Trie{}
}

func (t *Trie) Insert(word string) {
	node := t
	for _, c := range word {
		c -= 'a'
		if node.children[c] == nil {
			node.children[c] = &Trie{} // new 一个新的节点
		}
		node = node.children[c] // 当前节点递归到child节点
	}
	node.isEnd = true
}

func (t *Trie) SearchPrefix(prefix string) *Trie {
	node := t
	for _, c := range prefix {
		c -= 'a'
		if node.children[c] == nil {
			return nil
		}
		node = node.children[c]
	}
	return node
}

func (t *Trie) Search(word string) bool {
	node := t.SearchPrefix(word)
	return node != nil && node.isEnd;
}

func (t *Trie) StartsWith(prefix string) bool {
	return t.SearchPrefix(prefix) != nil;
}
```

### 用map存储child版本

```golang

type TrieNode struct {
	val rune  // 当前节点的值
	children map[rune]*TrieNode  // 子节点，用map存起来
	isEnd bool  // 是否为终止节点
}

func Constructor(val rune) *TrieNode {
	return &TrieNode{
		children: make(map[rune]*TrieNode),
		val: val,
		isEnd: false,
	}
}

func (t *TrieNode) Insert(word string)  {
	node := t
	for _, c := range word {
		if node.children[c] == nil {
			node.children[c] = &TrieNode{
				val: c,
				children: make(map[rune]*TrieNode),
				isEnd: false,
			}
		}
		node = node.children[c]
	}
	node.isEnd = true
}


func (t *TrieNode) SearchPrefix(prefix string) *TrieNode {
	node := t
	for _, c := range prefix {
		if node.children[c] == nil {
			return nil
		}
		node = node.children[c]
	}
	return node
}

func (t *TrieNode) Search(word string) bool {
	node := t.SearchPrefix(word)
	return node != nil && node.isEnd;
}

func (t *TrieNode) StartsWith(prefix string) bool {
	return t.SearchPrefix(prefix) != nil;
}

func main() {
	trie := TrieNode{
		val:      ' ',
		children: make(map[rune]*TrieNode),
		isEnd:    false,
	}

	trie.Insert("add")
	trie.Insert("pdd")
	trie.Insert("acd")

	fmt.Println(trie.Search("add"))
	fmt.Println(trie.StartsWith("ad"))
	fmt.Println(trie.Search("asd"))
}

```

### 前缀树相关算法题

#### 702 [词典中最长的单词](https://leetcode-cn.com/problems/longest-word-in-dictionary/)

可以直接用hashset来解决，需要注意的是，满足条件的必须是从一个字母的单词开始。

```go
func longestWord(words []string) string {
	m := make(map[string]bool)
	ans := ""
	sort.Strings(words)
	for _, word := range words {
		l := len(word)
		if len(word) == 1 || m[word[:l-1]] == true {
			if len(word) > len(ans) {
				ans = word
			}
            m[word] = true
		}
	}
	return ans
}
```

使用前缀树的解法:

```go
```

