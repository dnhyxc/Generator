## Generator

### Generator 概念

1. Generator 函数是 ES6 提供的一种异步编程的解决方案。

2. 与普通函数不一样的是 Generator 会在函数名后面加上一个 `*` 区别于普通函数，并且会在函数体内部使用 yield 表达式定义不同的内部状态。

3. 调用 Generator 函数时，函数并不会立即执行，而是需要使用 next() 方法之后才会执行。

4. 执行 Generator 函数返回的并不是函数执行的结果，而是返回一个遍历器对象（指向内部状态的指针对象），这个遍历器对象中存在两个值，`value`：为执行函数体中 yield 表达式的值，`done`：为一个 boolean 值，表示遍历是否结束，如果值为 false 则表示未结束，否则即为结束。

5. Generator 函数可以返回一系列的值，因为 Generator 函数中可以同时有任意多个 yield。

### yield 表达式

1. 执行 Generator 函数时，遇到 yield 表达式，就会暂停执行后面的操作，并将紧跟在 yield 后面表达式的值作为返回对象的 value 属性值。

2. 在下一次调用 next() 方法时，再继续往下执行，直到遇到下一个 yield 表达式。

3. 如果没有再遇到新的 yield 表达式，就一直运行到函数结束，直到 return 语句为止，并将 return 语句后面的表达式的值作为返回对象的 value 属性值。

4. 如果该函数没有 return 语句，则返回对象的 value 属性值为 undefined。

5. yield 后面的表达式只有当调用 next() 方法、内部指针指向该语句时才会执行。

6. yield 表达式与 return 语句既有相似之处，也有区别：

   - 相同点：都能返回紧跟在语句后面的那个表达式的值。

   - 不同点：yield 具有记忆功能，每次遇到 yield 时，函数暂停执行，下一次再从该位置继续向后执行，而return 语句不具备位置记忆功能。并且在一个函数中只能执行一次 return 语句，但是可以执行多次（或者说多个）yield 表达式。

7. **注意**：yield 表达式只能在 Generator 函数中使用，在普通函数中使用都将包语法错误。

    ```js
    var arr = [1, [[2, 3], 4], [5, 6]];

    var flat = function* (a) {
      a.forEach(function (item) {
        if (typeof item !== 'number') {
          yield* flat(item);
        } else {
          yield item;
        }
      });
    };

    for (var f of flat(arr)) {
      console.log(f);
    }
    ```

   上面代码将产生语法错误，因为 forEach() 方法中的参数是一个普通函数，所以在其中使用 yield 表达式就会产生语法错误。可以将代码改成如下形式解决错误：
  
    ```js
    var arr = [1, [[2, 3], 4], [5, 6]];
  
    var flat = function* (a) {
      var length = a.length;
      for (var i = 0; i < length; i++) {
        var item = a[i];
        if (typeof item !== 'number') {
          yield* flat(item);
        } else {
          yield item;
        }
      }
    };
  
    for (var f of flat(arr)) {
      console.log(f); // 1, 2, 3, 4, 5, 6
    }
    ```

8. yield 表达式用于另一个表达式中时，必须放在圆括号里面。

    ```js
    function* generator() {
      console.log('Hi' + yield);  // SyntaxError
      console.log('Hi' + yield 123);  // SyntaxError

      console.log('Hi' + (yield));  // Ok
      console.log('Hi' + (yield 123));  // Ok
    }
    ```

9.  yield 表达式用作函数参数或放在赋值表达式的右边，可以不加括号。
  
    ```js
    function* generator() {
      foo(yield 'a', yield 'b');  // OK
      let input = yield;  // OK
    }
    ```

### Generator 与 Iterator 接口的关系

1. **说明**：由于数组、类数组对象、Map、Set 结构都具有 iterator 接口，而对象并不具备，如果要使对象具有 iterator 接口，即可使用 Generator 函数实现。因为 Generator 函数是一个遍历器生成函数，因此可以把 Generator 赋值给对象的 Symbol.iterator 属性，从而使得该对象具有 Iterator 接口。

    ```js
    var myIterable = {};
    myIterable[Symbol.iterator] = function* () {
      yield 1;
      yield 2;
      yield 3;
    };
    console.log([...myIterable]) // [1, 2, 3]
    console.log(...myIterable); // 1, 2, 3
    ```

2. Generator 函数执行后，返回一个遍历器对象，该对象本身也具有 Symbol.iterator 属性，因此执行后会返回自身（执行 Generator 返回的遍历器对象）。

    ```js
    function* gen(){
      yield 1;
    }

    const g = gen();
    g[Symble.iterator]() === g;
    ```

   上述代码中，gen 是一个 Generator 函数，调用它会生成一个遍历器对象 g，而它的 Symbol.iterator 属性，也是一个遍历器对象生成函数，执行后返回它自己。

### next 方法的参数

1. yield 表达式本身没有返回值，或者说返回 undefined，而 next() 方法可以带一个参数，该参数就会被当作上一个 yield 表达式的返回值。

    ```js
    function* fun() {
      for(var i = 0; true; i++) {
        var reset = yield i;
        if(reset) {
          i = -1;
        }
      }
    }
    var g = fun();
    g.next(); // { value: 0, done: false }
    g.next(); // { value: 1, done: false }
    g.next(true); // { value: 0, done: false }
    ```

   由上述代码可以看出，当 next() 方法携带一个参数 true 时，变量 reset 就被重置为这个参数（即 true），因此 i 会等一 -1，下一轮循环就会从 -1 开始递增。由此可见 next() 方法携带参数可以在 Generator 函数开始执行之后，继续向函数体内部注入值，也就是说可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。而 Generator 函数从暂停状态到恢复运行，它的上下文状态是不变的。

    ```js
    function* foo(x) {
      var y = 2 * (yield (x + 1));
      var z = yield(y / 3);
      return (x + y + z);
    }

    var a = foo(5);
    a.next(); // Object {value: 6, done: false }
    a.next(); // Object {value: NaN, done: false }
    a.next(); // Object {value: NaN, done: false }

    var b = foo(5);
    b.next(); // {value: 6, done: false}
    b.next(12); // {value: 8, done: false}
    b.next(13); // {value: 42, done: true}

    function* test(x) {
      var y = 2 + (yield(x * 3));
        /*
          - 如果(yield(y - 3))被（）包裹了，
          那么本次就不会加上后面的1，而是直接返回(yield(y - 3))。
          - 如果yield(y - 3)没有被（）包裹，
          那么本次计算就会加上后面的1再返回结果（即为yield(y - 3) + 1的值）。
        */
        var z = (yield(y - 3)) + 1;
        return x + y + z; // X=1, y=4, z=4 
    }

    const t = test(1);
    console.log(t.next()); // {value: 3, done: false}
    console.log(t.next(2)); // {value: 1, done: false}
    console.log(t.next(3)); // {value: 9, done: true}
    ```

