[[Algorithm II]]
***
**Note:** For this semester, we will study most data structures (Stack, Queue, Trees, Hash tables, Heap, Linked Lists). We will study Graphs in a later semester. We keep on using C.
***
**Table of Contents**
```table-of-contents
```

***
## Abstract Data Types (ADTs)

An Abstract Data Type is a high-level description of a data structure:
- **Values**: Defines the domain (e.g., integers, strings).
- **Operations**: Describes the behaviours (e.g., push, pop, enqueue, dequeue).

An ADT is independent of any programming language as it is explained in a declarative way (in documentation, with formal logic). Furthermore, only the concept should be explained, while the implementation (interface) is up to the programmers.


***
## Linked List

Structure composed of links. Each link contains:
- A value (what we store)
- A pointer towards the next link (or `NULL` if last link of the list)
```c
typedef struct _link {
	int           data;
	struct _link* next;
} link;
```

A list containing (4, 6, 7) looks like:
![[linkedlist.png]]

It virtually allows for the same operations as an array, but the complexity varies when it comes to modification.
```c
// Similar to how we traverse an array, but instead of relying on an
// index, we jump to the next pointer until its not NULL.
void display_list(link *lst) {
	link *ptr;
	for (ptr = lst; ptr != NULL; ptr = ptr->next)
		printf("%d ", ptr->data);
	printf("\n");
}

link *find_list(link *lst, int elt) {
	link *ptr;
	while (ptr != NULL && ptr->data != elt) // better than for + if
		ptr = ptr->next;
	return NULL;
}
```

### Modification

Let's make a function that inserts a new element at the head of our list:
```c
void insert_first_essai(link *lst, int elt) {
	link *new_lnk = (link*)malloc(sizeof(link));
	new_lnk->data = elt;
	new_lnk->next = lst;
	lst = new_lnk; // Oops
}

void main() {
	link *list = create_list();
	display_list(list);
	insert_first(list, 9);
	display_list(list);
}
```
```bash
$ ./a.out
1 2 3
1 2 3       # And where is 9 ?
```
> [!bug] 
> As we can see, the insertion did not work as expected...

Here is what happens with our current code:
![[linkedlist_problem.png]]
> [!info]
> Local variables (and arguments) of a function are stored in the context, which is on the thread's **stack**.

We can solve this issue by returning the list after the branching is fixed:
```c
link *insert_first(link *lst, int elt) {
	link *new_lnk = (link*)malloc(sizeof(link));
	new_lnk->data = elt;
	new_lnk->next = lst;
	return new_lnk; // here (and signature)
}

// ...

link *list = createlist();
display_list(list);
ma_liste = insert_first(list, 9);
display_list(list);
```
![[linkedlist_solution.png]]
> [!Info] 
> Function that inserts in head must return a pointer towards the first cell of the "new" list, and the program calling it must re-assign it. This is because modification of list is **done on the stack**, we'll see [[01 - Stack, Queue, Linked List#Stack|later]] that we do not encounter this issue when implementing a stack since things are **done on the heap**.

We have a similar situation when we want to delete the head element of the list:
```c
link *remove_list(link *lst, int elt) {
	if (lst == NULL) return NULL; // case 1: list is empty
	
	/* main loop: search for elt */
	link *ptr = lst;
	link *pred = NULL;
	while (ptr != NULL && ptr->data != elt) {
		pred = ptr;
		ptr = ptr->next;
	}
	
	if (pred == NULL) { // c2: remove first if pred was never changed
		link *tmp = lst;
		lst = lst->next;
		free(tmp);
	} else if (ptr != NULL) { // c3: remove ptr if not at end of list
		link *tmp = ptr;
		pred->next = ptr->next;
		free(tmp);
	}
	return lst;
}
```

### Variations

What we have studied so far are **Singly linked list**, but this isn't convenient if we want to go backwards, nor if we want to access a node that is located at the tail of the list (we have to traverse the entire list to access it).

A very common variation is the **Doubly linked list**, where each link contains both a pointer to the previous and the next link. This allows to go backwards.

