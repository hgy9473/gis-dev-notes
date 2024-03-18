# CSS 缩放属性导致 Cesium 缩放地图异常




## 参考资料

计算摄像机地面位置和鼠标地面位置之间的矢量，以便计算向左移动量和向前移动量；这可能需要一个基于高度的缩放因子，类似于缩放量；

```js
camera.zoomIn (zoomAmount);
camera.moveLeft(leftAmount);
camera.moveForward(forwardAmount);  
```

https://community.cesium.com/t/zoom-in-to-mouse-point/2614

https://groups.google.com/g/cesium-dev/c/QSFf3RxNRfE/m/XotNBSBdFQgJ?pli=1

https://github.com/CesiumGS/cesium/pull/2840/commits/29dee9d5f082dfd767ad1fea4ef24315e5710e41

https://github.com/erezy/cesium-2D-zoom-in/blob/master/src/zoom-in.js



https://github.com/CesiumGS/cesium/blob/0fa31b670653d7920ee76aad5a30a20919ff59f6/Source/Scene/ScreenSpaceCameraController.js#L1378