2. 如果在第一次调用 next() 方法时传入参数，会被忽略，即第一次调用 next() 方法时传入的参数是无效的。从第二次开时才会是有效的。如果想要在第一次执行 next() 方法时就传入参数，则需要在 Generator 函数外面再包一层函数，代码如下：

    ```js
    function wrapper(generatorFun) {
      retutn function (...args) {
        let generatorObject = generatorFun(...args);
        generatorObject.next();
        return generatorObject;
      }
    }

    const wrapped = wrapper(function* (){
      console.log(`First input: ${yield}`); // First input: dnhyxc
      return 'DONE';
    });

    console.log(wrapped().next('dnhyxc')); // {value: "DONE", done: true}
    ```  

### for...of 循环

1. for...of 循环可以自动遍历 Generator 函数运行时生成的 iterator 对象，且不再需要调用 next() 方法。
  
    ```js
    function* foo() {
      yield 1;
      yield 2;
      yield 3;
      yield 4;
      yield 5;
      return 6;
    }
 
    for(let v of foo()) {
      console.log(v);  // 1 2 3 4 5
    }
    
    const fo = foo();
    console.log(fo.next()); // {value: 1, done:false}
    console.log(fo.next()); // {value: 2, done:false}
    console.log(fo.next()); // {value: 3, done:false}
    console.log(fo.next()); // {value: 4, done:false}
    console.log(fo.next()); // {value: 5, done:false}
    console.log(fo.next()); // {value: 6, done:true}
    ```
    上述代码使用 for...of 循环，依次显示 5 个 yield 表达式的值，但是一旦 next() 方法的返回对象 中的 done 属性为 true 时，for...of 循环就会终止，且不包含该返回对象，因此 return 6 时，由 于 done 已变为 true，所以该返回对象不会包括在 for...of 循环中，也就不会打印出 6。

2. 使用 for...of 和 Generator 函数实现斐波那契数列，示例如下：

    ```js
    function* fibonacci() {
      let [prev, curr] = [0, 1];
      for (;;) {
        yield curr;
        [prev, curr] = [curr, prev + curr];
        /*
         curr: 1, prev: 0
         curr: 2, prev: 1
         curr: 3, prev；2
         curr: 5, prev: 3
         curr: 8, prev: 5
         分析：从第二次fibonacci函数执行时，curr的值为上一次prev的值加上上一次的curr的值，
         因此第二次curr的值为第一次返回的curr的值1加上上一次prev的值0，
         因此第二次返回的值还是curr为1。再次执行时，prev的值已经被curr重新赋值为上一次curr的值1 了，
         因此第三次curr的结果就是1+1的结果2了。
        */
       }
    }
 
    for (let n of fibonacci()) {
      if (n > 1000) break;
      console.log(n);
    }
    ```

3. 通过 Generator 函数可以对象拥有 iterator 接口，可以使用 for...of 进行遍历接口。

    ```js
    function* objectEntries(obj) {
      let propKeys = Reflect.ownkeys(obj);
 
      for(let propKey of propKeys) {
        yield [propKey, obj[propKey]];
      }
    }
 
    let dnh = {first: 'dnh', last: 'dnhyxc'}
 
    for(let [key, value] of objectEntries(dnh)) {
      console.log(`${key}: ${value}`);
      // first: dnh
      // last: dnhyxc
    }
 
    const resObj = objectEntries(dnh);
    console.log(resObj.next()); // {value:["first", "dnh"], done: false}
    ```

    上述代码中，对象 dnh 原生不具备 iterator 接口，无法使用 for...of 遍历，但是通过 Generator 函数为其加上遍历器接口后，就可以使用 for...of 进行遍历了。还可以将 Generator 函数加到对象的 Symbol.iterator 属性上面，同样可以实现使用 for...of 进行遍历。

    ```js
    function* objectEntries1() {
      let propKeys = Object.keys(this);
 
      for (let propKey of propKeys) {
        yield [propKey, this[propKey]];
      }
    }
 
    let wyh = {
      first: 'wyh',
      last: 'wyhnxc'
    };
 
    wyh[Symbol.iterator] = objectEntries1;
 
    for (let [key, value] of wyh) {
      console.log(`${key}: ${value}`);
      // first: wyh
      // last: wyhnxc
    }
 
    const arrrr = [1, 2, 3, 4, 5]
    const resss = arrrr[Symbol.iterator]()
    console.log(resss.next()); // {value:1 ,done: false}
 
    const objjj = {
      name: 'aaa',
      age: 26
    }
    const objRes = objjj[Symbol.iterator](); // 报错，因为对象不具备 iterator 接口
    ```

