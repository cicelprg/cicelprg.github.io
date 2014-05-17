---
layout: post
title: LRU缓存改进算法LIRS
category: Algorithm
tags: Algorithm
---
####LRU算法
在cache替换中最常用的算法,当在缓存快满时而需要缓存新的数据时就需要找到一块最近没有使用过的缓存块去替换它.LRU算法的两个问题:  

 (1). 顺序扫描:顺序扫描的情况下LRU没有命中情况.而且会淘汰其它将要被访问的item从而污染cache  

 (2). 循环的数据集大于缓存大小.如果循环访问且数据集大于缓存大小,那么没有命中情况.例如有5个缓存块,六个数据集,扫描第一遍时1-5号缓存块都存放的是1-5好数据,到访问第六号数据时,淘汰1号缓存快.这时我们再访问1号数据块时,缓存中并没有1号块数据,这是又要淘汰2好缓存块,依次类推,这是缓存的命中率为0

####Memcached 服务中LRU算法

当Memcached在申请内存失败时就开始使用LRU淘汰数据,淘汰规则是从数据项`Item list`尾部开始遍历,在列表中找到一个应用计数器为0的Item,把Item释放掉.若果没有找到`refcount`为0的item,那么再次遍历`Item List`找到在3小时内没有访问过的Item然后再释放掉.如果还是找不到那么就申请内存失败.这里为什么要从List的尾部开始遍历呢?因为Memcached会把刚刚访问过的数据放到List的头部,所以尾部就是没有访问过的.

解析Memcached缓存命中率的问题[link](http://www.iteye.com/topic/225692)

####缓存替换改进算法LIRS

Reuse:一块使用后再次被使用 Recency:两次访问的距离

LIRS(Low Inter-reference Recency Set)算法将数据块分为两种LIR(Low Inner-reference Recency)[热点数据,两次访问的距离比较短]和HIR(High Inner-reference Recency)[冷数据,两次访问的距离比较长].  

LIRS可以看成是一种分级思想：第一级是HIR,第二级是LIR,数据先进入到第一级,当数据在较短的时间内被访问两次时成为热点数据则进入LIR,HIR和LIR内部都采用LRU策略. 这样,LIR中的数据比较稳定,解决了LRU的上述两个问题.  

LIRS论文中提出了一种实现方式,不过我们可以做一些变化,如可以实现两级cache,cache元素先进入第一级cache,当访问频率达到一定值（比如2）时升级到第二级,第一级和第二级均内部采用LRU进行替换。Oracle Buffer Cache中的Touch Count算法也是采用了类似的思想.

<strong>LIRS的实现:</strong>  

两个栈,一个大的LRU Queue存放热数据,另外一个小的LRU Queue存放冷数据.【在赵晓东教授中为stack我觉得为Queue更加便于我理解】

下面对几种访问情况加以分析:

首先有3种数据: 1. resident in cache 2.LIR BLOCK 3.HIR BLOCK 
>1. 当命中一块LIR BLOCK时,将这块BLOCK移动到LIR队列的尾部,并且保证LIR队列的首部元素为LIR BLOCK.(可能要移动的BLOCK恰好在队列首部)

>2. 当命中一块HIR resident block(可能在HIR中,也可能在LIR中,也可能两个中都有),将这个BLOCK加入到LIR队列中,查看LIR队列中是否已经存在这块BLOCK,如果存在那么就把该BLOCK标记为LIR,然后再把LIR首部的BLOCK标记为HIR BLOCK然后转入HIR队列,并保证LIR队列首部BLOCK为LIR块,移除HIR队列.如果不存在那么保留其HIR身份,并移到HIR队列的尾部.

>3. 当没有命中HIR和LIR队列中的BLOCK时(Miss Cache),那么将该BLOCK加入到LIR队列和HIR队列的尾部

<strong>LIRS 潜在条件</strong>
>1. LIR BLOCK都是resident的  
>2. HIR队列中的所有BLOCK都是resident的
>3. LIR队列首部必须是LIR BLOCK 
>4. LIR BLOCK 不会出现在HIR队列中
>5. 所有BLOCK 初始为HIR,但是LIR队列中未满,新访问的都已LIR状态入队
>6. HIR BLOCK 不能出现在LIR首部,如果是要直接移除

<strong>参考文章</strong>  
[文章一](http://www.longyusoft.com/detail.php?id=426) 
[文章二](http://blog.csdn.net/it_yuan/article/details/8488876)
[张晓东教授PDF](/resource/pdf/2010-10-19-11_27_12.pdf)





