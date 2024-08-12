- The Data property of System.Exception allows you to fill an exception object with relevant auxiliary information (such as a timestamp). The Data property returns an object implementing an interface named IDictionary, defined in the System.Collections namespace.
```csharp
throw new Exception($"{PetName} has overheated!")  
{  
	HelpLink = "http://www.CarsRUs.com",  
	Data = {  
		{"TimeStamp",$"The car exploded at {DateTime.Now}"},  
		{"Cause","You have a lead foot."}  
	}  
};  
```