4. 除了 `for...of` 循环以外，`扩展运算符（...）`、`结构赋值` 和 `Array.from` 方法内部调用的，都是遍历器接口，因此它们都可以将 Generator 函数返回的 iterator 对象作为参数，获取该遍历器对象中的 value 属性值。

    ```js
    function* numbers() {
      yield 1;
      yield 2;
      return 3;
      yield 4;
    }

    // 扩展运算符
    [...numbers()];  // [1, 2]

    // Array.from 方法
    Array.from(numbers());  // [1, 2]

    // 解构赋值
    let [x, y] = numbers();  // 1  2

    // for...of 循环
    for(let k of numbers()) {
      console.log(k); // 1  2
    }
    ```

### Generator.prototype.throw()

1. Generator 函数返回的遍历器对象，都有一个 throw 方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。

    ```js
    var ggg = function* () {
      try {
        yield;
      } catch (e) {
        console.log('内部错误', e); // 内部捕获 a
      }
    }

    var ig = ggg();
    ig.next();

    try {
      ig.throw('a');
      ig.throw('b');
    } catch (e) {
      console.log('外部捕获', e); // 外部捕获 b
    }
    ```

    上述代码中，遍历器对象 ig 连续抛出了两个错误，第一个错误被 Generator 函数体内的 catch 语句捕获了，由于 Generator 函数内部的 catch 语句已经执行过了，所以就不会捕获第二个错误了，因此这个错误就被抛出 Generator 函数体了，也就被函数体外的 catch 语句捕获了。

2. throw 方法可以接受一个参数，该参数会被 catch 语句接收，建议抛出 Error 对象的实例。
  
    ```js
    var gener = function* () {
      try {
        yield;
      } catch (e) {
        console.log(e);
      }
    };

    var igener = gener();
    igener.next();
    igener.throw(new Error('出错了！'));
    // Error: 出错了！...
    ```

    **注意**：遍历器对象的 throw 方法和全局的 throw 命令是不一样的，上述代码是用遍历器对象的 throw 方法抛出的，而不是用 throw 命令抛出的，全局的 throw 命令只能被函数体外部的 catch 语句捕获，而不会被函数体里面的 catch 语句捕获。

    ```js
    var gene = function* () {
      while (true) {
        try {
          yield;
        } catch (e) {
          if (e != 'a') throw e;
          console.log('内部捕获', e);
        }
      }
    };

    var igene = gene();
    igene.next();

    try {
      throw new Error('a');
      throw new Error('b');
    } catch (e) {
      console.log('外部捕获', e);
    }
    // 外部捕获 [Error: a]....
    ```

    上述代码只捕获了 a，是因为函数体外的 catch 语句块，捕获了抛出的 a 错误以后，就不会再继续 try 代码块里面剩余的语句了。

    如果 Generator 函数内部与外部都没有部署 try...catch 代码块时，那么程序就会直接报错，中断执行。

3. 如果 throw 方法抛出的错误要被 Generator 函数体内部捕获，就必须至少执行过一次 next() 方法。

    ```js
    function* gen() {
      try {
        yield 1;
      } catch (e) {
        console.log('内部捕获');
      }
    }

    var g = gen();
    g.throw(1); 
    // Uncaught 1
    ```

    上述代码中，g.throw(1) 执行时，next() 方法依次都没有执行过，这时，抛出的错误不会被内部捕获，而是直接被外部捕获了，导致程序出错。其原理就是因为第一次执行 next() 方法，等同于启动执行 Generator 函数的内部代码，而在 Generator 函数还没开始执行时，就抛出错误，只能被函数体尾部的 catch 语句捕获了。

4. throw 方法被捕获以后，会附带执行下一条 yield 表达式，也就是说，会附带执行一次 next() 方法。

    ```js
    var genera = function* gen() {
      try {
        yield console.log('a');
      } catch (e) {
        // ...
      }
      yield console.log('b');
      yield console.log('c');
    }

    var ggenera = genera();
    ggenera.next(); // a
    ggenera.throw(); // b
    ggenera.next(); // c
    ```

    上述代码中，genera.throw() 方法被捕获以后，自动执行了一次 next() 方法，所以会打印出 b，此外只要 Generator 函数内部部署了 try...catch 代码块，遍历器的 throw 方法抛出错误时，不会影响下一次遍历。

5. throw 命令与 Generator 的 throw() 方法是无关的，两者互不影响。

    ```js
    var gen = function* gen() {
      yield console.log('hello');
      yield console.log('world');
    }

    var g = gen();
    g.next();

    try {
      throw new Error();
    } catch (e) {
      g.next();
    }
    // hello
    // world
    ```

    上述代码中 throw 命令抛出的错误并不会影响到遍历器的状态，所以两次执行 next() 方法时都进行了正确的操作。

    这种函数体内捕获错误的机制，大大方便了对错误的处理。多个 yield 表达式，可以只用一个 try...catch 代码块来捕获错误。但是如果使用的是回调函数的写法，想要捕获多个错误，就不得不为每个函数内部写一个错误处理语句。

6. Generator 函数体外抛出的错误，可以在函数体内捕获，而 Generator 函数体内部抛出的错误也可以被函数体外的 catch 捕获。

    ```js
    function* foo() {
      var x = yield 3;
      var y = x.toUpperCase();
      yield y;
    }

    var it = foo();

    it.next(); // { value:3, done:false }

    try {
      it.next(42);
    } catch (err) {
      console.log(err);
    }
    ```

    上述代码中，第二个 next() 方法向函数体内传入了一个参数 26，而数值是没有 toUpperCase() 方法的，所以会抛出一个 TypeError 的错误，被函数体外的 catch 语句捕获了。

    **注意**：一旦 Generator 执行过程中抛出错误，且没有被内部捕获，此时函数就不会再执行下去了。如果此时还调用 next() 方法，将返回一个 value 属性等于 undefined，done 属性等于 true 的对象，即 JS 引擎会认为这个 Generator 函数已经运行结束了。

    ```js
    function* g() {
      yield 1;
      console.log('throwing an exception');
      throw new Error('generator broke!');
      yield 2;
      yield 3;
    }

    function log(generator) {
      var v;
      console.log('starting generator');
      try {
        v = generator.next();
        console.log('第一次运行next方法', v);
      } catch (err) {
        console.log('捕捉错误', v);
      }
      try {
        v = generator.next();
        console.log('第二次运行next方法', v);
      } catch (err) {
        console.log('捕捉错误', v);
      }
      try {
        v = generator.next();
        console.log('第三次运行next方法', v);
      } catch (err) {
        console.log('捕捉错误', v);
      }
      console.log('caller done');
    }

    log(g());
    // starting generator
    // 第一次运行next方法 { value: 1, done: false }
    // throwing an exception
    // 捕捉错误 { value: 1, done: false }
    // 第三次运行next方法 { value: undefined, done: true }
    // caller done
    ```

    上述代码一共执行了三次 next() 方法，第二次运行的时候会抛出错误，然后第三次运行的时候，Generator 函数已经结束了。

