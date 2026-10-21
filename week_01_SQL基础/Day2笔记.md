遗留问题：时间维度表  、 问题A,B,C


任务一：row_number/rank/dense_rank/ntile(n)
    row_number 不并列不跳号 1，2，3，4，5
    rank 并列跳号 1，1，3，4，5
    dense_rank 并列不跳号 1，1，2，3，4
    ntile(n) 分成n份， 当n=100时，即百分比
    percent_rank() 百分比排名
    cume_dist()  累积分布


with 表名1 as ( 
    查询语句，临时表
    select .......
),
    表名2 as (),
    ......
    表名n as ()
select ........

with 语句代替 表子子查询。。。
子查询套娃，复杂，可读性差，    且按照执行顺序来说。例如，窗口函数等执行在select阶段，在when之后,when 没办法直接引用她们来筛选。。。

任务二   偏移函数
lag(col,n,default)  即列名， 偏移量 ，默认值  :向前第n行
lead(col, n, default)  :往后第n行

取值类型的函数：
first_value(col)  窗口第一行
last_value(col)   窗口最后一行
nth_value(col,n)  窗口第n行


？？？若没有with，用子查询碰到这种问题怎么办



窗口帧，怎么设置默认值，，，，超出范围怎么办。。


'''
⚠️ 面试与实战中的核心坑点
时间断层问题
如果1月3日没有销售记录，LAG(sales) 取到的会是1月2日的数据，但业务上我们需要的是“前一天”的数据。
解决办法：如果是时间序列分析，必须先用 “日期维度表” 进行 LEFT JOIN 补齐日期，然后再用窗口函数。


日期维度表？？？

ROWS vs RANGE
ROWS BETWEEN 是按物理行数算的。
RANGE BETWEEN 是按逻辑值算的（例如同一天的数据算在一个窗口里）。
如果日期有重复，计算累计求和时最好用 RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW，避免同一天的数据算一半漏一半。

LAG 和 LEAD 经常配合 IFNULL 或 COALESCE 使用
当偏移越界时返回 NULL，直接参与计算会导致结果全是 NULL。
'''



题目 A（LAG/LEAD 实战）：环比增长率
要求：计算每天的销售额，以及相比于前一天的环比增长率（即：(今日-昨日)/昨日 * 100%）。如果前一天为空，则显示 0。

题目 B（累计求和实战）：分区累计
要求：假设表里有华东和华北两个区域的数据，请分别计算两个区域各自的累计销售额，并保证按日期排序。

题目 C（滑动窗口实战）：找峰值
要求：计算每天销售额，以及当前天及前后各1天（共3天）的最高销售额（提示：使用 MAX() + ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING）

真实业务两个问题：一是订单流水颗粒度，，，一天有多个订单，需要先聚合计算出一天的销售额再开窗，，，先聚合再开窗；
#                二是日期缺失，日期维度表或者 弃用rows 用range    好难啊啊啊啊啊














附录：代码部分（时间维度表还没做）；
CREATE DATABASE IF NOT EXISTS Day_02;
USE day_02;

CREATE TABLE sales_perf (
    emp_name VARCHAR(20),
    region VARCHAR(20),
    sales_amount INT
);
INSERT INTO sales_perf VALUES 
('老王', '华东', 1000), ('小李', '华东', 900), ('小张', '华东', 900), ('小赵', '华东', 800), ('小刘', '华东', 500),
('大刘', '华北', 2000), ('大张', '华北', 1500), ('小孙', '华北', 1000),
('阿强', '华南', 1200), ('阿珍', '华南', 800), ('阿明', '华南', 600), ('阿花', '华南', 600), ('阿伟', '华南', 400);


SELECT * FROM sales_perf;

WITH temp AS (
SELECT emp_name, region, sales_amount, ROW_NUMBER() OVER(PARTITION BY region ORDER BY sales_amount) AS rn,
RANK() OVER(PARTITION BY region ORDER BY sales_amount DESC) AS rnk ,
DENSE_RANK() OVER(PARTITION BY region ORDER BY sales_amount DESC) AS dense_rnk ,
NTILE(100) OVER(PARTITION BY region ORDER BY sales_amount DESC) AS tier,
PERCENT_RANK() OVER(PARTITION BY region ORDER BY sales_amount DESC) AS pct_rank,
CUME_DIST() OVER(PARTITION BY region ORDER BY sales_amount DESC) AS cume_dst
FROM sales_perf
)
SELECT * FROM temp WHERE cume_dst >= 0.5 OR pct_rank <=0.3;


CREATE TABLE daily_sales (
    order_date DATE,
    region VARCHAR(20),
    sales INT
);
INSERT INTO daily_sales VALUES 
('2023-01-01', '华东', 100), ('2023-01-02', '华东', 150), ('2023-01-03', '华东', 120), 
('2023-01-04', '华东', 200), ('2023-01-05', '华东', 180);

