---
title: Postgre CRUD
date: 2018-09-27
---
# Postgre CRUD
## string or keyword
    select 'string';
    select "count"(1) from "table_name"

## insert 
    CREATE TABLE users (id INT, counters JSONB NOT NULL DEFAULT '{}');
    INSERT INTO users (id, counters) VALUES (1, '{"bar": 10}');

语法

    INSERT INTO "table1" ("created_at","status") VALUES ('2019-05-13 15:34:51','1') RETURNING "table1"."id";

### last insert id
3 ways,all are concurrent safe:

    SELECT currval(table_name+'_id_seq');
    SELECT LASTVAL();
    insert into .... RETURNING id;

### insert+update
1. 加条insert+update 两句
INSERT INTO table (id, field, field2)
       SELECT 3, 'C', 'Z'
       WHERE NOT EXISTS (SELECT 1 FROM table WHERE id=3);
2. insert + onconflict do update

    INSERT INTO the_table (id, column_1, column_2) VALUES (1, 'A', 'X'), (2, 'B', 'Y'), (3, 'C', 'Z')
    ON CONFLICT (id) 
    DO UPDATE 
        SET column_1 = EXCLUDED.column_1, -- 更新值
            column_2 = the_table.column_2  -- 保留值
        [RETURNING id];
    DO NOTHING;

#### on conflict
`ON CONFLICT [target] action [RETURNING id]`:

    target:
        (uid, phone)
        ON CONSTRAINT constraint_name
        WHERE predicate
    action:
        DO UPDATE SET column_1 = EXCLUDED.value_1,v2, .. WHERE condition

可以什么都不做时，就不需要target:

    ON CONFLICT DO NOTHING

## select 
双引号 反引号。表示特殊的字段：

    select "abc";
    ERROR:  column "abc" does not exist

字符应该用单引号

    select 'abc'

### between

    hourt between 0 and 23

### group by 
`wmname` must appear in the GROUP BY clause or be used in an aggregate function

    SELECT cname, MAX(avg)  FROM makerar GROUP BY cname;

#### group by expression
group by hour

    group by date_trunc('hour', ctime)
    group by date_trunc('day', ctime)
    group by date_trunc('week', ctime)

不等价:

    select hour/2 as hour2 from users group by hour2;
    select hour/2 as hour from users group by hour;

with order by:

    select hour/2 as hour2 from users group by hour2 order by hour2;

#### group by with top N
https://stackoverflow.com/questions/3800551/select-first-row-in-each-group-by-group

不存在first 这个函数, 下面是伪代码

    # SELECT FIRST(id), customer, FIRST(total) FROM  purchases GROUP BY customer ORDER BY total DESC;

Supported by any database: 利用 group + column=max(column)

    SELECT MIN(x.id),  -- change to MAX if you want the highest id
         x.customer, 
         x.total
    FROM PURCHASES x
    JOIN (SELECT p.customer,
            MAX(total) AS max_total
            FROM PURCHASES p
            GROUP BY p.customer) y 
    ON y.customer = x.customer AND y.max_total = x.total
    GROUP BY x.customer, x.total




#### partition
partition:

    # SELECT FIRST(id), customer, FIRST(total) FROM  purchases GROUP BY customer ORDER BY total DESC;
    WITH summary AS (
        SELECT p.id, 
            p.customer, 
            p.total, 
            ROW_NUMBER() OVER(PARTITION BY p.customer 
                                    ORDER BY p.total DESC) AS rk
        FROM PURCHASES p)
    SELECT s.*
    FROM summary s
    WHERE s.rk = 1

##### delete duplicated row
delete and keep top 2 row(order by peg) with group by industry

    DELETE FROM stock where id in 
        (select id  FROM (
            SELECT
                ROW_NUMBER() OVER (PARTITION BY industry ORDER BY peg) AS r, stock.*
            FROM
            stock) x WHERE x.r > 2
        )

多维支持

    PARTITION BY company, department

### concat rows
合并为array

    SELECT uid, ARRAY_AGG (first_name || ' ' || last_name) fullname FROM users; 
    select code, ARRAY_AGG(label) l from t group by code;

合并为字符串：

    -- GROUP BY 1 is a positional reference and a shortcut for GROUP BY movie
    SELECT movie, string_agg(actor, ', ') AS actor_list FROM tbl GROUP  BY 1; 

### DISTINCT
> https://stackoverflow.com/questions/18539223/select-random-row-for-each-group-in-a-postgres-table
> SELECT DISTINCT ON expressions must match initial ORDER BY expressions

distinct 就是分组，然后按序(order by 取top1)/或随机取一个row

    SELECT DISTINCT ON (customer)
        id, customer, total
    FROM   purchases
    ORDER  BY customer, total DESC, id;

Or shorter (if not as clear) with ordinal numbers of output columns:

    # 2:customer 1:id 3:total
    SELECT DISTINCT ON (2)
        id, customer, total
    FROM   purchases
    ORDER  BY 2, 3 DESC, 1;

`distinct on(fieldList)` 必须匹配`order by list`. list与fieldlist 必须要有交集
`fieldList`中字段顺序不影响分组及结果, `list` 排序则影响顺序

    # ERROR: DISTINCT ON expressions must match initial ORDER BY expressions: (code,pe)与end_date 没有交集
    select distinct on (code,pe) code,pe,end_date from profits order by end_date;
    # 以code,end_date分组
    select distinct on (code,end_date) code,pe,end_date from profits order by end_date;
    # 以code分组, 取最新end_date
    select distinct on (code) code,end_date,pe,peg from profits order by code,end_date desc

where filter, 会导致提前过滤掉的分组top1行, 导致留下topN列上位顶替原来的top1

    # 比如我想找所有股票code 中，最新end_date财报中, peg<1 的(有问题)
    select distinct on (code,end_date) code,end_date,pe from profits where peg>0 order by code,end_date desc limit 1
    # 没有问题
    select p.* from (select distinct on (code) code,end_date,pe,peg from profits order by code,end_date desc  ) p where p.peg<1;


If total can be NULL (won't hurt either way, but you'll want to match existing indexes):

    ORDER  BY customer, total DESC NULLS LAST, id; //null 排序时放在最后

## del

    SELECT '{"a":[null,{"b":[3.14]}]}' #- '{a,1,b,0}'

## update

	UPDATE ed_names SET c_request = c_request+1 WHERE id = 'x'"

### update DUPLICATE
mysql update 多个unique key 时,如果遇到 `duplicate key`, 比如所有的i加1

一般情况下可以通过排序避免(mysql update/insert 时会按一定的顺序去查数据是否有效):

	UPDATE <table> set i=i+1 where id>10 order by i desc; # 大的先加1，避免冲突

如果`i`没有加索引，排序比较耗时或内存，就变通一下，比如：放弃排序，先负数：

	UPDATE <table> set i=-i where id>10; # 先变负，就不存在+1冲突
	UPDATE <table> set i=1-i where id>10;
