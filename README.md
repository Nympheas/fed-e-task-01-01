# fed-e-task-01-01
//一，谈谈你是如何理解JS异步编程的，EventLoop、消息队列都是做什么的，什么是宏任务，什么是微任务？
//答：JS 异步编程
//JavaScript 语言的执行环境是单线程的，一次只能执行一个任务，多任务需要排队等候，这种模式可能会阻塞代码，导致代码执行效率低下。
//为了避免这个问题，出现了异步编程。一般是通过 callback 回调函数、事件发布/订阅、Promise 等来组织代码，
//本质都是通过回调函数来实现异步代码的存放与执行
//EventLoop 事件环和消息队列
//EventLoop:是一种循环机制 ，不断去轮询一些队列 ，从中找到 需要执行的任务并按顺序执行的一个执行模型。
//消息队列:是用来存放宏任务的队列， 比如定时器时间到了， 定时间内传入的方法引用会存到该队列， 
//ajax回调之后的执行方法也会存到该队列。
//一开始整个脚本作为一个宏任务执行。执行过程中同步代码直接执行，宏任务等待时间到达或者成功后，
//将方法的回调放入宏任务队列中，微任务进入微任务队列。
//当前主线程的宏任务执行完出队，检查并清空微任务队列。接着执行浏览器 UI 线程的渲染工作，检查web worker 任务，有则执行。
//然后再取出一个宏任务执行。以此循环.
//宏任务
//宏任务可以理解为每次执行栈执行的代码就是一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行）。
//浏览器为了让 JS 内部宏任务 与 DOM 操作能够有序的执行，会在一个宏任务执行结束后，
//在下一个宏任务执行开始前，对页面进行重新渲染。
//宏任务包含：script(整体代码)、setTimeout、setInterval、I/O、UI交互事件、MessageChannel 等
//微任务
//可以理解是在当前任务执行结束后需要立即执行的任务。也就是说，在当前任务后，在渲染之前，执行清空微任务。
//所以它的响应速度相比宏任务会更快，因为无需等待 UI 渲染。
//微任务包含：Promise.then、MutaionObserver、process.nextTick(Node.js 环境)等
//代码题
//一、
let promise = New MyPromise((resolve,reject) => {
	setTimeout(function () {
		var a = 'hello'
		setTimeout(function () {
			var b = 'lagou'
			setTimeout(function () {
				var c = 'I ❤ U'
				console.log(a + b + c)
			}, 10)
		}, 10)
	}, 10)
});
//二、
//练习1
const fp = require('lodash/fp');
var f = fp.flowRight(fp.last(cars),fp.prop('in_stock',fp.last(cars)));
//练习2
const fp = require('lodash/fp');
var f = fp.flowRight(fp.first(cars),fp.prop('name',fp.first(cars)));
//练习3
let averageDollorValue = fp.flowRight(_average, fp.map(fp.curry(fp.props)('dollar_value')))
console.log(averageDollorValue(cars))
//练习四
let sanitizeNames = fp.flowRight(fp.map(_underscore), fp.map(car => car.name))
console.log(sanitizeNames(CARS))
//三、
//练习1
let ex1 = maybe.map(x => fp.map(fp.add(1), x))
console.log(ex1)
//练习2
let ex2 = fp.map(fp.first)
console.log(xs.map(ex2))
//练习3
let ex3 = fp.flowRight(fp.map(fp.first), safeProp('name'))
console.log(ex3(user))
//练习4
let ex4 = n=>Maybe.of(n).map(parseInt)
console.log(ex4)
//四、
class Mypromise {
    constructor(executor) {
        this.status = 'pending'  //状态值
        this.value = undefined   //成功的返回值
        this.reason = undefined	 //失败的返回值
        this.onResolvedCallbacks = [] //成功的回调函数
        this.onRejectedCallbacks = [] //失败的回调函数
        // 成功
        let resolve = (value) => {
            // pending用来屏蔽的，resolve和reject只能调用一个，不能同时调用，这就是pending的作用
            if (this.status == 'pending') {
                this.status = 'fullFilled'
                this.value = value
                // 发布执行函数
                this.onResolvedCallbacks.forEach(fn => fn())
            }
        }
        // 失败
        let reject = (reason) => {
            if (this.status == 'pending') {
                this.status = 'rejected'
                this.reason = reason
                //失败执行函数
                this.onRejectedCallbacks.forEach(fn => fn())
            }
        }
        try {
            // 执行函数
            executor(resolve, reject)
        } catch (err) {
            // 失败则直接执行reject函数
            reject(err)
        }
    }
    then(onFullFilled, onRejected) {
        // 同步
        if (this.status == 'fullFilled') {
            onFullFilled(this.value)
        }
        if (this.status == 'rejected') {
            onRejected(this.reason)
        }
        // 异步
        if (this.status == 'pending') {
            // 在pending状态的时候先订阅
            this.onResolvedCallbacks.push(() => {
                // todo
                onFullFilled(this.value)
            })
            this.onRejectedCallbacks.push(() => {
                // todo
                onRejected(this.reason)
            })
        }
    }
}

const p = new Mypromise((resolve, reject) => {
    setTimeout(function() {
        // resolve('success') // 异步调用的时候，this.status一直是pending状态,不会执行代码了，因此要改装成发布订阅者模式
        reject('failed')
    }, 1000)
    // resolve('success') // 走了成功就不会走失败了
    // throw new Error('失败') // 失败了也会走resolve
    // reject('failed')
})
p.then((res) => {
    console.log(res)
}, (err) => {
    console.log(err)
})
p.then((res) => {
    console.log(res)
}, (err) => {
    console.log(err)
})
p.then((res) => {
    console.log(res)
}, (err) => {
    console.log(err)
})


