title: 读 Why I No Longer Use MVC Frameworks
date: 2016-02-19 10:40:25
tags:
---

原文链接：[Why I No Longer Use MVC Frameworks](http://www.infoq.com/articles/no-more-mvc-frameworks)

HN 讨论链接：https://news.ycombinator.com/item?id=11104356

**PS**：这篇文章非常不直观地透露出民科和专科的区别。

> The worst part of my job these days is designing APIs for front-end developers. 

笑尿。前端程序员首先要是个程序员。程序员的专业素养需要有。

是的，作者直接答应了前端的没有设计的 API，然后项目完成后，就会有超级多的 API。

> and where I touched the sheer power of reactive and functional programming, a journey focused on simplicity and scraping the bloat that our industry is so good at producing.

好吧，这篇文章是推销 Functional 和 Reactive 的。作者认为它们是专注于简洁性的。

> Facebook, so far, has resisted fixing that gap at the framework level.

Facebook 拒绝在 React 的框架级别引入 MVC 这些解决数据（API）的特性。

> After using React and seeing what was coming in Angular2, I felt depressed: these frameworks systematically force me to use the BFF “Screen Scraping” pattern where every server-side API matches the dataset of a screen, in and out.

作者又叹息了。每一个服务端的 API 对应屏幕上的一个 dataset。

> That’s when I had my “to hell with it” moment. I’ll just build a Web app without React, without Angular, no MVC framework whatsoever, to see if I could find a better articulation between the View and the underlying APIs.

所以，这篇文章要探讨的就是，View 和底层 APIs 的关系。

> What I really liked about React was the relationship between the model and the view. The fact that React is not template based and that the view itself has no way to request data felt like a reasonable path to explore (you can only to pass data to the view).

作者很喜欢 React 处理 Model 和 View 的方式，是传递，而不是请求。职责解耦。sole purpose。

> When you look long enough, you realize that the sole purpose of React is to decompose the view in a series of (pure) functions and the JSX syntax:
> <V params={M}/>
> is nothing different than:
> V = f( M )

非常精简的 `V=f(M)`。

>  GraphQL is nothing more than a declarative way to create a view-model. Being forced to shape the model to match the view is the problem, not the solution. 

对作者的这个观点持保留态度。继续看。


> Unlike templates or “queries written by front-end engineers”, functions do not require the model to fit the view.
> When the view is created from a function (as opposed to a template or a query) you can transform the model as needed to best represent the view without adding artificial constraints on the shape of the model.

这是作者对上述观点的继续解答。因为 View 是 function，而不是 templates 或者 query。而 function 是不需要 model 来符合 view 的。因为你可以按需改变 model 的形状来符合 view。

> For instance, if the view displays a value v and a graphical indicator as to whether this value is great, good or bad, there is no reason to have the indicator’s value in your model: the function should simply compute the value of the indicator from the value v provided by the model.
>  V = f( vm(M) )

根据作者举得这个例子，我理解了。作者认为根据一个 v，来计算 great，good，bad。这个 great，good，bad 不应该放在 model。

但是应该放在 view model 里啊。问题是，model -> view model 的这层逻辑应该放在哪里？

其实很简单，以网络传输最小字节数为基准。我 view 只应该要我想要的。我不要有逻辑。给我什么就显示什么。逻辑意味着维护成本，浏览器端维护意味着可能的泄露。

> As a veteran MDE practitioner, I can assure you that you are infinitely better off writing code than metadata, be it as a template or a complex query language like GraphQL.

那是因为 metadata 做的语义化的东西概念太多太杂。

> The natural interface they create allows you to “theme” your Web app or Website or render the view in different technologies (native for instance).

theme 这个词很吸引我。

> The function implementations have the potential to enhance the way we implement responsive design as well.

作者说，函数式实现也有利于增强响应式设计的实现方式。

> But most importantly, this approach allows the view to declare the minimum contract with the model and leaves the decision to the model as to what it the best way to bring this data to the view. Aspects like caching, lazy loading, orchestration, consistency are entirely under the control of the model. Unlike templates or GraphQL there is never a need to serve a direct request crafted from the view perspective.

最重要的就是，view 可以对 model 声明尽可能少的 contract。model 来负责一些切面需求，比如缓存，延迟加载，组合，一致性。

注：model，不希望有 model 这个粒度，而是类似于 GraphQL 的，property 粒度的。这块的确会复杂，但是如果把数据结构图理清的话，会把这块完全抽象出去。

> The core issue here is, as Andre Medeiros so eloquently puts it, that the MVC pattern is “interactive” (as opposed to Reactive). In traditional MVC, the action (controller) would call an update method on the model and upon success (or error) decide how to update the view. As he points out, it does not have to be that way, there is another equally valid, Reactive, path if you consider that actions should merely pass values to the model, regardless of the outcome, rather than deciding how the model should be updated.

MVC 模式是交互式的。在传统的 MVC 模式中，action (controller) 会调用 model 上的 update 方法。然后根据是 success 或者 error 来更新 view。Andre Medeiros 也指出，可以使用另一种同样有效的，响应式的方案，只要你把 action 只是传递值给 model，不用关心结果。

> The key question then becomes: how do you integrate actions in the reactive flow?

这的确是很多人争论的，flux 中的 action 需不需要？

> If you want to understand a thing or two about Actions, you may want to take a look at TLA+. TLA stands for “Temporal Logic of Actions”, a formalism invented by Dr. Lamport, who got a Turing award for it.
>  In TLA+, actions are pure functions:
>             data’ = A (data)

> I really like the TLA+ prime notation because it reinforces the fact that functions are mere transformations on a given data set.

作者首先引入了 TLA 的概念。Actions 是纯函数。并且强化了事实，函数只是所给数据集的转化。

> With that in mind, a reactive MVC would probably look like:
>             V = f( M.present( A(data) ) ) 

> Actions, again, are pure functions, with no state and no side effect (with respect to the model, not counting logging for instance).

Actions 是纯函数，没有状态和副作用。比如没有计数日志这些。

> This paragraph is very important, so please read carefully. We have seen that, in TLA+, the actions have no side effects and the resulting state is computed, once the model processed the action outputs and updated itself. That is a fundamental departure from the traditional state-machine semantics where the action specifies the resulting state, i.e. the resulting state is independent of the model. In TLA+, the actions that are enabled and therefore available to be triggered in the state representation (i.e. the view) are not linked directly to the action that triggered the state change. In other words, state machines should not be specified as tuples that connect two states (S1, A, S2) as they traditionally are, they are rather tuples of the form (Sk, Ak1, Ak2,…) that specify all the actions enabled, given a state Sk, with the resulting state being computed after an action has been applied to the system, and the model has processed the updates.

这一段很重要，我也全段摘出解读。

在 TLA+ 中，actions 是没有副作用的，结果状态是计算出来的，有 model 来处理 actions outputs，自己更新自己。这和传统的状态机语法有本质的区别，在状态机中，结果状态是有 action 指定的，和 model 是独立的。

在 TLA+ 中，可以在 state representation（比如 view）被触发的 actions  不是用来触发 state change 的 action。

所以，状态机不能再表示为连接两种状态的管道（tuple），即 `(S1, A, S2)`，而是，给定一个状态，`Sk`，然后再给定所有的启用的 actions，即 `(Sk, Ak1, Ak2, ...)`。结果状态是在一个 action 被用于应用后计算出来的，由 model 来处理。



