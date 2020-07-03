# 读xml内容
#### 读取xml的标签
```
package gz.itcast.a_dom4j_read;

import java.io.File;
import java.util.List;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

/**
 * 使用dom4j读取xml的标签
 *      getRootElement(): 获取根标签
 *      getName(): 获取标签名称
 *      element("名称")： 获取第一个指定名称的子标签
 *      elements("名称"): 获取指定名称的所有的子标签
 *      elements(): 获取所有子标签
 * 
 * @author APPle
 *
 */
public class Demo2 {
	public static void main(String[] args) throws Exception {
		//1)创建xml解析器对象
		SAXReader reader = new SAXReader(); // ctrl+2 放手  +l
		//2)读取xml文件
		Document doc = reader.read(new File("./src/contact.xml"));
		
		/**
		 * 读取标签
		 */
		//1.1 读取根标签
		Element rootElem = doc.getRootElement();
		System.out.println(rootElem);
		//1.2 获取标签名称
		System.out.println(rootElem.getName());
		
		//1.3 获取第一个子标签(根据指定的名称获取第一个子标签)
		Element conElem = rootElem.element("contact");
		System.out.println(conElem);
		
		System.out.println("==============");
		//1.4 获取所有子标签（根据指定的名称获取所有同名子标签）
		List<Element> list = rootElem.elements("contact");
		//遍历List
		//几种方式?
		//1)传统for循环
		/*for(int i=0;i<list.size();i++){
			list.get(i); //根据角标获取指定对象
		}*/
		
		//2)for-each循环
		for(Element e: list){
			System.out.println(e);
		}
		
		//3)迭代器
		/*Iterator it = list.iterator();
		while(it.hasNext()){ //hasNext(): 判断是否有下一个元素
			it.next(); //next():取出当前对象
		}*/
		
		System.out.println("================");
		
		//1.4 获取所有子标签（不指定名称）
		List<Element> eList = rootElem.elements();
		for(Element e:eList){
			System.out.println(e);
		}
		System.out.println("===========");
		
		/**
		 * 注意，如果需要获取孙标签，首先得拿到子标签，再从子标签来获取孙标签！！！
		 */
		Element nameElem  = rootElem.element("contact").element("name");
		System.out.println(nameElem);
		
		
		
	}
}

```
#### 获取xml上的属性信息

```
package gz.itcast.a_dom4j_read;

import java.io.File;
import java.util.List;

import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

/**
 * 使用dom4j获取xml上的属性信息
 * @author APPle
 *
 */
public class Demo3 {

	public static void main(String[] args) throws Exception {
		//1)创建xml解析器
		SAXReader reader = new SAXReader();
		Document doc = reader.read(new File("./src/contact.xml"));
		
		/**
		 * 读取属性
		 * 注意：获取属性，必须先得到属性所在的标签
		 */
		Element conElem = doc.getRootElement().element("contact");
		//1.1 在标签上获取属性值(根据属性名称获取对应的属性值)
		String value = conElem.attributeValue("id");
		System.out.println(value);
		/**
		 * 练习： 拿到id=002属性
		 */
		Element conElem2 = (Element)doc.getRootElement().elements().get(1);
		System.out.println(conElem2.attributeValue("id"));
		
		//1.2 根据属性名称获取属性对象
		//拿到标签对象
		conElem = doc.getRootElement().element("contact");
		//拿到属性对象
		Attribute idAttr = conElem.attribute("id");
		//通过属性对象拿到 属性名
		String idName = idAttr.getName();
		//通过属性对象拿到 属性值
		String idValue = idAttr.getValue();
		System.out.println(idName+"="+idValue);
		
		System.out.println("===============");
		
		//1.3 获取标签的所有属性对象
		conElem = doc.getRootElement().element("contact");
		List<Attribute> attrList = conElem.attributes();
		for (Attribute attr : attrList) {
			System.out.println(attr.getName()+"="+attr.getValue());
		}
	}

}

```

#### 获取xml的文本信息

