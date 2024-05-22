
# ClippingPlaneCollection
## 概述
指定一组剪裁平面。剪裁平面有选择地关闭 ClippingPlane 对象列表外部区域中的渲染（只针对gltf模型、3D Tileset 和 地球）。

通常，剪裁平面的坐标相对于它们所附着的对象，因此距离设置为 0 的平面将裁剪穿过对象的中心。

对于 3D Tiles，根切片的变换用于定位剪切平面。如果未定义转换，则改用根图块的 Cesium3DTile#boundingSphere。

## 参数
```js
{
 [planes] // 一个 ClippingPlane 对象数组，用于有选择地禁用每个平面外部的渲染。
 [enabled]  // 是否启用裁剪面
 [modelMatrix] // 指定相对于剪裁平面原始坐标系的附加变换
 [unionClippingRegions]  // 如果为 true，则如果某个区域位于集合中任何平面的外部，则该区域将被裁剪。否则，仅当一个区域位于每个平面的外部时，才会被裁剪。
 [edgeColor] // 用于突出显示剪裁对象的边缘的颜色。
 [edgeWidth] // 边缘的宽度（以像素为单位）。
}
``` 

# ClippingPlane
## 概述
要与 ClippingPlaneCollection 一起使用的 Hessian Normal form 的平面。与平面中的数学函数兼容。

## 参数
```js
{
[normal] // 平面法线（归一化的）
[distance] // 从原点到平面的最短距离。距离符号决定了原点位于平面的哪一侧。如果距离为正，则原点位于法线方向的半空间中;如果为负数，则原点位于与正态相反的半空间中;如果为零，则平面通过原点。
}
```
示例代码
```js

/**
 * points Cartesian3 数组
 * false:保留多边形外，挖除 ; true:保留多边形内,挖出
 */
function areaClipping (points, type = false) {  
  // 获取点坐标，计算  
  const pointsCoor = points.map(({  
    x,  
    y,  
    z  
  }) => new Cesium.Cartesian3(x, y, z))  
  let sum = 0  
  for (let i = 0; i < pointsCoor.length; i++) {  
    const pointA = pointsCoor[i]  
    const pointB = pointsCoor[(i + 1) % pointsCoor.length]  
    const crossProduct = Cesium.Cartesian3.cross(pointA, pointB, new Cesium.Cartesian3()) // 计算pointA和pointB两个向量的叉乘  
    sum += crossProduct.z  
  }  
  
  console.log('summ', sum)  
  if (sum > 0 && type) { // 外裁，需要把点序变成顺序  
    points.reverse()  
  } else if (sum < 0 && !type) { // 内裁：需要把点变成逆序  
    points.reverse()  
  }  
  
  const clippingPlanes = [] // 存储ClippingPlane集合  
  for (let i = 0; i < points.length; ++i) {  
    const nextIndex = (i + 1) % points.length  
  
    // 计算两个坐标点的分量和，取中点。  
    const midpoint = Cesium.Cartesian3.add(points[i], points[nextIndex], new Cesium.Cartesian3())  
    Cesium.Cartesian3.multiplyByScalar(midpoint, 0.5, midpoint)  
  
    // up 是指从地球中心（原点）到 midpoint 的向量，即一个指向地球正上方的单位向量。  
    const up = Cesium.Cartesian3.normalize(midpoint, new Cesium.Cartesian3())  
  
    // 计算points[nextIndex]和midpoint的差值，得到一个表示从 points[nextIndex] 指向 midpoint 的向量  
    const right = Cesium.Cartesian3.subtract(points[nextIndex], midpoint, new Cesium.Cartesian3())  
    Cesium.Cartesian3.normalize(right, right) // 计算单位向量  
  
    // 通过叉乘及归一化得到单位法向量  
    const normal = Cesium.Cartesian3.cross(right, up, new Cesium.Cartesian3())  
    Cesium.Cartesian3.normalize(normal, normal) // 计算单位向量  
  
    const originCenteredPlane = new Cesium.Plane(normal, 0.0)  
    const distance = Cesium.Plane.getPointDistance(originCenteredPlane, midpoint) // 计算平面到中点的距离  
  
    // 最后，我们得到一个平面，这个平面垂直于地球表面  
    clippingPlanes.push(new Cesium.ClippingPlane(normal, distance))  
  }  
  
  // 为地球添加裁剪面  
  this.viewer.scene.globe.clippingPlanes = new Cesium.ClippingPlaneCollection({  
    planes: clippingPlanes,  
    enabled: true,  
    modelMatrix: Cesium.Matrix4.IDENTITY,  
    unionClippingRegions: type, // 内 || 外  
    edgeColor: Cesium.Color.YELLOW,  
    edgeWidth: 1.0  
  })  
  
  this.viewer.scene.globe.backFaceCulling = false  
  this.viewer.scene.globe.showSkirts = false  
}
```

