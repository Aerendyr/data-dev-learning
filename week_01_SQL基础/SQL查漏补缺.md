📝 SQL 知识清单自测表
一、 基础查询与过滤（DQL）
□ 基础查询：SELECT, FROM, DISTINCT 去重                                       都会，distinct用的少，不熟练。。。
□ 条件过滤：WHERE, AND, OR, NOT						   会用
□ 范围与集合：IN, BETWEEN, LIKE（模糊查询 / 通配符 % _）          会用，  like 有点忘了
□ 空值处理：IS NULL, IS NOT NULL                                                     会用
□ 排序与限制：ORDER BY (ASC/DESC), LIMIT / OFFSET (分页)      会用，  limit和offset有点忘记了
二、 常用函数
□ 聚合函数：COUNT, SUM, AVG, MAX, MIN						会用
□ 字符串函数：CONCAT 拼接, SUBSTRING 截取, TRIM 去空格, REPLACE 替换			听过
□ 日期函数：NOW() / CURRENT_DATE, DATE_FORMAT (日期格式化), DATEDIFF (日期差)		听过
□ 条件判断函数：CASE WHEN ... THEN ... ELSE ... END						会用，不熟练
□ 空值替换：IFNULL (MySQL) / COALESCE (标准SQL)                                           emm,学的MySQL
三、 分组与聚合
□ 分组：GROUP BY		会
□ 分组后过滤：HAVING （知道它和 WHERE 的区别：WHERE 在分组前，HAVING 在分组后）           会
□ 多字段分组                            会
四、 多表连接 (JOIN)
□ 内连接：INNER JOIN （只保留两表匹配的行）            会
□ 左连接 / 右连接：LEFT JOIN / RIGHT JOIN （保留左/右表所有行）   会
□ 全连接：FULL OUTER JOIN （MySQL 不支持，通常用 UNION 代替）           不会
□ 交叉连接：CROSS JOIN （笛卡尔积）       会
□ 集合操作：UNION (去重合并), UNION ALL (不去重合并)          不会
五、 子查询 (Subquery)
□ 标量子查询：返回单个值的子查询（放 SELECT 或 WHERE 里）    会
□ 列子查询：配合 IN, ANY, ALL 使用          会
□ 表子查询：放在 FROM 后面作为临时表       会，，，，不熟
□ 相关子查询：子查询依赖外部查询的字段（如 EXISTS / NOT EXISTS）         不会
六、 进阶查询（⭐ 重点区）
□ 窗口函数：OVER(), PARTITION BY, ORDER BY           会，不熟
□ 排名函数：ROW_NUMBER(), RANK(), DENSE_RANK()        会
□ 偏移函数：LAG(), LEAD()               不会
□ 聚合窗口：SUM() OVER(...), AVG() OVER(...)
□ CTE (公用表表达式)：WITH 表名 AS (子查询)            不会
□ 递归 CTE：WITH RECURSIVE （处理树形结构或层级数据）     不会
七、 数据操作与定义 (DML / DDL)    会，但都忘记了，不熟
□ 插入数据：INSERT INTO
□ 更新数据：UPDATE ... SET ... WHERE ...
□ 删除数据：DELETE FROM ... WHERE ...
□ 建表与删表：CREATE TABLE, DROP TABLE, TRUNCATE TABLE
□ 修改表结构：ALTER TABLE (加字段、改类型、删字段)
八、 约束与设计    会  ，，，，忘记了，不熟
□ 主键：PRIMARY KEY
□ 外键：FOREIGN KEY
□ 唯一约束：UNIQUE
□ 非空与默认值：NOT NULL, DEFAULT
九、 性能优化与高级概念（了解即可）       完全不会
□ 索引：什么是索引，普通索引、唯一索引、联合索引
□ 执行计划：EXPLAIN 分析 SQL 性能
□ 事务：BEGIN, COMMIT, ROLLBACK，ACID 特性
□ 视图：CREATE VIEW
□ 存储过程 / 触发器（非必须，看工作需求）