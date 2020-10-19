\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[c.lanmit.com\](https://c.lanmit.com/bianchengkaifa/Java/59964.html)

本文将为您描述在 java 中进行日期时间比较的 4 种方法, 教程操作步骤: 1. Date.compareTo（）  
java.util.Date 提供了在 Java 中比较两个日期的经典方法 compareTo（）。

如果两个日期相等，则

本文将为您描述在 java 中进行日期时间比较的 4 种方法, 教程操作步骤:

1\. Date.compareTo（）

`java.util.Date`提供了在 Java 中比较两个日期的经典方法 compareTo（）。

如果两个日期相等，则返回值为 0。 如果 Date 在 date 参数之后，则返回值大于 0。 如果 Date 在 date 参数之前，则返回值小于 0。

```
@Test
void testDateCompare() throws ParseException {
  SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
  Date date1 = sdf.parse("2009-12-31");
  Date date2 = sdf.parse("2019-01-31");

  System.out.println("date1 : " + sdf.format(date1));
  System.out.println("date2 : " + sdf.format(date2));

  if (date1.compareTo(date2) > 0) {
    System.out.println("Date1 时间在 Date2 之后");
  } else if (date1.compareTo(date2) < 0) {
    System.out.println("Date1 时间在 Date2 之前");
  } else if (date1.compareTo(date2) == 0) {
    System.out.println("Date1 时间与 Date2 相等");
  } else {
    System.out.println("程序怎么会运行到这里?正常应该不会");
  }
}
```

输出结果：

```
date1 : 2009-12-31
date2 : 2019-01-31
Date1 时间在 Date2 之前
```

2\. Date.before（），Date.after（）和 Date.equals（）

一种语义上比较友好的方法来比较两个`java.util.Date`

```
@Test
void testDateCompare2() throws ParseException {
  SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
  Date date1 = sdf.parse("2009-12-31");
  Date date2 = sdf.parse("2019-01-31");

  System.out.println("date1 : " + sdf.format(date1));
  System.out.println("date2 : " + sdf.format(date2));

  if (date1.after(date2)) {
    System.out.println("Date1 时间在 Date2 之后");
  }

  if (date1.before(date2)) {
    System.out.println("Date1 时间在 Date2 之前");
  }

  if (date1.equals(date2)) {
    System.out.println("Date1 时间与 Date2 相等");
  }
}
```

输出结果

```
date1 : 2009-12-31
date2 : 2019-01-31
Date1 时间在 Date2 之前
```

3\. Calender.before（），Calender.after（）和 Calender.equals（）

使用`java.util.Calendar`比较两个 Date 日期

```
@Test
void testDateCompare3() throws ParseException {
  SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
  Date date1 = sdf.parse("2009-12-31");
  Date date2 = sdf.parse("2019-01-31");

  System.out.println("date1 : " + sdf.format(date1));
  System.out.println("date2 : " + sdf.format(date2));

  Calendar cal1 = Calendar.getInstance();
  Calendar cal2 = Calendar.getInstance();
  cal1.setTime(date1);
  cal2.setTime(date2);

  if (cal1.after(cal2)) {
    System.out.println("Date1 时间在 Date2 之后");
  }

  if (cal1.before(cal2)) {
    System.out.println("Date1 时间在 Date2 之前");
  }

  if (cal1.equals(cal2)) {
    System.out.println("Date1 时间与 Date2 相等");
  }
}
```

输出结果：

```
date1 : 2009-12-31
date2 : 2019-01-31
Date1 时间在 Date2 之前
```

4\. Java 8 日期比较方法

在 Java 8 中，可以使用新的 isBefore（），isAfter（），isEqual（）和 compareTo（）来比较 LocalDate，LocalTime 和 LocalDateTime。以下示例以比较两个`java.time.LocalDate`

```
@Test
void testDateCompare4() throws ParseException {
  DateTimeFormatter sdf = DateTimeFormatter.ofPattern("yyyy-MM-dd");
  LocalDate date1 = LocalDate.of(2009, 12, 31);
  LocalDate date2 = LocalDate.of(2019, 1, 31);

  System.out.println("date1 : " + sdf.format(date1));
  System.out.println("date2 : " + sdf.format(date2));

  System.out.println("Is...");
  if (date1.isAfter(date2)) {
    System.out.println("Date1 时间在 Date2 之后");
  }

  if (date1.isBefore(date2)) {
    System.out.println("Date1 时间在 Date2 之前");
  }

  if (date1.isEqual(date2)) {
    System.out.println("Date1 时间与 Date2 相等");
  }
}
```

输出结果

```
date1 : 2009-12-31
date2 : 2019-01-31
Is...
Date1 时间在 Date2 之前
```

欢迎关注我的博客，里面有很多精品合集 本文转载注明出处（必须带连接，不能只转文字）：字母哥博客。

觉得对您有帮助的话，帮我点赞、分享！您的支持是我不竭的创作动力！ 。另外，笔者最近一段时间输出了如下的精品内容，期待您的关注。

《手摸手教你学 Spring Boot2.0》 《Spring Security-JWT-OAuth2 一本通》 《实战前后端分离 RBAC 权限管理系统》 《实战 SpringCloud 微服务从青铜到王者》 《VUE 深入浅出系列》在 java 中进行日期时间比较的 4 种方法就为您介绍到这里，感谢您关注懒咪学编程 c.lanmit.com.

本文地址：https://c.lanmit.com/bianchengkaifa/Java/59964.html