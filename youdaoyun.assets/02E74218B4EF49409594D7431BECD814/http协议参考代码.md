### request
#### 使用HttpServletRequest获取请求信息

```
package gz.itcast.b_request;

import java.io.IOException;
import java.io.InputStream;
import java.util.Enumeration;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 使用HttpServletRequest获取请求信息
 * @author APPle
 *   HttpServletRequest对象：获取请求数据
 *   	请求行：
 *   		请求方式： request.getMethod()
 *   		请求资源： request.getRequestURI() /  request.getRequestURL()
 * 			http协议版本： request.getProtocol();
 * 		请求头
 * 			request.getHeader("name") 
 * 			request.getHeaderNames()
 * 		实体内容:
 * 			request.getInputStream();
 */
public class RequestDemo1 extends HttpServlet {

	//1)tomcat服务器接收到浏览器发送的请求数据
	//2)tomcat服务器把请求数据封装成HttpServletRequest对象
	//3)tomcat服务器调用doGet方法，把request对象传入servlet
	
	//doGet方法只能接受get提交的请求
	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		//4)通过request对象获取请求数据
		
		/**
		 * 请求数据
		 */
		//t1(request);
		
		//t2(request);
			
	}
	
	
	//doPost接受post提交的请求
	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse resp)
			throws ServletException, IOException {

		//4.3 实体内容
		//注意： post提交的参数才会出现在实体内容中
		InputStream in = request.getInputStream();
		byte[] buf = new byte[1024];
		int len = 0;
		while(  (len=in.read(buf))!=-1   ){
			String str = new String(buf,0,len);
			System.out.print(str);
		}
	}

	private void t2(HttpServletRequest request) {
		//4.2 请求头
		String value = request.getHeader("host");// 根据头名称获取头值
		System.out.println("host:"+value);
		
		//遍历所有头
		Enumeration<String> enums = request.getHeaderNames();// 获取所有头名称
		while(enums.hasMoreElements()){
			String headerName = enums.nextElement();
			String headerValue = request.getHeader(headerName);
			System.out.println(headerName+":"+headerValue);
		}
	}

	private void t1(HttpServletRequest request) {
		//4.1 请求行
			// 请求方式
		System.out.println("请求方式："+request.getMethod());
			// 请求资源
		System.out.println("URI:"+request.getRequestURI());
		System.out.println("URL:"+request.getRequestURL());
			// http协议版本
		System.out.println("Http协议："+request.getProtocol());
	}

}

```
#### service方法和doXXX方法的关系

```
package gz.itcast.b_request;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 结论： service方法和doXXX方法的关系？
 * 	1）service方法是程序的入口。我们的代码逻辑就在这个方法被调用到。
 *  2）在HttpServlet的service方法源码中，根据不同请求方式调用了不同的doXX方法，
 *     所以我们在开发servlet的时候，就不需要去覆盖service方法，而是去doXX方法。
 *     因为get和post是最常用的的两种请求方式，所以只需要覆盖doGet和doPost方法即可！
 *     
 * @author APPle
 *
 */
public class RequestDemo2 extends HttpServlet {
	
	/**
	 * 这个service方法是servlet的核心的服务方法。我们的业务逻辑都是在这个方法开始被触发的。
	 * 这是一个入口
	 */
	/*@Override
	protected void service(HttpServletRequest req, HttpServletResponse resp)
			throws ServletException, IOException {
		System.out.println("调用了service方法");
	}*/
	
	/**
	 * doGet用于接收get提交的请求
	 */
	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		System.out.println("调用了doGet方法");
	}

	/**
	 * doPost用于接收post提交的请求
	 */
	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		System.out.println("调用了doPost方法");
	}

	
	/*public void doPut(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

	}
*/
}

```
#### 案例【user-agent】 浏览器类型

```
package gz.itcast.b_request;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 案例【user-agent】 浏览器类型
 * @author APPle
 */
public class RequestDemo3 extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		
		String userAgent = request.getHeader("user-agent");
		//System.out.println(userAgent);
		
		if(userAgent.contains("Firefox")){
			response.getWriter().write("你正在使用火狐浏览器");
		}else if(userAgent.contains("Chrome")){
			response.getWriter().write("你正在使用谷歌浏览器");
		}else if(userAgent.contains("Trident")){
			response.getWriter().write("你正在是IE浏览器");
		}else{
			response.getWriter().write("识别不了的浏览器");
		}
		
		
	}

}

```
#### 案例-【rerfer】

