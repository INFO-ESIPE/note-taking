[[Concurrence]]
****

**Paradigms:**
- Concurrency: Running multiple calculations simultaneously (what we are doing since the beginning of this class).
- Parallelism: Splitting a calculation across multiple threads.
- **Vectorization (Vectorized Operations)**: Executing a single operation (e.g., addition) on multiple data points in memory using a single thread.
- Distributed Processing: Spreading a calculation across multiple machines.

**Single Instruction, Multiple Data (SIMD)** is a type of CPU that can perform an identical operation on several datapoints.
	*MMX, SSE, AVX for Intel/AMD
	NEON, Helium/MVE for ARM*


****
## SIMD

SIMD CPUs uses special registers (`xmm0`, `ymm0`, `zmm0`) to make a calculation on several **lanes**:
`PADDW xmm1, xmm2, xmm3` (128 bits registers)
![[lanes.png]]

An instruction indicates:
- Register size (64, 128, 256, 512 bits)
- Operation (ADD, SUB ...)
- Size of rows in the lane (8, 32, 64, 128 bits)

Let's take the aforementioned instruction again:
**`P`**`ADD`**`W`** `xmm1, xmm2, xmm3`:
1. Prefix:
	- `P` - 128 bits lanes (register width)
	- (VEX) `VP` - 256 bits lanes
	- (EVEX) `VP` - 512 bits lanes
2. Instruction: `ADD`
3. Row size:
	- `B` (Byte) - 8 bits
	- `W` (Word) - 32 bits
	- `D` (Double) - 64 bits
	- `Q` (Quad) - 128 bits

Our instruction will make a **packed addition (`ADD`)** on **128 bits long registers (`P`)**, where **each line is 32 bits (`W`)** long.
	*Each 128 registers is divided into four 32-bit lines, and each of those in `xmm2` is added to the corresponding 32-bit line in `xmm3`, storing the result in `xmm1`.*


As we can see, vectorized registers have a larger size than normal registers, as they need to be more efficient for read/write operations.
Instead of accessing an array byte per byte
```java
for(var i = 0; i < 16; i++) { // 16 lanes
	a[i] = b[i] + c;
}
```

We use intermediate vectors:
```java
var vb = *Vector.fromArray(b, 0); // read 16 values
var vc = *Vector.broadcast(c); // we put "c" in each lane
var va = vb.add(vc);
va.intoArray(a, 0); // writes the 16 values
```


****
## Vector API

The `jdk.incubator.vector` API provides a way to leverage SIMD capabilities.
A Vector is an **immutable register that has both a type and a number of lanes**:
- `Vector` - Interface
- `Byte|Short|Int|Long|Float|DoubleVector` - Specific type
- `VectorSpecies` - Defines the vector’s **data type** (byte|short|int|long|float|double) **and shape** (64|128|256|512 bits), allowing JIT to select the best hardware instruction
- `VectorMask` - (Immutable) Boolean Vector (bit set)
- `VectorShuffle` - (Ordered and immutable) sequence of `int` values called **source indexes**, where each source index numerically selects a source lane from a compatible Vector
	*Vector of indirection indexes, pretty much*


A Vector is **NOT an object with a memory address**. It is a **value based object**...
	*We manipulate it like an Object, but it works as an `int` (immutable). It corresponds to a CPU register*
```java
v1.add(v2); // does not change v1 !
```


**Operations requiring memory addresses may fail or produce unexpected results**. This means that when an operation relies on a direct memory address (such as pointer-based access or memory manipulation functions), it may either not work as expected or produce incorrect, unpredictable values. 
	*This issue often arises in managed environments like Java where **direct memory manipulation is abstracted away**, or in concurrent environments where the memory location can change unexpectedly.*

So:
- No `synchronized`, `wait()`, `notify()`/`notifyAll()`*
	*needs a lock's address*
- No `v1 == v2`
	*Object comparing, checks if they point to the same address, which is nonsense here*
- No `System.identityHashCode()`
	*Method that returns a hash code for an object based on its memory address*
- No `new WeakRef()`
	*`WeakReference` is a type of reference that does not prevent an object from being garbage collected. It's often used to allow the system to reclaim memory from unused objects while still keeping a reference that can be checked later*


****
## `VectorSpecies<*>`

