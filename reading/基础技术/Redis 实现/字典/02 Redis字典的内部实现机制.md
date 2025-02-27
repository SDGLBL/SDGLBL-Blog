---
title: Redis字典的内部实现机制
description: 本文深入探讨了Redis字典的双哈希表实现,涵盖了字典结构、哈希表结构、节点结构,以及键值对存储、rehash等关键机制,分析了其内存管理和性能优化策略。通过对Redis字典的学习,可以了解优秀的工程设计理念。
author:
  - SDGLBL
created: 2025-02-04
tags:
  - Redis
  - 字典
  - 数据结构
  - redis-data-structures
  - hash-tables
  - internal-mechanism
  - note
  - 笔记
share: "true"
---



Redis的字典实现是一个精心设计的系统工程，它通过巧妙的数据结构组合和算法实现，在性能和可维护性之间取得了出色的平衡。让我们深入探讨其内部构造。

> **核心概念**: Redis字典采用了一个双哈希表结构，这种设计为动态扩容和收缩提供了基础支持，同时保证了操作的稳定性。

# 字典的内部结构

Redis的字典由三个关键结构组成，它们相互配合，形成了一个完整的键值存储系统：

## dict 结构

```c
typedef struct dict {
    dictType *type;     // 类型特定函数
    void *privdata;     // 私有数据
    dictht ht[2];       // 哈希表数组
    long rehashidx;     // rehash索引
    int iterators;      // 安全迭代器计数
} dict;
```

这个顶层结构展现了Redis字典实现的核心思想：

1. **类型抽象**
   - `dictType`提供了对不同类型键值对的操作函数
   - 支持自定义的哈希函数和比较函数
   - 实现了类型安全的值处理机制

2. **双哈希表设计**
   - `ht[2]`数组持有两个哈希表
   - 支持渐进式rehash的核心机制
   - 实现了无锁的数据迁移

> **设计亮点**: 双哈希表的设计是Redis字典实现的一个关键创新，它让字典能够在不中断服务的情况下进行扩容或收缩。

## dictht 哈希表结构

```c
typedef struct dictht {
    dictEntry **table;      // 哈希表数组
    unsigned long size;     // 哈希表大小
    unsigned long sizemask; // 掩码(size-1)
    unsigned long used;     // 已使用节点数
} dictht;
```

哈希表结构体现了以下设计考虑：

1. **内存效率**
   - size总是2的幂，便于哈希计算
   - sizemask用于快速计算索引
   - used字段支持负载因子计算

2. **性能优化**
   - table数组存储指针而非实际数据
   - 支持快速的索引定位
   - 便于动态扩容和收缩

## dictEntry 节点结构

```c
typedef struct dictEntry {
    void *key;             // 键
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;                   // 值
    struct dictEntry *next; // 链表指针
} dictEntry;
```

节点结构展现了以下特点：

1. **灵活性**
   - 通过union实现多类型值存储
   - 指针类型的key支持任意类型键
   - next指针支持链地址法解决冲突

2. **内存优化**
   - union共享内存空间
   - 针对不同数值类型的优化
   - 紧凑的内存布局

# 工作原理

## 键值对的存储过程

1. **键哈希计算**
   ```c
   hash = dict->type->hashFunction(key);
   index = hash & dict->ht[0].sizemask;
   ```

2. **冲突处理**
   - 使用链地址法处理哈希冲突
   - 新节点插入到链表头部
   - 查找时遍历链表

3. **rehash状态处理**
   - 检查是否在rehash过程中
   - 确定操作的目标哈希表
   - 必要时执行渐进式rehash步骤

## 内存管理机制

Redis的字典实现在内存管理上也做了精心设计：

4. **空间分配**
   - 预分配优化
   - 增量式空间释放
   - 内存对齐考虑

5. **内存布局**
   - 连续的table数组
   - 离散的节点存储
   - 优化的指针结构

# 实现细节的思考

Redis字典实现中的一些细节反映了深入的工程思考：

6. **类型系统**
   字典的类型系统设计展现了优秀的抽象能力：
   - 通过函数指针实现多态
   - 类型安全的值处理
   - 可扩展的接口设计

7. **性能考虑**
   实现中多处体现了性能优化：
   - 位运算优化
   - 缓存友好的数据结构
   - 算法复杂度的权衡

8. **可维护性**
   代码结构展现了良好的工程实践：
   - 清晰的职责分离
   - 一致的错误处理
   - 完善的注释文档

# 总结与启示

Redis字典的实现提供了以下工程启示：

9. **抽象的价值**
   - 良好的抽象带来了扩展性
   - 接口设计决定了使用方式
   - 实现细节影响了性能表现

10. **工程权衡**
   - 内存使用和性能的平衡
   - 复杂度和可维护性的取舍
   - 通用性和专用性的权衡

通过研究Redis字典的实现，我们不仅能学习到具体的技术细节，更能体会到优秀的工程设计思维。这些经验对于我们自己的系统设计都有重要的参考价值。