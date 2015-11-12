<style>
.green{ color:#3c763d; }
</style>
# log
前端记录

<h4>JavaScript设计模式笔记</h4>
<h4>第一章 富有表现力的JavaScript</h4>
<h5>1.1 JavaScript的灵活性</h5>
例子：启动和停止动画
<span class="green">method1：</span>
<pre>
/* 使用函数：简单、无法创建保存状态及内部操作的对象 */
function startAnimation(){
	...
}
function stopAnimation(){
	...
}
<span class="green">method2：</span>
</pre>
<pre>
/* 使用类 */
// class Anim
var Anim = function(){
	...
}
Anim.prototype.start = function(){
	...
}
Anim.prototype.stop = function(){
	...
}
// usage
var myAnim = new Anim();
myAnim.start();
...
myAnim.stop();
</pre>
<span class="green">method3：</span>
<pre>
/* 类方法统一定义 */
// class Anim
var Anim = function(){
	...
}
Anim.prototype = {
	start:function(){
		...
	},
	stop:function(){
		...
	}
}
</pre>
<span class="green">method4：</span>
<pre>
/* 使用Function */
Function.prototype.method = function(name,fn){
	this.prototype[name] = fn;
};
// class Anim
var Anim = function(){
	...
};
Anim.method("start",function(){
	...
})
Anim.method("stop",function(){
	...
})
</pre>
<span class="green">method5：</span>
<pre>
/* Function 链式 */
Function.prototype.method = function(name,fn){
	this.prototype[name] = fn;
	return this;	//返回this
}
var Anim = function(){
	...
};

Anim.method("start",functtion(){
	...
}).method("stop",function(){
	...
})
</pre>
<h5>1.2 弱类型语言</h5>
JavaScript有3种原始类型：布尔型、数值型和字符串类型；此外还有，对象类型、函数类型、空类型、未定义类型。原始类型按值传递，而其他类型按引用传递。
<h2>部分小节略...</h2>
<h4>第2章 接口</h4>
<h5>2.1 什么是接口<h5>
接口提供了一种用以说明一个对象应该具有哪些方法的手段。(提示对象内部必须实现某些方法)
<h5>2.3 JavaScript中模仿接口<h5>
<span class="green">用注释描述接口</span>
<pre>
	/*
	interface Composite{
		function add(child)
		function remove(child)
		function getChild(index)
	}
	
	interface FormItem{
		function save()
	}
	*/
	
	var CompositeForm = function(id,method,action){
		//implements Composite,FormItem
		...
	};
	// implement Composite interface
	CompositeForm.prototype.add = function(child){
		...
	};
	CompositeForm.prototype.remove = function(child){
		...
	};
	CompositeForm.prototype.getChild = function(index){
		...
	};
	//implement FormItem interface
	CompositeForm.prototype.save = function(){
		...
	}	
</pre>
此方法易于实现，但并不保证ComposiForm真正实现正确的方法集并进行检查。
<span class="green">用属性检查模仿接口</span>
<pre>
	/*
	interface Composite{
		function add(child)
		function remove(child)
		function getChild(index)
	}
	
	interface FormItem{
		function save()
	}
	*/
	
	var CompositeForm = function(id,method,action){
		//implements Composite,FormItem
		this.implementsInterfaces = ['Composite','FormItem'];
		...
	};
	...
	function addForm(formInstance){
		if(!implements(formInstance,['Composite','FormItem'])){
			throw new Error("Object does not implement a require interface.")
		}
		...
	}
	
	//接口检查函数
	function implements(object){
		for(var i=1; i<arguments.length; i++){
			//遍历第1个参数后的参数
			var interfaceName = arguments[i];
			var interfaceFound = false;
			for(var j=0; j<object.implementsInterfaces.length; j++){
				if(interfaceName == object.implementsInterfaces[j]){
					interfaceFound = true;
					break;
				}
			}
			if(!interfaceFound){
				//没有找到
				return false;
			}
		}
		//所有interface找到
		return true;
	}
</pre>
此方法并未确保类真正实现了自称实现的接口，只知道类是否说自己实现了借口
<span class="green">用鸭式辩型模仿接口</span>
/* 鸭式辩型：像鸭子一样走路并且嘎嘎叫的就是鸭子 */
<pre>
	//Interfaces
	var Composite = new Interface("Composite",["add","remove","getChild"]);
	var FormItem = new Interface("FormItem",["save"]);
	
	//CompositeForm class
	var CompositeForm = function(id,method,action){
		...
	};
	...
	function addForm(formInstance){
		Interface.ensureImplements(formInstance,Composite,FormItem);
		...
	}
</pre>
<span class="green">Interface 类定义</span>
<pre>
	//Comstructor
	var Interface = function(name,methods){
		if(arguments.length != 2){
			throw new Error("Interface constructor called with" + arguments.length + "arguments,but expected exactly 2");
		}
		this.name = name;
		this.methods = [];
		for(var i=0,len=methods.length; i<len; i++){
			if(typeof methods[i] !== 'string'){
				throw new Error("Interface contructor expects method name to be passed in as a string");
			}
			this.methods.push(methods[i]);
		}
	}
	
	//static class method
	Interface.ensureImplements = function(object){
		if(arguments.length < 2){
			throw new Error("Function Interface.ensureImplements called with" + arguments.length + "arguments,but expected at least 2")
		}
		
		for(var i=1,len=arguments.length; i<len; i++){
			var interface = arguments[i];
			if(interface.constructor !== Interface){
				throw new Error("Function Interface.ensureImplements expects arguments two and above to be instances of Interface.")
			}
			
			for(var j=0,methodLen=interface.methods.length; j<len; j++){
				var method = interface.methods[j];
				if(!object[method]|| typeof object[method] !== 'function'){
					throw new Error("Function Interface.ensureImplements:object does not implement the" + interface.name + " interface.Method "+ method +"was not found.")
				}
			}
		}
	}
</pre>
<h4>第3章 封装和信息隐藏</h4>
<h5>3.2 创建对象的基本模式</h5>
例：创建一个用于存储关于一本书的数据类。
<pre>
	//Book(isbn,title,author)
	//使用方式如下：
	var theHobbit = new Book('0-395-07122-4','The Hobbit','J.R.R Tolkien');
	theHobbit.display()
</pre>
<h5>3.2.1 门户大开型对象</h5>
用一个函数作其构造器，由于其所有属性及方法都是公开可访问的，这种创建对象模式叫做门户大开型
<pre>
	var Book = function(isbn,title,author){
		if(isbn == undefined){
			throw new Error("Book constructor require an isbn");
		}
		this.isbn = isbn;
		this.title = title || "No title specified";
		this.author = author || "No author specified";
	}
	Book.prototype.display(){
		...
	}
</pre>
以上Book由于无法验证isbn，可能导致display失效，做修改
<pre>
	var Book = function(isbn,title,author){
		if(isbn == undefined){
			throw new Error("Book constructor require an isbn");
		}
		this.isbn = isbn;
		this.title = title || "No title specified";
		this.author = author || "No author specified";
	}
	Book.prototype = {
		checkIsbn:function(isbn){
			if(isbn == 'undefined' || typeof isbn != 'string'){
				return false;
			}
			
			isbn = isbn.replace(/-/,"");	//替换掉'-'
			if(isbn.length != 10 && isbn.length != 13){
				//只能包含3个'-'或者不包含'-'
				return false;
			}
			
			var sum = 0;
			if(isbn.length === 10){//10位
				if(!isbn.match(/^\d{9}/)){
					//保证前9个是数字
					return false;
				}
				
				for(var i=0; i<9; i++){	//校验和
					sum += isbn.charAt(i)*(10-i);
				}
				var checksum = sum % 11;
				if(checksum === 10){
					checksum = 'X';
				}
				if(isbn.charAt(9) != checksum){
					return false;
				}
			}esle{//13位
				if(!isbn.match(/^\d{12}/)){
					//前12位是数字
					return false;
				}
				
				for(var i=0; i<12; i++){	//检验和
					sum += isbn.charAt(i)*((i%2 === 0)? 1 : 3);
				}
				var checksum = sum % 10;
				if(isbn.charAt(12) != checksum){
					return false;
				}			
			}
			return true;		
		},
		display:function(){
			...
		}
	}
</pre>
<span class="green">为保护内部数据应设置</span>
<pre>
	/* 加强版 */
	
	var Publication = new Interface("Publication",['getIsbn','setIsbn','getTitle','setTitle','getAuthor','setAuthor','display']); //定义接口
	
	var Book = function(isbn,title,author){
		//implement Publiction
		this.setIsbn(isbn);
		this.setTitle(title);
		this.setAuthor(author);
	};
	
	Book.prototype = {
		checkIsbn:function(isbn){
			...
		},
		getIsbn:function(){
			return this.isbn;
		},
		setIsbn:function(isbn){
			this.isbn = isbn;
		},
		getTitle:function(){
			return this.title;
		},
		setTitle:function(title){
			this.title = title;
		},
		getAuthor:function(){
			return this.author;
		},
		setAuthor:function(author){
			this.author = author;
		},
		display:function(){
			...
		}
	}
</pre>


































