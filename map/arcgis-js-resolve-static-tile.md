# ArcGIS JS API 4.x 加载 nginx 托管的 ArcGIS 切片文件

## 背景

解决了 [OpenLayers 加载 nginx 托管的 ArcGIS 切片文件](https://juejin.cn/user/2682464100685469) 之后，我试过了使用 [Cesium 实现](https://juejin.cn/post/7310423310168047666)。最后，我还想试试使用 ArcGIS JS API 4.x 加载 nginx 托管的 ArcGIS 切片文件

## 实现代码

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
  <title>CustomTileLayer</title>
  <link rel="stylesheet" href="https://js.arcgis.com/4.28/esri/themes/light/main.css" />

  <style>
    html,
    body,
    #viewDiv {
      padding: 0;
      margin: 0;
      height: 100%;
      width: 100%;
    }
  </style>

  <script src="https://js.arcgis.com/4.28/"></script>

  <script>
    require([
      "esri/Map",
      "esri/views/MapView",
      "esri/layers/BaseTileLayer"
    ], (Map, MapView, BaseTileLayer) => {

      /**
       * 转换数字长度
       * @param {number} num 十进制数
       * @param {number} radix 要转换的进制基数
       * @param {number} length 返回的字符串长度
       */
      function formatNumberLength(num, radix = 10, length = 0) {
        let str = num.toString(radix)
        if (length === 0) {
          return str
        }
        while (str.length < length) {
          str = '0' + str
        }
        return str
      }

      // 自定义图层类
      // 创建 BaseTileLayer 的子类
      const CustomLayer = BaseTileLayer.createSubclass({
        properties: {
          urlTemplate: null,
          spatialReference: null,
          fullExtent: null,
          tileInfo: null
        },

        // URL 模板解析
        getTileUrl: function (level, row, col) {
          return this.urlTemplate
            .replace("{z}", "L" + formatNumberLength(level, 10, 2))
            .replace("{x}", "C" + formatNumberLength(col, 16, 8))
            .replace("{y}", "R" + formatNumberLength(row, 16, 8));
        }
      });

      // 调用切片
      const testLayer = new CustomLayer({
        urlTemplate: "http://localhost:8888/EarthG11/图层/_alllayers/{z}/{y}/{x}.jpg",
        // spatialReference、fullExtent、tileInfo 可以从服务描述文件中获取。
        spatialReference: { wkid: 4490 },
        fullExtent: { xmin: -180, ymin: -90, xmax: 180, ymax: 90 },
        tileInfo: {
          "origin": {
            "x": -180,
            "y": 90
          },
          "spatialReference": {
            "wkid": 4490
          },
          "lods": [
            {
              "level": 0,
              "resolution": 0.7039130078552128,
              "scale": 295828763.7958547
            },
            {
              "level": 1,
              "resolution": 0.3519565039276064,
              "scale": 147914381.89792734
            },
            {
              "level": 2,
              "resolution": 0.1759782519638032,
              "scale": 73957190.94896367
            },
            {
              "level": 3,
              "resolution": 0.0879891259819016,
              "scale": 36978595.474481836
            },
            {
              "level": 4,
              "resolution": 0.0439945629909508,
              "scale": 18489297.737240918
            },
            {
              "level": 5,
              "resolution": 0.0219972814954754,
              "scale": 9244648.868620459
            },
            {
              "level": 6,
              "resolution": 0.0109986407477377,
              "scale": 4622324.4343102295
            },
            {
              "level": 7,
              "resolution": 0.00549932037386885,
              "scale": 2311162.2171551147
            },
            {
              "level": 8,
              "resolution": 0.002749660186934425,
              "scale": 1155581.1085775574
            }
          ]
        },
      });

      // 显示地图
      const view = new MapView({
        container: "viewDiv",
        map: new Map({
          layers: [testLayer]
        }),
        center: [106, 27],
        zoom: 3
      });
    });
  </script>
</head>

<body>
  <div id="viewDiv"></div>
</body>

</html>
```

加载原理同 [OpenLayers 加载 nginx 托管的 ArcGIS 切片文件](https://juejin.cn/user/2682464100685469)。

和 OpenLayers、Cesium 不一样，ArcGIS JS API 的图层加载类没有提供可以重新解析 URL 模板的属性。但是 ArcGIS JS API 提供了自定义切片加载类的解决方案：BaseTileLayer。通过 BaseTileLayer 创建子类时，可以自定义 URL 模板的解析方式。

（完）