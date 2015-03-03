---
layout: post
title: "Java时区转换与时间格式"
date: 2013-07-17 21:26:56 +0800
updated: 2013-07-17 21:26:56 +0800
comments: true
categories: Java
tags: [Java, Date, Calendar, TimeZone, DateFormat]
keywords: Java时区, 时间格式, Calendar, TimeZone, DateFormat
---
本文介绍Java API 中 Date, Calendar, TimeZone和DateFormat的使用，以及不同时区时间相互转化的方法和原理。
<!--more-->
 
**问题描述：**
向处于不同时区的服务器发请求时需要考虑时区转换的问题。譬如，服务器位于东八区（北京时间，GMT+8:00），而身处东四区的用户想要查询当天的销售记录。则需把东四区的“今天”这个时间范围转换为服务器所在时区的时间范围。
 
*Tips1. GMT时间：*即格林威治平时（Greenwich Mean Time）。平太阳时是与视太阳时对应的，由于地球轨道非圆形，运行速度岁地球与太阳距离改变而出现变化，因此视太阳时欠缺均匀性。为了纠正这种不均匀性，天文学家计算地球非圆形轨迹与极轴倾斜对视太阳时的效应。平太阳时就是指经修订之后的视太阳时。在格林威治子午线上的平太阳时称为世界时（UTC），又叫格林威治平时（GMT）。
 
类Date表示特定的瞬间，精确到毫秒。获得一个表示当前时间的Date对象有两种方式：   
 
{% coderay lang:java linenos:false %}   
Date date = new Date();  
Date date = Calendar.getInstance().getTime();  
{% endcoderay %} 

**Date**对象本身所存储的毫秒数可以通过date.getTime()方法得到；该函数返回自1970年1月1日 00:00:00 GMT以来此对象表示的毫秒数。
 
**Calendar**的getInstance()方法有参数为TimeZone和Locale的重载，可以使用指定时区和语言环境获得一个日历。无参则使用默认时区和语言环境获得日历。
 
**TimeZone**表示时区偏移量，本质上以毫秒数保存与GMT的差值。获取TimeZone可以通过时区ID，如"America/New_York"，也可以通过GMT+/-hh:mm来设定。例如北京时间可以表示为GMT+8:00。
TimeZone.getRawOffset()方法可以用来得到当前时区的标准时间到GMT的偏移量。上段提到的"America/New_York"和"GMT+8:00"两个时区的偏移量分别为-18000000和28800000。
 
于是问题就简单了，在时区间转换时间时，首先用原时间减掉原时间所在时区相对于GMT的偏移量，得到原时间相对与GMT的值，再加上目标时区相对GMT的偏移量即可。
这样得到的结果依然是毫秒数，需要按照指定日期格式重新转换成Date对象。

{% coderay lang:java linenos:true %}   
import java.text.*;    
import java.util.*;    
  
public class DateTransformer  
{  
    public static final String DATE_FORMAT = "MM/dd/yyyy HH:mm:ss";  
         
    public static String dateTransformBetweenTimeZone(Date sourceDate, 
	DateFormat formatter,  
        TimeZone sourceTimeZone, TimeZone targetTimeZone) {  
        Long targetTime = sourceDate.getTime()
 		- sourceTimeZone.getRawOffset() + targetTimeZone.getRawOffset();  
        return DateTransformer.getTime(new Date(targetTime), formatter);  
    }  
         
    public static String getTime(Date date, DateFormat formatter){  
       return formatter.format(date);  
    }  
         
    public static void main(String[] args){  
        DateFormat formatter = new SimpleDateFormat(DATE_FORMAT);  
        Date date = Calendar.getInstance().getTime();  
        TimeZone srcTimeZone = TimeZone.getTimeZone("EST");  
        TimeZone destTimeZone = TimeZone.getTimeZone("GMT+8");  
        System.out.println(DateTransformer.dateTransformBetweenTimeZone(
	date, formatter, srcTimeZone, destTimeZone));  
    }  
}  
{% endcoderay %}  
*Tips2. 字面大数字赋值给long类型变量的问题*
上面函数中的targetTime是计算得来的，测试用例中我们可能需要通过毫秒数来构建几个日期对象，但是在赋值long time = 1374004799999 时会提示错误“The literal1374004799999 of type int is out of range”。代码中的数字字面值是int类型，因此超出了长度。在大数字后面加个'L'，long time = 1374004799999L即可正确赋值。 
 
DateFormat是是日期/时间格式化子类的抽象类，它以与语言无关的方式格式化并解析日期或时间。日期/时间格式化子类（如 SimpleDateFormat）允许进行格式化（也就是日期 -> 文本）、解析（文本-> 日期）和标准化。将日期表示为 Date 对象，或者表示为从 GMT（格林尼治标准时间）1970 年 1 月 1 日 00:00:00 这一刻开始的毫秒数。SimpleDateFormat则是一个以与语言环境有关的方式来格式化和解析日期的具体类，可以以“日期和时间模式”字符串指定日期和时间格式。我们函数中所用模式字符串为"MM/dd/yyyy HH:mm:ss"，则输出日期："07/16/2013 04:00:00"
 
其他常见的模式字母定义如下：  

{% coderay lang:java linenos:false %}   
字母 日期或时间元素 表示 示例
G	Era 标志符	Text	AD
y	年	Year	1996; 96
M	年中的月份	Month	July; Jul; 07
w	年中的周数	Number	27
W	月份中的周数	Number	2
D	年中的天数	Number	189
d	月份中的天数	Number	10
F	月份中的星期	Number	2
E	星期中的天数	Text	Tuesday; Tue
a	Am/pm 标记	Text	PM
H	一天中的小时数（0-23）	Number	0
k	一天中的小时数（1-24）	Number	24
K	am/pm 中的小时数（0-11）	Number	0
h	am/pm 中的小时数（1-12）	Number	12
m	小时中的分钟数	Number	30
s	分钟中的秒数	Number	55
S	毫秒数	Number	978
z	时区	General time zone	Pacific Standard Time; PST; GMT-08:00
Z	时区	RFC 822 time zone	-0800
{% endcoderay %} 

**References:**  
1. [Java API 1.6](http://www.javaweb.cc/help/JavaAPI1.6/)  
2. [Java时区的转换](http://www.blogjava.net/liudawei/articles/387891.html)  
3. [java时区-DateFormat和TimeZone关系](http://www.cnblogs.com/mailingfeng/archive/2012/06/20/2556326.html)  
4. [java获取当前时区的时间](http://blog.sina.com.cn/s/blog_4c44d3110100w0gn.html)  