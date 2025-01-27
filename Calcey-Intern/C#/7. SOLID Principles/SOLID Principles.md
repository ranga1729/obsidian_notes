![[Pasted image 20240620114223.png]]
The SOLID principles are a set of five design principles intended to make software more understandable, flexible and maintainable. 
These are,
1. Single Responsibility Principle(SRP)
2. Open/Closed Principles(OCP)
3. Liskov Substitution Principle(LSP)
4. Interface Segregation Principle(ISP)
5. Dependency Inversion Principle(DIP)

****
##### 1. Single Responsibility Principle(SRP)
==*A class should have only one job responsibility.*==
This principles helps to reduce the complexity of a class by ensuring it does not try to handle too many concerns at once. 
##### 2. Open/Closed Principles(OCP)
==*Software entities(classes, modules, functions, ...etc.) should be open for extension but closed for modification.*==
This means that the behavior of a module can be extended without modifying its source code, often achieved through polymorphism and abstraction. 

Violating OCP
```C#
public class AreaCalcualator
{
	public double CalculateArea(object shape)
	{
		if(shape is circle)
		{
			Circle circle = (circle)shape;
			return Math.PI * circle.Radius * circle.Radius;
		}
		else if(shape is Rectangle)
		{
			Rectangle rectangle = (Rectangle)shpe;
			return rectangle.width * rectangle.width;
		}
	}
}
```

Following OCP
```C#
public interface Shape
{
	public double CalculateArea();
}

public class Circle : Shape
{
	public double Radius { get; set; } 
	public override double CalculateArea() 
	{ 
		return Math.PI * Radius * Radius; 
	}
}

public class Rectangle : Shape 
{ 
	public double Width { get; set; } 
	public double Height { get; set; } 
	public double CalculateArea() 
	{ 
		return Width * Height; 
	} 
}
```
In this code,
- We are open for extend the Shape interface into real shapes. 
- But we don't modify the area method based on the shape. Just extending and implementing the methods. 
##### 3. Liskov Substitution Principle(LSP)
==*Object of a super class should be replaceable with objects of a subclass without effecting the correctness of the program.*==
This ensures that a subclass can stand in for its parent class without altering the desired behavior. 
> [!NOTE] In other words
> You should use inheritance only when your superclass is replaceable by a subclass in all the instances.

Ex:
Violating LSP
```C#
public interface Bird
{
	public void Fly();
}

public class Ostrich : Bird
{
	publi void Fly()
	{
		//ostrich ca't fly.
	}
}
```
Following LSP
```C#
public abstract class Bird
{
	public abstract void Move();
}

public class Sparrow : Bird
{
	public override void Move()
	{
		Fly();
	}
	private void Fly()
	{
		//Fly implementation.
	}
}

public class Ostrich : Bird
{
	public override void Move()
	{
		Run();
	}
	private void Run()
	{
		//Run implementation.
	}
}
```
In this example,
- The interface doesn't have a Fly method but a Move method.
- Move can be then modified as we want.(as Fly or Run)
##### 4. Interface Segregation Principle(ISP)
==*Client should not be forced to depend on interfaces they do not use.*==
This principle suggest creating smaller, more specific interfaces rather than a large, general-purpose one, ensuring that implementing classes are not burdened with unnecessary methods.
Ex:
Violating ISP
```C#
public interface IWorker
{
	void Work();
	void Eat();
}
public class Worker : IWorker
{
	public void Work()
	{
		//work implementation
	}
	public class Eat();
	{
		//Eat implementation
	}
}
public class Robot : IWorker
{
	public void Work()
	{
		//work implementation
	}
	public class Eat();
	{
		throw new NotImplementedException(); //robpots don'r eat
	}
}
```
Following ISP
```C#
public interface IWorkable 
{ 
	void Work(); 
} 
public interface IEatable 
{ 
	void Eat(); 
}

public class Worker : IWorkable, IEatable 
{ 
	public void Work() 
	{ 
		// Working implementation 
	} public void Eat() 
	{ 
		// Eating implementation 
	} 
}
public class Robot : IWorkable 
{ 
	public void Work() 
	{ 
		// Working implementation 
	} 
}
```
##### 5. Dependency Inversion Principle(DIP)
==*High-level modules should not depend on low-level modules. Both should depend on abstractions (e.g., interfaces).==
==Additionally, abstractions should not depend on details; details should depend on abstractions.*==
This principle helps to decouple different pieces of code, making it easier to change one part without affecting others.

