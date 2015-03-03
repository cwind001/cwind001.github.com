---
layout: post
title: "从Introspector谈Java内省机制"
date: 2014-01-20 22:00:51 +0800
updated: 2014-01-20 22:00:51 +0800
comments: true
categories: Java
tags: [Java, Introspector, Reflection]
keywords: Java内省, Java反射, Introspector, Reflection
---
**内省**  
内省是Java语言的一种重要特性。使用内省我们可以在运行时得到一个类的内部信息。这些信息包括方法、属性、构造函数及其他。内省的一个应用是开发使用插件的应用程序。应用程序可以在运行时获取并使用插件类的构造函数、方法和属性。内省也可以应用于创建Java Beans和Javadocs中。  
<!--more-->
**Introspector类**  
Introspector类为访问目标Jave Bean支持的属性、事件和方法提供了标准方法。该方法可用于工具类（如BeanUtils）中。  
对于属性、事件和方法中的每一类信息，Introspector会分别分析目标bean以及其父类，寻找显式或隐式信息并用其构建一个能够全面描述目标bean的BeanInfo对象。  
{% img http://dl2.iteye.com/upload/attachment/0093/3705/b0194712-7259-3d67-9f84-da5282bee5cb.jpg %}  
通过调用Introspector.getBeanInfo()方法来获得指定类的bean信息。Java Bean规范允许通过实现BeanInfo接口，定义一个对象来描述bean。为了将BeanInfo与bean关联起来，须遵守如下命名模式：bean信息类的名字必须是将"BeanInfo"添加到bean名字的后面构成。例如：  

{% coderay lang:java linenos:true %} 
package com.cwind.introspector;  
  
public class Ultraman extends Superhero {  
        public String avanta ;  
  
        public Ultraman(String avanta) {  
               super ();  
               this .avanta = avanta;  
       }  
  
        public String getAvanta() {  
               return avanta ;  
       }  
  
        public void setAvanta(String avanta) {  
               this .avanta = avanta;  
       }  
}  
{% endcoderay %}  
相关信息类：

{% coderay lang:java linenos:true %} 
package com.cwind.introspector;  
  
import java.beans.IntrospectionException;  
import java.beans.PropertyDescriptor;  
import java.beans.SimpleBeanInfo;  
  
public class UltramanBeanInfo extends SimpleBeanInfo {  
        public PropertyDescriptor[] getPropertyDescriptors() {  
               try {  
                      return new PropertyDescriptor[]{  
                        new PropertyDescriptor("avanta" , Ultraman. class),  
                        new PropertyDescriptor("name" , Ultraman. class)  
                     };  
              } catch (IntrospectionException e) {  
                     e.printStackTrace();  
                      return null ;  
              }  
       }  
}  
{% endcoderay %}
信息类会先从Bean类所在的包内查找，如上例中搜索路径为com.cwind.introspector.UltramanBeanInfo。搜索路径也可以通过Introspector.setBeanInfoSearchPath()进行设置。使用BeanInfo类是为了获取对bean属性的控制权。只需提供属性名和所属的bean类，就可以为每个属性构建一个PropertyDescriptor。
演示类Superhero及其另一子类Titan定义：  

{% coderay lang:java linenos:true %} 
package com.cwind.introspector;  
  
public class Superhero {  
     private String name ;  
     private String superPower ;  
     private int age ;  
         
     public Superhero(){  
          this.name = "defaultName" ;  
          this.superPower  = "defaultSuperPower" ;  
          this.age = 0;  
     }  
         
     public Superhero(String name, String superPower, int age) {  
          super();  
          this.name = name;  
          this.superPower = superPower;  
          this.age = age;  
     }  
  
     public String getName() {  
          return name ;  
     }  
     public void setName(String name) {  
          this.name = name;  
     }  
     public String getSuperPower() {  
          return superPower ;  
     }  
     public void setSuperPower(String superPower) {  
          this.superPower = superPower;  
     }  
     public int getAge() {  
          return age ;  
     }  
     public void setAge(int age) {  
          this.age = age;  
     }  
}  
  
package com.cwind.introspector;  
  
public class Titan extends Superhero {  
     private double height ;  
     private double weight ;  
         
     public Titan(double height, double weight) {  
          super();  
          this.height = height;  
          this.weight = weight;  
     }  
         
     public double getHeight() {  
          return height ;  
     }  
     public void setHeight(double height) {  
          this.height = height;  
     }  
     public double getWeight() {  
          return weight ;  
     }  
     public void setWeight(double weight) {  
          this.weight = weight;  
     }    
}  
{% endcoderay %}  
可以看到，Ultraman类有一个显式的BeanInfo类，其中的属性描述符仅包括"avanta"和继承自父类的"name"。Titan没有显式的BeanInfo类。下面用一个测试类来打印Introspector获取的BeanInfo信息，分别打印两个Ultraman和Titan实例的属性名称及其对应的值，比较其异同。  

{% coderay lang:java linenos:true %} 
package com.cwind.introspector;  
  
import java.beans.IntrospectionException;  
import java.beans.Introspector;  
import java.beans.PropertyDescriptor;  
import java.lang.reflect.InvocationTargetException;  
  
public class IntrospectorTest {  
     public static void main(String[] args) 
	throws IntrospectionException, IllegalArgumentException,
	 IllegalAccessException, InvocationTargetException{  
          PropertyDescriptor[] ultramanProps 
	= Introspector.getBeanInfo(Ultraman.class).getPropertyDescriptors();  
          Ultraman sailor = new Ultraman("sailor" );  
          for(PropertyDescriptor prop : ultramanProps){
               System. out.println("Property name: " + prop.getName()
		+ ", value: "+ prop.getReadMethod().invoke(sailor, null));  
          }  
          System. out.println();  
          PropertyDescriptor[] titanProps 
	= Introspector.getBeanInfo(Titan.class).getPropertyDescriptors();  
          Titan titan = new Titan(999,888);  
          for(PropertyDescriptor prop : titanProps){  
               System. out.println("Property name: " + prop.getName()
		+ ", value: "+ prop.getReadMethod().invoke(titan, null));  
          }  
     }  
}  
{% endcoderay %}   
输出结果如下：

{% coderay lang:java linenos:true %} 
Property name: avanta, value: sailor  
Property name: name, value: defaultName  
  
Property name: age, value: 0  
Property name: class, value: class com.cwind.introspector.Titan  
Property name: height, value: 999.0  
Property name: name, value: defaultName  
Property name: superPower, value: defaultSuperPower  
Property name: weight, value: 888.0  
{% endcoderay %}   
可以看到，对于前者，只打印出其显式BeanInfo类中返回的属性描述符所对应的属性；对于后者，使用低层次的反射来获取所有属性，并按照属性名称字母序将属性描述符数组返回。
为了更好的性能，Introspector缓存BeanInfo；因此，若在使用多个类加载器的应用程序中使用Introspector须小心谨慎。可以调用Introspector.flushCaches或Introspector.flushFromCaches方法从缓存中清空内省的类。  
**Reference：**  
1. [Java API 1.6](http://docs.oracle.com/javase/6/docs/api/)  
2. [Java反射总结](http://my.oschina.net/zookeeper/blog/179269)  
3. [Java内省机制](http://blog.csdn.net/hahalzb/article/details/5972421)  
4. [Java语言的反射和内省](http://www.blogjava.net/wiflish/archive/2007/03/05/101964.html)  
5. [Using Introspection in Java](http://www.codeproject.com/Articles/235269/Using-Introspection-in-Java)  
6. Java2核心技术 卷II：高级特性，第8章：JavaBean构件；【美】Cay S. Horstmann, Gary Cornell 著；机械工业出版社