They can be used like so:
```java
// Choses best size according to CPU
VectorSpecies<Integer> species = IntVector.SPECIES_PREFERRED;
// VectorSpecies<Integer> species = IntVector.SPECIES_128; (specific size)

IntVector v1 = IntVector.zero(species); 
IntVector v2 = IntVector.broadcast(species, 42); 
IntVector v3 = v1.add(v2);

// displays several "42" (128 bits = 4, 256 bits = 8, etc)
for(var i = 0; i < v3.length(); i++) { // length == lane count
	System.out.println(v3.lane(i));
}
```


****
## `VectorMask<*>`

Boolean mask indicating if a lane must be taken into account or not
	*Enables selective lane operations*

**Blending (mask):**
```java
/*
Creates a mask with a specific pattern defined by the binary value `0b11010`
Each bit represents a lane, where 0 is false (not blended) or 1 (blended)

Lane 4 | Lane 3 | Lane 2 | Lane 1 | Lane 0 
   1        1        0       1        0
*/
VectorMask<Integer> mask = VectorMarsk.fromLong(species, 0b11010);

/*
Combines (blends) v1 and v2 based on the mask. Result is stored in v3 (as each vector is immutable)
If a lane in the mask is `1`, the result will take the value from `v2` in that lane. Otherwise, it will take `v1` in that lane.

Let's assume:
- `v1 = [a, b, c, d, e]`
- `v2 = [f, g, h, i, j]`
Given the mask `0b11010` (binary pattern for lanes [4, 3, 2, 1, 0] as [1, 1, 0, 1, 0]):
- Lane 4 Take from `v2` (`f`)
- Lane 3 Take from `v2` (`g`)
- Lane 2 Take from `v1` (`c`)
- Lane 1 Take from `v2` (`i`)
- Lane 0 Take from `v1` (`a`)

Resumts: `v3 = [a, i, c, g, f]`
*/
IntVector v3 = v1.blend(v2, mask);
```

Another example:
```java
var species = IntVector.SPECIES_256;
var v1 = IntVector.zero(species); // fill with zeros
var v2 = IntVector.broadcast(species, 42);

VectorMask<Integer> mask = VectorMarsk.fromLong(species, 0b11010);

var v3 = v1.blend(v2, mask); // v1 or v2 depending on the mask
System.out.println(v3); // prints [0, 42, 0, 42, 42, 0, 0, 0]
```


Using `if` statements in a calculation-purpose loop is very slow, we'd rather use a vector:
```java
// SLOW
for(var i = 0; i < 8; i++) {
	if (a[i] < 3) {
		b[i] = a[i] + c;
	}
}

// BETTER
IntVector va = IntVector.fromArray(species, a, 0);
IntVector vt = IntVector.broadcast(3);
IntVector vc = IntVector.broadcast(c);
VectorMask<Integer> mask = va.lt(vt); // Mask based on ‘<’ (lower than) results
IntVector vb = va.add(vc, mask); // Addition depending on the mask
vb.intoArray(b, 0);
```


****
## `VectorOperators`

Enumeration that defines operations we can do on a Vector
- Binary and Unary tests
	*`EQ`(=\=), `NE`(!=), `LE`(<=), `LT`(<), `GE`(>=), `GT`(>)
	`IS_NAN`, `IS_NEGATIVE`...
	Returns a `VectorMask`*
- Conversions
	*`I2L`, `I2D`, `ZERO_EXTENDED_B2I`, etc*
- Unary, Binary, Ternary operations
	*`ADD`, `SUB`, `MUL`, `DIV`, `AND`, `OR`...
	`ABS`, `COS`, `SIN`...
	`FMA`, `BITWISE_BLEND`*


****
## `Vector<*>`

Interface implemented by `Byte|Short|Int|Long|Float|DoubleVector`. 
It allows for basic operations (`length()`, `lane(int index)`, `withLane(int index, value)`,
`addIndex(int scale)`), specific operations (`blend()`, `rearrange()`...), and **lanewise** operations.


**Specific Operations**

See a vector like another (32-bit -> 8-bit)
```java
reinterpretShape(VectorSpecies, int part);
```

Cast data to another vector
```java
castShape(VectorSpecies, int part);
```

