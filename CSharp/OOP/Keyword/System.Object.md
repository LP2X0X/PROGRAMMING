- When you do build a class that does not explicitly define its parent, the compiler automatically derives your type from Object.
- Like any class, System.Object defines a set of members. In the following formal C# definition, note that some of these items are declared virtual, which specifies that a given member may be overridden by a subclass, while others are marked with static (and are therefore called at the class level):
```csharp
public class Object  
{  
	// Virtual members.  
	public virtual bool Equals(object obj);  
	protected virtual void Finalize();  
	public virtual int GetHashCode();  
	public virtual string ToString();  
	// Instance-level, nonvirtual members.  
	public Type GetType();  
	protected object MemberwiseClone();  
	// Static members.  
	public static bool Equals(object objA, object objB);  
	public static bool ReferenceEquals(object objA, object objB);  
}
```

| Instance Method of Object Class | Meaning in Life                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Equals()                        | By default, this method returns true only if the items being compared refer to the same item in memory. Thus, Equals() is used to compare object references, not the state of the object. Typically, this method is overridden to return true only if the objects being compared have the same internal state values (i.e., value-based semantics). Be aware that if you override Equals(), you should also override GetHashCode(), as these methods are used internally by Hashtable types to retrieve subobjects from the container. Also recall that the ValueType class overrides this method for all structures, so they work with value-based comparisons. |
| Finalize()                      | This method (when overridden) is called to free any allocated resources before the object is destroyed.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| GetHashCode()                   | This method returns an int that identifies a specific object instance.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ToString()                      | This method returns a string representation of this object, using the \<namespace\>.\<type name\> format (termed the fully qualified name). This method will often be overridden by a subclass to return a tokenized string of name-value pairs that represent the object’s internal state, rather than its fully qualified name.                                                                                                                                                                                                                                                                                                                                |
| GetType()                       | This method returns a Type object that fully describes the object you are currently referencing. In short, this is a runtime type identification (RTTI) method available to all objects                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| MemberwiseClone()               |              This method exists to return a member-by-member copy of the current object, which is often used when cloning an object.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

---

## Overriding System.Object.ToString()
Many classes (and structures) that you create can benefit from overriding ToString() to return a string textual representation of the type’s current state. This can be quite helpful for purposes of debugging (among other reasons). How you choose to construct this string is a matter of personal choice; however, a recommended approach is to separate each name-value pair with semicolons and wrap the entire string within square brackets (many types in the .NET Core base class libraries follow this approach). Consider the following overridden ToString() for your Person class:  
```csharp
public override string ToString() => $"[First Name: {FirstName}; Last Name: {LastName}; Age: {Age}]";
```

---

## Overriding System.Object.Equals()
- Let’s also override the behavior of Object.Equals() to work with value-based semantics. Recall that, by default, Equals() returns true only if the two objects being compared reference the same object instance in memory.
- First, notice that the incoming argument of the Equals() method is a general System.Object. Given this, your first order of business is to ensure the caller has indeed passed in a Person object and, as an extra safeguard, to make sure the incoming parameter is not a null reference.
- After you have established the caller has passed you an allocated Person, one approach to implement Equals() is to perform a field-by-field comparison against the data of the incoming object to the data of the current object.
```csharp
public override bool Equals(object obj)  
{  
	if (!(obj is Person temp))  
	{  
	return false;  
	}  
		if (temp.FirstName == this.FirstName  
		&& temp.LastName == this.LastName  
		&& temp.Age == this.Age)  
	{  
		return true;  
	}  
	return false;  
}
```

- While this approach does indeed work, you can certainly imagine how labor intensive it would be to implement a custom Equals() method for nontrivial types that may contain dozens of data fields. One common shortcut is to leverage your own implementation of ToString(). If a class has a prim-and-proper implementation of ToString() that accounts for all field data up the chain of inheritance, you can simply perform a comparison of the object’s string data (checking for null)
```csharp
// No need to cast "obj" to a Person anymore,  
// as everything has a ToString() method.  
public override bool Equals(object obj)  
	=> obj?.ToString() == ToString();
```

---

## Overriding System.Object.GetHashCode()
- When a class overrides the Equals() method, you should also override the default implementation of GetHashCode(). Simply put, a hash code is a numerical value that represents an object as a particular state. For example, if you create two string variables that hold the value Hello, you will obtain the same hash code. However, if one of the string objects were in all lowercase (hello), you would obtain different hash codes.
- By default, System.Object.GetHashCode() uses your object’s current location in memory to yield the hash value. However, if you are building a custom type that you intend to store in a Hashtable type (within the System.Collections namespace), you should always override this member, as the Hashtable will be internally invoking Equals() and GetHashCode() to retrieve the correct object.
```ad-note
To be more specific, the System.Collections.Hashtable class calls GetHashCode() internally to gain a general idea where the object is located, but a subsequent (internal) call to Equals() determines the exact match.
```