### Generator.prototype.return()

1. Generator 函数返回的遍历器对象，还有一个 return() 方法，可以返回给定的值，并且终结遍历 Generator 函数。

    ```js
    function* gen9() {
      yield 1;
      yield 2;
      yield 3;
    }

    var g9 = gen9();

    console.log(g9.next()); // { value: 1, done: false }
    console.log(g9.return('foo')); // { value: "foo", done: true }
    console.log(g9.next()); // { value: undefined, done: true }
    ```

2. 如果 return() 方法调用时，不提供参数，则返回值的 value 属性为 undefined。

    ```js
    function* gen() {
      yield 1;
      yield 2;
      yield 3;
    }

    var g = gen();

    g.next() // { value: 1, done: false }
    g.return() // { value: undefined, done: true }
    ```

3. 如果 Generator 函数内部有 try...finally 代码块，且正在执行 try 代码块，那么 return() 方法会导致立刻进入 finally 代码块，执行完以后，整个函数才会结束。

    ```js
    function* mnumbers() {
      yield 1;
      try {
        yield 2;
        yield 3;
      } finally {
        yield 4;
        yield 5;
      }
      yield 6;
    }
    var mg = mnumbers();
    console.log(mg.next()); // { value: 1, done: false }
    console.log(mg.next()); // { value: 2, done: false }
    console.log(mg.return(7)); // { value: 4, done: false }
    console.log(mg.next()); // { value: 5, done: false }
    console.log(mg.next()); // { value: 7, done: true }
    ```

    上述代码中，调用 return() 方法后，就开始执行 finally 代码块了，不执行 try 里面剩下的代码了，然后等到 finally 代码块执行完后，在返回 return() 方法指定的返回值。

### next()、throw()、return() 的共同点

1. 这三者的作用都是让 Generator 函数恢复执行，并且使用不同的语句替换 yield 表达式。
  
2. next() 是将 yield 表达式替换成一个值。
  
    ```js
    const g = function* (x, y) {
      let result = yield x + y;
      return result;
    };

    const gen = g(1, 2);
    gen.next(); // Object {value: 3, done: false}

    gen.next(1); // Object {value: 1, done: true}
    // 相当于将 let result = yield x + y
    // 替换成 let result = 1;
    ```

    上面代码中，第二个 next(1) 方法就相当于将 yield 表达式替换成一个值 1。如果 next() 方法没有参数，就相当于替换成 undefined。

3. throw() 是将 yield 表达式替换成一个 throw 语句。

    ```js
    const g = function* (x, y) {
      let result = yield x + y;
      return result;
    };

    const gen = g(1, 2);
    gen.throw(new Error('出错了')); // Uncaught Error: 出错了
    // 相当于将 let result = yield x + y
    // 替换成 let result = throw(new Error('出错了'));
    ```

4. return() 是将 yield 表达式替换成一个 return 语句。

    ```js
    const g = function* (x, y) {
      let result = yield x + y;
      return result;
    };

    const gen = g(1, 2);
    gen.return(2); // Object {value: 2, done: true}
    // 相当于将 let result = yield x + y
    // 替换成 let result = return 2;
    ```

### yield* 表达式

1. 如果在 Generator 函数内部，调用另一个 Generator 函数，就需要在前者的函数体内部，自己手动完成遍历。

    ```js
    function* foo() {
      yield 'a';
      yield 'b';
    }

    function* bar() {
      yield 'x';
      // 手动遍历 foo()
      for (let i of foo()) {
        console.log(i);
      }
      yield 'y';
    }

    for (let v of bar()) {
      console.log(v);
    }
    // x
    // a
    // b
    // y
    ```

    上述代码中，foo 和 bar 都是 Generator 函数，在 bar 里面调用 foo，就需要手动遍历 foo。如果有多个 Generator 函数嵌套，写起来就非常麻烦。因此，ES6 提供了 yield* 表达式作为解决办法，用来在一个 Generator 函数里面执行另一个 Generator 函数。如下示例：

    ```js
    function* foo() {
      yield 'a';
      yield 'b';
    }

    function* bar() {
      yield 'x';
      yield* foo();
      yield 'y';
    }

    // 等同于
    function* bar() {
      yield 'x';
      yield 'a';
      yield 'b';
      yield 'y';
    }

    // 等同于
    function* bar() {
      yield 'x';
      for (let v of foo()) {
        yield v;
      }
      yield 'y';
    }

    for (let v of bar()) {
      console.log(v);
    }
    // "x"
    // "a"
    // "b"
    // "y"
    ```

    以下是一个对比的示例：

    ```js
    function* inner() {
      yield 'haha';
    }

    function* outer1() {
      yield 'open'
      yield inner();
      yield 'close';
    }

    var gen = outer1();
    gen.next().value; // 'open'
    gen.next().value; // 返回一个遍历器对象
    gen.next().value; // 'open'

    function* outer2() {
      yield 'open';
      yield* inner();
      yield 'close';
    }

    var gen = outer2();
    gen.next().value; // 'open' 
    gen.next().value; // 'haha' 
    gen.next().value; // 'close' 
    ```

    上述代码中，outer2 使用了 yield* 表达式，而 outer1 没使用，结果就是 outer1 返回一个遍历器对象，outer2 返回该遍历器对象的内部值。

