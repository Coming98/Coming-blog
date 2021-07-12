---
title: 分词器Tokenizer使用方法与实践
date: 2021-07-07 16:26:56
summary: Tokenizer 工具类方法介绍以及小Demo
categories: NLP
tags:
  - keras
  - tensorflow
---
# Quick Start

该类允许使用两种方法向量化一个文本语料库：

1. 将每个文本转化为一个整数序列（每个整数都是词典中标记的索引）
2. 将每个文本转化为一个向量，其中每个标记的系数可以是二进制值、词频、TF-IDF权重等。

Tips：词索引分配从 1 开始

Tips：基于词频保留 `num_words - 1` 个词，其余词将会像过滤字符过滤掉

```python
keras.preprocessing.text.Tokenizer(
    num_words=None, # 处理最大单词数目(num_words-1)，基于词频筛选
    filters="!"#$%&()*+,-./:;<=>?@[\]^_`{|}~ ", # 过滤的元素，字符级匹配过滤
    lower=True, # 是否将文本转为小写
    split=" ", # 词分隔符
    char_level=False # 如果为 True，则每个字符都将被视为标记。
)
```

# Attributes

数据格式如：语料库 - `texts` = `[ document1:"AAAA.BBBB.CCCC.", document2, ...]`

```python
document_count
word_counts # 词在语料库中的频数
word_docs # 词在 documents 中出现的频数，用于 TF-IDF 算法
"""
for w in seq:
	if w in self.word_counts:
		self.word_counts[w] += 1
    else:
		self.word_counts[w] = 1
for w in set(seq):
	# In how many documents each word occurs
	self.word_docs[w] += 1
"""
index_docs # 同 word_docs 只不过将词映射为了索引值
index_word # 索引 To 词的映射词典
word_index # 词 To 索引的映射词典
```

# Methods

总体来说就是完成了语料库中 Token 与 index 的映射支持，映射以词频为基础

```python
fit_on_sequences(sequences) # 列表，内部元素为单词索引
fit_on_texts(texts) # 列表，内部元素为分词后的单个 token

sequences_to_matrix(sequences, mode='binary') # 将各个语料的元素索引信息转为 numpy 矩阵 - 通常用于构建数据集
sequences_to_texts(sequences) # list of word indices(int) TO list of tokens(string)
texts_to_sequences(texts)

get_config() # 字典，返回 tokenizer 实例的配置信息

to_json # json，返回 tokenizer 实例的配置信息
```

1. 通过 `fit_on_texts` 完成 Tokenizer 的拟合，更新 Attributes `document_count, word_counts, word_docs, index_docs, index_word, word_index`
2. 通过内置方法或属性名获取相关信息

# Demo

语料库数据如下：

```python
texts = ['United were league champions last season.',
        "They're in a different league from us.",
        "the League of Nations",
        "a meeting of the Women's League for Peace",
        "Her success has taken her out of my league"]
```

测试如下

```python
def test_default(title, tokenizer):

    fit_req = tokenizer.fit_on_texts(texts) # return None

    # 文本未分词先进行分词，然后将词转为索引值
    seq = tokenizer.texts_to_sequences(texts)
    seq_arr = tokenizer.sequences_to_matrix(seq)
    """
    seq : 可以看出 1 的位置是词频最高的 league
    [[6, 7, 1, 8, 9, 10],
    [11, 12, 3, 13, 1, 14, 15],
    [4, 1, 2, 16],
    [3, 17, 2, 4, 18, 1, 19, 20],
    [5, 21, 22, 23, 5, 24, 2, 25, 1]]
    seq_arr: 过长，请看下方示例
	"""

def test_limit_numWords(title, tokenizer):

    tokenizer.fit_on_texts(texts)
    seq = tokenizer.texts_to_sequences(texts)
	seq_arr = tokenizer.sequences_to_matrix(seq)
    texts_from_seq = tokenizer.sequences_to_texts(seq)

    """
    seq : 词频低的词直接舍去，知保留词频较高的前 num_words - 1 个词
	[[6, 7, 1, 8, 9], 
	[3, 1], 
	[4, 1, 2], 
	[3, 2, 4, 1], 
	[5, 5, 2, 1]]
    seq_arr : 索引 0 保留，因此词典数目为 seq_arr.shape[1]
    [[0. 1. 0. 0. 0. 0. 1. 1. 1. 1.]
     [0. 1. 0. 1. 0. 0. 0. 0. 0. 0.]
     [0. 1. 1. 0. 1. 0. 0. 0. 0. 0.]
     [0. 1. 1. 1. 1. 0. 0. 0. 0. 0.]
     [0. 1. 1. 0. 0. 1. 0. 0. 0. 0.]]
     texts_from_seq :
 	['united were league champions last',
 	'a league',
 	'the league of',
 	'a of the league',
 	'her her of league']
    """

"""Attributes
'document_count': 5

'word_counts': '{"united": 1, "were": 1, ... "her": 2, "my": 1}'

'word_docs': '{"united": 1, ... , "her": 1, "my": 1}'

'index_docs': '{"6": 1, "8": 1, ... "25": 1}'

'index_word': '{"1": "league", "2": "of", "3": "a", ... "24": "out", "25": "my"}'

'word_index': '{"league": 1, "of": 2, "a": 3, ... "out": 24, "my": 25}'
"""

def main():

    tokenizer_default = Tokenizer()
    tokenizer_limit_numWords = Tokenizer(num_words=10) # 只考虑词频前9的词

    test_default("default test", tokenizer_default)
    test_limit_numWords("limit num_words test", tokenizer_limit_numWords)

if __name__ == '__main__':
    main()
```

---

如果存在错误，欢迎留言指出哦~