需求：计算华东区每天的销售额，以及相比前一天的差值（今日销售额 - 昨日销售额）
SELECT order_date, 
sales AS today_sales, 
sales -LAG(sales,1) OVER(PARTITION BY region ORDER BY order_date) AS diff
FROM daily_sales WHERE region = '华东';

如果复杂一点，是按照第一个表计算的话，那要先求每日的销售额再计算差值

WITH temp AS(
SELECT order_date, region, 
SUM(sales) OVER(PARTITION BY order_date,region)AS sm
FROM daily_sales 
WHERE region = '华东'
)
SELECT order_date, region, sm,
sm-LAG(sm,1)
OVER(PARTITION BY region ORDER BY order_date) AS diff
FROM temp;



需求：计算华东区每天的销售额，以及截至当天的累计销售额。
SELECT order_date, region, sales, 
SUM(sales) OVER(PARTITION BY region ORDER BY order_date) AS sm
FROM daily_sales WHERE region = '华东';


WITH temp AS (
SELECT order_date, 
sales AS today_sales, 
sales -LAG(sales,1) OVER(PARTITION BY region ORDER BY order_date) AS diff
FROM daily_sales WHERE region = '华东'
)
SELECT diff, SUM()	

 


需求：计算每天的销售额，以及最近 3 天（含当天）的移动平均销售额。
emmmm,用sum()怎么办，，还不能除以3
窗口帧，怎么设置默认值，，，，超出范围怎么办。。
SELECT order_date, region, sales, 
AVG(sales) OVER(PARTITION BY region ORDER BY order_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS ag
FROM daily_sales WHERE region = '华东';


SELECT order_date, region, sales, 
SUM(sales)/3 OVER(PARTITION BY region ORDER BY order_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS ag
FROM daily_sales WHERE region = '华东';


SELECT 
    order_date, 
    sales,
    NTH_VALUE(sales, 3) OVER(
        ORDER BY order_date 
        -- 同样建议加上完整的窗口范围
        -- ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS third_day_sales
FROM daily_sales;




CREATE TABLE real_sales (
    order_id INT,
    order_date DATE,
    region VARCHAR(20),
    sales_rep VARCHAR(20),
    sales_amount INT
);

-- 插入包含多区域、一天多单、日期断层的真实数据
INSERT INTO real_sales VALUES 
(1, '2023-01-01', '华东', '张三', 100),
(2, '2023-01-01', '华东', '李四', 50),   -- 华东1月1日有两单，总计150
(3, '2023-01-02', '华东', '王五', 200),
(4, '2023-01-04', '华东', '张三', 120),  -- ⚠️ 注意：1月3日华东没有数据（断层）
(5, '2023-01-05', '华东', '李四', 180),
(6, '2023-01-01', '华北', '赵六', 300),
(7, '2023-01-03', '华北', '孙七', 150),
(8, '2023-01-02', '华南', '周八', 400),
(9, '2023-01-05', '华南', '吴九', 250);


计算每天的销售额，以及最近 3 天（含当天）的移动平均销售额
WITH temp AS (
	SELECT order_date, region, 
	SUM(sales_amount)  AS today_sales
	FROM real_sales
	GROUP BY order_date, region
)
SELECT order_date, region, today_sales,
	AVG(today_sales) OVER(PARTITION BY region ORDER BY order_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS av
	FROM temp;


需求：计算华东区每天的销售额，以及截至当天的累计销售额。

WITH temp AS (
SELECT  order_date, region, 
	SUM(sales_amount) AS today_sale
	FROM real_sales
	GROUP BY order_date, region
)
SELECT order_date, region, today_sale,
	SUM(today_sale) OVER(PARTITION BY region ORDER BY order_date ) AS sm
	FROM temp ;
	
需求：计算华东区每天的销售额，以及相比前一天的差值（今日销售额 - 昨日销售额）

WITH temp AS(
SELECT order_date, region, 
SUM(sales_amount) AS sm
FROM real_sales 
GROUP BY order_date,region
)
SELECT order_date, region, sm,
sm-LAG(sm,1)
OVER(PARTITION BY region ORDER BY order_date) AS diff
FROM temp;



WITH temp AS(
SELECT order_date, region, 
SUM(sales_amount) AS sm
-- SUM(sales_amount) OVER(PARTITION BY order_date,region)AS sm
FROM real_sales 
WHERE region = '华东'
GROUP BY order_date, region
)
SELECT order_date, region, sm,
sm-LAG(sm,1)
OVER(PARTITION BY region ORDER BY order_date) AS diff
FROM temp;