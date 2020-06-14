# css-loader源码解析

> 此版本为[webpack-contrib](https://github.com/webpack-contrib)官方版本

在css是不具有`@import`语法，但是我们可以通过css-loader实现该语法，其核心是esModule的实现。

**## Options**

| Name                                                         | Type                      | Default  | Description                                                  |
| ------------------------------------------------------------ | ------------------------- | -------- | ------------------------------------------------------------ |
| **[`url`](https://github.com/webpack-contrib/css-loader#url)** | `{Boolean|Function}`      | `true`   | Enables/Disables `url`/`image-set` functions handling        |
| **[`import`](https://github.com/webpack-contrib/css-loader#import)** | `{Boolean|Function}`      | `true`   | Enables/Disables `@import` at-rules handling                 |
| **[`modules`](https://github.com/webpack-contrib/css-loader#modules)** | `{Boolean|String|Object}` | `false`  | Enables/Disables CSS Modules and their configuration         |
| **[`sourceMap`](https://github.com/webpack-contrib/css-loader#sourcemap)** | `{Boolean}`               | `false`  | Enables/Disables generation of source maps                   |
| **[`importLoaders`](https://github.com/webpack-contrib/css-loader#importloaders)** | `{Number}`                | `0`      | Enables/Disables or setups number of loaders applied before CSS loader |
| **[`localsConvention`](https://github.com/webpack-contrib/css-loader#localsconvention)** | `{String}`                | `'asIs'` | Style of exported classnames                                 |
| **[`onlyLocals`](https://github.com/webpack-contrib/css-loader#onlylocals)** | `{Boolean}`               | `false`  | Export only locals                                           |
| **[`esModule`](https://github.com/webpack-contrib/css-loader#esmodule)** | `{Boolean}`               | `false`  | Use ES modules syntax                                        |

