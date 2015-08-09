---
layout: post
title: "MEAN全栈开发：前后端整合"
date: 2015-06-09 23:57:43 +0800
updated: 2015-06-09 23:58:10 +0800
comments: true
categories: MEAN MongoDB ExpressJS AngularJS NodeJS 
tags: [translation, MEAN, full stack, Nodejs, Angularjs, Express, MongoDB]
keywords: MEAN, 全栈开发, Nodejs, Angularjs, Express, MongoDB  
---
本文是由三个部分组成的系列教程的最后一篇。我们将使用MEAN技术栈（MongoDB，ExpressJS，AngularJS和NodeJS）构建一个待办事项应用程序。  

#第三部分：前后端整合  
{% img http://dl.iteye.com/upload/picture/pic/133685/7c4ef50f-b7cd-3334-a29e-816cfcea7eba.png %}   
<!--more-->  

##后台服务端简述

NodeJS是一个自底向上构建的非阻塞I/O范例，比起使用线程技术的其他语言如Java，它使得每个CPU的使用更加高效。  
LAMP技术栈（Linux-Apache-MySQL-PHP）是被广泛使用了多年的Web应用开发栈。很多著名的平台，如Wikipedia，Wordpress，甚至Facebook都正在使用它或是由它起步的。对于企业应用，通常来说走Java路线：Hibernate，Spring，Struts，JBoss。有些更敏捷的框架也被广泛使用，例如Ruby on Rails，对于Python而言则有Django和Pylon。  

{% img http://dl.iteye.com/upload/picture/pic/133687/f1084b0c-0f4a-323b-9834-0193a5896822.jpg %}  

（图片来自[backand.com](http://blog.backand.com/mean-vs-lamp/)）  

那么，为什么又有MEAN栈呢？

**JavaScript无处不在**  
目前，JavaScript已经无处不在：智能手机，电脑，浏览器，服务器，机器人，开源硬件，迷你电脑等快速发展的领域。因此，不管你选取何种技术栈来构建Web应用，你需要熟悉JavaScript。在这种情况下，在任何它能够胜任的地方使用它将会节省时间，特别是用于构建Web应用。MEAN栈包含了所需要的全部，使用JavaScript构建一个从前端到后端的完整的Web应用。  

**非阻塞架构**  
JavaScript是动态类型，面向对象的功能性脚本语言。在数年前的脚本语言战争中，它能够胜过Java Applets的特性是，简单轻量以及非阻塞的事件循环。阻塞是指当执行某行代码时，其他代码被锁住并等待那行代码执行结束。相对的，非阻塞给予了每行代码执行机会，当事件发生时，可以通过回调来返回。阻塞式编程语言（Java，Ruby，Python，PHP，……）使用多线程技术来解决并发问题，而JavaScript通过单线程非阻塞事件循环来处理该问题。  

{% img http://dl.iteye.com/upload/picture/pic/133689/ba48f6f9-6434-371a-a1b3-611ef83c74fd.png %}
{% img http://dl.iteye.com/upload/picture/pic/133691/c024765a-1a8d-3678-8f85-18298d0e670e.png %}

（图片来自[strongloop.com](http://strongloop.com/strongblog/node-js-is-faster-than-java/)）  

一些公司如[Paypal](https://www.paypal-engineering.com/2013/11/22/node-js-at-paypal/)将其Java后台服务端迁移到NodeJS上，然后发现性能有所提高，响应时间减少，同时开发速度加快了。同样的事情发生在[Groupon](https://engineering.groupon.com/2013/misc/i-tier-dismantling-the-monoliths/)，它们由Java/Rails后台迁移而来。    

**敏捷且充满活力的社区**    
JavaScript背后的社区非常有活力，并且几乎涉足了技术相关的所有领域：数据可视化，客户端框架，服务端框架，数据库，机器人领域，构建工具等等。  

##搭建开发环境  

###MEN：MongoDB，ExpressJS与NodeJS  
在[前一篇教程](http://gocwind.com/blog/2015/06/09/creating-a-restful-api-tutorial-with-nodejs-and-mongodb/)里，我们一起构建了RESTful API，现在我们使用它创建Web应用。[完整代码](https://github.com/amejiarosario/todoAPIjs)  

{% coderay lang:javascript linenos:true Getting the back-end code build on Part II %}
git clone https://github.com/amejiarosario/todoAPIjs.git
{% endcoderay %} 

###A：AngularJS  
在本系列教程的第一篇里，我们构建了一个非常简单的todoApp。你可以下载[源文件](https://gist.githubusercontent.com/amejiarosario/068143b53e54db43ef38/raw/ngTodo.html)作为参照，或者查看[动态演示](https://cdn.rawgit.com/amejiarosario/068143b53e54db43ef38/raw/ngTodo.html)。你可能注意到angularJS应用非常简单，甚至简单到可以完全在同一个文件。在后续的教程里，我们将把它模块化，放到不同文件，添加测试以及样式表。  

我们首先来看一下这个ExpressJS应用（todoAPIjs），回顾一下默认的路由机制。  

1. `app.js`加载所有的路由。
2. 根路径(/)在`routes/index.js`中挂载。  
3. `routes/index.js`设置变量“title”的值，并渲染`index.ejs`。  

{% coderay lang:javascript linenos:true Tracing ExpressJS index route %}
// app.js
var routes = require('./routes/index');
app.use('/', routes);

// ./routes/index.js
router.get('/', function(req, res) {
  res.render('index', { title: 'Express' });
});

// ./views/index.ejs
    <h1><%= title %></h1>
    <p>Welcome to <%= title %></p>
{% endcoderay %}  

让我们把`ngTodo.html`中的文件内容拷贝到`./views/index.ejs`，在`./routes/index.js`把title值设为“ngTodo App”。不要忘记添加ng-app指令。`<html ng-app="app">`。

[变更](https://github.com/amejiarosario/todoAPIjs/commit/ebf20f4093aa20c867777b4b3db825429b54a20d)  

##前后端整合  

###AngularJS CRUD  

**通过$http读取数据**  
你可能已经注意到了，在前面的工厂方法中，我们返回的是一个固定的数组。现在我们修改它，让它与我们刚刚构建的API进行通信。  

`$http`是一个Angular核心服务，为应用提供了发送`XMLHttpRequest`或`jsonp`请求的方式。你可以通过http指令传递对象，或者调用$http.verb(`$http.get`，`$http.post`)。  

`$http`返回一个promise，它有`success`和`error`两个方法。  

{% coderay lang:javascript linenos:true AngularJS $HTTP Usage Example %}
$http({method: 'GET', url: '/todos'}).
  success(function(data, status, headers, config) {
    // this callback will be called asynchronously
    // when the response is available.
    console.log('todos: ', data );
  }).
  error(function(data, status, headers, config) {
    // called asynchronously if an error occurs
    // or server returns response with an error status.
    console.log('Oops and error', data);
  });
{% endcoderay %}  

现在尝试把它用到我们的应用里。打开`vies/index.ejs`做如下改动：  

{% coderay lang:javascript linenos:true Using $http to retrieve data from database %}
    // Service
    .factory('Todos', ['$http', function($http){
      return $http.get('/todos');
    }])

    // Controller
    .controller('TodoController', ['$scope', 'Todos', function ($scope, Todos) {
      Todos.success(function(data){
        $scope.todos = data;
      }).error(function(data, status){
        console.log(data, status);
        $scope.todos = [];
      });
    }])
{% endcoderay %}  

如果你在MongoDB中有数据，你就可以在首页中看到它们。如果没有，可以参照上篇教程添加一些数据。  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/0221aebd62e88445629debe4f132684686cf48ec)  

**通过$resource读取数据**  
如果你点击任何一个代办事项，你将被重定向到详情页面。目前你还不会看到任何东西。我们需要先更新`TodoDetailCtrl`。目前为止，GET方法已经可以正常工作了。有个封装了处理RESTful请求的高层抽象的Angular服务：`$resource`。  

初始化：`$resource(url, [paramDefaults], [actions], options);`  

它包含了下面几个我们之前已经定义的动作。但是少了一个，你能发现缺少的是哪一个吗？  

{% coderay lang:javascript linenos:true $resource actions %}
{ 'get':    {method:'GET'},  // get individual record
  'save':   {method:'POST'}, // create record
  'query':  {method:'GET', isArray:true}, // get list all records
  'remove': {method:'DELETE'}, // remove record
  'delete': {method:'DELETE'} }; // same, remove record
}
{% endcoderay %}  

实例的使用方式见下（稍后给出示例）：  

- GET: `Resource.get([parameters], [success], [error])`  
- Non-GET: `Resource.action([parameters], postData, [success], [error])`  
- Non-GET: `resourceInstance.$action([parameters], [success], [error])`    
-  `$resource`并不是Angular核心的一部分，它需要引入`ngResource`作为依赖。我们可以通过CDN获取它： 
`<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.25/angular-resource.min.js"></script>`  

引入依赖并使用它的示例：  

{% coderay lang:javascript linenos:true $resource.query() %}
  // add ngResource dependency
  angular.module('app', ['ngRoute', 'ngResource'])
        .factory('Todos', ['$resource', function($resource){
          return $resource('/todos/:id', null, {
            'update': { method:'PUT' }
          });
        }])
        .controller('TodoController', ['$scope', 'Todos', function ($scope, Todos) {
          $scope.todos = Todos.query();
        }])
{% endcoderay %}  

请注意`$resource`并不像`$http`那样返回一个promise，它会返回一个空引用。Angular会使用一个空的`$scope.todos`来进行渲染。当`Todos.query()`返回从服务器读取的数据时，UI将会自动被重新渲染。  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/2aff6fe004bf7f7b2cd1b91d53e6958c3b158a20)  

**AngualrJS新建**  
我们需要创建一个新的文本框，一个按钮来发送`POST`请求给服务器，并将其添加到`$scope`。  

{% coderay lang:javascript linenos:true New textbox for adding Todos %}  
New task <input type="text" ng-model="newTodo"><button ng-click="save()">Create</button>
{% endcoderay %}  

请注意这里我们使用了一个新的指令`ng-click`，当它被点击时，执行指定函数。Angular会保证不同浏览器的行为都是一致的。  

{% coderay lang:javascript linenos:true Save function $resource.$save(…) %}  
.controller('TodoController', ['$scope', 'Todos', function ($scope, Todos) {
      $scope.todos = Todos.query();

      $scope.save = function(){
        if(!$scope.newTodo || $scope.newTodo.length < 1) return;
        var todo = new Todos({ name: $scope.newTodo, completed: false });

        todo.$save(function(){
          $scope.todos.push(todo);
          $scope.newTodo = ''; // clear textbox
        });
      }
    }])
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/46dd14023e2d9eff72d1366dbba9c9c8c872e07b)  

**显示待办事项详情**  
每次我们点击待办事项链接时，显示的内容为空。现在我们对此做点改动。首先我们需要设置真实的`_id`给链接，以取代索引`$index`。 
 
{% coderay lang:javascript linenos:true Change the ID link in the “/todos.html” template %}  
    <li ng-repeat="todo in todos | filter: search">
      <input type="checkbox" ng-model="todo.completed">
      <a href="#/"></a>
    </li>
{% endcoderay %} 

{% coderay lang:javascript linenos:true Update TodoDetailCtrl with $resource.get %} 
    .controller('TodoDetailCtrl', ['$scope', '$routeParams', 'Todos', 
function ($scope, $routeParams, Todos) {
      $scope.todo = Todos.get({id: $routeParams.id });
    }])
{% endcoderay %} 

现在你可以看到待办事项详情了。:-)

[变更](https://github.com/amejiarosario/todoAPIjs/commit/2484107294163a25621fba3785601adb32229ae9)  

**AngularJS更新（在线编辑）**  
这是一个非常酷的功能。先来了解一下相关的新的指令：  

- **ng-show**: 当指定的变量为true时，显示元素；当变量为false时隐藏。  
- **ng-change**: 发生任何改动后，用来求输入元素表达式的值的指令。  

{% coderay lang:javascript linenos:true Template todos.html %} 
<script type="text/ng-template" id="/todos.html">
  Search: <input type="text" ng-model="search.name">
  <ul >
    <li ng-repeat="todo in todos | filter: search">
      <input type="checkbox" ng-model="todo.completed" ng-change="update($index)">
      <a ng-show="!editing[$index]" href="#/"></a>
      <button ng-show="!editing[$index]" ng-click="edit($index)">edit</button>

      <input ng-show="editing[$index]" type="text" ng-model="todo.name">
      <button ng-show="editing[$index]" ng-click="update($index)">Update</button>
      <button ng-show="editing[$index]" ng-click="cancel($index)">Cancel</button>
    </li>
  </ul >
  New task <input type="text" ng-model="newTodo"><button ng-click="save()">Create</button>
</script>
{% endcoderay %} 

我们添加了一个新的变量`$scope.editing`用来指示表单中编辑区域显示与否。更进一步，请注意ng-click函数：编辑、更新与取消。我们来看看它们做了些什么。  

{% coderay lang:javascript linenos:true Todo Controller %} 
    .controller('TodoController', ['$scope', 'Todos', function ($scope, Todos) {
      $scope.editing = [];
      $scope.todos = Todos.query();

      $scope.save = function(){
        if(!$scope.newTodo || $scope.newTodo.length < 1) return;
        var todo = new Todos({ name: $scope.newTodo, completed: false });

        todo.$save(function(){
          $scope.todos.push(todo);
          $scope.newTodo = ''; // clear textbox
        });
      }

      $scope.update = function(index){
        var todo = $scope.todos[index];
        Todos.update({id: todo._id}, todo);
        $scope.editing[index] = false;
      }

      $scope.edit = function(index){
        $scope.editing[index] = angular.copy($scope.todos[index]);
      }

      $scope.cancel = function(index){
        $scope.todos[index] = angular.copy($scope.editing[index]);
        $scope.editing[index] = false;
      }
    }])
{% endcoderay %} 

当我们编辑时请注意我们把原始todo任务拷贝到editing变量。这能够起到两个作用：1. 将值置为`ture`以显示带有`ng-show`的表单元素，2. 保存原始值的拷贝，以防我们点击取消。  

现在，我们来看一下待办事项细节页面。我们将它像添加记录页面一样去更新一下。  

{% coderay lang:javascript linenos:true Todo Details %} 
<script type="text/ng-template" id="/todoDetails.html">
  <h1></h1>
  completed: <input type="checkbox" ng-model="todo.completed"><br>
  note: <textarea ng-model="todo.note"></textarea><br><br>

  <button ng-click="update()">Update</button>
  <a href="/">Cancel</a>
</script>
{% endcoderay %} 

类似的，我们添加了一个更新函数。然而，这一次我们不需要传递任何索引，因为每次只会有一个待办事项。保存了之后，我们转到根路径`/`。 

{% coderay lang:javascript linenos:true Todo Details %} 
    .controller('TodoDetailCtrl', ['$scope', '$routeParams', 'Todos', '$location', 
function ($scope, $routeParams, Todos, $location) {
      $scope.todo = Todos.get({id: $routeParams.id });

      $scope.update = function(){
        Todos.update({id: $scope.todo._id}, $scope.todo, function(){
          $location.url('/');
        });
      }
    }])
{% endcoderay %}  

- `$location.url([url])`是一个能够让我们改变url的getter/setter方法，从而改动路由/视图。  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/b6394448e1e1e8384815877df764507d6562dc4d)  

**AngularJS 删除**  
这是用于实现删除功能的函数。相当直白。请注意当我们从代办事项数组中删除元素时，`$scope.todos.splice(index, 1)`它同时会从DOM中消失。非常酷，是吧？  

{% coderay lang:javascript linenos:true Delete functionality (diff) %} 
diff --git a/views/index.ejs b/views/index.ejs
index 9c3ef46..afb37e1 100644
--- a/views/index.ejs
+++ b/views/index.ejs
@@ -22,6 +22,7 @@
           <input type="checkbox" ng-model="todo.completed" ng-change="update($index)">
           <a ng-show="!editing[$index]" href="#/"></a>
           <button ng-show="!editing[$index]" ng-click="edit($index)">edit</button>
+          <button ng-show="!editing[$index]" ng-click="remove($index)">remove</button>

           <input ng-show="editing[$index]" type="text" ng-model="todo.name">
           <button ng-show="editing[$index]" ng-click="update($index)">update</button>
@@ -37,6 +38,7 @@
       note: <textarea ng-model="todo.note"></textarea><br><br>

       <button ng-click="update()">update</button>
+      <button ng-click="remove()">remove</button>
       <a href="/">Cancel</a>
     </script>

@@ -85,6 +87,13 @@
             $scope.todos[index] = angular.copy($scope.editing[index]);
             $scope.editing[index] = false;
           }
+
+          $scope.remove = function(index){
+            var todo = $scope.todos[index];
+            Todos.remove({id: todo._id}, function(){
+              $scope.todos.splice(index, 1);
+            });
+          }
         }])

         .controller('TodoDetailCtrl', ['$scope', '$routeParams', 'Todos', '$location', 
function ($scope, $routeParams, Todos, $location) {
@@ -95,6 +104,12 @@
               $location.url('/');
             });
           }
+
+          $scope.remove = function(){
+            Todos.remove({id: $scope.todo._id}, function(){
+              $location.url('/');
+            });
+          }
         }])

         //---------------
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/b9ff3a863c78d72e71b5cc9eb573bb3cb9d87179)  

