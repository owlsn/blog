title: sketch布尔运算和路径剪切
date: 2020-01-01 15:45:10
tags: [sketch]
categories: 设计
about:
---
sketch中提供了布尔运算的功能，就是将几个简单图形组合成复杂图形，路径操作就是对图形上的锚点进行一些操作，一开始学习使用过程中使用这两个功能有遇到几个小问题。这里做一下记录总结。
<!-- more -->
### 布尔运算
+ 合并形状 (Union)：合并的结果是会得到两个矢量区域的总和。   
+ 减去顶层形状 (Subtract)：这一项的结果是顶层矢量的区域会从下一层的图形上移去。  
+ 与形状区域相交 (Intersect)：与形状区域相交的结果是会保留原图形重叠的部分。  
+ 排除重叠形状 (Difference)：排除重叠形状的结果是只保留原图形不重叠的部分，它是“与形状区域相交”这一运算的反向。 

问题来了，这里所说的几个布尔操作，并没有说详细说清楚填充和边界的问题，这里我经过几个测试后得到一些结论，就是union会将两个图形的边界都合并，而在公共部分内部的边界就会被覆盖了，边界继承的是底层图形的边界；substract是顶层的图形从下一层图形移走之后，下一层图形与顶层图形的边界地方，就会形成下一层图形的边界，边界的宽度和下一层图形的边界设定相同；intersect和difference同理，都是以下一层图形的边界和区域为准，如下图：   
<img src='https://raw.githubusercontent.com/owlsn/blog/master/source/source/origin.jpg' width='200px' heigth='200px' alt='origin'>
<img src='https://raw.githubusercontent.com/owlsn/blog/master/source/source/union.jpg' width='200px' heigth='200px' alt='union'>
<img src='https://raw.githubusercontent.com/owlsn/blog/master/source/source/subtract.jpg' width='200px' heigth='200px' alt='subtract'>
<img src='https://raw.githubusercontent.com/owlsn/blog/master/source/source/intersect.jpg' width='200px' heigth='200px' alt='intersect'>
<img src='https://raw.githubusercontent.com/owlsn/blog/master/source/source/difference.jpg' width='200px' heigth='200px' alt='difference'>

### paths
路径和锚点是联系在一起的，基础图形上都有一些锚点，比如正方形是4个顶点，圆形是上下左右四个点，这些点沿着图形的边就是路径，锚点可以通过编辑来增加新的点，因此路径也可以增加，当路径被剪刀工具去掉的时候，图形就不在是封闭的了，这个时候剪的是路径，沿线的border也会被剪掉。   
### 问题出现
这里我想画个半圆弧，作为声音图标的右边部分，先画了一个圆，只取border，去掉了fill部分，然后右边用正方形覆盖，布尔运算取subtract，这个时候半圆就会生成竖线的边，按照道理，然后用剪刀工具把这条边剪掉，就是半圆弧了，但是这个时候，剪刀工具居然发现这条竖线没有路径，不能剪掉，按照道理来说，路径都没有为什么会有border呢，   
<img src='https://raw.githubusercontent.com/owlsn/blog/master/source/source/no_path.jpg' width='200px' heigth='200px' alt='no_path'>

然后我再仔细看了，发现可能是图层上下的关系。  

+ 正方形在上，圆在下，取subtract，这个时候没有路径   
+ 正方形在下，圆在上，取intersect，这个时候有路径，虽然半圆是反过来的

而且应该和布尔运算也有关系，因为布尔运算主要是以下面的图形为主，按照规则去除上面图形所占的部分，所以会把对应的路径也去除，而且因为正方形的一条边的两个锚点和圆的上下的两个锚点重合了，所以以圆为准的话上下两个点之间是不存在路径的，所以也无法用剪刀工具去除，要解决这个问题的话，有很多办法，比如上面的将正方形和圆取intersect，得到的左边半圆的部分因为继承了正方形的路径，所以可以用剪刀工具去除，然后将半圆反过来
### 参考资料
[1]. [boolean-operations](https://www.sketch.com/docs/shapes/#boolean-operations)   
[2]. [paths](https://www.sketch.com/docs/vector-editing/#opening-and-closing-paths)