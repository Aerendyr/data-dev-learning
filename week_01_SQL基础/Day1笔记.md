迷迷又茫茫，第一天。。加油

窗口函数：保留明细行，数据不折叠

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

聚合函数 ＋ over（） 变成窗口函数
窗口帧： rows between 起点 and 终点   
计算范围内的聚合结果
unbounded preceding :分区第一行（起点）
n preceding : 当前行的前n行
current row: 当前行
n following: 当前行的后n行
unbounded following: 分区的最后一行（终点）

默认 between unbounded preceding and unbounded following 整个分区



CREATE DATABASE IF NOT EXISTS Day_01;
CREATE TABLE sales (
    id INT PRIMARY KEY,
    emp_name VARCHAR(20),
    dept VARCHAR(20),
    sale_date DATE,
    amount INT
);

INSERT INTO sales VALUES
(1, '张三', 'A', '2024-01-01', 100),
(2, '李四', 'A', '2024-01-02', 200),
(3, '王五', 'A', '2024-01-03', 150),
(4, '张三', 'A', '2024-01-04', 300),
(5, '赵六', 'B', '2024-01-01', 500),
(6, '钱七', 'B', '2024-01-02', 400),
(7, '赵六', 'B', '2024-01-03', 600),
(8, '钱七', 'B', '2024-01-04', 200);

-- select...over(partition by ..order by ... )... from ..where ..group by ...having ... order by ..limit...
-- 执行顺序 from-->where-->group by-->having-->select-->order by-->limit  (窗口函数在select阶段执行)


-- select * , 函数 over(partition by 分组列 order by .....)  
-- 常见窗口函数：
-- 排序函数
-- row_number()  连续唯一序号   1，2，3，4，5
-- rank()  并列跳号  1，1，3，4、
-- dense_rank() 并列不跳号  1，1，2，3，4
-- ntile(n) 分成n个桶
-- percent_rank() 百分比排名
-- cume_dist()  累积分布


-- 偏移函数
-- lag(col,n,default)

SELECT id, emp_name, dept, amount, 
ROW_NUMBER() OVER(PARTITION BY dept ORDER BY amount DESC) AS rn,
RANK() OVER(PARTITION BY dept ORDER BY amount DESC) AS rnk,
DENSE_RANK() OVER(PARTITION BY dept ORDER BY amount DESC ) AS dense_rnk
FROM sales
ORDER BY dept, amount DESC; 


SELECT id, emp_name, dept, sale_date, amount, 
SUM(amount) 
OVER(PARTITION BY dept 
ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS dept_running_total 
FROM sales ORDER BY dept, sale_date;

SELECT *, SUM(amount) OVER( PARTITION BY dept ORDER BY sale_date 
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS c FROM sales;











