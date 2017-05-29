---
title: 倒排索引（Inverted-Index）
date: 2017-03-13 21:50:06
tags:
description: 倒排索引（Inverted index）,也常被称为反向索引、置入档案或反向档案，被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射。它是文档检索系统中最常见的数据结构。
---
# 倒排索引（Inverted Index）
## 引言
***
**倒排索引（Inverted index）**,也常被称为**反向索引**、**置入档案**或**反向档案**，被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射。它是文档检索系统中最常见的数据结构。
有两种不同的反向索引形式：
* **文档水平**（document-level）反向索引（或者反向档案索引）包含每个引用单词的文档的列表。
* **单词水平**（word-level）反向索引（或者完全反向索引）又包含每个单词在一个文档中的位置。
后者的形式提供了更多的兼容性（比如短语搜索），但是需要更多的时间和空间来创建。
***
## 主要内容
1. 倒排索引（Inverted Index）结构
* 倒排索引由**字典文件（dictionary）**和**记录文件（postings file）**两部分组成。
* **字典（dictionary）**主要是由term,termID，包含该term的文档数目$f_t$，以及指向该记录表（postings list）的指针组成。
$ dictionary:< term,termID,f_t,pointer > $ $ \longrightarrow $ $ [postings list] $
* **记录（posting）**主要是记录term所在的文档（docID），term在文档中出现的次数，以及在文档中的位置，文档长度等信息。
$ posting:< docID,f_d,< position1,\cdots,positionf_d>> $

下图是简单的倒排索引结构例子：
![iis](/Inverted-Index/iis.png)
***
## 倒排文件（Inverted Files）
**定义：**一个倒排文件是针对下标一个文本集的一种面向单词的机制，来加速查询任务。
**倒排文件结构：**
* **字典（Vocabulary):**在一个文本中全部不同的单词的集合
* **事件（Occurrences):**列表，包含了字典每个单词所有必要的信息（文本位置，词频，单词出现的文件，等等）

---
### 倒排文件的布局（layout）
![layout](/Inverted-Index/layout.png)
#### 举个栗子
* Text:
![text](/Inverted-Index/text.png)
* Inverted File

| Vocabulary | Occurrences |
|:----|:----|
| beautiful | 70 |
| flowers   | 45,58 |
| garden    | 18,29 |
| house     | 6 |

### 块寻址（block addressing）
* 文本被划分成区块
* 事件指向单词出现的区块
* 好处：指针数量比位置更少；一个单词在一个单独区块的全部事件被收缩到一个引用
* 缺点：如果确切的位置被需求时，内连查询超过受限区块
#### 举个栗子
* Text:
![text2](/Inverted-Index/text2.png)
* Inverted File

| Vocabulary | Occurrences|
|:----------|:-----------|
| beautiful | 4 |
| flowers | 3 |
| garden | 2 |
| house | 1 |

### 查询(Searching)
在一个倒排列表上的查询算法遵循三步：
1. 字典查询：在查询中出现的单词在字典中定位
2. 检索事件：检索全部被发现的查询词出现的列表
3. 事件操作：事件被处理来解决查询

---
* 查询倒排文件以一个字典开始
* * 在一个独立的文件中存储字典
* 用来存储字典的结构
* * 哈希（Hashing）：$O(1)$lookup，不支持范围查询
* * Tries:$O(c)$lookup，$c = length(word)$
* * B-trees:$O(\log v)$lookup
* 另一个方式是以字典顺序简单地存储单词

### 构建（Construction）
#### 字典构建
#### 倒排文件构建
#### 更快更大的索引构建