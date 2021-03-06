---
layout: post
title: "Elasticsearch python 实践（一）"
date: 2019-11-22 14:13:10 +0800
categories: [elasticsearch, python]
---

> 最近在做的一个项目中要使用到Elasticsearch(以下简称ES)，在这儿记录一下相关知识点。

## 介绍

ES是基于Lucene框架、JAVA语言开发的搜索服务器。它提供了一个具备分布式、多用户功能的全文搜索引擎，同时还提供了一系列RESTful web API接口。由于其稳定、可靠、便捷、开源等优点，在目前其已成为了流行的企业级搜索引擎。

其使用场景有：

1. 全文检索
2. 搜索推荐
3. 数据分析，包括：用户行为日志分析、服务器日志分析
4. 数据挖掘
5. 等等

## 安装

> 由于项目需要的关系，本文仅介绍ES python客户端相关知识点。

### MAC 系统

```bash
brew install elasticsearch
```

如果想选择不同版本的ES，使用下列命令查找不同版本的ES。

```bash
brew search elasticsearch
```

安装完毕后，运行ES服务：

```bash
brew services start elasticsearch
```

运行成功后，打开浏览器，在地址栏输入:`127.0.0.1:9200`进行访问，页面正常显示说明ES服务已经可以正常使用。

### 安装ES python驱动

进入python运行环境，使用下列命令进行安装。

```bash
pip install elasticsearch
```

## 倒排索引

ES使用了名为**倒排索引**的结构，这种结构适用于快速全文搜索。倒排索引由文档中所有不重复的词列表所构成，其中对每个词来说，至少有一个文档列表包含它。

### 举例说明

假设一个文档集合中包含四个文档，文档内容下图所示:

![文档列表](/static/img/posts/es_python_shijian_1.png)

则以该文档集合建立的倒排索引如下图所示:

![文档倒排索引](/static/img/posts/es_python_shijian_2.png)

还可以在倒排索引中记录单词在某个文档中出现的位置，如**中国**可表示为：`(1;2;0,5),(2;1;4),(3;1;4)`。

倒排索引建立后，搜索引擎就可以方便的进行查询了。比如查询"中国"时，首先搜索系统会查找倒排索引，从中可以得到包含该词的文档集合，该文档集合就是候选搜索结果。然后利用词频等相关信息计算文档和查询的相似性，按照相似性对这些候选结果进行降序排序。

当然，以上过程仅仅是ES搜索过程的简化版本，仅仅是为了直观地解释倒排索引的运行原理。

## 分词器

从上一节，我们可以知道倒排索引中最基础的一个步骤就是对文本进行分词，以下我们开始介绍ES中分词器。

### 介绍

ES中的分词器，简单的说就是从一串文本中切分出一系列的词，并对每个词条进行标准化。

### 结构

分词器主要包括三个部分：

1. character filter：预处理，过滤掉HTML标签、特殊符号转化等；
2. tokenizer：分词
3. token filter：标准化，合并单复数、大小写等。

### ES内置分词器

1. standard分词器（默认）：该分词器会将词汇统一转化为小写形式，并去除停用词及标点符号，对中文采用单字切分，如，"我是中文"，会被切分为：["我","是", "中", "文"]；
2. simple分词器：该分词器首先会通过非字母字符来切割文本，然后再将词汇统一转化为小写形式，同时该分词器会去除数字类型的字符；
3. whitespace分词器：功能如其名，仅仅是去除空格，对字符也不小写化，也不对生成的词汇进行标准化处理，不支持中文；
4. language分词器：特定语言的分词器，但并不支持中文。

### 中文分词器

由于ES内置的分词器对中文支持度不够，若需处理中文文本，必须自行安装中文分词器。目前大多数项目中使用的中文分词器是：[IK分词器](https://github.com/medcl/elasticsearch-analysis-ik)，安装方式请自定查看其文档。

这里说一下安装IK分词器需要注意的地方：

1. IK插件版本必须与ES版本对应；
2. 若项目面向人群为中文为主，最好修改默认分词器为IK分词器。具体步骤：在ElasticSearch的配置文件config/elasticsearch.yml中的最后一行添加参数 index.analysis.analyzer.default.type: ik;
3. 在每次创建索引时，考虑是否有中文分词需求。若有，设置mapping来使用ik分词器。具体步骤：添加 "analyzer": "ik_max_word"或"analyzer": "ik_smart"到字段中;
4. IK分词器有两种分词模式：ik_max_word,ik_smart，要根据具体情况选择。

验证IK分词器是否安装成功，访问[该地址](http://localhost:9200/_analyze/?analyzer=ik_smart&text=中华人民共和国国歌)即可。

