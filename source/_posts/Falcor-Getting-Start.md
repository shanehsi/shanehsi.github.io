title: Falcor Getting Start
date: 2016-03-16 10:54:11
tags:
---

对 Falcor 的期望是：

- 在 app state 和 backend service 之间做一个中间层
- 不要管理太多 data flow
- 不要侵入 view

参考文章：[What is Falcor?](https://netflix.github.io/falcor/starter/what-is-falcor.html)

Falcor is the innovative data platform that powers the Netflix UIs. Falcor allows you to model all your backend data as a single Virtual JSON object on your Node server. On the client you work with your remote JSON object using familiar JavaScript operations like get, set, and call. If you know your data, you know your API.

建模 所有的 backend data 到 一个单独的 Virutal JSON object，到你的 Node server（但是似乎 Java server 也有了）。

在客户端，使用熟悉的 JavaScript 操作，比如 `get`，`set`，`call`。API 是不需要学习成本的。只要了解数据。

Falcor is middleware. It is not a replacement for your application server, database, or MVC framework. Instead Falcor can be used to optimize communication between the layers of a new or existing application.

Falcor 是一个 middleware。

不是代替 application server，database，或者 MVC framework。

Falcor 是用来 优化 layers 之间的 communication。

## One Model Everywhere

Falcor allows you to model all of your backend data as a single JSON resource on the application server.

Falcor 可以允许你将所有的 backend data 在 application server 上建模成一个单独的 JSON resource。

{% asset_img falcor-network-diagram.png %}

Clients request subsets of this JSON resource on demand, just like they would from an in-memory JSON object. To retrieve values from the JSON resource on the server, the client passes the server JavaScript paths to each desired value within the JSON object. The server responds with a subset of the JSON object that contains only those values.

客户端会按需请求 JSON resource 的子集，就像是获取一个内存中的 JSON 对象。

客户端会通过参数：每个需要的值的 JavaScript paths 来请求服务端。

```js
/model.json?paths=["user.name", "user.surname", "user.address"]

GET /model.json?paths=["user.name", "user.surname", "user.address"]
{
  user: {
    name: "Frank",
    surname: "Underwood",
    address: "1600 Pennsylvania Avenue, Washington, DC"
  }
}
```

Exposing all data as a single URL allows clients to request all of the data they need in a single network request. This can reduce latency by avoiding sequential server round trips.

一个 endpoint 暴露所有的数据，可以：

- 通过避免后续的往返请求减少网络延迟

In order to ensure that your application server remains stateless, Falcor provides a specialized Router which routes requests for values within the JSON object to different backend services.

为了保证你的 **应用服务器** 保持无状态型，Falcor 提供了一个特殊的 **Router**，用来路由对不同的 backend services 的请求。

Instead of matching URLs, the Falcor Router matches one or more JavaScript paths.

Falcor Router 不是通过符合 URLs，而是符合 JavaScript paths：

```js
var router = new Router([
  {
    // matches user.name or user.surname or user.address
    route: "user['name','surname','address']",
    get(pathSet) {
      // pathSet could be ["user", ["name"]], ["user", ["name", "surname"]], ["user", ["surname", "address"]] and so on...
      userService.
        getUser(getUserID()).
        then(function(user) {
          return pathSet[1].
            map(function(userKey) {
              // return response for each individual requested path
              return {
                path: ["user", userKey],
                value: user[userKey]
              };
            });
         });
    }
  }
]);

```

看出来了吗？这个 `get` 类似于 GraphQL 的 resolve 方法。

pathSet 是可能的路径集合。

The Falcor Router allows you to expose a single JSON model to the client, while giving you the flexibility to store your data anywhere.

只是对 backend data 做建模，最终暴露一个 JSON model 给客户端。

但也保证了灵活性来在其他地方存储 data。

{% asset_img services-diagram.png %}

## The Data is the API

You do not need to learn a complicated service layer to work with your data when you are using Falcor. If you know your data, you know your API. Falcor allows you to work with remote data the same way you work with local data, using JavaScript paths and operations. The primary difference is that Falcor’s client API is asynchronous.

你不需要学习一个复杂的 service layer 来处理 data。没有新增的语义，完全是 JavaScript Object 的 API。唯一的区别就是，Falcor's client API 是异步的。

Here’s an example retrieving the surname of a user directly from an in-memory JSON object using a simple JavaScript path.

```js
var model = {
  user: {
    name: "Frank",
    surname: "Underwood",
    address: "1600 Pennsylvania Avenue, Washington, DC"
  }
};

// prints "Underwood"
console.log(model.user.surname);
```

Applications that use Falcor do not work directly with JSON data. Instead they work with their JSON data indirectly using a Falcor Model. The Falcor Model allows you to use familiar idioms like JavaScript paths and JavaScript operations to work with your data. The primary difference between working with JSON data directly and using a Falcor Model, is that a Falcor Model has an asynchronous API.

使用 Falcor 的应用并不会直接和 JSON data 打交道，而是通过 Falcor Model。

Falcor Model 提供了类似于 JavaScript paths 和 operations 的 API。只是他是异步的。

Let’s rewrite the code above using a Falcor Model instead of working with the JSON directly:

```js
var model = new falcor.Model({
  cache: {
    user: {
      name: "Frank",
      surname: "Underwood",
      address: "1600 Pennsylvania Avenue, Washington, DC"
    }
  }
});

// prints "Underwood" eventually
model.
  getValue("user.surname").
  then(function(surname) {
    console.log(surname);
  });
```

Note that we use the same JavaScript path to retrieve our data as when we worked with the JSON data directly. The only difference is that the result is pushed to a callback when it becomes available.

The advantage of working with your data using Falcor’s asynchronous API is that you can move your data anywhere on the network without changing the client code that consumes the data. Using the Falcor Model, we only need to change a few lines of code to alter the previous example to work with remote data.

好处是，获取的 API 相同，不用变。但是 model 的值却可以放在其他地方。比如：

```js
var model = new falcor.Model({
  source: new falcor.HttpDataSource("/model.json")
});

// prints "Underwood" eventually
model.
  getValue("user.surname").
  then(function(surname) {
    console.log(surname);
  });
```

Now instead of working with in-memory JSON data, Falcor remotes requests for data to the application server’s Virtual JSON object. The Router retrieves the data from the data store with the information, and sends back only the requested fields.

现在是请求唯一的 endpoint，`/model.json`。

{% asset_img falcor-end-to-end.png %}

Using an asynchronous API allows you to use the same programming model regardless of whether your data is local or remote. With Falcor it is common practice to begin development immediately by mocking the server’s JSON object. Once the server’s JSON resource has been written, the Falcor Model is connected to the server JSON using an HttpDataSource. No other client code needs to change. This approach can shrink project timelines by decoupling client and server developers.

使用一套异步 API 操作数据，不用关心数据在 local 还是 remote。

通用的实践就是，立即 mock server 的 JSON object。一旦 server 的 JSON resource 写好后，Falcor Model 就可以通过 HTTPDataSource 连接上 server 的 JSON。

不需要改变任何客户端代码。

## Bind to the Cloud

In most MVC systems, controllers are responsible for retrieving data from the server. **One pattern for building applications with Falcor is to have your views retrieve data directly from the Falcor Model, just as they would from an in-memory JSON object**. This pattern is sometimes referred to as Async MVC, because the communication between the view and the model is asynchronous.

在大多的 MVC 系统里，controllers 负责从 server 获取数据。**在 Falcor 中，views 直接从 Falcor Model 获取数据。这是其中一种 pattern**。

这种模式有时叫做 Async MVC。因为 view 和 model 之间是 async 的。

{% asset_img async-mvc.png %}

In addition to decoupling controllers and views, Async MVC can improve the efficiency of your application’s network requests. When the view drives the data fetching, only the data absolutely required to render a view is retrieved from the server.

因为 Views 来声明需要什么数据，也就是必须的数据才获取。提高了效率。

In addition to Async MVC there are a variety of different patterns for integrating Falcor into your application. We are looking to the community for help building adapters for the various frameworks in use today.

除了 Async MVC，其他的 pattern 期待社区帮助构建。

# How Does Falcor Work?

原文：[How Does Falcor Work?][]

[How Does Falcor Work?]:https://netflix.github.io/falcor/starter/how-does-falcor-work.html

The Falcor Model transparently handles all network communication with the server. In order to ensure efficient client/server interactions, the Falcor Model applies a variety of different optimizations:

Faclor Model 隐式地处理着和 server 的网络请求。为了保证高效的 client/server 交互，Falcor Model 应用了一些列优化。

## Caching

Falcor maintains a fast in–memory cache that contains all of the values previously retrieved from the application server’s JSON object. If a request is made for information that is already available in the cache, the data will be retrieved from the cache and sent to the consumer’s callback as soon as possible.

Falcor 维护了一套快速的内存缓存，保存了之前所有从服务端获得的 JSON 对象。如果请求的信息在 cache 中已有，则从 cache 中获取，理解给 consumer 的 callback。

{% asset_img model-caching.png %}

To prevent the cache growing larger than the available memory on the device, developers can configure a maximum size for the cache. When the cache grows beyond the maximum size, the least-recently-used values are purged. This makes it possible to run the same application on an inexpensive mobile device or a powerful desktop machine.

为防止 cache 超过可用设备内存，开发者可以控制下 cache 的最大值。通过 LRU 算法清除。有客户端决定。

## Batching

Falcor’s ability to select as much or as little data from the network in a single request gives it the flexibility to batch multiple small requests for data into a single large request. The Falcor model can be configured to collect up multiple requests, and schedule them to be sent to the data source at once.

Falcor 可以打包小的请求到一个大的请求。涉及的逻辑是 schedule。

{% asset_img batching.png %}

In this example, we make a request for three individual values from the Model, and only a single coarse-grained request is sent to the data source

```js
var log = console.log.bind(console);
var httpDataSource = new falcor.HttpDataSource("/model.json");
var model = new falcor.Model({ source: httpDataSource });
var batchModel = model.batch();

batchModel.getValue("todos[0].name").then(log);
batchModel.getValue("todos[1].name").then(log);
batchModel.getValue("todos[2].name").then(log);

// The previous three model requests only send a single request
// to the httpDataSource: "todos[0..2].name" 
```

The ability to batch requests can significantly improve performance when using a network protocol with a high connection cost (ex. HTTP).

batch 可以极大的提升高开销网络协议（比如 HTTP）的性能。

## Request Deduping

In addition to batching outgoing requests, the Falcor Model dedupes requests. If a request is made for a value for which there is already an outstanding request, no additional request is made. This allows individual application views to retrieve their own data without having to coordinate with other views, just as if they were retrieving data from a fast in-memory JSON store.

除了 batching 对外的 requests，Falcor Model 也会对请求去重。
