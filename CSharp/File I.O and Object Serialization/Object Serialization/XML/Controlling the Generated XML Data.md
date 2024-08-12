- If you have a background in XML technologies, you know that it is often critical to ensure the data within an XML document conforms to a set of rules that establishes the validity of the data. Understand that a valid XML document does not have anything to do with the syntactic well-being of the XML elements (e.g., all opening elements must have a closing element). Rather, valid documents conform to agreed-upon formatting rules (e.g., field X must be expressed as an attribute and not a sub-element), which are typically defined by an XML schema or document-type definition (DTD) file.
- By default, XmlSerializer serializes all public fields/properties as XML elements, rather than as XML attributes. If you want to control how the XmlSerializer generates the resulting XML document, you can decorate types with any number of additional .NET attributes from the System.Xml.Serialization namespace.
![[Pasted image 20240724102805.png|center|700]]

```ad-note
The XmlSerializer demands that all serialized types in the object graph support a default constructor (so be sure to add it back if you define custom constructors).
```