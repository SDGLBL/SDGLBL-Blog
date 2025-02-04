---
title: Redis 有序集合 (Sorted Set) 的常见应用场景与性能优化
description: 本文介绍了 Redis 有序集合 (Sorted Set) 在排行榜系统、延迟队列、相关度搜索和 UV 统计等场景下的应用，并提供了一些性能优化建议。
tags:
  - Redis
  - Sorted
  - Set
  - 数据结构
  - redis-datastructure
  - performance-optimization
  - database-design
share: "true"
---

> **核心价值**: 有序集合(Sorted Set)是Redis中一个独特而强大的数据结构，它通过将分数与成员关联的方式，为解决排序、排名、范围查询等常见业务问题提供了优雅的解决方案。

# 常见应用场景

## 1. 实时排行榜系统

> **场景特点**: 需要频繁更新分数，同时要求快速获取排名和范围查询

```python
# 游戏积分排行榜实现
class GameLeaderboard:
    def __init__(self, redis_client, leaderboard_key):
        self.redis = redis_client
        self.key = leaderboard_key
    
    def update_score(self, player_id, score):
        """更新玩家分数"""
        self.redis.zadd(self.key, {player_id: score})
    
    def get_rank(self, player_id):
        """获取玩家排名（从0开始）"""
        return self.redis.zrevrank(self.key, player_id)
    
    def get_score(self, player_id):
        """获取玩家分数"""
        return self.redis.zscore(self.key, player_id)
    
    def get_top_players(self, count=10):
        """获取前N名玩家"""
        return self.redis.zrevrange(self.key, 0, count-1, withscores=True)
    
    def get_nearby_players(self, player_id, count=5):
        """获取玩家附近的排名"""
        rank = self.get_rank(player_id)
        start = max(0, rank - count)
        end = rank + count
        return self.redis.zrevrange(self.key, start, end, withscores=True)
```

使用示例：
```python
# 初始化排行榜
leaderboard = GameLeaderboard(redis_client, "game:scores:202401")

# 更新分数
leaderboard.update_score("player:1001", 2500)
leaderboard.update_score("player:1002", 3000)

# 获取前十名
top_players = leaderboard.get_top_players()

# 获取玩家周边排名
nearby = leaderboard.get_nearby_players("player:1001", 2)
```

## 2. 延迟队列/定时任务

> **场景特点**: 使用时间戳作为分数，实现按时间排序的任务队列

```python
class DelayedTaskQueue:
    def __init__(self, redis_client, queue_key):
        self.redis = redis_client
        self.key = queue_key
    
    def add_task(self, task_id, execute_time):
        """添加定时任务"""
        self.redis.zadd(self.key, {task_id: execute_time})
    
    def get_ready_tasks(self):
        """获取到期任务"""
        now = time.time()
        # 获取所有到期的任务
        tasks = self.redis.zrangebyscore(self.key, 0, now)
        if tasks:
            # 原子性移除这些任务
            self.redis.zremrangebyscore(self.key, 0, now)
        return tasks
    
    def schedule_task(self, task_id, delay_seconds):
        """延迟指定秒数执行任务"""
        execute_time = time.time() + delay_seconds
        self.add_task(task_id, execute_time)
```

使用示例：
```python
# 初始化延迟队列
delay_queue = DelayedTaskQueue(redis_client, "delayed:tasks")

# 添加延迟任务
delay_queue.schedule_task("email:1001", 3600)  # 1小时后执行
delay_queue.schedule_task("notification:2001", 1800)  # 30分钟后执行

# 在工作进程中获取到期任务
ready_tasks = delay_queue.get_ready_tasks()
```

## 3. 相关度搜索

> **场景特点**: 使用相关度分数实现搜索结果的权重排序

```python
class WeightedSearch:
    def __init__(self, redis_client, index_key):
        self.redis = redis_client
        self.key = index_key
    
    def index_document(self, doc_id, score):
        """索引文档，score代表相关度"""
        self.redis.zadd(self.key, {doc_id: score})
    
    def search(self, offset=0, limit=10):
        """获取相关度最高的文档"""
        return self.redis.zrevrange(
            self.key, offset, offset+limit-1, 
            withscores=True
        )
    
    def update_relevance(self, doc_id, increment):
        """增加文档的相关度分数"""
        self.redis.zincrby(self.key, increment, doc_id)
```

## 4. UV统计与去重

> **场景特点**: 使用时间戳作为分数，实现按时间范围的去重统计

```python
class TimeSeriesUV:
    def __init__(self, redis_client, uv_key):
        self.redis = redis_client
        self.key = uv_key
    
    def record_visit(self, user_id):
        """记录用户访问"""
        self.redis.zadd(self.key, {user_id: time.time()})
    
    def get_uv_in_range(self, start_time, end_time):
        """获取时间范围内的UV"""
        return self.redis.zcount(self.key, start_time, end_time)
    
    def cleanup_old_data(self, before_time):
        """清理指定时间之前的数据"""
        self.redis.zremrangebyscore(self.key, 0, before_time)
```

# 性能优化建议

1. **批量操作优化**
```python
# 不推荐
for user_id, score in user_scores:
    redis.zadd("leaderboard", {user_id: score})

# 推荐
redis.zadd("leaderboard", {
    user_id: score for user_id, score in user_scores
})
```

2. **范围查询优化**
```python
# 分页优化
def get_page(key, page_size, page_num):
    start = (page_num - 1) * page_size
    end = start + page_size - 1
    return redis.zrevrange(key, start, end, withscores=True)
```

3. **内存优化**
```python
# 定期清理过期数据
def cleanup_leaderboard(key, max_members):
    # 保留前N名
    redis.zremrangebyrank(key, 0, -max_members-1)
```

# 实践要点

1. **数据一致性**
   ```python
   def atomic_update(redis, key, member, new_score):
       with redis.pipeline() as pipe:
           pipe.zscore(key, member)  # 检查当前分数
           pipe.zadd(key, {member: new_score})
           results = pipe.execute()
           return results[0]  # 返回原始分数
   ```

2. **过期策略**
   ```python
   # 为整个有序集合设置过期时间
   redis.zadd("daily:ranks", {"user:1": 100})
   redis.expire("daily:ranks", 86400)  # 24小时后过期
   ```

3. **监控与维护**
   ```python
   def monitor_zset_size(redis, key):
       size = redis.zcard(key)
       members = redis.zrange(key, 0, -1, withscores=True)
       memory = redis.memory_usage(key)
       return {
           "size": size,
           "memory": memory,
           "sample": members[:5]
       }
   ```

# 总结

Redis有序集合通过其独特的设计，为众多实际应用场景提供了优雅的解决方案。关键优势：

1. **实时性**: 所有操作都是即时的，无需批处理
2. **原子性**: 单个命令的执行是原子的，保证数据一致性
3. **效率**: 大多数操作的复杂度都是对数级别
4. **灵活性**: 可以通过分数实现多种业务逻辑

> **最佳实践**: 在使用有序集合时，应当注意控制集合大小，合理设置过期策略，并善用批量操作来提升性能。同时，要根据实际业务场景选择合适的命令组合，避免不必要的操作。