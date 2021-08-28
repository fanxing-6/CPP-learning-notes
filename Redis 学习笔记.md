# Redis 学习笔记

Redis 是**Re**mote **Di**ctionary **S**ervice 的简称；也是远程字典服务；

Redis 是内存数据库，KV数据库，数据结构数据库；

Redis命令查看：http://redis.cn/commands.html

## 认识Redis

![image-20210823213819221](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210823213819221.png)

1. string 	是二进制安全字符串,不以分隔符'\0'界定字符串而是以长度界定字符串 相当于`std::string`
2. hash       相当于C++中的`unordered_map`用`hashtable`实现   
3. list           双向首尾相接链表
4. set          无序 如果都是整数<512 或者元素不是整数 会转换成哈希结构
5. zset        有序 指定排序字段 核心数据结构(分布式应用) 

## Redis存储结构



![image-20210823221137703](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210823221137703.png)





每种数据结构都有各种存储方式

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210823222254145.png" alt="image-20210823222254145" style="zoom:150%;" />

> redis 内存分配器 <= 64 字节 小字符串 >=64字节 大字符串

## string