```
package gz.itcast.b_request;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 案例-【rerfer】
 * @author APPle
 *
 */
public class RequestDemo4 extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		/**
		 * 代表下载资源文件
		 * referer: 表示当前请求来自于哪里
		 */
		String referer = request.getHeader("referer");
		System.out.println("referer="+referer);
		
		/**
		 * 判断非法请求（链接）
		 * 		1）直接访问  （referer==null）
		 *      2)当前请求不是来自于广告页面  （  !referer.contains("adv.html")  ）
		 */
		if(referer==null || !referer.contains("/adv.html")){
			response.getWriter().write("你当前请求是非法请求，请转到首页。<a href='/day08/adv.html'>首页</a>");
		}else{
			response.getWriter().write("资源正在下载.....");
		}
		
	}

}

```
#### 案例【获取请求的参数数据】

```
package gz.itcast.b_request;

import java.io.IOException;
import java.util.Collection;
import java.util.Enumeration;
import java.util.Map;
import java.util.Set;
import java.util.Map.Entry;

import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 案例【获取请求的参数数据】
 * @author APPle
 *
 */
public class RequestDemo5 extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		/**
		 * 设置参数解码时查询的码表
		 *   注意：
		 *   	只对post提交的参数有效，对get提交的参数无效的！
		 *   因为这个方法只能设置对请求实体内容的解码时查询的码表。post提交的参数时放在实体内容中，所以对post是有效的！
		 *   但是get提交参数时放在URI后面的，这个方法无法影响URI后面的内容。
		 */
		request.setCharacterEncoding("utf-8");

		
		/**
		 * 获取get提交的参数.（URI后面的参数数据）
		 */
		
		/**
		 * 问题：
		 * 	1）获取到的参数数据，还需要进一步处理，获取参数值
		 *  2）两种提交方式的获取代码完全不一样的，不通用。
		 */
		
		/**
		 * 通用的获取参数的方法：（无论get和post可用）
		 * 		reuqest.getParameter("name")  获取一个值的参数
		 * 		request.getParameterValue("name")  获取多个值的参数
		 *      request.getParameterNames() 获取所有参数名称
		 *      request.getParameterMap()   获取所有参数对象
		 */
		
		/**
		String params = request.getQueryString();
		System.out.println(params);
		*/
		String name = request.getParameter("name");//根据参数名称获取参数值（参数名称就是表单的name属性值）
		
		/**
		 * 只对get方式采取对参数进行手动解码
		 */
		if("GET".equals(request.getMethod())){
			name = new String(name.getBytes("iso-8859-1"),"utf-8");
		}
		
		System.out.println(name);
		
		System.out.println("=======");
		
		//遍历所有的参数
		Enumeration<String> enums = request.getParameterNames();//获取所有参数名称列表
		while(enums.hasMoreElements()){
			String paramName = enums.nextElement();
			String paramValue = request.getParameter(paramName);
			System.out.println(paramName+"="+paramValue);
		}
		System.out.println("=======");
		
		Map<String,String[]> map= request.getParameterMap(); //获取参数对象列名 （Map集合） 
		/**
		 * 每个map的对象就是一个参数（包含参数名称和参数值）
		 * 	key: 参数名称
		 * 	value: 参数值(默认情况下都是多个值的参数)
		 */
		/**
		 * 复习Map集合
		 * 	问题： Map集合如何遍历？
		 *     1）entrySet（）
		 *     2）keySet()
		 *     3) values() 		
		 */
		//1)entrySet()方法：获取键值对对象的Set集合
		//    Entry对象中包含一个键对象，和一个值对象
		
		Set<Entry<String,String[]>> entrySet = map.entrySet();
		for (Entry<String, String[]> entry : entrySet) {
			//获取键对象
			String key = entry.getKey();
			//获取值对象(数组的第一个元素就是参数值)
			String[] value = entry.getValue();
			System.out.println(key+"="+value[0]);
		}
		
		/**
		//2)keySet()： 获取所有键对象的Set集合
		Set<String> keySet = map.keySet();
		for (String key : keySet) {
			//通过键对象获取值对象
			String[] value = map.get(key);
			System.out.println(key+"="+value[0]);
		}
		
		System.out.println("==========");
		
		//3)values(): 获取所有值对象的Collection集合(只能获取值对象，不能获取键对象)
		Collection<String[]> values = map.values();
		for (String[] value : values) {
			System.out.println(value[0]);
		}
		*/
		
		
		String[] hobits = request.getParameterValues("hobit"); //根据参数名称获取多个参数值
		for (String h : hobits) {
			System.out.print(h+",");
		}
	}

	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		/**
		 * 获取post提交的参数（实体内容中） 
		 */
		/*ServletInputStream in = request.getInputStream();
		byte[] buf = new byte[1024];
		int len = 0;
		while( (len=in.read(buf))!=-1  ){
			String str = new String(buf,0,len);
			System.out.print(str);
		}  */
		
		
		//遍历所有的参数
		/*Enumeration<String> enums = request.getParameterNames();//获取所有参数名称列表
		while(enums.hasMoreElements()){
			String paramName = enums.nextElement();
			String paramValue = request.getParameter(paramName);
			System.out.println(paramName+"="+paramValue);
		}*/
		
		//在doPost方法中调用doGet里面的逻辑代码
		doGet(request,response);
		
	}

}

```

