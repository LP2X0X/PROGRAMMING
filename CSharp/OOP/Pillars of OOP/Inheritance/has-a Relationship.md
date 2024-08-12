- In object-oriented programming, a "has-a" relationship (also known as the containment/delegation model or aggregation) is used to describe a composition relationship between two classes, where one class contains an instance of another class. This indicates that the containing class "has a" member of the contained class. In C#, this is typically implemented by including an instance of one class as a field or property in another class.

Here's a simple example to illustrate the "has-a" relationship in C#:

### Example Scenario

Let's say we have a `Car` class and an `Engine` class. A car "has an" engine, so we can model this relationship by including an `Engine` instance inside the `Car` class.

### Class Definitions

1. **Engine Class**:
    ```csharp
    public class Engine
    {
        public string Model { get; set; }
        public int Horsepower { get; set; }

        public Engine(string model, int horsepower)
        {
            Model = model;
            Horsepower = horsepower;
        }

        public void Start()
        {
            Console.WriteLine("Engine started.");
        }
    }
    ```

2. **Car Class**:
    ```csharp
    public class Car
    {
        public string Make { get; set; }
        public string Model { get; set; }
        private Engine _engine; // Has-a relationship

        public Car(string make, string model, Engine engine)
        {
            Make = make;
            Model = model;
            _engine = engine;
        }

        public void StartCar()
        {
            Console.WriteLine($"Starting the {Make} {Model}.");
            _engine.Start();
        }
    }
    ```

### Usage

Here's how you might use these classes together:

```csharp
class Program
{
    static void Main()
    {
        Engine engine = new Engine("V8", 500);
        Car car = new Car("Ford", "Mustang", engine);

        car.StartCar();
    }
}
```

### Explanation

1. **Engine Class**:
    - The `Engine` class has properties `Model` and `Horsepower`.
    - It also has a method `Start` that prints a message to the console.

2. **Car Class**:
    - The `Car` class has properties `Make` and `Model`.
    - It has a private field `_engine` which is an instance of the `Engine` class. This represents the "has-a" relationship.
    - The constructor of `Car` takes an `Engine` instance as a parameter and assigns it to `_engine`.
    - The `StartCar` method calls the `Start` method of the `Engine` instance, demonstrating the composition relationship.

### Key Points

- **Encapsulation**: The `Engine` instance is a private member of the `Car` class, meaning it is encapsulated within the `Car` class.
- **Composition**: The `Car` class is composed of an `Engine` class, indicating a strong "has-a" relationship where the `Car` cannot function without an `Engine`.

This example demonstrates how you can use composition to model a "has-a" relationship in C#.

---
- Delegation is simply the act of adding public members to the containing class that use the contained object’s functionality.
```csharp
partial class Employee  
{  
	// Contain a BenefitPackage object.  
	protected BenefitPackage EmpBenefits = new BenefitPackage();  
	// Expose certain benefit behaviors of object.  
	public double GetBenefitCost()  
		=> EmpBenefits.ComputePayDeduction();  
	// Expose object through a custom property.  
	public BenefitPackage Benefits  
	{  
		get { return EmpBenefits; }  
		set { EmpBenefits = value; }  
	}  
}
```