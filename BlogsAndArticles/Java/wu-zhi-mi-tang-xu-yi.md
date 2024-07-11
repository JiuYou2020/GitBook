---
description: 对上一步缺少内容的补充，内容格式基本一致
---

# 吾之蜜糖（续一）

## 基础

### java创建对象的方式

1. new关键字
2. 反射
3. clone()方法
4. 对象序列化与反序列化
5. 使用静态工厂方法`MyClass obj = MyClass.createInstance();`

io

## 集合



## 并发



## 虚拟机



## SpringMVC



## SpringBoot



## Spring



## 缓存



## MySQL

### 内连，左连和右连

1. **内连接（Inner Join）**：
   - 内连接根据两个表中的连接谓词（Join Predicate）匹配的行，将符合条件的行组合起来。
   - 语法示例：
     ```sql
     SELECT *
     FROM table1
     INNER JOIN table2 ON table1.column = table2.column;
     ```
   - 内连接返回两个表中连接列匹配的行，如果某个表中的行在另一个表中找不到匹配行，则不包括在结果集中。

2. **左连接（Left Join 或 Left Outer Join）**：
   - 左连接返回左表中的所有行，以及右表中与左表中的行匹配的行。
   - 如果右表中没有与左表中的行匹配的行，则结果集中右表的对应列显示为 NULL。
   - 语法示例：
     ```sql
     SELECT *
     FROM table1
     LEFT JOIN table2 ON table1.column = table2.column;
     ```

3. **右连接（Right Join 或 Right Outer Join）**：
   - 右连接与左连接类似，返回右表中的所有行，以及左表中与右表中的行匹配的行。
   - 如果左表中没有与右表中的行匹配的行，则结果集中左表的对应列显示为 NULL。
   - 语法示例：
     ```sql
     SELECT *
     FROM table1
     RIGHT JOIN table2 ON table1.column = table2.column;
     ```



## Redis

### 如何优化Redis内存？

1. **优化数据结构选择**：使用更适合存储数据的数据结构。例如，如果某些数据可以使用更省内存的数据结构如Hashes、Sets、Zsets等替代，可以考虑进行转换。
2. **数据压缩**：对于某些值较大的字符串类型的数据，可以考虑使用Redis的压缩功能（如Redis 6.x引入的字符串压缩）来减少存储空间。
3. **过期策略调整**：根据业务需求和数据特性，合理设置过期时间，及时清理不再需要的数据，避免内存被长时间占用。
4. **增加持久性**：Redis 支持将内存中的数据保存到磁盘中，这样即使出现服务器宕机等情况，也不会丢失数据。但是，将数据写入磁盘需要消耗额外的内存和 CPU
5. **定期清理过期数据**：可以定期清理 Redis 中的过期键值对，以避免 Redis 中出现大量无用数据，从而节省内存。
6. **控制最大内存**：可以通过配置文件中的“maxmemory”选项设置 Redis 最大可用内存，当 Redis 内存使用达到该值时，Redis 将自动删除一些键值对，以释放空间。

## ElasticSearch



## MongoDB

