## 含义
- 文档流中，元素的最终边界是由margin决定的，margin为负的时候就相当于元素的边界向里收，文档流认的只是这个边界，不会管你实际的尺寸是多少
## 基本特点
  - margin始终是透明的
  - 值可以为auto length percentage
  - 垂直margin对**非置换内联元素**不起作用
  - 子元素的margin-top如果一直没有碰到父元素/祖先元素的border或者padding，父元素/祖先元素的margin-top会把这个margin-top与自己的margin-top重叠
## 参考线
  - left/top以containing block的content左/上或者以相连元素margin的右/下为参考线往右/下进行位移
  - right/bottom以自身的border的右/下为参考线往右/下进行位移
  - 总结：**margin的top left为负数时元素自身移动，bottom right为负值时后面跟着的元素移动**
## 负margin
### 对元素自身宽度的影响
- 未设置width(或者为auto)的元素，负数的左右margin会增加宽度
### 对浮动元素的影响
- float元素的margin-left为负，会往左移动，覆盖左边的float元素
### 应用
- 使绝对定位元素居中
  - 1.元素绝对定位
  - 2.left 50%，top 50%
  - 3.设置元素的width height为具体数值，margin-left为宽度的一半的负数，margin-top为高度的一半的负数
- Tab选项卡
  - 选项卡margin-bottom为-1px使内容往上位移1px，遮住黑色边框
  - 每个item的margin-left为-1px使右侧的item向左位移，保证左右都只有一条分割线
- 往大图片上增加小图片(比如商品图片上增加限时抢的小图片)
  - 传统做法为使用relative/absolute
  - 小图片的margin-top为合适的负数，margin-left为合适的正数即可达到效果
- 鳞片式导航
  - 除第一个item外的margin-left为负值
- 自适应左右布局
  - 左侧图片，右侧为文本，文本div的margin-top为图片的高度的负值
