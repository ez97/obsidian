	Representing and Manipulating Information : 信息的表示和处理

## 2.1 Information Storage

A machine-level program views memory as **a very large array of bytes**, referred to as virtual memory.

	bytes : block of 8 bits, smallest addressable unit of memory.

	Address :  A unique number that identifies every bytes of memory.

For example, the value of a pointer in C is the virtual address of **the first byte** of the block of storage.

And the C compiler associates **type** information with each pointer (not dynamic) , so that it can generate **different machine-level code** (from the same 0-1 data) to get the value.

Notice : Although compiler maintains type infromation, the actual machine-level **program has** **no information about data types** (it only cares about machine instructions)

### 2.1.1 Hexadecimal Notation
	Hex : 16进制

A single byte consist of 8 bits,
in binary notation : 00000000 ~ 11111111
in decimal notation : 0 ~ 255
in hexadecimal notation : 0 ~ FF

#### Practice 2.1

	0x25B9D2
0010 0101 1011 1001 1101 0010

	0x1010111001001001
1010 1110 0100 1001 -> 0xAE49

	0xA8B3D
1010 1000 1011 0011 1101

	0011 0010 0010 1101 1001 0110
0x322D96

4 binary notation bits -> 1 hexadecimal notation bit 
#### Practice 2.2

| n      | decimal(2^n) | hexadecimal(2^n) |
| ------ | ------------ | ---------------- |
| **5**  | **32**       | **0x20**         |
| **23** | 8388608      | 0x800000         |
| 15     | **32768**    | 0x8000           |
| 13     | 8192         | **0x2000**       |
| **12** | 4096         | 0x1000           |
| 6      | **64**       | 0x40             |
| 8      | 256          | **0x100**        |
$2^n$ -> n zeros in binary bits -> n/4 zeros in hexadecimal bits
$n = 15 = 4*3+3$ -> 0x$(1000)_2$ 000 -> 0x8000

#### Practice 2.3

| decimal | binarary          | hexadecimal |
| ------- | ----------------- | ----------- |
| **0**   | **0000** **0000** | **0x00**    |
| **158** | 1001 1110         | 0x9E        |
| **76**  | 0100 1100         | 0x4C        |
| **145** | 1001 0001         | 0x91        |
| 174     | **1010** **1110** | 0xAE        |
| 60      | **0011** **1100** | 0x3C        |
| 241     | **1111** **0001** | 0xF1        |
| 117     | 0111 0101         | **0x75**    |
| 189     | 1011 1101         | **0xBD**    |
| 245     | 1111 0101         | **0xF5**    |
158 = 128 + 16 + 8 + 4 + 2
76 = 64 + 8 + 4
145 = 128 + 16 + 1
16\*10+14=174
16\*3+12=60
16\*15+1=241
7\*16+5=117
11\*16+13=189
15\*16+5=245

#### Practice 2.4

0x605c + 0x5 = 0x6061
0x605c - 0x20 = 0x603c
0x605c + 32 = 0x605c + 0x10 = 0x607c
0x60fa - 0x605c = 0x9E

### 2.1.2 Data Sizes

	word size : the maximum size of virtual address space, equal to the nominal size of pointer data.

The distinction is how a program is **compiled**, rather than the type of machine on which it runs.

Compilers support multiple data formats using different ways to encode data, such as integers and floating point. In other words, data with the **same data** **type** may be represented as **different** **byte quantities**.

For example, "long" has 4 bytes in 32-bit program, and has 8 bytes in 64-bit pregram. To avoid this, ISO C99 introduced a class of data types regardless of compiler and machine settings such as int32_t and int64_t, having exactly 4 and 8 bytes.

Most of the data types encode **signed values** unless prefixed by the keyword "unsigned", and the exception is type "char".

A **pointer** uses the **full word size** of the program.

### 2.1.3 Addressing and Byte Ordering

0x01234567

| memory address      | 0x100 | 0x101 | 0x102 | 0x103 |
| ------------------- | ----- | ----- | ----- | ----- |
| Big endian value    | 01    | 23    | 45    | 67    |
| Little endian value | 67    | 45    | 23    | 01    |

For most application programmers, the byte orderings used by their machines are totally invisible. However, **byte ordering** may become an issue when binary data are communicated over a network **between different machines**.

Second case : inspecting machine-level programs(such as using **disassembler**)

Thied case : pregrams are written that circumvent the normal type system(such as using **cast** or **union**)

	circumvent : to avoid something, especially cleverly or illegally

An example:
```C
#include <stdio.h>

typedef unsigned char *byte_pointer;

void show_bytes(byte_pointer start, size_t len){
    int i;
    for(i = 0; i < len; i++){
        printf(" %.2x", start[i]);
        printf("\n");
    }
}

void show_int(int x) {
    show_bytes((byte_pointer) &x, sizeof(int));
}

int main(){
    int a = 12345; // 0x00003039
    show_int(a);
    return 0;
}
```
The result is 39 30 00 00, indicating a little-endian machine.

