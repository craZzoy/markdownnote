#### sax解析入门

```
package gz.itcast.b_sax;

import java.io.File;

import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

import org.xml.sax.SAXException;

/**
 * sax解析入门
 * @author APPle
 *
 */
public class Demo1 {

		public static void main(String[] args) throws Exception, SAXException {
			//1)创建SAXParser解析对象
			SAXParser parser = SAXParserFactory.newInstance().newSAXParser();
			
			//2)解析xml文件
			/**
			 * 参数一： 需要解析的xml文件
			 * 参数二： 指定的DefaultHandler
			 */
			/**
			 * 事件编程模式三要求：
			 *   事件源：xml文件 
			 *   事件：解析到开始标签（包含属性），解析到结束标签，解析文本内容
			 *   监听器：DefaultHandler
			 */
			//类似于注册监听器
			parser.parse(new File("./src/contact.xml"), new MyDefaultHandler1());
			
		}
}

```

```
package gz.itcast.b_sax;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;
/**
 * sax解析事件处理程序（类似于事件的监听器）
 * @author APPle
 *
 */
public class MyDefaultHandler1 extends DefaultHandler{
	
	/**
	 * 遇到xml文档的开始位置触发此方法
	 */
	@Override
	public void startDocument() throws SAXException {
		System.out.println("MyDefaultHandler1.startDocument()");
	}
	
	

	/**
	 * 遇到每个开始标签触发次方法
	 * @param qName: 表示当前读到的开始标签名称
	 * @param attributes : 属性列表
	 */
	@Override
	public void startElement(String uri, String localName, String qName,
			Attributes attributes) throws SAXException {
		System.out.println("MyDefaultHandler1.startElement()->"+qName);
	}
	
	/**
	 * 遇到每个结束标签时触发此方法
	 * @param qName: 当前读到的结束标签名称
	 */
	@Override
	public void endElement(String uri, String localName, String qName)
			throws SAXException {
		System.out.println("MyDefaultHandler1.endElement()->"+qName);
	}
	
	/**
	 * 遇到文本内容触发此方法
	 * 如何获取当前读到的内容？
	 *   char[]: 表示到目前为止读到的文本内容
	 *   start: 表示当前内容的起始位置
	 * 	 length: 表示当前内容的长度
	 */
	@Override
	public void characters(char[] ch, int start, int length)
			throws SAXException {
		/**
		 * char[]内容：              张三               男                 1341111122222
		 */               
		//获取当前读到的内容
		String content = new String(ch,start,length);
		System.out.println("MyDefaultHandler1.characters()->"+content);
	}
	
	
	/**
	 * 遇到xml文档 的结尾
	 */
	@Override
	public void endDocument() throws SAXException {
		System.out.println("MyDefaultHandler1.endDocument()");
	}

	
}

```
#### 使用sax解析方式读取contact.xml文件内
```
package gz.itcast.b_sax;

import java.io.File;

import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

import org.xml.sax.SAXException;

/**
 * 使用sax解析方式读取contact.xml文件内容，原封不动打印文件信息
 * @author APPle
 *
 */
public class Demo2 {

	public static void main(String[] args) throws Exception, SAXException {
		//1)创建SAXParser对象
		SAXParser parser = SAXParserFactory.newInstance().newSAXParser();
		
		//创建事件处理程序
		MyDefaultHandler2 handler2 = new MyDefaultHandler2();
		
		//2)读取xml文件
		parser.parse(new File("./src/contact.xml"), handler2);
		
		System.out.println(handler2.getContent());
	}
}

```

