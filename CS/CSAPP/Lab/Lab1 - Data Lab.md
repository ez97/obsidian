```C
//1
/*
 * bitXor - x^y using only ~ and &
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
  int case1 = ~x & y; // 1 ^ 0 = 1
  int case2 = ~y & x; // 0 ^ 1 = 1
  return ~(~case1 & ~case2);
}
/*
 * tmin - return minimum two's complement integer
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 1 << 31;
}

//2
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  int tmin_check = x + 1; // tmax + 1 = tmin
  return !!(tmin_check ^ 0) & !(tmin_check ^ (~tmin_check + 1)); // check = -check && check != 0
}
/*
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
  int mask = 0xAA + (0xAA << 8) + (0xAA << 16) + (0xAA << 24);
  return !((mask & x) ^ mask);
}
/*
 * negate - return -x
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x + 1;
}

//3
/*
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
  int is0x3_ = !((x >> 4) ^ 0x03); // 0000 0000 0101 xxxx
  // 0 ~ 7 : 0000 ~ 0111 -> 0xxx
  // 8 : 1000, 9 : 1001 -> 100x
  int case1 = !(x & 0x08);
  int case2 = !((x & 0x0E) ^ 0x08);
  return is0x3_ & (case1 | case2);
}
/*
 * conditional - same as x ? y : z
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  // x == 0, !x = 1, !!x = 0, mask = 0000
  // x != 0, !x = 0, !!x = 1, mask = 1111
  int mask = (!!x << 31) >> 31;
  return (mask & y) + (~mask & z);
}
/*
 * isLessOrEqual - if x <= y  then return 1, else return 0
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int sign_x = (x >> 31) & 0x01;
  int sign_y = (y >> 31) & 0x01;
  int case1 = sign_x & (!sign_y);// x < 0 && y >= 0
  int sub = ~x + y + 1;
  int sign_sub = (sub >> 31) & 0x01; // y - x >= 0
  int case2 = !(sign_x ^ sign_y) & !sign_sub;
  return case1 | case2;
}

//4
/*
 * logicalNeg - implement the ! operator, using all of
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4
 */
int logicalNeg(int x) {
  // 0 >= 0 && -0 >= 0
  int sign_x = (x >> 31) & 0x01; // x < 0
  int negx = ~x + 1;
  int sign_negx = (negx >> 31) & 0x01; // -x < 0
  return (sign_x | sign_negx) ^ 0x01;
  // another ans : return ((x | (~x + 1)) >> 31) + 1;
}
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
  int sign_x = x >> 31;
  x = (sign_x & ~x) | (~sign_x & x);
  int b16 = (!!(x >> 16)) << 4;
  x = x >> b16;
  int b8 = (!!(x >> 8)) << 3;
  x = x >> b8;
  int b4 = (!!(x >> 4)) << 2;
  x = x >> b4;
  int b2 = (!!(x >> 2)) << 1;
  x = x >> b2;
  int b1 = !!(x >> 1);
  int b0 = x >> b1;
  return b0 + b1 + b2 + b4 + b8 + b16 + 1;
}

//float
/*
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
  int sign = uf & 0x80000000;
  int exp = uf & 0x7F800000;
  int frac = uf & 0x007FFFFF;
  // infty or NaN
  if(exp == 0x7F800000){
    return uf;
  }
  // Normalized
  if(exp != 0x0){
    exp = exp + 0x00800000;
    if(exp == 0x7F800000){
      frac = 0x0;
    }
    return sign | exp | frac;
  }
  // Denormalized : although ans may becomes normalized number, it works
  frac = frac << 1;
sign | frac;
}
/*
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) { // 3f80 0000 = 0[011 1111 1]000 0000, exp = 127, E = 0
  int exp = (uf & 0x7F800000) >> 23;
  // infty or NaN
  if(exp == 0xFF){
    return 0x80000000;
  }
  // 0 <= |uf| < 1
  if(exp < 127){
    return 0;
  }
  int E = exp - 127; // E = exp - bias;
  unsigned M = (uf & 0x007FFFFF) | 0x00800000; // M = 1 + f
  // ans = 2^E * M
  if(E > 31){
    return 0x80000000;
  }
  if(E > 23){
    M = M << (E - 23);
  }
  else{
    M = M >> (23 - E);
  }
  // positive : 0 ~ 0x7FFFFFFF
  // negative : 0 ~ 0x80000000
  if ((M >> 31) & 1) {
    return 0x80000000;
  }
  if (uf & 0x80000000) {
    return -M;
  }
  else {
    return M;
  }
}

/*
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 *
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while
 *   Max ops: 30
 *   Rating: 4
 */

unsigned floatPower2(int x) {
  int exp = 127 + x;
  if(exp >= 255){
    return 0x7F800000;
  }
  if(exp < 0){
    return 0;
  }
  return exp << 23;
}
```

- **Negation using `~x + 1`**:
    - The operation `~x + 1` always works to compute the negation of `x` in two's complement representation.
- **Equality check with `!(x ^ y)`**:
    - You can use the expression `!(x ^ y)` to check if `x` and `y` are equal. If they are equal, `x ^ y` results in 0, and `!0` is 1 (true).
- **Conditional checks using 0 and 1**:
    - When performing conditional checks, you can convert a value to 0 or 1 using `!!x`. This double negation converts any non-zero value to 1 and zero remains 0.
- **Using masks to access specific bits**:
    - It's common to use bit masks to access or manipulate specific bits within a value. A mask can isolate, set, or clear particular bits of interest.
- **Creating masks by right-shifting**:
    - By right-shifting a number by 31 bits (or 63 bits for a 64-bit integer), you can create a mask that is either all 1s for negative numbers or all 0s for non-negative numbers.

![[Pasted image 20240616195840.png]]