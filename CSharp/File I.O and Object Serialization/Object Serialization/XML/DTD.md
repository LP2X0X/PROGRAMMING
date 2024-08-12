A Document Type Definition (DTD) is a set of rules and specifications used to define the structure, elements, and attributes of an XML document. It ensures that the data adheres to a specific format and is both valid and well-formed. Here are the key components of a DTD:

1. **Element Declarations**: Define the allowed elements in the XML document and their content model (what elements or text can appear within them).
   ```xml
   <!ELEMENT note (to, from, heading, body)>
   ```

2. **Attribute Declarations**: Define the attributes that can be associated with elements and their data types.
   ```xml
   <!ATTLIST note date CDATA #REQUIRED>
   ```

3. **Entities**: Define reusable pieces of text or symbols.
   ```xml
   <!ENTITY author "John Doe">
   ```

4. **Notations**: Provide information about non-XML data within the XML document.
   ```xml
   <!NOTATION gif SYSTEM "image/gif">
   ```

5. **Comments**: Add notes or explanations within the DTD that are not part of the actual markup.
   ```xml
   <!-- This is a comment -->
   ```

DTDs can be either internal or external:

- **Internal DTD**: Defined within the XML document itself.
  ```xml
  <!DOCTYPE note [
      <!ELEMENT note (to, from, heading, body)>
      <!ELEMENT to (#PCDATA)>
      <!ELEMENT from (#PCDATA)>
      <!ELEMENT heading (#PCDATA)>
      <!ELEMENT body (#PCDATA)>
  ]>
  <note>
      <to>Tove</to>
      <from>Jani</from>
      <heading>Reminder</heading>
      <body>Don't forget me this weekend!</body>
  </note>
  ```

- **External DTD**: Defined in a separate file and referenced from the XML document.
  ```xml
  <!DOCTYPE note SYSTEM "note.dtd">
  <note>
      <to>Tove</to>
      <from>Jani</from>
      <heading>Reminder</heading>
      <body>Don't forget me this weekend!</body>
  </note>
  ```

The primary purpose of a DTD is to validate the XML document's structure and content, ensuring it meets the predefined rules.