---
title: Restful API 设计规范( Restful API Design References )
description: 
categories:
 - Restful API
tags:
 - Restful API
---

> REST（英文：Representational State Transfer，又称具象状态传输）是Roy Thomas Fielding博士于2000年在他的博士论文中提出来的一种万维网软件架构风格，目的是便于不同软件/程序在网络（例如互联网）中互相传递信息。

<!-- more -->

> 本文内容几乎全部来自于网络，经过整理归纳形成记录。文章末尾均有出处，如有遗漏，可联系本人。

> * API的就是程序员的UI，和其他UI一样，你必须仔细考虑它的用户体验！
>
> * RESTful是理想化产物, 目前还没看到一家开放平台的API完全按照restful协议来，取其精神即可。

**REST**（英文：**Representational State Transfer**，又称**具象状态传输**）是Roy Thomas Fielding博士于2000年在他的博士论文中提出来的一种万维网软件架构风格，目的是便于不同软件/程序在网络（例如互联网）中互相传递信息。

目前在三种主流的Web服务实现方案中，因为REST模式与复杂的SOAP和XML-RPC相比更加简洁，越来越多的web服务开始采用REST风格设计和实现。例如，Amazon.com提供接近REST风格的Web服务运行图书查询；雅虎提供的Web服务也是REST风格的。

## REST的优点

* 可更高效利用缓存来提高响应速度
* 通讯本身的无状态性可以让不同的服务器的处理一系列请求中的不同请求，提高服务器的扩展性
* 浏览器即可作为客户端，简化软件需求
* 相对于其他叠加在HTTP协议之上的机制，REST的软件依赖性更小
* 不需要额外的资源发现机制
* 在软件技术演进中的长期的兼容性更好

## 特点及标准

需要注意的是，REST是设计风格而不是标准。REST通常基于使用HTTP，URI，和XML以及HTML这些现有的广泛流行的协议和标准。

* 每一个URI代表一种资源；
* 对资源的操作包括获取、创建、修改和删除资源，这些操作正好对应HTTP协议提供的GET、POST、PUT和DELETE方法。
* 通过操作资源的表现形式来操作资源。
* 连接协议具有无状态性（**Stateless**）
* 资源的表现形式则是JSON、XML或HTML等，取决于读者是机器还是人，是消费web服务的客户软件还是web浏览器。当然也可以是任何其他的格式。

## 关于版本（Version）

