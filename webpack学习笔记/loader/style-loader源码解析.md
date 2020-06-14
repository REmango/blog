# style-loader源码解析

**原理：**在js是不具有`import './styles.css';`语法，但是我们可以通过css-loader实现该语法，然后使用style-loader解析将其写入到html。解析的方式分两种：

1. 将css内容通过**<style></style>**内联的形式插入到html中；
2. 输出一个css文件，创建一个link标签添加到html的head（默认）引入css

## 核心代码分析

### 获取配置

```javascript
  // 通过loaderUtils获取webpack中的配置
	const options = loaderUtils.getOptions(this);
	// 验证配置信息
  validateOptions(schema, options, {
    name: 'Style Loader',
    baseDataPath: 'options',
  });
	// 获取插入css的标签， 默认为head
  const insert =
    typeof options.insert === 'undefined'
      ? '"head"'
      : typeof options.insert === 'string'
      ? JSON.stringify(options.insert)
      : options.insert.toString();
  const injectType = options.injectType || 'styleTag';
	// 获取es模块的导入的形式
  const esModule =
    typeof options.esModule !== 'undefined' ? options.esModule : false;

  delete options.esModule;
```

在实际应用中 `injectType` `insert`用的较多

### 模块解析方式

下面主要介绍**linkTag**、**styleTag**两种模式

#### linkTag

```javascript
// css热更新实现，当module.hot为true时更新css模块
const hmrCode = this.hot
        ? `
if (module.hot) {
  module.hot.accept(
    ${loaderUtils.stringifyRequest(this, `!!${request}`)},
    function() {
     ${
       esModule
         ? 'update(content);'
         : `content = require(${loaderUtils.stringifyRequest(
             this,
             `!!${request}`
           )});

           content = content.__esModule ? content.default : content;

           update(content);`
     }
    }
  );

  module.hot.dispose(function() {
    update();
  });
}`
        : '';
			// 根据es模块环境，引入相应的injectStylesIntoLinkTag模块调用，并运行hmrCode
      return `${
        esModule
          ? `import api from ${loaderUtils.stringifyRequest(
              this,
              `!${path.join(__dirname, 'runtime/injectStylesIntoLinkTag.js')}`
            )};
            import content from ${loaderUtils.stringifyRequest(
              this,
              `!!${request}`
            )};`
          : `var api = require(${loaderUtils.stringifyRequest(
              this,
              `!${path.join(__dirname, 'runtime/injectStylesIntoLinkTag.js')}`
            )});
            var content = require(${loaderUtils.stringifyRequest(
              this,
              `!!${request}`
            )});

            content = content.__esModule ? content.default : content;`
      }

var options = ${JSON.stringify(options)};

options.insert = ${insert};

var update = api(content, options);

${hmrCode}

${esModule ? 'export default {}' : ''}`;
```

上面的代码总结为 调用injectStylesIntoLinkTag函数，并实现模块热更新

**injectStylesIntoLinkTag**

```javascript
module.exports = (url, options) => {
  options = options || {};
  // 处理attributes
  // options.attributes = ...
  
  // 创建link标签
  const link = document.createElement('link');

  link.rel = 'stylesheet';
  link.href = url;

  // 设置 link.setAttribut

  // 在html中插入link标签
  if (typeof options.insert === 'function') {
    options.insert(link);
  } else {
    const target = getTarget(options.insert || 'head');

    if (!target) {
      throw new Error(
        "Couldn't find a style target. This probably means that the value for the 'insert' parameter is invalid."
      );
    }

    target.appendChild(link);
  }
  
  // 可以设置新的newUrl
  return (newUrl) => {
    if (typeof newUrl === 'string') {
      link.href = newUrl;
    } else {
      link.parentNode.removeChild(link);
    }
  };
};

```

**styleTag**

injectStylesIntoStyleTag类似上述injectStylesIntoLinkTag的加载，也实现热更新，不同的之处在于函数的入参content（css文件内容）不同，

如下：

```javascript
if (typeof content === 'string') {
              content = [[module.id, content, '']];
            }
// ...
injectStylesIntoLinkTag(content, options)
```

以下是将css写入<style></style>标签的核心

modulesToDom函数

```javascript
function modulesToDom(list, options) {
  const idCountMap = {}; // 遍历时的id状态存储
  const identifiers = []; // 新建一个id数组，用于返回

  for (let i = 0; i < list.length; i++) {
    const item = list[i];
    const id = options.base ? item[0] + options.base : item[0];
    const count = idCountMap[id] || 0;
    const identifier = `${id} ${count}`;

    idCountMap[id] = count + 1;

    const index = getIndexByIdentifier(identifier);
    // 封装css的信息作为入参
    const obj = {
      css: item[1],
      media: item[2],
      sourceMap: item[3],
    };

    if (index !== -1) {
      // stylesInDom存在id则直接进行更新css
      stylesInDom[index].references++;
      stylesInDom[index].updater(obj);
    } else {
      // 添加对象到stylesInDom 并调用addStyle
      stylesInDom.push({
        identifier,
        updater: addStyle(obj, options),
        references: 1,
      });
    }

    identifiers.push(identifier);
  }
```

addStyle

```javascript
  if (options.singleton) {
    const styleIndex = singletonCounter++;
    // 插入<style></style>标签
    style = singleton || (singleton = insertStyleElement(options));
    // 返回一个style中插入css的update函数
    update = applyToSingletonTag.bind(null, style, styleIndex, false);
    remove = applyToSingletonTag.bind(null, style, styleIndex, true);
  } else {
    style = insertStyleElement(options);

    update = applyToTag.bind(null, style, options);
    remove = () => {
      removeStyleElement(style);
    };
  }
```



注意：在style-loader中是不再导出content的，而是直接输出了结果

