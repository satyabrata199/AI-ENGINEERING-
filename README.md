Perfect. If your goal is **100 DSA questions in 15 days** for service-based companies, don't solve random problems. Follow a progression where each concept builds on the previous one.

## 15-Day Roadmap (100 Problems)

| Topic          | Problems |
| -------------- | -------: |
| Arrays         |       20 |
| Hashing        |       10 |
| Two Pointers   |        8 |
| Sliding Window |        8 |
| Binary Search  |        8 |
| Linked List    |       10 |
| Stack          |        8 |
| Queue          |        4 |
| Trees          |       10 |
| Heap           |        4 |
| Graph          |        6 |
| Backtracking   |        2 |
| DP             |        2 |
| **Total**      |  **100** |

---

# Phase 1: Arrays (20 Problems)

### Easy Foundation (1-10)

1. Find Largest Element
2. Find Second Largest Element
3. Check Array Sorted
4. Remove Duplicates from Sorted Array
5. Move Zeroes
6. Left Rotate Array by One
7. Left Rotate Array by K Places
8. Missing Number
9. Find Single Number
10. Union of Two Sorted Arrays

### Medium (11-20)

11. Kadane's Algorithm
12. Best Time to Buy and Sell Stock
13. Rearrange Array by Sign
14. Next Permutation
15. Majority Element
16. Set Matrix Zeroes
17. Rotate Matrix
18. Spiral Matrix
19. Merge Intervals
20. Maximum Subarray Sum

### Concepts

* Traversal
* Prefix Sum
* Kadane
* Greedy
* Matrix Manipulation

---

# Phase 2: Hashing (10 Problems)

21. Frequency Count
22. First Non-Repeating Character
23. Valid Anagram
24. Two Sum
25. Contains Duplicate
26. Longest Consecutive Sequence
27. Group Anagrams
28. Subarray Sum Equals K
29. Count Distinct Elements
30. Find Duplicate Number

### Concepts

* HashMap
* HashSet
* Frequency Arrays

---

# Phase 3: Two Pointers (8 Problems)

31. Two Sum Sorted
32. Remove Duplicates
33. Move Zeroes
34. Container With Most Water
35. 3 Sum
36. 4 Sum
37. Trapping Rain Water
38. Sort Colors

### Pattern

```text
left = 0
right = n-1

while(left < right)
```

---

# Phase 4: Sliding Window (8 Problems)

39. Maximum Sum Subarray K
40. First Negative in Every Window
41. Longest Substring Without Repeating
42. Max Consecutive Ones III
43. Fruits Into Baskets
44. Longest Repeating Character Replacement
45. Minimum Window Substring
46. Permutation in String

### Pattern

```text
expand right
while invalid:
    shrink left
```

---

# Phase 5: Binary Search (8 Problems)

47. Binary Search
48. Lower Bound
49. Upper Bound
50. Search Insert Position
51. First and Last Position
52. Search Rotated Array
53. Find Peak Element
54. Koko Eating Bananas

### Pattern

```text
low=0
high=n-1

while(low<=high)
```

---

# Phase 6: Linked List (10 Problems)

55. Traverse Linked List
56. Reverse Linked List
57. Middle of Linked List
58. Detect Cycle
59. Remove Nth Node
60. Merge Two Sorted Lists
61. Palindrome Linked List
62. Intersection Point
63. Add Two Numbers
64. Reverse in K Groups

### Must Master

```text
slow-fast pointers
```

---

# Phase 7: Stack (8 Problems)

65. Valid Parentheses
66. Next Greater Element
67. Previous Greater Element
68. Next Smaller Element
69. Min Stack
70. Largest Rectangle Histogram
71. Daily Temperatures
72. Stock Span Problem

### Pattern

```text
while stack and condition:
    stack.pop()
```

---

# Phase 8: Queue (4 Problems)

73. Implement Queue
74. Circular Queue
75. First Non-Repeating Character Stream
76. Sliding Window Maximum

---

# Phase 9: Trees (10 Problems)

77. Inorder Traversal
78. Preorder Traversal
79. Postorder Traversal
80. Level Order Traversal
81. Height of Tree
82. Diameter of Tree
83. Balanced Binary Tree
84. Lowest Common Ancestor
85. Binary Search Tree Search
86. Validate BST

### Must Know

```text
DFS
BFS
Recursion
```

---

# Phase 10: Heap (4 Problems)

87. K Largest Elements
88. Kth Largest Element
89. Top K Frequent Elements
90. Merge K Sorted Arrays

### Pattern

```text
PriorityQueue
```

---

# Phase 11: Graph (6 Problems)

91. BFS Traversal
92. DFS Traversal
93. Number of Islands
94. Flood Fill
95. Detect Cycle Undirected Graph
96. Rotten Oranges

### Must Know

```text
visited[]
queue
recursion
```

---

# Phase 12: Backtracking (2 Problems)

97. Generate Parentheses
98. Subsets

### Pattern

```text
choose
explore
unchoose
```

---

# Phase 13: DP (2 Problems)

99. Climbing Stairs
100. House Robber

### Pattern

```text
dp[i] = best answer till i
```

---

## Daily Target

* **Days 1-3:** Arrays (20)
* **Days 4-5:** Hashing + Two Pointers (18)
* **Days 6-7:** Sliding Window + Binary Search (16)
* **Days 8-9:** Linked List (10)
* **Days 10-11:** Stack + Queue (12)
* **Days 12-13:** Trees (10)
* **Day 14:** Heap + Graph (10)
* **Day 15:** Backtracking + DP + Revision (4 + revision)

### Rule for Every Problem

1. Brute Force
2. Better Solution
3. Optimal Solution
4. Time Complexity
5. Space Complexity
6. Write template from memory

Start with **Problem #1: Find Largest Element in Array**, then move sequentially. This ordering is designed so that by problem 100 you'll have covered almost every pattern commonly asked in service-based company interviews.

