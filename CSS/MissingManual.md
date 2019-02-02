### 选择器
- 属性选择器
  - `a[href="xx"]` `a[href^="xx"]` `a[href$="xx"]` `a[href*="xx"]`
- 兄弟选择器
  - `a+p`(一个) `h2~p`(所有)
- 目标选择器：有时可用于替换JS
  - `<a href="#id1" ...> <form id="#id1" ...>`
  - 点击a则form被选中，form的样式被切换为target的样式
  - `#id1{}` `#id1:target{}`
- 否定选择器
  - `.classy{}` `p:not(.classy){}`
  - `a[href^="xx"]:not([href^="yy"])`
  - 只能用于非符合选择器，而且不能同时用2个not
- 伪元素与伪类
  - `:first-child` `nth-child(5)` `nth-child(3n)` `nth-child(3n+2)` `:nth-child(even)` `nth-child(-n+3)` 
  - `p:first-of-type` `p:last-of-type` `img:nth-of-type(odd)`

### 继承原则
- 影响布局、边框、背景、margin等属性不继承
- 浏览器固有样式不变(部分)，比如`<h1>`比`<h2>`大

### 层叠规则
- tag/伪元素:1
- class/伪类:10
- id: 100
- inline:1000

### 盒模型
- background-color填出border内部，包括padding
- 建议先全局移除padding、margin等
- padding、margin的纵向百分值的基数为容器的width而不是height
- margin重合：水平不重合，float元素之间也不重合
- inline box和block box的共有属性包括font color background border
- 除了`<img>`外，inline box没有垂直方向的padding和margin
- 背景色under边框，可用`background-clip:padding-box`来规避
- width、height的百分值依赖容器的width/height
- height只建议用于`<img>`或者banner，否则内容超出高度容易引发问题
- 除了`<img>`外，所有的float元素都最好加一个width
- background border对float的响应与content不同，本质上，文字是环绕content浮动而非background/border，使用overflow:hidden可防止这些不同造成的问题
