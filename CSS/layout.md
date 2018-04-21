## box-sizing
- IE盒模型: border-box， W3C盒模型: content-box
- IE6 IE7不可用box-sizing，只能用W3C盒模型
- IE8 不能和min-width max-width min-height max-height一起用
- IE9以后建议全局用content-box: 默认优于配置，border-box在calc出现后优势不明显

## 块级元素和行内级元素
### 块级元素block-level
- width: 100%
- height: 0
- 可设置width padding margin border
### 行内级元素inline-level
#### 1.不可置换元素
- 不能设置width、height和上下的margin
#### 2.可置换行内元素
- 展示内容不在CSS作用域内，通过value src css content等属性来控制展示内容
- `<img><input><textarea><video><embed>`
- `<audio><canvas><object>`在特殊情况下为可置换元素
- 通过css的content属性来插入的对象为匿名可置换元素
- 行内可置换元素的布局特性与inline-block-level元素相同
- 可置换元素、inline-block-level元素默认在baseline上显示，而不是行框的bottom线上
#### 特性
- 水平方向的对齐方式受text-align控制，默认左对齐
- 垂直方向的对齐方式受vertical-align控制，默认baseline
### 行内块级元素 inline-block-level
- 外层inline，内层block
- 水平、垂直方向上的对齐方式与inline-level元素相同
- width默认为0，可设置width、padding、margin、border

## FC
- BFC IFC FFC GFC
### BFC
#### 生成BFC
- 根元素或包含根元素的元素
- overflow不为visible
- float
- 绝对定位
- inline-block table-cell table-caption grid inline-grid flex inline-flex
#### 特点
- 子元素的css不影响BFC元素外部  
  - 普通块级元素，子元素的margin-top不会隔开自身与父元素，但会作用到父元素外部
  - BFC元素，子元素的margin-top会隔开自身与父元素
![image](https://github.com/luoruiAlex/blogs/blob/master/CSS/images/layout_bfc1.png)
- 浮动子元素也会参与BFC父元素的高度计算
  - 普通块级元素，不能识别浮动子元素，会出现"高度塌陷"的问题
  - BFC元素能识别浮动子元素
  - 利用此原理： clearfix能解决父元素高度塌陷
- 占据文档流的BFC元素能识别浮动的兄弟元素
  - 普通块级元素，会被浮动的兄弟元素覆盖部分内容
  - BFC元素与浮动兄弟元素同行而不被覆盖，与浮动兄弟元素同行显示  
  ![image](https://github.com/luoruiAlex/blogs/blob/master/CSS/images/layout_bfc2.png)
- 占据数据文档流的BFC元素，`width:auto`时，占据当前行的剩余宽度，`width:100%`时，宽度为父元素宽度，即换行。
- 同一个BFC内上下相邻box的margin会重叠

## 包含块(Containing Box)
- 设置尺寸(width、height、padding、margin、border)的百分比值或者偏移属性(top、right、bottom、left)的值时，有一个“相对参考系”。具有相对参考系作用的祖先元素的容纳区域(content box、padding box)叫做包含块。
- 特例: relative的偏移相对于元素在文档流原来的位置
- ICB 初始包含块，根元素的包含块，一个不可见的矩形框，不是它的父元素
- static、relative的包含块：块级祖先元素的content box
- absolute的包含块：最近非static得祖先元素的padding box或者ICB
- fixed的包含块：当前viewport

## 文档流
- 左到右、上到下，彼此识别，有序排列
- 脱离文档流 不被父元素识别且不参与父元素高度的计算
  - float
  - absolute、fixed
### float
- 起始位置：最后一行最左侧的空白位置(包含左右)
- 浮动方向： 左浮动向左，右浮动向右
- 结束位置： 左浮动————遇到第一个左浮动元素或包含块的最左侧padding；右浮动————遇到第一个右浮动元素或包含块的最右侧padding
### 清除浮动
- 改变"当前元素"与"前一个声明的浮动元素"之间的默认布局规则，主要为让当前元素换行显示
  - 使用者：当前元素
  - 作用对象： 前一个声明的浮动元素
  - 目的： 让当前元素换行显示
- clear: left只清除左浮动，right只清除右浮动，both清除左浮动和右浮动
- 清除浮动后
  - 当前元素为float，垂直方向上的mragin不会与前一个声明的浮动元素合并
  - 当前元素不是float，会合并
- 特殊作用： 解决父元素高度塌陷

## line box（行框）
- 结构: top线、text-top(h字母上方)、middle、baseline(a下方)、text-bottom(p下方)、bottom
- content-area: text-top到text-bottom之间
- 行半距 half-leading：top到text-top以及text-bottom到bottom
### 行框高度: top到bottom
- content-area + 2 x half-leading
- 受自身line-height和内部inline-level子元素的margin box高度值、line-height、vertical-align的影响
  - 1.可由自身的line-height设置
  - 2.受不可置换(`span a label`等)子元素的content-area影响
  - 3.可由不可置换子元素的line-height属性设置
  - 4.可由可置换子元素或inline-block子元素的margin box高度和vertical-align属性设置：当vertical-align为top/bottom时，行框高度最小，刚好为子元素的margin box的高度
  - 5.取以上最大值
### line-height
- 块级元素每一行的行高都可以不同
- 不可置换子元素的line-height可决定所在行框的高度，进而影响父元素的高度
- 如果一个父元素不设置height，则height为所有行高度之和
### vertical-align
- 设置inline元素自身在行框内的垂直对齐方式
- table-cell元素用于控制其内部子元素在垂直方向上的对齐方式

## marin
- 水平auto表示取所在行剩余宽度
- 垂直auto表示取0
### 垂直margin合并
- float元素之间垂直margin不合并
- 子元素在垂直方向与父元素不合并垂直margin的情况：
  - 1.父元素为BFC
  - 2.父元素有border/padding
  - 3.子元素为可置换或inlie-block/inline-table/inline-caption
