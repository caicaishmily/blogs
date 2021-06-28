# 题目

```js
// nodejs utils.promisify 接受error first的回调函数为参数异步函数转换为可以返回promise的函数

// fs.loadFile 使用演示
fs.loadFile("./xxx/x.md", (err, data) => {
  if (err) {
    console.log(err);
  } else {
    console.log(data);
  }
});

// promisify函数包装后演示
// fs.loadFile('./xxx.md').then(data => {}).catch(err => {})
const loadFile = utils.promisify(fs.loadFile);
loadFile("./xxxx.md")
  .then((data) => {})
  .catch((err) => {});

// 请实现
utils.promisify = function (fn) {
  //todo
};
```
