---
layout: post
title: "Java 8 Stream API 实战"
date: 2017-03-20 00:17:11 +0800
updated: 2017-03-20 00:17:11 +0800
comments: true
categories: Java 
tags: [Java8, stream api, map, reduce]
keywords: Java8, Java8特性, StreamAPI, MapReduce, 代码实例  
---
谈起Java Stream API，我们希望能够弄明白它究竟是什么，能够用来做什么，有什么优势，并能够应用于具体场景。分别简述如下。

## 不是什么
- Java Stream API 不是输入输出流，与java.io包里的InputStream和OutputStream概念完全不同
- 不是用于解析XML的XMLStream
- 不是Valve公司的[游戏平台](http://store.steampowered.com/)  
- 也不是集合元素，不是数据结构不保存数据

## 是什么
- 是Java 8 中引入的新特性，是对集合（Collection）对象功能的增强
- 是关于算法和计算的，更像一个高级版本的迭代器（Iterator）

<!--more-->  

## 能够用来做什么
用于对集合对象进行各种便利、高效的聚合操作，或大批量数据操作

## 有何优势
以往对于集合的聚合操作，需要使用Iterator遍历集合，代码繁冗；对于过滤和计算得到的中间结果，需要额外的空间进行存储  
Java Stream API解决了以上问题，遍历逻辑可以精简为一行，使得代码更加简洁易读。  
Java Stream提供串行和并行两种模式进行汇聚操作，能够充分利用多核处理器的优势，更方便写出高性能的并发程序且不易出错

## 典型应用场景  
[StreamAPIDemo](https://github.com/cwind001/CwindJavaLab/blob/8cc89129a5ef3d288b59e02e4924ba2d7a597838/AdvancedJava/src/main/java/com/cwind/java8/stream/StreamAPIDemo.java)  
定义商品对象并初始化购物清单如下：  

{% coderay lang:java %}
public class StreamAPIDemo {

    private List<Item> shoppingList;

    @Before
    public void setUp() throws Exception {
        shoppingList = Lists.newArrayList();
        shoppingList.add(new Item("iPhone 7", 7250L));
        shoppingList.add(new Item("Rolex Watch", 28888L));
        shoppingList.add(new Item("Electric Toothbrush", 388L));
        shoppingList.add(new Item("Kindle Paperwhite", 1098L));
        shoppingList.add(new Item("Coca Cola", 3L));
    }
    class Item {
        String itemName;
        long price;

        public Item(String itemName, long price) {
            this.itemName = itemName;
            this.price = price;
        }
    }
}
{% endcoderay %}  

### 集合遍历  

{% coderay lang:java %}
    // 打印购物清单
    @Test
    public void printShoppingList(){
        shoppingList.stream().forEach(System.out::println);
    }
{% endcoderay %}  
输出：  

```
Item{itemName='iPhone 7', price=7250}
Item{itemName='Rolex Watch', price=28888}
Item{itemName='Electric Toothbrush', price=388}
Item{itemName='Kindle Paperwhite', price=1098}
Item{itemName='Coca Cola', price=3}
```

### 数学统计  

{% coderay lang:java %}
    // 统计购物清单总数与总价
    @Test
    public void printTotalPrice() {
        long itemNum = shoppingList.stream().count();
        System.out.println("Sum of items in the shopping list: " + itemNum);

        long totalPrice = shoppingList.stream().collect(Collectors.summingLong(Item::getPrice));
        System.out.println("Total price: " + totalPrice);
    }
{% endcoderay %}  
输出：  

```
Sum of items in the shopping list: 5
Total price: 37627
```

### 过滤与排序  

{% coderay lang:java %}
    // 过滤价格小于1000的商品
    @Test
    public void filterItems(){
        shoppingList.stream().filter(p -> p.getPrice() >= 1000L).forEach(System.out::println);
    }

    // 按价格排序
    @Test
    public void sortItemsByPrice(){
        shoppingList.stream().sorted(Comparator.comparingLong(Item::getPrice)).forEach(System.out::println);
    }
{% endcoderay %}  

### MapReduce(映射与规约)  

{% coderay lang:java %}
    // 所有商品价格减100 - Map
    @Test
    public void streamMap(){
        shoppingList.stream().map(p -> new Item(p.getItemName(), p.getPrice()-100)).forEach(System.out::println);
    }
{% endcoderay %}  

{% coderay lang:java %}
    // 取出如上优惠之后金额超过1000元的商品中，价格最低的商品（最便宜的奢侈品）
    @Test
    public void streamReduce(){
        Item cheapestLuxury = shoppingList.stream().map(p -> new Item(p.getItemName(), p.getPrice()-100))
            .filter(p -> p.getPrice() > 1000).reduce((a, b) -> a.getPrice()<b.getPrice()?a:b).get();
        System.out.println(cheapestLuxury);
    }
{% endcoderay %}  

## 参考 
- [Java 8 中的Streams API 详解](http://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/) 
- [Java 8系列之Stream中万能的reduce](http://blog.csdn.net/io_field/article/details/54971679)