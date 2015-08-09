---
layout: post
title: "MEAN全栈开发：AngularJS实战教程"
date: 2015-06-05 21:36:10 +0800
updated: 2015-06-05 21:36:10 +0800
comments: true
categories: MEAN MongoDB ExpressJS AngularJS NodeJS 
tags: [translation, MEAN, full stack, Nodejs, Angularjs, Express, MongoDB]
keywords: MEAN, 全栈开发, Nodejs, Angularjs, Express, MongoDB  
---  
本系列教程的主要目的是尽可能清楚地阐释如何使AngularJS与Node.js、Express.js和MongoDB实现的后台服务连接 -- 这套技术栈也被称为MEAN stack（M=MongoDB, E = Express.js, A = AngularJS, N = Node.js）。让我们从AngularJS开始。  

#第一部分：AngularJS  

我们将会在一个单独的HTML文件中构建所有的示例，它将嵌入javascript，同时为了简化，我们并不关心样式/CSS。我们将在后续教程中讨论如何利用angularJS模块来分解代码，加入测试以及样式。  
<!--more-->

**Angular.js是什么？**  
{% img http://dl.iteye.com/upload/picture/pic/133659/9d9848ea-e94e-322c-9d73-f7d9c072146f.jpg %}  
Angular.js是一个MVW（模型-视图-随便什么）开源的JavaScript Web框架，使得创建单页应用（SPA）和数据驱动的应用更加方便快捷。  
  
##背景简介  
**AngularJS vs jQuery vs BackbonesJS vs EmberJS**  
AngularJS非常适用于构建可测试的单页应用（SPA），以及数据驱动的CRUD应用。  

AngularJS的格言是："HTML enhanced for web apps（为网络应用增强HTML）！"。它扩展了标准HTML标签和属性，用以绑定事件和数据。它与其他js库诸如jQuery, Backbones.JS和Ember.js等采用了不同的处理方法——它们更接近于“Unobtrusive JavaScript（译者注：此词叫法不一，为防止误解保留原文。[了解更多](http://www.thinksaas.cn/group/topic/285570/)）”。  

传统的Unobtrusive JavaScript的处理方式是，通过ID或元素中的class获取将要操作的元素，而不是通过在这些元素上添加事件处理器。这样做的好处是使得结构（HTML）和行为（Javascript）分离。然而，它对降低代码复杂度和增强可读性并没有帮助。  

随着时间的发展前端技术也在演变。让我们来看一下AngularJS是如何尝试降低代码复杂读和增强可读性的。  

- **单元测试**：通常来说，当DOM的操作与业务逻辑结合在一起时，JavaScript是非常难以进行单元测试的（例如基于jQuery的代码）。AngularJS将DOM的操作保持在HTML内部，与业务逻辑相分离。数据和依赖在需要的时候通过$inject注入。
- **DOM操作**：将DOM操作与应用逻辑解耦。
- **单页应用（SPA）**：AngularJS是用于创建单页应用的不二之选。
- **全局命名空间**：表达式和方法定义在controllers中并限制在其范围内，因此它们并不会污染全局命名空间。
- **数据模型**：普通老式Java对象（POJO）。
- **写更少的代码**：AngularJS的主要特性诸如指令（directives），过滤器（filters）以及自动数据绑定减少了代码量。
- AngularJS提供了编写模块化的代码和依赖管理的解决方案。  

##AngularJS主要组件  

###AngularJS指令  

关于AngularJS，你需要了解的第一个概念就是指令（directives）。

**Directives**是指HTML标记扩展，通常以属性、元素名称、CSS类甚至HTML注释等形式存在。当AngularJS框架被加载之后，ng-app指令中的一切 -- 数据、事件和DOM转换将与该指令绑定。

请注意，在下面的例子中有两个指令：ng-app和ng-model。  

{% coderay lang:HTML linenos:true Hello World in AngularJS %}  
<html ng-app>
<head>
  <title>Hello World in AngularJS</title>
</head>
<body>

<input ng-model="name"> Hello { { name } }

<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.25/angular.min.js"></script>
</body>
</html>
{% endcoderay %}  

让我们了解一下主要的内嵌指令：  
  
- **ng-app**: 是一个用于引导AngularJS的指令，它同时把调用者元素指派为根。通常置于<html>或<body>标签上  
- **ng-model**: 是一个用于绑定如input、select、checkboxes、textarea或其他定制页面元素到一个称为$scope的属性上的指令。有关$scope和数据绑定将在下一节中详细介绍。现在我们只需要记住文本框的值被绑定到 \{\{ name \}\}上  
- **\{\{ name \}\}**:  \{\{ \}\} 是绑定模型到HTML页面元素的一种方式。在上面的示例中ng-model 名字被绑定到占位符\{\{ name \}\}。  

你可能在猜想添加这个指令是否会使HTML验证器产生警告，提示存在未知的或非标准化的属性 —— 的确如此。然而，这可以通过在每个Angular.js指令前添加 'data-'前缀并且作为属性、类或者注释而不是作为元素来使用加以解决。让我们看看下面的示例，同时来创建自己的指令：  

{% coderay lang:HTML linenos:true Directive types: elements, attributes, comments and classes %}  
  <hello>Element</hello> Element
  <div data-hello>Attribute</div> data-Attribute
  <div hello>Attribute</div> Attribute
  <!-- directive: hello --> Comment
  <p class="hello"></p> Class
{% endcoderay %}  

{% coderay lang:JavaScript linenos:true Custom AngularJS directives %}  
var app = angular.module('app', []);

app.directive('hello', [function () {
  return {
    restrict: 'CEMA', // C: class, E: element, M: comments, A: attributes
    replace: true, // replaces original content with template
    template: '<span><br>Hello</span>'
  }
}]);
{% endcoderay %}  

如果你对指令的其他选项感兴趣，可以阅读 [A Practical Guide to AngularJS Directives](http://www.sitepoint.com/practical-guide-angularjs-directives/)

###AngularJS数据绑定  
数据绑定是AngularJS的特性之一，它使得模型数据可以与HTML自动同步。这非常了不起因为模型是“唯一的真相来源”同时你不需要关心如何去更新它。这里有一张来自docs.angularjs.org的图：  
{% img http://dl.iteye.com/upload/picture/pic/133665/3d238e26-a15f-3865-b217-1e8723407f67.png %}  

无论什么时候HTML发生变化时，模型将会被更新；同时无论什么时候模型被更新时，它将会反映到HTML上。  

###AngularJS作用域  
$scope是一个包含了HTML绑定的所有数据的对象。它是javascript代码(controllers)与视图(HTML)之间的胶水。附加在$scope上的所有数据，将会被AngularJS自动地"$watch"以及更新。  

作用域可以绑定到javascript函数上。你也可以使用多个$scope，或者从外部$scope继承。在控制器一节中我们将继续深入讨论。  

###AngularJS控制器  
AngularJS控制器是用于"控制"包含了特定DOM元素的区域的代码段。它们封装了行为、回调函数并且把$scope模型和视图粘合起来。让我们通过下面示例代码来加深对此概念的理解：  

{% coderay lang:HTML linenos:true AngularJS Controller Example %}  
<body ng-controller="TodoController">
  <ul>
    <li ng-repeat="todo in todos">
      <input type="checkbox" ng-model="todo.completed">
      { { todo.name } }
    </li>
  </ul>

  <script>
    function TodoController($scope){
      $scope.todos = [
        { name: 'Master HTML/CSS/Javascript', completed: true },
        { name: 'Learn AngularJS', completed: false },
        { name: 'Build NodeJS backend', completed: false },
        { name: 'Get started with ExpressJS', completed: false },
        { name: 'Setup MongoDB database', completed: false },
        { name: 'Be awesome!', completed: false },
      ]
    }
  </script>
</body>
{% endcoderay %} 

你可能会注意到这些新朋友：ng-controller, ng-repeat和$scope  

- **ng-controller** 是一个指令，它告诉angular对于特定的视图应当使用哪个控制器。每当AngularJS加载时，它读取该指令的参数（示例中是"TodoController"）。然后，它在javascript对象(POJO)中查找具有相同名字的函数，或者查找与angular.controller匹配的名字。
- **$scope**，正如之前提到的，$scope是粘合控制器中数据模型和视图的胶水。看一下我们的"TodoController"，它有一个名为$scope的参数。AngularJS将会传递（注入）那个参数，以及与其相关的所有数据，从而使其在视图中可用。示例中主要是todos这个对象数组。
- **ng-repeat**，正如其名，它将会在声明该指令的地方“重复”输出元素及其子元素。对于示例，它将会遍历$scope.todos数组中的每一个元素。
- **ng-model**，请注意复选框被绑定到todo.completed属性。如果todo.completed为true，复选框将被自动选中，反之亦然。

###AngularJS模块  
模块是用来封装应用中不同部分（指令、控制器、工厂……）的一种方式，可以方便地在其他地方重用。这里有一个通过模块来重写我们之前的控制器的实例：  

{% coderay lang:JavaScript linenos:true Custom AngularJS directives %}  
angular.module('app', [])
  .controller('TodoController', ['$scope', function ($scope) {
    $scope.todos = [
      { title: 'Learn Javascript', completed: true },
      { title: 'Learn Angular.js', completed: false },
      { title: 'Love this tutorial', completed: true },
      { title: 'Learn Javascript design patterns', completed: false },
      { title: 'Build Node.js backend', completed: false },
    ];
  }]);
{% endcoderay %}  

使用模块可以带来许多好处，比如可以以任意顺序加载模块，并行加载依赖，测试可以只加载需要的模块以加快速度，使得依赖关系更加清晰。  

###AngularJS模板  
模板同时包含HTML与Angular元素（指令、标记、过滤器或表单控件）。它们可以被缓存或者被通过id引用。  

这里有一个示例：  

{% coderay lang:HTML linenos:true %}  
  <script type="text/ng-template" id="/todos.html">
    <ul>
      <li ng-repeat="todo in todos">
        <input type="checkbox" ng-model="todo.completed">
       
      </li>
    </ul>
  </script>
{% endcoderay %} 

代码看起来眼熟吗？；）

请注意script标签中的类型属性为“text/ng-template”。

###AngularJS路由（ngRoutes）  
ngRoutes模块使得我们能够更改我们在应用中看到的东西，基于URL(路由)。通常，它使用模板向应用中注入HTML。  

它并没有包含在AngularJS核心模块里面，我们必须将其作为一个依赖引入。我们可以从Google CDN上获取它：  

`<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.25/angular-route.min.js"></script>`

**新功能**：向todo任务中添加记录。让我们从路由开始。  

{% coderay lang:JavaScript linenos:true Custom AngularJS directives %}  

angular.module('app', ['ngRoute'])
  .config(['$routeProvider', function ($routeProvider) {
    $routeProvider
      .when('/', {
        templateUrl: '/todos.html',
        controller: 'TodoController'
      });
  }]);

{% endcoderay %}  

- 首先请注意我们将ng-controller="TodoController"从body标签移除了。现在控制器将通过路由被调用。
- ngView是$routeProvider所使用的指令，用于渲染HTML。每次URL变化时，一个新的HTML模板和控制器将会被注入ngView。

###AngularJS服务（工厂）  
请注意目前暂时无法创建第二个控制器并共享$scope.todos。当我们使用服务时，就方便了许多。服务是我们将数据依赖注入到控制器中的一种方式，他们通过工厂生成。我们看实例：

{% coderay lang:JavaScript linenos:true Custom AngularJS directives %}  
 angular.module('app', ['ngRoute'])

    .factory('Todos', function(){
      return [
        { name: 'AngularJS Directives', completed: true },
        { name: 'Data binding', completed: true },
        { name: '$scope', completed: true },
        { name: 'Controllers and Modules', completed: true },
        { name: 'Templates and routes', completed: true },
        { name: 'Filters and Services', completed: false },
        { name: 'Get started with Node/ExpressJS', completed: false },
        { name: 'Setup MongoDB database', completed: false },
        { name: 'Be awesome!', completed: false },
      ];
    })

    .controller('TodoController', ['$scope', 'Todos', function ($scope, Todos) {
      $scope.todos = Todos;
    }])
{% endcoderay %}  

我们现在把数据依赖“Todo”注入到控制器中。通过这种方式我们可以在需要的任何控制器或模块中重用这些数据。不仅可用于静态数据（如上例中的数组），也可以用于通过$http或$resource从服务器获得的数据。  

我们想要在点击的时候展示任务的细节。为了实现这个需求，我们需要添加同样使用这个服务的第二个控制器、模板以及路由。  

发生的动作如下：  
1. 我们在HTML标签中创建了第二个模板 '/todoDetails.html', 它包含了想要展示的待做事项细节  
2. 同样，在我们前一个模板 '/todos.html' 我们想要有一个链接指向 todo 细节。我们使用了$index，它是ng-repeat中相应想的序号。  
3. 在JS标签中，我们创建了一个新的$routeProvider，它指向一个新的控制器 'TodoDetailCtrl' 与#1中新创建的模板。在控制器中：id参数通过$routeParams来访问。  
4. 创建新的控制器 'TodoDetailCtrl'并且注入$scope, Todos（工厂）等依赖，以及包含了id参数的$routeParams  
5. 在新的控制器中设置$scope。我们将通过第二步中设置的id选择唯一的一个待办事项，而不是整个待办事项数组。  

注意：在codepen中你无法看到URL。如果你想要看到它的变化，你可以在 这里 [ url ]下载整个示例。  

###AngularJS过滤器  

过滤器使你可以格式化并转换双括号里表达式的输出。AngularJS内置了许多有用的过滤器。  

**内置过滤器**：
  
- **filter**: 在一个数组中查找给定的字符串，返回匹配值。  
- **Number**: 在每个千位添加逗号分隔，设置两位小数  
- **Currency**：与Number类似，但是最前面添加美元$符号  
- **Date**：取一个Unix时间戳（例如：1288323623006）或日期字符串作为输入，用指定的格式输出（例如：‘longDate’或'yyyy'用以表示四位数字的年）。完整列表：https://docs.angularjs.org/api/ng/filter/date  
- **JSON**：把javascript对象转换为JSON字符串。  
- **lowercase/uppercase**: 把字符串转换为小写/大写  
- **limitTo**: 从数组中展示的元素数量  
- **orderBy**: 利用指定的key排好序的对象数组  

**注意**你可以将多个过滤器组合为过滤器链使用，也可以定义你自己的过滤器  

**新功能**：通过名字搜索代办任务。让我们使用过滤器来解决这个问题。

{% coderay lang:HTML linenos:true %}  
  <script type="text/ng-template" id="/todos.html">
    Search: <input type="text" ng-model="search.name">
    <ul>
      <li ng-repeat="todo in todos | filter: search">
        <input type="checkbox" ng-model="todo.completed">
        <a href="#/"></a>
      </li>
    </ul>
  </script>
{% endcoderay %}  

请注意我们在ng-model中使用search.name用于搜索。这将会把搜索限制在'name'属性，search.notes将会只在notes里搜索。猜猜不指定属性只使用‘search’将会做些什么？没错！它会在所有的属性中搜索。  

##下一步？  
祝贺你！你已经完成了第一部分。我们将会使用目前所学构建一些东西。在后续文章中我们将会使用NodeJS和MongoDB搭建后台，连接到AngularJS上来提供一个具备完整功能的CRUD应用。  

第二部分 - [MEAN全栈开发：使用NodeJS和MongoDB创建REST服务](http://gocwind.com/blog/2015/06/09/creating-a-restful-api-tutorial-with-nodejs-and-mongodb/)  

{% img left http://dl.iteye.com/upload/picture/pic/133663/abf21050-a55a-347e-9e0f-501586af10c7.png %}  
{% img http://dl.iteye.com/upload/picture/pic/133661/636d3c41-5c03-33a4-af9f-a539fc997fcd.png %}  

第三部分 - [MEAN全栈开发： 前后端整合](http://gocwind.com/blog/2015/06/09/mean-stack-tutorial-mongodb-expressjs-angularjs-nodejs/)  

如果想要学习BackboneJS，可以访问我的[BackboneJS教程](http://adrianmejia.com/blog/categories/backbonejs)  

**ng-test**  

现在是时候测试一下你所学的知识了。测试驱动学习（TDL）;). 挑战如下：使用你最喜欢的编辑器打开[这个文件](https://gist.githubusercontent.com/amejiarosario/26751cb85d088fd59c28/raw/c2dde0797c8d47d359c2137fc9a15a9228c272ca/index.html)。拷贝样板代码，构建我们前面例子中完整的应用。当然，当你卡住时你可以不时地参考一下。  

把这个文件下载为：index.html  

**ng-solution**  

这里是完整的[应用展示](https://cdn.rawgit.com/amejiarosario/068143b53e54db43ef38/raw/b703b591bc84f2d59a2a483169294e2fb232419d/ngTodo.html#/)。  

[项目代码@Github](https://github.com/cwind001/CwindJsLab/tree/master/todoAPIjs)  

原文链接：[AngularJS Tutorial for Beginners With NodeJS ExpressJS and MongoDB (Part I)](http://adrianmejia.com/blog/2014/09/28/angularjs-tutorial-for-beginners-with-nodejs-expressjs-and-mongodb/)