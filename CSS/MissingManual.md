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
  
