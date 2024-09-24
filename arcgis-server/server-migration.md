# GeoScene 单机 Server 迁移步骤

## 一、在新服务器安装 GeoScene Server。

## 二、配置许可、确认安装成功、停止 GeoScene Server 服务。

## 三、替换文件

### config-store 目录

```js
目录 ./server/usr/config-store/data 下的文件全部替换

目录 ./server/usr/config-store/services/ 下的文件 SampleWorldCities.MapServer , System , Utilities 三个文件夹保留，SampleWorldCities.MapServer.* 文件保留，其他全部替换。
```

### directories 目录

```js
目录 ./server/usr/directories/geoscenesystem/arcgisinput/ 下的 .site ，SampleWorldCities.MapServer , System , Utilities 四个文件夹保留，SampleWorldCities.MapServer.* 文件保留。

目录 ./server/usr/directories/geosceneoutput 下的 .site ，SampleWorldCities_MapServer , System , Utilities 四个文件夹保留

目录 ./server/usr/directories/geoscenecache 下的 .site 文件夹保留

除以上提到的文件之外，directories 目录下其他文件都要替换。
```
## 四、启动 GeoScene Server 服务。