```
package gz.itcast.b_sax;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;

/**
 * sax事件处理程序
 * @author APPle
 *
 */
public class MyDefaultHandler2 extends DefaultHandler{
	//存储contact.xml文件信息
	//当contact.xml读取完毕之后，这个变量就有了所有xml文件信息
	private StringBuffer sb = new StringBuffer();
	
	public String getContent(){
		return sb.toString();
	}
	
	//开始标签
	/**
	 * qName:开始标签的名称
	 * attributes： 属性列表
	 */
	@Override
	public void startElement(String uri, String localName, String qName,
			Attributes attributes) throws SAXException {
		sb.append("<"+qName);
		//属性列表
		if(attributes!=null){
			//遍历属性
			for(int i=0;i<attributes.getLength();i++){
				String name = attributes.getQName(i);//属性名称
				String value = attributes.getValue(i);//属性值
				sb.append(" "+name+"=\""+value+"\"");
			}
		}
		sb.append(">");
	}
	
	//文本内容
	@Override
	public void characters(char[] ch, int start, int length)
			throws SAXException {
		//当前文本内容
		String content = new String(ch,start,length);
		sb.append(content);
	}
	
	//结束标签
	//qName: 结束标签名称
	@Override
	public void endElement(String uri, String localName, String qName)
			throws SAXException {
		sb.append("</"+qName+">");
	}
}

```
#### 使用sax解析把contact.xml文件信息封装成List对象
```
package gz.itcast.b_sax;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

import org.xml.sax.SAXException;

/**
 * 使用sax解析把contact.xml文件信息封装成List对象
 * @author APPle
 *
 */
public class Demo3 {
	public static void main(String[] args) throws Exception, SAXException {
		//1)读取contact.xml文件
		SAXParser parser = SAXParserFactory.newInstance().newSAXParser();
		//创建事件处理程序
		MyDefaultHandler3 handler3 = new MyDefaultHandler3();
		//解析xml文件
		parser.parse(new File("./src/contact.xml"), handler3);
		//获取到封装好的List对象
		List<Contact> conList = handler3.getList();
		//打印
		for (Contact contact : conList) {
			System.out.println(contact);
		}
		
		
	}
}

```

```
package gz.itcast.b_sax;

import java.util.ArrayList;
import java.util.List;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;
/**
 * sax事件处理程序
 * @author APPle
 *
 */
public class MyDefaultHandler3 extends DefaultHandler{
	//用于存储所有Contact对象信息
	private List<Contact> conList = new ArrayList<Contact>();
	//用于存储一个contact标签中的信息 
	private Contact contact = null;
	//返回封装好的List对象
	public List<Contact> getList(){
		return conList;
	}
	
	//用于记录当前标签是哪个
	private String curTag;
	
	/**
	 * 思路：
	 * 	1）创建一个新的Contact对象，用于封装contact标签信息
	 *  2) 把当前读到的这个contact标签的信息封装到Contact对象中
	 *  3）把封装好的Contact对象放入List中
	 */
	
	//开始标签
	@Override
	public void startElement(String uri, String localName, String qName,
			Attributes attributes) throws SAXException {
		curTag = qName;
		//1)在读到contact的开始标签时创建Contact对象
		if(qName.equals("contact")){
			//创建Contact对象
			contact = new Contact();
			//封装id属性
			String id = attributes.getValue("id");
			contact.setId(id);
		}
	}
	//文本内容(注意：包含换行和空格)
	@Override
	public void characters(char[] ch, int start, int length)
			throws SAXException {
		//2)把contact子标签中的文本内容封装到Contact对象中
		String content = new String(ch,start,length);
		
		//如果当前标签是name
		if("name".equals(curTag)){
			contact.setName(content);
		}
		//如果当前标签是phone
		if("phone".equals(curTag)){
			contact.setPhone(content);
		}
		//如果当前标签是email
		if("email".equals(curTag)){
			contact.setEmail(content);
		}
		//如果当前标签是address
		if("address".equals(curTag)){
			contact.setAddress(content);
		}
		//如果当前标签是gender
		if("gender".equals(curTag)){
			contact.setGender(content);
		}
		
	}
	//结束标签
	@Override
	public void endElement(String uri, String localName, String qName)
			throws SAXException {
		//把curTag置空
		curTag = null;
		//3)在读到contact结束标签时把Contact对象放入List中
		if(qName.equals("contact")){
			conList.add(contact);
		}
	}
	
}

```