2. 从语法角度看，如果 yield 表达式后面跟的是一个遍历器对象，需要在 yield 表达式后面加上星号，表明它返回的是一个遍历器对象。而这也被称为 yield* 表达式。

    ```js
    let delegatedIterator = (function* () {
      yield 'hello';
      yield 'Bye';
    }());

    let delegatingIterator = (function* () {
      yield 'Greetings';
      yield* delegatedIterator;
      yield 'OK, bye';
    }());

    for (let value of delegatingIterator) {
      console.log(value);
    }
    // Greetings
    // hello
    // Bye
    // OK, bye
    ```

    上述代码中，delegatingIterator 是一个代理者，而 delegatedIterator 是被代理者，由于 yield* delegatedIterator 语句得到的值是一个遍历器，所以要用星号表示。运行结果就是使用一个遍历器，遍历了多个 Generator 函数，有递归的效果。

3. yield* 后面的 Generator 函数中没有 return 语句时，等同于在 Generator 函数内部，部署一个 for...of 循环。

    ```js
    function* concat(iter1, iter2) {
      yield* iter1;
      yield* iter2;
    }

    // 等同于

    function* concat(iter1, iter2) {
      for (var value of iter1) {
        yield value;
      }
      for (var value of iter2) {
        yield value;
      }
    }
    ```

    上面代码说明，yield* 后面的 Generator 函数没有 return 语句时，完全可以使用 for...of 代替 yield*，而 yield* 不过是一种简写形式。但是如果在有 return 语句的情况下，则需要使用 var value = yield* iterator 的形式获取 return 语句的值。

    ```js
    let hasReturn = function* () {
      yield '111';
      yield '222';
      return '333';
    }

    let getRes = function* () {
      yield 'aaa';
      // 如果不使用value接收就无法获取到return '333'的值
      let value = yield* hasReturn();
      yield value;
      yield 'bbb';
    }

    let res = getRes();
    for (let v of res) {
      console.log(v);
      // aaa
      // 111
      // 222
      // 333
      // bbb
    }
    ```

4. 如果 yield* 后面跟着一个数组，由于数组原生就支持遍历器，因此就会遍历数组成员。

    ```js
    function* gen() {
      yield* ['a', 'b', 'c'];
    }

    let ge = gen();

    for(let v of ge) {
      console.log(v); // a b c
    }
    ```

    上述代码中，yield 命令后面如果不加星号，返回的是整个数组，加了星号就表示返回的是数组的遍历器对象。

5. 任何数据结构只要有 iterator 接口，就可以被 yield* 遍历。

    ```js
    let read = (function* () {
      yield 'hello';
      yield* 'hello';
    })();

    console.log(read.next().value); // hello
    console.log(read.next().value); // h
    ```

    上述代码中，yield 表达式返回整个字符串，而 yield* 返回的是单个字符串，因为字符串具有 iterator 接口，所以可以被 yield* 遍历。

6. 如果被代理的 Generator 函数有 return 语句，那么就可以向代理它的 Generator 函数返回数据。

    ```js
    function* foo() {
      yield 2;
      yield 3;
      return 'foo';
    }

    function* bar() {
      yield 1;
      var v = yield* foo();
      console.log('v: ' + v);
      yield 4;
    }

    var it = bar();
    console.log(it.next()); // {value: 1, done: false}
    console.log(it.next()); // {value: 2, done: false}
    console.log(it.next()); // {value: 3, done: false}
    console.log(it.next()); // v: foo  {value: 4, done: false}
    console.log(it.next()); // {value: undefined, done: true}
    ```

    上述代码中，在第四次调用 next() 方法的时候，      console.log('v: ' + v) 会有输出是因为函数 foo 的 return 语句向函数 bar 提供了返回值。

    下面是另一个实例：

    ```js
    function* genFuncWithReturn() {
      yield 'a';
      yield 'b';
      return 'The result';
    }

    function* logReturned(genObj) {
      let result = yield* genObj;
      console.log(result); // The result
    }

    console.log([...logReturned(genFuncWithReturn())]); // ["a", "b"]
    ```

    上述代码中，存在两次遍历，第一次是扩展运算符遍历函数 logReturned 返回的遍历器对象，第二次是 yield* 语句遍历函数 genFuncWithReturn 返回的遍历器对象。这两次遍历的效果是叠加的，最终表现为扩展运算符遍历函数 genFuncWithReturn 返回的遍历器对象，所以最后的数据表达式得到的值等于 ['a', 'b']，但是函数 genFuncWithReturn 的 return 语句的返回值 The result 会返回给函数 logReturned 内部的 result 变量，因此会有终端输出。

7. yield* 命令可以很方便的取出嵌套数组的所有成员。

    ```js
    function* iterTree(tree) {
      if (Array.isArray(tree)) {
        for (let i = 0; i < tree.length; i++) {
          yield* iterTree(tree[i]);
        }
      } else {
        yield tree;
      }
    }

    const tree = ['a', ['b', 'c'],
      ['d', 'e']
    ];

    for (let x of iterTree(tree)) {
      console.log(x);
    }
    // a
    // b
    // 1
    // 3
    // 4
    // 5
    // 2
    // c
    // d
    // e

    console.log(...iterTree(tree)); // ["a", "b", 1, 3, 4, 5, 2, "c", "d", "e"]
    // 类似于flat()方法
    const r = tree.flat(1);
    console.log(r); // ["a", "b", 1, 3, 4, 5, 2, "c", "d", "e"]
    ``` 

