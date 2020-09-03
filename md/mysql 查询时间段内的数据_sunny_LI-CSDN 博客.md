\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.csdn.net\](https://blog.csdn.net/u013028876/article/details/79118464)

        参照文章（ [mysql 查询时间段内数据](http://blog.csdn.net/bj262948/article/details/79087472)）进行了操作。

        先来建表语句：

```
SET FOREIGN\_KEY\_CHECKS=0;
 
-- ----------------------------
-- Table structure for t\_user
-- ----------------------------
DROP TABLE IF EXISTS \`t\_user\`;
CREATE TABLE \`t\_user\` (
  \`userId\` bigint(20) NOT NULL,
  \`fullName\` varchar(64) NOT NULL,
  \`userType\` varchar(16) NOT NULL,
  \`addedTime\` datetime NOT NULL,
  PRIMARY KEY (\`userId\`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
-- ----------------------------
-- Records of t\_user
-- ----------------------------
INSERT INTO \`t\_user\` VALUES ('1', '爽爽', '普通', '2018-01-21 10:20:09');
INSERT INTO \`t\_user\` VALUES ('2', '贵贵', '普通', '2017-11-06 10:20:22');
INSERT INTO \`t\_user\` VALUES ('3', '芬芬', 'vip', '2017-11-13 10:20:42');
INSERT INTO \`t\_user\` VALUES ('4', '思思', 'vip', '2018-01-21 10:20:55');
INSERT INTO \`t\_user\` VALUES ('5', '妍妍', 'vip', '2017-09-17 10:21:28');
```

  
        下面是 sql 语句：

```
\-- 今天  
select fullName,addedTime from t\_user where to\_days(addedTime) <= to\_days(now()); 
-- 昨天  
select fullName,addedTime from t\_user where to\_days(NOW()) - TO\_DAYS(addedTime) <= 1;  
-- 近7天  
select fullName,addedTime from t\_user where date\_sub(CURDATE(),INTERVAL 7 DAY) <= DATE(addedTime);  
-- 近30天  
SELECT fullName,addedTime FROM t\_user where DATE\_SUB(CURDATE(), INTERVAL 30 DAY) <= date(addedTime);
-- 本月  
SELECT fullName,addedTime FROM t\_user WHERE DATE\_FORMAT( addedTime, '%Y%m' ) = DATE\_FORMAT( CURDATE() , '%Y%m' );
-- 上一月  
SELECT fullName,addedTime FROM t\_user WHERE PERIOD\_DIFF( date\_format( now( ) , '%Y%m' ) , date\_format( addedTime, '%Y%m' ) ) =1; 
-- 查询本季度数据  
select fullName,addedTime FROM t\_user where QUARTER(addedTime)=QUARTER(now()); 
-- 查询上季度数据  
select fullName,addedTime FROM t\_user where QUARTER(addedTime)=QUARTER(DATE\_SUB(now(),interval 1 QUARTER));  
-- 查询本年数据  
select fullName,addedTime FROM t\_user where YEAR(addedTime)=YEAR(NOW());  
-- 查询上年数据  
select fullName,addedTime FROM t\_user where year(addedTime)=year(date\_sub(now(),interval 1 year));  
-- 查询距离当前现在6个月的数据  
select fullName,addedTime FROM t\_user where addedTime between date\_sub(now(),interval 6 month) and now();  
 
-- 查询当前这周的数据  
SELECT fullName,addedTime FROM t\_user WHERE YEARWEEK(date\_format(addedTime,'%Y-%m-%d')) = YEARWEEK(now());  
-- 查询上周的数据  
SELECT fullName,addedTime FROM t\_user WHERE YEARWEEK(date\_format(addedTime,'%Y-%m-%d')) = YEARWEEK(now())-1;  
-- 查询上个月的数据   
select fullName,addedTime FROM t\_user where date\_format(addedTime,'%Y-%m')=date\_format(DATE\_SUB(curdate(), INTERVAL 1 MONTH),'%Y-%m'); 
-- 查询当前月份的数据
select fullName,addedTime FROM t\_user where DATE\_FORMAT(addedTime,'%Y%m') = DATE\_FORMAT(CURDATE(),'%Y%m');
select fullName,addedTime FROM t\_user where date\_format(addedTime,'%Y-%m')=date\_format(now(),'%Y-%m'); 
 
-- 查询指定时间段的数据
select fullName,addedTime FROM t\_user where addedTime between  '2017-1-1 00:00:00'  and '2018-1-1 00:00:00';   
select fullName,addedTime FROM t\_user where addedTime >='2017-1-1 00:00:00'  and addedTime < '2018-1-1 00:00:00';
```

  
        归纳一下：

        1、查询时间段内的数据，一般可以用 between and 或 <> 来指定时间段。

        2、mysql 的时间字段类型有：datetime，timestamp，date，time，year。

        ![](https://img-blog.csdn.net/20180121103856515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHMxNjQ1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

       3、 获取系统当前时间的函数：

        select CURDATE();  
        select NOW();

        4、获取时间差的函数：

        period\_diff()    datediff(date1,date2)      timediff(time1,time2)  

       5、日期加减函数：

        date\_sub() 

        date\_add()     adddate()      addtime()

        period\_add(P,N)    

        -------- 以上参考文章（**[mysql 日期加减](https://www.2cto.com/database/201110/107788.html)**）  

        6、时间格式转化函数：

        date\_format(date, format) ，MySQL 日期格式化函数 date\_format()  
        unix\_timestamp()   
        str\_to\_date(str, format)   
        from\_unixtime(unix\_timestamp, format) ，MySQL 时间戳格式化函数 from\_unixtime

        -------- 以上参考文章（[MYSQL 日期 字符串 时间戳互转](http://www.cnblogs.com/jhy-ocean/p/5560857.html)）

        顺带写一下 oracle 的查询语句：

```
select \* from Oracle.alarmLog where alarmtime between to\_date('2007-03-03 18:00:00','yyyy-mm-dd hh24:mi:ss') and to\_date('2007-09-04 18:00:00','yyyy-mm-dd hh24:mi:ss')
```

总结：

        时间就像海绵里的水，只要愿挤，总是有的。比如总结 mysql 中时间段查询方式的时间。o(\*￣︶￣\*)o  

        用现在的话说，做好时间管理，高效的学习及工作。