#### Practice 2.5

```C
int a = 0x12345678;
byte_pointer ap = (byte_pointer) &a;
show_bytes(ap, 1); // A
show_bytes(ap, 2); // B
show_bytes(ap, 3); // C
```

|       | **Little endian** | **Big endian** |
| ----- | ----------------- | -------------- |
| **A** | 78                | 12             |
| **B** | 78 56             | 12 34          |
| **C** | 78 56 34          | 12 34 56       |
### 2.1.4 Representing Strings

A string in C is encoded by an array of characters **terminted by the null** character.
#### Practice 2.7
```C
const char *m = "mnopqr";
show_bytes((byte_pointer) m, strlen(m));
```
result : 6D 6E 6F 70 71 72

### 2.1.5 Representing Code

Different **machine types** use different and **incompatible instructions and encodings** , even indentical processors running different **operating systems** have differences in their coding conventions.

### 2.1.6 Introduction to Boolean Algebra

NOT

| ~   |     |
| --- | --- |
| 0   | 1   |
| 1   | 0   |
AND

| &   | 0   | 1   |
| --- | --- | --- |
| 0   | 0   | 0   |
| 1   | 0   | 1   |
OR

| \|  | 0   | 1   |
| --- | --- | --- |
| 0   | 0   | 1   |
| 1   | 1   | 1   |
EXCLUSIVE-OR

| ^   | 0   | 1   |
| --- | --- | --- |
| 0   | 0   | 1   |
| 1   | 1   | 0   |
Let $a$ and $b$ denote the bit vectors, then extend four operations to also operate on **bit vectors**.

#### Practice 2.8

| **Operations** | **Result**   |
| -------------- | ------------ |
| **a**          | **01001110** |
| **b**          | **11100001** |
| **~a**         | 10110001     |
| **~b**         | 00011110     |
| **a & b**      | 01000000     |
| **a \| b**     | 11101111     |
| **a ^ b**      | 10101111     |

One useful application of bit vectors is to **represent finite sets**.

In a backtracking algorithm, it's common to use each bit to encode whether an option is selected or not.

And we can selectively enable or disable different signals by specifying a **bit-vector mask**.

### 2.1.7 Bit-Level Operations in C

#### Practice 2.10

```C
void inplace_swap(int *x, int *y){
	*y = *x ^ *y; // Step 1
	*x = *x ^ *y; // 2
	*y = *x ^ *y; // 3
}
```

| Step      | *x                      | *y                            |
| --------- | ----------------------- | ----------------------------- |
| Initially | **a**                   | **b**                         |
| Step 1    | a                       | a ^ b                         |
| Step 2    | a ^ (a ^ b) = 0 ^ b = b | a ^ b                         |
| Step 3    | b                       | b ^ (a ^ b) = b ^ (b ^ a) = a |

> [!NOTE] 
>The logical operators `&`, `|`, and `^` satisfy the commutative and associative property.

#### Practice 2.11

```C
void reverse_array(int a[], int cnt){
	int first, last;
	for(first = 0, last = cnt-1; first <= last; first++, last--){
		inplace_swap(&a[first], &a[last]);
	}
}
```

modification : **first < last**

inplace_swap can only used between **different address**.


One common use of bit-level operations is to implement **masking operations**.
~0 will yield a mask of all ones, regardless of the size of the data representation.(portable)
#### Practice 2.12
```C
/* The least significant byte of x, with all other bits set to 0.*/
x = 0xFF & x; 

/* All but the least significant byte of x complemented, with the least significant byte left unchanged.*/
x = ~x ^ 0xFF;

/*The least significant byte set to all ones, and all other bytes of x left unchanged.*/

x = x | 0xFF;
```

> [!NOTE] 
> x ^ 0 = x
> x ^ 1 = ~x

#### Practice 2.13
```C
/* Declarations of functions implementing operations bis and bic */
int bis(int x, int m);
int bic(int x, int m);

/* Compute x|y using only calls to functions bis and bic */
int bool_or(int x, int y) {
	int result = bis(x, y);
	return result;
}

/* Compute x^y using only calls to functions bis and bic */
	int bool_xor(int x, int y) {
	int result = bis(bic(x, y), bic(y, x));
	return result;
}
```


### 2.1.8 Logical Operations in C

#### Practice 2.14
a = 0x55 = 0101 0101
b = 0x46 = 0100 0110

| a & b    | 0100 0100=0x44 | a && b     | 0x01 |
| -------- | -------------- | ---------- | ---- |
| a \| b   | 0101 0111=0x57 | a \|\| b   | 0x01 |
| ~a \| ~b | 1011 1011=0xBB | !a \|\| !b | 0x00 |
| a & !b   | 0x00           | a && ~b    | 0x01 |

#### Practice 2.15

```C
// x == y
result = !(x ^ y);
```

