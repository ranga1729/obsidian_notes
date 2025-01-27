# Skip Lists: An Efficient Data Structure

While learning about data structures and algorithms, I encountered a fascinating data structure called the "Skip List." The intriguing part is not just the structure itself, but the mathematical principles behind it.

## Linked Lists (Pros and Cons)

Linked lists are a common data structure in computer science, known for their dynamic adaptability. They offer O(1) time complexity for updates (adding or removing a node once it is identified). However, the downside is that to find a node with a particular value, we must iterate through the list from node to node, resulting in O(n) time complexity for searching.

## Arrays (Pros and Cons)

Arrays are also widely used in everyday programming tasks, allowing random access to elements using indices. In a sorted array, we can perform binary search, achieving O(log n) time complexity for searching operations. However, resizing an array (adding or removing an element) requires shifting all subsequent elements, which results in O(n) time complexity.

## Motivation for Skip Lists

The motivation behind creating the Skip List is to combine the dynamic adaptability of a linked list with the efficient searching capabilities of a sorted array.

## Skip List Nodes (Structure)

Skip lists are built with nodes arranged in a sorted manner, where each node can have multiple pointers. Imagine a skip list node as a tower: the bottom level stores the value, and the upper levels contain pointers that link to other nodes, skipping one or more nodes in between.

These nodes are connected in levels:

- The base level includes the value and a pointer to the next node, resembling a traditional linked list.
- Level 1 pointers link to the nearest node with a level 1 pointer.
- Level 2 pointers link to the nearest node with a level 2 pointer, and so on.

Each level's pointers connect nodes within the same level but not across different levels.

## Height of a Skip List Node

The 'height' of a skip list node (similar to a tower's height) is determined by the number of additional pointers it has. For example, if a node has three additional pointers (Level 1, 2, 3), its height is three.

This height is generated using a random number generating function, which adds a fascinating aspect to skip lists. Initially, it might seem unclear how a random number generator contributes to an efficient data structure, but the beauty and purpose of this method become evident through mathematical explanation.

## Geometric Distribution

The random number generating function creates a geometric distribution. In statistics, a geometric distribution models the number of trials needed to get the first success in a series of independent Bernoulli trials (each trial having only two possible outcomes: success or failure), each with the same probability of success.

For example, if we flip a balanced coin until we get tails:

- The probability of getting heads the first time is 1/2.
- The probability of getting heads twice in a row is 1/4.
- The probability of getting heads three times in a row is 1/8.

The probability of getting tails decreases exponentially with the trial count.

## Creating a Skip List Node with Geometric Distribution

When creating a new node in the skip list, we flip a coin. For each heads, we add an additional pointer. We continue adding pointers until we get tails. This coin flip operation results in a geometric distribution, meaning the probability of having a high-height node decreases exponentially. Thus, very few nodes reach the highest height.

Since the coin flip has a 50% chance for each result:

- Every node has at least one pointer (the base level).
- About half of the nodes will have a level 1 pointer.
- A quarter of the nodes will have a level 2 pointer.
- An eighth of the nodes will have a level 3 pointer.

For example, if we have 100 nodes:

- All 100 nodes have at least one pointer.
- About 50 nodes will have a level 1 pointer.
- About 25 nodes will have a level 2 pointer.

This pattern provides a well-distributed set of nodes throughout the skip list.

## Skip List Searching Operation

The additional pointers in a node and connecting nodes further apart help speed up the searching process. Searching in a skip list follows a top-down approach, starting from the highest level and moving downwards. To find a value, we start at the highest level of nodes. We compare the next node's value with the target value and decide whether to move forward or drop to a lower level.

If the target value is greater than the next node's value, we move forward. If the target value is less than the next node's value, we drop one level and continue the comparison.

This process continues until we either find the target value or reach the base level. At the base level, we perform a traditional linked list search until we find the target or determine it is not present (since the data is sorted).

### Example: Searching for 39

Suppose we are searching for the value 39 in our skip list. Here are the steps:

1. Start at the node with value 17 at Level 4.
    - Compare 39 with the value where the L4 pointer of Node 17 points. It's NULL, so we drop to Level 3.
2. Compare 39 with the value where the L3 pointer of Node 17 points. It's 25.
    - Since 39 > 25, move to the L3 pointer of Node 25.
3. Compare 39 with the value where the L3 pointer of Node 25 points. It's 55.
    - Since 39 < 55, drop to Level 2 of Node 25.
4. Compare 39 with the value where the L2 pointer of Node 25 points. It's 31.
    - Since 39 > 31, move to the L2 pointer of Node 31.
5. Compare 39 with the value where the L2 pointer of Node 31 points. It's 55.
    - Since 39 < 55, drop to Level 1 of Node 31.
6. Compare 39 with the value where the L1 pointer of Node 31 points. It's 44.
    - Since 39 < 44, drop to the base level of Node 31.
7. Compare 39 with the next pointer of Node 31. It's 38.
	-  Since 39 > 38, move forward.
8. Compare 39 with the next pointer of Node 38. It's 39.
	- 39 = 39. We found the value.

If the target value were 40, we would follow the same steps until reaching Node 39's base level. 
Then we compare 40 with the next pointer of Node 39. It's 44.
Since 44 > 40, we conclude 40 is not in the list.
(Since the skip data is sorted, no point of iterating the list beyond the node 44.)
## Comparison

Comparing the search steps in different structures:

- In a traditional sorted linked list, we have to iterate through nodes: 12 -> 17 -> 20 -> 25 -> 31 -> 38 -> 39.
- In an unsorted linked list, it would be worse.

Using a skip list, the nodes we pass through to find 39 are: 17 -> 25 -> 31 -> 38 -> 39, skipping nodes 12 and 20.

## Conclusion

The skip list searching process demonstrates that we iterate through about half the number of nodes at each level, making it clearly more efficient than a linear search. This efficiency arises from the geometric distribution of node heights. Thus, the search operation has a time complexity of O(log n).

In summary, that's how a random number generating function helps build an efficient, dynamically adjustable data structure with reduced cost of searching.