Here are some **Data Structures and Algorithms (DSA) questions** that are commonly asked in **internship interviews**, along with brief hints or solutions.

---
## **1. Arrays and Strings**

### **1.1 Find the missing number in an array**

**Problem:** Given an array containing `n-1` distinct numbers from `1` to `n`, find the missing number.  
🔹 **Hint:** Use the sum formula `n*(n+1)/2` or XOR.  
🔹 **Example:**

```java
int missingNumber(int[] nums) {
    int n = nums.length + 1; // One number is missing
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) actualSum += num;
    return expectedSum - actualSum;
}
```

---

### **1.2 Two Sum Problem**

**Problem:** Given an array of numbers and a target sum, return the indices of two numbers that add up to the target.  
🔹 **Hint:** Use a HashMap for O(n) time complexity.  
🔹 **Example:**

```java
int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        map.put(nums[i], i);
    }
    return new int[] {};
}
```

---

### **1.3 Longest Substring Without Repeating Characters**

**Problem:** Given a string, find the length of the longest substring without repeating characters.  
🔹 **Hint:** Use a sliding window approach with a HashSet.  
🔹 **Example:**

```java
int lengthOfLongestSubstring(String s) {
    Set<Character> set = new HashSet<>();
    int left = 0, maxLength = 0;
    for (int right = 0; right < s.length(); right++) {
        while (set.contains(s.charAt(right))) {
            set.remove(s.charAt(left++));
        }
        set.add(s.charAt(right));
        maxLength = Math.max(maxLength, right - left + 1);
    }
    return maxLength;
}
```

---

## **2. Linked Lists**

### **2.1 Reverse a Linked List**

🔹 **Hint:** Use two pointers (prev and current).  
🔹 **Example:**

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode nextTemp = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nextTemp;
    }
    return prev;
}
```

---

### **2.2 Detect Cycle in a Linked List**

🔹 **Hint:** Use Floyd’s Cycle Detection Algorithm (slow and fast pointer).  
🔹 **Example:**

```java
boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

---

## **3. Stack and Queue**

### **3.1 Implement a Min Stack**

🔹 **Hint:** Use a second stack to track the minimum element.  
🔹 **Example:**

```java
class MinStack {
    Stack<Integer> stack = new Stack<>();
    Stack<Integer> minStack = new Stack<>();

    void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek()) minStack.push(val);
    }

    void pop() {
        if (stack.pop().equals(minStack.peek())) minStack.pop();
    }

    int top() { return stack.peek(); }
    
    int getMin() { return minStack.peek(); }
}
```

---

### **3.2 Implement a Queue using Stacks**

🔹 **Hint:** Use two stacks (`pushStack` and `popStack`).  
🔹 **Example:**

```java
class MyQueue {
    Stack<Integer> pushStack = new Stack<>();
    Stack<Integer> popStack = new Stack<>();

    void push(int x) {
        pushStack.push(x);
    }

    int pop() {
        if (popStack.isEmpty()) {
            while (!pushStack.isEmpty()) {
                popStack.push(pushStack.pop());
            }
        }
        return popStack.pop();
    }

    int peek() {
        if (popStack.isEmpty()) {
            while (!pushStack.isEmpty()) {
                popStack.push(pushStack.pop());
            }
        }
        return popStack.peek();
    }

    boolean empty() {
        return pushStack.isEmpty() && popStack.isEmpty();
    }
}
```

---

## **4. Binary Trees and Graphs**

### **4.1 Find the Maximum Depth of a Binary Tree**

🔹 **Hint:** Use DFS (recursion) or BFS (level-order traversal).  
🔹 **Example (Recursive DFS):**

```java
int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

---

### **4.2 Check if a Binary Tree is Balanced**

🔹 **Hint:** A tree is balanced if the height difference of every node’s left and right subtree is ≤ 1.  
🔹 **Example:**

```java
boolean isBalanced(TreeNode root) {
    return checkHeight(root) != -1;
}

int checkHeight(TreeNode node) {
    if (node == null) return 0;
    int left = checkHeight(node.left);
    int right = checkHeight(node.right);
    if (Math.abs(left - right) > 1 || left == -1 || right == -1) return -1;
    return 1 + Math.max(left, right);
}
```

---

## **5. Searching and Sorting**

### **5.1 Binary Search**

🔹 **Hint:** Divide and conquer, O(log n).  
🔹 **Example:**

```java
int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

---

### **5.2 QuickSort Algorithm**

🔹 **Hint:** Choose a pivot, partition, and recursively sort subarrays.  
🔹 **Example:**

```java
void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

int partition(int[] arr, int low, int high) {
    int pivot = arr[high], i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    swap(arr, i + 1, high);
    return i + 1;
}

void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

---

These **15+ DSA problems** cover core topics for an **internship interview**. Let me know if you need more! 🚀