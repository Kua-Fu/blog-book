# lucene介绍

## 一、参考

> [Apache](https://www.apache.org/)

> [Celebrating 20 years of Apache Lucene](https://www.elastic.co/cn/celebrating-lucene)

> [lucene in action, 2rd](https://book.douban.com/subject/3726306/)

> [Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/foreword_id.html)

> [Practical Apache Lucene 8: Uncover the Search Capabilities of Your Application](https://learning.oreilly.com/library/view/practical-apache-lucene/9781484263457/)

> [Lucene 9.0 file format](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/package-summary.html)

> [存储系统中的算法：LSM 树设计原理](https://mp.weixin.qq.com/s/BsW_SeGdnMHfg1_bM_sgSQ)

> [LSM核心实现讲解](https://mp.weixin.qq.com/s/GsnBpZPzizX9ODuQLT-uAg)

> [RocksDB](https://github.com/facebook/rocksdb/wiki/RocksDB-Overview)

> [Lucene-文件格式](https://zhuanlan.zhihu.com/p/354105864)

> [Using Finite State Transducers in Lucene](https://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html)

> [Elasticsearch 7.3 的 offheap 原理](https://www.easyice.cn/archives/346)

> [Similarity](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/search/similarities/Similarity.html)



## 一、lucene 初识

### 1.1 lucene 是什么？

* lucene 是个 Java 搜索库，[简单易用，功能强大](https://lucene.apache.org/core/)

* lucene 和 Elasticsearch 关系，可以类比为 发动机和汽车关系，lucene提供写入/查询核心功能，ES实现分布式/分析功能。

* lucene 和 google 等搜索公司关系，

	lucene 是开源软件，可以免费使用；google公司的搜索功能需要付费
	
	它们之间的搜索能力不太好比较，lucene经过多年发展，已经成为信息检索领域的开源软件标准
	
* lucene 与关系型数据库的区别

	lucene基于文档索引和搜索，一般用于全文搜索，无法实现事务
	
* 基于lucene，实现搜索功能的公司或者产品有，github/ wiki/ twitter/ linkedin 等等

### 1.2 发展历程

![Doug](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/cloudera-doug-cutting.png?raw=true)


#### （1）  2000年3月，[Doug Cutting](https://en.wikipedia.org/wiki/Doug_Cutting) 第一次发布开源版本 0.01

* 1997年，Doug 因为工作不稳定和 Java 当时非常热门，所以，他想利用 Java 商业化一些自己的软件，但是后来放弃商业化想法，将 lucene 放到 SourceForge 上开源， Doug曾经是 Apache 基金会主席（2011年）
	
* lucene 是 Doug 的第五个搜索引擎，之前在施乐PARC写过两个，一个在苹果，一个在Excite
	
* lucene 是 Doug 老婆中间的名字

#### （2） 2000年10月，发布 1.0

* 2001年9月，lucene 加入 [Apache Jakarta](https://jakarta.apache.org/) 项目

* 2002年，1.2版本成为第一个 Apache 许可证的 lucene 版本
	
* 2004年，[Shay Banon](https://en.wikipedia.org/wiki/Elasticsearch) 发布了Elasticsearch的前身-[compass](http://www.compass-project.org/overview.html)
	
> 许多年前，一个刚结婚的名叫 Shay Banon 的失业开发者，跟着他的妻子去了伦敦，他的妻子在那里学习厨师。 在寻找一个赚钱的工作的时候，为了给他的妻子做一个食谱搜索引擎，他开始使用 Lucene 的一个早期版本。
> 直接使用 Lucene 是很难的，因此 Shay 开始做一个抽象层，Java 开发者使用它可以很简单的给他们的程序添加搜索功能。 他发布了他的第一个开源项目 Compass。
>
> 后来 Shay 获得了一份工作，主要是高性能，分布式环境下的内存数据网格。这个对于高性能，实时，分布式搜索引擎的需求尤为突出， 他决定重写 Compass，把它变为一个独立的服务并取名 Elasticsearch。
>
> 第一个公开版本在2010年2月发布，从此以后，Elasticsearch 已经成为了 Github 上最活跃的项目之一，他拥有超过300名 contributors(目前736名 contributors )。 一家公司已经开始围绕 Elasticsearch 提供商业服务，并开发新的特性，但是，Elasticsearch 将永远开源并对所有人可用。
> 
> 据说，Shay 的妻子还在等着她的食谱搜索引擎…​
	
* 2005年，lucene 成为 Apache 顶级项目

#### （3） 2006年5月，发布 2.0

#### （4） 2009年11月，发布 3.0

* 2010年2月7日，Elasticsearch 0.1.0 发布
	
* 2010年，solr 作为lucene子项目，合并入lucene
	
* 2011年，twitter 使用 lucene 实现实时搜索

### （5） 2012年，发布 4.0

* 是一个"革命性"版本，让lucene 进入严肃分析领域，(1) 添加编码/解码器; (2) 新增相似性分析模型(BM25, DFR)
	
* 2012年2月9日，Elastic公司成立
	
### （6） 2015年，发布 5.0

### （7） 2016年，发布 6.0

* 添加了对多维索引的支持 BKD树
	
* 将BM25 设置为默认相似性模型，之前是 TF/IDF
	
* lucene 源码从 Subversion 转移到 git
	
### （8） 2017年，发布 7.0

### （9） 2019年，发布 8.0

### （10） 2021年，发布 9.0

* solr 独立出 lucene，成为Apache 顶级项目
	
![elastic](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/shayBanon.png?raw=true)
	
## 二、写入（索引）


### 2.1 倒排索引 Inverted index

![index](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/doc-index.png?raw=true)

⚠️ MySQL 8.0 也实现了倒排索引 （ 2018/4/19 ）



### 2.2 索引流程

* 一般先把不同格式的文本，先转变为特定格式文本(json)

* 每个文档要确定field结构，每个field都需要经过分析器分析后，最终持久化

* 分析器分析过程，会流式经过不同类型的分析器，产生token列表

![i1](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/index-1.png?raw=true)

![i2](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/index-2.png?raw=true)

![i3](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/index-3.png?raw=true)

![i6](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/index-6.png?raw=true)

| 术语| 描述|
|---|---|
| document | 文档|
| field | 文档中的字段|
| index | 名词，lucene中文档保存的集合|
| segment | 段，表示写入时候的commit点|
| term | lucene中存储和搜索的基本单元|
| token | 经过分词器后，切分的最小单元| 

term 可以理解为 token 加上 fieldName，例如： 文档中title字段，值为 hello world, 分析器分析后，生成 hello 和 world两个token，则 term也是两个, <title, hello>, <title, world>


### 2.3 LSM树实现 


[LSM tree (log-structured merge-tree) ](https://en.wikipedia.org/wiki/Log-structured_merge-tree)

[B-tree](https://en.wikipedia.org/wiki/B-tree)

* 是一种对频繁写操作非常友好的数据结构，同时兼顾了查询效率。

* 之所以有效是基于以下事实：磁盘或内存的连续读写性能远高于随机读写性能(局部性原理)

* 是许多 key-value 型或日志型数据库所依赖的核心数据结构，例如 BigTable、HBase、Cassandra、LevelDB、SQLite、Scylla、RocksDB 等。


LSM树 三个主要组成部分: memtable，log，SSTable

* memtable

	是红黑树或者跳表这样的有序内存数据结构，起到缓存和排序的作用，把新写入的数据按照键的大小进行排序。当memtable到达一定大小之后，会被转化成SSTable格式刷入磁盘持久化存储

* SSTable（Sorted String Table）

	说白了就是一个特殊格式的文件，其中的数据按照键的大小排列，你可以把它类比成一个有序数组。而 LSM 树，说白了就是若干SSTable的集合。

* log

	文件记录操作日志，在数据写入memtable的同时也会刷盘写入到log文件，作用是数据恢复。比如在memtable中的数据还没转化成SSTable持久化到磁盘时，如果突然断电，那么memtable里面的数据都会丢失，但有log文件在，就可以恢复这些数据。当然，等memtable中的数据成功转化成SSTable落盘之后，log文件中对应的操作日志就没必要存在了，可以被删除

![RocksDB](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/lsm-0.png?raw=true)

![lsm](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/lsm-5.png?raw=true)

[持久化变更](https://www.elastic.co/guide/cn/elasticsearch/guide/current/translog.html)

### 2.4 索引文件

下面是[lucene9.3.0](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/package-summary.html) 中的索引文件 

![file](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/lucene-file-format.png?raw=true)

| 序号| 文件后缀  | 描述说明| 示例|
|---| ---|---|---|
|  | [.cfs](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90CompoundFormat.html) | An optional "virtual" file consisting of all the other index files for systems that frequently run out of file handles.| |
|  | [.cfe](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90CompoundFormat.html) | The "virtual" compound file's entry table holding all entries in the corresponding .cfs file.| | 
|  | [.doc](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PostingsFormat.html) | Frequencies and Skip Data| |
|  | [.dvd](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90DocValuesFormat.html) | DocValues data | |
|  | [.dvm](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90DocValuesFormat.html) | DocValues metadata | |
|  | [.fdt](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90StoredFieldsFormat.html) | This file stores a compact representation of documents in compressed blocks of 16KB or more. When writing a segment, documents are appended to an in-memory byte[] buffer. When its size reaches 16KB or more, some metadata about the documents is flushed to disk, immediately followed by a compressed representation of the buffer using the LZ4 compression format.| | 
|  | [.fdx](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90StoredFieldsFormat.html) | This file stores two monotonic arrays, one for the first doc IDs of each block of compressed documents, and another one for the corresponding offsets on disk. At search time, the array containing doc IDs is binary-searched in order to find the block that contains the expected doc ID, and the associated offset on disk is retrieved from the second array.| | 
|  | [.fdm](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90StoredFieldsFormat.html) | This file stores metadata about the monotonic arrays stored in the index file.| | 
|  | [.fnm](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90FieldInfosFormat.html) | Field names are stored in the field info file| | 
|  | [.kdm](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PointsFormat.html) | A .kdm file that records metadata about the fields, such as numbers of dimensions or numbers of bytes per dimension.| | 
|  | [.kdi](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PointsFormat.html) | A .kdi file that stores inner nodes of the tree.| |
|  | [.kdd](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PointsFormat.html) | A .kdd file that stores leaf nodes, where most of the data lives.| |
|  | [.liv](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90LiveDocsFormat.html) | The .liv file is optional, and only exists when a segment contains deletions.| |
|  | [.nvd](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90NormsFormat.html) | Norms data| |
|  | [.nvm](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90NormsFormat.html) | Norms metadata| |
|  | [.pay](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PostingsFormat.html) | Payloads and Offsets|
|  | [.pos](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PostingsFormat.html) | Positions| | 
|  | [.si](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90SegmentInfoFormat.html)  | Header, SegVersion, SegSize, IsCompoundFile, Diagnostics, Files, Attributes, IndexSort, Footer| |
|  | [.tim](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PostingsFormat.html) | Term Dictionary| | 
|  | [.tip](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90PostingsFormat.html) | Term Index | |
|  | [.tvm](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90TermVectorsFormat.html)| A vector metadata file | | 
|  | [.tvd](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90TermVectorsFormat.html) | This file stores terms, frequencies, positions, offsets and payloads for every document. Upon writing a new segment, it accumulates data into memory until the buffer used to store terms and payloads grows beyond 4KB. Then it flushes all metadata, terms and positions to disk using LZ4 compression for terms and payloads and blocks of packed ints for positions. | | 
|  | [.tvx](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/lucene90/Lucene90TermVectorsFormat.html) | An index file | |


⚠️ [lucene3.0.3](https://lucene.apache.org/core/3_0_3/fileformats.html) 中的文件格式


|Name|	Extension|	Brief Description|
| ---| ---| ---|
| Segments File|	segments.gen, segments_N|	Stores information about segments|
|Lock File|	write.lock|	The Write lock prevents multiple IndexWriters from writing to the same file.|
|Compound File|	.cfs|	An optional "virtual" file consisting of all the other index files for systems that frequently run out of file handles.|
|Fields|	.fnm|	Stores information about the fields|
|Field Index|	.fdx|	Contains pointers to field data|
|Field Data|	.fdt|	The stored fields for documents|
|Term Infos|	.tis|	Part of the term dictionary, stores term info|
|Term Info Index|	.tii|	The index into the Term Infos file|
|Frequencies|	.frq|	Contains the list of docs which contain each term along with frequency|
|Positions|	.prx|	Stores position information about where a term occurs in the index|
|Norms|	.nrm|	Encodes length and boost factors for docs and fields|
|Term Vector Index|	.tvx|	Stores offset into the document data file|
|Term Vector Documents|	.tvd|	Contains information about each document that has term vectors|
|Term Vector Fields| .tvf|	The field level info about term vectors|
|Deleted Documents|	.del|	Info about what files are deleted|
 
![file-format](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/file-format.png?raw=true)


客户实际示例

|文件 | 大小|说明|
| --- | ----| ---|
| wksp_975d8bf6d6204f9ca72efb65eaee718c_tracing-000033| 303.8gb| 索引大小|
| fdt | 154.2gb | 保存每个文档的属性值对应关系|
| tim | 57.8gb |  term 词典| 
| kdd | 32.1gb |  [多维数据结构](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=173081898)，叶节点信息|
| dvd | 20.7gb | 列式存储原始字段| 
| doc | 8.6gb | term出现的频率| 
| tip | 947.7mb | term词典的索引，实际上即是下章介绍的fst数据结构| 


### 2.5 fst数据结构

[Using Finite State Transducers in Lucene](https://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html)

[关于Lucene的词典FST深入剖析](https://www.shenyanchao.cn/blog/2018/12/04/lucene-fst/)



## 三、查询

### 3.1 查询流程

* lucene查询语句，先经过解析器，然后再经过同样的分析器，最后查询

![s1](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/search-1.png?raw=true)

![s2](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/search-2.png?raw=true)

![s3](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic-101/lucene/search-3.png?raw=true)

### 3.2 相似度算法


(1)  TF/IDF

[相关度评分背后的理论](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scoring-theory.html)


相关性判断三个元素

a. 检索词频率

检索词在该字段出现的频率？出现频率越高，相关性也越高。 字段中出现过 5 次要比只出现过 1 次的相关性高。

b. 反向文档频率

每个检索词在索引中出现的频率？频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。
	
c. 字段长度准则

字段的长度是多少？长度越长，相关性越低。 检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大。
	
具体的计算公式 : [TFIDFSimilarity](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/search/similarities/TFIDFSimilarity.html)

(2) BM25 

[可插拔的相似度算法](https://www.elastic.co/guide/cn/elasticsearch/guide/current/pluggable-similarites.html)

具体实现方式: [BM25Similarity](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/search/similarities/BM25Similarity.html)


## 四、更多内容

### 4.1 lucene中的编码/解码如何实现的?

[Codec](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/codecs/package-summary.html) 

Codecs API: API for customization of the encoding and structure of the index.
	
[使用aliyun-codec插件](https://help.aliyun.com/document_detail/363036.html)

### 4.2 lucene中的存储类型? 

NioFs（ 注：非阻塞文件系统）和 MMapFs （ 注：内存映射文件系统）零拷贝技术？

[FSDirectory](https://lucene.apache.org/core/9_3_0/core/org/apache/lucene/store/FSDirectory.html)

### 4.3 force merge过程, lucene中的事务

[Transactional Lucene](https://blog.mikemccandless.com/2012/03/transactional-lucene.html)
	

### 4.4 源码分析

### 4.5 分布式架构 / 聚合分析 / 机器学习

### 4.6 社区贡献 （实际使用 / 开发新功能 等）

