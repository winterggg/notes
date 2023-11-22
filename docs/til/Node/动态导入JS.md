# 动态导入 JS

调试时可能需要动态导入一个可能变化的 js 文件。

可以用 `import()`，配合清除缓存使用：

```ts
// 假设你要重新加载的模块是 './dynamic.js'
const path = require('path');
const modulePath = path.resolve(__dirname, './dynamic.js');

// 清除特定模块的缓存
if (require.cache[modulePath]) {
    delete require.cache[modulePath];
}

// 重新加载模块
const dynamicModule = require('./dynamic.js');
```



自定义模块加载器

```ts
const fs = require('fs');
const vm = require('vm');

function customRequire(modulePath) {
    // 读取模块文件的内容
    const moduleCode = fs.readFileSync(modulePath, 'utf8');

    // 创建一个新的模块对象
    const module = { exports: {} };

    // 包装模块代码
    const wrapper = `(function(module, exports) {
        ${moduleCode}
    })(module, module.exports);`;

    // 执行模块代码
    vm.runInThisContext(wrapper);

    // 返回模块的导出
    return module.exports;
}

// 使用自定义加载器加载模块
const dynamicModule = customRequire('./dynamic.js');

```





---

src: chatgpt