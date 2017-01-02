# 前端异步编程

Javascript语言的执行环境是单线程，即每次只能完成一个任务。如果有多个任务，那么必然存在一个任务队列，前一个任务完成，后一个任务才能开始。  

## 同步和异步
单线程实现比较简单，但是只要存在某一个任务执行时间过长，那么后面的任务都会等待该任务的完成，就有可能造成‘假死’的现象。为了解决这个问题，Javascript将任务的执行模式分为同步（Synchronous）和异步（Asynchronous）两种。
所谓的同步模式就是后一个任务等待前一个任务执行完成后再执行。任务的执行顺序和任务的排列顺序是一致的。异步模式是指任务有一个及以上个的回调函数（callback）,每个任务执行完毕立即执行回调函数，每个任务后一个任务会立即执行，而不会等待该任务执行完毕后再执行。  
通俗来讲，同步模式就相当于排队买肯德基，买和取都在一条队里完成，后面的人需要等到前面的人取到餐了才能点餐，一旦前面人慢，后面的人都需要等待。异步模式就相当于排队买麦当劳，点餐排成一条队，取餐在旁边的取餐口，那么后面人就无需等待前面人取好餐就能直接点餐，而前面人点餐后当快餐准备完毕会被直接叫号，便能完成取餐动作。  

## 回调函数
异步编程最基本的方法便是回调函数。  
现在假设一个场景，需要从a文件里读取数据，然后对a文件的数据做处理handle1、handle2，然后将处理的结果写入b文件。用同步模式写法如下：
```javascript
let res;
res = readFile('a');
res = handle1(res);
res = handle2(res);
writeFile('b',res);
```  
通过上述代码我们可以看到如果文件比较大的话那么每一步操作都会阻塞程序执行很久，就有可能造成假死的现象。那么用回调函数改写如下：  
```javascript
readFile('a',function(res){
	res = handle1(res);
	res = handle2(res);
	writeFile('b',res);
})
```
通过上述代码我们可以看到当文件比较大的时候，即使读文件a花了很长时间，也不会影响后续代码的执行，只有在a读取完毕后才会去调用回调函数去做相应的处理。

## 回调函数缺点
回调函数能够使单线程的Javascript代码在不阻塞UI的前提下去执行网络等耗时操作，但是这种异步风格也带来了一些问题：

### 回调地狱
还是上述那个场景，若是handle1, handle2都是耗时操作呢？那么代码如下： 
```javascript
readFile('a',function(res){
	handle1(res,function(res){
		handle2(res,function(res){
			......
		})
	})
})
```  
在这种情形下代码便成为了倒金字塔风格，俗称回调地狱。这样不仅使代码变得更难看懂，而且也更难调试。

### 异常处理困难
这里异常处理包括了两方面：异步函数错误及回调函数错误。  
异步事务是无法通过try/catch进行错误捕获的，如果异步过程出现错误，一般是通过将错误通过参数的形式传递给回调函数，例如NodeJs中规定将错误作为第一个入参传递给回调函数。  
```javascript
function(err,...args){...}
```
回调函数错误是回调函数在执行的时候函数内部出现错误，但是回调函数执行的时候上下文已经不在，所以无法通过try/catch去捕获，一般得在函数体内单独写try/catch来进行单独捕获错误。  
上面的例子如果包含了异常处理，代码如下：  
```javascript
readFile('a',function(err,res){
	if(err) {
		//异步错误处理
		...
	}
	try {
		//处理逻辑
		
	} catch(e) {
		//回调函数异常
		...
	}
})
```
从上述代码可以看到仅一个回调函数便使得代码变得更难懂，并且错误处理逻辑和正常处理逻辑混合在一起，大大增加的代码的耦合性。

## Promise
[Promises/A+](https://promisesaplus.com/)对Promise的描述如下：
> A promise represents the eventual result of an asynchronous operation.   

简而言之，Promise就是个有限状态机，一个Promise具有pending、fulfilled、rejected三种状态。初始状态为pending，结束状态根据成功与否分为fulfilled和rejected。不同的结束状态触发不同的事件。

### 构造函数
一个Promise对象通过调用Promise的构造函数生成，入参接受一个factory，factory入参包括resolve和reject，其中resolve将Promise对象的状态由pending转为fulfilled, reject将Promise对象的状态由pending转为rejected。
```javascript
var myPromise = new Promise(function(resolve,reject){
	if(success) {
		resolve(result)
	} else {
		reject(err)
	}
});
```

### Then方法
[Promises/A+](https://promisesaplus.com/)规定一个 promise 必须提供一个 then 方法以访问其当前值、终值和据因。  
```javascript
promise.then(onFulfilled, onRejected)
```
 then 方法必须返回一个Promise对象，这样就可以写成链式调用。一个promise对象还提供了一个catch方法用来捕捉错误，这个catch其实是then方法的语法糖。
```javascript
promise.then(undefined, onRejected) //catch
```

### 类方法

#### Promise.resolve 
接受任何入参，返回状态为fulfilled状态的Promise对象；
#### Promise.reject
接受任何入参，返回状态为rejected状态的Promise对象；
#### Promise.all
入参为子元素均为Promise对象的数组，当所有Promise状态转化为fulfilled，该方法返回的Promise对象的状态转化为fulfilled，否则为rejected；
#### Promise.race
入参为子元素均为Promise对象的数组，当任意一个Promise对象状态发生变化，该方法返回的Promise对象的状态也发生相应的变化；

### 代码示例
利用Promise，我们可以将之前场景写成如下代码：
```javascript
const promise = new Promise(function(resolve,reject){
	readFile('a',function(err,res){
		if(err) {
			reject(err);
		}
		resolve(res)
	})
});

promise.then(function(res){
	return handle1(res);
}).then(function(res){
	return handle2(res);
}).then(function(res){
	writeFile('b',res)
}).catch(function(err){
	//err处理
})
```

由以上代码我们可以看到整个场景的逻辑都可以通过then函数进行链式调用，而且每个处理都可以进行和主逻辑进行解耦，所有的异常都可以通过catch进行处理。

### 结语
前端异步编程解决方案还包括Generator和async/await，这里不再进行阐述。在众多异步解决方案中，回调函数方案虽然有着种种缺陷，但是天然无坑。Promise方案在很多低版本的浏览器并不支持，需要引入相关的polyfill。Generator和async/await方案更是如此。读者可以根据自己掌握的程度和项目本身来选择适当的异步解决方案。