An **XML (EXtensible Markup Language) Document** contains declarations, elements, text, and attributes. It is made up of entities (storing units) and It tells us the structure of the data it refers to. It is used to provide a standard format of data transmission. As it helps in message delivery, it is not always stored physically, i.e. in a disk but generated dynamically but its structure always remains the same.

**XML Standard structure and its rules:**

**Rule 1:** Its standard format consists of an **XML prolog** which contains both XML Declaration and XML DTD (Document Type Definition) and the body. If the XML prolog is present, it should always be the beginning of the document. The XML Version, by default, is **1.0**, and including only this forms the shortest XML Declaration. **UTF-8** is the default character encoding and is one of seven character-encoding schemes. If it is not present, it can result in some encoding errors.

**Syntax of XML Declaration:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

**Syntax of DTD:**

```xml
<!DOCTYPE root-element [<!element-declarations>]>
```

**Example:**

```xml
<!DOCTYPE website [ 
<!ELEMENT website (name,company,phone)> 
<!ELEMENT name (#PCDATA)> 
<!ELEMENT company (#PCDATA)> 
<!ELEMENT phone (#PCDATA)> 
]> 

<website> 
<name>GeeksforGeeks</name> 
<company>GeeksforGeeks</company> 
<phone>011-24567981</phone> 
</website>

```

**Rule 2:** XML Documents must have a root element (the supreme parent element) and its child elements (sub-elements). To have a better view of the hierarchy of the data elements, XML follows the XML tree structure which comprises of one single root (parent) and multiple leaves (children).
![[Screenshot22.png|center|500]]

**Source Code of the above diagram:**
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<website> 
	<company category="geeksforgeeks"> 
		<title>Machine learning</title> 
		<author>aarti majumdar</author> 
		<year>2022</year> 
	</company> 
	<company category="geeksforgeeks"> 
		<title>Web Development</title> 
		<author>aarti majumdar</author> 
		<year>2022</year> 
	</company> 
	<company category="geeksforgeekse"> 
		<title>XML</title> 
		<author>aarti majumdar</author> 
		<year>2022</year> 
	</company> 
</website>

```

**Rule 3:** All XML Elements are required to have Closing and opening Tags(similar to HTML).
```xml
<message>Welcome to GeeksforGeeks</message>
```

**Rule 4:** The opening and closing tags are case-sensitive. For Example, **\<Message>** is different from **\<message>** from above example.

**Rule 5:** Values of XML attributes are required to have quotations:
```xml
<website category="open source"> 
	<company>geeksforgeeks</company> 
</website>
```

**Rule 6:** White-Spaces are retained and maintained in XML.

```xml
<message>welcome	 to		 geeksforgeeks</message>
```

**Rule 7:** Comments can be defined in XML enclosed between _<!–_ and –> tags.

```xml
<!-- XML Comments are defined like this -->
```

**Rule 8:** XML elements must be nested properly.

```xml
<message> 
	<company>GeeksforGeeks</company> 
</message>
```