8. 使用 yield* 语句遍历完全二叉树。

    ```js
    // 下面是二叉树的构造函数，
    // 三个参数分别是左树、当前节点和右树
    function Tree(left, label, right) {
      this.left = left;
      this.label = label;
      this.right = right;
    }

    // 下面是中序（inorder）遍历函数。
    // 由于返回的是一个遍历器，所以要用generator函数。
    // 函数体内采用递归算法，所以左树和右树要用yield*遍历
    function* inorder(t) {
      if (t) {
        yield* inorder(t.left);
        yield t.label;
        yield* inorder(t.right);
      }
    }

    // 下面生成二叉树
    function make(array) {
      // 判断是否为叶节点
      if (array.length == 1) return new Tree(null, array[0], null);
      return new Tree(make(array[0]), array[1], make(array[2])); 
    }
    let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);

    // 遍历二叉树
    var result = [];
    for (let node of inorder(tree)) {
      result.push(node);
    }

    console.log(result); // ['a', 'b', 'c', 'd', 'e', 'f', 'g']
    ```

### 作为对象属性的 Generator 函数

1. 如果一个对象的属性是 Generator 函数，可以简写成下面的形式。

    ```js
    let obj = {
      * myGeneratorMethod() {
        yield ...
      }
    }
    // 等价于
    let obj = {
      myGeneratorMethod: function* () {
        yield ...
      }
    }
    ```

    上述代码中，myGeneratorMethod 属性前面有一个星号，则表示这个属性是一个 Generator 函数。

### Generator 函数的 this

1. Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的 prototype 对象上的方法。

    ```js
    function* gf() {}
    gf.prototype.hello = function () {
      return 'hi!';
    };
    let objf = gf();
    console.log(objf instanceof gf); // true
    console.log(objf.hello()); // 'hi!'
    ```

    上述代码说明 Generator 函数 gf 返回的遍历器对象 objf 是 gf 的实例，而且继承了 gf 函数原型上的 hello 方法。但是如果把 gf 当作普通的构造函数时，就不会产生以上效果，因为 gf 返回的总是遍历器对象，而不是 this 对象。

    ```js
    function* g() {
      this.a = 22;
    }

    let obj = g();
    obj.next();
    console.log(obj.a); // undefined
    ```

    上述代码中，Generator 函数 g 在 this 对象上添加了一个属性 a，但是 obj 对象拿不到这个属性。

2. Generator 函数也不能跟 new 命令一起使用，否则会报错。

    ```js
    function* F() {
      yield this.x = 1;
      yield this.y = 2;
    }

    new F();  // TypeError: F is not a constructor
    ```

    上述代码中，new 命令跟 Generator 函数一起使用，结果就报错了，因为 Generator 函数并不是构造函数。

3. 要使 Generator 函数返回一个正常的对象实例，既可以用 next() 方法，又可以获得正常的 this，就需要先生成一个空对象，再使用 call 方法绑定 Generator 函数内部的 this，这样构造函数调用以后，这个空对象就是 Generator 函数的实例对象了。

    ```js
    function* F() {
      this.a = 1;
      yield this.b = 2;
      yield this.c = 3;
    }
    var obj = {};
    var f = F.call(obj);

    console.log(f.next()); // Object {value: 2, done: false}
    console.log(f.next()); // Object {value: 3, done: false}
    console.log(f.next()); // Object {value: undefined, done: true}

    console.log(obj.a); // 1
    console.log(obj.b); // 2
    console.log(obj.c); // 3
    ```

    上述代码中，首先使 F 内部的 this 对象绑定为 obj 对象，然后调用 F，返回一个 iterator 对象，这个对象执行了三次 next() 方法（因为 F 内部有两个 yield 表达式），完成了 F 内部所有代码的运行，此时所有内部属性都绑定在 obj 对象上了，因此 obj 对象也就成了 F 的实例。

    而上述代码中，执行的是遍历器对象 f，但是生成的对象实例是 obj，要使这两个对象统一，就需要将 obj 换成 F.prototype，再创建一个构造函数，将调用 Generator 函数返回的 iterator 对象在这个创建的构造函数中使用 return 返回出去即可使用 new 命令了。

    ```js
    function* F() {
      this.a = 1;
      yield this.b = 2;
      yield this.c = 3;
    }
    var f = F.call(F.prototype);

    console.log(f.next()); // Object {value: 2, done: false}
    console.log(f.next()); // Object {value: 3, done: false}
    console.log(f.next()); // Object {value: undefined, done: true}

    console.log(f.a); // 1
    console.log(f.b); // 2
    console.log(f.c); // 3

    // 最终写法：将F改成构造函数
    function* geng() {
      this.a = 1;
      yield this.b = 2;
      yield this.c = 3;
    }

    function F() {
      return geng.call(geng.prototype);
    }

    var f = new F();

    console.log(f.next()); // Object {value: 2, done: false}
    console.log(f.next()); // Object {value: 3, done: false}
    console.log(f.next()); // Object {value: undefined, done: true}

    console.log(f.a); // 1
    console.log(f.b); // 2
    console.log(f.c); // 3
    ```

### Generator 与状态机

1. Generator 是实现状态机的最佳结构，比如，下面的 click 函数就是一个状态机。

    ```js
    var ticking = true;
    var clock = function () {
      if (ticking) {
        console.log('Tick!');
      } else {
        console.log('Tock!')
      };
      ticking = !ticking;
    }
    clock(); // Tick
    clock(); // Tock
    clock(); // Tick
    ```

    上述代码的 clock 函数一共有两种状态（Tick 和 Tock)，每运行一次，就改变一次状态。这个函数如果用 Generator 实现，可以像如下操作：

    ```js
    val clock = function* () {
      while (true) {
        console.log('Tick');
        yield;
        console.log('Tock');
        yield;
      }
    }
    ```

    上述代码中，Generator 实现与 ES5 实现对比，可以看到少了用来保存状态的外部变量 ticking，这样就更简洁，更安全（状态不会被非法篡改），也更符合函数式编程的思想，在写法上也更优雅。Generator 之所以可以不用外部变量保存状态，是因为它本身就包含了一个状态信息，即目前是否处于暂停状态。

### Generator 与协程

