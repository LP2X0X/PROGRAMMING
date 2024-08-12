- The first pillar of OOP is called encapsulation. This trait boils down to the language’s ability to hide unnecessary implementation details from the object user. For example, assume you are using a class named DatabaseReader, which has two primary methods, named Open() and Close().  
```csharp
// Assume this class encapsulates the details of opening and closing a database.  
DatabaseReader dbReader = new DatabaseReader();  
dbReader.Open(@"C:\AutoLot.mdf");  
// Do something with data file and close the file.  
dbReader.Close();  
```
- The fictitious DatabaseReader class encapsulates the inner details of locating, loading, manipulating, and closing a data file. Programmers love encapsulation, as this pillar of OOP keeps coding tasks simpler. There is no need to worry about the numerous lines of code that are working behind the scenes to carry out the work of the DatabaseReader class. All you do is create an instance and send the appropriate messages (e.g., “Open the file named AutoLot.mdf located on my C drive”).  
- Closely related to the notion of encapsulating programming logic is the idea of data protection. Ideally, an object’s state data should be specified using either the private, internal, or protected keyword. In this way, the outside world must ask politely in order to change or obtain the underlying value. This is a good thing, as publicly declared data points can easily become corrupted (ideally by accident rather than intent!).