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