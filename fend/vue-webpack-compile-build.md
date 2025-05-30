# 一、环境说明
```shell
node v14.21.3
vue@2.7.16
webpack@5.99.9
```
# 二 、HTML 处理
2.1 `html-webpack-plugin` 

用于生成根据模板和配置生成、更新网页运行需要的 HTML 文件，以及为网页注入资源等。

样例：

```html
<!-- HTML 模板文件 -->
<!DOCTYPE html>  
<html lang="en">  
<head>  
  <meta charset="UTF-8">  
  <meta name="viewport" content="width=device-width, initial-scale=1.0">  
  <title><%= htmlWebpackPlugin.options.title %></title>  
</head>  
<body>  
  <div id="app"></div>  
</body>  
</html>
```


```js
// webpack.config.js 片段
const path = require("path"); // 引用path模块
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = (env) => {
  return {
    // ...
    plugins: [
      new HtmlWebpackPlugin({
        title: "title@HtmlWebpackPlugin",
        template: path.join(__dirname, "./src/index.html"),
        filename: "index.html", //输出的文件名
      })
    ]
    // ...
  };
};
```
# 三、Vue 处理
3.1 `vue-loader`

（1）用于解析 Vue 文件，将 template、script、style 分别提取出来，并进行相应处理。

- template 会被编译成 JavaScript 语言的渲染函数，一边 Vue.js 的渲染器可以使用。
  
- script 会被作为模块进行处理，还会结合其他 loader(如 babel-loader)进行处理。
  
- style 会被提取出来，根据配置进行处理，可以结合 sass-loader 等进行处理。
  
（2）为开发过程提供热更新支持。

3.2 `vue-style-loader`

用于将样式文件动态注入到页面中，支持热更新、CSS模块处理、作用域样式。主要应用于开发环境。

`vue-loader` 默认依赖 `vue-style-loader`，所以大多数情况不需要配置这个 loader。

# 四、CSS 处理

4.1 `css-loader`

是 Webpack 中处理 CSS 文件的核心 loader 。

(1) 解析 CSS 文件中的 @import 和 url() 规则。

(2) 将 CSS 转换为 JavaScript 模块,以便 Webpack 进行后续处理。

(3) 解析 CSS 模块化语法、Source Map 等。

4.2 `style-loader`

主要作用为将 CSS 样式动态注入 HTML 页面，支持热更新、CSS 模块处理、SourceMap 等，主要用于开发环境。

4.3 `sass-loader`

用于将 SASS/SCSS 代码转换为 CSS 代码。

4.4 `postcss-loader`

用于实现 CSS 兼容性处理、代码优化、清理未使用的 CSS。

4.5 `mini-css-extract-plugin`

将 CSS 提取到单独的文件中。适用于生产环境。

4.6 `css-minimizer-webpack-plugin`

对生成的 CSS 文件进行压缩和优化，减少文件大小。适用于生产环境。

# 五、JS 处理

# 六、其他文件处理

6.1 `file-loader`

将文件（如字体文件、图片等）复制到输出目录，并返回文件的路径。
