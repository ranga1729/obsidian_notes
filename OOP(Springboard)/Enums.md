Enum or enumeration is a user defined datatype in which set of values are specified for a variable and the variable can take out one value from the set of values.
`Enum` keyword is used to define enumeration.

As the description above Enum is consist of 2 parts,
	1. Integer value
	2. A name that is used in order to describe that integer values.

#### Usage: 
Computers are good with numbers but humans are more likely to remember words than numbers. So we can create a list of words with each word has a integer values. 

***Syntax :***
```C++
enum name{value1, value2, value3,...};
```
By default,
	value1 = 0,
	value2 = 1,
	value3 = 3, ...

Ex:
```C++
enum EyeColor{Brown, Blue, Green, Gray};

int main()
{
	EyeColor mine = Brown;
	cout << mine; //output = 0
	cout << mine+1 //output = (0+1) = 1
}
```

#### Changing values
For this section, refer this example,
```C++
enum Colors{Red, Orange, Yellow, Green, Blue, Violet};
```
Values,
	Red = 0,
	Orange = 1,
	Yellow = 2, 
	Green = 3, 
	Blue = 4,
	Violet = 5

**Method 1**
```C++
enum EyeColor{Red=2, Orange, Yellow, Green, Blue, Violet};
```
Now values are,
	Red = 2,
	Orange = 3,
	Yellow = 4, 
	Green = 5, 
	Blue = 6,
	Violet = 7
- When you change a value, other elements beyond that element will increment their values like in a sequence. No need to explicitly mark them one by one. 

```C++
enum EyeColor{Red, Orange, Yellow, Green=10, Blue, Violet};
```
Now values are,
	Red = 0,
	Orange = 1,
	Yellow = 2, 
	Green = 10, 
	Blue = 11,
	Violet = 12

**Method 2**
You can design a Enum explicitly saying what are the integer values.
```C++
enum EyeColor{Red=10, Orange=6, Yellow=7, Green=15, Blue=23, Violet=99};
```
Now values are,
	Red = 10,
	Orange = 6,
	Yellow = 7, 
	Green = 15, 
	Blue = 23,
	Violet = 99

#### Type cast
Ex:
```C++
enum Colors{Red, Orange, Yellow, Green, Blue, Violet};
int main()
{
	int input;
	cin >> input;
	
	Colors mycolor;
	mycolor = (Colors)input;
}
```
You have to cast the user input into colors type.
`(Colors)input`

#### Iterations
You can iterate through a Enum using a for loop.
```C++
enum Colors{Red, Orange, Yellow, Green, Blue, Violet};
int main()
{
	for(int myColor=Red; myColor<Violet+1; myColor++)
	{
		//enter you code
	}
}
```

### Enum as a function Parameter
You can use Enum as a function parameter.
Ex:
```C++
enum Colors{Red, Orange, Yellow, Green};

void getNumbers(Colors mycolor) {
	switch (myColor)
	{
		case Red: cout<<"Red is the color" << endl; break;
		case Orange: cout<<"Orange is the color" << endl; break;
		case Yellow: cout<<"Yellow is the color" << endl; break;
		case Green: cout<<"Green is the color" << endl; break;
	}
}

int main()
{
	getColor(Red);
}
```
This is useful when you working in a larger projects to understand the codes. 