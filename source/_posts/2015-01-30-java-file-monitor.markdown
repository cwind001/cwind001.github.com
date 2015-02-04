---
layout: post
title: "Java文件变更监控的两种实现"
date: 2015-01-30 14:25:04 +0800
comments: true
categories: Java
tags: [Java]
keywords: Java监控文件修改, Timer, TimerTask, WatchService
---
**对文件及文件夹进行修改变更监测有很广泛的应用，例如：**  

- 通知配置文件的改变  
- 跟踪某些关键的系统文件的变化  
- 监控某个分区磁盘的整体使用情况  
- 系统崩溃时进行自动清理  
- 自动触发备份进程  
- 向服务器上传文件结束时发出通知    
下面给出Java的两种实现，源码可以在GitHub上找到 [FileMonitor](https://github.com/cwind001/CwindJavaLab/tree/master/FileMonitor)

**JDK1.6及之前版本: 基于Timer实现**  
<!--more-->
**两个关键类：**  

- java.util.Timer  
- java.util.TimerTask

Timertask是由Timer执行的实际任务，实现了Rannable接口。通过重写run()方法来指定具体任务细节。  
{% img http://dl2.iteye.com/upload/attachment/0105/5397/ddf9a7c5-f08a-3fd3-b1f8-6859e1054bd8.jpg %}

**Timer工作原理：**  
Timer是用于调度一次性执行或重复执行的工具类。通过TaskQueue保存需要调度的TimerTask，当某个Task被废弃时（一次性任务结束或TimerTask.cancel()），将其从该队列中移除。  
Timer类维护一个后台线程（守护线程或用户线程，取决于如何创建Timer对象），该线程通常称为Timer任务执行线程。在TimerThread的mainLoop()中依据各个TimerTask的状态和调度时间设定，决定执行或移除TimerTask。  
**TimerTask应设计为执行不占用太长时间**，否则同一个Timer队列中其他的TimerTask的执行将会延迟。  
更多可参见：[What is Timer and TimerTask in Java](http://javarevisited.blogspot.com/2013/02/what-is-timer-and-timertask-in-java-example-tutorial.html)

**基于Timer的FileMonitor的实现：**  
{% img http://dl2.iteye.com/upload/attachment/0105/5399/4b7c54fa-cac9-3d7b-85c7-6e655ff8bbcb.jpg %}

通过实现FileChangeObserver接口，该FileMonitor允许对任意文件添加任意多个Observer。  
[FileChangeMonitor及FileChangeTask源码](https://github.com/cwind001/CwindJavaLab/blob/master/FileMonitor/src/main/java/com/cwind/file/FileChangeMonitor.java)  
FileChangeMonitor本身是一个单例。fileObservers由Collections.synchronizedMap()初始化，保证在该map上的每一个原子操作都将被同步。在其addObserver方法中为每一个fileChangeObserver创建一个FileChangeTask，将其加入fileObservers中。FileChangeTask扩展了TimerTask，由Timer调度执行。

``` java FileChangeMonitor.addObserver() https://github.com/cwind001/CwindJavaLab/blob/master/FileMonitor/src/main/java/com/cwind/file/FileChangeMonitor.java
	public void addObserver(FileChangeObserver observer, String filename, long delay) throws FileNotFoundException {  
    	TimerTask task = new FileChangeTask(observer , filename );  
    	List<TimerTask> tasks = fileObservers.get(filename );  
    	if(tasks ==null){  
        	tasks = new ArrayList<TimerTask>();  
   		}  
    	tasks.add( task);  
    	fileObservers.put(filename , tasks );  
    	timer.schedule( task, delay, delay);  
	}  
```
在FileChangeTask的run()函数中，通过比对时间戳来判断文件是否修改，若发生改动，则通知其Observer进行相应处理。  
``` java FileChangeTask.run() https://github.com/cwind001/CwindJavaLab/blob/master/FileMonitor/src/main/java/com/cwind/file/FileChangeMonitor.java
	public void run() {
		try	{
			long newLastModified = file.lastModified();
			if (newLastModified > lastModified) {
				lastModified = newLastModified;
				observer.fileChanged(file.getPath());
			}
		}
		catch (Exception e)	{
			System.err.println(e.getMessage());
		}
	} 
```

测试用例[FileMonitorTest](https://github.com/cwind001/CwindJavaLab/blob/163448ce07ecca1738b306bed9bf1b39464d345c/FileMonitor/src/test/java/com/cwind/file/FileMonitorTest.java)中为sample1.txt添加了consoleObserver和emailObserver，为sample2.txt添加了consoleObserver。然后对这两个文件分别进行修改。  
``` java FileMonitorTest https://github.com/cwind001/CwindJavaLab/blob/163448ce07ecca1738b306bed9bf1b39464d345c/FileMonitor/src/test/java/com/cwind/file/FileMonitorTest.java
package com.cwind.file;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

public class FileMonitorTest {
	
	File sampleFile1, sampleFile2;
	FileChangeMonitor monitor;
	ConsoleFileChangeObserver consoleObserver;
	EmailFileChangeObserver emailObserver;
	
	@Before
	public void setUp() throws Exception {
		sampleFile1 = new File("sample1.txt"); 
		sampleFile2 = new File("sample2.txt");
		monitor = FileChangeMonitor.getInstance();
		consoleObserver = new ConsoleFileChangeObserver();
		emailObserver = new EmailFileChangeObserver("billchen01@163.com");
	}

	@After
	public void tearDown() throws Exception	{
		
	}
	
	@Test
	public void testMonitorSampleFile() throws InterruptedException, IOException{
		monitor.addObserver(consoleObserver, sampleFile1.getPath(), FileChangeMonitor.DELAY_TIME);
		monitor.addObserver(emailObserver, sampleFile1.getPath(), FileChangeMonitor.DELAY_TIME);
		monitor.addObserver(consoleObserver, sampleFile2.getPath(), FileChangeMonitor.DELAY_TIME);
		
		FileOutputStream fos1 = new FileOutputStream(sampleFile1);
		FileOutputStream fos2 = new FileOutputStream(sampleFile2);
		fos1.write(0);
		fos2.write(0);
		fos1.flush();
		fos2.flush();
		fos1.close();
		fos2.close();
		Thread.sleep(3000);
	}
}
```

输出结果如下：  
`Console: File sample1.txt is changed, will print warning message to console.`  
`File sample1.txt is changed, will send email to billchen01@163.com.`  
`Console: File sample2.txt is changed, will print warning message to console.`  

**JDK 1.7 及之后版本：基于WatchService实现**  
Java 7 的新IO - NIO.2提供了一组新的类和方法，主要存在于java.nio包内。它完全取代了java.io.File与文件系统的交互，并提供了新的异步处理类，无需手动配置线程池和其他底层并发控制，便可在后台线程中执行文件和网络IO操作。  
其中Path是新文件IO的基石。为与之前版本兼容，java.io.File类中新增了toPath()方法，Path类中提供了toFile()方法。
Watch Service API可用于将指定目录注册到监视服务上。注册时须指定事件类型，如文件创建、修改、删除等。相关类图如下：  
{% img http://dl2.iteye.com/upload/attachment/0105/5403/f20e959b-2ded-3a35-b984-61f5010f7efb.jpg %}  

WatchService是监视服务接口，在不同系统上有不同的实现类。实现了Watchable接口的对象方可注册监视服务，java.nio.file.Path实现了此接口。WatchKey表示Watchable对象和WatchService的关联关系，在注册时被创建。注册完成后，WatchKey将被置为'ready'状态，直到下列三种情况之一发生：  

1. WatchKey.cancel()被调用
2. 被监控的目录不存在或不可访问
3. WatchService对象被关闭  

当文件改动发生时，WatchKey的状态将会被置为"signaled"然后被放入待处理队列中。WatchService提供了**三种从队列中获取WatchKeys的方式：**

1. poll - 返回队列中的一个key。如果没有可用的key，将立即返回null。
2. poll(long, TimeUnit) - 如果队列中存在可用的key则将之返回，否则在参数预置的时间内等待可用的key。TimeUnit用来指定前一个参数表示的时间是纳秒、毫秒或是其他的时间单位。
例子：final WatchKey watchKey = watchService.poll(1, TimeUnit.MINUTES);将会等待1分钟
3. take - 方法将会等待直到可用的key被返回。

**获取WatchKey后进行处理：**

1. 通过WatchKey.pollEvents()函数得到WatchEvents列表。
2. 对于每一个WatchEvent，可以通过kind()函数获得其改动类型。
3. 通过WatchEvent.context()函数得到发生该事件的文件名
4. 当该key的所有事件处理完成后，需要调用WatchKey.reset()方法把该key重置为ready状态。若不重置，该key将无法接收后续的改动。若reset返回false，表示该WatchKey不再合法，主循环可以退出。

``` java WatchServiceTest https://github.com/cwind001/CwindJavaLab/blob/163448ce07ecca1738b306bed9bf1b39464d345c/FileMonitor/src/test/java/com/cwind/file/WatchServerTest.java
package com.cwind.file;

import java.io.IOException;
import java.nio.file.FileSystems;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardWatchEventKinds;
import java.nio.file.WatchEvent;
import java.nio.file.WatchKey;
import java.nio.file.WatchService;

public class WatchServerTest {
	public static void main(String[] args) throws InterruptedException, IOException {
		WatchService watchService = FileSystems.getDefault().newWatchService();
		final Path path = Paths.get(".");
		final WatchKey watchKey = path.register(watchService, StandardWatchEventKinds.ENTRY_MODIFY, 
				StandardWatchEventKinds.ENTRY_CREATE, StandardWatchEventKinds.ENTRY_DELETE);
		boolean fileNotChanged = true;
		int count = 0;
		while (fileNotChanged) {
			final WatchKey wk = watchService.take();
		    System.out.println("Loop count: " + count);
		    for (WatchEvent<?> event : wk.pollEvents()) {
		        final Path changed = (Path) event.context();
		        System.out.println(changed + ", " + event.kind());
		        if (changed.endsWith("sample1.txt")) {
		            System.out.println("Sample file has changed");
		        }
		    }
		    // reset the key
		    boolean valid = wk.reset();
		    if (!valid) {
		        System.out.println("Key has been unregisterede");
		    }
		    count++;
		}
	}
}
```  
总结，使用WatchService步骤如下：  

1. 创建WatchService
2. 得到待检测目录的Path
3. 将目录登记到变化监测名单中
4. 执行WatchService的take()方法，直到WatchKey到来。
5. 得到WatchKey后遍历WatchEvent进行检测
6. 重置key准备下一个事件，继续等待  

大多数文件系统实现包含了文件更改通知的本地支持，Watch Service API正是利用了文件系统的这种机制。若文件系统并不支持变更通知机制，Watch Service仍然会轮询文件系统，等待事件产生。
  
**References:**  

1. [Watching a Directory for Changes](http://docs.oracle.com/javase/tutorial/essential/io/notification.html)
2. [Using Java 7's WatchService to Monitor Directories](http://java.dzone.com/articles/using-java-7s-watchservice)