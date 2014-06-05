---
layout: post
title: Zend 内存管理
category: PHP
tags: PHP
---
<h3>Zend 内存管理 </h3>
<h4>前言</h4>
使用memory_get_usage()测试脚本当前使用的内存的大小,这里在定义一个变量不初始化时Zend是不会给它分配对应的内存的,例如`<?php $a ?>`这里在当前符号表也是找不到$a这个变量的.

内存管理一般分为下面几步:  
1. 是否有足够的内存供程序使用  
2. 如何从足够可用的内存中获取部分内存  
3. 对于使用后的内存,是否可以将其销毁并将其重新分配给其他的程序使用  

Zend 内存管理分为下面几层:  
存储层(storage)【四种分配方案:malloc,win32,mmap_annon,mmap_zero】,堆层(heap),接口层(emalloc/efree) [这里PHP内核定义了一些列的宏而不是直接调用函数,这样就直接隔离了外部调用和PHP内存管理的内部实现]

<h4>Heap 层</h4>
PHP 中内存管理主要是维护三个列表,小块内存列表(free_buckets)【双向链表+数组】,大块内存列表(large_free_buckets)【数组+树形结构+双向链表】和剩余内存列表(rest_buckets)【双向链表】.

堆层结构体如下:  
	`struct _zend_mm_heap {`  
	`int                 use_zend_alloc;//是否采用zend管理内存`  
	`void               *(*_malloc)(size_t); //内存申请函数`  
	`void                (*_free)(void*);	//内存释放函数`  
	`void               *(*_realloc)(void*, size_t);//内存重新分配函数`  
	`size_t              free_bitmap;//小块空闲标识`  
	`size_t              large_free_bitmap; //大块空闲标识`  
	`// 一次内存分配的段大小，即ZEND_MM_SEG_SIZE指定的大小，默认为ZEND_MM_SEG_SIZE   	(256 * 1024)下面有宏定义`  
	`size_t              block_size;`   
	`size_t              compact_size;//压缩操作边界值，为ZEND_MM_COMPACT指定大		小，默认为 2 * 1024 * 1024`  
	`zend_mm_segment    *segments_list;//段指针列表`  
	`zend_mm_storage    *storage;//所调用的存储层`  
	`size_t              real_size;//堆的真实大小`  
	`size_t              real_peak;//堆真实大小的峰值`   
	`size_t              limit;//堆的内存边界`  
	`size_t              size;//堆大小`  
	`size_t              peak;//堆大小的峰值 ` 
	`size_t              reserve_size;//备用堆大小`  
	`void               *reserve;//备用堆`  
	`int                 overflow;//内存溢出数`  
	`int                 internal;`  
	`#if ZEND_MM_CACHE`  
	`unsigned int        cached;// 已缓存大小`  
	`zend_mm_free_block *cache[ZEND_MM_NUM_BUCKETS]; //缓存的数组 ` 
	`#endif`  
	`//小块内存数组`  
	`zend_mm_free_block *free_buckets[ZEND_MM_NUM_BUCKETS*2];`   
	`// 大块内存`        
	`zend_mm_free_block *large_free_buckets[ZEND_MM_NUM_BUCKETS];`
	`zend_mm_free_block *rest_buckets[2];//剩余内存数组 ` 
	`int                 rest_count;`  
	`#if ZEND_MM_CACHE_STAT ` 
	`struct {`  
		`int count;`  
		`int max_count;`  
		`int hit;`  
		`int miss;`  
	`} cache_stat[ZEND_MM_NUM_BUCKETS+1]; ` 
	`#endif ` 
	`};`  


<strong>小块内存管理</strong>  
free_buckets 是一个数组指针,本质是指向zend_mm_free_block的结构体,zend_mm_free_bloc结构体定义如下:  
	`typedef struct _zend_mm_free_block {`  
	`zend_mm_block_info info;`  
	`#if ZEND_DEBUG //这里ZEND_DEBUG是常量`  
	`unsigned int magic;` 
	`# ifdef ZTS`  
	`THREAD_T thread_id;`  
	`#endif`  
	`#endif`  
	`struct _zend_mm_free_block *prev_free_block;`  
	`struct _zend_mm_free_block *next_free_block;`  
	`struct _zend_mm_free_block **parent;`
	`struct _zend_mm_free_block *child[2];`  
	`} zend_mm_free_block;`  
在内存初始化时,PHP内核初始化free_buckets数组链表,free_buckets数组默认大小为64,通过下面的宏来查找数组小标:



