---
title: "BigDecimal and BigInteger in Java"
categories:
  - Java
tags:
  - BigDecimal
  - BigInteger
  - Java
  - Precision
---

Try adding 0.1 and 0.2 in Java:

```java
System.out.println(0.1 + 0.2); // 0.30000000000000004
```

The result is not 0.3. This is not a Java bug. It is how IEEE 754 floating-point arithmetic works: `double` and `float` store numbers in binary, and most decimal fractions (like 0.1) cannot be represented exactly in binary. For everyday calculations the tiny error is irrelevant, but in domains where every digit matters (financial systems, billing, tax calculations) it is unacceptable.

Java provides two classes for this: `BigDecimal` for arbitrary-precision decimal arithmetic, and `BigInteger` for arbitrary-precision integer arithmetic.

### BigDecimal

**BigDecimal** represents an immutable, arbitrary-precision signed decimal number. Internally it consists of two parts:

* **Unscaled value**: an arbitrary-precision integer
* **Scale**: a 32-bit integer representing the number of digits to the right of the decimal point

For example, the BigDecimal `7.14` has an unscaled value of 714 and a scale of 2 (because 714 x 10^-2 = 7.14).

#### Creating BigDecimal

```java
// From a String (preferred)
BigDecimal price = new BigDecimal("19.99");

// From a long via factory method
BigDecimal quantity = BigDecimal.valueOf(5);

// Predefined constants
BigDecimal zero = BigDecimal.ZERO;
BigDecimal one  = BigDecimal.ONE;
BigDecimal ten  = BigDecimal.TEN;
```

Always prefer `new BigDecimal(String)` or `BigDecimal.valueOf(double)` over `new BigDecimal(double)`. The `double` constructor captures the imprecision inherent in binary floating point:

```java
BigDecimal fromDouble = new BigDecimal(0.1);
System.out.println(fromDouble);
// 0.1000000000000000055511151231257827021181583404541015625

BigDecimal fromString = new BigDecimal("0.1");
System.out.println(fromString);
// 0.1
```

#### Operations

Like other Number classes, BigDecimal provides methods for arithmetic and comparison. Because BigDecimal is immutable, every operation returns a new object:

```java
BigDecimal a = new BigDecimal("10.50");
BigDecimal b = new BigDecimal("3.25");

BigDecimal sum        = a.add(b);         // 13.75
BigDecimal difference = a.subtract(b);    // 7.25
BigDecimal product    = a.multiply(b);    // 34.1250
BigDecimal quotient   = a.divide(b, 2, RoundingMode.HALF_UP); // 3.23
```

Notice that `divide` requires a scale and rounding mode. Without them, `divide` throws an `ArithmeticException` if the result is a non-terminating decimal:

```java
BigDecimal one   = new BigDecimal("1");
BigDecimal three = new BigDecimal("3");

// Throws ArithmeticException: Non-terminating decimal expansion
BigDecimal bad = one.divide(three);

// Correct: specify scale and rounding mode
BigDecimal safe = one.divide(three, 10, RoundingMode.HALF_UP);
```

Always specify a scale and rounding mode when dividing, unless you are certain the result is exact.

For comparison, use `compareTo` rather than `equals`. The `compareTo` method compares numerical value (so `3.0` and `3.00` are equal), while `equals` also compares scale (so `3.0` and `3.00` are not equal):

```java
BigDecimal x = new BigDecimal("3.0");
BigDecimal y = new BigDecimal("3.00");

x.compareTo(y) == 0  // true  (same value)
x.equals(y)          // false (different scale)
```

This distinction matters when using BigDecimal in collections. A `HashSet` uses `equals` and `hashCode`, so it would treat `3.0` and `3.00` as different entries. A `TreeSet` uses `compareTo`, so it would treat them as the same.

#### Rounding

By rounding a number, we replace it with another having a shorter, simpler representation. Java provides two classes to control rounding behavior.

The enum **RoundingMode** defines eight rounding strategies:

* `CEILING`: rounds towards positive infinity
* `FLOOR`: rounds towards negative infinity
* `UP`: rounds away from zero
* `DOWN`: rounds towards zero
* `HALF_UP`: rounds towards the nearest neighbor; if equidistant, rounds up
* `HALF_DOWN`: rounds towards the nearest neighbor; if equidistant, rounds down
* `HALF_EVEN`: rounds towards the nearest neighbor; if equidistant, rounds towards the even neighbor (also known as banker's rounding)
* `UNNECESSARY`: asserts that no rounding is needed; throws `ArithmeticException` if it is

**MathContext** combines a precision (number of significant digits) with a rounding mode:

```java
BigDecimal bd = new BigDecimal("7.5");
BigDecimal rounded = bd.round(new MathContext(2, RoundingMode.HALF_EVEN));
// 7.5 -> 7.5 (already 2 significant digits)
```

MathContext also provides predefined contexts like `DECIMAL32` (7-digit precision, HALF_EVEN), `DECIMAL64` (16 digits), and `DECIMAL128` (34 digits) that mirror the IEEE 754 decimal formats.

### BigInteger

**BigInteger** represents an immutable, arbitrary-precision integer. It is useful when values exceed the range of `long` (which maxes out at about 9.2 x 10^18). A classic example is computing large factorials: 100! has 158 digits and cannot fit in any primitive type.

```java
BigInteger biFromString    = new BigInteger("1289231289098714924");
BigInteger biFromLong      = BigInteger.valueOf(7897343012089369395L);
BigInteger biFromByteArray = new BigInteger(new byte[] { 64, 64, 64, 64, 64, 64 });
```

BigInteger provides operations analogous to Java's primitive integer operators: `add`, `subtract`, `multiply`, `divide`, `mod`, `pow`, `abs`, `min`, `max`, and `signum`. It also supports bitwise operations (`and`, `or`, `xor`, `not`, `shiftLeft`, `shiftRight`), modular arithmetic (`modPow`, `modInverse`), and primality testing (`isProbablePrime`, `probablePrime`).

```java
BigInteger a = new BigInteger("50");
BigInteger b = new BigInteger("18");
BigInteger c = new BigInteger("12");

BigInteger gcd = b.gcd(c);                   // 6
BigInteger product = b.multiply(c).mod(a);    // 16
BigInteger modPow = b.modPow(c, a);           // 32
```

Like BigDecimal, BigInteger is immutable: every operation returns a new object. And like BigDecimal, use `compareTo` for value comparison rather than `==` (which compares object references).

### When to Use Which

Use **BigDecimal** when you need exact decimal arithmetic: monetary values, tax calculations, interest rates, or any domain where rounding errors are unacceptable. The cost is performance: BigDecimal operations are significantly slower than primitive `double` arithmetic, so use it where correctness matters more than speed.

Use **BigInteger** when you need integers larger than `long` can hold: cryptographic calculations, combinatorics, or any domain with very large whole numbers.

For everything else, `int`, `long`, and `double` are simpler and faster.
