# RocksDB 常见问题

<!--
ID: 4adddd36-3d0c-4600-a756-e78320ca0e76
Status: publish
Date: 2017-11-11T03:34:00
Modified: 2020-05-16T11:51:49
wp_id: 574
-->

## RocksDB 会抛出异常吗？

不会，RocksDB 会返回一个 Status 表示成功或者失败，但是 RocksDB 并没有捕获 STL 中的异常，比如 bad_alloc 这种

## 基础操作是线程安全的吗？

是的

## 可以使用多个进程 (process) 同时读写 RocksDB 吗？

不可以，当然只读模式随意
