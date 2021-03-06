---
title: 查询性能优化
date: 2019-03-30 13:58:59
image: /assets/images/markdown.jpg
headerImage: false
tag:
- MySQL
star: true
category: blog
author: Jack
description: MySQL 查询性能优化
---

1, 慢查询基础

向数据库请求了不需要的数据

响应时间 服务时间 + 等待时间（IO/锁）

	扫描表 通过设置db_block_multiblock_read_count可以设置一次IO能读取的数据块个数，从而有效减少全表扫描时的IO总次数，也就是通过预读机制将将要访问的数据块预先读入内存中。只有在全表扫描情况下才能使用多块读操作。


	扫描索引 
	范围访问 1，在唯一索引上使用了range操作符（>,<,<>,>=,<=,between）；2，在组合索引上，只使用部分列进行查询；3，对非唯一索引上的列进行的查询。
	单值访问 
	rowid访问（oracle) 由于rowid中记录了行存储的位置，所以这是oracle存取单行数据的最快方法。	


扫描的数据
返回的数据

	MySQL使用如下三种方式应用Where条件，从好到坏
	a, 索引中使用where条件过滤不匹配的记录，在存储引擎完成
	b, 使用索引覆盖扫描来返回记录，直接从索引中过滤不需要的记录并返回命中的结果，在服务层完成
	c, 从数据中返回数据，然后过滤不满足条件的结果，这是在MySQL服务层完成的。


2, 重构查询方式

	一个复杂的查询还是多个简单查询
	切分查询
	分解关联查询
		优点：
			缓存效率更高
			减少锁竞争
			应用层做关联，更容易高性能和扩展
			查询本身可以提高
			可以减少冗余，应用层查询，只需要查询一次

3, 查询基础

查询流程：

	客户端发送查询到服务器
	服务器查询缓存，命中直接返回
	服务器进行SQL解析，预处理，优化器生成执行计划
	调用存储层API执行存储计划
	将结果返回给调用者

通信协议： 半双工（服务器端发送或者客户端发送数据），一旦开始发送，另一端必须接受所有的数据


查询状态 

		sleep  线程等待客户端的请求
		query  执行查询/结果返回给客户端
		locked 服务层线程等待锁
		analyzing and statstics 收集存储引擎的统计信息，生成执行计划
		copy to the tmp table [on disk] 结果复制到临时表
		sorting result 对结果进行排序


查询缓存


查询优化处理

	语法解析和预处理
	查询优化器

		优化器由于错误原因导致选择执行计划	

			统计信息不准确
			执行计划的成本估算不等同与实际的执行成本
			没有考虑并发执行
			可能不是基于成本估计的


		能够处理的优化类型：

			重新定义关联表的顺序
			外连接转换为内连接
			使用等价变换规则
			优化count(), min(), max()
			预估并转换为常数表达式
			覆盖扫描索引
			子查询优化
			提前终止查询
			等值传播
			列表In的比较 ： in() log(n) ; or() : n


4, 查询优化器的限制
	
	关联子查询
	union限制
	索引合并优化
	等值传递
	并行执行
	哈希关联
	松散索引扫描
	最大值与最小值
	在同一个表上查询和更新

5，查询优化器的提示(hint)

	high_prioriy/low_prioriy
	delayed
	straign_join 
	sql_small_result/sql_big_result
	sql_buffer_result
	sql_cache/sql_no_cache
	sql_cal_found_rows
	for update/lock in share mode
	use index/force index/ignore index

6, 优化特定查询
	
	优化count() 
	优化关联查询
	优化子查询
	优化group by/distinct
	优化limit
	优化union



7，总结

尽量少做
不做
尽可能快的完成




