[[Algorithm II]]
***
**Table of Contents**
```table-of-contents
```

***
## Concept

>[!caution]
>Do not misunderstood the concepts of binary heap and memory heap. 
>Those aren't related at all besides their name.

An **almost-perfect tree** is a tree where **all levels are completely filled except possibly the last level**, and where **nodes in the last level are filled from left to right**.
![[almost_perfect.png]]

A **tournament tree** (A.K.A winner tree) is a binary tree satisfying the following property:
`property(x) <= property(y)` if `x` is the parent of `y`.
![[tournament_tree.png]]

A binary heap is the representation of an almost-perfect tournament tree as an array, where nodes are represented in **breadth-first order**.
![[binary_heap.png]]
> [!info]
> It follows this index calculation:
> - Root Node: Index 0.
>- Left Child of node at index `i`: Index `2i + 1`.
>- Right Child of node at index `i`: Index `2i + 2`.
>- Parent of node at index `i`: Index `⌊(i−1) / 2⌋`.


***
## Insertion

To insert a new element into the heap:
1. Add the element at the next available position in the array (maintaining the almost-perfect property).
2. Bubble up the element by swapping it with its parent (maintaining the tournament tree property).
```c
void insert(int heap[], int *size, int value) {
    heap[(*size)++] = value;  // Add at the next available position
    int i = *size - 1;
    while (i > 0 && heap[(i - 1) / 2] > heap[i]) {  // Bubble up
        swap(&heap[(i - 1) / 2], &heap[i]);
        i = (i - 1) / 2;
    }
}
```
![[heap_insert.gif]]


***
## Extract Minimum

To extract the minimum element:
1. Remove the root element (minimum).
2. Replace the root with the last element in the array.
3. Bubble down the new root by swapping it with the smaller of its children until the heap property is restored.
```c
int extract_min(int heap[], int *size) {
    int min = heap[0];
    heap[0] = heap[--(*size)];  // Replace root with the last element
    int i = 0;
    while (2 * i + 1 < *size) {  // Bubble down
        int left = 2 * i + 1, right = 2 * i + 2, smallest = left;
        if (right < *size && heap[right] < heap[left])
            smallest = right;
        if (heap[i] <= heap[smallest])
            break;
        swap(&heap[i], &heap[smallest]);
        i = smallest;
    }
    return min;
}

```


***
## Summary

Convenient structure to implement a priority queue.
	*`PriorityQueue` in Java, `std::priority_queue` in C++*

`find_min` in `O(1)`, other operations in `O(log n)`.