Swap lines of a vector
```java
rearrange(VectorShuffle, VectorMask?);
rearrange(VectorShuffle, Vector, VectorMask?);

// Example 1
var species = IntVector.SPECIES_128;
var v1 = IntVector.zero(species);
for(var i = 0; i < v1.length; i++) {
	v1 = v1.withLane(i, i * 10);
}
System.out.println(v1); // [0, 10, 20, 30]

VectorShuffle<Integer> shuffle = VectorShuffle.fromValues(species, 0, 2, 1, 3);
var v2 = v1.rearrange(shuffle);
System.out.println(v2); // [0, 20, 10, 30]

// Example 2
var species = IntVector.SPECIES_128;
var v1 = IntVector.zero(species).addIndex(10);
System.out.println(v1); // [0, 10, 20, 30]

VectorShuffle<Integer> shuffle = VectorShuffle.fromValues(species, 0, 2, 1, 3);
var v2 = v1.rearrange(shuffle);
System.out.println(v2); // [0, 20, 10, 30]
```

Mix two vectors according to a mask (Mentioned [[12 - Vectorization#`VectorMask<*>`|here]])
```java
blend(Vector, VectorMask);
```

Slice/Unslice lanes + fill with a vector's elements
```java
slice(int origin, Vector, VectorMask?);
unslice(int origin, Vector, int part, VectorMark?);
```

Lanewise operations (*Each operation is defined by [[12 - Vectorization#`VectorOperators`|VectorOperators]]*)
```java
// Unary/Binary tests
test(Test op, VectorMask?); // → VectorMask
compare(Comparison op, Vector, VectorMask?); // → VectorMask

// Conversions
convert(Conversion op, int part);
convertShape(Conversion op, Species, int part);

// Unary/Binary/Ternary operations
lanewise(Unary op, VectorMask?);
lanewise(Binary op, Vector, VectorMask?);
lanewise(Ternary op, Vector, Vector, VectorMask?);

// Reduction operation on every lane (not defined in the Vector interface actually, as the type is different and there is no diamond syntax)
reduceLanes(VectorOperators);


// Example 1: Lanewise
var species = IntVector.SPECIES_128;
var v1 = IntVector.fromArray(species, new int[] { 7, 3, 16, 5 }, 0);
var v2 = IntVector.broadcast(species, 42);

var v3 = v1.lanewise(VectorOperators.ADD, v2);
System.out.println(v3); // [49, 45, 58, 47]


// Example 2: reduction
var species = IntVector.SPECIES_128;
var array = new int[] { 1, 2, 3, 4, 11, 12, 13, 14 };
var v1 = IntVector.fromArray(species, array, 0)
var v2 = IntVector.fromArray(species, array, 4);
var v3 = v1.add(v2);

int sum = v3.reduceLanes(VectorOperators.ADD);
System.out.println(sum); // 60
```


****
## Read/Write in Array

Static reading methods:
```java
fromArray(Species, array, offset, VectorMask?);
fromByteArray(Species, byte[], offset, ByteOrder);
```

Instance writing methods:
```java
intoArray(array, offset, VectorMask?);
intoByteArray(byte[], offset, ByteOrder);
```

*The optional mask allows to limit the amount of data that is read/written (and avoid `ArrayIndexOutOfBoundsException`*


**Example: postLoop**

```java
static int[] copy(int[] array) {
	var length = array.length;
	var newArray = new int[length];
	var loopBound = length – length % SPECIES.length();
	
	var i = 0;
	for(; i < loopBound; i += SPECIES.length()) {
		var v = IntVector.fromArray(SPECIES, array, I);
		v.intoArray(newArray, i);
	}
	
	for(; i < length; i++) { // post loop
		newArray[i] = array[i];
	}
	return newArray;
}
```


**Example: mask**

```java
static int[] copy(int[] array) {
	var length = array.length;
	var newArray = new int[length];
	var loopBound = SPECIES.loopBound(length);
	
	var i = 0;
	for(; i < loopBound; i += SPECIES.length()) {
		var v = IntVector.fromArray(SPECIES, array, I);
		v.intoArray(newArray, i);
	}
	
	var mask = SPECIES.indexInRange(i, length);
	var v = IntVector.fromArray(SPECIES, array, i, mask);
	v.intoArray(newArray, i, mask);
	return newArray;
}
```


****
## Performance

As the API does not allow to give the type of the vector and not it's size (`IntVector`, not `IntVector128`), the information indicating its size must always remain visible to the JIT.
	**Solution:** `SPECIES` must be a constant (`static final`)

Vectorization in Java is not finished yet, storing a vector as a class property or in an array cell is very ineffective
	**Solution:** Well, avoid doing it
