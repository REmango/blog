# webpack核心概念

webpack的几大核心概念如下：

- **Entry**：入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。
- **Module**：模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。
- **Chunk**：代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割。
- **Loader**：模块转换器，用于把模块原内容按照需求转换成新内容。简单来说，webpack只能将非标准文件转换成理想的文件。
- **Plugin**：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。webpack为plugin提供了广泛的钩子，几乎能够使我们在webpack各大生命周期内进行操作。
- **Output**：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。
- **Manifest:** webpack在runtime时管理所有模块的交互生成的代码。
- **HMR：** 模块热更新，在应用程序运行过程中替换、添加或删除模块，而无需重新加载整个页面。这是webpack-dev-server提供的功能，单独拿出来说是因为使用比较广泛和频繁