\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[blog.csdn.net\](https://blog.csdn.net/noaman\_wgs/article/details/80984873)

### 一、什么是 Quartz

什么是 Quartz?

> Quartz 是 OpenSymphony 开源组织在 Job scheduling 领域又一个开源项目，完全由 Java 开发，可以用来执行定时任务，类似于 java.util.Timer。但是相较于 Timer， Quartz 增加了很多功能：
> 
> *   持久性作业 - 就是保持调度定时的状态;
> *   作业管理 - 对调度作业进行有效的管理;

大部分公司都会用到定时任务这个功能。  
拿火车票购票来说，当你下单后，后台就会插入一条待支付的 task(job)，一般是 30 分钟，超过 30min 后就会执行这个 job，去判断你是否支付，未支付就会取消此次订单；当你支付完成之后，后台拿到支付回调后就会再插入一条待消费的 task（job），Job 触发日期为火车票上的出发日期，超过这个时间就会执行这个 job，判断是否使用等。

在我们实际的项目中，当 Job 过多的时候，肯定不能人工去操作，这时候就需要一个任务调度框架，帮我们自动去执行这些程序。那么该如何实现这个功能呢？

（1）首先我们需要定义实现一个定时功能的接口，我们可以称之为 Task（或 Job），如定时发送邮件的 task（Job），重启机器的 task（Job），优惠券到期发送短信提醒的 task（Job），实现接口如下：  
![](https://img-blog.csdn.net/20180710135410275?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（2）有了任务之后，还需要一个能够实现触发任务去执行的触发器，触发器 Trigger 最基本的功能是指定 Job 的执行时间，执行间隔，运行次数等。  
![](https://img-blog.csdn.net/20180710135421739?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（3）有了 Job 和 Trigger 后，怎么样将两者结合起来呢？即怎样指定 Trigger 去执行指定的 Job 呢？这时需要一个 Schedule，来负责这个功能的实现。  
![](https://img-blog.csdn.net/20180710135431806?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上面三个部分就是 Quartz 的基本组成部分：

*   调度器：Scheduler
*   任务：JobDetail
*   触发器：Trigger，包括 SimpleTrigger 和 CronTrigger

### 二、Quartz Demo 搭建

下面来利用 Quartz 搭建一个最基本的 Demo。  
1、导入依赖的 jar 包：

```
<!-- https://mvnrepository.com/artifact/org.quartz-scheduler/quartz -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.0</version>
</dependency>
```

2、新建一个能够打印任意内容的 Job：

```
/\*\*
 \* Created by wanggenshen
 \* Date: on 2018/7/7 16:28.
 \* Description: 打印任意内容
 \*/
public class PrintWordsJob implements Job{

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        String printTime = new SimpleDateFormat("yy-MM-dd HH-mm-ss").format(new Date());
        System.out.println("PrintWordsJob start at:" + printTime + ", prints: Hello Job-" + new Random().nextInt(100));

    }
}
```

3、创建 Schedule，执行任务：

```
/\*\*
 \* Created by wanggenshen
 \* Date: on 2018/7/7 16:31.
 \* Description: XXX
 \*/
public class MyScheduler {

    public static void main(String\[\] args) throws SchedulerException, InterruptedException {
        // 1、创建调度器Scheduler
        SchedulerFactory schedulerFactory = new StdSchedulerFactory();
        Scheduler scheduler = schedulerFactory.getScheduler();
        // 2、创建JobDetail实例，并与PrintWordsJob类绑定(Job执行内容)
        JobDetail jobDetail = JobBuilder.newJob(PrintWordsJob.class)
                                        .withIdentity("job1", "group1").build();
        // 3、构建Trigger实例,每隔1s执行一次
        Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "triggerGroup1")
                .startNow()//立即生效
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                .withIntervalInSeconds(1)//每隔1s执行一次
                .repeatForever()).build();//一直执行

        //4、执行
        scheduler.scheduleJob(jobDetail, trigger);
        System.out.println("--------scheduler start ! ------------");
        scheduler.start();

        //睡眠
        TimeUnit.MINUTES.sleep(1);
        scheduler.shutdown();
        System.out.println("--------scheduler shutdown ! ------------");


    }
}
```

运行程序，可以看到程序每隔 1s 会打印出内容，且在一分钟后结束：  
![](https://img-blog.csdn.net/20180710135458277?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 三、Quartz 核心详解

下面就程序中出现的几个参数，看一下 Quartz 框架中的几个重要参数：

*   Job 和 JobDetail
*   JobExecutionContext
*   JobDataMap
*   Trigger、SimpleTrigger、CronTrigger

（1）Job 和 JobDetail  
Job 是 Quartz 中的一个接口，接口下只有 execute 方法，在这个方法中编写业务逻辑。  
接口中的源码：  
![](https://img-blog.csdn.net/20180710135513678?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

JobDetail 用来绑定 Job，为 Job 实例提供许多属性：

*   name
*   group
*   jobClass
*   jobDataMap

JobDetail 绑定指定的 Job，每次 Scheduler 调度执行一个 Job 的时候，首先会拿到对应的 Job，然后创建该 Job 实例，再去执行 Job 中的 execute() 的内容，任务执行结束后，关联的 Job 对象实例会被释放，且会被 JVM GC 清除。

为什么设计成 JobDetail + Job，不直接使用 Job

> JobDetail 定义的是任务数据，而真正的执行逻辑是在 Job 中。  
> 这是因为任务是有可能并发执行，如果 Scheduler 直接使用 Job，就会存在对同一个 Job 实例并发访问的问题。而 JobDetail & Job 方式，Sheduler 每次执行，都会根据 JobDetail 创建一个新的 Job 实例，这样就可以规避并发访问的问题。

（2）JobExecutionContext  
JobExecutionContext 中包含了 Quartz 运行时的环境以及 Job 本身的详细数据信息。  
当 Schedule 调度执行一个 Job 的时候，就会将 JobExecutionContext 传递给该 Job 的 execute() 中，Job 就可以通过 JobExecutionContext 对象获取信息。  
主要信息有：  
![](https://img-blog.csdn.net/20180710135537517?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（3）JobExecutionContext  
JobDataMap 实现了 JDK 的 Map 接口，可以以 Key-Value 的形式存储数据。  
JobDetail、Trigger 都可以使用 JobDataMap 来设置一些参数或信息，  
Job 执行 execute() 方法的时候，JobExecutionContext 可以获取到 JobExecutionContext 中的信息：  
如：

```
JobDetail jobDetail = JobBuilder.newJob(PrintWordsJob.class)                        .usingJobData("jobDetail1", "这个Job用来测试的")
                  .withIdentity("job1", "group1").build();

 Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "triggerGroup1")
      .usingJobData("trigger1", "这是jobDetail1的trigger")
      .startNow()//立即生效
      .withSchedule(SimpleScheduleBuilder.simpleSchedule()
      .withIntervalInSeconds(1)//每隔1s执行一次
      .repeatForever()).build();//一直执行
```

Job 执行的时候，可以获取到这些参数信息：

```
@Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {

        System.out.println(jobExecutionContext.getJobDetail().getJobDataMap().get("jobDetail1"));
        System.out.println(jobExecutionContext.getTrigger().getJobDataMap().get("trigger1"));
        String printTime = new SimpleDateFormat("yy-MM-dd HH-mm-ss").format(new Date());
        System.out.println("PrintWordsJob start at:" + printTime + ", prints: Hello Job-" + new Random().nextInt(100));


    }
```

（4）Trigger、SimpleTrigger、CronTrigger

*   **Trigger**

Trigger 是 Quartz 的触发器，会去通知 Scheduler 何时去执行对应 Job。

```
new Trigger().startAt():表示触发器首次被触发的时间;
new Trigger().endAt():表示触发器结束触发的时间;
```

*   **SimpleTrigger**  
    SimpleTrigger 可以实现在一个指定时间段内执行一次作业任务或一个时间段内多次执行作业任务。  
    下面的程序就实现了程序运行 5s 后开始执行 Job，执行 Job 5s 后结束执行：

```
Date startDate = new Date();
startDate.setTime(startDate.getTime() + 5000);

 Date endDate = new Date();
 endDate.setTime(startDate.getTime() + 5000);

        Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "triggerGroup1")
                .usingJobData("trigger1", "这是jobDetail1的trigger")
                .startNow()//立即生效
                .startAt(startDate)
                .endAt(endDate)
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                .withIntervalInSeconds(1)//每隔1s执行一次
                .repeatForever()).build();//一直执行
```

*   **CronTrigger**

CronTrigger 功能非常强大，是基于日历的作业调度，而 SimpleTrigger 是精准指定间隔，所以相比 SimpleTrigger，CroTrigger 更加常用。CroTrigger 是基于 Cron 表达式的，先了解下 Cron 表达式：  
由 7 个子表达式组成字符串的，格式如下：

> \[秒\] \[分\] \[小时\] \[日\] \[月\] \[周\] \[年\]

Cron 表达式的语法比较复杂，  
如：\* 30 10 ? \* 1/5 \*  
表示（从后往前看）  
\[指定年份\] 的 \[ 周一到周五\]\[指定月\]\[不指定日\]\[上午 10 时\]\[30 分\]\[指定秒\]

又如：00 00 00 ？ \* 10,11,12 1#5 2018  
表示 2018 年 10、11、12 月的第一周的星期五这一天的 0 时 0 分 0 秒去执行任务。

下面是给的一个例子：  
![](https://img-blog.csdn.net/20180710135615747?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可通过在线生成 Cron 表达式的工具：[http://cron.qqe2.com/](http://cron.qqe2.com/) 来生成自己想要的表达式。  
![](https://img-blog.csdn.net/20180710135623995?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25vYW1hbl93Z3M=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

下面的代码就实现了每周一到周五上午 10:30 执行定时任务

```
/\*\*
 \* Created by wanggenshen
 \* Date: on 2018/7/7 20:06.
 \* Description: XXX
 \*/
public class MyScheduler2 {
    public static void main(String\[\] args) throws SchedulerException, InterruptedException {
        // 1、创建调度器Scheduler
        SchedulerFactory schedulerFactory = new StdSchedulerFactory();
        Scheduler scheduler = schedulerFactory.getScheduler();
        // 2、创建JobDetail实例，并与PrintWordsJob类绑定(Job执行内容)
        JobDetail jobDetail = JobBuilder.newJob(PrintWordsJob.class)
                .usingJobData("jobDetail1", "这个Job用来测试的")
                .withIdentity("job1", "group1").build();
        // 3、构建Trigger实例,每隔1s执行一次
        Date startDate = new Date();
        startDate.setTime(startDate.getTime() + 5000);

        Date endDate = new Date();
        endDate.setTime(startDate.getTime() + 5000);

        CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "triggerGroup1")
                .usingJobData("trigger1", "这是jobDetail1的trigger")
                .startNow()//立即生效
                .startAt(startDate)
                .endAt(endDate)
                .withSchedule(CronScheduleBuilder.cronSchedule("\* 30 10 ? \* 1/5 2018"))
                .build();

        //4、执行
        scheduler.scheduleJob(jobDetail, cronTrigger);
        System.out.println("--------scheduler start ! ------------");
        scheduler.start();
        System.out.println("--------scheduler shutdown ! ------------");

    }
}
```

* * *

2018/07/07 20:10 in SH.