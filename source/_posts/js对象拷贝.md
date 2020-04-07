title: js对象拷贝
date: 2019-11-23 15:55:22
tags: [js]
categories: 学习笔记
about: 

---
### 问题出现
1.因为后端传过来的数据的形式和展示到页面上所需要的形式不太一致，所以需要将数据重新组装一下。  
2.首先想的是重新自己定义一个对象，然后把数据遍历一边，把需要的数据拿出来，放到自己定义对象里面，这是很常规的操作，但是最后得到的却不是我预想的结果  

### 具体代码  
		// 原始数据 origin
		// 定义两个对象存储最后组装的结果
		let result = []
		for(let i = 0; i < origin.length ;i++){
			let attr1 = orogin[i].attr1
			for(let j = 0; j < attr1.length; j++){
				let attr2 = attr1[j].attr2
				let item = {}
				let item = {'a1': attr2['a1'], 'a2': attr2['a2']...}
				result.push[item]
			}
		}
<!-- more -->
这样最后得到的result里面的结果item,都是最后一个元素的值。  

### 问题分析  
1.其实这个问题是很容易想到的，只要最后的结果发现不一致，稍微推敲一下就知道是result存的是item对象的引用，而没有存入真正的值  
2.这里面涉及到的js对象的一些知识还是比较丰富的，单纯解决这个问题比较简单，采用深复制的方法，result里面存item深复制后的结果就好了

### 具体原理  
1.js对象,对象是属性的集合。  
  1.1.访问未声明的对象的属性得到的是undefined,而不是null。  
  1.2.属性名如果是动态的，只能通过方括号标记访问。  
  1.3.枚举对象所有属性的方式(ES5以后)：for...in...可以访问对象本身和原形链中的属性；Object.keys()返回不包括原形链中的自身所有可枚举属性的数组；Object.getOwnPropertyNames()返回不包括原形链中的所有属性，包括不可枚举的属性   
  1.4.对象的属性可以是函数  
2.原型和类   

|  基于类的（Java）   | 基于原型的（JavaScript）  |
|  ----  | ----  |
| 类和实例是不同的事物。  | 所有对象均为实例。   |
| 通过类定义来定义类；通过构造器方法来实例化类。  | 通过构造器函数来定义和创建一组对象。 |
| 通过 new 操作符创建单个对象。  | 相同。 |
| 通过类定义来定义现存类的子类，从而构建对象的层级结构。  | 指定一个对象作为原型并且与构造函数一起构建对象的层级结构 |
| 遵循类链继承属性。  | 遵循原型链继承属性。 |
| 类定义指定类的所有实例的所有属性。无法在运行时动态添加属性。  | 构造器函数或原型指定初始的属性集。允许动态地向单个的对象或者整个对象集中添加或移除属性。 |   
3.原形链
4.深拷贝与浅拷贝   
4.1.浅拷贝只拷贝对象里的一层的属性，而对象的属性有的是对其他对象的引用，如果只拷贝引用，当引用的对象值发生变化的时候，拷贝得到的结果也发生了变化
4.2.浅拷贝的实现方式：=号赋值；for...in...赋值；Object.assign;
4.3.深拷贝就是拷贝多层，将属性中引用的其他对象的值也拷贝一份
4.4.深拷贝的实现方式：递归拷贝对象所有层级上的属性；使用JSON对象的序列化和反序列化方法；lodash函数；Object.create方式

### 解决办法   
1.深拷贝的实现方式  
1.1.对于没有深层次引用的对象，深拷贝很容易，for...in...和Object.assign都可以实现   
1.2.对于有深层次结构，但是都是值都是json中可表示的类型，比如number，string,array等等，可以直接用JSON的处理方式，如果是其他的类型，比如function这种，用这种方式就不行，因为function会被转化成其他类型的对象   
1.3.递归拷贝的方式可以处理大部分的情况，但是对于互相引用的情况可能会陷入死循环  
1.4.Object.create方式现在看来是最简单的深拷贝的方式



### 参考资料
[1]. [Working with objects](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_Objects).   
[2]. [Details of the object model](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Details_of_the_Object_Model).  
[3]. [继承与原形链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

