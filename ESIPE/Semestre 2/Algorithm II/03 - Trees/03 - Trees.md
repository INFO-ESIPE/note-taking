[[Algorithm II]]
***
**Table of Contents**
```table-of-contents
```

***
## Concept

A tree is a hierarchical data structure composed of:
- **Nodes**: Elements of the tree.
- **Edges**: Unidirectional links connecting nodes.

A tree necessarily begins with a **root node**. Each node is either an **inner node (inode)**, or a **leaf node** (does not have child nodes).
![[ESIPE/Semestre 2/Algorithm II/03 - Trees/tree.png]]

A tree must be **connected and acyclic** (All nodes are linked, and the tree contains no cycle). Nodes are **organised by a children-parent relationship** (every node besides the root node has one parent).
	*Nodes sharing the same parents are called siblings*
![[tree_errors.png]]

### Metrics

**Distance** between two nodes is the amount of links we have to traverse in order to reach one from the other.
**Depth** of a node is the distance between this node and the root node.
**Level `k`** represents the set of nodes which shares the same depth `k`.
**Height** of a node is the maximal distance with any of its descendant.
![[tree_example.png]]
> [!Info] 
> The height of the tree is the height starting from the root node (3 in this case)

### Type of trees

As we can see, our trees are **oriented** and each nodes can possess **up to two children** (one on the left, one on the right). That is because we will focus on [[03 - Trees#Binary Search Tree (BST)|binary search trees]] later on, but rules can be less strict...
	*For instance, a tree that is not oriented (same as a non-oriented graph, in fact)*


***
## Binary tree

As we explained right above, a **binary tree** is a tree where each node has at most two children:
- **Left Child**
- **Right Child**
> In this class, our trees will be oriented (because it's the most common situation we have to face), but this isn't a mandatory characteristic for binary trees...

A binary tree is called **strictly binary** if every node has either 0 or 2 children.
A binary tree is called **perfect** if all levels are completely filled.
![[binary_types.png]]


***
## Implementation

A node is simply a value and two pointers: one per possible child. A tree is a pointer to the root node (or NULL if the tree is empty):
```c
typedef struct _node {
	int data;
	struct _node *left;  // points to left child (or NULL)
	struct _node *right; // points to right child (or NULL)
} node;
```
![[tree_impl.png]]


***
## Traversal

Like for lists or arrays, traversing a tree means visiting all its nodes. We have two generic ways of doing it with trees.

### Depth-First Traversal (DFT)

DFT explores **as far down a branch as possible before backtracking**. This method of traversal has three variations:
1. **Preorder:** Visit the node, then its left child, and finally its right child.
```c
void preorder(node *t) {
    if (t == NULL) return;
    printf("%d ", t->data);  // Process node
    preorder(t->left);       // Process left child
    preorder(t->right);      // Process right child
}
```

2. **Inorder:** Visit the node's left child, then the node itself, and finally its right child.
```c
void inorder(node *t) {
    if (t == NULL) return;
    inorder(t->left);       // Process left child
    printf("%d ", t->data); // Process node
    inorder(t->right);      // Process right child
}
```

3. **Postorder:** Visit the node's left child, then the node's right child, and finally the node itself. 
```c
void postorder(node *t) {
    if (t == NULL) return;
    postorder(t->left);       // Process left child
    postorder(t->right);      // Process right child
    printf("%d ", t->data);   // Process node
}
```

Depending on which strategy our DFT applies, the result of the traversal will differ. Let's traverse the following tree with DFT:
![[tree_traverse.png]]
> Here are the different outputs we get:
> - Preorder: `A, B, D, E, H, I, L, C, F, G, J, K`
> - Inorder: `D, B, H, E, L, I, A, F, C, J, G, K`
> - Postorder: `D, H, L, I, E, B, F, J, K, G, C, A`

Here is how to implement preorder in an iterative manner (via a stack):
```c
void preorder_iterative(node *t) {
    stack *s = create_stack();
    if (t != NULL) push(s, t);
    while (!is_empty(s)) {
        node *n = pop(s);
        printf("%d ", n->data);  // Process node
        if (n->right != NULL) push(s, n->right);
        if (n->left != NULL) push(s, n->left);
    }
    free_stack(s);
}
```

### Breadth-First Traversal (BFT)

Unlike the previous method, BFT visits nodes **level by level from left to right**, making it ideal for wide exploration. It requires a **queue** to process nodes.
![[tree_bft.png]]
> We obtain `A, B, C, D, E, F, G, H, I, J, K, L`

The simplest way to implement it is iteratively (via a queue):
```c
void breadth_first(node *t) {
    queue *q = create_queue();
    if (t != NULL) enqueue(q, t);
    while (!is_empty(q)) {
        node *n = dequeue(q);
        printf("%d ", n->data);  // Process node
        if (n->left != NULL) enqueue(q, n->left);
        if (n->right != NULL) enqueue(q, n->right);
    }
    free_queue(q);
}
```


***
## Ordered Operations

In general, our nodes are arranged to facilitate ordered operations.
For instance, the smallest value is the leftmost node, while the greatest is the rightmost one (this is something we will see in greater details with [[03 - Trees#Binary Search Tree (BST)|binary search trees]]).

### Min & Max

Knowing those, we can make simple functions that retrieves the largest and smallest value of our tree:
```c
node* find_min(node *t) {
    while (t->left != NULL) t = t->left;
    return t;
}

node* find_max(node *t) {
    while (t->right != NULL) t = t->right;
    return t;
}
```

### Floor & Ceiling

**Floor(x)** is the largest value `≤ x`, and **ceiling(x)** is the smallest value `≥ x`.
![[floor_ceiling.png]]

For floor, the logic is the following:
1. If x = root’s key, return the root.
2. If x < root’s key, recurse into the left subtree.
3. If x > root’s key, check the right subtree; if no result, return the root.
```c
node* floor(node *t, int x) {
    if (t == NULL) return NULL;
    if (x == t->data) return t;
    if (x < t->data) return floor(t->left, x);
    node *f_right = floor(t->right, x);
    return (f_right != NULL) ? f_right : t;
}
```

### Selecting the `k`th Smallest Element

Efficiently find the `k`th smallest element by storing the size of each subtree in each node.
![[tree_select.png]]
> [!Caution] This means the values must be maintained for each insertion/deletion of nodes

We do the following:
1. Let `L` be the left subtree, `R` the right subtree.
2. If `k ≤ L.size`, recurse into the left subtree.
3. If `k = L.size + 1`, return the root.
4. Otherwise, recurse into the right subtree with `k − L.size − 1`.
```c
node* select(node *t, int k) {
    if (t == NULL) return NULL;
    int left_size = (t->left) ? t->left->size : 0;
    if (k <= left_size) return select(t->left, k);
    if (k == left_size + 1) return t;
    return select(t->right, k - left_size - 1);
}
```


***
## Binary Search Tree (BST)

A Binary Search Tree is a binary tree where:
- The left subtree of a node contains keys less than the node’s key.
- The right subtree of a node contains keys greater than the node’s key.
- Both subtrees are themselves BSTs.
![[bst.png]]
> [!Info]
> As the name implies, the purpose is to organise our data in a way that we can easily manipulate the tree with the lowest complexity possible. The time complexity of operations on the binary search tree is linear with respect to the height of the tree...

### Implementation

Here are two ways of doing it, with and without value:
```c
// With both key and value
typedef struct _node {
	KEY_TYPE key;
	VALUE_TYPE value;
	struct _node *left;
	struct _node *right;
} node;

// Classic approach
typedef struct _node {
	int data;
	struct _node *left;
	struct _node *right;
} node;
```
![[bst_impl.png]]

### Search

Let's say we want to find `5` in the [[03 - Trees#Binary Search Tree (BST)|aforementioned BST]], we start at the root node and traverse like so:
- If `node < 5`, we go right
- If `node > 5`, we go left
- If `node = 5`, we stop
```c
node *find_bst(node *t, int elt) {
	node *ptr = t;
	while (ptr != NULL && ptr->data != elt) {
		if (ptr->data > elt) ptr = ptr->left;
		else ptr = ptr->right;
	}
	return ptr;
}

node *find_bst_rec(node *t, int elt) {
	if (t == NULL) return NULL;
	else if (t->data == elt) return t;
	else if (t->data > elt) return find_bst_rec(t->left, elt);
	return find_bst_rec(t->right, elt);
}
```
> The value is "not found" if we can't keep traversing the tree (e.g. `13` can't be found because `15` is a leaf node that does not lead to `13`).

### Insert

We will follow this logic:
1. Start at the root.
2. Compare the new key with the current node’s key:
    - If smaller, move to the left subtree.
    - If larger, move to the right subtree.
3. Insert the new node when a `NULL` position is found (leaf passed).
```c
node *insert_bst_rec(node *t, int elt) {
	if (t == NULL) t = create_node(elt, NULL, NULL);
	else if (t->data > elt) t->left = insert_bst_rec(t->left, elt);
	else if (t->data < elt) t->right = insert_bst_rec(t->right, elt);
	return t;
}
```
![[insert_bst.png]]

### BST Structure

What is the relation between the amount of nodes and the tree's height?
Several structure are possible depending on our insertion's order:
![[bst_structures.png]]
> [!Question]
> All those structures are possible, and they contain the same data. Which one offers the best complexity?

1 is the best case scenario: `h ≈ log2(n)`, while 3 is `h ≈ n`.
We can solve this problem by using an [[03 - Trees#AVL Tree|AVL tree]] instead.

### Deletion

Let's delete the smallest value of the BST. 
If the node has no child, we just unlink it from its parent (parent's `t->left` set to `NULL`), and then free the node.

Now, what if a node (minimal or not, doesn't matter) has one child? Here is the approach:
```c
node *remove_min_rec(node *t) {
	if (t == NULL) return NULL;
	if (t->left == NULL) {
		node *to_remove = t;
		t = t->right;
		free(to_remove);
	} else t->left = remove_min_rec(t->left);
	return t;
}
```
> - Go leftmost
> - If the smallest value has a child, we **replace the small node** (parent node's left child) **by the small node's right child**
![[bst_delete.png]]

Now, what if instead of deleting the minimal node, we want to delete an inner node with two children? We process like so:
![[bst_delete_1.png]]
![[bst_delete_2.png]]
> [!Note]
> Code will be done in laboratory class


***
## AVL

An Adelson-Velsky Landis tree is simply a **self-balancing BST**.
To be considered an AVL, our BST must comply to two rules:
- For every node, the **heights of the left and right subtrees differ by at most 1**.
- The tree **must be rebalanced using rotations after insertions or deletions**.
![[avl.png]]

A node has the following structure:
```c
typedef struct _node {
	int data;
	int height;
	struct _node *left;
	struct _node *right;
} node;
```

### Rotation

Rotations are tree transformations used to restore balance in an AVL tree after insertion/deletion of a node.
Four types of rotations in total

#### Right rotation (single)

Occurs when a left-heavy tree (`Balance Factor =+2`) has a left-heavy child (`Balance Factor = +1`).
1. Promote the left child to root.
2. Demote the original root to the right child.
3. Adjust the subtrees accordingly.

```c
node* rotate_right(node *y) {
    node *x = y->left;
    y->left = x->right;
    x->right = y;

    update_height(y);
    update_height(x);

    return x;  // New root
}
```
![[rotate_right.png]]

#### Left rotation (single)

Similar idea to right rotation, but for the left side...

```c
node* rotate_left(node *x) {
    node *y = x->right;
    x->right = y->left;
    y->left = x;

    update_height(x);
    update_height(y);

    return y;
}
```

#### Left-Right Rotation (double)

Occurs when a **left-heavy** tree (`Balance Factor = +2`) has a **right-heavy** child (`Balance Factor = −1`).
1. Perform a left rotation on the left child.
2. Perform a right rotation on the root.
```bash
Before:
       z
      / \
     x   D
    / \
   A   y
      / \
     B   C

After Left Rotation on x:
       z
      / \
     y   D
    / \
   x   C
  / \
 A   B

After Right Rotation on z:
       y
      / \
     x   z
    / \ / \
   A  B C  D
```
> [!note]
> We will implement this during labs

#### Right-Left Rotation (double)

Similar idea but opposite


### Maintaining Balance in AVL Trees

How to insert a value at the root:
1. Insert the node as in a normal BST.
2. Update the heights of all nodes along the path from the insertion point to the root.
3. Perform rotations if the **balance factor** of any node becomes −2 or +2.

```c
node *insert_at_root(node *t, elt) {
	if (t == NULL) t = create_node(elt, NULL, NULL);
	else if (t->data > elt) {
		t->left = insert_at_root(t->left, elt);
		t = rotate_right(t);
	} else if (t->data < elt) {
		t->right = insert_at_root(t->right, elt);
		t = rotate_left(t);
	}
	return t;
}
```
> [!info]
> The balance factor of a node is the height difference between his left subtree and right subtree.
> Refresher: An empty tree's height is -1

Here is a strategy to follow when inserting a node on the tree:
```c
node *insert_balanced(node *t, int elt) {
	if (t == NULL) return create_node(elt, NULL, NULL);
	else if (t->data > elt) t->left = insert_balanced(t->left, elt);
	else if (t->data < elt) t->right = insert_balanced(t->right, elt);
	update_height(t);
	t = rebalance(t);
	return t;
}
```
![[avl_insert.png]]


***
## Other types of trees

### Red-Black Trees

Self-balancing BST where each node is assigned a colour (red or black). The tree maintains balance using a set of rules involving these colours.
![[redblack_tree.png]]

It follows those properties:
- Every node is either **red** or **black**.
- The root node is always **black**.
- Red nodes cannot have red children (**no consecutive red edges**).
- Every path from the root to a `NULL` pointer contains the same number of black nodes (**black-height invariant**).

The height of a Red-Black Tree is at most: 
$$Height≤2log⁡2(n)$$
> [!info]
> This guarantees `O(log n)` for all operations (search, insert, delete).
> Red-Black trees are implemented in Java as `TreeMap`, and C++ as `std::map`.


Similarly to AVL, Red-black trees also leverage a rotation mechanism to stay balanced. However, rotations are more relaxed here than they are for AVLs (easier to implement!)


### B-Trees

Self-balancing search tree optimised for systems that read and write large blocks of data, such as databases and file systems. Unlike binary trees, B-Trees can have more than two children.
![[btree.png]]

It follows those properties:
- A node contains up to `M − 1` keys and `M` children (where `M` is a fixed order of the tree).
- All leaves are at the same level.
- Internal nodes duplicate keys to guide the search.
- Keys in a node are sorted, and subtrees follow the search tree property:
    Keys in the left subtree are less than the current node’s keys.
    Keys in the right subtree are greater than the current node’s keys.

The height of a B-Tree is at most: 
$$Height≤logM/2​(n)$$

An insertion works as follows:
- Add the key to the correct leaf.
- If a node overflows (>M−1> M-1>M−1 keys), split the node and propagate upward.
![[btree_insert.png]]
