#### 内部使用dtd

```
<?xml version="1.0"?>
<!DOCTYPE note [
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend</body>
</note>
```
#### 引用外部文件

```
<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don't forget me this weekend!</body>
</note> 
```
##### note.dtd文件

```
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
```
#### 一个案例
```
<!DOCTYPE NEWSPAPER [ 

<!ELEMENT NEWSPAPER (ARTICLE+)>
<!ELEMENT ARTICLE (HEADLINE,BYLINE,LEAD,BODY,NOTES)>
<!ELEMENT HEADLINE (#PCDATA)>
<!ELEMENT BYLINE (#PCDATA)>
<!ELEMENT LEAD (#PCDATA)>
<!ELEMENT BODY (#PCDATA)>
<!ELEMENT NOTES (#PCDATA)> 

<!ATTLIST ARTICLE AUTHOR CDATA #REQUIRED>
<!ATTLIST ARTICLE EDITOR CDATA #IMPLIED>
<!ATTLIST ARTICLE DATE CDATA #IMPLIED>
<!ATTLIST ARTICLE EDITION CDATA #IMPLIED>

]>

<NEWSPAPER>
	<ARTICLE AUTHOR="123" EDITOR="123">
		<HEADLINE>abc</HEADLINE>
		<BYLINE>1212</BYLINE>
		<LEAD>dfdf</LEAD>
		<BODY>343</BODY>
		<NOTES>sdsdf</NOTES>
	</ARTICLE>
	<ARTICLE AUTHOR="123">
		<HEADLINE>abc</HEADLINE>
		<BYLINE>1212</BYLINE>
		<LEAD>dfdf</LEAD>
		<BODY>343</BODY>
		<NOTES>sdsdf</NOTES>
	</ARTICLE>
</NEWSPAPER>


```