1. 协程（coroutine）是一种程序运行的方式，可以理解成 `协作的线程` 或 `协作的函数`。协程既可以用单线程实现，也可以用多线程实现。前者是一种特殊的子例程，后者是一种特殊的线程。

2. 协程与子例程的差异：

    > 传统的`子例程（subroutine）`采用堆栈式`后进先出`的执行方式，只有当调用的子函数完全执行完毕，才会结束执行父函数。协程与其不同，多个线程（单线程情况下，即多个函数）可以并行执行，但是只有一个线程（或函数）处于正在运行的状态，其他线程（或函数）都处于暂停状态（suspended），线程（或函数）之间可以交换执行权。也就是说，一个线程（或函数）执行到一半，可以暂停执行，将执行权交给另一个线程（或函数），等到稍后收回执行权的时候，在恢复执行。这种`可以并行执行，交换执行权的线程（或函数），就称为协程`。

    > 从实现上看，在内存中，子例程只使用一个栈，而协程是同时存在多个栈，但只有一个栈是运行状态，也就是说，协程是以多占用内存为代价，实现多任务的并行。

3. 协程与普通线程的差异：

    > 协程适合用于多任务运行的环境。在这个意义上，它与普通的线程很相似，都有自己的执行上下文，可以分享全局变量。它们的不同之处在于，同一时间可以有多个线程处于运行状态，但是运行的协程只能有一个，其它协程都处于暂停状态。此外，普通的线程是抢先式的，到底哪个线程优先得到资源，必须由运行环境决定，但是协程是合作式的，执行权由协程自己分配。

    > 由于 JS 是单线程语言，只能保持一个调用栈，引入协程之后，每个任务可以保持自己的调用栈，这样做的最大好处就是抛出错误的时候，可以找到原始的调用栈，不至于像异步操作的回调函数那样，一旦出错，原始的调用栈早就结束。

    > Generator 函数是 ES6 对协程的实现，但是属于不完全实现，Generator 函数被称为`半协程（semi-couroutine）`，意思是只有 Generator 函数的调用者，才能将程序的执行权交还给 Generator 函数，如果是完全执行的协程，任何函数都可以让暂停的协程继续执行。

    > 如果将 Generator 函数当作协程，完全可以将多个需要相互协作的任务写成 Generator 函数，它们之间使用 yield 表达式交换控制权。

### Generator 与上下文

1. JS 代码运行时，会产生一个全局的上下文环境（context，又称为运行环境），包含了当前所有的变量和对象。然后执行函数（或块级代码）的时候，又会在当前上下文环境的上层，产生一个函数运行的上下文，变成当前（active）的上下文，由此形成一个上下文环境的堆栈（context stack）。

2. 堆栈是`后进先出`的结构，最后产生的上下文环境首先执行完成，退出堆栈，然后再执行完成它下层的上下文，直至所有代码执行完成，堆栈清空。

3. Generator 函数则不像上述那样，它执行产生的上下文环境，一旦遇到 yield 命令，就会暂时退出堆栈，但是并不消失，里面的所有变量和对象会被冻结在当前状态，等到对它执行 next() 命令时，这个上下文环境又会重进加入调用栈，冻结的变量和对象恢复执行。

    ```js
    function* genrr() {
      yield 1;
      return 2;
    }

    let grr = genrr();

    console.log(
      grr.next(), grr.next(), // {value: 1, done: false} {value: 2, done: true}
    )
    ```

    上述代码中，第一次执行 grr.next() 时，Generator 函数 genrr 的上下文会加入堆栈，即开始运行 genrr 内部的代码，等遇到 yield 1 时，genrr 上下文退出堆栈，内部状态冻结，第二次执行 g.next() 时，genrr 上下文重新加入堆栈，变成当前的上下文，重新恢复执行。

### Generator 的应用

1. 异步操作的同步化表达。

    > Generator 函数的暂停执行的效果，意味着可以把异步操作写在 yield 表达式里面，等到调用 next() 方法时再往后执行。这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在 yield 表达式下面，反正要等到调用 next() 方法时在执行。所以 Generator 函数的一个重要实际意义就是用来处理异步操作，改写回调函数。

    > ```js
    > function* loadUI() {
    >   showloadingScreen();
    >   yield loadUIDataAsynchronously();
    >   hideLoadingScreen();
    > }
    > var loader = loadUI();
    > // 加载UI
    > loader.next();
    > // 卸载UI
    > loader.next();
    > ```

    > 上述代码中，第一次调用 loadUI 函数时，该函数不会立即执行，仅返回一个遍历器，下一次对该遍历器调用 next() 方法，则会显示 loading 界面（showloadingScreen），并且异步加载数据（loadUIDataAsynchronously），等到数据加载完成，再一次使用 next() 方法，则会影藏 loading 界面。这种写法的好处是所有 loading 界面的逻辑，都封装在一个函数中，显得非常清晰。

    > ajax 是典型的异步操作，通过 Generator 函数部署 Ajax 操作，可以用同步的方式表达。

    > ```js
    > function* main() {
    >   var result = yield request("http://some.url");
    >   var resp = JSON.parse(result);
    >   console.log(resp.value);
    > }
    > 
    > function request(url) {
    >   makeAjaxCall(url, function(response) {
    >     it.next(response);
    >   })
    > }
    >
    > var it = main();
    > it.next();
    > ```

    > 上述代码的 main 函数，就是通过 Ajax 操作获取数据，需要注意的是：makeAjaxCall 函数中的 next() 方法，必须加上 response 参数，因为 yield 表达式，本身是没有值的，总是等于undefined。

    > 下面是通过 Generator 函数逐行读取文本文件。
  
    > ```js
    > function* numbers() {
    >   let file = new FileReader('number.txt');
    >   try {
    >     while(!file.eof) {
    >       yield parseInt(file.readLine(), 10);
    >     }
    >   } finally {
    >     file.close();
    >   }
    > }
    > ``` 

    > 上述代码打开文本文件，使用 yield 表达式可以手动逐行读取文件。