Ex: Using interfaces to decouple high-level and low-level modules. 
Violating DIP
```C#
public class LightBulb
{
	public void TurnOn()
	{
		//Turn on the light bulb
	}
	public void TurnOff()
	{
		//turn off the light bulb
	}
}
public class Switch
{
	private LightBulb _lightBulb;
	public Switch()
	{
		_lightBulb = new LightBuld();
	}
	public void Operate()
	{
		_lightbuld.TurnOn();
	}
}
```
Following DIP
```C#
public interface IDevice
{
	void TurnOn();
	void TunrOff();
}
public class Lightbulb : IDevices
{
	public void TurnOn()
	{
		//turn on the light bulb
	}
	public void TurnOff()
	{
		//turn off the light bulb
	}
}
public class Switch
{
	private IDevice _device;
	public Switch(IDevice device)
	{
		_device = device;
	}
	public void Operate()
	{
		_device.TurnOn();
	}
}
```
In this implementation, `switch` depends on the "IDevice" interface rather than a specific implementation like `LightBulb`. This allows for greater flexibility and easier testing.
****
In the last example code illustrating the Dependency Inversion Principle (DIP), the `Switch` class does not actually create an object of `IDevice`. Instead, it takes an `IDevice` object as a dependency through its constructor. This is known as **Dependency Injection**.

**Concept** : Instead of a class creating its own dependencies, it receives them from an external source. This promotes loose coupling and adherence to DIP.

**Example break down,**
1. Interface definition
```C#
public interface IDevice
{
	void TurnOn();
	void TurnOff();
}
```
	'IDevice' is an interface defining two methods, 'TurnOn' and 'turnOff'. Interfaces cannot be instantiated directly. They only provide a contract that classes can implement.
2. Implementation class
```C#
public class LightBulb : IDevice
{
	public void TurnOn()
	{
		//Turn on the light bulb
	}
	public void TurnOff()
	{
		//turn off the light bulb
	}
}
```
	'LightBul' implements the 'IDEvice' interface, providing concrete implementations for the 'TurnOn' and 'TurnOff'.
3. Dependent class
```C#
public class Switch
{
    private IDevice _device;

    // Constructor injection
    public Switch(IDevice device)
    {
        _device = device;
    }

    public void Operate()
    {
        _device.TurnOn();
    }
}
```
	The 'Switch' class depends on an 'IDevice' object, but it does not create it. Instead, it recieves an 'IDevice' object through its constructor. This is known as 'Constructor injection'. 

##### Dependency Injection in Action
Here’s another way to use the `Switch` class with a `LightBulb` object:
```C#
IDevice lightBulb = new LightBulb(); // Create a LightBulb object Switch
lightSwitch = new Switch(lightBulb); // Inject the LightBulb into the Switch
lightSwitch.Operate(); // Operate the switch, which turns on the light bulb
```

Benefits of Dependency injection,
1. Decoupling : `Switch` is decoupled from `LightBulb`. It only on the `IDevice` interface, not on any specific implementation.
2. Flexibility : You can easily switch to different implementation without changing the `Switch` class.
3. Testability : you can easily mock the `IDevice` interface in unit tests to test the `Switch` class in isolation.
****
#### Why use SOLID ?
- Improved maintainability
	Code becomes more manageable and easier to understand. Each module or a class has a clear responsibility. 
- Enhanced flexibility
	Promote writing more modular code which can be extended and modified with minimal impact in the existing system. Useful in rapidly changing environments. 
- Increased reusability
	Individual components become more reusable because they are well defined and decoupled. 
- Better testability
	Easier to test because each component has a single responsibility and it reduce the complexity of unit tests.
#### Benefits
- Reduced code smell(indicators of potential problems)
- Easier refactoring
- Enhanced collaboration 
- Greater scalability