# scene.pick

常用于拾取Entity、Primitive、3D Tiles

**`scene.pick(windowPosition: {x: number, y: number}, width=3, height=3)`**
> windowPosition 窗口坐标
> width 拾取矩形的宽度
> height 拾取矩形的高度

> 返回一个具有 `primitive`  属性的对象，该属性包含特定窗口坐标处场景中的第一个（顶部）图形。如果该位置没有任何图形，则返回 `undefined` 。根据图形的类型，可能会设置其他属性，这些属性可用于进一步识别选中的对象。拾取3D Tiles切片集中的要素时，会返回一个 `Cesium3DTileFeature` 对象。

# scene.pickPosition

常用于拾取模型、地形表面的坐标

**`pickPosition(windowPosition: {x: number, y: number})`**

> 返回根据深度缓冲区和窗口位置重建的坐标位置。
> 从 2D 深度缓冲区重建的位置可能与在 3D 和 Columbus 视图中重建的位置略有不同。这是由透视投影和正交投影的深度值分布差异引起的。
> 将 Scene.pickTranslucentDepth 设为 true，以包含半透明基元的深度；否则，此功能基本上会穿过半透明基元。

# scene.drillPick

适用于多个对象堆叠在一起时要拾取多个对象的情况。可以理解为穿透拾取，是从当前拾取位置获取所有对象的列表，列表按照其在场景中的视觉顺序从前到后排列。

**`drillPick(windowPosition: {x: number, y: number}, limit, width=3, height=3)`**

limit 参数可以限制最多拾取几个对象。

# globe.pick

查找射线与渲染的地球仪表面的交点。射线必须以世界坐标表示。一般用于获取鼠标点击处的地球表面坐标（包含地形，不包含模型）。

**`pick(ray, scene)`**
```js
// 示例代码
const ray = viewer.camera.getPickRay(windowPosition);
const intersection = globe.pick(ray, scene);
```

# camera.pickEllipsoid

拾取椭球体（数学模型）表面的位置。拾取到的坐标的高程始终是0。

**`pickEllipsoid(windowPosition, ellipsoid)`**