```
package gz.itcast.a_dom4j_read;

import java.io.File;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

/**
 * 使用dom4j获取xml的文本信息
 * @author APPle
 *
 */
public class Demo4 {
	public static void main(String[] args) throws Exception {
		SAXReader reader = new SAXReader();
		Document doc = reader.read(new File("./src/contact.xml"));
		
		/**
		 * 注意：
		 *     在xml文件中，空格和换行会作为xml的内容被解析到。
		 *     xml中空格和换行和java代码中空格换行不一样。
		 *     java代码中的空格和换行是没意义的，为了代码的格式格式好看而已。
		 */
		Element con = doc.getRootElement().element("contact");
		System.out.println(con.getText());
		
		
		/**
		 * 读取文本：
		 * 	注意： 获取文本，要先获取文本所在的标签对象
		 */
		//1.1 拿到所在标签上的文本内容
		Element nameElem = doc.getRootElement().
						element("contact").element("name");
		String content = nameElem.getText();
		System.out.println(content);
		
		
		//1.2 通过父标签获取指定子标签的文本内容
		Element conElem = doc.getRootElement().element("contact");
	    content = conElem.elementText("gender");
	    System.out.println(content);
	}
}

```
#### 把xml文件的信息封装成对象

```
package gz.itcast.a_dom4j_read;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

/**
 * 使用dom4j把xml文件的信息封装成对象
 * @author APPle
 *
 */
public class Demo5 {

	public static void main(String[] args) throws Exception {
		//目标： contact.xml信息 -> List集合
		//1）读取xml文件
		SAXReader reader = new SAXReader();
		Document doc = reader.read(new File("./src/contact.xml"));
		
		//2)创建List对象
		List<Contact> list = new ArrayList<Contact>(); // List接口-》 ArrayList/LinkedList/Vector
		
		
		//3)把xml信息->list对象
		//3.1 读取到所有contact标签
		List<Element> conList = doc.getRootElement().elements("contact");
		for (Element elem : conList) {
			//3.2 创建Contact对象
			Contact con = new Contact();
			
			//3.3 把contact标签数据放入contact对象中
			con.setId(elem.attributeValue("id"));
			con.setName(elem.elementText("name"));
			con.setGender(elem.elementText("gender"));
			con.setPhone(elem.elementText("phone"));
			con.setEmail(elem.elementText("email"));
			con.setAddress(elem.elementText("address"));
			
			//3.4 把contact对象放入list对象
			//保存数据   list.add(对象)
			list.add(con);
		}
		
		//4)输出
		for (Contact con : list) {
			System.out.println(con);
		}
	}
}

```
# 写xml内容
#### 写出一个xml文件
```
package gz.itcast.b_dom4j_write;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;

import org.dom4j.Document;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;

/**
 * 写出一个xml文件
 * @author APPle
 *
 */
public class Demo1 {
	public static void main(String[] args) throws Exception {
		
		/**
		 * 修改xml信息的步骤
		 * 1）读取到原来的xml文件（document对象）
		 * 2）操作document对象，改变xml信息（docuement对象）
		 * 3）把修改后的document对象写出到xml文件中（覆盖原来的文件）
		 */
		
		
		Document doc = new SAXReader().read(new File("./src/contact.xml"));
		/**
		 * 输出流
		 * 	  字符输出流： 
		 * 		 Writer ->  FileWriter/BufferedWriter
		 * 				方法：
		 * 					write(char c)： 写出一个字符
		 * 					write(char[] data): 写出多个字符
		 * 					write(String str): 写出一个字符串  
		 * 	  字节输出流
		 * 		 OutputStream -> FileOutputStream/BufferedOutputStream/ObjectOutputStream
		 * 					write(byte) :写出一个字节
		 * 					write(byte[] data): 写出多个字节
		 */
		/**
		 * 把内存的document对象写出到硬盘的xml文件
		 */
		//创建输出流
		OutputStream outStream = new FileOutputStream("e:/contact.xml");
		//1)创建输出对象
		XMLWriter writer = new XMLWriter(outStream);
		//2)写出数据
		writer.write(doc);
	}
}

```
#### 写出xml文件的细节
```
package gz.itcast.b_dom4j_write;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;

import org.dom4j.Document;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;

/**
 * 写出xml文件的细节
 * @author APPle
 *
 */
public class Demo2 {
	public static void main(String[] args) throws Exception {
		Document doc = new SAXReader().read(new File("./src/contact.xml"));
		
		//创建输出流
		OutputStream outStream = new FileOutputStream("e:/contact.xml");
		//一、设置输出的格式
		//OutputFormat format = OutputFormat.createCompactFormat();//紧凑的格式.空格和换行去掉了！！ 系统上线了使用
		OutputFormat format = OutputFormat.createPrettyPrint();//漂亮的格式。包含空格和换行。 测试时使用
		
		//二、 设置输出的编码格式
		/**作用：
		 * 1)影响了xml的文档声明的encoding编码
		 * 2)影响了xml内容保存的编码
		 */
		format.setEncoding("gbk");
		
		//1)创建输出对象
		XMLWriter writer = new XMLWriter(outStream,format);
		//2)写出数据
		writer.write(doc);
	}
}

```
#### 修改xml文件
```
package gz.itcast.b_dom4j_write;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.io.UnsupportedEncodingException;

import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.DocumentHelper;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;

/**
 * 修改xml文件：
 * 		添加： 文档   标签   属性   
 * 		修改： 属性值，文本内容
 * 		删除： 标签  属性 
 * @author APPle
 *
 */
public class Demo3 {
	public static void main(String[] args) throws  Exception {
		//add();		
		
		//edit();
		
		
		Document doc = new SAXReader().read(new File("./src/contact.xml"));
		/**
		 * 删除
		 */
		/*//1.1 删除标签
		Element conElem = doc.getRootElement().element("contact");
		//conElem.detach(); //自杀
		conElem.getParent().remove(conElem); //他杀
*/		

		//1.2 删除属性
		Attribute idAttr = doc.getRootElement().element("contact").attribute("id");
		idAttr.detach();
		
		
		//1.2 把文档写出到xml文件中
		OutputStream out = new FileOutputStream("e:/contact.xml");
		OutputFormat format = OutputFormat.createPrettyPrint();
		format.setEncoding("utf-8");
		
		XMLWriter writer = new XMLWriter(out,format);
		writer.write(doc);
	}

	private static void edit() throws DocumentException, FileNotFoundException,
			UnsupportedEncodingException, IOException {
		Document doc = new SAXReader().read(new File("./src/contact.xml"));
		/**
		 * 修改
		 */
		//修改属性
		//1.1 先得到属性对象，再调用方法修改属性值
		/*Element conElem = doc.getRootElement().element("contact");
		Attribute idAttr = conElem.attribute("id");
		idAttr.setValue("003");*/
		
		
		//1.2 在标签中添加同名的属性，覆盖属性值
		Element conElem = doc.getRootElement().element("contact");
		conElem.addAttribute("id", "004");
		
		//修改文本
		Element nameElem = doc.getRootElement().element("contact").element("name");
		nameElem.setText("王五");

		
		//1.2 把文档写出到xml文件中
		OutputStream out = new FileOutputStream("e:/contact.xml");
		OutputFormat format = OutputFormat.createPrettyPrint();
		format.setEncoding("utf-8");
		
		XMLWriter writer = new XMLWriter(out,format);
		writer.write(doc);
	}

	/**
	 * 添加
	 * @throws FileNotFoundException
	 * @throws UnsupportedEncodingException
	 * @throws IOException
	 */
	private static void add() throws FileNotFoundException,
			UnsupportedEncodingException, IOException {
		/**
		 * 添加
		 */
		//1.1 添加空文档
		Document doc = DocumentHelper.createDocument();
		
		//1.2 添加标签
		Element conListElem = doc.addElement("contact-list");
		//doc.addElement("contact-list"); //不能添加两个根标签！！！

		Element conElem = conListElem.addElement("contact");
		conElem.addElement("name");
		
		//1.3 添加属性
		conElem.addAttribute("id", "001");
		conElem.addAttribute("name","eric");
		
		//1.2 把文档写出到xml文件中
		OutputStream out = new FileOutputStream("e:/contact.xml");
		OutputFormat format = OutputFormat.createPrettyPrint();
		format.setEncoding("utf-8");
		
		XMLWriter writer = new XMLWriter(out,format);
		writer.write(doc);
	}
}

```


