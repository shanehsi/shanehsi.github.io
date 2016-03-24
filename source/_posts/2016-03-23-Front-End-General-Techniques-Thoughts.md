title: 前端技术选型思考于2016年03月23日
date: 2016-03-23 09:40:04
tags:
---

* HTML/CSS
* React
* Redux
* React Router
* Rx
* Falcor/Relay/GraphQL
* Express

分层的话：

| 分层 |
| -- |
| HTML/CSS - UI, UE |
| View - React |
| Data flow - Redux/Rx |
| Ajax/Cache - Falcor |
| Backend - Express/Falcor Endpoint |

其他的：

* Test
    * Ava
    * Enzyme
* React Router
* Webpack module bundler
* Proxy
* Mock Server

**优先级**：

1. Ava/Enzyme - 黑盒
2. Redux/Rx
3. HTML/CSS
4. Faclor
5. Express
6. Mock Server
7. Proxy