Another variation is the **Circular linked list**, where the tail link points to the first link, allowing to go back to the beginning of the list (hence the "circular" naming).
	*In the case of a circular doubly linked list, the first node also points to the last node of the list. It looks like this:*
![[doubly.png]]
> [!info]
> `LinkedList` (deprecated) in Java, List in C++


***
## Stack

A **Last-In First-Out** (LIFO) data structure.
	*Try to imagine a stack of plates, you can only do two major operations: extract (pop) the upmost plate, or insert (push) a new plate on top of the stack.*

A first way to implement it is with an array:
```c
#define MAX_SIZE 100
typedef struct {
    int tab[MAX_SIZE];
    int size;
} stack;

stack *create_stack(void) {
	stack *s = (stack*)malloc(sizeof(stack));
	s->size = 0;
	return s;
}

void push_stack(stack *s, int elt) {
    s->tab[s->size] = elt;
    s->size++;
}

int pop_stack(stack *s) {
    s->size--;
    return s->tab[s->size];
}

int size_stack(stack *s) {
	return s->size;
}

void free_stack(stack *s) {
	free(s);
}
```
> [!Info] 
> As we can see, our stack has a **fixed size** (like the memory stack used by our thread, in fact). If we need to enlarge the stack, we have to double the array size when its full (which isn't convenient).

To solve the static (fixed size) issue, we use a linked list that works on dynamic memory allocation:
```c
typedef struct link {
    int data;
    struct link* next;
} link;

typedef struct {
    link* top;
    int size;
} stack;

void push_stack(stack *s, int elt) {
    link* new_lnk = (link*)malloc(sizeof(link));
    new_lnk->data = elt;
    new_lnk->next = s->top;
    s->top = new_lnk;
    s->size++;
}

int pop_stack(stack *s) {
    link* tmp = s->top;
    int elt = tmp->data;
    s->top = s->top->next;
    free(tmp);
    s->size--;
    return elt;
}

// ...
```


***
## Queue

A **First-In First-Out** (FIFO) data structure.
	*Like a regular queue at the bakery, people who joined the queue first are served first. We can only do two major operations: insert (enqueue) a client to the end of the queue, or extract (dequeue) a client from the head of the queue.*
![[ESIPE/Semestre 2/Algorithm II/01 - Stack, Queue, Linked List/queue.png]]

We implement it with a linked list:
```c
typedef struct link {
    int data;
    struct link* next;
} link;

typedef struct {
    link* first;
    link* last;
    int size;
} queue;

void enqueue_queue(queue *q, int elt) {
    link* new_lnk = (link*)malloc(sizeof(link));
    new_lnk->data = elt;
    new_lnk->next = NULL;
    if (q->last != NULL) q->last->next = new_lnk;
    else q->first = new_lnk;
    q->last = new_lnk;
    q->size++;
}

int dequeue_queue(queue *q) {
    link* tmp = q->first;
    int elt = tmp->data;
    q->first = q->first->next;
    if (q->first == NULL) q->last = NULL;
    free(tmp);
    q->size--;
    return elt;
}
```


***
## Priority Queue

Unlike a traditional queue, this one dequeues elements **based on their priority rather than the order of insertion**.
	*Two major operations: 
	- insert, which adds an element with a defined priority 
	- remove-min, which removes and return the element with the highest priority (smallest value in min-priority queues)*

Priority queues are commonly implemented using **binary heaps** for efficient insertion and removal. This will be covered in a future lecture.


***
## Summary

| Data Structure     | Insertion | Deletion | Access | Applications                                    |
| ------------------ | --------- | -------- | ------ | ----------------------------------------------- |
| **Stack (LIFO)**   | O(1)      | O(1)     | O(1)   | Function calls, parsers, undo operations.       |
| **Queue (FIFO)**   | O(1)      | O(1)     | O(1)   | Scheduling, simulations, data buffering.        |
| **Priority Queue** | O(log n)  | O(log n) | O(1)   | Pathfinding, top-K problems, prioritized tasks. |
