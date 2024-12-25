[[Algorithm I]]
***
**Table of Contents**
```table-of-contents
```

***
## Selection Sort

Iteratively selects the smallest unsorted element and swaps it into its correct position:
1. Traverse the array
2. For each iteration `i`, find the minimum element index in the unsorted portion
3. Swap the minimum element with the element at `i`.
```c
void swap(int *a, int *b) {
	int c = *a; 
	*a = *b; 
	*b = c;
}

int less(int a, int b) {
	return a < b;
}

void selection_sort(int arr[], int size) {
    int i;
    for (i = 0; i < size - 1; i++) {
        int min = i, j;
        for (j = i + 1; j < size; j++) {
            if (less(arr[j], arr[min]))
                min = j;
        }
        if (i != min)
            swap(&arr[i], &arr[min]);
    }
}
```
> O(n²) complexity, usually not the fastest sorting algorithm

Looks like:
![[sort_select.png]]


***
## Insertion Sort

Builds a sorted array by inserting elements into their correct position one at a time:
1. Traverse the array
2. For each element, move it to its correct position in the sorted portion of the array
```c
void insertion_sort(int arr[], int size) {
    int i;
    for (i = 1; i < size; i++) {
        int elt = arr[i], j = i;
        while (j > 0 && less(elt, arr[j - 1])) {
            arr[j] = arr[j - 1];
            j--;
        }
        arr[j] = elt;
    }
}
```
> O(n²) in worst case scenario, but O(n) for a nearly-sorted array.
> **Convenient algorithm for partially sorted array**.

Looks like:
![[sort_insert.png]]


***
## Quicksort

Divide-and-conquer algorithm that splits the array into two subarrays like so:
1. Select a **pivot** element
2. Partition the array into the two subarrays: elements **smaller than the pivot** and elements **larger than the pivot**
3. Recursively apply quicksort on the left and right subarrays
![[pivot.png]]
> where the pivot appears in lime

### Partitioning

Partitioning divides the array such that the pivot is in its final position, with smaller elements on the left and larger elements on the right.

We define `i = 0` (beginning of the array) and `j = size - 1` (end of the array).
Until `i` and `j` cross each other:
- Increment `i` until an element greater than the pivot is found.
- Decrement `j` until an element smaller than the pivot is found.
- Swap elements at `i` and `j`.
![[quicksort.gif]]
> After they have crossed each other, we can now place the pivot in its final position. Now, all elements on the left subarray are smaller than the pivot, and elements on the right one are greater.

Here is a code to do so:
```c
int partition(int arr[], int lo, int hi) {
    int i = lo + 1, j = hi;
    while (1) {
	    // Increment i while arr[i] < arr[lo]
        while (less(arr[i], arr[lo]) && i < hi) i++;

		// Decrement j while arr[j] > arr[lo]
        while (less(arr[lo], arr[j]) && j > lo) j--;

		// Do not exit the loop until i and j crosses each other
        if (i >= j) break;

        swap(&arr[i], &arr[j]);
        i++;
        j--;
    }
    swap(&arr[lo], &arr[j]); // Switch arr[j] with pivot
    return j;                // Return pivot index
}
```
> Here lo and hi can be custom values, not necessarily 0 and size-1

==Do not partition arrays of length 1...==

### Sorting and complexity

After partitioning, quicksort recursively sorts the subarrays on either side of the pivot. The algorithm is way more straightforward than the one above:
```c
void quicksort(int arr[], int lo, int hi) {
    if (lo >= hi) return;
    int pivot = partition(arr, lo, hi);
    quicksort(arr, lo, pivot - 1);
    quicksort(arr, pivot + 1, hi);
}
```

The best scenario is when our **pivot divides the array evenly**, we then obtain an `O(n log n)` complexity, which is great. In fact, the average scenario provides more or less the same complexity...
The **worst case** scenario is when the array we sort is **reverse-sorted** or **already sorted**. This end up in an `O(n²)` complexity, but again, this is a rare and avoidable situation you can avoid...
	*This happens because our pivot will be an extreme value, which doesn't split our array into even subarrays*

### Optimise pivot selection

Several strategies:
- Median-of-three: Choose the median of the first, middle, and last elements.
- Random Pivot: Helps avoid worst-case scenarios.
> Both are in `O(n log n)`


