# 一、Cesium 中有哪些坐标系
## 1、地理坐标系

Cesium 中的地理坐标系默认使用 WGS84 椭球参数，地理坐标表达方式有度和弧度两种。相关的类是 `Cartographic` 。要使用其他的地理坐标系，需要自己定义椭球以及相关的加载数据的类，例如 `Ellipsoid` 、`ImageryProvider`。

> Cartographic 坐标也叫制图坐标，它和经纬度的概念一样，只是单位不一样。
## 2、世界坐标系
世界坐标系是一种地心坐标系（ECEF，地心固定坐标系）依然默认采用 WGS84 椭球，是一个采用右手系的空间直角坐标系。坐标系的原点为椭球中心，X 轴指向 0 度经线与赤道的交点，Z 轴指向北极极点，Y 轴指向东经 90 度与赤道的交点。相关的类是 `Cartesian3`。
![[image-20241205153728450.png]]
图片来源：https://zh.wikipedia.org/wiki/地心地固坐标系
## 3、屏幕坐标系
屏幕坐标系即屏幕显示的二维坐标系，鼠标事件中获取到的坐标系即为屏幕坐标系。坐标系的原点为地图显示区域左上角，X 轴指向右边，Y 轴指向下方。相关的类是 `Cartesian2`。
```js
o-------------- x
|
|
|
| 
y
```
## 4 、局部坐标系
也叫模型坐标系。坐标原点为局部指定的点，X 轴指向正东方向，Y 轴指向正北方向，Z 轴指向原点的椭球面法线方向。
![[image-20241205164744019.png]]
# 5 、相机坐标系（EyeCoordinate）
原点为相机的位置，Z 轴指向相机（即相机的朝向为 Z 轴负方向），Y 轴指向相机上方，符合右手坐标系。

![[image-20241205170522466.png]]

# 应用场景

# 如何转换

# 常用

# 参考文档
【1】https://xiaofengzeng.github.io/kite-notes/gis/cesium/coordinate-system.html
【2】https://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/08%20Coordinate%20Systems/
【3】https://blog.csdn.net/caoshangpa/article/details/80272903

