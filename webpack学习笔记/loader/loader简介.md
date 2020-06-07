# loader简介

webpack 可以使用 loader来预处理文件。这允许你打包除 JavaScript 之外的任何静态资源。你可以使用 Node.js 来很简单地编写自己的 loader。

换句话说，loader的本质是通过node的文件io时，可以对文件

常见的文件如下

- vue-loader 将代码作为模块执行，并将 exports 转为 JS 代码
- url-loader 像 file loader 一样工作，但如果文件小于限制，可以返回 [data URL](https://tools.ietf.org/html/rfc2397)
- file-loader 将文件发送到输出文件夹，并返回（相对）URL
- Json5-loader 加载和转译 JSON 5 文件