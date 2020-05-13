### 实现一个 trim 函数，去除字符串的首位空格

目的： const str = ' adf df ' trim(str) -> 'adf df'

思路：字符串替换一般使用正则表达式操作，简单明了。

```javascript
const trim = s => s.replace(/(^\s+)|(\s+$)/g, '');
```

注意：别将正则表达式后面的 g 忘了，否则只匹配一次。
