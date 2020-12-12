## 异步编程的方法

1，回调函数。

2，事件监听。

3，发布/订阅。

4，Promise 对象。

Generator 函数则将 JS 异步编程带入了一个全新的阶段。

## 基本概念

### 异步：

1，所谓的异步，简单说就是一个任务不是连续完成的，可以理解成该任务被人为分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头来执行第二段。

### 回调函数

1，JS 语言对异步编程的实现，就是回调函数。所谓的回调函数，就是把任务的第二段单独写在一个函数里面，等到重新执行这个任务的时候，就直接调用这个函数。

2，读取文件进行处理，可以像如下这样操作：

```js
fs.fedaFile('/etc/passwd', 'utf-8', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

> 上述代码中，readFile 函数的第三个参数，就是回调函数，也就是任务的第二段。等到操作系统返回了 /etc/passwd 这个文件之后，回调函数才会执行。
