title: Neos CMS Goes for a Full UI Rewrite with React and Redux
date: 2016-05-27 16:42:39
tags:
---

原文：http://dimaip.github.io/2016/03/13/neos-react-redux-rewrite/

{% asset_img Modal.png %}

The AddNodeModal dialog that we are going to implement

通过这个功能，我们来看下它们是怎么组织代码。

Our task would be to create a dialog for creating nodes

## State

The main reason for the switch to this new stack was the desire to give more predictability and integrity to the UI. You see, our case is slightly complicated by the fact that we have the same data distributed across multiple places: the navigation tree, inline editing etc. Before we did not have a unified data model, and all of this modules functioned independently, carefully glued together by some state syncing code. Yes, that was kind of a nightmare. That is why here from the start we for having all data clearly normalised and stored in the state. But that includes not only the content data, but also the state of the UI itself: all trees, panels, user preferences and so on now have a dedicated place in the application state.

