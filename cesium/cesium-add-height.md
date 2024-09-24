# Cesium 增加点位高度的问题


```js
Cesium boundCenterTop.z = boundCenterTop.z + layers * layerHeight + 8 //倾斜了

function Cartesian3AddHeight(point, height) {
  const ellipsoid = viewer.scene.globe.ellipsoid
  const cartographic = ellipsoid.cartesianToCartographic(point)

  return Cartesian3.fromRadians(cartographic.longitude, cartographic.latitude, cartographic.height + height)
}
```