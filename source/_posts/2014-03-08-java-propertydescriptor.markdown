---
layout: post
title: "Java PropertyDescriptor 应用及源码分析"
date: 2014-03-08 22:15:28 +0800
updated: 2014-03-08 22:15:28 +0800
comments: true
categories: Java
tags: [Java, Introspector, Reflection]
keywords: Java内省, Java反射, PropertyDescriptor  
---
前文[从Introspector谈Java内省机制](http://gocwind.com/blog/2014/01/20/java-introspector/)提到了通过Introspector.getBeanInfo()方法获取属性描述符数组，进而读取属性值的方式，但未对PropertyDescriptor的应用和实现作进一步阐释，在此作个补完。  
<!--more-->
**1. 概述**  
PropertyDescriptor描述Java Bean中通过一对存储器方法（getter / setter）导出的一个属性。我们可以通过该PropertyDescriptor对bean中的该属性进行读取和写入操作，也可以设置其getter / setter。  
{% img http://dl2.iteye.com/upload/attachment/0094/6177/5c9f92d1-c105-38f0-88e5-18f00e4d5531.jpg %}   
[PropertyDescriptor源码见此](http://www.oschina.net/code/explore/gcc-4.5.2/libjava/classpath/java/beans/PropertyDescriptor.java)  
**2. 关键接口及内部属性**  

{% coderay lang:java linenos:true %} 
public PropertyDescriptor(String name, 
	Class<?> beanClass) throws IntrospectionException  
public PropertyDescriptor(String name, Class<?> beanClass, 
String getMethodName, String setMethodName) throws IntrospectionException  
public PropertyDescriptor(String name, 
Method readMethod, Method writeMethod) throws IntrospectionException  
  
public Class<?> getPropertyType()  
public Method getReadMethod()  
public Method getWriteMethod()  
  
public void setReadMethod(Method readMethod) throws IntrospectionException  
public void setWriteMethod(Method writeMethod)  
public boolean equals(Object o)   
{% endcoderay %}  

相关的PropertyDescriptor内部属性如下：  

{% coderay lang:java linenos:false %} 
    Class<?> propertyType;     //该属性的类型  
    Method getMethod;     //getter  
    Method setMethod;     //setter  
{% endcoderay %}  
还有继承自其父类FeatureDescriptor的功能，用于指定该属性的编程名称  
**3. 简单应用**  
现有Person类如下：  

{% coderay lang:java linenos:true %}   
package com.cwind.property;  
  
public class Person {  
        private String name ;  
        private int age ;  
         
        public Person(){ this.name = ""; this.age = 0; }  
        public Person(String name, int age) { super(); this.name = name; 
	this. age = age; }  
  
        public String getName() { return name; }  
        public void setName(String name) { this. name = name; }  
  
        public int getAge() { return age; }  
        public void setAge(int age) { this. age = age; }  
         
        public String getNameInUpperCase(){  
               return this .name .toUpperCase();  
       }  
        public void setNameToLowerCase(String name){  
               this.name = name.toLowerCase();  
       }  
}  
{% endcoderay %}  

该类中除了name和age两个属性的标准getter和setter之外，还有增加了一个获取大写name的get方法和一个将name设置为小写的set方法。  
在测试类中，首先获得这两个方法对象。  

{% coderay lang:java linenos:true %} 
Class personClass = Class.forName("com.cwind.property.Person");  
Method read = personClass.getMethod("getNameInUpperCase", null);  
Method write = personClass.getMethod("setNameToLowerCase", String.class );  
  
//然后可以通过两种方式构造PropertyDescriptor  
PropertyDescriptor prop1 = new PropertyDescriptor( "name", Person.class );
     //使用其标准getter和setter  
PropertyDescriptor prop2 = new PropertyDescriptor( "name", read, write);
     //使用read和write两个方法对象所自定义的getter和setter  
  
//下面构建一个Person对象  
Person person = new Person("Kobe" , 36);  
System. out.println(prop1.getReadMethod().invoke(person, null));
     // --实际调用Person.getName(), result: Kobe  
System. out.println(prop2.getReadMethod().invoke(person, null));
     // --实际调用Person.getNameInUpperCase(), result: KOBE  
                       
prop1.getWriteMethod().invoke(person, "James");
     // --实际调用Person.setName(), person.name被设置为James  
prop2.getWriteMethod().invoke(person, "James");
     // --实际调用Person.setNameToLowerCase(), person.name被设置为james 
{% endcoderay %}  
**4. 源码分析**  
构造函数1：  

{% coderay lang:java linenos:true %} 
public PropertyDescriptor(String name, Class<?> beanClass)  
        throws IntrospectionException {  
        setName(name);     //设置属性编程名，本例中即'name'  
        if (name.length() == 0){  
            throw new IntrospectionException("empty property name");       
// 编程名为空则抛出异常  
        }  
        String caps = Character.toUpperCase(name.charAt(0))
	 + name.substring(1);       
// 标准getter应为getName()或isName(), 先将首字母大写  
        findMethods(beanClass, "is" + caps, "get" + caps, "set" + caps);       
// 参数依次为：类类型，可能的getter函数名1，可能的getter函数名2，setter函数名  
        if (getMethod == null){
   // findMethods()设置PropertyDescriptor的getMethod和setMethod属性  
            throw new IntrospectionException(  
                "Cannot find a is" + caps + " or get" + caps + " method");  
        }  
        if (setMethod == null){  
            throw new IntrospectionException(  
                "Cannot find a " + caps + " method" );  
        }  
        propertyType = checkMethods(getMethod, setMethod);       
// checkMethods()函数用来检测getMethod得到的类型与setMethod的参数类型是否匹配，
若匹配则置propertyType为该类型  
    }  
{% endcoderay %} 
构造函数2：  
`public PropertyDescriptor(String name, Class<?> beanClass, String getMethodName, String setMethodName) throws IntrospectionException`  
其实现与构造函数1类似，只是调用findMethods时直接查找指定的getter和setter函数名：  
        `findMethods(beanClass, getMethodName, null, setMethodName);`  
构造函数3则更加直观，直接设置方法对象  
`public PropertyDescriptor(String name, Method readMethod, Method writeMethod) throws IntrospectionException`  
两个比较重要的私有辅助函数分别为`findMethods()`和`checkMethods()`，分别看一下其实现  
findMethods用来按指定的getter和setter函数名在指定Class中查找getMethod和setMethod，并设置PropertyDescriptor的相关属性   

{% coderay lang:java linenos:true %} 
private void findMethods(Class beanClass,
 String getMethodName1, String getMethodName2, String setMethodName)
 throws IntrospectionException {  
        try {  
            // 首先查找getMethodName1指定的getter (isXXX)  
            if (getMethodName1 != null) {  
                try {  
                    getMethod = beanClass.getMethod(getMethodName1,
	 new Class[0]);  
                }  
                catch (NoSuchMethodException e)  
                {}  
            }  
            // 若失败，则查找getMethodName2指定的getter (getXXX)  
            if (getMethod == null && getMethodName2 != null) {  
                try {  
                    getMethod = beanClass.getMethod(getMethodName2,
	 new Class[0]);  
                }  
                catch (NoSuchMethodException e)  
                {}  
            }  
            if (setMethodName != null) {  
                if (getMethod != null) {  
                    // 如果得到了getMethod，则通过其返回值类型决定setMethod的参数类型  
                    Class propertyType = getMethod.getReturnType();  
                    if (propertyType == Void.TYPE) {   
// 若getter的返回值为Void类型则抛出异常  
                        String msg
			 = "The property's read method has return type 'void'";  
                        throw new IntrospectionException(msg);  
                    }  
  
                    Class[] setArgs = new Class[] { propertyType };   
                    try {  
                        setMethod = beanClass.getMethod(setMethodName,
	 setArgs);   
// 通过函数名和参数类型获得setMethod  
                    }  
                    catch (NoSuchMethodException e)  
                    {}  
                }  
                else if (getMethodName1 == null && getMethodName2 == null) {  
// getMethodName1和2均为空，则此属性为只写属性，此时遍历bean中的函数，
// 返回第一个名称与setMethodName一致且返回类型为Void的单参数函数  
                    Method[] methods = beanClass.getMethods();  
                    for (int i = 0; i < methods.length; i++) {  
                        if (methods[i].getName().equals(setMethodName)  
                            && methods[i].getParameterTypes().length == 1  
                            && methods[i].getReturnType() == Void.TYPE) {  
                            setMethod = methods[i];  
                            break;  
                        }  
                    }  
                }  
            }  
        }  
        catch (SecurityException e) {  
            String msg
		 = "SecurityException thrown on attempt to access methods.";
     // 作者在纠结要不要修改异常类型  
            throw new IntrospectionException(msg);  
        }  
    }  
{% endcoderay %}
checkMethods方法  

{% coderay lang:java linenos:true %} 
private Class<?> checkMethods(Method readMethod, Method writeMethod)
 throws IntrospectionException {  
        Class<?> newPropertyType = propertyType;  
         // 合法的read方法应该无参同时带有一个非空的返回值类型  
        if (readMethod != null) {  
            if (readMethod.getParameterTypes().length > 0) {  
                throw new IntrospectionException("read method
	 has unexpected parameters");  
            }  
            newPropertyType = readMethod.getReturnType();  
             if (newPropertyType == Void.TYPE) {  
                throw new IntrospectionException("read method
	 return type is void");  
            }  
        }  
         // 合法的write方法应该包含一个类型相同的参数  
        if (writeMethod != null) {  
            if (writeMethod.getParameterTypes().length != 1) {
	 // 参数不能超过一个  
                String msg = "write method
	 does not have exactly one parameter" ;  
                throw new IntrospectionException(msg);  
            }  
            if (readMethod == null) {  
                // 若无read方法，属性类型就应为writeMethod的参数类型  
                newPropertyType = writeMethod.getParameterTypes()[0];  
            }  
            else {  
                // 检查read方法的返回值类型是否与write方法的参数类型相匹配  
                if (newPropertyType != null  
                    && !newPropertyType.isAssignableFrom(  
                        writeMethod.getParameterTypes()[0])) {  
                     throw new IntrospectionException("read and write method  
	 are not compatible");  
                }  
            }  
        }  
         return newPropertyType;  
    }  
{% endcoderay %}  
最后提一句`PropertyDescriptor.equals()`,只有当属性类型、标志、读写方法和  `PropertyEditorClass` 均相同时才认为两个`PropertyDescriptor`相等  

{% coderay lang:java linenos:false %}
return samePropertyType  
                && sameFlags  
                && sameReadMethod  
                && sameWriteMethod  
                && samePropertyEditorClass; 
{% endcoderay %}   
