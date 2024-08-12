- C# 6 introduced a new clause that can be placed on a catch scope, via the when keyword. When you add this clause, you have the ability to ensure that the statements within a catch block are executed only if some condition in your code holds true. This expression must evaluate to a Boolean (true or false) and can be obtained by using a simple code statement in the when definition itself or by calling an additional method in your code. In a nutshell, this approach allows you to add “filters” to your exception logic.
```csharp
catch (CarIsDeadException e) when (e.ErrorTimeStamp.DayOfWeek != DayOfWeek.Friday)  
{  
	// This new line will only print if the when clause evaluates to true.  
	Console.WriteLine("Catching car is dead!");  
	Console.WriteLine(e.Message);  
}
```