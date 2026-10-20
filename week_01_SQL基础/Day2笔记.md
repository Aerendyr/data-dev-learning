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
