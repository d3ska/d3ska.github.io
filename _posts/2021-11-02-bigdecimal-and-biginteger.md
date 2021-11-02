---
title: "BigDecimal and BigInteger in Java"
categories:
  - Blog
tags:
  - BigDecimal and BigInteger
  - Precise numbers
---

### BigDecimal and BigInteger in Java

#### BigDecimal

BigDecimal represents an immutable arbitrary-precision signed decimal number. It consists of two parts:

Unscaled value – an arbitrary precision integer
* Scale – a 32-bit integer representing the number of digits to the right of the decimal point
* For example, the BigDecimal 7.14 has the unscaled value of 712 and the scale of 2.

Java BigDecimal class is used mostly to deal with financial data. BigDecimal is preferred while dealing with high-precision arithmetic or situations that require more granular control over rounding off calculations.

#### Operations on BigDecimal

Just like others Number classes like Integer, Long, Double etc. BigDecimal provides operations for arithmetic and comparison operations. 

Such as: add, subtract, multiply, divide and compareTo.

BigDecimal has also methods to extract various attributes, such as precision, scale, and sign:

#### Worth to remember

* BigDecimals are immutable, operations do not modify the existing objects. Rather, they return new objects.                            

* Comparing BigDecimal using == ignores the scale while comparing.

* the equals method considers two BigDecimal objects as equal only if they are equal in value and scale. Thus, BigDecimals 3.0 and 3.00 are not equal when compared by this method.

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