2. 控制流管理

    > 如果有一个多步操作非常耗时，采用回调函数可能会写成下面这样，形成回调地狱的噩梦。

    > ```js
    > step1(function (value1) {
    >   step2(value1, function(value2) {
    >     step3(value2, function(value3) {
    >       step4(value3, function(value4) {
    >         // do something with value4
    >       })
    >     })
    >   })
    > })
    > 
    > // 采用 Promise 改写上面的代码
    > Promise.resolve(step1)
    >   .then(step2)
    >   .then(step3)
    >   .then(step4)
    >   .then(function (value4) {
    >     // do something with value4 
    >   }, function (error) {
    >     // handle any error from step1 through step4
    >   }).done();
    > ```

    > 上面代码以及把回调函数，改写成直线执行的形式了，但是加入了大量 Promise 语法，Generator 函数可以进一步改善代码运行流程。

    > ```js
    > function* longRunningTask(value1) {
    >   try {
    >     var value2 = yield step1(value1);
    >     var value3 = yield step1(value2);
    >     var value4 = yield step1(value3);
    >     var value5 = yield step1(value4);
    >     // do something with value4
    >   } catch (e) {
    >     // handle any error from step1 through step4
    >   }
    > }
    > 
    > // 使用一个函数，按次序自动执行所有步骤
    > scheduler(longRunningTask(initialValue));
    > 
    > function scheduler(task) {
    >   var taskObj = task.next(task.value);
    >   // 如果Generator函数未结束，就继续调用
    >   if(!taskObj.done) {
    >     task.value = taskObj.value;
    >     scheduler(task);
    >   }
    > }
    > ```

    > **注意**：上面这种做法只适合同步操作，即所有的 task 都必须是同步的，不能有异步操作，因为这里的代码一得到返回值，就继续往下执行，没有判断异步操作何时完成。

    > 利用 for...of 循环自动依次执行 yield 命令的特性，提供一种更一般的控制流管理的方法。

    > ```js
    > let steps = [step1Func, step2Func, step3Func];
    > 
    > function* iterateSteps(steps) {
    >   for (var i = 0; i < steps.length; i++) {
    >     var step = step[i];
    >     yield step();
    >   }
    > }
    > ```

    > 上面代码中，数据 steps 封装了一个任务的多个步骤，Generator 函数 iterateSteps 则依次为这些步骤加上 yield 命令。

    > 将任务分解成步骤之后，还可以将项目分解成多个依次执行的任务。

    > ```js
    > let jobs = [job1, job2, job3];
    > 
    > function* iterateJobs(jobs) {
    >   for (var i = 0; i < job.length; i++) {
    >     var job = job[i];
    >     // 依次为任务加上yield*命令
    >     yield* iterateSteps(job.steps);
    >   }
    > }
    > 
    > // 使用for...of循环一次性依次执行所有任务的所有步数
    > for (var step of iterateJobs(jobs)) {
    >   console.log(step.id);
    > }
    > ```

    > **再次说明**：上面的做法只能用于所有步骤都是同步操作的情况，不能有异步操作的步骤。

    > for...of 的本质是一个 while 循环，所以上面的代码实质上执行的是下面的逻辑

    > ```js
    > var it = iterateJobs(jobs);
    > var res = it.next();
    > 
    > while (!res.done) {
    >   var result = res.value;
    >   // ...
    >   res = it.next();
    > }
    > ```

3. 部署 iterator 接口

    > 利用 Generator 函数，可以在任意对象上部署 iterator 接口。

    ```js
    function* iterEntries(obj) {
      let keys = Object.keys(obj);
      for (let i = 0; i < keys.length; i++) {
        let key = keys[i];
        yield [key, obj[key]];
      }
    }

    let myObj = {foo: 3, bar: 7};

    for(let [key, value] of iterEntries(myObj)) {
      console.log(key, value);
    }

    // foo 3
    // bar 7
    ```

    > 上述代码中，myObj 是一个普通对象，通过 iterEntries 函数，就有了 iterator 接口，也就是说，可以在任意对象上部署 next() 方法。

    > 对数组部署 iterator 接口，尽管数组原生具有这个接口。

    > ```js
    > function* makeSimpleGenerator(array) {
    >   var nextIndex = 0;
    > 
    >   while(nextIndex < array.length) {
    >     yield array[nextIndex++];
    >   }
    > }
    > 
    > var gen = makeSimpleGenerator (['yo', 'ya']);
    > 
    > gen.next().value; // 'yo'
    > gen.next().value; // 'ya'
    > gen.next().done; // true
    > ```

4. 作为数据结构

    > Generator 可以看作是数据结构，跟确切的说，可以看作是一个数组结构，因为 Generator 函数可以返回一系列的值，这意味着它可以对任意表达式提供类似数组的接口。

    > ```js
    > function* doStuff() {
    >   yield fs.readFile.bind(null, 'hello.txt');
    >   yield fs.readFile.bind(null, 'world.txt');
    >   yield fs.readFile.bind(null, 'and-such.txt');
    > }
    > ```

    > 上面代码就是依次返回三个函数，但是由于使用了 Generator 函数，导致可以像处理数组那样，处理这三个返回的函数。

    > ```js
    > for (task of doStuff()) {
    >   // task是一个函数，可以像回调函数那样使用它
    > }
    > ```

    > 实际上，如果使用 ES5 表达，完全可以用数组模拟 Generator 的这种用法。 
    
    > ```js
    > function doStuff() {
    >   return [
    >     yield fs.readFile.bind(null, 'hello.txt');
    >     yield fs.readFile.bind(null, 'world.txt');
    >     yield fs.readFile.bind(null, 'and-such.txt');
    >   ];
    > }
    > ```

    > 上面的函数，可以用一摸一样的 for...of 循环处理，两相一比较，就不难看出 Generator 使得数据或者操作具备了类似数组的接口。
