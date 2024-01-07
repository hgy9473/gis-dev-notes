Cesium 为空间数据提供了丰富的 API， 可以分为两类：面向图形开发者的接近底层的 `Primitive API` 和用于数据驱动可视化的高级的 `Entity API`。

> primitive 一般理解为图元。Entity 一般理解为实体。

Primitive API 的特点 

- 只暴露了执行任务所需的最少抽象量。
- 功能强大且灵活。

Entity API 的特点
- 暴露了一组设计一致的高级对象，这些对象将相关的可视化和信息聚合到统一的数据结构中。
- 专注于数据的呈现，而不是担心可视化的底层机制。
- 可以轻松构建复杂的时间动态可视化，这种方式与静态数据自然匹配。
- 提供了易于学习且易于使用的接口。



## 参考文档

[1] . [Creating Entities – Cesium](https://cesium.com/learn/cesiumjs-learn/cesiumjs-creating-entities/)