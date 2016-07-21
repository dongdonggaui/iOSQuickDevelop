# LoopBack - Getting Start

## 安装

LoopBack 是基于 Node 的 web API 开发框架，要开发 LoopBack 应用需要先安装 Node，然后通过以下命令安装 LoopBack 的开发工具 StrongLoop：

~~~bash
$ npm install -g strongloop
~~~

## 创建应用

安装好 StrongLoop 工具后使用以下命令可以创建一个有默认的目录结构的应用：

~~~bash
$ slc loopback
[?] What's the name of your application? hello-world
  create hello-world/
  info change the working directory to hello-world
  I'm all done. Running npm install for you to install
  the required dependencies.
  ... 
~~~

## 创建 models

使用 model 生成工具可以快速创建 model：

~~~bash
$ slc loopback:model
~~~

接下来 model 生成工具会通过一系列的问题让开发者设置 model 的相关信息：

~~~bash
[?] Enter the model name: person
[?] Select the data-source to attach person to: db (memory)
[?] Select model`s base class (PersistedModel)
[?] Expose person via the REST API? Yes
[?] Custom plural form (used to build REST URL): people
[?] Common model or server only? common
~~~

设置完基本信息之后 model 生成工具会引导给 model 设置属性：

~~~bash
Let's add some person properties now.
Enter an empty property name when done.
[?] Property name: firstname
[?] Property type: (Use arrow keys)
❯ string
  number
  boolean
  object
  array
  date
  buffer
  geopoint
  any
  (other)
[?] Required? (y/N) y
~~~

重复以上属性设置过程来设置所有 model 需要的属性，所有属性都添加完成之后，在引导提醒输入属性名称的时候通过直接输入 Enter 来结束属性的设置。

## 让应用跑起来

在本地环境下通过以下命令让应用跑起来：

~~~bash
$ node .
Browse your REST API at http://0.0.0.0:3000/explorer
Web server listening at: http://0.0.0.0:3000/
~~~

如果在生成环境，需要使用 `slc start` 来运行你的 app，这样会在本地启动一个 StrongLoop Process Manager 实例帮助管理你的 app。它还提供了 profile、monitor、control clustering 等功能。更多信息可以访问 [strong-pm.io](https://strong-pm.io)

## 访问 REST API

完成以上步骤之后，通过浏览器访问 <http://0.0.0.0:3000/explorer> 就可以看到一个内置的 API 浏览器：

![](http://loopback.io/images/helloworld-api-explorer.png)

更详细的使用说明可以参考 [Using the API Explorer](http://docs.strongloop.com/display/LB/Use+API+Explorer?_ga=1.143153405.979495713.1465783764)，更多教程请点击[Getting Started](http://docs.strongloop.com/display/LB/Getting+Started+with+LoopBack?_ga=1.145896572.979495713.1465783764)

## 连接数据库

### 创建 data source

使用 [Data source generator](https://docs.strongloop.com/display/LB/Data+source+generator) 创建 data source：

~~~bash
slc loopback:datasource
~~~

按照引导设置相关信息：

~~~bash
[?] Enter the data-source name:

[?] Select the connector for mysqlDS: (Use arrow keys)
  other
  In-memory db (supported by StrongLoop)
  MySQL (supported by StrongLoop)
  PostgreSQL (supported by StrongLoop)
  Oracle (supported by StrongLoop)
  Microsoft SQL (supported by StrongLoop)
  MongoDB (supported by StrongLoop)
(Move up and down to reveal more choices)
~~~

### 连接 data source 和 model

