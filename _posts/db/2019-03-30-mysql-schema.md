---
title: Schema与数据类型
date: 2016-02-24 22:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
- MySQL
star: true
category: blog
author: Jack
description: MySQL 优化
---


1， 数据类型

a, 更小的通常更好
b, 简单就好
c, 尽量避免NULL

选择合适的大类型 ： 数字， 字符串， 时间等


整数类型

整数 ：tinyint(8) smallint(16) mediumint(24) int(32) bigint(64), 具有unsigned属性，范围提高一倍。
实数 ： float/doube使用标准浮点运算；decimal用户存储精确小数。
字符串 ： varchar使用额外的1或者2字节记录字符串长度；char是固定长度，存储时删除所有末尾的空格。
	binary/varbinary存储二进制字符串(\0填充）；
	blob/text存储很大数据设计的字符串,mysql当作单独对象处理。
	枚举代替字符串，存储为数字，添加或者修改修改表。varchar/char与枚举关联比varchar/char直接关联更慢；
日期： datetime 8位存储最多9999年，最小单位位秒；timestamp保存自1970年以来的时间秒数，存储4个字节。
位： bit保存指定位数的，尽量避免使用，可以采用char(1)替代；set存储多个true/false，可以使用tinyint替代。
选择标识符： 整数是最好的选择，速度最快可以设置auto_increment;enum/set是个糟糕的选择；字符串很消耗空间，并且比数字要慢很多，存储uuid可以使用uhhex()转换为uuid16进制数字
其他类型： IP 地址转换为数字保存

设计陷阱

a, 太多的列
b, 太多的关联
c, 全能的枚举
d, 变相的枚举
e, 不要使用null，提供默认值


范式与反范式

范式更新更快；存储和更新更小的数据；


缓存表与汇总表

提升性能的方法冗余数据到数据表；缓存表将缓慢获取的数据保存到表中；汇总表使用group by聚合的数据

物化视图 预先计算并存储在磁盘上的表，提供不同的策略刷新/更新。过程： 变更数据的获取；创建管理物化视图的过程；应用到数据库
计数器表 


加快alter table 的速度

MySQL修改表过程： 创建空表，复制数据，删除老的表，这个过程中，会导致MySQL服务中断
在不提供服务的表中进行alter table，然后提供主库切换； 创建与要求表结构创建一张源表无关的新表，然后通过重命名和删除操纵完成

只修改frm文件
创建新表
flush table with read lock 
交换.frm文件
unlock table

快速创建MyISAM 索引 ： 禁用索引，载入数据，重新启动索引



2， 

