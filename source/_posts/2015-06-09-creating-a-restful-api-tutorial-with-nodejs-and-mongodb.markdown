---
layout: post
title: "MEAN全栈开发：使用NodeJS和MongoDB创建REST服务"
date: 2015-06-09 21:30:19 +0800
updated: 2015-06-09 21:36:10 +0800
comments: true
categories: MEAN MongoDB ExpressJS AngularJS NodeJS 
tags: [translation, MEAN, full stack, Nodejs, Angularjs, Express, MongoDB]
keywords: MEAN, 全栈开发, Nodejs, Angularjs, Express, MongoDB  
---
本教程介绍如何使用Node.js (Express.js) 和MongoDB (mongoose) 创建REST服务。你可以参考本教程创建一个独立的后台服务，也可以回顾之前的[AngularJS](http://gocwind.com/blog/2015/06/05/angularjs-tutorial-for-beginners/)或是[BackboneJS](http://adrianmejia.com/blog/2012/09/11/backbone-dot-js-for-absolute-beginners-getting-started)教程来构建一个javascript客户端，来与我们将要构建的后台集成。  

{% img left http://dl.iteye.com/upload/picture/pic/133663/abf21050-a55a-347e-9e0f-501586af10c7.png %}  
{% img http://dl.iteye.com/upload/picture/pic/133661/636d3c41-5c03-33a4-af9f-a539fc997fcd.png %}  
<!--more-->  

#第二部分 使用NodeJS和MongoDB创建REST服务  

##RESTful API是什么  

REST指表述性状态转移。它是允许以统一的接口进行客户端-服务器通信的架构。REST是“无状态”、“可缓存”以及“幂等”的。幂等意味着多次调用与单次请求的结果相同。  

HTTP RESTful API 由以下要素组成：  

- HTTP方法，如GET，PUT，DELETE，PATCH，POST，……  
- 基本URL，如 http://gocwind.com/   
- URL路径，如 /blog/2014/10/01/creating-a-restful-api-tutorial-with-nodejs-and-mongodb/  
- 媒介类型，如html，JSON，XML，Microformats，Atom，Images……  

下表是我们将要实现的API摘要：  
<table>
<tbody>
<tr><td><strong>Resource(URI) </strong></td><td><strong>POST(创建) </strong></td><td><strong>GET(读取) </strong></td><td><strong>PUT(更新 )</strong></td><td><strong>DELETE(删除) </strong></td></tr>
<tr><td>/todos</td><td>创建新的任务 </td><td>列出所有任务  </td><td>N/A（更新全部）</td><td>N/A（删除全部） </td></tr>
<tr><td>/todos/1</td><td>错误 </td><td>显示ID为1的任务  </td><td>更新ID为1的任务  </td><td>删除ID为1的任务  </td></tr>
</tbody>
</table>
<br/>

**注意**：我们采用JSON格式。批量更新和批量删除并不安全，所以我们将不实现这两个接口。POST，GET，PUT，DELETE方法分别对应创建(CREATE)，查询(READ)，更新(UPDATE)，删除(DELETE)操作，即CRUD。  

##搭建开发环境  

MEAN技术栈的两个主要组件是NodeJS以及MongoDB。  
{% img http://dl.iteye.com/upload/picture/pic/133677/a1b8cfdb-27cc-3d4f-8494-7cde0cb41ff9.png %}  

注意：如果你已经安装了NodeJS，MongoDB(Mongoose)，ExpressJS并且分别对它们已经有所了解，你可以跳过下面一节。如果你想要回顾或了解以上的每个成员，请继续阅读。  

##NodeJS  
简言之，NodeJS是运行在服务器上，浏览器之外的JavaScript。    

安装NodeJS，可以访问[NodeJS官方网站](http://nodejs.org/)。如果你使用Mac和[brew](http://brew.sh/)你可以执行brew install nodejs，如果你使用ubuntu可以利用[nvm](https://github.com/creationix/nvm)来安装它。  

检查node版本和npm版本：  

{% coderay lang:javascript linenos:true %}  
node -v
# => v0.12.4

npm -v
# => 2.10.1
{% endcoderay %}  

##ExpressJS  

ExpressJS是运行在NodeJS上的Web应用框架。它可以用于构建Web应用或API服务（后文详述）。  

利用npm安装它：  

{% coderay lang:javascript linenos:true %} 
npm install -g express
{% endcoderay %}  

请注意`-g`选项。它将会把`express`安装供全局使用，并加入`PATH`环境变量，因此你可以在任何地方运行它。  

检查版本：  

{% coderay lang:javascript linenos:true %}   
express --version
# => 4.12.4
{% endcoderay %}  

##MongoDB  

MongoDB是一个面向文档的NoSQL数据库（可用于处理大数据）。它将数据以JSON格式存储，允许执行类似SQL的查询。
你可以参照[这篇文档](http://docs.mongodb.org/manual/installation/)来安装它。如果你使用Mac和brew，就可以简单执行：`brew install mongodb && mongod`。在ubuntu下则是 `sudo apt-get -y install mongodb`。   

检查版本：  

{% coderay lang:javascript linenos:true %}  
# Mac
mongod --version
# => db version v2.6.4
# => 2014-10-01T19:07:26.649-0400 git version: nogitversion

# Ubuntu
mongod --version
# => db version v2.0.4, pdfile version 4.5
# => Wed Oct  1 23:06:54 git version: nogitversion
{% endcoderay %}   

##理解MEAN技术栈  

经过以上几步你已经准备好了用于完成本教程的所有事情。简单地说，我们将会构建RESTful API，使得用户可以执行CRUD（创建-读取-更新-删除）操作，来处理数据库中的Todo任务。  

###Mongoose CRUD  

CRUD = Create-Read-Update-Delete (创建-读取-更新-删除)  

我们可以在控制台里使用Mongoose。在`todoAPIjs`目录，键入`node`来进入node CLI，然后：  

{% coderay lang:javascript linenos:true %} 
/* prompt> */ var mongoose = require('mongoose');

/* prompt> */ mongoose.connect('mongodb://localhost/test3');

/* prompt> */ var TodoSchema = new mongoose.Schema({
  name: String,
  completed: Boolean,
  note: String,
  updated_at: { type: Date, default: Date.now },
});

/* prompt> */ var Todo = mongoose.model('Todo', TodoSchema);
{% endcoderay %} 

**Mongoose 创建**  

{% coderay lang:javascript linenos:true %}  
/* prompt> */ var todo = new Todo({name: 'Master NodeJS', completed: false, note: 'Getting 
there...'});

/* prompt> */ todo.save(function(err){
    if(err)
        console.log(err);
    else
        console.log(todo);
});
{% endcoderay %}  

你可以创建对象，并利用`create`来进行保存：  

{% coderay lang:javascript linenos:true %}  
/* prompt> */ Todo.create({name: 'Master Javscript', completed: true, note: 'Getting better 
everyday'}, function(err, todo){
    if(err) console.log(err);
    else console.log(todo);
});
{% endcoderay %}  

**Mongoose 读取与查询**  

读取/查询数据有下列多种方式：  

- Model.find(conditions, [fields], [options], [callback])   
- Model.findById(id, [fields], [options], [callback])   
- Model.findOne(conditions, [fields], [options], [callback])

一些例子：  

{% coderay lang:javascript linenos:true Find all %}  
/* prompt> */ Todo.find(function (err, todos) {
  if (err) return console.error(err);
  console.log(todos)
});
{% endcoderay %}  

你也可以加入查询条件：  

{% coderay lang:javascript linenos:true Find with queries %}  
/* prompt> */ var callback = function (err, data) {
  if (err) return console.error(err);
  else console.log(data);
}

// Get all completed tasks
/* prompt> */ Todo.find({completed: true }, callback);

// Get all tasks ending with `JS`
/* prompt> */ Todo.find({name: /JS$/ }, callback);
{% endcoderay %} 

当然，也可以加入多个查询条件，例如：  

{% coderay lang:javascript linenos:true %}  
/* prompt> */ var oneYearAgo = new Date();
oneYearAgo.setYear(oneYearAgo.getFullYear() - 1);

// Get all tasks staring with `Master`, completed
/* prompt> */ Todo.find({name: /^Master/, completed: true }, callback);

// Get all tasks staring with `Master`, not completed and created from year ago to now...
/* prompt> */ Todo.find({name: /^Master/, completed: false }).where('updated_at').gt(oneYearAgo)
.exec(callback);
{% endcoderay %}  

**Mongoose 更新**  

每个模型都有一个`update`方法，可以接受多条数据的更新操作（用于批量更新，并不返回数据数组）。同时`findOneAndUpdate`方法可以用于更新单独一条数据并将该条数据返回。  

- Model.update(conditions, update, [options], [callback])   
- Model.findByIdAndUpdate(id, [update], [options], [callback])   
- Model.findOneAndUpdate([conditions], [update], [options], [callback])  

{% coderay lang:javascript linenos:true Todo.update and Todo.findOneAndUpdate %}  
// Model.update(conditions, update, [options], [callback])
// update `multi`ple tasks from complete false to true

/* prompt> */ Todo.update({ completed: false }, { completed: true }, { multi: true }, 
function (err, numberAffected, raw) {
  if (err) return handleError(err);
  console.log('The number of updated documents was %d', numberAffected);
  console.log('The raw response from Mongo was ', raw);
});

//Model.findOneAndUpdate([conditions], [update], [options], [callback])
/* prompt> */ Todo.findOneAndUpdate({name: /JS$/ }, {completed: false}, callback);
{% endcoderay %}  

**Mongoose 删除**  

mongoose的`update`与`remove` API非常相似，唯一的区别是并没有任何元素被返回。  

- Model.remove(conditions, [callback])   
- Model.findByIdAndRemove(id, [options], [callback])   
- Model.findOneAndRemove(conditions, [options], [callback])  

###ExpressJS与中间件

ExpressJS是一个完备的Web框架解决方案。它包括HTML模板解决方案（jade, ejs, handlebars, hogan.js）与CSS预编译器（less, stylus, compass）。在中间件层它能够处理：cookies, sessions, caching, CSRF, 压缩以及许多其他的功能。  

**中间件**是一组用于处理每个发往服务器的请求的软件栈。你可以使用任意数量的中间件，以串行方式一个接一个地处理请求。其中的一些可能用于改变请求输入，打印日志输出，添加数据并将其传递到处理链中的下一个中间件。  

中间件通过`app.use`被加载到ExpressJS栈，从而可以被任何方法或app.动词（如app.get, app.delete, app.post, app.update, ...）所使用。  

{% img http://dl.iteye.com/upload/picture/pic/133679/eb6d578c-b8fe-33ab-aad1-20728b95ffe3.png %}

假设我们想要打印每个请求的来源客户端的IP：  

{% coderay lang:javascript linenos:true Log the client IP on every request %}  
app.use(function (req, res, next) {
  var ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress;
  console.log('Client IP:', ip);
  next();
});
{% endcoderay %}  

你也可以指定路径，使得你的中间件在该路径生效： 
 
{% coderay lang:javascript linenos:true Middleware mounted on “/todos/:id” and log the request method %}  
app.use('/todos/:id', function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});
{% endcoderay %}  

最终，你可以使用app.get来捕捉相匹配路由的GET请求，在中间件链末端通过`response.send`来为该请求产生一个响应。让我们使用mongoose读取与查询一节中的函数来返回一条与参数id相匹配的用户数据。  

{% coderay lang:javascript linenos:true Middleware mounted on “/todos/:id” and returns %}  
app.get('/todos/:id', function (req, res, next) {
  Todo.findById(req.params.id, function(err, todo){
    if(err) res.send(err);
    res.json(todo);
  });
});
{% endcoderay %}  

请注意之前所有的中间件都调用了`next()`，除了这最后一个，因为它将把包含指定`todo`数据的响应（以JSON格式）发送给客户端。  

除了路由之外，你不必自己去开发各种功能的中间件。因为ExpressJS已经包含了许多常用的中间件。  

###Express 4.0 默认中间件  

- [morgan](https://github.com/expressjs/morgan): 日志处理  
- [body-parser](https://github.com/expressjs/body-parser): 解析请求体，从而可以访问请求体`req.body`中的参数。例如：`req.body.name`。  
- [cookie-parser](https://github.com/expressjs/cookie-parser): 解析cookies，从而可以访问cookies中的参数。例如：`req.cookies.name`。  
- [serve-favicon](https://github.com/expressjs/serve-favicon): 顾名思义，为路由/favicon.ico提供图标。它应该在其他任何路由/中间件之前被调用，从而避免不必要的解析。  

###其他Express中间件  
下列中间件并非内置，但了解一下很有益处。  

- [compression](https://github.com/expressjs/compression): 压缩所有请求。例：app.use(compression())  
- [session](https://github.com/expressjs/session): 创建会话。例：app.use(session({secret: 'Secr3t'}))  
- [method-override](https://github.com/expressjs/method-override): `app.use(methodOverride('_method'))`，以`_method`参数值来覆盖方法。例：`GET /resource/1?_method=DELETE`将会变为`DELETE /resource/1`  
- [response-time](https://github.com/expressjs/response-time): `app.use(responseTime())`向响应添加响应头`X-Response-Time`。  
- [errorhandler](https://github.com/expressjs/errorhandler): 当错误发生时，通过向客户端发送完整的错误堆栈来辅助开发。`app.use(errorhandler())`。一个最佳实践是在加载它之前检测环境：`process.env.NODE_ENV === 'development'`。  
- [vhost](https://github.com/expressjs/vhost): 允许你根据请求的`hostname`不同使用不同的中间件栈。例：`app.use(vhost('*.user.local', userapp))`以及`app.use(vhost('assets-*.example.com', staticapp))`，其中`userapp`与`staticapp`是有不同中间件栈的不同express实例。  
- [csrurf](https://github.com/expressjs/csurf): 使用`session`或`cookie-parser`在响应中添加token，起到防止跨站请求伪造（Cross-site request forgery, CSRF）的作用。例：`app.use(csrf())`。  
- [timeout](https://github.com/expressjs/timeout): 当程序执行时间超过预设值时终止程序。例：`app.use(timeout('5s'));`。你需要自定义一个中间件检查每一个请求`if(!req.timeout) next();`。  

##API 客户端（浏览器，Postman和curl）  
我知道你还没有创建路由，我们在下一节中将会创建。通过你创建的API，有三种方式来查询、改动或删除数据。  

###Curl  

{% coderay lang:javascript linenos:true Create tasks %} 
# Create task
curl -XPOST http://localhost:3000/todos -d 'name=Master%20Routes&completed=false&note=soon...'

# List tasks
curl -XGET http://localhost:3000/todos
{% endcoderay %}  

###浏览器和Postman  
当你打开浏览器并在地址栏输入`localhost:3000/todos`你将会看到所有的任务（当你实现了API之后）。然而，默认情况下你并不能POST命令。为了后面的测试，我们可以使用一个名为Postman[link]的Chrome插件。它使你能够很容易地使用所有的HTTP命令，如果需要添加参数时，选中`x-www-form-urlencoded`。（译者注：使用RestClient for Firefox一样很方便）。  

{% img http://dl.iteye.com/upload/picture/pic/133681/84a293ae-e910-37af-abae-194e60294aed.png %}

###网站和移动应用  
这很有可能是最主要的API调用者。你可以使用jQuery`$ajax`方便地与RESTful API交互，或者使用它的包装器--BackboneJS的Collections/models， AngularJS的`$http`或`$resource`，或许许多多其他的库/框架以及移动客户端。  

最后，我们来阐释如何使用AngularJS与API交互。  

{% img http://dl.iteye.com/upload/picture/pic/133683/1737b0c6-2c33-3ed9-89aa-9bc385dd89cd.png %}  
（图片来自CodeSchool）  

整合MEAN技术栈

##引导ExpressJS  

花了较大篇幅来了解Node CLI，MongoDB，Mongoose，工具以及中间件之后，让我们回到我们的express应用todoApp。现在我们创建路由并最终实现我们的RESTful API。  
通过`express -e todoApp`创建应用。安装所有依赖`cd todoApp && npm install`。运行该应用：`DEBUG=todoApp ./bin/www`；  

{% coderay lang:javascript linenos:true %} 
express -e todoApp
# =>   create : todoApp                  # app directory  
# =>   create : todoApp/package.json     # file containing all the dependencies
# =>   create : todoApp/app.js           # Entry point of the application: defines middleware, 
initialize database connections, routes and more.
# =>   create : todoApp/public           # all files contained here are accessible through to 
public (browser or API calls).
# =>   create : todoApp/public/javascripts
# =>   create : todoApp/public/images
# =>   create : todoApp/public/stylesheets
# =>   create : todoApp/public/stylesheets/style.css
# =>   create : todoApp/routes           # containes all the routes files
# =>   create : todoApp/routes/index.js
# =>   create : todoApp/routes/users.js
# =>   create : todoApp/views            # contains all the HTML templates
# =>   create : todoApp/views/index.ejs
# =>   create : todoApp/views/error.ejs
# =>   create : todoApp/bin              # contains executable files
# =>   create : todoApp/bin/www          # bootstrap the app: loads app.js, and set the port 
for the webserver.
# =>
# =>   install dependencies:
# =>     $ cd todoApp && npm install
# =>
# =>   run the app:
# =>     $ DEBUG=todoApp ./bin/www
{% endcoderay %}  

###将ExpressJS与MongoDB连接  
在上一节中你已经安装好了MongoDB，键入以下命令来启动它：  
`mongod`  
为NodeJS安装名为mongoose的MongoDB驱动：  
`npm install mongoose --save`  

注意`--save`参数，这将会把它加到`todoApp/package.json`里。  
接下来，你需要在`todoApp/app.js`里引入mongoose。  

{% coderay lang:javascript linenos:true %} 
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/todoApp', function(err) {
    if(err) {
        console.log('connection error', err);
    } else {
        console.log('connection successful');
    }
});
{% endcoderay %}  

现在，你可以运行`npm start`或者`./bin/www`，你将会注意到下面的信息：`connection successful`。看到了吗？很好！  

你可以查看[完整的代码](https://github.com/amejiarosario/todoAPIjs)， 或者截止目前我们所做的[改动](https://github.com/amejiarosario/todoAPIjs/commit/d3be6a287e8aff39ab862971da4f050d04e552a1)。  

###使用Mongoose创建Todo模型  

表演时间到！目前为止，上面所做的工作都是搭建环境和准备工作。现在我们开始专注于实现API。  
创建`models`目录以及`Todo.js`模型：  
`mkdir models  
touch models/Todo.js`

编辑models/Todo.js文件：  

{% coderay lang:javascript linenos:true %} 
var mongoose = require('mongoose');
var TodoSchema = new mongoose.Schema({
  name: String,
  completed: Boolean,
  note: String,
  updated_at: { type: Date, default: Date.now },
});
module.exports = mongoose.model('Todo', TodoSchema);
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/afc908027339b22f10de3b77518ac0728668d470)  

这里发生了什么？MongoDB难道不是无模式的吗？没错，它确实无模式并且很灵活，然而，很多情况下我们会想要使我们的数据保持一个一致的结构，从而方便验证，也方便我们的API/WebApp实际应用。Mongoose帮我们做了这些事情。  

我们可以使用下面的类型：  

- String  
- Boolean  
- Date  
- Array   
- Number   
- ObjectId   
- Mixed   
- Buffer  

###ExpressJS 路由  
我们将要实现以下API：  
<table>
<tbody>
<tr><td><strong>Resource(URI) </strong></td><td><strong>POST(创建) </strong></td><td><strong>GET(读取) </strong></td><td><strong>PUT(更新 )</strong></td><td><strong>DELETE(删除) </strong></td></tr>
<tr><td>/todos</td><td>创建新的任务 </td><td>列出所有任务  </td><td>N/A（更新全部）</td><td>N/A（删除全部） </td></tr>
<tr><td>/todos/:id </td><td>错误 </td><td>显示ID为:id的任务  </td><td>更新ID为:id的任务  </td><td>删除ID为:id的任务  </td></tr>
</tbody>
</table>
<br/>

建立路由：  

{% coderay lang:javascript linenos:true Create a new route called `todos.js` in the `routes` folder or rename `users.js` %}  
mv routes/users.js routes/todos.js
{% endcoderay %}  

在`app.js`中，添加新的`todos`路由，或者替换`./routes/users`为`./routes/todos`  

{% coderay lang:javascript linenos:true Adding todos routes %}
var todos = require('./routes/todos');
app.use('/todos', todos);
{% endcoderay %}  

搞定！现在返回编辑`routes/todos.js`。 

**查询： GET /todos**  
还记得Mongoose查询API吗？下面的例子显示如何在上下文中使用它：  

{% coderay lang:javascript linenos:true routes/todos.js %}
var express = require('express');
var router = express.Router();
var mongoose = require('mongoose');
var Todo = require('../models/Todo.js');
/* GET /todos listing. */
router.get('/', function(req, res, next) {
  Todo.find(function (err, todos) {
    if (err) return next(err);
    res.json(todos);
  });
});
module.exports = router;
{% endcoderay %}  

收获时间到！数据库里暂时没有任务记录，不过我们至少可以证明它能够正常工作：  

{% coderay lang:javascript linenos:true Testing all together %}
# Start database
mongod
# Start Webserver (in other terminal tab)
DEBUG=todoApp ./bin/www
# Test API (in other terminal tab)
curl localhost:3000/todos
# => []% 
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/54ab912ea9aa2b6633ae12816beb6e6c3d2702e6)

如果看到返回空数组[]则证明一切都准备就绪了。如果你看到错误，尝试回顾并确认没有遗漏每个步骤，或者在本贴子下面添加评论以寻求帮助。  

**创建： POST/ todos**  
回到`routes/todos.js`，我们将使用mongoose create[link]来实现用于创建的API。你能够在不参照下面例子的情况下尝试实现它吗？  

{% coderay lang:javascript linenos:true routes/todos.js (showing just create route) %}
/* POST /todos */
router.post('/', function(req, res, next) {
  Todo.create(req.body, function (err, post) {
    if (err) return next(err);
    res.json(post);
  });
});
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/28b60c4bf9c6d8b08c3351f725e17c7f40a077be)  

几点需要注意：  

- 这里我们使用`router.post`而不是`router.get`。  
- 你必须关掉并且重启服务：`DEBUG=todoApp ./bin/www`。强烈推荐使用`nodemon`以自动刷新。执行`npm install nodemon`，然后通过`nodemon`运行程序。  

**展示单条任务： GET /todos/:id**  
以下是一个使用`Todo.findeById`和`req.params`的快照。请注意`params`与路径中占位符名称相匹配。这里我们用的是`:id`。  

{% coderay lang:javascript linenos:true routes/todos.js (showing just show route) %}
/* GET /todos/id */
router.get('/:id', function(req, res, next) {
  Todo.findById(req.params.id, function (err, post) {
    if (err) return next(err);
    res.json(post);
  });
});
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/7d8bc67178a4f162858395845c076d9223926bf8)

通过*POSTMAN*，使用一个你已经创建的元素`_id`来进行测试。例如：`localhost:3000/todos/542d7d290a705126360ac635`。  

**更新： PUT /todos/:id**  
回到`routes/todos.js`，我们来实现用于更新任务的API。请回顾findByIdAndUpdate[link]方法，并尝试利用它来实现该API。  

{% coderay lang:javascript linenos:true routes/todos.js (showing just update route) %}
/* PUT /todos/:id */
router.put('/:id', function(req, res, next) {
  Todo.findByIdAndUpdate(req.params.id, req.body, function (err, post) {
    if (err) return next(err);
    res.json(post);
  });
});
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/00dafe491e0d0b59fa53e86d8c187c42d7824200)

同样请在*POSTMAN*中测试 :-)  

**删除： DELETE /todos/:id**  
终于轮到最后一个API了！几乎与`update`完全相同，使用`findByIdAndRemove`。  

{% coderay lang:javascript linenos:true routes/todos.js (showing just update route) %}
/* DELETE /todos/:id */
router.delete('/:id', function(req, res, next) {
  Todo.findByIdAndRemove(req.params.id, req.body, function (err, post) {
    if (err) return next(err);
    res.json(post);
  });
});
{% endcoderay %}  

[变更](https://github.com/amejiarosario/todoAPIjs/commit/cbf5366e2b4e1a683ed50d2148ed6a548616d3f8)  

上面的API都正常工作吗？非常好，你已经完成了我们教程的第二部分。如果有错误，请参照[完整代码](https://github.com/amejiarosario/todoAPIjs)。  

##下一步？  
将AngularJS与后台服务连接。  

- 第三部分 - [MEAN全栈开发：前后端整合](http://gocwind.com/blog/2015/06/09/mean-stack-tutorial-mongodb-expressjs-angularjs-nodejs/)    

{% img http://dl.iteye.com/upload/picture/pic/133685/7c4ef50f-b7cd-3334-a29e-816cfcea7eba.png %}  

**相关教程：**  

- 第一部分 - [MEAN全栈开发：AngularJS实战教程](http://gocwind.com/blog/2015/06/05/angularjs-tutorial-for-beginners/)  
- 第三部分 - [MEAN全栈开发：前后端整合](http://gocwind.com/blog/2015/06/09/mean-stack-tutorial-mongodb-expressjs-angularjs-nodejs/)  
- [BackboneJS教程](http://adrianmejia.com/blog/categories/backbonejs)    

原文链接：[Creating RESTful APIs With NodeJS and MongoDB Tutorial (Part II)](http://adrianmejia.com/blog/2014/10/01/creating-a-restful-api-tutorial-with-nodejs-and-mongodb/)