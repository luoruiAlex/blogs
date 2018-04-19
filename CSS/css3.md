## border-radius: 左上 => 右上 => 右下 => 左下

## 阴影
### box-shadow
- h-shadow
- v-shadow
- blur
- spread
- color
- inset
### text-shadow
- h-shadow
- v-shadow
- blur
- color

## 2D转换
- translate(Xpx, Ypx) 移动
- scale(X, Y) 缩放
- rotate(Xdeg) 旋转
- skew(Xdeg, Ydeg) 倾斜

## 过渡
- transition: property duartion timingfunction, property duration timingfunction...
- property: all | none | property， 默认all
- duration: 默认0
- timing function: ease | ease-in | ease-out | ease-in-out| 自定义 cubic-bezier(n, n, n, n)

## 动画
### 1.定义
- ```
  @keyframes name {
    from {
    }
    to {
    }
  }
```
- from = 0% to = 100% 中间可加若干个 number%
### 2.绑定
- name duration timingfunction delay count(播放次数)

## 渐变
### line-gradient(color1, color2, ...)
- 默认上到下
- (to right | bottom right) 左 => 右 | 左上 => 右下
### radial-gradient(circle | ellipse, color1, color2, ...)

## 多媒体查询
- @media not | only | all  print | screen | speech
- and (min-width: xxpx) and (max-width: yypx) { xxclass{}}
