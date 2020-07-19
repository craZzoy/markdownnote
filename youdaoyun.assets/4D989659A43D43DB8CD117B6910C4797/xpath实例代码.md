#### xpath入门

```
package gz.itcast.a_xpath;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.util.List;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;
/**
 * xpath入门
 * @author APPle
 *
 */
public class Demo1 {

	
	public static void main(String[] args) throws Exception {
		/**
		 * 需求： 删除contact.xml中id属性值为002的contact标签
		 */
		//1)读取contact.xml文件
		Document doc = new SAXReader().read(new File("e:/contact.xml"));
		
		//2)删除id=002的contact标签
		/*List<Element> conList = doc.getRootElement().elements("contact");
		for(Element e:conList){
			if(e.attributeValue("id").equals("002")){
				//删除
				e.detach();
				break;
			}
		}*/
		
		//查询到id属性值为002的contact标签
		Element conElem = (Element)doc.selectSingleNode("//contact[@id='002']");
		conElem.detach();
		
		
		//3)把修改后的document写出到硬盘的xml文件中
		OutputStream out = new FileOutputStream("e:/contact.xml");
		OutputFormat format = OutputFormat.createPrettyPrint();
		XMLWriter writer = new XMLWriter(out,format);
		writer.write(doc);
		//关闭
		writer.close();
	}

}

```
#### xpath语法

```
package gz.itcast.a_xpath;

import java.io.File;
import java.util.List;

import org.dom4j.Document;
import org.dom4j.Node;
import org.dom4j.io.SAXReader;

/**
 * 			/   绝对路径    斜杠在最前面，代表xml文件的根。斜杠在中间，表示子元素。
			//  相对路径    选择后代元素（不分层次结构）
			*   通配			选择所有元素
			[ ]   条件        选择什么条件下的元素。例如 /AAA/BBB[1] 选择第一个BBB子元素
			@   属性         选取属性
			=    内容 （值）  

 * @author APPle
 *
 */
public class Demo2 {
	public static void main(String[] args) throws Exception {
		//1）读取xml文件
		Document doc = new SAXReader().read(new File("./src/contact.xml"));
		
		//2)利用xpath方法查询xml文件
		String xpath = "";
		
		//2.1    / 
		xpath = "/contact-list"; //根标签contact-list
		xpath = "/contact-list/contact"; //contact-list根标签下的contact子标签
		
		
		//2.2   // 
		xpath = "//contact"; //选择所有contact标签（不分层次）
		xpath = "//contact/name"; //选择所有父标签是contact的name标签
		
		//2.3   *
		xpath = "/contact-list/*"; //选择根据标签contact-list下的所有子标签
		xpath = "/contact-list//*"; //选择根标签contact-list下的所有后代标签（不分层次结构）
		
		
		//2.4 [ ]
		xpath = "//contact[1]";// 第一个contact标签
		xpath = "//contact[last()]";//最后一个contact标签
		
		//2.5 @ 
		xpath = "//@id"; // 选择所有id属性
		xpath = "//contact[@id]"; //选择所有包含id属性的contact标签
		
		
		//2.6 = 
		xpath = "//contact[@id='002']"; //选择id属性值为002的contact标签
		
		//2.7 and  逻辑与
		//选取id属性为002,且name属性为eric的contact标签
		xpath = "//contact[@id='002' and @name='eric']";
		
		//2.8   text()   选取文本
		xpath = "//contact[@id='001']/name[1]/text()";//选择第一个name标签的文本
		xpath = "//name[text()='陈六']";//文本内容为”陈六“的name标签
		
		
		List<Node> list = doc.selectNodes(xpath);
		for (Node node : list) {
			System.out.println(node);
		}
		
		
		
		
		
		
	}
}

```
#### 案例1 模拟用户登录

```
package gz.itcast.a_xpath;

import java.io.BufferedReader;
import java.io.File;
import java.io.InputStreamReader;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

/**
 * 模拟用户登录
 * @author APPle
 *
 */
public class Demo3 {
	public static void main(String[] args) throws Exception {
		//1)获取用户输入的用户名和密码
		//注意： System.in是字节流，BufferedReader是字符流，字节流转字符流需要使用转换流
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		
		System.out.println("请输入用户名：");
		String name = br.readLine();
		
		System.out.println("请输入密码：");
		String password = br.readLine();
		
		
		//2)在user.xml中查询  
	//name标签文本为’rose‘,password标签文本为’123456‘的user标签
		Document doc = new SAXReader().read(new File("./src/user.xml"));
		
		//查询  文本为’xxx‘的name标签
		Element nameElem = (Element)doc.selectSingleNode("//user/name[text()='"+name+"']");
		
		//判断name标签是否存在
		if(nameElem!=null){
			//存在
			//判断密码是否正确
			Element userElem = nameElem.getParent();
			//判断password子标签的文本内容
			String dbpwd = userElem.elementText("password");
			//数据库的密码和用户输入的密码匹配
			if(password.equals(dbpwd)){
				System.out.println("登录成功");
			}else{
				//不正确
				System.out.println("密码错误，请重新输入！");
			}
		}else{
			//不存在
			System.out.println("该用户名不存在的！");
		}
	
	}
}

```
#### xpath案例2- 可以把标准的html当前xml解析

```
package gz.itcast.a_xpath;

import java.io.File;
import java.util.List;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

/**
 * 	xpath案例- 可以把标准的html当前xml解析
 * @author APPle
 *
 */
public class Demo4 {
	public static void main(String[] args) throws Exception {
		Document doc = new SAXReader().read(new File("./src/personList.html"));
		System.out.println(doc);
		
		//获取title标签文本
		Element titleElem = (Element)doc.selectSingleNode("//title");
		System.out.println(titleElem.getText());
		
		
		//获取table的内容，以下列各式输入:
		/**
		 *  编号  姓名 性别 年龄 地址 电话
		 *  1  xx  xx xx xxx  xx
		 *  2  xx xxx xx  xx xx  
		 */
		//获取tbody中所有的tr
		List<Element> trList = (List<Element>)doc.selectNodes("//tbody/tr");
		System.out.println("编号\t姓名\t性别\t年龄\t地址\t\t电话");
		for(Element e:trList){
			//获取td内容
			//((Element)e.elements("td").get(0)).getText();
			String id = e.selectSingleNode("td[1]").getText();
			String name = e.selectSingleNode("td[2]").getText();
			String gender = e.selectSingleNode("td[3]").getText();
			String age = e.selectSingleNode("td[4]").getText();
			String address = e.selectSingleNode("td[5]").getText();
			String phone = e.selectSingleNode("td[6]").getText();
			
			System.out.println(id+"\t"+name+"\t"+gender+"\t"+age+"\t"+address+"\t"+phone);
		}
		
		
		
		
		
	}
}

```


