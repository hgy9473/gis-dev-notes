# Cesium 计算指定点的高度

高度可能来自地形、模型、图形。

## clampToHeight

> clampToHeight(cartesian: Cartesian3, objectsToExclude: Array.<Object>, width: number, result: Cartesian3): Cartesian3

将给定的坐标位置沿大地表面法线与场景几何体贴紧。返回贴紧的位置。如果没有场景几何体可贴紧，则返回`undefined`。

可用于将物体贴紧到场景中的地表、三维瓦片或图元上。

对于球面切片和 3D Tiles，此函数仅对当前视图中已渲染完成的切片有效。对于图元，无论是否可见都有效。

## clampToHeightMostDetailed

> clampToHeightMostDetailed(cartesians: [Cartesian3], objectsToExclude:Array[Object] , width: number) → Promise.<Array.<Cartesian3>>

使用场景中切片的最高 LOD，对一组坐标执行异步 Scene.clampToHeight 查询。每个位置的高程值会被直接修改。如果某个位置没有图形，或发生其他错误，数组中的元素将被设置为 `undefined`。

## sampleHeightMostDetailed

> sampleHeightMostDetailed(positions: Array.<Cartographic>, objectsToExclude: Array.<Object>, width:number) → Promise.<Array.<Cartographic>>

使用场景中切片的最大 LOD，对`Cartographic`数组执行异步 Scene.sampleHeight 查询。输入位置的高度将被忽略。每个点的高程值会被直接修改。如果由于某个点因没有图形而无法确定高度，或者发生其他错误，则高度将设置为 `undefined`。

