### JS Sorting
1. Default Behavior (Lexicographical Sorting)
Convert elements to strings and sort them based on Unicode. 
Use this method for alphabetical sorting.
```js
let numbers = [25, 8, 90, 3];
numbers.sort();
Console.log(numbers); //[25, 3, 8, 90]
```

2. Syntax of comparing function
```js
array.sort((a,b) => a-b);
```
- `a-b` : Ascending order(small to large)
- `b-a` : Descending order (large to small)

3. Sorting object by property
To sort an array of objects by a property.
```ls
let user = [
	{ name: "Alice", age:25 },
	{ name: "Bob", age: 22 },
	{ name: "Charlie", age: 28 }
]
users.sort((a,b) => a.age - b.age);
console.log(users);

users.sort((a, b) => a.name.localCompare(b.name));
console.log(users);
```

- ES10(2019) sorting method is stable. 

4. Use  `reverse()` to reverse an already sorted array. 
```js
let numbers = [3, 8, 25, 90];
umbers.reverse();
console.log(numbers); //90, 25, 8, 3
```

****
### JS string methods
1. String length
```js
let str = "Hello";
console.log(str.length); //5
```
2. `str.toUpperCase()` : Converts to uppercase
3. `str.toLowerCase()` : Convert to lowercase
4. `str.slice(start, end)` : Extracts part of a string
- start : The starting index(0-based)
- end : (Optional) stops before this index
```js
let str = "JavaScript";
console.log(str.slice(0, 4)); //Java
console.log(str.slice(4)); //Script
console.log(str.slice(-6)); //Script
```
5. `substring(start, end)` : Similar to `slice` but no negative indexes.
6. `replace(old, new)` : Replace part of a string. First occurrence only. Case sensitive.
```js
let str = "I love JavaScript";
str.replace("JavaScript", "Python"); //I love Python
```
To replace all occurrences, use regex `g` flag,
```js
let text = "Banana is a banana, banana, banana";
text.replace(/banana/gi, "Apple");
```
7. `replaceAll(old, new)` : To replace all.
```js
console.log(text.replaceAll("banana", "apple"));
```

****
### Splitting & Joining
1. `split(separator, limit)` - Split a string into an array
```js
let str = "apple, banana, grape";
let fruit = str.aplit(",");
console.log(fruits); //['apple','banana','grape']
```
2. `join(seperator)` - Joins an array into a string
```js
let joined = fruits.join(" - ");
console.log(joined); //apple - banana - grape
```
*****
### Searching in strings
1. `indexOf(substring)` - Finds first occurrence index
```js
let text = "Hello, world";
console.log(text.indexOf("world")); //6
console.log(text.indexOf("JavaScript")); //-1
```
2. `lastIndexOf(substring)` - Finds last occurrence index
```js
let text = "Banana, Apple, Banana";
console.log(text.lastIndexOf("Banana")); //15
```
3. `include(substring)` : Check if a string contains another string
```js
console.log(text.includes("world")); //true
console.log(text.includes("'JavaScript")); //false
```
4. `startWith(substring)` & `endWith(substring)`
```js
console.log(text.startsWith("Hello")); // true
console.log(text.endsWith("world")); // true
```
****
### Trimming Strings
1. `trim()` - Remove space from both ends
```js
let str = "  Hello, World  ";
console.log(str.trim()); //"Hello, World"
```
2. `strimStart()` & `trimEnd()`