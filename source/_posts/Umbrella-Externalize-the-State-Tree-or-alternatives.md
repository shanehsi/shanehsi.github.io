title: 'Umbrella: Externalize the State Tree (or alternatives) '
date: 2016-05-06 23:37:40
tags:
---

> React provides the notion of implicitly allowing a child component to store state (using the setState functionality). However, it is not just used for business logic state. It is also used to remember DOM state, or tiny ephemeral state such as scroll position, text selection etc. It is also used for temporary state such as memoization.

子组件可以 store 状态

作用：
- 业务逻辑状态
- DOM 状态
- 小的瞬间状态（比如滚动位置，文字选择）
- 临时存储，比如记忆

> This is kind of a magic black box in React and the implementation details are largely hidden. People tend to reinvent the wheel because of it, and invent their own state management systems. E.g. using Flux.

黑盒子，细节被隐藏，人们倾向于重新造轮子（Flux）

> There is still plenty of use cases for Flux, but not all state belongs in Flux stores.

当然不应该所有 state 交给 flux

> Manually managing the adding/removing of state nodes for all of this becomes a huge burden. So, regardless you're not going to keep doing this manually, you'll end up with your own system that does something similar. We need a convenient and standard way to handle this across components. This is not something that should be 100% in user space because then components won't be able to integrate well with each other. Even if you think you're not using it, because you're not calling setState, you still are relying on the capability being there.

> It undermines the ecosystem and eventually everyone will reconverge on a single external state library anyway. We should just make sure that gets baked into React.

> We designed the state tree so that the state tree data structure would be opaque so that we can optimize the internals in clever ways. It blocks many anti-patterns where external users breaks through the encapsulation boundaries to touch someone else's state. That's exactly the problem React's programming model tries to address.

state 要不透明,对内部进行优化,不能碰到其他的 state.

> However, unfortunately this state tree is opaque to end users. This means that there are a bunch of legitimate use cases are not available to external libraries. E.g. undo/redo, reclaiming memory, restoring state between sessions, debugging tools, hot reloading, moving state from server to the client and more.

因为是不透明的, 就有不能做 undo/redo reclaiming memory,  restoring state between sessions, debugging tools, hot reloading, moving state from server to the client and more.

> We could provide a standard externalized state-tree. E.g. using an immutable-js data structure. However, that might make clever optimizations and future features more difficult to adopt. It also isn't capable of fully encapsulating the true state of the tree which may include DOM state, it may be ok to treat this state differently as a heuristic but the API need to account for it. It also doesn't allow us to enforce a certain level of encapsulation between components.

> Another approach is to try to add support for more use cases to React, one-by-one until the external state tree doesn't become useful anymore. I've created separate issues for the ones we we're already planning on supporting:


