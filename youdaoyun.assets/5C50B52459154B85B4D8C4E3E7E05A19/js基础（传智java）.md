1. javascript基础语言
	javascript语法体系
			1）EMCA基础语法（统一）
			2）BOM编程（不统一）
			3）DOM编程（不统一）
	1.1 javascript使用
	1.2 变量和数据类型
	1.3 类型转换函数
		 string->number(整数) :  parserInt(变量)
		 string->number(小数):  parserFloat(变量)

	1.4 运算符
	1.5 流程控制语句
		if
		swtich
		for
		while
		do-while
		for-in
		with
	1.6 函数
	1.7 基于对象编程
		内置对象
			String对象
			Number对象
			Boolean对象
			Math对象
			Date对象
			Array数组对象
  1.8 自定义对象
		java：使用class来定义对象
		javascript： 使用function来定义对象
	1.9 原型
		给内置对象追加方法

		
2. 练习（原型作业）

```
<!-- 
给String对象，添加两个方法：
	toCharArray(): 返回字符数组
    reverse(): 反转字符串的内容

-->
<script type="text/javascript">
	String.prototype.toCharArray = function(){
		//1.创建数组
		var charArray = new Array();
		//2.遍历字符
		for(var i=0;i<this.length;i++){
			charArray[i]=this.charAt(i);	
		}	
		return charArray;
	}
	
	String.prototype.reverse = function(){
		//1.把String转为数组
		var charArray = this.toCharArray();
		//2.对数组进行反转	
		charArray = charArray.reverse();
		//3.把数组变为String
		return charArray.join("");
	}
	
	var str = "hellojava";
	var arr = str.toCharArray();
	for(var i=0;i<arr.length;i++){
		document.write(arr[i]+",");	
	}
	document.write("<hr/>");
	str = str.reverse();
	document.write(str);
	
	

</script>
```

#### BOM编程
1. 概念
全称 Browser Object Model，浏览器对象模型。
	JavaScript是由浏览器中内置的javascript脚本解释器程序来执行javascript脚本语言的。
	为了便于对浏览器的操作，javascript封装了对浏览器的各个对象使得开发者可以方便的操作浏览器。

2. window对象：Window 对象是 JavaScript 层级中的顶层对象。
Window 对象代表一个浏览器窗口或一个框架。
Window 对象会在 <body> 或 <frameset> 每次出现时被自动创建。
###### window中的方法

    alert()		显示一个警告框。
	confirm()	     选择确定框。
	prompt()        输入框。

	moveto()  将窗口左上角的屏幕位置移动到指定的 x 和 y 位置。
	moveby()    相对于目前的位置移动。
    resizeTo()   调整当前浏览器的窗口。

	open()		打开新窗口显示指定的URL（有的浏览器中是打一个新的选项卡）
	setTimeout(vCode, iMilliSeconds)		超时后执行代码。
    setInterval(vCode, iMilliSeconds)		定时执行代码，第一次也是先待，到时再执行。

```
<script type="text/javascript">
	/*
	 open(): 在一个窗口中打开页面
	 
	 setInterval(): 设置定时器（执行n次）
	 setTimeout(): 设置定时器(只执行1次)
	 clearInterval(): 清除定时器
	 clearTimeout(): 清除定时器
	 
	 alert(): 提示框
	 confirm(): 确认提示框
	 propmt(): 输入提示框
	 
	 注意：
	 	因为window对象使用非常频繁，所以当调用js中的window对象的方法时，可以省略对象名不写。
	
	*/
	function testOpen(){
		/*
		参数一： 打开的页面
		参数二：打开的方式。 _self: 本窗口  _blank: 新窗口（默认）
		参数三： 设置窗口参数。比如窗口大小，是否显示任务栏
		*/
		window.open("02.广告页面.html","_blank","width=300px;height=300px;toolbar=0");
	}
	
	var taskId;
	function testInterval(){
		
		/*
		定时器： 每隔n毫秒调用指定的任务（函数）
		参数一：指定的任务（函数）
		参数二：毫秒数
		*/
		taskId = window.setInterval("testOpen()",3000);	
	}
	
	function testClearInterval(){
		/*清除任务
		参数一：需要清除的任务ID
		*/
		window.clearInterval(taskId);	
	}
	
	var toId;
	function testTimeout(){
		/*设置定时任务*/
		toId = window.setTimeout("testOpen()",3000);	
	}
	
	function testClearTimeout(){
		window.clearTimeout(toId);
	}


	function testAlert(){
		window.alert("提示框");	
	}
	
	function testConfirm(){
		/*
		返回值就是用户操作
			true: 点击了确定
			false： 点击了取消
		*/
		var flag = window.confirm("确认删除吗？一旦删除不能恢复，请慎重！");	
		if(flag){
			alert("确定删除，正在删除中....");	
		}else{
			alert("取消了操作");	
		}
	}
	
	function testPrompt(){
		/*
		输入提示框
		*/
		var flag = window.prompt("请输入你的U顿密码");
		if(flag){
			alert("密码正确，转账中...");	
		}else{
			alert("取消了操作");	
		}
	}
    
var name=prompt("Please enter your name","")
  if (name!=null && name!="")
    {
    document.write("Hello " + name + "!")
    }
  }

</script>
```

3. location对象：Location 对象是由 JavaScript runtime engine 自动创建的，包含有关当前 URL 的信息。
location中的重要方法：
	href属性	设置或获取整个 URL 为字符串。
	reload()	重新装入当前页面

4. history对象：

```
window.history.forward:前进到下一页
back:上一页
go:某一页
```

5. screen对象

```
/*
		availHeight和availWidth是排除了任务栏（window任务栏）之后的高度和宽度
	*/
	document.write(window.screen.availWidth + "<br/>");
	document.write(window.screen.availHeight + "<br/>");
	document.write(window.screen.width + "<br/>");
	document.write(window.screen.height + "<br/>");
```



