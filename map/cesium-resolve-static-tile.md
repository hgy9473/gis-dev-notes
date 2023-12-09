# Cesium 加载 nginx 托管的 ArcGIS 切片文件

## 背景

解决了 [OpenLayers 加载 nginx 托管的 ArcGIS 切片文件](https://juejin.cn/user/2682464100685469) 之后，我还想试试使用 Cesium 实现。

## 实现代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=, initial-scale=">
	<meta http-equiv="X-UA-Compatible" content="">
	<title>loaded-arcgis-static-tile-in-cesium</title>
	<script src="https://cesium.com/downloads/cesiumjs/releases/1.85/Build/Cesium/Cesium.js">
	</script>
	<link href="https://cesium.com/downloads/cesiumjs/releases/1.85/Build/Cesium/Widgets/widgets.css" rel="stylesheet">
    <style>
        #cesiumContainer{
            height: 500px;
            width: 500px;
        }
    </style>
</head>

<body>
	<div id="cesiumContainer"></div>
	<script>
	const viewer = new Cesium.Viewer('cesiumContainer', {
        timeline: false, // 禁用时间线控件
        animation: false, // 禁用动画控件
        homeButton: false, // 禁用主页控件
        geocoder: false, // 禁用地名搜索定位控件
        navigationHelpButton:  false, // 禁用帮助控件
        sceneModePicker: false, // 禁用场景模式切换控件
        baseLayerPicker: false, // 禁用底图切换控件
    });

    viewer.imageryLayers.removeAll()
    // 禁用版权信息控件
    viewer._cesiumWidget._creditContainer.style.display = "none";

    /**
     * 转换数字长度
     * @param {number} num 十进制数
     * @param {number} radix 要转换的进制基数
     * @param {number} length 返回的字符串长度
     */
    function formatNumberLength(num = 0, radix = 10, length = 0){
        let str = num.toString(radix)
        if (length === 0) {
            return str
        }
        while (str.length < length) {
            str = '0' + str
        }
        return str
    }

    // 加载图层
    viewer.imageryLayers.addImageryProvider(
        new Cesium.UrlTemplateImageryProvider({
            url: 'http://localhost:8888/EarthG11/图层/_alllayers/{zz}/{yy}/{xx}.jpg',
            rectangle: Cesium.Rectangle.fromDegrees(-180, -90, 180, 90),
            minimumLevel: 0,
            maximumLevel: 8,
            // CGCS2000坐标系的切片使用GeographicTilingScheme切片方案也可以加载，目前没有发现异常
            tilingScheme: new Cesium.GeographicTilingScheme(),
            // 解析自定义占位符号
            customTags: {
                xx: (imageryProvider, x, y, level) => {
                    const xx = 'C' + formatNumberLength(x, 16, 8)
                    return xx
                },
                yy: (imageryProvider, x, y, level) => {
                    const yy = 'R' + formatNumberLength(y, 16, 8)
                    return yy
                },
                zz: (imageryProvider, x, y, level) => {
                    const zz = 'L' + formatNumberLength(level, 10, 2)
                    return zz
                },
            },
        })
    )

	</script>
</body>

</html>
```

加载原理同 [OpenLayers 加载 nginx 托管的 ArcGIS 切片文件](https://juejin.cn/user/2682464100685469)。只是 API 使用方法不一样。 

（完）