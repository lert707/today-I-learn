## 异步 setTimeout Promist async/await
```javascript
  async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
  }
  async function async2() {
    console.log('async2');
  }
  console.log('script start');
  setTimeout(function() {
    console.log('setTimeout');
  }, 0)
  async1();
  new Promise(function(resolve) {
    console.log('promise1');
    resolve();
  }).then(function() {
    console.log('promise2');
  });
  console.log('script end');

  /*
    输出结果: 
    script start
    async1 start
    async2
    promise1
    script end
    async1 end  // 在node下 先输出的是promise2 再是async1 end
    promise2    // 在chrome下 先输出的是async1 end 再是promise2
    setTimeout
  */

  async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
  }
  // 等价于
  async function async1() {
    console.log('async1 start');
    Promise.resolve(async2()).then(() => {
      console.log('async1 end');
    })
  }

  /*
    Microtasks和Macrotasks是异步任务的一种类型，Microtasks的优先级要高于Macrotasks，下面是它们所包含的api：
    microtasks
      process.nextTick
      promise
      Object.observe (废弃)
      MutationObserver
    macrotasks
      setTimeout
      setImmerdiate
      setInterval
      I/O
      UI 渲染

    JS主线程不断的循环往复的从任务队列中读取任务，执行任务，这中运行机制称为事件循环（event loop）。
    1. 每一个 event loop 都有一个 microtask queue
    2. 每个 event loop 会有一个或多个macrotaks queue ( 也可以称为task queue )
    3. 一个任务 task 可以放入 macrotask queue 也可以放入 microtask queue中
    4. 每一次event loop，会首先执行 microtask queue， 执行完成后，会提取 macrotask queue 的一个任务加入 microtask queue， 接着继续执行microtask queue，依次执行下去直至所有任务执行结束。
  */
```

## es6交换变量
```javascript
  let a = 1, b = 2;

  [a, b] = [b, a] // 利用到了es6的解构赋值

  console.log(a, b) // 2, 1
```

## 深拷贝 浅拷贝

  1. JSON.parse(JSON.stringify())可以用来深拷贝,但是有几个注意点

	  会忽略 undefined
	  会忽略 symbol
	  不能序列化函数,这种方式不能够拷贝函数
	  不能解决循环引用的对象

	  	let obj = {
			a: 1,
			b: {
			  c: 2,
			  d: 3,
			},
	  	}
		  obj.c = obj.b
		  obj.e = obj.a
		  obj.b.c = obj.c
		  obj.b.d = obj.b
		  obj.b.e = obj.b.c
		  let newObj = JSON.parse(JSON.stringify(obj))
		  console.log(newObj)

	  如果你有这么一个循环引用对象，你会发现你不能通过该方法深拷贝

  2. Object.assign()只能用来浅拷贝。如果对象中的属性存在复杂数据类型，改变了拷贝对象的值，源对象的值也会发生改变。

  3. 可以使用递归来进行深拷贝
  ```javascript
    function deepCopy(obj) {
      var result = Array.isArray(obj) ? [] : {};
      for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
          if (typeof obj[key] === 'object' && obj[key]!==null) {
            result[key] = deepCopy(obj[key]);   // 递归复制
          } else {
            result[key] = obj[key];
          }
        }
      }
      return result;
    }
  ```

## for in

  - for(let key in arr) 我一直以为这种只能用来遍历对象。原来数组也是可以的
  	但是 for in遍历数组效率极差。所以还是老老实实用for。
  
## js实现sleep函数

```javascript
  function sleep(interval) {
    return new Promise(resolve => {
      setTimeout(resolve, interval);
    })
  }
  async function test() {
    for (let i = 0; i < 10; i++) {
      console.log(i);
      await sleep(1000)
    }
  }
```

## react中让setState支持 async 调用

```javascript
  // 定义
  setStateAsync(state) {
    return new Promise((resolve) => {
      this.setState(state, resolve);
    });
  }
  // 使用
  async fn() {
    await setStateAsync({a: 1})
  }
```

## 利用a标签解析URL

```javascript
  function parseURL(url) {
    var a =  document.createElement('a');
    a.href = url;
    return {
        host: a.hostname,
        port: a.port,
        query: a.search,
        params: (function(){
            var ret = {},
                seg = a.search.replace(/^\?/,'').split('&'),
                len = seg.length, i = 0, s;
            for (;i<len;i++) {
                if (!seg[i]) { continue; }
                s = seg[i].split('=');
                ret[s[0]] = s[1];
            }
            return ret;
        })(),
        hash: a.hash.replace('#','')
    };
  }
```