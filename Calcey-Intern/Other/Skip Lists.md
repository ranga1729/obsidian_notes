While learning data structures and algorithms I came across with a fascinating data Structure called Skips lists. Fascinating part is not the data structure itself, but the mathematical explanation behind it. 

### Linked List
List is a very common data structure we used to learn in Computer science and very famous for it's dynamic adaptability.
Lists gives us O(1) time complexity to update the list.(add or remove a node once it's identified).
Downside of a traditional list is, in order to find a node with a particular value, we have to iterate from node to node in the list. This searching operation on list shows O(n) time complexity. 

### Traditional Array
Array is also a very common data structure we use in day-today programming tasks which allows the programmer to randomly access it's elements using the index. 
If we consider an sorted array, We can perform binary search algorithm with it's random access capability. This allows us to get O(log n) time complexity in searching operations.
But the downside of an traditional array is, if we want to resize the array(add or remove an element) we have to shift all other elements beyond the changed element.  
This operation give us a O(n) time complexity.

### Motivation behind building the Skip List
The motivation behind building the Skip list is to create a data structure which has the dynamic adaptability of a linked list and to reduce the cost of the searching operation like a sorted array.

### Intro to Skip List: Nodes
Skips list is built with nodes arranged in a sorted pattern and each node can have additional pointers.(more than 1)

Imagine a skip list node as a tower. Bottom of the node stores the value and the other levels are consist of pointers which link other nodes in the list skipping one or more nodes in-between them.

These nodes are connected in level. 
The base level includes the value.
Since links list nodes use to have pointers to create the list structure, level 1 pointers are connected to each other like a traditional link list.

But level 2 pointer can link the closest node with a level 2 pointer and that node's level 2 pointer can link to the next node with a level 2 pointer.

Level 3 pointers can link to the closest node with a level 3 pointer and same apply to other level 3 nodes as well. And the level 2 pointer of this node is having a link with another level 2 node.

2nd level pointers can be linked with another 2nd level pointer, but not with a 3rd level pointer.
(Look at the image)
### 'Height' of a node
The 'height' of a skip list node( like the height of a tower) is based on the number of pointers its have. 
If a node is having 3 pointers(Level1, 2, 3), the height of this node is 3.

And this number is generated on a random number generating function. 
That's the fascinating part here. First I wondered how a random number generating function can be used to develop an efficient data structure.

The real beauty of this data structure and the purpose of having a random number generating function started to reveal after I got into the mathematical explanation of the skip list data structure.

### Geometric Distribution
This random number generating function is used to create a geometric distribution. In statistics, A geometric distribution is a discrete probability distribution that models the number of trials needed to get the first success in a series of independent Bernoulli trials(each trial have only two possible outcomes: success or failure), each with the same probability of success.

For an example, let's say we flip a balanced coin until we get tail.
Probability of getting 'head' first time = 1/2
Probability of getting 'head' two times in a row = 1/2 * 1/2 = 1/4
Probability of getting 'head' three times in a row = 1/2 * 1/2 * 1/2 = 1/8

Probability of getting the 'tail' after more trails decreases exponentially.
### Creating a new node using a Geometric distribution
when creating a new node in the skip list, we flip a coin. If we get 'head', we add an additional pointer. We continue adding additional pointer to this new node until we get a tail.

Coin flip operation results a geometric distribution, so the probability of having a high height for a node decreases exponentially.
Due to this reason,
There will be very few nodes reach the highest height(number of additional pointers).

Since the coin flip has 50% chance for both results, 
Every node has at least a one(Level 1) pointer.
half of the number of the total nodes will have a level 2 pointer.
Half of the number of the nodes which reaches to the 2nd level will have a 3rd level pointer.
Half of the number of the nodes which reaches to the 3rd level will have a 4th level pointer. 

Possibility of a node reaches level 2 = 50%
Possibility of a node reaches level 3 = 25%
Possibility of a node reaches level 4 = 12.5%

For an example,
Let's say we have 100 nodes,
All 100 nodes has at least a one pointer, which helps to create the list structure. 
Nearly 50 of them will have a 2 level's additional pointer.
Nearly 25 of them will have a 3rd level's additional pointer. 

This pattern gives us a good distribution of nodes throughout the skip list.
### Searching Process
purpose of having additional pointers in a node and connecting nodes further away on several levels is to jump over some nodes in-between them to speedup the searching process. 

When ever we want to search a value in the skip list, we start from nodes with the highest height. 
Then we compare if the next node's value is the value we are looking for.
If not we compare two values and decide whether the next node's value is more than or less than the value we are looking for.(Remember link lists is built on sorted data.)
If the value we are looking for is less than the next node's value, we jump to that node using the pointer.
If the value we are looking for is more than the next node's value, we decrease the level we are searching currently by 1 and continue comparing with the next node's value.

Every time the next node's values is less than what we are looking for, we move forward to the next node. Every time the next node's value is more than what we are looking for we decrease the level of searching. 
We continue this process until we get the value we are looking for or we reach the base level. 
After reaching the base level, We continue to search like a traditional skip list until we find the value we we looking for, if the next node's value is greater than the value we are looking for then the searching value is not in this skip list.(Since skip list data is sorted)

Ex: Lets say we are looking for the value 39 in our skip list. 
Search starts from nodes with the highest hights.

**Step 1:** We start from the node with value 17 because it has a Level 4 pointer.
Then we compare 39 with the value where L4 pointer of the Node 17 is pointing at. It's NULL. So we decrease our Level of searching to the Level 3 of the Node 17.

**Step 2:** Then we compare 39 with the value where L3 pointer of the Node 17 is pointing at. It's 25. 
Since 39 > 25, We jump to the L3 pointer of the Node 25. 

**Step 3:** Then we compare 39 with the value where L3 pointer of the Node 25 is pointing at. It's 55. 
Since 39 < 55, We decrease the Level of searching from 3 to Level 2 of the Node 25. 

**Step 4:** Then we compare 39 with the value where L2 pointer of the Node 25 is pointing at. It's 31. 
Since 39 > 31, We jump to the L2 pointer of the Node 31. 

**Step 5:** Then we compare 39 with the value where L2 pointer of the Node 31 is pointing at. It's 55. 
Since 39 < 55, We decrease the Level of searching from 2 to Level 1 of the Node 31. 

**Step 6:** Then we compare 39 with the value where L1 pointer of the Node 31 is pointing at. It's 44. 
Since 39 < 44, We jump to the Base Level of the Node 31. 

Now we have reached the base level, 
**Step 7:** Then we compare 39 with the Next pointer of the Node 31. It's 38.
**Step 8:** since 39 > 38, We move forward. Now we got the value we need.

If the value we are looking for is 40, 
Same steps will be followed and we reach to the Node 39 Base level. 
Then we compare the Next pointer of the Node 39. It's 44.
44 > 40. Since the skip data is sorted, no point of iterating the list beyond the node 44.
We can surely say that 40 is not in the list.

### Comparison
If we use a traditional link list with sorted data to search for the value 39, Node we have to iterate will be,
12 -> 17 -> 20 -> 25 -> 31 -> 38 -> 39
If the data is not sorted, this would be worse. 

Using the skip list approach, the nodes we had to pass in order to search the value 39 is,
17 -> 25 -> 31 -> 38 -> 39
We have skipped Nodes 12 and 20.
Since the data is sorted, we don't even have to think about other values beyond 39.
No need to reaching Nodes 44, 50, 55.

Since this Geometric distribution of Nodes and their heights, theoretically searching operation's time complexity will be Log N. 

And that's how we can use a Random number generator function to build an efficient data structure that dynamically adjustable and with reduced cost of searching. 

Isn't that fascinating ? 