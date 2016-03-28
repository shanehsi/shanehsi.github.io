title: Why is this inline-block element pushed downward?
date: 2016-03-27 19:21:34
tags:
---

[Why is this inline-block element pushed downward?](http://stackoverflow.com/questions/9273016/why-is-this-inline-block-element-pushed-downward)

```html
<div id="container">
    <div id="firstDiv">FIRST</div>
    <div id="secondDiv">SECOND</div>
    <div id="thirdDiv">THIRD
        <br>some more content
        <br> some more content
    </div>
</div>
```

```css
body{
    width: 350px;
    margin: 0px auto;
}
#container {
    border: 15px solid orange;   
}
#firstDiv{
    border: 10px solid brown;     
    display:inline-block;
    width: 70px;      
    overflow:hidden;  
}
#secondDiv{
    border: 10px solid skyblue;         
    float:left;
    width: 70px;     
}
#thirdDiv{
    display:inline-block;
    border: 5px solid yellowgreen;    
}
```

回到：

Basically you have added more clutter in your code which is creating more confusion so first I try to remove clutter which hinders understanding the real issue.

First of all we have to establish that what's the real question? Its that why "inline-block" element is pushed downward.

首先看下，

Now we start to understand it and remove the clutter first.

Here comes the turn of some [literature](http://www.w3.org/TR/CSS2/visudet.html#leading) to grasp the idea of line boxes and how they are lined in the same line esp read last paragraph carefully because there lies the answer of your question.

> The baseline of an 'inline-block' is the baseline of its last line box in the normal flow, unless it has either no in-flow line boxes or if its 'overflow' property has a computed value other than 'visible', in which case the baseline is the bottom margin edge.

If you are not sure about [baseline](http://www.w3.org/TR/CSS2/visudet.html#leading) then here is brief explanation in simple words.

inline-block 的 baseline 是最后一个 line box 的 normal flow。