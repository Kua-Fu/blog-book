# hyperloglog

## 一、参考

> [HyperLogLog: A Simple but Powerful Algorithm for Data Scientists](https://towardsdatascience.com/hyperloglog-a-simple-but-powerful-algorithm-for-data-scientists-aed50fe47869)

> [Philippe Flajolet](https://en.wikipedia.org/wiki/Philippe_Flajolet)

> [Pro ba bit istic Cou nting Algorithms for Data Base Applications ](https://github.com/Kua-Fu/blog-book-images/blob/main/paper/FlMa85.pdf)

> [Loglog Counting of Large Cardinalities (Extended Abstract)](https://github.com/Kua-Fu/blog-book-images/blob/main/paper/DuFl03-LNCS.pdf)

> [HyperLogLog: the analysis of a near-optimal cardinality estimation algorithm](https://github.com/Kua-Fu/blog-book-images/blob/main/paper/FlFuGaMe07.pdf)


## 二、论文

Probabitistic Counting Algorithms for Data Base Applications

数据库应用的概率计数算法

This paper introduces a class of probabilistic counting algorithms with which one can estimate the number of distinct elements in a large collection of data (typically a large file stored on disk) in a single pass using only a small additional storage (typically less than a hundred binary words) and only a few operations per element scanned. 

The algorithms are based on statistical observations made on bits of hashed values of records. They are by construction totally insensitive to the replicative structure of elements in the file; they can be used in the context of distributed systems without any degradation of performances and prove especially useful in the context of data bases query optirnisation.

本文介绍了一类概率计数算法，通过该算法，可以在一次扫描中，仅仅使用少量的额外存储（通常少于100个二进制字），通过对每一个扫描元素进行少量的操作，就可以估算大量数据中不同元素的数量（数据通常是保存在磁盘上的大文件）

该算法，基于对记录散列值的比特进行的统计观察。通过构造，它们对文件中元素的复制结构完全不敏感，它们可以在分布式系统的环境中使用，而不会降低性能，并且在数据库查询优化的环境中特别有效。

### 2.1 INTRODUCTION

介绍

As data base systems allow the user to specify more and more complex queries, the need arises for efficient processing methods. A complex query can however generally be evaluated in a number of different manners, and the overall performance of a data base system depends rather crucially on the selection of appropriate decomposition strategies in each particular case.

因为数据库系统允许用户指定越来越复杂的查询，因此，需要更加高效的处理方法。然而，复杂的查询通常可以以多种不同的方式评估，数据库系统的总体性能，关键取决于在每个特定情况下，选择适当的分散策略。

Even a problem as trivial as computing the intersection of two collections of data A and B lends itself to a number of different treatments: 

1. Sort A, search each element of B in A and retain it if it appears in A;

1. sort A, sort B, then perform a merge-like operation to determine the intersection;

1. eliminate duplicates in A and/or B using hashing or hash filters, then perform Algorithm 1 or 2

即使像是计算两个数据集合 A、B的交集这样一个简单的问题，也有许多不同的方法

1. 先把 A 排序，遍历 B 中的每一个元素，判断是否存在于A中，如果存在，则保留该元素;

1. 排序A，排序B，然后执行一个类似于 merge 的操作，计算出交集;

1. 先利用hash算法，将A和B集合去重，然后，在使用1或者2中的方法计算交集

Each of these evaluation strategy will have a cost essentially determined by the number of records a, b in A and B, and the number of distinct elements α, β in A and B, and for typical sorting methods, the costs are:


for strategy 1: \\( O(a \cdot log\alpha + b \cdot log\beta) \\)

for strategy 2: \\( O(a \cdot log a + b \cdot log b + a + b) \\) 

In a number of similar situations, it appears thus that, apart from the sizes of the files on which one operates (i.e., the number of records), a major determinant of efficiency is the cardinalities of the underlying sets, i.e., the number of distinct elements they comprise.

The situation gets much more complex when operations like projections, selections, multiple joins in combination with various boolean operations appear in queries. As an example, the relational system system R has a sophisticated query optimiser. In order to perform its task, that programme keeps several statistics on relations of the data base.The most important ones are the sizesof relations as well as the number of different elements of some key fields. The choices are made in order to minimise a cer- tain cost function that depends on specific CPU and disk access costs as well as sizes and cardinalities of relations or fields. In system R, this information is periodically recomputed and kept in catalogues that are companions to the data base records and indexes.


上面每一种计算方法的性能，基本上取决于集合A、B中的元素a, b 的数量，和集合A、B中的不相同元素α, β的数量，对于典型的排序方法，复杂度如下

因此，在很多类似场景下，似乎除了操作文件的大小（即记录数量）之外，效率的主要决定因素是——基础集合的基数，即集合中包含的不同元素的个数。

当查询中出现了取子集，取具体列，或者多重join操作，以及布尔运算，查询情况将变得更加复杂。例如，关系型系统 R 有一个复杂的查询优化器。为了执行该任务，该方案保留了关于数据库关系的若干统计数据。最重要的是，关系的大小和一些关键字段的不同元素的数量。这些信息，用于确定在任何给定时间属性的选择，以便决定键的选择，以及计算关系运算符时候，选择适当的算法。这些选择是为了最小化特定的成本函数，该函数取决于特定的CPU和磁盘访问成本，已经关系或字段的大小和基数。在系统R中，该信息会定期更新，并且保存在数据库记录和索引的目录中。


In this paper, we propose efficient algorithms to estimate the cardinalities of multisets of data as are commonly encountered in data base practice. A trivial method consists in determining card(M) by building a list of all elements of M without replication; this method has the advantage of being exact but it has a cost in number of disk accessesand auxiliary storage (at least \\( O(a) \\) or \\( O(a \cdot log a \\) ) if sorting is used) that might be higher than the possible gains which one can obtain using that information.

在本文中，我们提出了一个有效的算法来估计数据库实践中常见的多个数据集的基数。一个简单的方法是，通过创建一个没有重复值的集合M 来计算 \\(card(M) \\) ; 该方法的优点是计算值准确，但其磁盘访问和额外存储（如果使用排序，则至少是 \\( O(a) \\) 或者 \\( a \cdot log(a) \\)) 的成本可能高于使用该去重值 可以获得的收益。


The method we propose here is probabilistic in nature since its result depends on the particular hashing function used and on the particular data on which it operates. It uses minimal extra storage in core and provides practically useful estimates on cardinalities of large collections of data. The accuracy is inversely related to the storage: using 64 binary words of typically 32 bits, we attain a typical accuracy of 10%; using 256 words, the accuracy improves to about 5%. The performances do not degrade as files get large: with 32 bit words, one can safely count cardinalities well over 100 million. Furthermore, by design, our algorithms are totally insensitive to the replication structures of files: as opposed to sampling techniques,  the result will be the same whether elements appear a million times or just a few times.


我们在这里提出的方法，本质上是概率性的，因为它的结果取决于所使用的特定哈希函数和它所操作的特定数据。它的设计核心是使用最少的额外存储，并对大量数据集合的基数提供了实际有用的估计。准确度和额外存储用量成反比，如果使用64个二进制字（每一个都占用32位bit），我们可以获得10%的准确性。如果提高到256个二进制字，准确度相应提高位 5%，当文件变大时候，性能不会降低。使用32个二进制字，用户可以安全的计算超过1亿基数的集合。这个方法的唯一假设是，我们可以用适当方法对记录进行散列，实现伪统一性。然而，这不是一个严重的限制，因为对大型工业文件的实证研究表明，标准哈希技术的谨慎实施确实实现了哈希值的实际一致性。此外，根据设计，我们的算法对于重复的文件结构，不敏感，与采样技术相反：无论元素出现了一百万次，还是出现了几次，基数计算结果都是一样的。

From a more theoretical standpoint, these techniques constitute yet another illustration of the gains that may be achieved in many situations through the use of probabilistic methods. We mention here Morris’ approximate counting algorithm  which maintains approximate counters with an expected constant relative accuracy using only

\\( log2 \cdot log_2^n + O(1) \\)

bits in order to count up to n. Morris’ algorithm may be used to reduce by a factor of 2 the memory size necessary to store large statistics on a large number of events in computer systems.

从更加理论的角度看，这些技术构成了通过使用概率方法在许多情况下可能实现收益的另一个例证。这里我们提到了莫里斯的近似计数算法，该算法保持近似计数通过一个可以预测的相对精度. 通过  \\( log2 \cdot log_2^n + O(1) \\) 个bit，可以估算基数为 n. 莫里斯算法，可以将存储在计算机中的大量事件的大量统计数据，所需要的存储器大小减少2倍。






