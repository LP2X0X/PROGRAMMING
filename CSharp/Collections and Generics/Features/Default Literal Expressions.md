- In addition to setting the default value of a property, C# 7.1 introduced default literal expressions. This eliminates the need for specifying the type of the variable in the default statement.
```csharp
public void ResetPoint()  
{  
	_xPos = default;  
	_yPos = default;  
}
```