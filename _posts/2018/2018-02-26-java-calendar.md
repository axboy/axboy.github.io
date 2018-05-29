---
layout: post
title: java calendar对象踩坑记录
description: java calendar对象踩坑记录
categories: java
tags: java
---

### 问题

有这样一个需求，查询印尼（+7时区）当天的订单量，心想，很容易嘛，顺手就把如下代码提交上线了，先指定时区，时分秒置0。

```java
Calendar cal = Calendar.getInstance();
cal.setTimeZone(TimeZone.getTimeZone("Asia/Pontianak"));
cal.set(Calendar.HOUR_OF_DAY, 0);
cal.set(Calendar.MINUTE, 0);
cal.set(Calendar.SECOND, 0);
cal.set(Calendar.MILLISECOND, 0);
Long fromTime = cal.getTimeInMillis();
...
```

上线后发现在每天凌晨北京时间1点到8点之间（使用docker部署，默认utc时区），今天的订单量都累计到了昨天，到8点后恢复正常显示。咋看一眼，没发现什么bug。

初步分析，8点后正常显示，应该就是fromTime在凌晨期间计算出错了，刚好8小时时差。

### 解决

请看下图代码和运行结果，系统时间改为了北京时间2018年2月26日凌晨0点31分，此时印尼应该是2月25日23点31分，然后设置时分秒都为0，我们期望的结果是北京时间2月25日凌晨1点（即印尼0点），但结果如25行输出一样，是26日，这显然没按预想的走。

![ide-code](/images/2018/java-calendar/ide-code.png)

19~25行和上面提交的代码一样，27~34行在上面的基础上增加了第29行内容，即打印日期。
获取日期后再修改时间，结果就是我们想要的了，理论上get方法不应该对值有影响，这里api源码待进一步查看。
