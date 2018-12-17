### 简介
- 一个Python的2D图形库

### pyplot
- pyplot包含一系列MATLAB中绘图函数的相关函数，每个函数对当前图像进行一些修改
- `import matplotlib.pyplot as plt`
- plt.show()
  - plt默认不显示图像，show()调用后显示
  - ipython中 `%matplotlib inline`则图像输出在notebook中
- plt.plot()
  - plt.plot(x, y)
  - plt.plot(x, y, format_str)
  - plt.plot(y)，x为0~N-1
  - plt.plot(y, format_str)，x为0~N-1
  - format_str
    - color: b(blue) g(green) r(red) m(magenta) y(yellow) k(black) w(white)
    - type: '-'实线	'--'虚线 '-.'虚点线 ':'点线 '.'点 ','像素点 'o'圆点 'v'下三角点 '^'上三角点 '<'左三角点 '>'右三角点 '1234'下上左右三叉点 's'正方点 'p'五角点 '*'星形点 'h'六边形点1 'H'六边形点2 '+'加号点 'x'乘号点 'D'实心菱形点 'd'瘦菱形点 '_'横线点
- plt.ylabel(str) plt.xlabel(str) 指定轴的说明文字
- `plt.axis([xmin, xmax, ymin, ymax])`指定轴的显示范围
- 使用Numpy数组作为参数
  ```
  t = np.arange(0., 5., 0.2)
  plt.plot(t, t,    'r--',
           t, t**2，'bs',
           t, t**3, 'g^')
  ```
- 使用关键字参数来修改线条
  ```
  x = np.linspace(-np.pi, np.pi)
  y = np.sin(x)
  plt.plot(x, y, linewidth=2.0, color='r')
  ```
