title: Learn Github RESTful API v3
date: 2016-03-22 11:37:01
tags:
---

**meida type:**

最基本：
    application/json

其他类似于上传功能等单独定义。

**响应**

Response Header:

```
Status: 200 OK

{
    // ...
}
```

**分页**

/search/users?q=xx&pageIndex=1&pageSize=10

| 名称 | 类型 | 描述 |
| q | string | 关键词 |
| pageIndex | number | 第几页 |
| pageSize | number | 每页分几个 |

**错误**

```
Status: 500 INTERNAL SERVER ERROR

{
    error: '...'
}
```


