[[Algorithm I]]
***
**Table of Contents**
```table-of-contents
```

***
## Concept

We consider a function recursive if it contains a call to itself. This mechanism is often used for repetitions, it replaces loops.
	*Recursive functions / Iterative functions*
```c
// Iterative: Uses a loop
int factorial(int n) {
	int res = 1;
	while (n > 0) {
		res = res * n;
		n--;
	}
	return res;
}

// Recursive: calls itself
int factorial(int n) {
	if (n == 0)
		return 1;
	else
		return n * factorial(n - 1);
}
```

The approach depends of the situation: Some algorithms are way more elegant and natural when recursive, while sometimes its preferable to go for an iterative approach.

A recursive function contains at least two parts:
1. **Base Case**: The simplest instance of the problem, which can be solved directly.
    - Example: Returning 0 when summing an empty array.

2. **Recursive Case**: The logic for breaking the problem into smaller instances and solving them recursively.
    - Example: Summing the first element and the result of summing the rest of the array.


***
## Recursive Decomposition

To solve a problem recursively:
1. Identify base cases: For example, the base case for summing an array is when the array is empty.
2. Define a simpler problem: Break the problem into smaller sub-problems.
3. Combine results: Use the result of the smaller problem to build the solution to the original problem.

Sum of an array:
```c
// Iterative
int sum_iter(int arr[], int lo, int hi) {
    int sum = 0;
    for (int i = lo; i <= hi; i++) {
        sum += arr[i];
    }
    return sum;
}

// Recursive
int sum(int arr[], int lo, int hi) {
    if (lo > hi)
        return 0;
    return arr[lo] + sum(t, lo + 1, hi);
}
```


***
## Rules of recursion

Simple guidelines to help building recursive functions:
1. **Always define a base case first**:
    The return conditions are often at the beginning of the function. It avoids infinite recursion by ensuring the function eventually terminates.
    
2. **Change the arguments in the recursive call to approach the base case**:
    Ensure that the recursive case simplifies the problem.
    
3. **Simplify only after correctness**:
    Write a clear, correct function before optimising or simplifying it.


***
## Common mistakes

In general:
- **Missing or incorrect base case**: Leads to infinite recursion.
- **Improper simplification in recursive case**: Arguments do not approach the base case.
- **Incorrect combination of results**: Miscalculations when building up the final result.


***
## Basic examples

Let's write a recursive version of our method that finds duplicates in an array:
```c
int find(int arr[], int lo, int hi, int elt) { // Linear search
	if (lo > hi)
		return 0;
	else if (arr[lo] == elt)
		return 1;
	return find(arr, lo + 1, hi, elt);
}

int duplicates(int arr[], int lo, int hi) {
	if (lo > hi)
		return 0;
	else if (find(arr, lo+1, hi, arr[lo]))
		return 1;
	return duplicates(arr, lo+1, hi);
}
```

Another example: reverse the convent of an array:
```c
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

void reverse(int arr[], int lo, int hi) {
    if (lo > hi)
        return;
    swap(&arr[lo], &arr[hi]);
    reverse(arr, lo + 1, hi - 1);
}
```


***
## Complexity

How can we find a recursive function's complexity?
We define:
- `A(n) = O(f(n))`, the total amount of function calls
- `B(n) = O(g(n))`, the total amount of fundamental operations per call
We obtain:
$$T(n) = O(A(n) × B(n)) = O(f(n) × g(n))$$

```c
int find(int arr[], int lo, int hi, int elt) {
	if (lo > hi)
		return 0;
	else if (arr[lo] == elt)
		return 1;
	return find(arr, lo + 1, hi, elt);
}
```
> Recursive find: `A(n) = O(n), B(n) = 1 ⇒ T(n) = O(A(n) × B(n)) = O(n)`

```c
int duplicates(int arr[], int lo, int hi) {
	if (lo > hi)
		return 0;
	else if (find(arr, lo + 1, hi, arr[lo]))
		return 1;
	return duplicates(arr, lo + 1, hi);
}
```
> `A(n) = O(n), B(n) = O(n) ⇒ T(n) = O(A(n) × B(n)) = O(n × n) = O(n²)`


***
## Tail Recursion

A recursive function is **tail-recursive** if it returns **immediately after the recursive call**. Tail recursion can be converted to an iterative form:
```c
void reverse_tail(int arr[], int lo, int hi) { 
start: 
    if (lo > hi) 
        return; 
    swap(&arr[lo], &arr[hi]); 
    lo += 1;    // Update arguments before rolling back to the top of
    hi -= 1;    // the function
    goto start;
}
```

However, the iterative approach is more efficient for this situation:
```c
void reverse_iter(int arr[], int lo, int hi) {
	for (; lo <= hi; lo += 1, hi -= 1) {
		swap(&arr[lo], &arr[hi]);
	}
}
```


***
## Backtracking

Backtracking is a method used to solve problems by exploring all possible solutions and systematically **eliminating invalid ones**. It is particularly useful for problems involving combinatorial optimisation or constraint satisfaction.
Key Characteristics:
- **Divide the Problem**: Break the problem into smaller sub-problems (e.g., placing a queen in one column at a time).
- **Constraint Checking**: At each step, ensure that the current partial solution meets the problem constraints (e.g., no two queens attack each other).
- **Recursive Exploration**: If the current step is valid, proceed to the next step; otherwise, backtrack to try a different possibility.
- **Termination**: The process stops when all possibilities have been explored, or a solution is found.

This can be applied to solve sudokus, labyrinths, graph traversal...  It has an exponential complexity in the worst case scenario.

### Example: The N-Queens Problem

The N-Queens Problem involves placing `N` queens on an `N×N` chessboard such that no two queens attack each other.
![[queens.png]]
> Can we do it? If so, how many solutions exists?

We represent the board with an `int[4]` array:
![[board.png]]
> `[0, 3, 1, 2]`

Here is a solution involving recursive calls:
```c
int conflict(int board[], int size, int pos) {
	int i;
	for (i = 0; i < pos; i++) { // Detect all attacks (v, h, d)
		if (board[i] == board[pos] ||
			(pos-i == board[pos]-board[i]) ||
			(pos-i == board[i]-board[pos]))
			return 1;
	}
	return 0;
}

int queens(int board[], int size, int pos) { 
    if (pos == size) // We placed all the queens
        return 1;
    int count = 0;
    for (int i = 0; i < size; i++) { 
        board[pos] = i; 
        if (!conflict(board, size, pos)) // If no conflict, place queen
            count += queens(board, size, pos + 1); 
    } 
    return count;
}
```
> Note that this can work for all board sizes, not only 4.

With out solution, we see that there are two solutions:
![[queens_solution.png]]