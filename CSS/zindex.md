## 定义
- z-index: auto | number(可为负)
- object.style.zIndex="value"
- postion:static时无效，即只适用于定位元素

## stacking context
- 触发
  - 定位元素且定义了非auto的z-index
  - IE6 IE7下定位元素定义了z-index
  - FF Safari Chrome设置了非1的opacity
- 堆叠级别
  - 同一个堆叠上下文中：数值大的在上，数值相同的后来居上
  - 不同的堆叠上下文中，子元素显示顺序以父级的堆叠上下文的堆叠级别来决定显示的先后情况。于自身堆叠级别无关