***
## Mergesort

Another divide-and-conquer algorithm, stable and offering a predictable `O(n log ⁡n)` complexity. 
It works by dividing the array into two halves, sorting them recursively, and then merging the sorted halves.
![[merge.png]]
> Unlike quicksort, half can contain any value

### Merge

A function that **combines two sorted arrays** into a single sorted array:
```c
void merge(int arr[], int aux[], int lo, int mid, int hi) {
    int i = lo, j = mid + 1, k;
    for (k = lo; k <= hi; k++) aux[k] = arr[k];

    for (k = lo; k <= hi; k++) {
        if (i > mid) arr[k] = aux[j++];
        else if (j > hi) arr[k] = aux[i++];
        else if (less(aux[j], aux[i])) arr[k] = aux[j++];
        else arr[k] = aux[i++];
    }
}
```

Here, `lo`, `mid` and `hi` are used to separate our array into our two zones:
- First half: `t[lo]` ... `t[mid]`
- Second half: `t[mid+1]` ... `t[hi]`
`aux[]` is an auxiliary array provided to help the merging.

Once merged, values between `t[lo]` and `t[hi]` are sorted, we successfully "merged" them.

### Sorting and complexity

In fact, `aux[]` is allocated once for first call, and freed at the end:
```c
void merge_sort(int arr[], int size) {
    int *aux = (int *)malloc(size * sizeof(int));
    sort(arr, aux, 0, size - 1);
    free(aux); // of course
}

void sort(int arr[], int aux[], int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    sort(arr, aux, lo, mid);
    sort(arr, aux, mid + 1, hi);
    merge(arr, aux, lo, mid, hi);
}
```

As we said above, the complexity is an `O(n log n)` predictable complexity. However, there is also an `O(n)` "space complexity" for the `aux[]` array.

Merge sort is usually what languages like C, C++ or Java chose for their generic algorithms, as it fits generic situations. However, there are situations where it isn't necessarily the best choice...

### Optimise

Some optimisations:
- If the largest element in the left half is less than the smallest in the right half, skip merging.
- Avoid copying: Saves some time but doesn't prevent the usage of an auxiliary array
```c
void sort(int arr[], int aux[], int lo, int hi) {
	if (lo >= hi) return;
	int mid = lo + (hi-lo) / 2;
	sort(aux, arr, lo, mid);     // swap aux and arr
	sort(aux, arr, mid + 1, hi); // swap aux and arr
	merge(t, aux, lo, mid, hi);
}

void merge(int arr[], int aux[], int lo, int mid, int hi) {
    // No need for this anymore
    // int k;
    // for (k = lo; k <= hi; k++) aux[k] = arr[k];

	int i = lo, j = mid + 1;
    for (k = lo; k <= hi; k++) {
        if (i > mid) arr[k] = aux[j++];
        else if (j > hi) arr[k] = aux[i++];
        else if (less(aux[j], aux[i])) arr[k] = aux[j++];
        else arr[k] = aux[i++];
    }
}
```
> Where `t` and `aux` both contains the same values at the beginning


***
## Comparison of Algorithms


| Algorithm      | Best Case   | Worst Case  | Stable | In-Place |
| -------------- | ----------- | ----------- | ------ | -------- |
| Selection Sort | O(n²)       | O(n²)       | No     | Yes      |
| Insertion Sort | O(n)        | O(n²)       | Yes    | Yes      |
| QuickSort      | O(n log ⁡n) | O(n²)       | No     | Yes      |
| MergeSort      | O(n log ⁡n) | O(n log ⁡n) | Yes    | No       |

**Quicksort**: Typically the fastest but not stable
**Mergesort**: Best for stability and predictable performance
**Insertion Sort**: Effective for small or nearly sorted datasets
**Selection Sort**: Simple but inefficient for large datasets


***
## Real World

C++:
- `std::sort`: Introsort, a hybrid of Quicksort, Heapsort, and Insertion Sort.
- `std::stable_sort`: A modified Mergesort ensuring stability.

Python:
`sorted()` and `list.sort()` both uses **Timsort**, hybrid of Mergesort and Insertion Sort (very performant).

Java:
Uses Timsort for objects, and an upgraded quicksort for primitive types.
	*In fact, the API says: "the algorithm used by sort does not have to be a mergesort, but it does have to be stable”*

