## 自适应布局
### 高度自适应
- 1.position:absolute
- 2.绝对定位的元素，如果没有设置高度/宽度，那么高度/宽度由top+bottom/left+right决定
### 宽度自适应
- 1.左右分别左右浮动，并给一个固定宽度
- 2.为兼容IE6 IE7，中间列设置margin-left为左列宽度，margin-right设置为右列宽度
- 3.IE6的3px间隙BUG，左列margin-right和右列margin-left设置为-3px

## 负margin
### margin普通文档流中的作用
- 文档流中，元素的最终边界是由margin决定的，margin为负的时候就相当于元素的边界向里收，文档流认的只是这个边界，不会管你实际的尺寸是多少
### 对元素自身宽度的影响
- 未设置width(或者为auto)的元素，负数的左右margin会增加宽度
### 对浮动元素的影响
- float元素的margin-left为负，会往左移动，覆盖左边的float元素
### 使绝对定位元素居中
- 1.元素绝对定位
- 2.left 50%，top 50%
- 3.设置元素的width height为具体数值，margin-left为宽度的一半的负数，margin-top为高度的一半的负数
