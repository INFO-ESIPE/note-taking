[[Algorithm II]]
***
**Table of Contents**
```table-of-contents
```

***
## Concept

Dictionaries (A.K.A. associative array, map, symbol table) abstractly represent **key-value pairs** and allow efficient operations such as insertion, retrieval, and deletion.
	*We can obtain a value via its associated unique key*

A lot of use case:
- File systems: Mapping filenames to file attributes.
- Text indexing: Mapping words to occurrences in a text.
- Compilers: Mapping variable names to types.
- DNS/Reverse DNS: Mapping domain names to IP addresses.
- Relational databases: Mapping primary keys to rows.

Implementation in most modern programming language: `dict` in Python, `Map<K, V>` interface in Java, `std::map` and `stf::unordered_map` in C++, `HashMap<K, V, S = RandomState>` in Rust...

A dictionary should allow for the following basic operations:
- **Insert**: Add a key-value pair, or update the value if the pair already exists.
- **Find**: Retrieve the value associated with a given key, or return NULL.
- **Remove**: Delete a key-value pair.
- **Iterate**: Loop through all keys, possibly in order.
- **Query**: Get the smallest key, largest key, or keys in a range.


***
## Naive Implementation

Two naive implementation that don't really work well:
1. Use a linked list where each link stores the pair.
![[map_list.png]]
	Inconvenient: `O(n)` complexity for every operation, not good

2. Use an array sorted by keys that we traverse through binary search.
![[map_array.png]]
	*Inconvenient: Even though find operation is less costly (only `O(log n)`), it remains too high and Insertions are still expensive*

That won't do... Is there a better alternative?

Let's say we want to implement a key-value pair where keys are telephone numbers and values are the name of the holder. How do we implement this?
If we exclude the space concern, using the phone number as the array index where the associated value is offers best complexity: `O(1)` for insertion, search and deletion.
```c
char *map[10000]

insert(int num, char *nom) {
	map[num] = nom;
}

find(int num) {
	return map[num];
}

remove(int num) {
	map[num] = NULL;
}
```
> [!Caution] 
> The approach is good, but it requires a massive array... Let's fix this space problem bellow


***
## Hash Tables

Hash tables store key-value pairs in a **fixed-size (M) array**. A **hash function maps keys to array indices**, ensuring constant-time operations in most cases.
![[hashmap.png]]
> [!Info] 
> Using `mod` ensures we never go beyond the array's size boundary.

### Collision

This works great, but it comes with (one of) the major [[07 - Hashing Functions#Principle|concern]] introduced by hashing functions: **What if two different values outputs the same hash?**

#### 1. Chaining
*First solution: Each index will possess its own linked list*

Elements resulting in the same hash (sharing the same index in the array) must be stored in a shared **bucket**. 
Buckets are here to hold colliding values (e.g., linked list):
![[buckets.png]]
> Buckets becomes linked list if they encounter a collision

In the case of chaining
```c
typedef struct _link {
	int           data;
	struct _link* next;
} link;

typedef struct _hash_table {
    link **buckets; // Array of linked lists
    int M;          // Number of buckets
} hash_table;
```

#### 2. Open Addressing
*Second solution: Store value in the cell itself, and rely on a probe sequence to find an unoccupied slot if a collision occurs*

Another collision resolution strategy: all elements are stored directly in the hash table array itself, **without using additional data structures like linked lists**. 
When a collision occurs (two keys hashing to the same index), the algorithm **probes for the next available bucket in the array**.
1. **Hash the Key**: Use the hash function to find the initial index.
2. **Probe**: If the bucket at that index is occupied, look for the next available slot according to a probing scheme.
	*A simple solution (used in the example bellow) is linear probing, resulting in a fixed interval (`interval=1`) for instance*
3. **Wrap Around**: The table is treated as circular; if you reach the end of the array, you continue from the beginning.
![[openaddressing.png]]
> [!Important] 
> Here, John Smith and Sandra Doe collides, so the later is moved to the next available bucket (153). Now, Ted Baker has a unique hash which results in 153, but since this slot is already taken due to probing, it is moved to bucket 154.

#### Comparing both solutions

As the table fills, performance of linear probing will decrease. This is due to the **load factor** of our probe sequence approaching 1 as our table fills. 
![[collision_compare.png]]


***
## Hashing

Efficiency of our hash table relies heavily on the hashing function's properties:
- Uniform Distribution: Keys should be evenly distributed across buckets.
	*Else we will end up with a lot of collisions*
- Determinism: Same key should always hash to the same index.
	*Fundamental characteristic of hashing, in fact*
- Speed: Hashing computation should be efficient.

A simple hashing function we can use in labs:
```c
unsigned hash(char *key) {
    unsigned h = 0;
    while (*key) {
        h = 31 * h + *key++;
    }
    return h;
}
```
> Generates an index by combining the ASCII values of the characters in the key.

Note that cryptographic hashes (SHA-256, SHA-1, MD5...) are generally not a good option for hash tables due to computational cost, unless collision resistance is extremely critical and efficiency is expandable.


***
## Performance

With `N` the number of elements, `M` the number of buckets, and the load factor (`α=N/M`) being the average number of elements per bucket: Lookup, insertion, and deletion take `O(1)` **on average**.

Performance considerably relies on the hashing function it uses.


***
## Drawbacks

In a nutshell, hash tables are relatively inefficient for ordered operations. Furthermore, resizing the table is extremely costly.