## ArrayList源码笔记

* ArrayList默认的大小的是10
* 允许最大容量为0x7fffffff-8
* 容量扩充 容量扩充默认为 原数组容量+原数组容量/2
* 如果无法分配内存扩展容量会引发`OutOfMemoryError`异常
* ArrayList本质是内部维护了一个`Object[]`数组