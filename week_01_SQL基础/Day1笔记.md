迷迷又茫茫，第一天。。加油

窗口函数：保留明细行，数据不折叠



聚合函数 ＋ over（） 变成窗口函数
窗口帧： rows between 起点 and 终点     窗口帧即划定计算范围
计算范围内的聚合结果
unbounded preceding :分区第一行（起点）
n preceding : 当前行的前n行
current row: 当前行
n following: 当前行的后n行
unbounded following: 分区的最后一行（终点）

没写order by 默认窗口帧为整组 between unbounded preceding and unbounded following 整个分区
写了order by 但是没写窗口帧,默认从第一行到当前行。。。例如求累计结果。




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



附录：代码
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











