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
### 对文档流的影响
- **元素无float，position:static**：bottom right负值时后面元素会移动并且覆盖前面的
- **元素无float，position:relative**：bottom right负值时后面元素会移动并且被该元素覆盖
- **元素无float，position:absolute**：bottom right负值时后面元素不会移动
- **元素float，position:static/relative**：如果margin方向与float方向一致，则元素往float方向移动；如果方向相反，margin方向的相邻元素往float方向移动并覆盖
- float元素的margin-left为负，会往左移动，覆盖左边的float元素
### 对元素自身的影响
- 未设置width(或者为auto)的元素，负数的左右margin会增加宽度
- margin-top为负值不会增加高度，只会产生向上唯一
- margin-bottom为负值不会产生唯一，会减少自身的供css读取的高度(obj.height仍为原高度)
### 应用
- 使绝对定位元素居中
  - **绝对定位元素，负margin会基于其绝对定位坐标再偏移**
  - 1.元素绝对定位
  - 2.left 50%，top 50%
  - 3.设置元素的width height为具体数值，margin-left为宽度的一半的负数，margin-top为高度的一半的负数；或者translate3d(-50%, -50%, 0)
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
- 圣杯布局与双飞翼布局
  - 1.content在前面,width:100%;left right均为float:left，指定width
  - 2.left right都跑到content同一行，且分别最左最右：left的margin-left:-100%;right的margin-left:-right-width;//float元素的margin-left为负，会往左移动，覆盖左边的float元素
  - 3.1双飞翼布局：在content中增加一个div存放内容，设置该div的左右margin分别为left right的width
  - 3.2圣杯布局：父元素增加padding-left：left-width，padding-right:right-width;left right分别通过relative left/right的负数值让它们回到它们各自的位置上。
  
