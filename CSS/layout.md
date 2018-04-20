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
- inline-block table-cell table-caption
#### 特点
- 子元素的css不影响BFC元素外部
- 浮动子元素也会参与BFC父元素的高度计算
  - 利用此原理： clearfix解决父元素高度塌陷
- 占据文档流的BFC元素能识别浮动的兄弟元素
  - BFC元素与浮动兄弟元素同行而不被覆盖
- 站数据文档流的BFC元素，`width:auto`时，占据当前行的剩余宽度
- 同一个BFC内上下相邻box的margin会重叠