### responce
#### 使用HttpServletResponse修改响应数据

```
package gz.itcast.c_response;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 使用HttpServletResponse修改响应数据
 *      response.setStatus(404) 设置状态码
 *      response.setHeader("name","value")  修改响应头
 *      response.getWriter().write()   以字符形式发送实体内容
 *      response.getOutputStream().write()  以字节形式发送实体内容
 * @author APPle
 *
 */
public class ResponseDemo1 extends HttpServlet {

	//1)tomcat服务器提供了一个HttpServletResponse对象，用于给开发者修改响应数据
	//2)通过service方法把response对象传入servlet中
	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		//3)通过response对象修改响应数据
		/**
		 * 修改响应
		 */
		//3.1 响应行
			//设置状态码
		//response.setStatus(404);
		//response.sendError(404); // 404+404错误页面
		
		//3.2 响应头
		//response.setHeader("server", "webLogic");
		
		//3.3 实体内容(在浏览器主题部分看到的内容)
		//response.getWriter().write("this is content!");   字符流   
		response.getOutputStream().write("this is content!!!".getBytes());  //字节流
		
	}
	
	//4)tomcat服务器把response对象转换成响应格式的字符串，发送给浏览器

}

```
#### 案例【location+302】请求重定向

```
package gz.itcast.c_response;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 案例【location+302】请求重定向
 * @author APPle
 *
 */
public class ResponseDemo2 extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		/**
		 * 请求重定向（  跳转到其他页面 ）
		 */
		/*//设置302状态码
		response.setStatus(302);
		//设置location响应头
		response.setHeader("location", "/day08/adv.html");*/
		
		/**
		 * 简化版本
		 */
		response.sendRedirect("/day08/adv.html");
	}

}

```
#### 案例【refresh】--定时刷新或每隔n秒跳转页面

```
package gz.itcast.c_response;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 案例【refresh】--定时刷新或每隔n秒跳转页面
 * @author APPle
 *
 */
public class ResponseDemo3 extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		
		//设置refresh响应头
		//设置秒数
		//response.setHeader("refresh", "2");
		
		//每隔n秒跳转页面
		response.getWriter().write("注册成功！3秒之后会跳转到主页");
		//设置refresh
		response.setHeader("refresh", "3;/day08/adv.html");
	}

}

```

#### 案例【content-type】--服务器发送给浏览器的数据类型和数据编码格式

```
package gz.itcast.c_response;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 案例【content-type】--服务器发送给浏览器的数据类型和数据编码格式
 * @author APPle
 *
 */
public class ResponseDemo4 extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		//设置content-type响应
		//response.setHeader("content-type", "text/html;charset=utf-8");
		//response.setContentType("text/html;charset=utf-8");//等价于上面的代码
		
		
		/**
		 * 1)设置数据类型
		 */
		//response.setContentType("text/html");//告诉浏览器以什么样的格式来解析实体内容
		//response.setContentType("image/jpg");//告诉浏览器以什么样的格式来解析实体内容
		/**
		 * 注意： 一定要写服务器支持的数据类型，如果写了服务器不支持的类型，就会报错
		 */
		/**
		 * 2）设置数据编码格式
		 */
		/**
		 *    两个作用：
		 *    		1）设置输出数据的编码
		 *          2）告诉浏览器自动适应输出数据的编码
		 */
		response.setContentType("text/html;charset=gbk"); //和下面的代码是效果是一样的。
		//response.setCharacterEncoding("utf-8"); //不会告诉浏览器自动跳转解码的码表 
		
		response.getWriter().write("<html><head><title>this is tille</title></head><body>主题</body></html>");
		
		
	}

}

```
#### 案例【Content-Disposition】 -- attachment; filename=aaa.zip   -- 以下载方式打开资源

```
package gz.itcast.c_response;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
 * 案例【Content-Disposition】 -- attachment; filename=aaa.zip   -- 以下载方式打开资源
 * @author APPle
 *
 */
public class ResponseDemo5 extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		File file = new File("e:/mm.jpg");
		/**
		 * 告诉浏览器以下载的方法打开
		 */
		response.setHeader("content-disposition", "attachment;filename="+file.getName());
		
		/**
		 * 文件下载
		 */
		//1)读取本地文件
		FileInputStream in = new FileInputStream(file);
		
		//2)写出给浏览器(字节内容)
		OutputStream out = response.getOutputStream();
		byte[] buf = new byte[1024];
		int len = 0;
		//边读边写
		while( (len=in.read(buf))!=-1  ){
			out.write(buf, 0, len);
		}
		
		//关闭
		in.close();
		out.close();
		
		
	}

}

```



