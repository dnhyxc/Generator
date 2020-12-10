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

8. 上面代码将产生语法错误，因为 forEach() 方法中的参数是一个普通函数，所以在其中使用 yield 表达式就会产生语法错误。可以将代码改成如下形式解决错误：
  
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

9. yield 表达式用于另一个表达式中时，必须放在圆括号里面。

    ```js
    function* generator() {
      console.log('Hi' + yield);  // SyntaxError
      console.log('Hi' + yield 123);  // SyntaxError

      console.log('Hi' + (yield));  // Ok
      console.log('Hi' + (yield 123));  // Ok
    }
    ```

10. yield 表达式用作函数参数或放在赋值表达式的右边，可以不加括号。
  
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

3. 上述代码中，gen 是一个 Generator 函数，调用它会生成一个遍历器对象 g，而它的 Symbol.iterator 属性，也是一个遍历器对象生成函数，执行后返回它自己。

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

2. 由上述代码可以看出，当 next() 方法携带一个参数 true 时，变量 reset 就被重置为这个参数（即 true），因此 i 会等一 -1，下一轮循环就会从 -1 开始递增。由此可见 next() 方法携带参数可以在 Generator 函数开始执行之后，继续向函数体内部注入值，也就是说可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。而 Generator 函数从暂停状态到恢复运行，它的上下文状态是不变的。

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

3. 如果在第一次调用 next() 方法时传入参数，会被忽略，即第一次调用 next() 方法时传入的参数是无效的。从第二次开时才会是有效的。如果想要在第一次执行 next() 方法时就传入参数，则需要在 Generator 函数外面再包一层函数，代码如下：

    ```js
    function wrapper(generatorFun) {
      retutn function (...args) {
        let generatorObject = generatorFun(...args);
        generatorObject.next();
        return generatorObject;
      }
    }
  
    const wrapped = wrapper(function* (){
      console.log(`First input: ${yield}`);
      return 'DONE';
    });
  
    console.log(wrapper().next('dnhyxc'));
    ```  