# Decimal floating point arithmetic for micropython
This Python module for [*micropython*](https://micropython.org/) and [*CircuitPython*](https://circuitpython.org/) provides support for decimal floating point arithmetic. It tries to overcome the limitations of single precision float numbers (32-bit) and provides a solution when double precision float numbers (64 bit) are not enough.

The Python Standard Library contains the wonderful module [*decimal*](https://docs.python.org/3/library/decimal.html), but it has not been ported to *micropython*. This module provides a small, but valuable, part of the functionality of *decimal*.

## Introduction

The module **mpy_decimal** defines the class **DecimalNumber** that contains all the functionality for decimal floating point arithmetic. A **DecimalNumber** can be of arbitrary precision. Internally, it is composed of an *int* that contains all the digits of the **DecimalNumber**, an *int* equal to the number of decimal places and a *bool* that determines whether the **DecimalNumber** is positive or negative. Example:

    DecimalNumber: -12345678901.23456789
        number   = 1234567890123456789
        decimals = 8
        positive = False

The precision of **DecimalNumber** is mainly limited by available memory and procesing power. **DecimalNumber** uses the concept **scale**, which is the number of decimal places that the class uses for its numbers and operations. The concept is similar to the use of 'scale' in the calculator and language [*bc*](https://www.gnu.org/software/bc/manual/html_mono/bc.html).  The default value for **scale** is 16. It is a global value of the class that can be changed at any time. For rounding, **DecimalNumber** uses [*round half to even*](https://en.wikipedia.org/wiki/Rounding#Round_half_to_even).

## Performance ##

All the internal operations of **DecimalNumber** are done with integers (*int* built-in type of Python) and the number of decimals are adjusted according to the operation. It is fast, but not as fast as Python's *decimal* class because **DecimalNumber** is pure Python and *decimal* is written in C. The *test* folder contains the file "*perf_decimal_number.py*" that calculates the performance of **DecimalNumber** on the device where it runs. This is the output of that program executed on a [*Raspberry Pi Pico*](https://www.raspberrypi.org/products/raspberry-pi-pico/). Basic operations take about one millisecond with scale = 16:

    +---------------------------------------------------------------+
    |  SYSTEM INFORMATION                                           |
    +---------------------------------------------------------------+
    Implementation name:           micropython
    Implementation version:        1.17.0
    Implementation platform:       rp2
    CPU frequency:                 125 Mhz

    +---------------------------------------------------------------+
    |  PERFORMANCE WITH SCALE = 16                                  |
    +---------------------------------------------------------------+
    Scale (max. decimals):         16
    Iterations per test:           1000
    Number 1:                      63107666423864.1503618336011148
    Number 2:                      21513455188640.8462728528253754
    Addition (n1 + n2):            1.564 ms
    Subtraction (n1 - n2):         1.695 ms
    Multiplication (n1 * n2):      0.988 ms
    Division (n1 / n2):            1.184 ms
    Square root abs(n1):           3.984 ms
    Power: (pi/2) ** 15            9.799 ms
    DecimalNumber from int:        0.378 ms
    DecimalNumber from string:     3.685 ms
    Iterations per test:           10
    Sine: sin(0.54321)             85.80001 ms
    Cosine: cos(0.54321)           86.7 ms
    Tangent: tan(0.54321)          206.7 ms
    Arcsine: asin(0.54321)         270.2 ms
    Arccosine: acos(0.65432)       468.8 ms
    Arctangent: atan(1.2345)       485.5 ms
    Arctangent2: atan2(1.23, 2.34) 349.2 ms
    Exponential: exp(12.345)       151.0 ms
    Natural logarithm: ln(12.345)  152.0 ms

    +---------------------------------------------------------------+
    |  PERFORMANCE WITH SCALE = 50                                  |
    +---------------------------------------------------------------+
    Scale (max. decimals):         50
    Iterations per test:           400
    Number 1:                      1008440840554324243744283306032711702.13787157234850455642441824006520576852374144010113
    Number 2:                      -603053031456028063831871487068.81514266625502376325215658564112814751837118860547
    Addition (n1 + n2):            1.9575 ms
    Subtraction (n1 - n2):         2.25 ms
    Multiplication (n1 * n2):      1.4625 ms
    Division (n1 / n2):            1.52 ms
    Square root abs(n1):           12.455 ms
    Power: (pi/2) ** 15            11.7425 ms
    DecimalNumber from int:        0.385 ms
    DecimalNumber from string:     10.055 ms
    Iterations per test:           4
    Sine: sin(0.54321)             186.25 ms
    Cosine: cos(0.54321)           184.5 ms
    Tangent: tan(0.54321)          398.75 ms
    Arcsine: asin(0.54321)         831.25 ms
    Arccosine: acos(0.65432)       1322.25 ms
    Arctangent: atan(1.2345)       1377.75 ms
    Arctangent2: atan2(1.23, 2.34) 893.4999 ms
    Exponential: exp(12.345)       281.75 ms
    Natural logarithm: ln(12.345)  281.75 ms

    +---------------------------------------------------------------+
    |  CALCULATING PI                                               |
    +---------------------------------------------------------------+
    Pi with 300 decimals:          5.862 s
    3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622948954930381964428810975665933446128475648233786783165271201909145648566923460348610454326648213393607260249141274

## How to use

To test this module on a PC, you can start by importing the module:

```python
from mpy_decimal.mpy_decimal import *
```

For testing on a Micropython board, you have probably copied the file 'mpy_decimal.py' to the board or a compiled version of it: 'mpy_decimal.mpy'. There is a guide that explains how to compile this module and how to copy it to a device at the end of this document. You can import the module with:

```python
from mpy_decimal import *
```

If you need your code to run on both, a computer and a micropython board, you will probably need to run different code depending on the device. You can do it this way:

```python
import sys

if sys.implementation.name == "cpython":
    # ... your imports or other code for CPython here ...

if sys.implementation.name == "micropython":
    # ... your imports or other code for Micropython here ...
```

(Note: 'sys' is standard Python and it has nothing to do with **DecimalNumber**)

### Initialization ###
A **DecimalNumber** with default value, equal to zero:


```python
n = DecimalNumber()
```

An integer, for example, 748:

```python
n = DecimalNumber(748)
```

A decimal number, for example, 93402.5184:

```python
n = DecimalNumber(934025184, 4)
```

Notice that the first parameter is an integer with all the digits and the second one the number of decimals.

The same number can be created providing a string with the number:

```python
n = DecimalNumber("93402.5184")
```

### Printing and formating ###
Numbers can be printed using 'print()':

```python
print(n)
# Result: 93402.5184
```

They can be converted to a string using 'str()':

```python
str(n)
```


The method **to_string_thousands()** of **DecimalNumber** returns a string with the number formatted with ',' as thousands separator. Decimals figures are not modified:

```python
print(n.to_string_thousands())
# Result: 93,402.5184
```

Micropython can be used to print information on displays with a limited number of characters. For example, on a 16x2 LCD (two lines of 16 characters). For these kind of cases exists the method **to_string_max_length()**. It limits the representation of the number to a maximum length of characters, including '.' and '-'. The minimum value is 8. If decimals cannot fit in, they are discarded. If the integer part of the number is bigger than the maximum length, the result is the string "Overflow". Some examples:

```python
n = DecimalNumber("123456789.012")
#                           ¹¹¹¹
#                  ¹²³⁴⁵⁶⁷⁸⁹⁰¹²³ 

print(n.to_string_max_length(12))
# Result: 123456789.01

print(n.to_string_max_length(11))
# Result: 123456789

print(n.to_string_max_length(8))
# Result: Overflow
```

### Modifying the **scale** of **DecimalNumber** ###

**scale** is a global value of the class **DecimalNumber** that stores the number of decimals that the class uses for its numbers an operations. The default value is 16. **DecimalNumber.get_scale()** returns the current **scale** and the method **DecimalNumber.set_scale()** sets **scale**:

```python
current_scale = DecimalNumber.get_scale()   # Gets the scale
DecimalNumber.set_scale(100)                # Sets the scale to 100

    # ... Code to calculate something using the new scale

DecimalNumber.set_scale(current_scale)      # Back to the previous scale 
```

### Operations ###

**Basic operations**

The basic operations (addition, subtraction, multiplication and division) allow to mix **DecimalNumber** objects and *int* objects. The result is always a **DecimalNumber**. Examples:

```python
a = DecimalNumber("7.3329")
b = DecimalNumber("157.82")

# Addition
c = a + b       # DecimalNumber + DecimalNumber
c = a + 5       # DecimalNumber + int
c = 5 + a       # int + DecimalNumber
a += b          # DecimalNumber + DecimalNumber
a += 3          # DecimalNumber + int

# Subtraction
c = a - b       # DecimalNumber - DecimalNumber
c = a - 5       # DecimalNumber - int
c = 5 - a       # int - DecimalNumber
a -= b          # DecimalNumber - DecimalNumber
a -= 3          # DecimalNumber - int

# Multiplication
c = a * b       # DecimalNumber * DecimalNumber
c = a * 5       # DecimalNumber * int
c = 5 * a       # int * DecimalNumber
a *= b          # DecimalNumber * DecimalNumber
a *= 3          # DecimalNumber * int

# Division
c = a / b       # DecimalNumber / DecimalNumber
c = a / 5       # DecimalNumber / int
c = 5 / a       # int / DecimalNumber
a /= b          # DecimalNumber / DecimalNumber
a /= 3          # DecimalNumber / int
```
**Exponentiation**

The operands for exponentition are a **DecimalNumber**, the base, and an *int*, the exponent. It calculates the base raised to the exponent. Examples:

```python
a = DecimalNumber("1.01234567")
b = a ** 15     # a¹⁵ = 1.2020774344056969

# 11th Mersenne prime = 2¹⁰⁷ - 1 = 162259276829213363391578010288127
m11 = DecimalNumber(2) ** 107 - 1
```

**Square root**

It calculates the square root of a positive **DecimalNumber**. For negative numbers, a **DecimalNumberExceptionMathDomainError** exception is raised. Example:

```python
a = DecimalNumber("620433.785")
b = a.square_root()
print(b)        # Result: 787.6761929879561873
```

**Absolute**

It returns the absolute value of a **DecimalNumber**. Examples:

```python
a = DecimalNumber("12.762")
b = DecimalNumber("-12.762")
print(abs(a))   # Result: 12.762
print(abs(b))   # Result: 12.762
```

**Unary - operator**

Given a **DecimalNumber** n, it returns -n. Example:

```python
a = DecimalNumber("12.762")
b = DecimalNumber("-12.762")
print(-a)   # Result: -12.762
print(-b)   # Result: 12.762
```

**Unary + operator**

Given a **DecimalNumber** n, it returns +n. This can give the idea that it does nothing to the number n. That is true if the number of decimals of n is less or equal than the **scale** of **DecimalNumber**. If it is not, if the number of decimals of n is greater than **scale**, the decimals of n are reduced and rounded to match **scale**.

This operator can be useful to adjust the **scale** of a number to the **scale** of **DecimalNumber** after changing the **scale** with the method **DecimalNumber.set_scale()**. An example of this is a method that increases **scale** to obtain better accuracy in the calculation of a number x. Before returning from the method, it sets **scale** back to the value it had before entering the method and it returns +x instead of just x, adjusting the decimals of x to **scale**. Example:

```python
a = DecimalNumber(2)
DecimalNumber.set_scale(30)     # Sets the scale = 30
r2 = a.square_root()            # Calculates the square root of 2 with 30 decimals
print(r2)                       # 1.414213562373095048801688724209
DecimalNumber.set_scale(10)     # Sets the scale = 10
print(r2)                       # 1.414213562373095048801688724209
                                # Changing the scale does not modify a number
                                # unless an operation is made with it, like,
                                # for example, the unary + operator:
print(+r2)                      # 1.4142135624
```

### Comparisons operators ###

These are the common comparison operators we all know: <, <=, ==, !=, >=, >.

A **DecimalNumber** can be compared to other **DecimalNumber** or to an *int* number. Examples:

```python
a = DecimalNumber("1.2")
b = DecimalNumber("8.3")
print(a > b)    # False
print(b > a)    # True
print(a == b)   # False
print(a != b)   # True
print(a > 0)    # True
print(1 > b)    # False
```

### Other mathematical functions ###

**exp()**: exponential function e<sup>x</sup>.

**ln()**: natural logarithm (base e).

**Trigonometric functions**:

**sin()**: sine.

**cos()**: cosine.

**tan()**: tangent.

The argument of trigonometric functions sin(), cos() and tan() is an angle expressed in radians.

**asin()**: arcsine.

**acos()**: arccosine.

**atan()**: arctangent.

**atan2()**: 2-argument arctangent.

The functions asin(), acos(), atan() and atan2() return an angle in radians.

Example:

```python
n = DecimalNumber("0.732")
n.exp()     # 2.0792349218188443
n.ln()      # -0.3119747650208255
n.sin()     # 0.6683586490759965
n.cos()     # 0.7438391736157144
n.tan()     # 0.8985257469396026
n.asin()    # 0.821252884452186
n.acos()    # 0.7495434423427106
n.atan()    # 0.6318812315412357
n2 = DecimalNumber("1.732")
DecimalNumber.atan(n, n2)   # 0.3998638956924461
```

### Other methods ###

**to_int_truncate()** returns and *int* that contains the integer part of a **DecimalNumber** after truncating the decimals.

**to_int_round()** returns and *int* that contains the integer part of a **DecimalNumber** after rounding it to zero decimals.

Example:

```python
a = DecimalNumber("623897401.877314")
print(a.to_int_truncate())  # 623897401
print(a.to_int_round())     # 623897402
```

**clone()** returns a new **DecimalNumber** object as a clone of a **DecimalNumber**. Example:

```python
a = DecimalNumber("657.31")
b = a.clone()
print(a == b)   # True: they are equal numbers.
print(a is b)   # False: they are different objects.
```

**copy_from()** copies into a **DecimalNumber** other **DecimalNumber** passed as a parameter. The reason for this method is that Python does not allow to overload the *assign* operator: '='. "b = a" would make b to point to object a, they would be the same object. We need a way to copy one into the other while remaining independent objects. Example:

```python
a = DecimalNumber("657.31")
b = DecimalNumber("-1.9")
b = a
print(a, b)     # 657.31 657.31
print(a is b)   # True: they are the same object

# Let's try again with 'copy_from()'

a = DecimalNumber("657.31")
b = DecimalNumber("-1.9")
b.copy_from(a)
print(a, b)     # 657.31 657.31
print(a is b)   # False: they are different objects.
```

**pi()** is a class method that returns the number PI with as many decimals as the **scale** of **DecimalNumber**. Example:

```python
pi = DecimalNumber.pi()         # Default scale, equal to 16
print(DecimalNumber.pi())       # 3.1415926535897932
DecimalNumber.set_scale(36)     # Set scale = 36
print(DecimalNumber.pi())       # 3.141592653589793238462643383279502884
```

PI is precalculated with 100 decimals and stored in the class. If **pi()** method is used with **scale** <= 100, PI is not calculated, but returned using the precalculated value. If **scale** is set to a value greater than 100, for example, 300, PI is calculated, stored in the class and returned. After that, the precalculated limit is 300 instead of 100, and any call to **pi()** with a **scale** <= 300 returns the value of PI from the precalculated value.

To calculate PI, this method uses the very fast algorithm present on the section [Recipes](https://docs.python.org/3/library/decimal.html#recipes) of the documentation for the module **decimal**, part of Python Standard Library.

**e()** is a class method that returns the number e number, the base of natural logarithms, with as many decimals as the **scale** of **DecimalNumber**. Its value is precalculated and it functions in a similar way as pi(). Example:

```python
e = DecimalNumber.e()           # Default scale, equal to 16
print(DecimalNumber.pi())       # 2.7182818284590452
DecimalNumber.set_scale(36)     # Set scale = 36
print(DecimalNumber.e())        # 2.718281828459045235360287471352662498
```

### Other considerations ###

**DecimalNumber** class can operate mixing *int* numbers and **DecimalNumber** objects. *float* numbers were not considered because of their imprecision.

Try this using CPython (on a PC):
```python
print( (0.1 + 0.1 + 0.1) == 0.3 )   # False!!!
```

There is nothing wrong with Python, it is the way *float* numbers work.

Try this using Micropython (I have used a Raspberry Pi Pico):

```python
# Compares 10¹¹ * 10⁻² to 10⁹, which should be True
print( (1e11 * 1e-2) == 1e9 )       # False!!!
```


## Exceptions ##

This module defines four exceptions:

* **DecimalNumberExceptionParseError**: a 
**DecimalNumber** can be initialize providing a string that contains a number. If the content of the string cannot be parsed as a correct number, this exception is raised.

* **DecimalNumberExceptionBadInit**: this exception is raised when a negative number of decimals is provided when initializing a **DecimalNumber**.

* **DecimalNumberExceptionMathDomainError**: this exception occurs when trying to calculate the square root of a negative number or atan2(0, 0).

* **DecimalNumberExceptionDivisionByZeroError**: this is the division by zero exception.


## Example ##

This example shows how to use **DecimalNumber** to solve quadratic equations.

```python
from mpy_decimal.mpy_decimal import *


def solve_quadratic_equation(a: DecimalNumber, b: DecimalNumber, c: DecimalNumber) -> Tuple[bool, DecimalNumber, DecimalNumber]:
    """It solves quadratic equations:
    a * x² + b * x + c = 0
    x₁ = (-b + sqrt(b*b - 4*a*c)) / (2*a)
    x₂ = (-b - sqrt(b*b - 4*a*c)) / (2*a)
    """

    # This is done by catching the Exception raised when calculating
    # the square root of a negative number. It is instructive, but it
    # can be avoided by simply checking if (b * b - 4 * a * c) is a
    # negative number before calling square_root().
    try:
        r = (b * b - 4 * a * c).square_root()
        x1 = (-b + r) / (2 * a)
        x2 = (-b - r) / (2 * a)
        return True, x1, x2
    except DecimalNumberExceptionMathDomainError:
        return False, None, None


def string_equation(a: DecimalNumber, b: DecimalNumber, c: DecimalNumber) -> str:
    return "{0}x² {1}x {2} = 0".format(
        a,
        "- " + str(abs(b)) if b < 0 else '+ ' + str(b),
        "- " + str(abs(c)) if c < 0 else '+ ' + str(c)
    )


list_equations = [
    (7, -5, -9),
    (1, -3, 10),
    (4, 25, 21),
    (1, 3, -10)
]

print("-" * 50)
for e in list_equations:
    a = DecimalNumber(e[0])
    b = DecimalNumber(e[1])
    c = DecimalNumber(e[2])
    solution, x1, x2 = solve_quadratic_equation(a, b, c)
    print(string_equation(a, b, c))
    if solution:
        print("   x₁ =", x1)
        print("   x₂ =", x2)
    else:
        print("   The equation does not have a real solution.")
    print("-" * 50)
```

This is what the program prints:

    --------------------------------------------------
    7x² - 5x - 9 = 0
       x1 = 1.545951212649517
       x2 = -0.8316654983638027
    --------------------------------------------------
    1x² - 3x + 10 = 0
       The equation does not have a real solution.
    --------------------------------------------------
    4x² + 25x + 21 = 0
       x1 = -1
       x2 = -5.25
    --------------------------------------------------
    1x² + 3x - 10 = 0
       x₁ = 2
       x₂ = -5
    --------------------------------------------------


## Using DecimalNumber on a Micropython board ##

This is not exactly part of this module, but someone might find it useful.

**".mpy" files**: a micropython board can run ".py" files directly. It compiles them before executing them, and that takes time. It is a good practice to copy to your micropython board a precompiled version, a ".mpy" file.

You can compile the Python file "mpy_decimal.py", that contains all the functionality of **DecimalNumber**, before copying it to your micropython board:

```shell
python -m mpy_cross mpy_decimal.py
```

It creates the file "mpy_decimal.mpy". You should have **mpy_cross** installed. More information at: [https://pypi.org/project/mpy-cross/](https://pypi.org/project/mpy-cross/)

The next step is copying the ".mpy" file to your micropython board. You can copy python source code files with an editor, for example [**Thonny**](https://thonny.org/), but it does not copy (as of today) ".mpy" files. The tool [**mpy-repl-tool[mount]**](https://mpy-repl-tool.readthedocs.io/en/latest/index.html) can help you connect your computer to your micropython board. It also allows to mount your board and use it as any other disk or memory unit of your computer.
