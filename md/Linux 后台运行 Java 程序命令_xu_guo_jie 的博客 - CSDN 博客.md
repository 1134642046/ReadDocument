> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/xu_guo_jie/article/details/82624376)

转自：[https://blog.csdn.net/weixin_41574643/article/details/79716052](https://blog.csdn.net/weixin_41574643/article/details/79716052)

方式一：java -jar shareniu.jar

特点：当前 ssh 窗口被锁定，可按 CTRL + C 打断程序运行，或直接关闭窗口，程序退出

那如何让窗口不锁定？

方式二：java -jar shareniu.jar &

& 代表在后台运行。

特定：当前 ssh 窗口不被锁定，但是当窗口关闭时，程序中止运行。

继续改进，如何让窗口关闭时，程序仍然运行？

方式三：nohup java -jar shareniu.jar &

nohup 意思是不挂断运行命令, 当账户退出或终端关闭时, 程序仍然运行

当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到 nohup.out 的文件中，除非另外指定了输出文件。

方式四：nohup java -jar shareniu.jar >temp.txt &

解释下 >temp.txt

command >out.file

command >out.file 是将 command 的输出重定向到 out.file 文件，即输出内容不打印到屏幕上，而是输出到 out.file 文件中。

可通过 jobs 命令查看后台运行任务

jobs

那么就会列出所有后台执行的作业，并且每个作业前面都有个编号。  
如果想将某个作业调回前台控制，只需要 fg + 编号即可。

fg 23

查看某端口占用的线程的 pid

netstat -nlp |grep :9181