**恭喜！你现在已经是一个MEAN全栈开发者了！**  

##下一步？  
学习如何使用GruntJS，来自动完成MEAN开发工作流中那些重复的任务。  
[GruntJS 教程](http://adrianmejia.com/blog/2014/10/07/grunt-js-tutorial-from-beginner-to-ninja/)  

同时，你也可以去更多了解一些全栈开发框架解决方案。  

##JavaScript Web全栈开发框架  
我们在系列教程中做的事情可以通过只是在命令行中敲击几个按键完成 ;-)。然而，了解发生了什么是很有好处的。所以，我将会介绍几个框架给你，这将会节省许多时间。  

###使用MEAN.io   

[MeanIO](http://mean.io/)使用一个定制的CLI工具：'mean'。它的自包含的包中既有客户端也有服务器端代码，在模块化的道路上更进了异步。在写本文的时候，它包含了MEAN-Admin，翻译，文件上传，图像处理等若干实用模块。   

###使用MEAN.js  
[MeanJS](http://meanjs.org/)由MEAN.IO的一个分支发展而来，它使用Yeoman生成器来产生Angular的CRUD模块，路由，控制器，视图，服务及其他。也包含了用于Express的生成器：模型，控制器，路由和测试。它有非常好的文档支持。  

###其他框架  
- [Meteor](https://www.meteor.com/) - Meteor是一个用于快速构建高质量Web应用的开源平台，不管你是专家还是初学者。  
- [Sails](http://sailsjs.org/) - 一个用于开发下一代Web应用的梦想中的Web框架。  
- [Yahoo! Mojito](https://developer.yahoo.com/cocktails/mojito/) - 一个用于开发移动应用的JavaScript MVC框架，Yahoo! Cocktails的组成部分。  
- [Tower.js](http://towerjs.org/) - 用于构建应用，操作数据，自动化分布式基础设施的小型组件  

原文链接：[MEAN Stack Tutorial MongoDB ExpressJS AngularJS NodeJS (Part III)](http://adrianmejia.com/blog/2014/10/03/mean-stack-tutorial-mongodb-expressjs-angularjs-nodejs/)



