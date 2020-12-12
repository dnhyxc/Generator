## 异步编程的方法

1，回调函数。

2，事件监听。

3，发布/订阅。

4，Promise 对象。

Generator 函数则将 JS 异步编程带入了一个全新的阶段。

## 基本概念

### 异步

1，所谓的异步，简单说就是一个任务不是连续完成的，可以理解成该任务被人为分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头来执行第二段。这种不连续的执行，就叫做异步。

2，同步就是连续执行。但由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件时，在读的过程，程序只能干等着。

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

> PS：为什么 Node 约定，回调函数的第一个参数，必须是错误对象 err （如果没有错误，该参数就是 null）？

> 原因就是：执行分成了两段，第一次执行完以后，任务所在的上下文环境就已经结束了，在这以后抛出的错误，原来的上下文环境已经无法捕捉，只能当作参数，传入第二段。

### Promise

1，回调函数本身并没有问题，它的问题出现在多个回调函数嵌套。

```js
fs.readFile(fileA, 'utf-8', function(err, data) {
  fs.readFile(fileB, 'utf-8', function(err, data) {
    // ...
  })
})
```

> 从上述代码中不难看出，如果依次读取两个以上的文件，就会出现多重嵌套，代码不是纵向发展，而是横向发展的，这样就会使代码很混乱，不利于管理。因为多个异步操作形成了强耦合，只要有一个操作需要修改，它的上层回调函数和下层回调函数可能都要跟着修改。这种情况就被称为`回调地狱`。

2，Promise 对象就是为了解决回调地狱而提出的。它不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用。采用 Promise 连续读取多个文件，写法如下：

```js
var readFile = require('fs-readFile-promise');

readFile(fileA)
  .then(function (data) {
    console.log(data.toString());
  })
  .then(function () {
    return readFile(fileB);
  })
  .then(function (data) {
    console.log(data.toString());
  })
  .catch(function (err) {
    console.log(err);
  })
```

> 上面代码中，使用了 fs-readFile-promise 模块，它的作用就是返回一个 Promise 版本的 readFile 函数。Promise 提供 then 方法加载回调函数，catch 方法捕捉执行过程中抛出的错误。可以看到，Promise 的写法只是回调函数的改进，使用 then 方法以后，异步任务的两段执行看的更清楚了。而 Promise 的最大问题就是代码冗余，原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆 then，原来的语义变得很不清楚。

## Generator 函数

### 协程

1，传统的编程语言，早有异步编程的解决方案（其实是多个任务的解决方案）。其中有一种叫做`协程（coroutine）`，意思是多个线程相互协作，完成异步任务。

2，协程有点像函数，有有点像线程。它的运行流程大致如下：

第一步：协程 A 开始执行。

第二步：协程 A 执行到一半，进入暂停，执行权转移到协程 B。

第三步：（一段时间后）协程 B 交还执行权。

第四步：协程 A 恢复执行。

> 上述流程中的 协程A 就是异步任务，因为它分成了两段（或多段）执行。举例来说，读取文件的协程写法如下：

> ```js
> function* asyncJob() {
>   // ...
>   var f = yield readFile(fileA);
>   // ...
> }
> ```

> 上面代码 asyncJob 函数就是一个协程，它的奥妙就在其中的 yield 命令。它表示执行到此处，执行权将交给其他协程。也就是说，yield 命令是异步两个阶段的分界线。协程遇到 yield 命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。它的最大优点，就是代码的写法非常像同步操作，如果去除 yield 命令，简直一摸一样。

### 协程的 Generator 函数实现

1，Generator 函数是协程在 ES6 的实现，最大特点就是可以交出函数的执行权（即暂停执行）。

2，整个 Generator 函数就是一个封装的异步任务，或者说是异步任务的容器。异步操作需要暂停的地方，都用 yield 语句注明。

```js
function* gen(x) {
  var y = yield x + 2;
  return y;
}

var g = gen(1);
console.log(g.next());  // {value: 3, done: false}
console.log(g.next());  // {value: undefined, done: false}
```

> 上面代码中调用 Generator 函数会返回一个内部指针（即遍历器）g。这个 Generator 函数不同于普通函数的另一个地方，就是执行它不会立即返回结果，而是返回一个指针对象。调用指针 g 的next() 方法，会移动内部指针（即执行异步任务的第一段），指向第一个遇到的 yield 语句，上例是执行到 x + 2 为止。换句话说，next() 方法的作用是分阶段执行 Generator 函数，每次调用 next() 方法，会返回一个对象，表示当前阶段的信息（value 属性和 done 属性）。value 属性是 yield 语句后面表达式的值，表示当前阶段的值。done 属性是一个布尔值，表示 Generator 函数是否执行完毕，即是否还有下一个阶段。

### Generator 函数的数据交换和错误处理

1，Generator 函数可以暂停执行和恢复执行，这是它能分装异步任务的根本原因。除此以外，它还有两个特性，使它可以作为异步编程的完整解决方案：`函数体内外的数据交换和错误处理机制`。

2，next() 返回值的 value 属性，是 Generator 函数向外输出数据，next() 方法还可以接收参数，向 Generator 函数体内输入数据。

```js
function* gen(x) {
  var y = yield x + 2;
  return y;
}

var g = gen(1);
g.next(); // {value: 3, done: false}
g.next(2); // {value: 2, done: true}
```

> 上面代码中，第一个 next() 方法的 value 属性，返回表达式 x + 2 的值 3。第二个 next() 方法带有参数 2，这个参数可以传入 Generator 函数，作为上一个阶段异步任务的返回结果，被函数体内的变量 y 接收。因此，这一步的 value 属性，返回的就是 2 （变量 y 的值）。

3，Generator 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。

```js
function* gen(x) {
  try {
    var y = yield x + 2;
  } catch (e) {
    console.log(e);
  }
  return y;
}

var g = gen(1);
g.next();
console.log(g.throw('出错了'));  // 出错了
```

> 上面代码的最后一行，Generator 函数体外，使用指针对象的 throw 方法抛出的错误，可以被函数体内的 try...catch 代码块捕获。这意味着，出错的代码与处理错误的代码，实现了时间和空间上的分离。