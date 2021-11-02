---
title: "BigDecimal and BigInteger in Java"
categories:
  - Blog
tags:
  - BigDecimal and BigInteger
  - Precise numbers
---

### BigDecimal

BigDecimal represents an immutable arbitrary-precision signed decimal number. It consists of two parts:

Unscaled value – an arbitrary precision integer
* Scale – a 32-bit integer representing the number of digits to the right of the decimal point
* For example, the BigDecimal 7.14 has the unscaled value of 712 and the scale of 2.

Java BigDecimal class is used mostly to deal with financial data. BigDecimal is preferred while dealing with high-precision arithmetic or situations that require more granular control over rounding off calculations.

#### Operations on BigDecimal

Just like others Number classes like Integer, Long, Double etc. BigDecimal provides operations for arithmetic and comparison operations. 

Such as: add, subtract, multiply, divide and compareTo.

BigDecimal has also methods to extract various attributes, such as precision, scale, and sign:

#### Roundings

By rounding a number, we replace it with another having shorter, simpler and more meaningful representation. 

There are two classes which control rounding behavior – RoundingMode and MathContext.

The enum **[RoundingMode](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/RoundingMode.html)** provides eight rounding modes:

* CEILING – rounds towards positive infinity

* FLOOR – rounds towards negative infinity

* UP – rounds away from zero

* DOWN – rounds towards zero

* HALF_UP – rounds towards “the nearest neighbor” unless both neighbors are equidistant, in which case rounds up

* HALF_DOWN – rounds towards “the nearest neighbor” unless both neighbors are equidistant, in which case rounds down

* HALF_EVEN – rounds towards the “nearest neighbor” unless both neighbors are equidistant, in which case, rounds towards the even neighbor

* UNNECESSARY – no rounding is necessary and ArithmeticException is thrown if no exact result is possible

#### MathContext

**[MathContext](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/MathContext.html) encapsulates both precision and rounding mode**, and it provides four rounding modes:

* DECIMAL32 – 7 digits precision and a rounding mode of HALF_EVEN

* DECIMAL64 – 16 digits precision and a rounding mode of HALF_EVEN

* DECIMAL128 – 34 digits precision and a rounding mode of HALF_EVEN

* UNLIMITED – unlimited precision arithmetic

**Example**

The code below round to 2 digits using HALF_EVEN

```java
BigDecimal bd = new BigDecimal("7.5");

BigDecimal rounded = bd .round(new MathContext(2, RoundingMode.HALF_EVEN));
```


### BigInteger

BigInteger class is used for mathematical operation which involves very big integer calculations that are outside the limit of all available primitive data types.

For example factorial of 100 contains 158 digits in it, so we can’t store it in any primitive data type available. We can store as large Integer as we want in it. 

There is no theoretical limit on the upper bound of the range because memory is allocated dynamically but practically as memory is limited you can store a number which has Integer.MAX_VALUE number of bits in it which should be sufficient to store mostly all large values.


**We can create BigInteger from a byte array or String:**
```java
 BigInteger biFromString = new BigInteger("1289231289098714924");

 BigInteger biFromByteArray = new BigInteger(new byte[] { 64, 64, 64, 64, 64, 64 });

 BigInteger biFromSignMagnitude = new BigInteger(-1, new byte[] { 64, 64, 64, 64, 64, 64 });
```

In addition, we can convert a long to BigInteger using the static method valueOf:
```java
 BigInteger biFromLong =  BigInteger.valueOf(7897343012089369395L);
```

#### Operations on BigInteger

BigInteger class provides operations analogues to all of Java's primitive integer operators and for all relevant methods from java. lang. Math. 
It also provides operations for modular arithmetic, GCD calculation, primality testing, prime generation, bit manipulation, and a few other miscellaneous operations like abs, min, max, pow, signum from Math class.

**We compare the value of two BigIntegers using the compareTo method:**
```java
 BigInteger biFirst = new BigInteger("123456789012345678901234567890");
 BigInteger biSecond = new BigInteger("123456789012345678901234567891");
 boolean firstIsGreater = biFirst.compareTo(biSecond) > 0;
```

**BigInteger has the bit operations similar to int and long. But, we need to use the methods instead of operators:**
```java
 BigInteger biFirst = new BigInteger("4");
 BigInteger biSecond = new BigInteger("7");

 BigInteger and = biFirst.and(biSecond);
 BigInteger or = biFirst.or(biSecond);
 BigInteger not = biSecond.not();
 BigInteger xor = biFirst.xor(biSecond);
 BigInteger andNot = biFirst.andNot(biSecond);
 BigInteger shiftLeft = biFirst.shiftLeft(1);
 BigInteger shiftRight = biFirst.shiftRight(1);
```

**It has additional bit manipulation methods:**

```java
 BigInteger bi = new BigInteger("1024");

 BigInteger setBit8 = bi.setBit(8);
 BigInteger flipBit0 = bi.flipBit(0);
 BigInteger clearBit2 = bi.clearBit(2);

```

**BigInteger provides methods for GCD(Greatest Common Divisor) computation and modular arithmetic:**
```java
 BigInteger biFirst = new BigInteger("50");
 BigInteger biSecond = new BigInteger("18");
 BigInteger biThird = new BigInteger("12");

 BigInteger gcd = biSecond.gcd(biThird);  //6
 BigInteger multiplyAndMod = biSecond.multiply(biThird).mod(biFirst); //20
 BigInteger modInverse = biSecond.modInverse(biFirst); //30
 BigInteger modPow = biSecond.modPow(biThird, biFirst); //1

```

**It also has methods related to prime generation and primality testing:**
```java
 BigInteger i = BigInteger.probablePrime(100, new Random());
        
 boolean isProbablePrime = i.isProbablePrime(1000); //true
```



#### Worth to remember

* BigDecimals and BigInteger are immutable, operations do not modify the existing objects. Rather, they return new objects.                            

* Comparing using == ignores the scale while comparing.

* the equals method considers two BigDecimal objects as equal only if they are equal in value and scale. Thus, BigDecimals 3.0 and 3.00 are not equal when compared by this method.




