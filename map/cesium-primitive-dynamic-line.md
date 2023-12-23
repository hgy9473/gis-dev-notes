# 深度剖析 Cesium 使用 Primitive 实现动态线

## 背景

在 Cesium 中实现加载动态线的功能。会遇到两个问题，一个是使用 Entity 还是 Primitive；另一个是动态材质怎么实现。

第一个问题我选择 Primitive，因为测试发现数据量上万时 Entity 加载开始有明显的卡顿。第二个问题真不容易，因为可参考的资料不多，也没有人仔细讲原理。

## 先看一个示例
```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
  <title>test</title>
  <link rel="stylesheet" href="//cdn.bootcdn.net/ajax/libs/cesium/1.97.0/Widgets/widgets.css" />

  <style>
    #cesiumContainer {
      height: 500px;
      width: 500px;
    }
  </style>

  <script src="//cdn.bootcdn.net/ajax/libs/cesium/1.97.0/Cesium.js"></script>

</head>

<body>
  <div id="cesiumContainer"></div>
  <script>
    const viewer = new Cesium.Viewer('cesiumContainer', {
      infoBox: false,
      timeline: false, // 禁用时间线控件
      animation: false, // 禁用动画控件
      homeButton: false, // 禁用主页控件
      geocoder: false, // 禁用地名搜索定位控件
      navigationHelpButton: false, // 禁用帮助控件
      sceneModePicker: false, // 禁用场景模式切换控件
      baseLayerPicker: false, // 禁用底图切换控件
    });

    viewer.imageryLayers.removeAll()
    // 禁用版权信息控件
    viewer._cesiumWidget._creditContainer.style.display = "none";

    // 添加底图
    viewer.imageryLayers.addImageryProvider(new Cesium.ArcGisMapServerImageryProvider({
      url: "//map.geoq.cn/arcgis/rest/services/ChinaOnlineStreetPurplishBlue/MapServer",
    }))

    // 设置视角
    viewer.camera.setView({
      "destination": {
        "x": -1909713.5752720681,
        "y": 6335549.864149072,
        "z": 2681442.787104271
      },
      "orientation": {
        "heading": 6.262743028873723,
        "pitch": -0.843392801136492,
        "roll": 0.000022777458393186123
      }
    })

    // 图形
    const geometryInstance = new Cesium.GeometryInstance({
      geometry: new Cesium.PolylineGeometry({
        positions: Cesium.Cartesian3.fromDegreesArray([
          102.536539, 30.301741,
          110.683813, 30.301741,
          110.683813, 26.037747,
          102.536539, 26.037747,
          102.536539, 30.301741
        ]),
        width: 20,
        vertexFormat: Cesium.PolylineMaterialAppearance.VERTEX_FORMAT,
      })
    })

    // 材质图片，此处用的是图片的base64编码
    // 原图片地址：https://www.freeimg.cn/i/2023/12/11/65767ffac43f3.png
    const image = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAAgCAYAAABkS8DlAAAACXBIWXMAAAsTAAALEwEAmpwYAAAAIGNIUk0AAHolAACAgwAA+f8AAIDpAAB1MAAA6mAAADqYAAAXb5JfxUYAAADSSURBVHja7NYxEoUgDEDBYM39z2qHtZViwMFxt1FJnF/98ZXWWkRE7LWWOOt5Lsm9q/vsbu9Zdtazs/J19O5bs1XPZrwze/6V31zxbOZs1n905Wt2p3f25GzE7ohv6q3nLQCA3xEAACAAAAABAAAIAABAAAAAAgAAEAAAgAAAAAQAACAAAAABAAAIAABAAAAAAgAAEAAAgAAAAAQAACAAAEAAAAACAAAQAACAAAAABAAAIAAAAAEAAAgAAEAAAAACAAAQAACAAAAA8g4AAAD//wMA4WEFTJOT5UIAAAAASUVORK5CYII=';

    // 自定义材质
    const material = new Cesium.Material({
      translucent: true,
      fabric: {
        uniforms: {
          color: new Cesium.Color(1.0, 1.0, 1.0, 0.5),
          image: image,
          speed: 20,
        },
        source: `
            czm_material czm_getMaterial(czm_materialInput materialInput) {
              czm_material material = czm_getDefaultMaterial(materialInput);
              vec2 st = materialInput.st;
              float st_map_s = fract(st.s - speed * czm_frameNumber * 0.001);
              vec4 colorImage = texture2D(image, vec2(st_map_s, st.s));
              vec4 fragColor;
              fragColor.rgb = color.rgb / 1.0;
              material.alpha = colorImage.a * color.a;
              material.diffuse = fragColor.rgb;
              material.emission = fragColor.rgb;
              return material;
            }
          `,
      },
    });

    // 显示图形
    const primitive = new Cesium.Primitive({
      geometryInstances: [geometryInstance],
      appearance: new Cesium.PolylineMaterialAppearance({
        material: material
      }),
      asynchronous: false,
    });

    viewer.scene.primitives.add(primitive)
  </script>
</body>

</html>

```
以上这段代码主要参考了 [前端3D引擎-Cesium自定义动态材质](https://juejin.cn/post/7054970604104974344)。

代码可以直接运行，运行效果如下：

![gif](../assets/images/dynamic-line.gif)

以上代码的主干逻辑还比较好理解，也有足够的资料可以参考。但是 fabric 的配置中有一串代码中的代码实在不容易理解。

```c
czm_material czm_getMaterial(czm_materialInput materialInput) {
  czm_material material = czm_getDefaultMaterial(materialInput);
  vec2 st = materialInput.st;
  float st_map_s = fract(st.s - speed * czm_frameNumber * 0.001);
  vec4 colorImage = texture2D(image, vec2(st_map_s, st.s));
  vec4 fragColor;
  fragColor.rgb = color.rgb / 1.0;
  material.alpha = colorImage.a * color.a;
  material.diffuse = fragColor.rgb;
  material.emission = fragColor.rgb;
  return material;
}
```

动态纹理的的难点就在这一段代码。难在哪里呢？

（1）这段代码使用了一种一般开发人员不熟悉的语言（GLSL, 全称 OpenGL Shading Language，是一种 C 风格的着色器语言）<sup>[4]</sup>;

（2）Cesium 官方对于这一块代码涉及的内容没有提供足够的参考资料。

（3）网上能查到的可参考的资料也不多。大多数博客都是给出代码，并不解释代码原理。

（4）即使仔细阅读每一句代码也不一定能理解为什么就这样变成了动态纹理。

本文尝试对这段代码稍微解释仔细一点。

## 先从整体上看看

整体代码结构
```c
czm_material czm_getMaterial(czm_materialInput materialInput) {
  czm_material material = czm_getDefaultMaterial(materialInput);

  return material;
}
```

从整体上看，这段代码有如下几个特点：

（1）以字符串的形式赋值给 `source` 属性。
（2）这段代码是一个函数定义，函数头固定为 `czm_material czm_getMaterial(czm_materialInput materialInput)`，函数返回值固定为 `czm_material` 对象，这个对象用于给图形上的一个像素着色。。
（3）这段代码中 `czm_` 开头的变量类型和函数不属于 GLSL，是 Cesium 封装的。
（4）修改 `material` 变量就可以修改图形的纹理。

> 思考一个问题，这段代码会被调用多少次呢？
> 
> 调用一次只能设置一个像素的颜色，所以调用次数跟构成图形的像素数量有关。
> 
> 根据计算机图形学，我们在显示器上看到的画面是不断刷新的，即使是静态图片，显示时也在不断的刷新画面。刷新的单位是帧，刷新的频率叫帧率,帧率的意思表示画面每秒被绘制的次数。
> 
> 在 Cesium 程序中添加代码 `viewer.scene.debugShowFramesPerSencond = true` 即可看到帧率。 
> 
> 所以这段代码调用次数为 pixels(构成图形的像素数量) x fps（帧率） x seconds（图形显示的秒数）


## 细看每一句代码

```c
czm_material czm_getMaterial(czm_materialInput materialInput) {
  czm_material material = czm_getDefaultMaterial(materialInput);
  vec2 st = materialInput.st;
  float st_map_s = fract(st.s - speed * czm_frameNumber * 0.001);
  vec4 colorImage = texture2D(image, vec2(st_map_s, st.s));
  vec4 fragColor;
  fragColor.rgb = color.rgb / 1.0;
  material.alpha = colorImage.a * color.a;
  material.diffuse = fragColor.rgb;
  material.emission = fragColor.rgb;
  return material;
}
```

(1) `czm_material` 类型
`czm_material` 是着色器代码的返回值类型，是一个结构体<sup>[2]</sup>
```c
struct czm_material
{
  vec3 diffuse; // 漫反射颜色，设置这个值会影响图形的颜色。
  float specular; // 镜面反射强度（高光）。
  float shininess; // 镜面反射的锐度（光泽度），值越高，会创建更小、更聚焦的镜面反射高光。
  vec3 normal; // 法线。
  vec3 emission; // 自发光颜色。
  float alpha; // 材质透明度，0.0表示完全透明；1.0表示完全不透明。
};
```

(2) `czm_materialInput` 类型
`czm_materialInput` 是着色器代码的输入值类型，也是一个结构体<sup>[2]</sup>

```c
struct czm_materialInput
{
  float s; // 一维材质坐标
  vec2 st; // 二维材质坐标
  vec3 str; // 三维材质坐标
  mat3 tangentToEyeMatrix; // 片元切线空间到视点坐标系的转换矩阵，用于法线贴图、凹凸贴图等。 
  vec3 positionToEyeEC; // 从片元到视点之间的向量，用于反射、折射等计算。向量的模表示了从片元到视点的距离。
  vec3 normalEC; // 片元在视点空间的（标准化的）法向量，用于凹凸贴图、反射、折射等计算。
};
```












## 参考资料

[1] . [前端3D引擎-Cesium自定义动态材质](https://juejin.cn/post/7054970604104974344)

[2] . [Cesium Wiki Fabric](https://github.com/CesiumGS/cesium/wiki/Fabric)

[3] . https://cesium.com/downloads/cesiumjs/releases/b28/Documentation/czm_material.html

[4] . [OpenGL Shading Language Wiki](https://www.khronos.org/opengl/wiki/OpenGL_Shading_Language)