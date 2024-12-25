[[Algorithm I]]
***
**Note:** For this semester, we will focus on arrays (we will take a look at other data structures later).
All of this chapter will be done in C (Using python is cheating...)
***
**Table of Contents**
```table-of-contents
```

***
## Why Study Algorithms?

We need to ensure our code is:
1. **Correct**: Does the algorithm solve the intended problem?
	*Testing and debugging are essential, bugs are costly in production*
2. **Efficient**: How well does the algorithm use resources?
    Metrics include:
        Time: How long does it take?
        Space: How much memory is required?
        Energy: Computational power used.
        Communication: Data exchanged in distributed systems.

By doing so, we can:
- Compare algorithm solutions for a same task
- Diagnose inefficiencies in implemented algorithms
- Predict performance on larger datasets


***
## Complexity?

Algorithm complexity measures the resource usage of an algorithm as a function of input size, typically focusing on the **worst-case scenario**:

A first approach would be to determine complexity by execution time.
The problem(s) are that execution duration varies a lot, depending on: 
- The programming language we chose
- The compiler/interpreter
- The scheduler
- The hardware (CPU, RAM)
- Caching
- ...
> Too many factors can influence the execution duration of our algorithm, it's not the best approach.

Let's take a basic linear search example. Our algorithm must (on an unsorted array):
1. Traverse each element.
2. Compare it with the target.
3. Return success if found; otherwise, continue.

We can have the following algorithm in C:
```c
int find_linear(int t[], int size, int elt) {
	int i;
	for (i = 0; i < size; i++) { // 1
		if (t[i] == elt)         // 2
			return 1;            // 3-true
	}
	return 0;                    // 3-false
}
```

What can we observe here? If we get lucky (best-case scenario), we will found the element on first position. However, the worst-case scenario would be that the element is placed in last position (or not at all, even).
The later forces us to **traverse all `n` elements** of the array. Our algorithm is in `O(n)`


***
## Big O Notation

Big O notation describes the upper bound of an algorithm's growth rate. 
if `f(n)` is the runtime, we say:
$$
f(n)=O(g(n))
\text{ if: } f(n)≤C⋅g(n) for n≥n0f(n)≤C⋅g(n)
$$
where `C` and `n0` are constants.

**Big O follow those rules:**
- Constants are ignored: 
$$C⋅f(n)=O(f(n))$$
- Additive complexities: $$O(f(n))+O(g(n))=O(max⁡(f(n),g(n)))$$
- Multiplicative complexities: $$O(f(n))⋅O(g(n))=O(f(n)⋅g(n))$$
***
## Common Complexities

| Complexity | Description | Examples                                          |
| ---------- | ----------- | ------------------------------------------------- |
| O(1)       | Constant    | Get element in a map, check parity of a number... |
| O(log n)   | Logarithmic | Binary search                                     |
| O(n)       | Linear      | Traverse an array                                 |
| O(n log n) | Log-linear  | Merge sort                                        |
| O(n²)      | Quadratic   | Compare all pairs in an array                     |
| O(2^n)     | Exponential | Brute force combinatorics                         |


![[complexity.jpg]]


***
## Basic Algorithms

### Binary Search

Let's find an element in a sorted array. We could use our `find_linear` algorithm in `O(n)`, but a binary search is more efficient as it's in `O(log n)`.

Here's the logic:
1. Begin with the middle element.
2. If it is smaller, search in the left half; if larger, search in the right half; If target, If return success.
3. Repeat until the array is empty.
![[binary_search.png]]
> We attempt to find "25", and begin at the middle element: 9.
> 9 is smaller than 25, so we jump to the middle of the right part (containing greater values)
> We keep on doing so until we find our element, or until nothing is left (which is the case here, 25 isn't in the array).

Here is a way of doing it in C:
```c
int find_binary(int arr[], int size, int elt) {
	int left = 0, right = size - 1;          // Set initial bounds
	while (left <= right) {                  // Continue while range is valid
		int mid = left + (right - left) / 2; // Compute midpoint
		if (arr[mid] == elt)                 // Check midpoint
			return mid;                      // Return position
		
		if (arr[mid] < elt)                  // Search right half
			left = mid + 1;
		else                                 // Search left half
			right = mid - 1;
	}
	return -1;                               // Element not found
}
```


### Nested loops: Finding duplicates

Let's find duplicate entries in our array:
```c
int duplicates(int arr[], int n) {
    for (int i = 0; i < n; i++) {       // Runs n times
        for (int j = i+1; j < n; j++) { // Runs n − i − 1 times.
            if (arr[i] == arr[j])
	            return 1;
        }
    }
    return 0;
}
```
![[duplicates.png]]

Here we can observe that a nested loop traversing the entire structure results in an `O(n²)` complexity:
$$
∑i=1n​(n−i)=2n(n−1)​=O(n2).
$$

If loops iterate over the same range (e.g., both depend on n), their nested complexity is typically `O(n²)` for a two-level loop, `O(n³)` for a three-level loop, and so on:
```c
for (i = 0; i < n; i++) {
	for (j = 0; j < n; j++) {
		// ...
	}
}
```

If loops **iterate over different ranges**, their complexity is `O(m×n)` for a two-level loop, `O(m×n×p)` for a three-level loop, and so on:
```c
for (i = 0; i < m; i++) {
	for (j = 0; j < n; j++) {
		// ...
	}
}
```

