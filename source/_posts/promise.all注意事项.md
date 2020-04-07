title: promise.all注意事项
date: 2019-11-14 10:11:02
tags: [js]
categories: 学习笔记

---
记录一下最近遇到的bug。
### 问题出现
		//最外层
		async doRequest(){
			...
			// 调用处
			let ret = await fun(...)
			...
		}
		
		
		// fun()内部
		fun(){
			...
			let promises = []
			for(;;){
				...
				promises.push(new Promise(resolve, reject){
					axios.post(...).then(()=>{resolve()}).catch(()=>{reject()})
				})
				...
			}
			...
			
			let results = await Promise.all(promises)
			//遍历results，整理结果
			for(;;){
				if(results[i].status == 200){
					...
				}
				else{
					...
				}
			}
			return ...
		}
<!-- more -->
出现bug的时候，axios请求的后端接口直接报500错误，后端数据库报错，database error，然后整个方法好像就停在了`let ret = await fun(...)`这一步，后面的代码都不再执行了。  

### 问题分析
1.	promise.all没有考虑出错情况，或者是有相当出错情况，但是处理方式不对
2. promise的运行方式不太了解
3. 使用promise.all一次发送了太多的请求，可以和后端沟通提供批量处理的接口，将所有的参数包裹在一次请求里面  

### 具体原理
1.	promise.all获得成功的结果顺序和promises数组顺序是相同的
2.	promise.all如果有一个回调执行失败，那么整个都会失败，并且会立即返回失败
3. 在async函数内部，await后面的函数如果抛出错误，而且没有被正确的处理，那么async函数也会中止执行，等同于promise对象被reject了
4. axios默认是200到300之间的返回status执行resolve，而其他的状态码执行reject

### 执行过程
1.	promise.all这里，当其中一个axios.post返回500时，执行axios的catch方法，即new Promise的reject方法，reject方法返回rejected状态的promise对象.   
2. new Promise返回rejected状态的promise对象，  
3. 错误被promise.all捕获，promise.all返回rejected，  
4. promise.all没有catch方法对错误进行处理，所以被async函数捕获错误，直接中止后面的代码执行  

### bug修复
在上面说的任意一步对抛出的异常进行处理，然后返回处理后的结果，那么后面就不会终止async函数的执行，但是在各个步骤加错误处理得到的结果可能会不太一样  
1.	定义axios的响应拦截器,  `Axios.interceptors.response.use(response=>{}, error=>{})`.   
在error处理里面修改对状态码的判断，即使是返回500也执行resolve方法,这个修改方式对现有的逻辑影响会比较大
2. 在`new Promise`这里定义`catch`函数，将错误处理后执行resolve结果，这个方式是比较妥当的，promise.all可以继续按原来的逻辑执行，也不会影响其他的地方
3. 对promise.all方法，添加`catch`函数，将错误处理后执行resolve，这个方式虽然能够让async函数继续执行下去，但是`let results = await Promise.all(promises)`这一步results就不会有任何返回了，后面的代码执行下去也是错的了，所以这样修改虽然能够保证代码执行，但是得到的是错误的结果

### 参考资料
[1].	[axios文档](http://www.axios-js.com/docs/)  
[2].	[promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)   
[3].	[async](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)