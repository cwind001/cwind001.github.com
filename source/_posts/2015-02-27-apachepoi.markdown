---
layout: post
title: "Java读写Excel：Apache POI, JXL与OpenCSV"
date: 2015-02-27 07:53:53 +0800
updated: 2015-02-27 07:53:53 +0800
comments: true
categories: Java
tags: [Java, ApachePOI, JXL, OpenCSV]
keywords: Java读写CSV, ApachePOI, JXL, OpenCSV   
---
前些日子把JXL替换为ApachePOI，原因很简单，JXL在2009年10月已经停止更新，并且不支持Excel 2007 OOXML （.xlsx）格式的文件。事实上把JXL与POI进行比较并不公平，因为JXL只能够操作OLE2格式的Excel（即.xls），而POI则是能够读写xls(x)、doc(x)、ppt(x)的一整套解决方案。  
<!--more-->

不同版本Excel的行列数限制：  

{% coderay lang:java linenos:false %}   
+-----------------+-----------+--------------+---------------------+
|                 | Max. Rows | Max. Columns | Max. Cols by letter |
+-----------------+-----------+--------------+---------------------+
| Excel 365*      | 1,048,576 | 16,384       | XFD                 |
| Excel 2013      | 1,048,576 | 16,384       | XFD                 |
| Excel 2010      | 1,048,576 | 16,384       | XFD                 |
| Excel 2007      | 1,048,576 | 16,384       | XFD                 |
| Excel 2003      | 65,536    | 256          | IV                  |
| Excel 2002 (XP) | 65,536    | 256          | IV                  |
| Excel 2000      | 65,536    | 256          | IV                  |
| Excel 97        | 65,536    | 256          | IV                  |
| Excel 95        | 16,384    | 256          | IV                  |
| Excel 5         | 16,384    | 256          | IV                  |
+-----------------+-----------+--------------+---------------------+  
{% endcoderay %} 

*\*Excel 365 unverified.*

**JXL - JExcelApi**  
[Maven Repo](http://mvnrepository.com/artifact/net.sourceforge.jexcelapi/jxl/2.6.12)  
[官方网站](http://www.andykhan.com/jexcelapi/index.html)  
最后更新：Oct 24，2009

{% coderay lang:java linenos:false %}   
<dependency>
 <groupId>net.sourceforge.jexcelapi</groupId>
 <artifactId>jxl</artifactId>
 <version>2.6.12</version>
</dependency>
{% endcoderay %}   

JXL是一个日本人写的简单类库。[作者主页](http://www.jexcelapi.org/)。[POI和jxl.jar性能比较](http://blog.csdn.net/jarvis_java/article/details/4924099)一贴中提到其性能较poi更高，内存消耗更少。当且仅当目标文档是行数接近但不超过65536的xls格式时成立。  

类图：  
{% img http://dl.iteye.com/upload/picture/pic/132574/73b48deb-3ba5-396c-b01c-5546b1aecba0.jpg %}  

{% coderay lang:java linenos:true JXL Demo https://github.com/cwind001/CwindJavaLab/blob/master/POITest/src/main/java/com/cwind/jxl/JXLDataSheetWriter.java %}   
 public static void main(String[] args) {
  try {
   // create writable wookbook
   WritableWorkbook workbook 
	= Workbook.createWorkbook(new File("jxlOutput.xls"));
   
   // create writable sheet
   WritableSheet sheet = workbook.createSheet("First Sheet", 0);
   for(int i = 0; i < data.length; i++) {
    for(int j = 0; j < data[i].length; j++){
     
     // create a cell at position (i, j) and add to the sheet
     Label label = new Label(i, j, data[i][j]);
     sheet.addCell(label);
    }
   }
   workbook.write();
   workbook.close();
  } catch (IOException | WriteException e) {
   e.printStackTrace();
  }
 }
{% endcoderay %} 

**Apache POI**  
[Maven Repo](http://mvnrepository.com/artifact/org.apache.poi/poi)  
[官方网站](http://poi.apache.org/)  
最后更新：Dec 17，2014  

类图：  
{% img http://dl.iteye.com/upload/picture/pic/132576/6230920a-edc2-3e7c-ac23-d4590f095048.jpg %} 

{% coderay lang:java linenos:false %}   
<dependency> 
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId>
  <version>3.10.1</version>
</dependency> 
 
 <dependency>
     <groupId>org.apache.poi</groupId>
     <artifactId>poi-ooxml</artifactId>
     <version>3.9</version>
 </dependency>
{% endcoderay %}   

**Apache POI API的一些重点：**
  
- Apache POI包含 Excel 97(-2007)文件格式(.xls)的Java实现 -- HSSF。（彩蛋：H指Horrible）  
- Apache POI包含 Excel 2007 OOXML文件格式(.xlsx)的Java实现 -- XSSF。  
- Apache POI的HSSF和XSSF API提供了读写和修改Excel电子表格的功能。  
- Apache POI也提供了SXSSF API（流式XSSF），它是XSSF的扩展，用于写入非常大的excel文件。SXSSF API需求较小的内存，适用于在堆内存受限时处理较大excel文件的情况。  
- 可以选择两种模型：Event Model和User Model。Event Model需求较小的内存，流式读取并处理每个单元；User Model更具备面向对象的特征，方便操作。  
- Apache POI提供了对excel附加功能的完美支持，如公式、单元格样式、颜色、字体、数据验证、图像和超链接等。  

SpreadSheet API 功能摘要：  
{% img http://dl.iteye.com/upload/picture/pic/132578/9b044f00-622e-3a07-9471-3ee912e42819.jpg %}   
以下是两个基于XSSF读写xlsx文件的例子：  
[读取xlsx文件](https://github.com/cwind001/CwindJavaLab/blob/master/POITest/src/main/java/com/cwind/poi/SimpleDatasheetReader.java)  
[写入xlsx文件](https://github.com/cwind001/CwindJavaLab/blob/master/POITest/src/main/java/com/cwind/poi/SimpleDatasheetWriter.java)  

**OpenCSV：**   
CSV文件以纯文本形式存储表格数据（数字和文本）。OpenCSV是一个用于读写CSV文件的简单Java类库。
[Maven Repo](http://mvnrepository.com/artifact/net.sf.opencsv/opencsv/2.3)
[官方网站](http://opencsv.sf.net)
最近更新：Jul 28，2011

{% coderay lang:java linenos:false %}  
<dependency>
 <groupId>net.sf.opencsv</groupId>
 <artifactId>opencsv</artifactId>
 <version>2.3</version>
</dependency>
{% endcoderay %}

OpenCSV将CSV文件中的每一行读取为一个String数组。相应地，写文件时通过`csvWriter.writeNext(array)`把String数组内容作为一行写入CSV文件

读写CSV文件的例子：  
[读取csv文件内容](https://github.com/cwind001/CwindJavaLab/blob/master/POITest/src/main/java/com/cwind/opencsv/ReadCSVDemo.java)  
[将xlsx文件内容写入csv](https://github.com/cwind001/CwindJavaLab/blob/master/POITest/src/main/java/com/cwind/opencsv/OpenCSVDemo.java)  

**References:**  
1. [POI-HSSF and POI-XSSF - Java API To Access Microsoft Excel Format Files](http://poi.apache.org/spreadsheet/)  
2. [Java Read/Write Excel File using Apache POI API](http://www.journaldev.com/2562/java-readwrite-excel-file-using-apache-poi-api)  