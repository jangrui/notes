# bc

bc是unix/linux下的计算器，除了可以作为计算器来使用，还可以作为命令行计算工具使用。

## 将bc作为计算器来应用

```bash
jangrui@jangrui:~$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
1+1
2
3*3
9
quit
jangrui@jangrui:~$ 
```

## 命令行下使用

```bash
jangrui@jangrui:~$ echo 3+3|bc
6
jangrui@jangrui:~$ echo 3.3+5|bc
8.3
jangrui@jangrui:~$ echo 3*3.3|bc
9.9
jangrui@jangrui:~$ echo "scale=2;255/113"|bc 	#保留两位小数
2.25
jangrui@jangrui:~$ echo "scale=5;255/113"|bc 	#保留5为小数
2.25663
jangrui@jangrui:~$ echo "obase=2;192"|bc 	#转为2进制
11000000
jangrui@jangrui:~$ echo "obase=16;192"|bc 	#转为16进制
C0
jangrui@jangrui:~$ echo "obase=10;ibase=2;11000000" |bc 	#2进制转10进制 
192
jangrui@jangrui:~$ echo "obase=10;ibase=16;C0" |bc 	#16进制转10进制
192
jangrui@jangrui:~$ i=5
jangrui@jangrui:~$ i=`echo $i+6|bc`
jangrui@jangrui:~$ echo $i
11
jangrui@jangrui:~$ echo "10^10" |bc #计算10的10次方
10000000000
jangrui@jangrui:~$ echo "sqrt(100)" |bc #计算平方根
10
```

> scale 指定保留有效位

> obase=2 指定转换成2进制

> ibase=16 制定现在是16进制

> sqrt() 平方根