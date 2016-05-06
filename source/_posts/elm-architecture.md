title: elm architecture
date: 2016-04-28 15:14:46
tags:
---

- model
- update
- view 

```js
-- MODEL

type alias Model = { ... }


-- UPDATE

type Action = Reset | ...

update : Action -> Model -> Model
update action model =
  case action of
    Reset -> ...
    ...


-- VIEW

view : Model -> Html
view =
  ...
```


## Example 1: A Counter

```js
type alias Model = Int

// 看到了吗？使用 union type， TypeScript 也有。然后通过 namespace
type Action = Increment | Decrement

update : Action -> Model -> Model
update action model =
  case action of
    Increment -> model + 1
    Decrement -> model - 1

```

view

```js
view : Signal.Address Action -> Model -> Html
view address model =
  div []
    [ button [ onClick address Decrement ] [ text "-" ]
    , div [ countStyle ] [ text (toString model) ]
    , button [ onClick address Increment ] [ text "+" ]
    ]

countStyle : Attribute
countStyle =
  ...
```


注意理解下 `Address` 函数，后面在讲解到。

## Example 2: A Pair of Counters

我们看这个 Fractal 的架构如何组合？如何重用？

我们封装了一个模块，它的接口是统一的：

`Model init Action update view`


```js
module Counter (Model, init, Action, update, view) where

type Model

init : Int -> Model

type Action

update : Action -> Model -> Model

view : Signal.Address Action -> Model -> Html
```

我们期待的是提供合适的函数接口，但是隐藏具体执行过程。













































































