### 2.1.9 Shift Operations in C

logical right shift 2 : 12345 -> 00123
arithmetic right shift 2 : 12345 -> 11123
left shift 2 : 34500

#### Practice 2.16
Logical, arithmetic

| a        |           | a<<2 |           | a>>3 |           | a>>3 |           |
| -------- | --------- | ---- | --------- | ---- | --------- | ---- | --------- |
| **0xD4** | 1101 0100 | 0x50 | 0101 0000 | 0x1A | 0001 1010 | 0xFA | 1111 1010 |
| **0x64** | 0110 0100 | 0x90 | 1001 0000 | 0x0C | 0000 1100 | 0x0C | 0000 1100 |
| **0x72** | 0111 0010 | 0xC8 | 1100 1000 | 0x0E | 0000 1110 | 0x0E | 0000 1110 |
| **0x44** | 0100 0100 | 0x10 | 0001 0000 | 0x08 | 0000 1000 | 0x08 | 0000 1000 |
## 2.2 Integer Rrepresentations

	teminology : special words or expressions used in relation to a particular subject or activity

### 2.2.1 Integral Data Types

	Integral : necessary and important as part of a whole

The only machine-dependent range indicated is for size designator *long*.

The ranges of intergral data types are not symmetric--the range of **negative** numbers extends **one further** than the range of **positive** numbers.

### 2.2.2 Unsigned Encodings

$$
B2U_w(\boldsymbol x)=\sum_{i=0}^{w-1}x_i2^i
$$
Function $B2U_w$ is a bijection.
$U_{max}=2^w-1$
$U_{min}=0$

	bijection : one-to-one and onto

### 2.2.3 Two's-complement Encodings

$$
B2T_w(x)=-x_{w-1}2^{w-1}+\sum_{i=0}^{w-2}x_i2^i
$$
Function $B2T_w$ is a bijection.
$x_{w-1}$ : sign bit
$T_{max}=2^{w-1}-1$
$T_{min}=-2^{w-1}$

#### Practice 2.17

| Hex     | Binary   | B2U    | B2T    |
| ------- | -------- | ------ | ------ |
| **0xA** | **1010** | **10** | **-6** |
| **0x1** | 0001     | 1      | 1      |
| **0xB** | 1010     | 11     | -5     |
| **0x2** | 0010     | 2      | 2      |
| **0x7** | 0111     | 7      | 7      |
| **0xC** | 1100     | 12     | -4     |

	delimit : to mark or describe the limits of somthing

#### Prcatice 2.18

| **0x2e0** | 0010 1110 0000 | 736 |
| --------- | -------------- | --- |
| **-0x58** | 0101 1000      | -88 |
| **0x28**  | 0010 1000      | 40  |
| **-0x30** | 0011 0000      | -48 |
| **0x78**  | 0111 1000      | 120 |
| **0x88**  | 1000 1000      | 136 |
| **0x1f8** | 0001 1111 1000 | 504 |
| **0xc0**  | 1100 0000      | 192 |
| **-0x48** | 0010 1000      | -72 |

### 2.2.4 Conversions between Signed and Unsigned

	convention : a usual or accepted way of behaving, especially in social situations, often following an old way of thinking or a custom in one particular society

When handling conversions between signed and unsigned numbers with the same word size, the numeric values might change, but the **bit patterns** do not.

