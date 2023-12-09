# OpenLayers 加载 nginx 托管的 ArcGIS 切片文件

## 一、背景

最近遇到一个项目有大约 1000 个切片地图服务要发。然而根据经验，ArcGIS Server 发布的服务超过 100 个之后，稳定性和性能都要受到影响了。项目经费很有限，再加购 ArcGIS Server 有点困难。于是我就想到了一个方法：把地图服务切片做成松散型，然后使用 nginx 托管，前端以 restful 风格去请求切片文件。

## 二、实现思路

1. OpenLayers 支持使用模板链接加载地图服务。（参考示例 [XYZ Esri](https://openlayers.org/en/v6.15.1/examples/xyz-esri.html)）
2. ArcGIS 发布的服务支持使用“服务地址/z/y/x"（ restful 风格）的形式去请求切片。
3. ArcGIS 发布服务后存储的切片是按照层级存放的。（参考[ArcGIS 切片文件结构解析](https://juejin.cn/post/7309013073171316799)）
4. 将“服务地址/z/y/x"指向切片文件的路径即可取到切片文件来显示地图。

## 三、实现步骤

### 1. 先实现加载自己发布的服务。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <link rel="stylesheet" href="//cdn.jsdelivr.net/gh/openlayers/openlayers.github.io@master/en/v6.15.1/css/ol.min.css" type="text/css">
    <script src="//cdn.jsdelivr.net/gh/openlayers/openlayers.github.io@master/en/v6.15.1/build/ol.min.js"></script>
    <style>
      .map {
        height: 400px;
        width: 100%;
      }
    </style>
    
    <!-- 引入坐标转换库 -->
    <script src="//cdn.bootcdn.net/ajax/libs/proj4js/2.7.4/proj4.min.js"></script>
    <title>加载一张地图</title>
  </head>
  <body>
    <div id="map" class="map"></div>
    <script type="text/javascript">

        // 定义坐标系, 这里定义之后后续代码中的坐标系代码 4490 才会有效
        proj4.defs("EPSG:4490","+proj=longlat +ellps=GRS80 +no_defs");
        ol.proj.proj4.register(proj4);

        // 计算各个缩放级别的分辨率
        const maxZoom = 9; // 从服务说明中获取
        const minResolution = 0.7039130078552128; // 从服务说明中获取
        // 计算
        const resolutions = [];
        const matrixIds = [];

        for (let z = 0; z < maxZoom; ++z) {
            resolutions[z] = minResolution / Math.pow(2, z);
            matrixIds[z] = z;
        }

        // 此处用的是笔者自己本机的服务地址。测试此代码时需改为自己的服务地址。
        const mapServerBasePath = 'http://localhost:6080/arcgis/rest/services/EarthG11/MapServer';
        const map = new ol.Map({
            target: 'map',
            layers: [
                new ol.layer.Tile({
                    source: new ol.source.XYZ({
                        url:  mapServerBasePath +'/tile/{z}/{y}/{x}',
                        tileGrid: new ol.tilegrid.WMTS({
                            origin: [-180, 90],// 从服务说明中获取
                            resolutions: resolutions,
                            matrixIds: matrixIds,
                        }),
                        wrapX: true,
                        projection: 'EPSG:4490'
                    })
                })
            ],
            view: new ol.View({
              projection: 'EPSG:4490',
              center: [106, 27],
              zoom: 7
            })
        });
    </script>
  </body>
</html>
```

示例 [XYZ Esri](https://openlayers.org/en/v6.15.1/examples/xyz-esri.html) 的代码很简单，但是自己发的服务坐标系和切片方案都比较特别，所以需要额外补充一些代码。

### 2. 将切片文件使用 nginx 托管。
操作步骤略

### 3. 基于以上代码，改为调用 nginx 托管的切片文件。

```html

<!DOCTYPE html>
<html lang="en">
  <head>
    <link rel="stylesheet" href="//cdn.jsdelivr.net/gh/openlayers/openlayers.github.io@master/en/v6.15.1/css/ol.min.css" type="text/css">
    <script src="//cdn.jsdelivr.net/gh/openlayers/openlayers.github.io@master/en/v6.15.1/build/ol.min.js"></script>
    <style>
      .map {
        height: 400px;
        width: 100%;
      }
    </style>
    
    <!-- 引入坐标转换库 -->
    <script src="//cdn.bootcdn.net/ajax/libs/proj4js/2.7.4/proj4.min.js"></script>
    <title>加载一张地图</title>
  </head>
  <body>
    <div id="map" class="map"></div>
    <script type="text/javascript">

        // 定义坐标系, 这里定义之后后续代码中的坐标系代码 4490 才会有效
        proj4.defs("EPSG:4490","+proj=longlat +ellps=GRS80 +no_defs");
        ol.proj.proj4.register(proj4);

        // 计算各个缩放级别的分辨率
        const maxZoom = 9; // 从服务说明中获取
        const minResolution = 0.7039130078552128; // 从服务说明中获取
        // 计算
        const resolutions = [];
        const matrixIds = [];

        for (let z = 0; z < maxZoom; ++z) {
            resolutions[z] = minResolution / Math.pow(2, z);
            matrixIds[z] = z;
        }

        /**
         * 转换数字长度
         * @param {number} num 十进制数
         * @param {number} radix 要转换的进制基数
         * @param {number} length 返回的字符串长度
         */
        function formatNumberLength (num, radix = 10, length = 0) {
            let str = num.toString(radix)
            if (length === 0) {
                return str
            }
            while (str.length < length) {
                str = '0' + str
            }
            return str
        }

        // 此处用的是笔者自己本机的服务地址。测试此代码时需改为自己的服务地址。
        const mapServerBasePath = 'http://localhost:8888/EarthG11/图层/_alllayers';
        const map = new ol.Map({
            target: 'map',
            layers: [
                new ol.layer.Tile({
                    source: new ol.source.XYZ({
                        url:  mapServerBasePath +'/{z}/{y}/{x}',
                        tileGrid: new ol.tilegrid.WMTS({
                            origin: [-180, 90],// 从服务说明中获取
                            resolutions: resolutions,
                            matrixIds: matrixIds,
                        }),
                        wrapX: true,
                        projection: 'EPSG:4490',
                        // URL 重新构建
                        tileLoadFunction: (imageTile, src) => {
                            const Z = 'L' + formatNumberLength(imageTile.tileCoord[0], 10, 2)
                            const X = 'C' + formatNumberLength(imageTile.tileCoord[1], 16, 8)
                            const Y = 'R' + formatNumberLength(imageTile.tileCoord[2], 16, 8)
                            imageTile.getImage().src = mapServerBasePath + '/' + Z + '/' + Y + '/' + X + '.png'
                        }
                    })
                })
            ],
            view: new ol.View({
              projection: 'EPSG:4490',
              center: [106, 27],
              zoom: 7
            })
        });
    </script>
  </body>
</html>

```

以上代码关键之处在于

（1）服务地址换成了静态文件路径 `http://localhost:8888/EarthG11/图层/_alllayers`。

（2）添加了用于将十进制数转为固定长度的字符串的函数 `formatNumberLength`。根据文章 [ArcGIS 切片文件结构解析](https://juejin.cn/post/7309013073171316799)，切片的层级文件夹名为 L + "2位十进制数"；切片的行号文件夹为 R + "8位16进制数"；切片的列号为 C +  "8位16进制数"；

（3）`ol.source.XYZ` 的参数中添加了 `tileLoadFunction`，用于重新构建切片请求路径。

（完）