在API上加入版本信息可以有效的防止用户访问已经更新了的API，同时也能让不同主要版本之间平稳过渡。关于是否将版本信息放入url还是放入请求头有过争论：[API version should be included in the URL or in a header.](https://stackoverflow.com/questions/389169/best-practices-for-API-versioning) 学术界说它应该放到header里面去，但是如果放到url里面我们就可以跨版本的访问资源了。（参考openstack）。

#### 观点：
> 一个好的RESTful API会在URL中包含版本信息。另一种比较常见的方案是在请求头里面保持版本信息，在请求头里面包含版本信息远没有放在URL里面来的容易。

#### 例子：

    https://example.org/api/v1/*
    
    https://api.example.com/v1/*


## 关于资源（Endpoints）

从API用户的角度来看，”资源“应该是个名词。即使内部数据模型和资源已经有了很好的对应，API设计的时候仍然不需要把它们一对一的都暴露出来。这里的关键是隐藏内部资源，暴露必需的外部资源。

尽量使用复数描述资源使得URL更加规整。会让API使用者更加容易理解，对开发者来说也更容易实现。

构建一个虚构的API来展现几个不同的动物园，每一个动物园又包含很多动物，员工和每个动物的物种，可能会有如下的Endpoints信息：

> * https://api.example.com/v1/**zoos**
>
> * https://api.example.com/v1/**animals**
>
> * https://api.example.com/v1/**animal_types**
>
> * https://api.example.com/v1/**employees**

针对每一个Endpoints来说，可能列出所有可行的HTTP动词和Endpoints的组合。如下所示:

> * GET /zoos: List all Zoos (ID and Name, not too much detail)
> 
> * POST /zoos: Create a new Zoo
> 
> * GET /zoos/ZID: Retrieve an entire Zoo object
> 
> * PUT /zoos/ZID: Update a Zoo (entire object)
> 
> * PATCH /zoos/ZID: Update a Zoo (partial object)
> 
> * DELETE /zoos/ZID: Delete a Zoo
> 
> * GET /zoos/ZID/animals: Retrieve a listing of Animals (ID and Name).
> 
> * GET /animals: List all Animals (ID and Name).
> 
> * POST /animals: Create a new Animal
> 
> * GET /animals/AID: Retrieve an Animal object
> 
> * PUT /animals/AID: Update an Animal (entire object)
> 
> * PATCH /animals/AID: Update an Animal (partial object)
> 
> * GET /animal_types: Retrieve a listing (ID and Name) of all Animal Types
> 
> * GET /animal_types/ATID: Retrieve an entire Animal Type object
> 
> * GET /employees: Retrieve an entire list of Employees
> 
> * GET /employees/EID: Retreive a specific Employee
> 
> * GET /zoos/ZID/employees: Retrieve a listing of Employees (ID and Name) who work at this Zoo
> 
> * POST /employees: Create a new Employee
> 
> * POST /zoos/ZID/employees: Hire an Employee at a specific Zoo
> 
> * DELETE /zoos/ZID/employees/EID: Fire an Employee from a specific Zoo

#### 关联资源使用的另一个例子：

> * GET /tickets/12/messages- Retrieves list of messages for ticket #12
>
> * GET /tickets/12/messages/5- Retrieves message #5 for ticket #12
>
> * POST /tickets/12/messages- Creates a new message in ticket #12
>
> * PUT /tickets/12/messages/5- Updates message #5 for ticket #12
>
> * PATCH /tickets/12/messages/5- Partially updates message #5 for ticket #12
>
> * DELETE /tickets/12/messages/5- Deletes message #5 for ticket #12
>
> 其中，如果这种关联和资源独立，那么我们可以在资源的输出表示中保存相应资源的endpoint。然后API的使用者就可以通过点击链接找到相关的资源。如果关联和资源联系紧密。资源的输出表示就应该直接保存相应资源信息。（例如这里如果message资源是独立存在的，那么上面 GET /tickets/12/messages就会返回相应message的链接；相反的如果message不独立存在，他和ticket依附存在，则上面的API调用返回直接返回message信息）

#### 特殊资源类型

理想的 REST 世界，一切事物都抽象为资源，一切操作都抽象为增删改查。然而，所有事物与操作都可以很容易的按照这个规则作抽象吗？让我们看看这个例子：

检入和检出一个文档 ---- 这个时候，我们要处理的资源是一个文档，然而增删改查似乎都无法与“检入检出”这个动作进行对应。当然，我们可以在文档资源中，设计一个检入检出状态的元素，通过编辑文档资源来实现。但是，这种设计从自然语义上看，并不是很贴切；并且增加了资源编辑操作的复杂度。

如果我们来定义一个新的集合 ----“我检出的文档”，用创建一个集合资源来对应检出（创建一个文档锁），用删除一个集合资源来对应检入（删除一个文档锁）， 是不是逻辑上可以变得更加清楚？

在 REST 这个以名词为核心的构架结构中，当你遇到一些动词特性比较强的操作，而又很难用原始资源的增删改查来匹配的时候，不妨换个思路， 通过引入新的逻辑资源集合的方式， 来进行 API 的设计与规划。

## 关于动词(Verbs)

显然你了解GET和POST请求。当你用浏览器去访问不同页面的时候，这两个是最常见的请求。POST术语如此流行以至于开始侵扰通俗用语。即使是那些不知道互联网如何工作的人们也能“post”一些东西到朋友的Facebook墙上。

这里至少有四个半非常重要的HTTP动词需要你知道。我之所以说“半个”的意思是PATCH这个动词非常类似于PUT，并且它们俩也常常被开发者绑定到同一个API上。

> * GET (选择)：从服务器上获取一个具体的资源或者一个资源列表。
> 
> * POST （创建）： 在服务器上创建一个新的资源。
> 
> * PUT （更新）：以整体的方式更新服务器上的一个资源。
> 
> * PATCH （更新）：只更新服务器上一个资源的一个属性。
> 
> * DELETE （删除）：删除服务器上的一个资源。

还有两个不常用的HTTP动词：

> * HEAD ： 获取一个资源的元数据，如数据的哈希值或最后的更新时间。
>
> * OPTIONS：获取客户端能对资源做什么操作的信息。

一个好的RESTful API只允许第三方调用者使用这四个半HTTP动词进行数据交互，并且在URL段里面不出现任何其他的动词。

一般来说，GET请求可以被浏览器缓存（通常也是这样的）。例如，缓存请求头用于第二次用户的POST请求。HEAD请求是基于一个无响应体的GET请求，并且也可以被缓存的。

## 无状态性（Stateless）

HTTP 协议从本质上说是一种无状态的协议，客户端发出的 HTTP 请求之间可以相互隔离，不存在相互的状态依赖。

基于 HTTP 的 ROA，以非常自然的方式来实现无状态服务请求处理逻辑。对于分布式的应用而言，任意给定的两个服务请求 Request 1 与 Request 2, 由于它们之间并没有相互之间的状态依赖，就不需要对它们进行相互协作处理，其结果是：Request 1 与 Request 2 可以在任何的服务器上执行，这样的应用很容易在服务器端支持负载平衡 (load-balance)。

## 结果过滤，排序，搜索

**过滤**：为所有提供过滤功能的接口提供统一的参数。例如：你想限制get /tickets 的返回结果:只返回那些open状态的ticket–get /tickektsstate=open这里的state就是过滤参数。

**排序**：和过滤一样，一个好的排序参数应该能够描述排序规则，而不业务相关。复杂的排序规则应该通过组合实现：

> * GET /ticketssort=-priority- Retrieves a list of tickets in descending order of priority
>
> * GET /ticketssort=-priority,created_at- Retrieves a list of tickets in descending order of priority. Within a specific priority, older tickets are ordered first
>
> 这里第二条查询中，排序规则有多个rule以逗号间隔组合而成。

**搜索**：有些时候简单的排序是不够的。我们可以使用搜索技术（ElasticSearch和Lucene）来实现（依旧可以作为url的参数）。

> * GET /ticketsq=return&state=open&sort=-priority,created_at- Retrieve the highest priority open tickets mentioning the word ‘return’

对于经常使用的搜索查询，我们可以为他们设立别名,这样会让API更加优雅。例如：
> * GET /ticketsq=recently_closed -> get /tickets/recently_closed.


下面例子为Github的分页处理：

> * curl 'https://api.github.com/user/repos?page=2&per_page=100'

## 使用HTTP响应状态码

* 200	已创建，请求成功且服务器已创建了新的资源。
* 201	是否只显示处于警告状态的应用实例
* 301	重定向 , 请求的网页已被永久移动到新位置。服务器返回此响应时，会自动将请求者转到新位置。
* 302	重定向 , 请求的网页临时移动到新位置，但求者应继续使用原有位置来进行以后的请求。302 会自动将请求者转到不同的临时位置。
* 304	未修改，自从上次请求后，请求的网页未被修改过。服务器返回此响应时，不会返回网页内容。
* 400	错误请求 , 服务器不理解请求的语法。
* 401	未授权 , 请求要求进行身份验证。
* 403	已禁止 , 服务器拒绝请求。
* 404	未找到 , 服务器找不到请求的网页。
* 405	方法禁用 , 禁用请求中所指定的方法。
* 406	不接受 , 无法使用请求的内容特性来响应请求的网页。
* 408	请求超时 , 服务器等候请求时超时。
* 410	已删除 , 如果请求的资源已被永久删除，那么，服务器会返回此响应。
* 412	未满足前提条件 , 服务器未满足请求者在请求中设置的其中一个前提条件。
* 415	不支持的媒体类型 , 请求的格式不受请求页面的支持。
* 500	内部服务器错误。

> 更多：[HTTP Status Codes](https://httpstatuses.com/)

## 速度（率）限制（Rate Limiting）

为了避免请求泛滥，给API设置速度限制很重要。为此 RFC 6585 引入了HTTP状态码429（too many requests）。加入速度设置之后，应该提示用户，至于如何提示标准上没有说明，不过流行的方法是使用HTTP的返回头。

下面是几个必须的返回头（依照twitter的命名规则）：

> X-Rate-Limit-Limit :当前时间段允许的并发请求数
>
> X-Rate-Limit-Remaining:当前时间段保留的请求数。
>
> X-Rate-Limit-Reset:当前时间段剩余秒数

为什么使用当前时间段剩余秒数而不是时间戳？

时间戳保存的信息很多，但是也包含了很多不必要的信息，用户只需要知道还剩几秒就可以再发请求了这样也避免了clock skew问题。

有些API使用UNIX格式时间戳，我建议不要那么干。为什么？HTTP 已经规定了使用 RFC 1123 时间格式

## 提供标准的时间戳

提供默认的资源创建时间，更新时间 created_at and updated_at , 例如:

    {
      ...
      "created_at": "2012-01-01T12:00:00Z",
      "updated_at": "2012-01-01T13:00:00Z",
      ...
    }

这些时间戳可能不适用于某些资源，这种情况下可以忽略省去。

## 出错处理

就像html错误页面能够显示错误信息一样，API 也应该能返回可读的错误信息–它应该和一般的资源格式一致。API应该始终返回相应的状态码，以反映服务器或者请求的状态。API的错误码可以分为两部分，400系列和500系列，400系列表明客户端错误：如错误的请求格式等。500系列表示服务器错误。API应该至少将所有的400系列的错误以json形式返回。如果可能500系列的错误也应该如此。json格式的错误应该包含以下信息：一个有用的错误信息，一个唯一的错误码，以及任何可能的详细错误描述。如下：

    {
      "code" : 1234,
      "message" : "Something bad happened :-(",
      "description" : "More details about the error here"
    }

对PUT,POST,PATCH的输入的校验也应该返回相应的错误信息，例如：

    {
      "code" : 1024,
      "message" : "Validation Failed",
      "errors" : [
        {
          "code" : 5432,
          "field" : "first_name",
          "message" : "First name cannot have fancy characters"
        },
        {
           "code" : 5622,
           "field" : "password",
           "message" : "Password cannot be blank"
        }
      ]
    }

## 需要注意的细节与技巧

#### 更新和创建操作应该返回资源

PUT、POST、PATCH 操作在对资源进行操作的时候常常有一些副作用：例如created_at,updated_at 时间戳。为了防止用户多次的API调用（为了进行此次的更新操作），我们应该会返回更新的资源（updated representation.）例如：在POST操作以后，返回201 created 状态码，并且包含一个指向新资源的url作为返回头


#### 在post,put,patch上使用json作为输入

很多的API使用url编码格式：就像是url查询参数的格式一样：单纯的键值对。这种方法简单有效，但是也有自己的问题：它没有数据类型的概念。这使得程序不得不根据字符串解析出布尔和整数,而且还没有层次结构–虽然有一些关于层次结构信息的约定存在可是和本身就支持层次结构的json比较一下还是不很好用。

当然如果API本身就很简单，那么使用url格式的输入没什么问题。但对于复杂的API你应该使用json。或者干脆统一使用json。
注意使用json传输的时候，要求请求头里面加入：

> Content-Type：application/json

否则抛出415异常（unsupported media type）。

#### 批量操作

当用户需要更新多个资源的时候，你打算让开发者一次次的发送 HTTP 请求逐个更新吗？你可以考虑在设计 API 的时候允许客户同时创建或者更新多个资源。

#### 登录、注册和注销

> 登录实际是对应的资源是 session，因此GET /session # 获取会话信息 POST /session # 创建新的会话（登录） PUT /session # 更新会话信息 DELETE /session # 销毁当前会话（注销） 而注册对应的资源是user，api如下： GET /user/:id # 获取id用户的信息 POST /user # 创建新的用户（注册） PUT /user/:id # 更新id用户的信息 DELETE /user/:id # 删除id用户（注销
>
> **作者**：sz chen
> **链接**：https://www.zhihu.com/question/20346297/answer/127007546
> **来源**：知乎

#### RPC接口和Restful借口的比较

> RPC更偏向内部调用，REST更偏向外部调用。
>
> RPC好比与图灵机，任何需求都可以新开一个RPC接口，不用理会过去的接口，后端可以针对单一需求编写复杂逻辑去执行请求。这是一种非常直观的方式，缺点也很明显：接口设计上太过自由导致太容易设计出混乱的系统，对整个项目来说是可能有灾难性的后果。
>
>而REST好比lambda-calculus，resources的概念使得所有状态都有了统一的形式，状态转移也变得不再透明，直接好处是可以设计出非常优美的接口，非常便利地使用HTTP提供infrastructure，然而，对于复杂的需求，由于接口设计不像RPC那样随意，在REST里设计出这些接口，需要更好地照顾整个系统，尽可能地隔离好各个resources之间的关系，在客户端使用的时候也会有“反直觉”的感觉，但作为系统整体会更加健壮。
>
> 我的结论是，如果你的系统很简单，随便用哪个都好，用RPC更符合大多数人的直觉。如果你的系统很复杂，用RPC就要小心地去控制复杂度了，用REST反而会简单些。
>
> **作者**：jamesr
> **链接**：https://www.zhihu.com/question/28570307/answer/48294448
> **来源**：知乎

#### 最佳实践

* [Github Developer REST API v3](https://developer.github.com/v3/)

* [Enchant REST API](http://dev.enchant.com/api/v1)

#### 参考文章和文档

* [REST-维基百科，自由的百科全书](https://zh.wikipedia.org/zh-hans/REST)

* [Github Developer REST API v3](https://developer.github.com/v3/)

* [Enchant REST API](http://dev.enchant.com/api/v1)

* [Principles of good RESTful API Design](https://codeplanet.io/principles-good-restful-api-design/) (译：[好RESTful API的设计原则](http://www.cnblogs.com/moonz-wu/p/4211626.html))

* [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api) (译：[RESTful API 设计最佳实践](http://blog.jobbole.com/41233/))

***

> Update at 2017-09-07 12:35:00
