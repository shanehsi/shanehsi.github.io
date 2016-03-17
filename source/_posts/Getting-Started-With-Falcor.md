title: Getting Started With Falcor
date: 2016-03-16 14:55:41
tags:
---

[Getting Started With Falcor](https://auth0.com/blog/2015/08/28/getting-started-with-falcor/)

## Outline

In this tutorial, we will look at some of Falcor's basic operations and show how they can be used to construct a virtual JSON model from a data source. The example data we'll use will model a handful of JavaScript conferences. We'll start on the client side and eventually move everything over to be served from a NodeJS back end. 

会涉及到 Falcor 的一些基本操作。

如何从 data source 构造 virutal JSON model。

mock 数据是 JavaScript 会议。

先从 client 端，在写到 NodeJS backend。

Specifically, we will cover:

- √ What Falcor is and what problems it solves
- How to set up a Falcor model on the client and prime it with data 纯 client 端
- How to use Falcor's JSON Graph to avoid having duplicate data JSON Graph 上场，dedupe
- How to set up up a Falcor Router with Falcor Express Falcor Router 上场
- How to do a basic search using a Falcor route 一个基本的搜索功能

## What is Falcor?

One of the key problems that Falcor solves is the fact that HTTP wasn't designed and really isn't optimized for how web applications should ideally request data. With HTTP, only a single resource can be retrieved per request. Since web applications tend to need to request many small resources, many HTTP requests are often required to get all the data the application needs.

HTTP 并不是为 web application 设计和优化的。

web application 需要很多次小的资源请求。开销又很大。

Another problem that Falcor solves is that JSON objects represent data hierarchically, as trees, whereas data is much more often a graph. In other words, an application's data doesn't always have a strict parent-child relationship, but rather will tend to have many predecessors, or be related to many other pieces of data. For example, suppose that we have several categories for our JavaScript conferences. A conference like ng-conf could fit under the category of "JavaScript", but could also fit under "AngularJS". If we were to model this with JSON, we might get something like this:

另一个解决的问题，是 data representation，数据展示。

JSON 对象是 tree。

而 data 一般是 graph。

也就是一般不是父子单线关系，而是多线联系。

```js
categories: {
  javascript: [
    0: {
      name: ng-conf
    }
  ],
  angularjs: [
    0: {
      name: ng-conf
    }
  ]
}
```

As it is expressed here, ng-conf has a parent-child relationship with both categories, when really it is related to both. Falcor makes it possible to deal with data as a graph, but still output it as a normal JSON object.

Falcor 做到是，可以用 graph 来表示 data，但是输出的还是普通的 JSON 对象。

Falcor also helps to make data retrieval more efficient. Instead of pulling many JSON objects from an API and then only using some parts of them in the view layer, Falcor wires together a single virtual JSON model that provides only the data that is required at the time that it is needed. For example, if we wanted to get the name of a conference and the first names of its attendees, we typically would first have our API serve the user data and then we would use the first name value in our view. However, with Falcor, we can easily say that we want only the first name value for each attendee and then have that be the only piece of user data that is returned. In this way, Falcor essentially customizes data to the application's view. Falcor does this through its JSON Graph convention, which is used to model graph information as a JSON object.

数据获取高效：按需索取。

Furthermore, Falcor caches the virtual JSON model so that it can be accessed in-memory. When it becomes necessary to request more or different data, Falcor first consults the cache and if the data is not available there, it makes another request for only the fragment that is missing.

缓存。

后面的就直接练习吧。

## Demo

```js
var falcor = require('falcor');
var falcorExpress = require('falcor-express');
var Router = require('falcor-router');

var express = require('express');
var _ = require('lodash');
var app = express();

// Have Express request index.html
app.use(express.static('.'));

var $ref = falcor.Model.ref;

// Same data that was used in the view for our
// events, but this time on a simple object
// and not a Falcor model.
var eventsData = {
  locationsById: {
    1: {
      city: "Salt Lake City",
      state: "Utah"
    },
    2: {
      city: "Las Vegas",
      state: "Nevada"
    },
    3: {
      city: "Minneapolis",
      state: "Minnesota"
    },
    4: {
      city: "Walker Creek Ranch",
      state: "California"
    }
  },
  events: [
    {
      name: "ng-conf",
      description: "The world's best Angular Conference",
      location: $ref('locationsById[1]')
    },
    {
      name: "React Rally",
      description: "Conference focusing on Facebook's React",
      location: $ref('locationsById[1]')
    },
    {
      name: "ng-Vegas",
      description: "Two days jam-packed with Angular goodness with a focus on Angular 2",
      location: $ref('locationsById[2]')
    },
    {
      name: "Midwest JS",
      description: "Midwest JS is a premier technology conference focused on the JavaScript ecosystem.",
      location: $ref('locationsById[3]')
    },
    {
      name: "NodeConf",
      description: "NodeConf is the longest running community driven conference for the Node community.",
      location: $ref('locationsById[4]')
    }
  ]
}

app.use("/model.json", falcorExpress.dataSourceRoute(function(req, res) {
    return new Router([{
      // route 需要符合 整形模式，作为 eventIds
    route: "events[{integers:eventIds}]['name', 'description']",
        get: function(pathSet) {
            var results = [];
            // 在上面我们指定了 eventIds 标识符
            // 是 array，可以 loop
            pathSet.eventIds.forEach(function(eventId) {
              // pathSet[2] 维护的是 key names array
              // JavaScript 嵌套一层（共两层），就会有两个 forEach
                pathSet[2].forEach(function(key) {
                    var eventRecord = eventsData.events[eventId];
                    // 就是构造一个 {path, value}，让 client 能读懂
                    results.push({
                        path: ['events', eventId, key],
                        value: eventRecord[key]
                    });
                });
            });
            return results;
        }
    }]);
}));

app.listen(3000);

console.log("Listening on http://localhost:3000");
```

Routes 是匹配 KeySet。在route string 的第一个数组。

`route: "events[{integers:eventIds}]['name', 'description']"`

这里，我们指定：

- 想匹配 integers
- 要被 eventId 检索

`get` 中的 `pathSet` 参数可以让我们访问 KeySet（KeySet 是客户端提供的，也就是 get 里的）。

```js
var model = new falcor.Model({ source: new falcor.HttpDataSource('/model.json') });

model
    .get([
        "events",
        { from: 0, to: 2 },
        ["name", "description"]
    ])
    .then(function(response) {
        document.getElementById('event-data').innerHTML = JSON.stringify(response, null, 2);
    });
```

总结下，这个 route 就是要访问 `name` 和 `description`。

如果要获取 `location`，要写另一个 route，**同时记得在第一个 route 里加上 'location'**。

下面的 route string，加上了 `'location'`。

```js
route: "events[{integers:eventIds}]['name', 'description', 'location']",
```


```js
{
    route: "locationsById[{integers:locationId}]['city', 'state']",
    get: function(pathSet) {
        var results = [];
        pathSet.locationId.forEach(function(locationId) {
            pathSet[2].forEach(function(key) {
                var location = eventsData.locationsById[locationId];
                results.push({
                    path: ['locationsById', locationId, key],
                    value: location[key]
                })
            });
        });
        return results;
    }
},

```

最后一个 route，**Search for an Event by Name**：

The client can send integers as a KeySet to pull a range of results, but it can also provide strings. We can use this to setup an "event by name" search route.

> 吐槽下，route string 非常的诡异啊。得写一个 parser 啊。

```js
[
    "events",
    { from: 0, to: 2 },
    ["name", "description"]
]
```

可以用 `{ from: 0, to: 2 }` 作为 KeySet 来表示 range。

当然，也可以提供 string 啦。可以用这个特性，做一个 search route（“event by name“）。

```js
{
    route: "events.byName[{keys}]['description']",
    get: function(pathSet) {
        var results = [];
        pathSet[2].forEach(function(name) {
            eventsData.events.forEach(function(event) {
                if (_.includes(event, name)) {
                    results.push({
                        path: ['events', 'byName', name, 'description'],
                        value: event.description
                    });
                }
            });
        });
        return results;
    }
}
```

client 调用：

```js
model.get(["events", "byName", ["Midwest JS"], ["description"]])
```

