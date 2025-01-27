A union in C++ is a user-defined data type that can store only one of its member variables at a time.
All of the member variables of a union share the same memory location, so when one member variable is assigned a value, the other member variables are overwritten.

Difference of unions and structs are,
- Structs can hold up every member variable at the same time while unions allows you to chose only a one at a time.
- Structs get enough memory to store all of it's variables while unions takes up memory enough to hold up the largest variable.

Unions can be declared by using `union` keyword.

Ex:
```C++
union myUnion
{
	int i;
	float f;
	char c;
};

struct myStruct
{
	int i;
	float f;
	char c;
};
```
This union takes up memory equal to the size of an integer, which is the largest of three member variables. And you only choose one variable out of three at a time.
This struct takes up takes up memory equal to the sum of an integer, a float and a char. You can use all the 3 variables.

**Syntax :**
```C++
union union_name
{
	data_type var_name1,
	data_type var_name2,...
};
```

#### To use a unions
```C++
union MyUnion
{
	int i;
	char c;
}
int main()
{
	MyUnion union1;
	//assigning a integer value to the union
	union1.i = 10;
	//now other member variables ore overwritten. You can't use them for now.
	//using the value.
	cout << union1.i << endl;
	//assigning a charatcer to the union
	union1.c = 'A';
	//now other member variables ore overwritten. You can't use them for now.(Previous integer value has been overwritten)
	cout << union1.c << endl;
	
}
```

- Unions are useful for saving memory.
- Need to be careful as it's easy to accidentally overwrite the data in the unions.

Use cases,
- In operating system, to improve manage memory usage.
- In Embedded systems.

### Anonymous Unions
You can create anonymous unions in the main method without a name.
Ex:
```C++
int main()
{
	//anonymous union
	union
	{
		int x;
		double d;
		//memory usage is 8bit.(double is the largets.)
	}; 
	
	x= 1;
	cout << x << endl;
	d = 1024.435 //now x is overwritten and it may have garbage values.
	cout << d << endl;
}
```
Using anonymous unions you can use member variable without using `.` operator, address them as a usual variable. But when you assign another member variable in the union, previous variable will be overwritten.
