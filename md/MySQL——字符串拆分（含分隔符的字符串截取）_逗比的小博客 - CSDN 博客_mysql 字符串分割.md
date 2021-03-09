> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/pjymyself/article/details/81668157)

有分隔符的字符串拆分
----------

#### 题目要求

数据库中 num 字段值为：  
![](https://img-blog.csdn.net/20180814150220508?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

实现的效果：需要将一行数据变成多行  
![](https://img-blog.csdn.net/20180814150316890?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### 实现的 SQL

```
SELECT 
    SUBSTRING_INDEX(SUBSTRING_INDEX('7654,7698,7782,7788',',',help_topic_id+1),',',-1) AS num 
FROM 
    mysql.help_topic 
WHERE 
    help_topic_id < LENGTH('7654,7698,7782,7788')-LENGTH(REPLACE('7654,7698,7782,7788',',',''))+1
```

#### 涉及的知识点

**一、字符串拆分： SUBSTRING_INDEX（str, delim, count）**

1.  参数解说

<table><thead><tr><th>参数名</th><th align="center">解释</th></tr></thead><tbody><tr><td>str</td><td align="center">需要拆分的字符串</td></tr><tr><td>delim</td><td align="center">分隔符，通过某字符进行拆分</td></tr><tr><td>count</td><td align="center">当 count 为正数，取第 n 个分隔符之前的所有字符； 当 count 为负数，取倒数第 n 个分隔符之后的所有字符。</td></tr></tbody></table>

2. 举例  
（1）获取第 2 个以 “，” 逗号为分隔符之前的所有字符。

```
SUBSTRING_INDEX('7654,7698,7782,7788',',',2)
```

![](https://img-blog.csdn.net/20180814151309820?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（2）获取倒数第 2 个以 “，” 逗号分隔符之后的所有字符

```
SUBSTRING_INDEX('7654,7698,7782,7788',',',-2)
```

![](https://img-blog.csdn.net/20180814151412185?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**二、替换函数：replace(str, from_str, to_str)**

1.  参数解说

<table><thead><tr><th>参数名</th><th align="center">解释</th></tr></thead><tbody><tr><td>str</td><td align="center">需要进行替换的字符串</td></tr><tr><td>from_str</td><td align="center">需要被替换的字符串</td></tr><tr><td>to_str</td><td align="center">需要替换的字符串</td></tr></tbody></table>

2. 举例  
（1）将分隔符 “，” 逗号替换为 “” 空。

```
REPLACE('7654,7698,7782,7788',',','')
```

![](https://img-blog.csdn.net/20180814151757552?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**三、获取字符串长度：LENGTH(str)**

1.  参数解说

<table><thead><tr><th>参数名</th><th align="center">解释</th></tr></thead><tbody><tr><td>str</td><td align="center">需要计算长度的字符串</td></tr></tbody></table>

2. 举例  
（1）获取 ‘7654,7698,7782,7788’ 字符串的长度

```
LENGTH('7654,7698,7782,7788')
```

![](https://img-blog.csdn.net/20180814151922796?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 实现的 SQL 解析

```
SELECT 
    SUBSTRING_INDEX(SUBSTRING_INDEX('7654,7698,7782,7788',',',help_topic_id+1),',',-1) AS num 
FROM 
    mysql.help_topic 
WHERE 
    help_topic_id < LENGTH('7654,7698,7782,7788')-LENGTH(REPLACE('7654,7698,7782,7788',',',''))+1
```

此处利用 mysql 库的 help_topic 表的 help_topic_id 来作为变量，因为 help_topic_id 是自增的，当然也可以用其他表的自增字段辅助。

**help_topic 表：**  
![](https://img-blog.csdn.net/20180814152141964?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### 实现步骤：

**Step1：**首先获取最后需被拆分成多少个字符串，利用 help_topic_id 来模拟遍历 第 n 个字符串。

涉及的代码片段：

```
help_topic_id < LENGTH('7654,7698,7782,7788')-LENGTH(REPLACE('7654,7698,7782,7788',',',''))+1
```

![](https://img-blog.csdn.net/20180814152417186?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**Step2：**根据 “，” 逗号来拆分字符串，此处利用 SUBSTRING_INDEX（str, delim, count） 函数，最后把结果赋值给 num 字段。

涉及的代码片段：

```
SUBSTRING_INDEX(SUBSTRING_INDEX('7654,7698,7782,7788',',',help_topic_id+1),',',-1) AS num
```

##### 第一步：

以”,” 逗号为分隔符，根据 help_topic_id 的值来截取第 n+1 个分隔符之前所有的字符串。 （此处 n+1 是因为 help_topic_id 是从 0 开始算起，而此处需从第 1 个分隔符开始获取。）

```
SUBSTRING_INDEX('7654,7698,7782,7788',',',help_topic_id+1)
```

eg：  
当 help_topic_id = 0 时，获取到的字符串 = 7654  
当 help_topic_id = 1 时，获取到的字符串 = 7654,7698  
…（以此类推）

##### 第二步：

以”,” 逗号为分隔符，截取倒数第 1 个分隔符之后的所有字符串。

```
SUBSTRING_INDEX(SUBSTRING_INDEX('7654,7698,7782,7788',',',help_topic_id+1),',',-1)
```

eg：  
根据第一步，当 help_topic_id = 0 时，获取到的字符串 = 7654，此时第二步截取的字符串 = 7654  
根据第一步，当 help_topic_id = 1 时，获取到的字符串 = 7654,7698，此时第二步截取的字符串 = 7698  
…（以此类推）

#### 最终成功实现了以下效果 ~

![](https://img-blog.csdn.net/2018081415304427?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqeW15c2VsZg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

注：不含分隔符的字符串拆分可参考 [MySQL——字符串拆分（无分隔符的字符串截取）](https://blog.csdn.net/pjymyself/article/details/81669928)

如果以上有错误的地方，希望大家能够指正 ~ 谢谢 ~  
如果你有更好的方法，那就赶紧留言分享噢 ~ 谢谢 ~