$$
\begin{equation}
T2U_w(x)=\left\{
\begin{aligned}
&x+2^w, & & x<0\\
&x, & & x\geq0
\end{aligned}
\right
.
\end{equation}
$$
$$
\begin{equation}
U2T_w(u)=\left\{
\begin{aligned}
&u, & & u\leq T_{max}\\
&u-2^w, & & u > T_{max}
\end{aligned}
\right
.
\end{equation}
$$
#### Practice 2.19

| x          | **-1** | **-5** | **-6** | **-4** | **1** | **8** |
| ---------- | ------ | ------ | ------ | ------ | ----- | ----- |
| $T2U_4(x)$ | 15     | 9      | 10     | 12     | 1     | 8     |

### 2.25 Signed versus Unsigned in C

	arithmetic : calculations that involves the adding and multiplying, etc. of numbers

Generally, most numbers are **signed by default**.(12345 or 0x1A2B) Adding character 'U' or 'u' as a suffix creates an unsigned constant.(12345U or 0x1A2Bu)

> [!NOTE] printf function
> %d : signed decimal
> %u : unsigned decimal
> %x : hexadecimal

When an **operation** is performed where one operand is signed and the other is unsigned, C implicitly **casts the signed argument to unsigned** and performs the operations assuming th numbers are nonnegative.

#### Practice 2.21

| Expression                   | Type     | Evaluation |
| ---------------------------- | -------- | ---------- |
| -2147483647-1 == 2147483648U | unsigned | 1          |
| -2147483647-1 < 2147483647   | signed   | 0          |
| -2147483647-1U < 2147483647  | unsigned | 0          |
| -2147483647-1 < -2147483647  | signed   | 1          |
| -2147483647-1U < -2147483647 | unsigned | 1          |
(32-bit program)

### 2.2.6 Expanding the Bit Representation of a Number

	retain : to keep or continue to have something.

Converting from a smaller to a larger data type should always be possible.To convert an **unsigned** number to a larger data type, we can simply **add** **leading zeros**. This operation is known as **zero extension**.

For converting a two's-complement number to a larger data type, the rule is to perform a **sign extension**, adding copies of sign bit.

When converting from short to unsigned, the program first changes the **size** and then the **type**.

#### Practice 2.23

```C
int fun1(unsigned word) {
	return (int) ((word << 24) >> 24);
}

int fun2(unsigned word) {
	return ((int) word << 24) >> 24;
}
// 32-bit program
```

| w              | fun1(w)    | fun2(w)    |
| -------------- | ---------- | ---------- |
| **0x00000076** | 0x00000076 | 0x00000076 |
| **0x87654321** | 0x00000021 | 0x00000021 |
| **0x000000C9** | 0x000000C9 | 0xFFFFFFC9 |
| **0xEDCBA987** | 0x00000087 | 0xFFFFFF87 |
### 2.2.7 Truncating Numbers

	truncate : to make something shorter or quicker, especially by removing the end of it.

When truncating a w-bit number to a k-bit number, we simply **drop** the high-order w-k bits. Truncation a number can **alter** its value.
Truncation of an unsigned number : $x' = x\ \rm{mod}\ 2^k$

	intuition : an ability to understand or know something immediately based on your feelings rather than facts.

#### Practice 2.24

| Hex        |           | Unsigned |           | Two's complement |           |
| ---------- | --------- | -------- | --------- | ---------------- | --------- |
| Original   | Truncated | Original | Truncated | Original         | Truncated |
| **1**=0001 | 1         | **1**    | 1         | **1**            | 1         |
| **3**=0011 | 3         | **3**    | 3         | **3**            | 3         |
| **5**=0101 | 5         | **5**    | 5         | **5**            | 5         |
| **C**=1100 | 4         | **12**   | 4         | **-4**           | -4        |
| **E**=1110 | 6         | **14**   | 6         | **-2**           | -2        |
truncate a 4-bit value to a 3-bit value.

### 2.2.8 Advice on Signed versus Unsigned

The implicit casting leads to **nonintuitive behavior**, and nonintuitive features often lead to program **bugs**. Many other language designers viewed unsigned integers as more trouble than they are worth.

	overlook : to fail to notice or consider something or someone.

#### Practice 2.25

```C
float sum_elements(float a[], unsigned length) {
	int i;
	float result = 0;

	for(i = 0; i <= length-1; i++){
		result += a[i];
	}
	return result;
}
```
When length = 0, i <= length-1 becomes i <= $U_{max}$

#### Practice 2.26

```C
size_t strlen(const char *s);

int strlonger(char *s, char *t) {
	return strlen(s) - strlen(t) > 0;
}

// to fix it : return strlen(s) > strlen(t)
```
When compiled as a 32-bit program is defined (via typedef) in header file stdio.h to be unsigned.
So when strlen(s) is not equal to strlen(t), it returns 1.

	apparently : used when the real situation is different from what you thought it was.

## 2.3 Integer Arithmetic

### 2.3.1 Unsigned Addition

$$
x+^{u}_{w}y=\left\{
\begin{aligned}
& x+y, & &x+y<2^w \\
& x+y-2^w,& &2^w\leq x+y<2^{w+1} 
\end{aligned}
\right
.
$$
Detecting overflow : $s\geq x$ ?
$$
-^{u}_{w}x=\left\{
\begin{aligned}
& x, & &x=0 \\
& 2^w-x,& &x>0 
\end{aligned}
\right
.
$$
#### Practice 2.27

```C
int uadd_ok(unsigned x, unsigned y) {
	unsigned s = x + y;
	return s >= x;
}
```
#### Practice 2.28

| x   |         | $-^{u}_{4}x$ |     |
| --- | ------- | ------------ | --- |
| Hex | Decimal | Decimal      | Hex |
| 1   | 1       | 15           | F   |
| 4   | 4       | 12           | C   |
| 7   | 7       | 9            | 9   |
| A   | 10      | 6            | 6   |
| E   | 14      | 2            | 2   |
|     |         |              |     |
### 2.3.2 Two's complement Addition

$$
x +^{t}_{w}y = \left\{
\begin{aligned}
&x + y - 2^w, & &x + y \geq 2^{w-1}\\
&x + y, & &-2^{w-1}\leq x + y < 2^{w-1}\\
&x + y + 2^{w}, & &x + y < -2^{w-1}
\end{aligned}
\right
.
$$
Since $-2^{w-1} \leq x, y \leq 2^{w-1}-1$, we have $-2^w\leq x + y \leq 2^w -2$.
to get $T_{min}$, $x+y = 2^{w-1}$
to get $T_{max}$, $x+y=-2^{w-1}-1$
In **positive overflow**, the max sum is **-2**, and in **negative overflow** the min sum is **0**
Detecting positive overflow : $x > 0$ and $y>0$ and $s \leq -2$
Detecting negative overflow : $x<0$ and $x<0$ and $s \geq 0$
#### Practice 2.29

| x             | y             | x+y    | $x+^t_5 y$ | Case              |
| ------------- | ------------- | ------ | ---------- | ----------------- |
| **10100**=-12 | **10001**=-15 | 100101 | 00101=5    | negative overflow |
| **11000**=-8  | **11000**=-8  | 110000 | 10000=-16  | normal            |
| **10111**=-9  | **01000**=8   | 11111  | 11111=-1   | normal            |
| **00010**=2   | **00101**=5   | 00111  | 00111=7    | normal            |
| **01100**=12  | **00100**=4   | 10000  | 10000=-16  | positive overflow |
#### Practice 2.30
```C
int tadd_ok(int x, int y){
	int s = x + y;
	if(x > 0 && y > 0 && s <= 0){
		return 0;
	}
	if(x < 0 && y < 0 && s >= 0){
		return 0;
	}
	return 1;
}
```
#### Practice 2.31

```C
int tadd_ok(int x, int y) {
	int sum = x+y;
	return (sum - x == y) && (sum - y == x);
}
```
wrong.
in positive overflow,  $\rm{sum} = x+y - 2^w$
then $\rm{B2T}(\rm{sum} - x) = \rm{B2T}(y - 2^w) = y$
#### Practice 2.32

```C
int tsub_ok(int x, int y) {
	return tadd_ok(x, -y);
}
```
wrong.
When y is INT_MIN, -y becomes INT_MIN
### 2.3.3 Two's complement Negation
$$
-^{t}_{w}x=\left\{
\begin{aligned}
& TMin_w, & &x=TMin_w \\
& -x,& &x>Tmin_w 
\end{aligned}
\right
.
$$
#### Practice 2.33

| x     |         | $-^t_4 x$ |         |
| ----- | ------- | --------- | ------- |
| Hex   | Decimal | Hex       | Decimal |
| **0** | 0       | 0         | 0       |
| **5** | 5       | B         | -5      |
| **8** | -8      | 8         | -8      |
| **D** | -3      | 3         | 3       |
| **F** | -1      | 1         | 1       |

### 2.3.4 Unsigned Multiplication

$$
x *^u_w y = (x\cdot y)\ \rm{mod}\ 2^w
$$
### 2.3.5 Two's complement Multiplication

**Truncate** the 2$w$-bit to $w$ **bits**, then use $\rm{U2T}$
$$
x *^t_w y = \rm{U2T}_w ((x\cdot y)\ \rm{mod}\ 2^w)
$$
#### Practice 2.34

| Mode               | x   |         | y   |         | $x\cdot y$ |        | Truncated $x\cdot y$ |     |
| ------------------ | --- | ------- | --- | ------- | ---------- | ------ | -------------------- | --- |
| **Unsigned**       | 4   | **100** | 5   | **101** | 20         | 010100 | 4                    | 100 |
| **2's complement** | -4  | **100** | -3  | **101** | -12        | 110100 | -4                   | 100 |
| **Unsigned**       | 2   | **010** | 7   | **111** | 14         | 001110 | 6                    | 110 |
| **2's complement** | 2   | **010** | -1  | **111** | -2         | 111110 | -2                   | 110 |
| **Unsigned**       | 6   | **110** | 6   | **110** | 36         | 100100 | 4                    | 100 |
| **2's complement** | -2  | **110** | -2  | **110** | 4          | 000100 | -4                   | 100 |
#### Practice 2.35
```C
int tmult_ok(int x, int y) {
	int p = x * y;
	return !x || p/x == y;
}
```
Right.
1. $x \cdot y = p + t2^w$
2. $p = x\cdot q + r$
3. $q=y$ iff $r=t=0$
#### Practice 2.36

```C
int tmult_ok(int x, int y) {
	int64_t p = (int64_t) x * y; // important!!
	return p == (int) p;
}
```
(int64_t) x\*y will not work 
#### Practice 2.37

```C
uint64_t asize = ele_cnt * (uint64_t) ele_size;
void *result = malloc(asize);
// malloc has type size_t
```
wrong.
size_t is unsigned 32-bit number.
```C
uint64_t requied_size = ele_cnt * (uint64_t) ele_size;
size_t final_size = (size_t) requied_size;
if (requied_size != final_size){
	return NULL;
}
void *result = malloc(final_size);
```
### 2.3.6 Multiplying by Constants

Since integer **multiplying** is fairly **slow**, compilers attempt to **replace** multiplications by constant factors with combinations of **shift and addition** operations.

#### Practice 2.38

With a single LEA instruction :
$$
2^k\ \rm{or}\ 2^k+1 \rightarrow 1,2,3,4,5,8,9
$$

#### Practice 2.39

multiply weight :  1 ... 1 0 ... 0
$x << (n+1) \rightarrow 0...0 0 ...0 = 0$
new form B : -(x << m)
#### Practice 2.40

| K      | Shifts | Add/Subs | Expression                                |
| ------ | ------ | -------- | ----------------------------------------- |
| **7**  | **1**  | **1**    | (x << 3) - x                              |
| **30** | **4**  | **3**    | (x << 4) + (x << 3) + (x << 2) + (x << 1) |
| **28** | **2**  | **1**    | (x << 5) - (x << 2)                       |
| **55** | **2**  | **2**    | (x << 6) - (x << 3) - x                   |
7 = 8 - 1
30 = 16 + 8 + 4 + 2
28 = 32 - 4
55 = 64 - 8 - 1
### 2.3.7 Dividing by Power of 2

Integer division on most machines is even **slower** than integer multiplication, requiring 30 or more clock cycles.

Unsigned number : logical right shift, rounding down

2's complement number :
For **positive** number, x >> k.
But for **negative** numbers, simply right shifting may get a rounding up result, so we can firstly add a **bias** : (1 << k) - 1, the C expression is (x + (1<<k) - 1) >> k

#### Practice 2.42
```C
// x : int32_t
int div16(int x) {
	int bias = 1 << 4 - 1;
	bias = (x >> 31) & bias;
	return (x + bias) >> 4; 
}
```
### 2.3.8 Final Thoughts on Integer Arithmetic

#### Practice 2.44

| Expression                      | always true?                      |
| ------------------------------- | --------------------------------- |
| (x > 0) \|\| (x - 1 < 0)        | false, x != INT_MIN               |
| (x & 7) != 7 \|\| (x << 29 < 0) | true, $x_2$                       |
| (x * x) >= 0                    | false, overflow                   |
| x < 0 \|\| -x <= 0              | true                              |
| x > 0 \|\| -x >= 0              | false, INT_MIN                    |
| x + y == uy + ux                | true, **same bit-level behavior** |
| x\*~y + uy\*ux == -x            | true, ~y=-y-1                     |
## 2.4 Floating Point

### 2.4.1 Fractional Binary Numbers

	fractional : extremly small

Fractional binary notation can **only represent** numbers that can be written $x\times 2^y$, other values can only be **approximated**.

#### Practice 2.45

| Fractional value     | Binary representation | Decimal representation       |
| -------------------- | --------------------- | ---------------------------- |
| **1/8**              | **0.001**             | **0.125**                    |
| **3/4**=1/2+1/4      | 0.11                  | 0.75                         |
| **25/16**=1+1/2+1/16 | 1.1001                | 1.5625                       |
| 2+1/2+1/8+1/16=43/16 | **10.1011**           | 2.6875                       |
| 1+1/8=9/8            | **1.001**             | 1.125                        |
| 47/8                 | 101.111               | **5.875**=4+1+0.5+0.25+0.125 |
| 51/16                | 11.0011               | **3.1875**=2+1+0.125+0.0625  |
#### Practice 2.46
$x = 0.000110011 00110011001100$
$$
\begin{aligned}
0.1 - x &= 0.000110011[0011]_{2}... - 0.000110011 0011 0011 0011 00\\
&= 0.[1100]_2...\times 10^{-23}\\
&= 0.000[1100]_2...\times 10^{-20}\\
&= 0.1\times 2^{-20}\\
&\approx 9.54\times 10^{-8}\\
\Delta t &= 100\times 60\times 60\times10\times(0.1-x) \approx 0.343\ \rm{s}\\
\Delta x &\approx 687\ \rm{m}
\end{aligned}
$$
### 2.4.2 IEEE Floating-Point Representation
![[Pasted image 20240612215233.png]]
$$
V = (-1)^S \times M \times 2^E
$$
- sign s : **sign** bit, negative?
- significand M : **fractional binary number**, $1\sim 2-\epsilon$  or $0 \sim 1-\epsilon$
- exponent E : **weights** the value by a power of 2


|                  | sign | exp | frac | total |
| ---------------- | ---- | --- | ---- | ----- |
| single precision | 1    | 8   | 23   | 32    |
| double precision | 1    | 11  | 52   | 64    |
- sign bit : encode s
- exp : encodes E
- frac : encodes M, but the value also **depends** on whether exp = 0
![[Pasted image 20240612215504.png]]
- Case1 Normalized Values : exp **not all zeros**, **not all ones**
The exponent field is interpreted as representing a **signed integer** in a **biased form**
$E = e - \rm{Bias}, \ \rm{Bias} = 2^{k-1}-1$, (in single, Bias = 127; in double, Bias = 1023)
For single precision, as $1 \leq e \leq 254$(not all 1 and not all 0), $-126 \leq E \leq 127$
For double precision, $-1022 \leq E \leq 1023$

The fraction field frac is interpreted as representing the fractional value $f = 0.f_{n-1}\cdots f_1 f_0$
then $M = 1 + f$, called **implied leading 1 representation**
$\Delta M = 2^{-n} = 2^{-23}(\rm{single}) = 2^{-52}(\rm{double})$

- Case2 Denormalized Values : exp **all zeros**
$E = 1 - \rm{Bias}$
$M = f$
fraction field does not have leading 1, so we can represent 0
IEEE format, we have value -0.0 and +0.0

- Case3 Infinity : exp **all ones**, frac **all zeros**
There are $+\infty$  and $-\infty$, infinity can represent **overflow results**, as when we **multiply** two large numbers, or **divide** by zero

- Case4 NaN : exp **all ones**, frac **not all zeros**
**not a real number**, like $\sqrt{-1}$ or $\infty - \infty$

### 2.4.3 Example Numbers

8-bit floating-point
exp k = 4
frac n = 3

- Case1 Normalized
Bias = 7, $1 \leq e \leq 14$, $-6 \leq E \leq 7$
$M =1. f_{1} f_{2} f_{3}$
0 1110 111 : $\rm{Max} = (1.111)_2 \times 2^7 = 1.875\times 2^7=240.0$
**0 0001 000** : $\rm{Min}= (1.000)_2 \times 2^{-6}=1 \times 2^{-6}=0.015625$

- Case2 Denormalized
$E = 1 - \rm{Bias} = -6$
**0 0000 111** : $\rm{Max} = (0.111)_2\times 2^{-6} = 0.013671875$
0 0000 001 : $\rm{Min} = (0.001)_2\times 2^{-6} = 0.001953125$
0 0000 000 : $\rm{Zero} = 0$

- Case3 infinity
0 1111 000 : $\infty$
1 1111 000 : $-\infty$

- Case4 NaN
0 1111 ????

Notice : There is a smooth transition between the **largest denormalized number** and the **smallest normalized number**

#### Practice 2.47

| Bits        | $e$   | $E$   | $2^E$ | $f$     | $M$     | $2^E\times M$ | $V$      | Decimal  |
| ----------- | ----- | ----- | ----- | ------- | ------- | ------------- | -------- | -------- |
| **0 00 00** | 0     | 0     | 1     | 0       | 0       | 0             | 0        | 0        |
| **0 00 01** | 0     | 0     | 1     | 1/4     | 1/4     | 1/4           | 1/4      | 0.25     |
| **0 00 10** | 0     | 0     | 1     | 1/2     | 1/2     | 1/2           | 1/2      | 0.5      |
| **0 00 11** | 0     | 0     | 1     | 3/4     | 3/4     | 3/4           | 3/4      | 0.75     |
| **0 01 00** | 1     | 0     | 1     | 0       | 1       | 1             | 1        | 1        |
| **0 01 01** | **1** | **0** | **1** | **1/4** | **5/4** | **5/4**       | **5/4**  | **1.25** |
| **0 01 10** | 1     | 0     | 1     | 1/2     | 3/2     | 3/2           | 3/2      | 1.5      |
| **0 01 11** | 1     | 0     | 1     | 3/4     | 7/4     | 7/4           | 7/4      | 1.75     |
| **0 10 00** | 2     | 1     | 2     | 0       | 1       | 2             | 2        | 2        |
| **0 10 01** | 2     | 1     | 2     | 1/4     | 5/4     | 5/2           | 5/2      | 2.5      |
| **0 10 10** | 2     | 1     | 2     | 1/2     | 3/2     | 3             | 3        | 3        |
| **0 10 11** | 2     | 1     | 2     | 3/4     | 7/4     | 7/2           | 7/2      | 3.5      |
| **0 11 00** | -     | -     | -     | -       | -       | -             | $\infty$ | -        |
| **0 11 01** | -     | -     | -     | -       | -       | -             | NaN      | -        |
| **0 11 10** | -     | -     | -     | -       | -       | -             | NaN      | -        |
| **0 11 11** | -     | -     | -     | -       | -       | -             | NaN      | -        |
Bias = 2 - 1 = 1

#### Practice 2.48

integer 3510593 : 001**1 0101 1001 0001 0100 0001**
$\rm{0x00359141} = 0011 0101 1001 0001 0100 0001_2 = 1.1 0101 1001 0001 0100 0001\times 2^{21}$
sign : 0, 1 bit
bias : 127
exp : 127+21 = 148 = 128 + 16 + 4 = $10010100_2$, 8 bits
frac : $1 0101 1001 0001 0100 0001 00_2$, 23 bits

0 10010100 **1 0101 1001 0001 0100 0001**00
low-order bits of the integer -> frac
#### Practice 2.49
the smallest positive integer the **cannot be represented** : $2^{n+1}+1$
for single, that is $2^{24}+1 = 16777217$
16777215 : $1.11...11_2 \times 2^{23}$
16777216 : $1.00...00_2 \times 2^{24}$
16777217 : $> 2^{24}$
### 2.4.4 Rounding

Round-to-even **avoids statiscal bias** in most real-life situations, 50% round upward and 50% round downward.
It's ok when we are **not** rounding to **a whole number**. Rounding 1.235000 and 1.245000 to 1.24, since 4 is even.
Round-to-even can also be applied to **binary fractional numbers**. We consider **least significant bit value 0** to be **even** and 1 to be odd. ($XX\cdots X.YY\cdots Y 100\cdots$)
Remember we will only use round-to-even when the value is **halfway between** two possible results, then we prefer to have the least significant bit equal to 0.
#### Practice 2.50
$10.111_2 = 2+1/2+1/4+1/8=23/8=2.875 \rightarrow 11.0_2$
$11.010_2 = 2+1+1/4=13/4=3.25\rightarrow 11.0_2$
$11.000_2 =2+1=3\rightarrow 11.0_2$
$10.110_2=2+1/2+1/4=11/4=2.75 \rightarrow 11.0_2$
#### Practice 2.51
$x' = 0.00011001100110011001101_2$
$x'-0.1=(0.1_2 - 0.000[1100]_2 \times 2^{2}) \times 2^{-22}=(0.5 - 0.4) \times 2^{-22}\approx 2.3842 \times 10^{-8}\ \rm{s}$
$100\times 60\times 60\times 10 \times (x' - 0.1) = 8.58\times 10 ^{-2}\ \rm{s}$
$172\ \rm{m}$
#### Practice 2.52\
- Format A : k = 3, n = 4, bias = 3
- Format B : k = 4, n = 3, bias = 7

| A            |       | B            |       |
| ------------ | ----- | ------------ | ----- |
| Bits         | Value | Bits         | Value |
| **011 0000** | **1** | **0111 000** | **1** |
| **101 1110** | 15/2  | 1001 111     | 15/2  |
| **010 1001** | 25/32 | 0110 100     | 3/4   |
| **110 1111** | 31/2  | 1011 000     | 16    |
| **000 0001** | 1/64  | 0001 000     | 1/64  |
4*(1+1/2+1/4+1/8)
1/2*(1+1/2+1/16)
8*(1+1/2+1/4+1/8+1/16)
1/4\*1/16=1/64
### 2.4.5 Floating-Point Operations

the computation should yield $\rm{Round}(x\bigodot y)$, the result of applying **rounding** to the exact result of the **real operation**
$1/-0=-\infty$
$1/+0=+\infty$

- floating-point addition
the operation is **not associative** (3.14+1e10-1e10=0, and 1e10-1e10+3.14=3.14)
As with an abelian group, most values have inverses under floating-point addition($x+^f -x=0$), the **exceptions** are **infinities**($+\infty - \infty = \rm{NaN}$) and **NaNs**($\rm{NaN} +^f x = \rm{NaN})$
On the other hand, for and x (except NaN), floating-point addition satisfies **monotonicity**

- floating-point multiplication 
**not associative** ($1e20*1e20*1e-20=\infty$, $1e20*(1e20*1e-20)=1e20$)
satisfies **monotonicity**

### 2.4.6 Floating Point in C

C standards do **not** require the machine to use IEEE floating point
When casting values :
- int $\rightarrow$ float : cannot overflow, may be **rounded**
- int or float $\rightarrow$ double : **exact** numeric value
- double $\rightarrow$ float : can **overflow** to $+\infty$ or $-\infty$, and may be **rounded**
- float or double $\rightarrow$  int : **rounded toward zero**, may be **overflow** (C standards do **not** specify a fixed result)

#### Practice 2.53

```C
#define POS_INFINITY 1e400
#define NEG_INFINITY (-POS_INFINITY)
#define NEG_ZERO (-1.0/POS_INFINITY)
```
#### Practice 2.54

| `x == (int)(double)x`   | true                                                       |
| ----------------------- | ---------------------------------------------------------- |
| `x == (int)(float)x`    | not always true, precise range of int is larger than float |
| `d == (double)(float)x` | not always true                                            |
| `f ==(float)(double)f`  | true                                                       |
| `f == -(-f)`            | true                                                       |
| `1.0/2 == 1/2.0`        | true                                                       |
| `d*d >= 0.0`            | true                                                       |
| `(f+d)-f == d`          | not always true, overflow                                  |
`x : int, f : flaot, d : double`
## 2.5 Summary

Most important :
- IEEE 754 floating-point numbers
- two's complement arithmetic
- data overflow
- data casting
- tricky bit level operations(`~x+1 == -x`, `(1<<8)-1 == 0xFF`)
