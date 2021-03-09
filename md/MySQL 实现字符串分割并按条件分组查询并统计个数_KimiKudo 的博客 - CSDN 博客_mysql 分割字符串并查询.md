> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u010827544/article/details/88738264)

需求背景
----

昨天下午, 公司的项目需要新增一个对试卷的分析, 就是根据一张试卷中的所有试题进行知识点统计, 每个知识点各有几道题.  
乍一看十分简单, COUNT(0),GROUP BY 就好了呀, 可是问题就出在这个 GROUP BY 上, 因为一道题可能对应多个知识点, 所以在数据库中, 知识点这个字段是这么存储的:  
![](https://img-blog.csdnimg.cn/20190322105320467.png)  
是用逗号分隔的, 如果直接 GROUP BY 就可能会出现对于同一个知识点是分成两条记录来统计的.  
所以需要我们来进行字符串分割, 这个工作 Java 可以很轻松的完成, 可是之后还是要将分割结果再进行多次关联查询, 所以我更希望能够在 SQL 语句中实现.  
最后确实实现了, 所以特意写这篇博客来记录下来.

设计思路
----

逻辑并不困难, 关联查询而已, 至于分割字符串的, 我也在网上找到了参考写法  
参考了这篇博客:  
[https://blog.csdn.net/pjymyself/article/details/81668157](https://blog.csdn.net/pjymyself/article/details/81668157)  
对于这个写法的详细分析见最后的 "参考分析"

下面就要来理一下关联查询的思路

1.  表关系  
    ![](https://img-blog.csdnimg.cn/20190322113921841.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA4Mjc1NDQ=,size_16,color_FFFFFF,t_70)  
    不是太会画数据库关系图, 按着类图那么画的, 无关字段已经省略.  
    可以明显看的明白, 通过接口参数 PAPER_ID 可以得到当前试卷的试题, 通过试题可以查到试题对应的知识点 ID, 根据知识点 ID 分割之后就能查到整张试卷所有知识点.
2.  查询步骤
    *   根据参数 PAPER_ID 查询出当前试卷的试题
    *   将查询出来的试题表作为查询条件去查询 bs_questions 表, 得到知识点表
    *   将查询结果进行字符串分割操作, 最后得到所有知识点的表
    *   将知识点表关联查询 bs_chapter 表, 得到每个知识点的名称
    *   将知识点表同时再次关联通过 bs_paper_question_relations 表和 bs_questions 表关联查询的查询结果, 分组统计每个知识点对应的个数

难点
--

在理清思路之后, 发现的难点一共有两个

1.  如何分割字符串, 前面已经解决
2.  如何统计各知识点的试题个数, 不能通过 bs_questions 表的 CHAPTER_ID 字段来分组, 而是要根据该字段的值是否包含前面查出来的知识点表的 chapter_id. 之前并不知道该如何去条件判断, 结果在实际实现的过程中尝试了一下, 发现居然可以实现, 十分欣喜.

实现过程
----

一步一步来

1.  首先是查询试卷中的所有试题 ID

```
SELECT QUESTIONS_id FROM bs_papers_questions_relations WHERE PAPERS_ID = 1054
```

得到如下结果 (由于记录很多, 所以只截取了一部分, 后面均是如此):  
![](https://img-blog.csdnimg.cn/20190322115100161.png)  
2. 再根据获取到的 QUESTION_ID 去查 bs_questions 表

```
SELECT CHAPTER_ID FROM bs_questions q, ( SELECT QUESTIONS_id FROM bs_papers_questions_relations WHERE PAPERS_ID = 1054 ) r WHERE q.ID = r.QUESTIONS_id
```

得到结果:  
![](https://img-blog.csdnimg.cn/2019032211520793.png)  
可以看到, 有多条多个知识点的记录  
3. 将这个查询结果按照前面说过的方法分割字符串, 顺便使用 DISTINCT 关键字去重

```
SELECT DISTINCT
		SUBSTRING_INDEX( SUBSTRING_INDEX( a.CHAPTER_ID, ',', help_topic_id + 1 ), ',',- 1 ) AS CHAPTER_ID 
	FROM
		mysql.help_topic m,
		( SELECT CHAPTER_ID FROM bs_questions q, ( SELECT QUESTIONS_id FROM bs_papers_questions_relations WHERE PAPERS_ID = 1054 ) r WHERE q.ID = r.QUESTIONS_id ) a 
	WHERE
		help_topic_id < LENGTH( a.CHAPTER_ID ) - LENGTH( REPLACE ( a.CHAPTER_ID, ',', '' ) ) + 1
```

得到查询结果:  
![](https://img-blog.csdnimg.cn/20190322115334346.png)  
很好看! 就是想要的结果有没有!!! 看见这个的时候感觉自己已经成功了一半了!  
BTW, 对于空字符串记录无需理会, 是在分割字符串时, 对于 ",3301" 这样的记录插上的, 关联查询之后会自动省略的, 所以我们不去管它

4.  将这个 CHAPTER_ID 的查询结果去再次关联查询试卷中的试题  
    这里就需要考虑如何根据条件分组了, 条件很简单, bs_questions 表中的记录的 CHAPTER_ID 字段是否包含我们查出来的 ID  
    语句是这样的

```
CHAPTER_ID LIKE concat( '%', chapter.CHAPTER_ID, '%' )
```

首先考虑的是能不能用 IF 函数, 明显不行, 后来想到, 干脆把这句话作为 GROUP BY 的条件, 试了一下居然真的可以  
于是此步骤实现的 SQL 如下

```
SELECT
	chapter.CHAPTER_ID AS CHAPTER_ID,
	COUNT( question.id ) AS COUNT 
FROM
	(
	SELECT DISTINCT
		SUBSTRING_INDEX( SUBSTRING_INDEX( a.CHAPTER_ID, ',', help_topic_id + 1 ), ',',- 1 ) AS CHAPTER_ID 
	FROM
		mysql.help_topic m,
		( SELECT CHAPTER_ID FROM bs_questions q, ( SELECT QUESTIONS_id FROM bs_papers_questions_relations WHERE PAPERS_ID = 1054 ) r WHERE q.ID = r.QUESTIONS_id ) a 
	WHERE
		help_topic_id < LENGTH( a.CHAPTER_ID ) - LENGTH( REPLACE ( a.CHAPTER_ID, ',', '' ) ) + 1 
	) chapter,
	(
	SELECT
		CHAPTER_ID,
		ID 
	FROM
		bs_questions q,
		( SELECT QUESTIONS_id FROM bs_papers_questions_relations WHERE PAPERS_ID = 1054 ) r 
	WHERE
		q.ID = r.QUESTIONS_id 
	) question 
WHERE
	 question.CHAPTER_ID LIKE concat( '%', chapter.CHAPTER_ID, '%' ) 
GROUP BY
	chapter.CHAPTER_ID,
	question.CHAPTER_ID LIKE concat( '%', chapter.CHAPTER_ID, '%' )
```

查询结果:  
![](https://img-blog.csdnimg.cn/20190322120051581.png)  
太开心了, 这不就是想要的结果吗?! 第一条记录不去管它, 马上就给他去掉!!!

5.  最后一步! 将知识点内联查询 bs_chapter 表, 得到知识点的名称, 这一步出来的就是我们的最终 SQL 语句了!

```
#按知识点统计试卷的题目
SELECT
	chapter.CHAPTER_ID AS CHAPTER_ID,
	c.CHAPTER_NAME AS CHAPTER_NAME,
	COUNT( question.id ) AS COUNT 
FROM
	(
	SELECT DISTINCT
		SUBSTRING_INDEX( SUBSTRING_INDEX( a.CHAPTER_ID, ',', help_topic_id + 1 ), ',',- 1 ) AS CHAPTER_ID 
	FROM
		mysql.help_topic m,
		( SELECT CHAPTER_ID FROM bs_questions q, ( SELECT QUESTIONS_id FROM bs_papers_questions_relations WHERE PAPERS_ID = 1054 ) r WHERE q.ID = r.QUESTIONS_id ) a 
	WHERE
		help_topic_id < LENGTH( a.CHAPTER_ID ) - LENGTH( REPLACE ( a.CHAPTER_ID, ',', '' ) ) + 1 
	) chapter,
	bs_chapter c,
	(
	SELECT
		CHAPTER_ID,
		ID 
	FROM
		bs_questions q,
		( SELECT QUESTIONS_id FROM bs_papers_questions_relations WHERE PAPERS_ID = 1054 ) r 
	WHERE
		q.ID = r.QUESTIONS_id 
	) question 
WHERE
	chapter.CHAPTER_ID = c.ID 
	AND question.CHAPTER_ID LIKE concat( '%', chapter.CHAPTER_ID, '%' ) 
GROUP BY
	chapter.CHAPTER_ID,
	question.CHAPTER_ID LIKE concat( '%', chapter.CHAPTER_ID, '%' )
```

查询结果:  
![](https://img-blog.csdnimg.cn/20190322120303361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA4Mjc1NDQ=,size_16,color_FFFFFF,t_70)  
我甚至手动统计了一下, 结果完全正确! 问题解决, 不用加班皆大欢喜!  
接下来写了一个查询类来接受这些数据, 返回给前端制作条形图饼状图就 OK 啦!!!

参考分析
----

**分割字符串**

```
SELECT 
    SUBSTRING_INDEX(SUBSTRING_INDEX('7654,7698,7782,7788',',',help_topic_id+1),',',-1) AS num 
FROM 
    mysql.help_topic 
WHERE 
    help_topic_id < LENGTH('7654,7698,7782,7788')-LENGTH(REPLACE('7654,7698,7782,7788',',',''))+1
```

对于分割字符串的这个写法的分析我放在最后的参考分析里面了  
使用 SUBSTRING_INDEX(str, delim, count) 这个函数  
参数含义, 第一个 str 表示待分割的字符串, 第二个 delim 表示分隔符, 这里是 ",", 第三个 count, 当 count 为正数，取第 n 个分隔符之前的所有字符； 当 count 为负数，取倒数第 n 个分隔符之后的所有字符

通过这句`help_topic_id < LENGTH('7654,7698,7782,7788')-LENGTH(REPLACE('7654,7698,7782,7788',',',''))+1`得到待分割的字符串中有几个分隔符, 以此作为 SUBSTRING_INDEX 的参数`SUBSTRING_INDEX('7654,7698,7782,7788',',',help_topic_id+1)`这一句得到当前分隔符左边的字符串, 外层的函数得到分隔符右侧的字符串, 最终就能得到一条单独的分割后的字符串.

参考资料
----

[https://blog.csdn.net/pjymyself/article/details/81668157](https://blog.csdn.net/pjymyself/article/details/81668157)