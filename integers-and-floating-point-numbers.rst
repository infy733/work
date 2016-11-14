.. 
  .. _man-integers-and-floating-point-numbers:

  .. currentmodule:: Base

  *************************************
   Integers and Floating-Point Numbers
  *************************************

  Integers and floating-point values are the basic building blocks of
  arithmetic and computation. Built-in representations of such values are
  called numeric primitives, while representations of integers and
  floating-point numbers as immediate values in code are known as numeric
  literals. For example, ``1`` is an integer literal, while ``1.0`` is a
  floating-point literal; their binary in-memory representations as
  objects are numeric primitives.

  Julia provides a broad range of primitive numeric types, and a full complement
  of arithmetic and bitwise operators as well as standard mathematical functions
  are defined over them. These map directly onto numeric types and operations
  that are natively supported on modern computers, thus allowing Julia to take
  full advantage of computational resources. Additionally, Julia provides
  software support for :ref:`man-arbitrary-precision-arithmetic`, which can
  handle operations on numeric values that cannot be represented effectively in
  native hardware representations, but at the cost of relatively slower
  performance.

  The following are Julia's primitive numeric types:

  -  **Integer types:**

  ================  =======  ==============  ============== ==================
  Type              Signed?  Number of bits  Smallest value Largest value
  ================  =======  ==============  ============== ==================
  :class:`Int8`        ✓         8            -2^7             2^7 - 1
  :class:`UInt8`                 8             0               2^8 - 1
  :class:`Int16`       ✓         16           -2^15            2^15 - 1
  :class:`UInt16`                16            0               2^16 - 1
  :class:`Int32`       ✓         32           -2^31            2^31 - 1
  :class:`UInt32`                32            0               2^32 - 1
  :class:`Int64`       ✓         64           -2^63            2^63 - 1
  :class:`UInt64`                64            0               2^64 - 1
  :class:`Int128`      ✓         128           -2^127          2^127 - 1
  :class:`UInt128`               128           0               2^128 - 1
  :class:`Bool`       N/A        8           ``false`` (0)  ``true`` (1)
  ================  =======  ==============  ============== ==================

  -  **Floating-point types:**

  ================ ========= ==============
  Type             Precision Number of bits
  ================ ========= ==============
  :class:`Float16` half_          16
  :class:`Float32` single_        32
  :class:`Float64` double_        64
  ================ ========= ==============

  .. _half: https://en.wikipedia.org/wiki/Half-precision_floating-point_format
  .. _single: https://en.wikipedia.org/wiki/Single_precision_floating-point_format
  .. _double: https://en.wikipedia.org/wiki/Double_precision_floating-point_format

  Additionally, full support for :ref:`man-complex-and-rational-numbers` is built
  on top of these primitive numeric types. All numeric types interoperate
  naturally without explicit casting, thanks to a flexible, user-extensible
  :ref:`type promotion system <man-conversion-and-promotion>`.

  Integers
  --------

  Literal integers are represented in the standard manner:

  .. doctest::

      julia> 1
      1

      julia> 1234
      1234

  The default type for an integer literal depends on whether the target
  system has a 32-bit architecture or a 64-bit architecture::

      # 32-bit system:
      julia> typeof(1)
      Int32

      # 64-bit system:
      julia> typeof(1)
      Int64

  The Julia internal variable :const:`Sys.WORD_SIZE` indicates whether the target system
  is 32-bit or 64-bit.::

      # 32-bit system:
      julia> Sys.WORD_SIZE
      32

      # 64-bit system:
      julia> Sys.WORD_SIZE
      64

  Julia also defines the types :class:`Int` and :class:`UInt`, which are aliases for the
  system's signed and unsigned native integer types respectively.::

      # 32-bit system:
      julia> Int
      Int32
      julia> UInt
      UInt32


      # 64-bit system:
      julia> Int
      Int64
      julia> UInt
      UInt64

  Larger integer literals that cannot be represented using only 32 bits
  but can be represented in 64 bits always create 64-bit integers,
  regardless of the system type::

      # 32-bit or 64-bit system:
      julia> typeof(3000000000)
      Int64

  Unsigned integers are input and output using the ``0x`` prefix and hexadecimal
  (base 16) digits ``0-9a-f`` (the capitalized digits ``A-F`` also work for input).
  The size of the unsigned value is determined by the number of hex digits used:

  .. doctest::

      julia> 0x1
      0x01

      julia> typeof(ans)
      UInt8

      julia> 0x123
      0x0123

      julia> typeof(ans)
      UInt16

      julia> 0x1234567
      0x01234567

      julia> typeof(ans)
      UInt32

      julia> 0x123456789abcdef
      0x0123456789abcdef

      julia> typeof(ans)
      UInt64

  This behavior is based on the observation that when one uses unsigned
  hex literals for integer values, one typically is using them to
  represent a fixed numeric byte sequence, rather than just an integer
  value.

  Recall that the variable :data:`ans` is set to the value of the last expression
  evaluated in an interactive session. This does not occur when Julia code is
  run in other ways.

  Binary and octal literals are also supported:

  .. doctest::

      julia> 0b10
      0x02

      julia> typeof(ans)
      UInt8

      julia> 0o10
      0x08

      julia> typeof(ans)
      UInt8

  The minimum and maximum representable values of primitive numeric types
  such as integers are given by the :func:`typemin` and :func:`typemax` functions:

  .. doctest::

      julia> (typemin(Int32), typemax(Int32))
      (-2147483648,2147483647)

      julia> for T in [Int8,Int16,Int32,Int64,Int128,UInt8,UInt16,UInt32,UInt64,UInt128]
               println("$(lpad(T,7)): [$(typemin(T)),$(typemax(T))]")
             end
         Int8: [-128,127]
        Int16: [-32768,32767]
        Int32: [-2147483648,2147483647]
        Int64: [-9223372036854775808,9223372036854775807]
       Int128: [-170141183460469231731687303715884105728,170141183460469231731687303715884105727]
        UInt8: [0,255]
       UInt16: [0,65535]
       UInt32: [0,4294967295]
       UInt64: [0,18446744073709551615]
      UInt128: [0,340282366920938463463374607431768211455]

  The values returned by :func:`typemin` and :func:`typemax` are always of the
  given argument type. (The above expression uses several features we have
  yet to introduce, including :ref:`for loops <man-loops>`,
  :ref:`man-strings`, and :ref:`man-string-interpolation`,
  but should be easy enough to understand for users with some existing
  programming experience.)


  Overflow behavior
  ~~~~~~~~~~~~~~~~~

  In Julia, exceeding the maximum representable value of a given type results in
  a wraparound behavior:

  .. doctest::

      julia> x = typemax(Int64)
      9223372036854775807

      julia> x + 1
      -9223372036854775808

      julia> x + 1 == typemin(Int64)
      true

  Thus, arithmetic with Julia integers is actually a form of `modular arithmetic
  <https://en.wikipedia.org/wiki/Modular_arithmetic>`_. This reflects the
  characteristics of the underlying arithmetic of integers as implemented on
  modern computers. In applications where overflow is possible, explicit checking
  for wraparound produced by overflow is essential; otherwise, the ``BigInt`` type
  in :ref:`man-arbitrary-precision-arithmetic` is recommended instead.

  Division errors
  ~~~~~~~~~~~~~~~

  Integer division (the ``div`` function) has two exceptional cases: dividing by
  zero, and dividing the lowest negative number (:func:`typemin`) by -1. Both of
  these cases throw a :exc:`DivideError`. The remainder and modulus functions
  (``rem`` and ``mod``) throw a :exc:`DivideError` when their second argument is
  zero.

  Floating-Point Numbers
  ----------------------

  Literal floating-point numbers are represented in the standard formats:

  .. doctest::

      julia> 1.0
      1.0

      julia> 1.
      1.0

      julia> 0.5
      0.5

      julia> .5
      0.5

      julia> -1.23
      -1.23

      julia> 1e10
      1.0e10

      julia> 2.5e-4
      0.00025

  The above results are all ``Float64`` values. Literal ``Float32`` values can
  be entered by writing an ``f`` in place of ``e``:

  .. doctest::

      julia> 0.5f0
      0.5f0

      julia> typeof(ans)
      Float32

      julia> 2.5f-4
      0.00025f0

  Values can be converted to ``Float32`` easily:

  .. doctest::

      julia> Float32(-1.5)
      -1.5f0

      julia> typeof(ans)
      Float32

  Hexadecimal floating-point literals are also valid, but only as ``Float64`` values:

  .. doctest::

      julia> 0x1p0
      1.0

      julia> 0x1.8p3
      12.0

      julia> 0x.4p-1
      0.125

      julia> typeof(ans)
      Float64

  Half-precision floating-point numbers are also supported (``Float16``), but
  only as a storage format. In calculations they'll be converted to ``Float32``:

  .. doctest::

      julia> sizeof(Float16(4.))
      2

      julia> 2*Float16(4.)
      Float16(8.0)

  The underscore ``_`` can be used as digit separator:

  .. doctest::

      julia> 10_000, 0.000_000_005, 0xdead_beef, 0b1011_0010
      (10000,5.0e-9,0xdeadbeef,0xb2)

  Floating-point zero
  ~~~~~~~~~~~~~~~~~~~

  Floating-point numbers have `two zeros
  <https://en.wikipedia.org/wiki/Signed_zero>`_, positive zero and negative zero.
  They are equal to each other but have different binary representations, as can
  be seen using the ``bits`` function: :

  .. doctest::

      julia> 0.0 == -0.0
      true

      julia> bits(0.0)
      "0000000000000000000000000000000000000000000000000000000000000000"

      julia> bits(-0.0)
      "1000000000000000000000000000000000000000000000000000000000000000"

  .. _man-special-floats:

  Special floating-point values
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  There are three specified standard floating-point values that do not
  correspond to any point on the real number line:

  =========== =========== ===========  ================= =================================================================
  Special value                        Name              Description
  -----------------------------------  ----------------- -----------------------------------------------------------------
  ``Float16`` ``Float32`` ``Float64``
  =========== =========== ===========  ================= =================================================================
  ``Inf16``   ``Inf32``    ``Inf``     positive infinity a value greater than all finite floating-point values
  ``-Inf16``  ``-Inf32``   ``-Inf``    negative infinity a value less than all finite floating-point values
  ``NaN16``   ``NaN32``    ``NaN``     not a number      a value not ``==`` to any floating-point value (including itself)
  =========== =========== ===========  ================= =================================================================

  For further discussion of how these non-finite floating-point values are
  ordered with respect to each other and other floats, see
  :ref:`man-numeric-comparisons`. By the
  `IEEE 754 standard <https://en.wikipedia.org/wiki/IEEE_754-2008>`_, these
  floating-point values are the results of certain arithmetic operations:

  .. doctest::

      julia> 1/Inf
      0.0

      julia> 1/0
      Inf

      julia> -5/0
      -Inf

      julia> 0.000001/0
      Inf

      julia> 0/0
      NaN

      julia> 500 + Inf
      Inf

      julia> 500 - Inf
      -Inf

      julia> Inf + Inf
      Inf

      julia> Inf - Inf
      NaN

      julia> Inf * Inf
      Inf

      julia> Inf / Inf
      NaN

      julia> 0 * Inf
      NaN

  The :func:`typemin` and :func:`typemax` functions also apply to floating-point
  types:

  .. doctest::

      julia> (typemin(Float16),typemax(Float16))
      (-Inf16,Inf16)

      julia> (typemin(Float32),typemax(Float32))
      (-Inf32,Inf32)

      julia> (typemin(Float64),typemax(Float64))
      (-Inf,Inf)

  Machine epsilon
  ~~~~~~~~~~~~~~~

  Most real numbers cannot be represented exactly with floating-point numbers,
  and so for many purposes it is important to know the distance between two
  adjacent representable floating-point numbers, which is often known as
  `machine epsilon <https://en.wikipedia.org/wiki/Machine_epsilon>`_.

  Julia provides :func:`eps`, which gives the distance between ``1.0``
  and the next larger representable floating-point value:

  .. doctest::

      julia> eps(Float32)
      1.1920929f-7

      julia> eps(Float64)
      2.220446049250313e-16

      julia> eps() # same as eps(Float64)
      2.220446049250313e-16

  These values are ``2.0^-23`` and ``2.0^-52`` as ``Float32`` and ``Float64``
  values, respectively. The :func:`eps` function can also take a
  floating-point value as an argument, and gives the absolute difference
  between that value and the next representable floating point value. That
  is, ``eps(x)`` yields a value of the same type as ``x`` such that
  ``x + eps(x)`` is the next representable floating-point value larger
  than ``x``:

  .. doctest::

      julia> eps(1.0)
      2.220446049250313e-16

      julia> eps(1000.)
      1.1368683772161603e-13

      julia> eps(1e-27)
      1.793662034335766e-43

      julia> eps(0.0)
      5.0e-324

  The distance between two adjacent representable floating-point numbers is not
  constant, but is smaller for smaller values and larger for larger values. In
  other words, the representable floating-point numbers are densest in the real
  number line near zero, and grow sparser exponentially as one moves farther away
  from zero. By definition, ``eps(1.0)`` is the same as ``eps(Float64)`` since
  ``1.0`` is a 64-bit floating-point value.

  Julia also provides the :func:`nextfloat` and :func:`prevfloat` functions which return
  the next largest or smallest representable floating-point number to the
  argument respectively: :

  .. doctest::

      julia> x = 1.25f0
      1.25f0

      julia> nextfloat(x)
      1.2500001f0

      julia> prevfloat(x)
      1.2499999f0

      julia> bits(prevfloat(x))
      "00111111100111111111111111111111"

      julia> bits(x)
      "00111111101000000000000000000000"

      julia> bits(nextfloat(x))
      "00111111101000000000000000000001"

  This example highlights the general principle that the adjacent representable
  floating-point numbers also have adjacent binary integer representations.

  Rounding modes
  ~~~~~~~~~~~~~~

  If a number doesn't have an exact floating-point representation, it must be
  rounded to an appropriate representable value, however, if wanted, the manner
  in which this rounding is done can be changed according to the rounding modes
  presented in the `IEEE 754 standard <https://en.wikipedia.org/wiki/IEEE_754-2008>`_.

  .. doctest::

      julia> x = 1.1; y = 0.1;

      julia> x + y
      1.2000000000000002

      julia> setrounding(Float64,RoundDown) do
                 x + y
             end
      1.2

  The default mode used is always :const:`RoundNearest`, which rounds to the nearest
  representable value, with ties rounded towards the nearest value with an even
  least significant bit.

  .. warning:: Rounding is generally only correct for basic arithmetic functions
         (:func:`+`, :func:`-`, :func:`*`, :func:`/` and :func:`sqrt`) and
         type conversion operations. Many other functions assume the
         default :const:`RoundNearest` mode is set, and can give erroneous
         results when operating under other rounding modes.

  Background and References
  ~~~~~~~~~~~~~~~~~~~~~~~~~

  Floating-point arithmetic entails many subtleties which can be surprising to
  users who are unfamiliar with the low-level implementation details. However,
  these subtleties are described in detail in most books on scientific
  computation, and also in the following references:

  - The definitive guide to floating point arithmetic is the `IEEE 754-2008
    Standard <http://standards.ieee.org/findstds/standard/754-2008.html>`_;
    however, it is not available for free online.
  - For a brief but lucid presentation of how floating-point numbers are
    represented, see John D. Cook's `article
    <http://www.johndcook.com/blog/2009/04/06/anatomy-of-a-floating-point-number/>`_
    on the subject as well as his `introduction
    <http://www.johndcook.com/blog/2009/04/06/numbers-are-a-leaky-abstraction/>`_
    to some of the issues arising from how this representation differs in
    behavior from the idealized abstraction of real numbers.
  - Also recommended is Bruce Dawson's `series of blog posts on floating-point
    numbers <https://randomascii.wordpress.com/2012/05/20/thats-not-normalthe-performance-of-odd-floats/>`_.
  - For an excellent, in-depth discussion of floating-point numbers and issues of
    numerical accuracy encountered when computing with them, see David Goldberg's
    paper `What Every Computer Scientist Should Know About Floating-Point
    Arithmetic
    <http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.22.6768&rep=rep1&type=pdf>`_.
  - For even more extensive documentation of the history of, rationale for,
    and issues with floating-point numbers, as well as discussion of many other
    topics in numerical computing, see the `collected writings
    <https://people.eecs.berkeley.edu/~wkahan/>`_ of `William Kahan
    <https://en.wikipedia.org/wiki/William_Kahan>`_, commonly known as the "Father
    of Floating-Point". Of particular interest may be `An Interview with the Old
    Man of Floating-Point
    <https://people.eecs.berkeley.edu/~wkahan/ieee754status/754story.html>`_.

  .. _man-arbitrary-precision-arithmetic:

  Arbitrary Precision Arithmetic
  ------------------------------

  To allow computations with arbitrary-precision integers and floating point numbers,
  Julia wraps the `GNU Multiple Precision Arithmetic Library (GMP) <https://gmplib.org>`_ and the `GNU MPFR Library <http://www.mpfr.org>`_, respectively.
  The :class:`BigInt` and :class:`BigFloat` types are available in Julia for arbitrary precision
  integer and floating point numbers respectively.

  Constructors exist to create these types from primitive numerical types, and
  :func:`parse` can be use to construct them from :class:`AbstractString`\ s.  Once
  created, they participate in arithmetic with all other numeric types thanks to
  Julia's
  :ref:`type promotion and conversion mechanism <man-conversion-and-promotion>`:

  .. doctest::

      julia> BigInt(typemax(Int64)) + 1
      9223372036854775808

      julia> parse(BigInt, "123456789012345678901234567890") + 1
      123456789012345678901234567891

      julia> parse(BigFloat, "1.23456789012345678901")
      1.234567890123456789010000000000000000000000000000000000000000000000000000000004

      julia> BigFloat(2.0^66) / 3
      2.459565876494606882133333333333333333333333333333333333333333333333333333333344e+19

      julia> factorial(BigInt(40))
      815915283247897734345611269596115894272000000000

  However, type promotion between the primitive types above and
  :class:`BigInt`/:class:`BigFloat` is not automatic and must be explicitly stated.

  .. doctest::

      julia> x = typemin(Int64)
      -9223372036854775808

      julia> x = x - 1
      9223372036854775807

      julia> typeof(x)
      Int64

      julia> y = BigInt(typemin(Int64))
      -9223372036854775808

      julia> y = y - 1
      -9223372036854775809

      julia> typeof(y)
      BigInt

  The default precision (in number of bits of the significand) and
  rounding mode of :class:`BigFloat` operations can be changed globally
  by calling :func:`setprecision` and
  :func:`setrounding`, and all further calculations will take
  these changes in account.  Alternatively, the precision or the
  rounding can be changed only within the execution of a particular
  block of code by using the same functions with a ``do`` block:

  .. doctest::

      julia> setrounding(BigFloat, RoundUp) do
             BigFloat(1) + parse(BigFloat, "0.1")
             end
      1.100000000000000000000000000000000000000000000000000000000000000000000000000003

      julia> setrounding(BigFloat, RoundDown) do
             BigFloat(1) + parse(BigFloat, "0.1")
             end
      1.099999999999999999999999999999999999999999999999999999999999999999999999999986

      julia> setprecision(40) do
             BigFloat(1) + parse(BigFloat, "0.1")
             end
      1.1000000000004


  .. _man-numeric-literal-coefficients:

  Numeric Literal Coefficients
  ----------------------------

  To make common numeric formulas and expressions clearer, Julia allows
  variables to be immediately preceded by a numeric literal, implying
  multiplication. This makes writing polynomial expressions much cleaner:

  .. doctest::

      julia> x = 3
      3

      julia> 2x^2 - 3x + 1
      10

      julia> 1.5x^2 - .5x + 1
      13.0

  It also makes writing exponential functions more elegant:

  .. doctest::

      julia> 2^2x
      64

  The precedence of numeric literal coefficients is the same as that of unary
  operators such as negation. So ``2^3x`` is parsed as ``2^(3x)``, and
  ``2x^3`` is parsed as ``2*(x^3)``.

  Numeric literals also work as coefficients to parenthesized
  expressions:

  .. doctest::

      julia> 2(x-1)^2 - 3(x-1) + 1
      3

  Additionally, parenthesized expressions can be used as coefficients to
  variables, implying multiplication of the expression by the variable:

  .. doctest::

      julia> (x-1)x
      6

  Neither juxtaposition of two parenthesized expressions, nor placing a
  variable before a parenthesized expression, however, can be used to
  imply multiplication:

  .. doctest::

      julia> (x-1)(x+1)
      ERROR: MethodError: objects of type Int64 are not callable
      ...

      julia> x(x+1)
      ERROR: MethodError: objects of type Int64 are not callable
      ...

  Both expressions are interpreted as function application: any
  expression that is not a numeric literal, when immediately followed by a
  parenthetical, is interpreted as a function applied to the values in
  parentheses (see :ref:`man-functions` for more about functions).
  Thus, in both of these cases, an error occurs since the left-hand value
  is not a function.

  The above syntactic enhancements significantly reduce the visual noise
  incurred when writing common mathematical formulae. Note that no
  whitespace may come between a numeric literal coefficient and the
  identifier or parenthesized expression which it multiplies.

  Syntax Conflicts
  ~~~~~~~~~~~~~~~~

  Juxtaposed literal coefficient syntax may conflict with two numeric literal
  syntaxes: hexadecimal integer literals and engineering notation for
  floating-point literals. Here are some situations where syntactic
  conflicts arise:

  -  The hexadecimal integer literal expression ``0xff`` could be
     interpreted as the numeric literal ``0`` multiplied by the variable
     ``xff``.
  -  The floating-point literal expression ``1e10`` could be interpreted
     as the numeric literal ``1`` multiplied by the variable ``e10``, and
     similarly with the equivalent ``E`` form.

  In both cases, we resolve the ambiguity in favor of interpretation as a
  numeric literals:

  -  Expressions starting with ``0x`` are always hexadecimal literals.
  -  Expressions starting with a numeric literal followed by ``e`` or
     ``E`` are always floating-point literals.

  Literal zero and one
  --------------------

  Julia provides functions which return literal 0 and 1 corresponding to a
  specified type or the type of a given variable.

  ====================== =====================================================
  Function               Description
  ====================== =====================================================
  :func:`zero(x) <zero>` Literal zero of type ``x`` or type of variable ``x``
  :func:`one(x) <one>`   Literal one of type ``x`` or type of variable ``x``
  ====================== =====================================================

  These functions are useful in :ref:`man-numeric-comparisons` to avoid overhead
  from unnecessary :ref:`type conversion <man-conversion-and-promotion>`.

  Examples:

  .. doctest::

      julia> zero(Float32)
      0.0f0

      julia> zero(1.0)
      0.0

      julia> one(Int32)
      1

      julia> one(BigFloat)
      1.000000000000000000000000000000000000000000000000000000000000000000000000000000
    

.. _man-integers-and-floating-point-numbers:

.. currentmodule:: Base

.. 
  *************************************
   Integers and Floating-Point Numbers
  *************************************

*************************************
 整数と浮動小数点数
*************************************

.. 
  Integers and floating-point values are the basic building blocks of
  arithmetic and computation. Built-in representations of such values are
  called numeric primitives, while representations of integers and
  floating-point numbers as immediate values in code are known as numeric
  literals. For example, ``1`` is an integer literal, while ``1.0`` is a
  floating-point literal; their binary in-memory representations as
  objects are numeric primitives.
  
整数と浮動小数点値は、演算処理の基本的要素です。コード内の即値としての整数と浮動小数点値の表現は
数値リテラルとして知られている一方で、ビルトインのこれらの値の表現は数値プリミティブと呼ばれています。
例えば、 ``1`` は整数リテラルですが、``1.0`` は浮動小数点リテラルです。これらのオブジェクトとしての
インメモリ表現が数値プリミティブです。

.. 
  Julia provides a broad range of primitive numeric types, and a full complement
  of arithmetic and bitwise operators as well as standard mathematical functions
  are defined over them. These map directly onto numeric types and operations
  that are natively supported on modern computers, thus allowing Julia to take
  full advantage of computational resources. Additionally, Julia provides
  software support for :ref:`man-arbitrary-precision-arithmetic`, which can
  handle operations on numeric values that cannot be represented effectively in
  native hardware representations, but at the cost of relatively slower
  performance.

Juliaは、幅広いプリミティブ数値型を提供し、標準的な数学関数はもちろん算術演算やビット演算子の
補完が定義されています。これらは、現代のコンピュータでサポートされている数値型や演算処理と関連づけます。
これにより、Juliaは演算処理のリソースを最大限活用することができます。さらに、Juliaは、
使用しているハードウェアでは効果的に表現できない数値の演算処理を行うことができる :ref:`man-arbitrary-precision-arithmetic` の
ソフトウェアサポートを提供します。

.. 
  The following are Julia's primitive numeric types:

以下はJuliaにおける数値プリミティブ型です。

.. 
  -  **Integer types:**

-  **整数型:**

.. 
  ================  =======  ==============  ============== ==================
  Type              Signed?  Number of bits  Smallest value Largest value
  ================  =======  ==============  ============== ==================
  :class:`Int8`        ✓         8            -2^7             2^7 - 1
  :class:`UInt8`                 8             0               2^8 - 1
  :class:`Int16`       ✓         16           -2^15            2^15 - 1
  :class:`UInt16`                16            0               2^16 - 1
  :class:`Int32`       ✓         32           -2^31            2^31 - 1
  :class:`UInt32`                32            0               2^32 - 1
  :class:`Int64`       ✓         64           -2^63            2^63 - 1
  :class:`UInt64`                64            0               2^64 - 1
  :class:`Int128`      ✓         128           -2^127          2^127 - 1
  :class:`UInt128`               128           0               2^128 - 1
  :class:`Bool`       N/A        8           ``false`` (0)  ``true`` (1)
  ================  =======  ==============  ============== ==================

================  ==========  ==============  ============== ==================
型                符号の有無   バイト数          最小値          最大値
================  ==========  ==============  ============== ==================
:class:`Int8`        ✓            8            -2^7              2^7 - 1
:class:`UInt8`                    8             0                2^8 - 1
:class:`Int16`       ✓            16           -2^15            2^15 - 1
:class:`UInt16`                   16            0               2^16 - 1
:class:`Int32`       ✓            32           -2^31            2^31 - 1
:class:`UInt32`                   32            0               2^32 - 1
:class:`Int64`       ✓            64           -2^63            2^63 - 1
:class:`UInt64`                   64            0               2^64 - 1
:class:`Int128`      ✓            128           -2^127          2^127 - 1
:class:`UInt128`                  128           0               2^128 - 1
:class:`Bool`       N/A           8           ``false`` (0)  ``true`` (1)
================  ==========  ==============  ============== ==================

.. 
  -  **Floating-point types:**

-  **浮動小数点型:**

================ ========= ==============
型               精度       バイト数
================ ========= ==============
:class:`Float16` half_          16
:class:`Float32` single_        32
:class:`Float64` double_        64
================ ========= ==============

.. _half: https://en.wikipedia.org/wiki/Half-precision_floating-point_format
.. _single: https://en.wikipedia.org/wiki/Single_precision_floating-point_format
.. _double: https://en.wikipedia.org/wiki/Double_precision_floating-point_format

.. 
  Additionally, full support for :ref:`man-complex-and-rational-numbers` is built
  on top of these primitive numeric types. All numeric types interoperate
  naturally without explicit casting, thanks to a flexible, user-extensible
  :ref:`type promotion system <man-conversion-and-promotion>`.

加えて、:ref:`man-complex-and-rational-numbers` へのサポートは、これらの数値プリミティブ型を踏まえて構築されています。
全ての数値型は、明示的に割り当てることなく自然に相互運用します。 :ref:`type promotion system <man-conversion-and-promotion>`

.. 
  Integers
  --------

整数
--------

.. 
  Literal integers are represented in the standard manner:

  .. doctest::

      julia> 1
      1

      julia> 1234
      1234

整数リテラルは標準的な方法で表現されます。:
    
.. doctest::

    julia> 1
    1

    julia> 1234
    1234

.. 
  The default type for an integer literal depends on whether the target
  system has a 32-bit architecture or a 64-bit architecture::

      # 32-bit system:
      julia> typeof(1)
      Int32

      # 64-bit system:
      julia> typeof(1)
      Int64

整数リテラルのデフォルトの型は、ターゲットシステムのアーキテクチャが32ビットであるか
64ビットであるかによって異なります。::

    # 32-bit system:
    julia> typeof(1)
    Int32

    # 64-bit system:
    julia> typeof(1)
    Int64

.. 
  The Julia internal variable :const:`Sys.WORD_SIZE` indicates whether the target system
  is 32-bit or 64-bit.::

      # 32-bit system:
      julia> Sys.WORD_SIZE
      32

      # 64-bit system:
      julia> Sys.WORD_SIZE
      64

Juliaの内部変数 :const:`Sys.WORD_SIZE` はターゲットシステムが32ビットまたは64ビットのどちらかであることを示します。::

    # 32-bit system:
    julia> Sys.WORD_SIZE
    32

    # 64-bit system:
    julia> Sys.WORD_SIZE
    64

.. 
  Julia also defines the types :class:`Int` and :class:`UInt`, which are aliases for the
  system's signed and unsigned native integer types respectively.::

      # 32-bit system:
      julia> Int
      Int32
      julia> UInt
      UInt32


      # 64-bit system:
      julia> Int
      Int64
      julia> UInt
      UInt64

Juliaは、システムの符号つきまたは符号なしの固有の整数型のエイリアスである :class:`Int` 
および :class:`UInt` 型を定義します。::

    # 32-bit system:
    julia> Int
    Int32
    julia> UInt
    UInt32


    # 64-bit system:
    julia> Int
    Int64
    julia> UInt
    UInt64

.. 
  Larger integer literals that cannot be represented using only 32 bits
  but can be represented in 64 bits always create 64-bit integers,
  regardless of the system type::

      # 32-bit or 64-bit system:
      julia> typeof(3000000000)
      Int64

32ビットを使用して表現できないが、64ビットでは表現できるような大きな整数リテラルの場合、
システムタイプに関係なく常に64ビット整数を作成します。::

    # 32-bit or 64-bit system:
    julia> typeof(3000000000)
    Int64

.. 
  Unsigned integers are input and output using the ``0x`` prefix and hexadecimal
  (base 16) digits ``0-9a-f`` (the capitalized digits ``A-F`` also work for input).
  The size of the unsigned value is determined by the number of hex digits used:

  .. doctest::

      julia> 0x1
      0x01

      julia> typeof(ans)
      UInt8

      julia> 0x123
      0x0123

      julia> typeof(ans)
      UInt16

      julia> 0x1234567
      0x01234567

      julia> typeof(ans)
      UInt32

      julia> 0x123456789abcdef
      0x0123456789abcdef

      julia> typeof(ans)
      UInt64

符号なし整数は、 ``0x`` プレフィックスおよび16進数の ``0-9a-f`` （大文字の ``A-F`` は入力時のみ使用可能）を
使用して入力および出力されます。符号なしの値の大きさは、使用されている16進数の桁数により決定されます。:

.. doctest::

    julia> 0x1
    0x01

    julia> typeof(ans)
    UInt8

    julia> 0x123
    0x0123

    julia> typeof(ans)
    UInt16

    julia> 0x1234567
    0x01234567

    julia> typeof(ans)
    UInt32

    julia> 0x123456789abcdef
    0x0123456789abcdef

    julia> typeof(ans)
    UInt64

.. 
  This behavior is based on the observation that when one uses unsigned
  hex literals for integer values, one typically is using them to
  represent a fixed numeric byte sequence, rather than just an integer
  value.

この動作は、整数値に符号なし16進リテラルを使用する際、ただの整数ではなく、
通常固定された数値のバイト列を表すという傾向に基づいています。

.. 
  Recall that the variable :data:`ans` is set to the value of the last expression
  evaluated in an interactive session. This does not occur when Julia code is
  run in other ways.

前述の通り、変数 :data:`ans` は対話セッションで最後に出力された式の値がセットされます。
この処理は、Juliaコードがその他の方法で実行された場合には実施されません。

.. 
Binary and octal literals are also supported:

  .. doctest::

      julia> 0b10
      0x02

      julia> typeof(ans)
      UInt8

      julia> 0o10
      0x08

      julia> typeof(ans)
      UInt8

2進および8進リテラルについてもサポートされています。:

.. doctest::

    julia> 0b10
    0x02

    julia> typeof(ans)
    UInt8

    julia> 0o10
    0x08

    julia> typeof(ans)
    UInt8    

.. 
  The minimum and maximum representable values of primitive numeric types
  such as integers are given by the :func:`typemin` and :func:`typemax` functions:

  .. doctest::

      julia> (typemin(Int32), typemax(Int32))
      (-2147483648,2147483647)

      julia> for T in [Int8,Int16,Int32,Int64,Int128,UInt8,UInt16,UInt32,UInt64,UInt128]
               println("$(lpad(T,7)): [$(typemin(T)),$(typemax(T))]")
             end
         Int8: [-128,127]
        Int16: [-32768,32767]
        Int32: [-2147483648,2147483647]
        Int64: [-9223372036854775808,9223372036854775807]
       Int128: [-170141183460469231731687303715884105728,170141183460469231731687303715884105727]
        UInt8: [0,255]
       UInt16: [0,65535]
       UInt32: [0,4294967295]
       UInt64: [0,18446744073709551615]
      UInt128: [0,340282366920938463463374607431768211455]

整数のような数値プリミティブ型の最小値と最大値は、 :func:`typemin` および :func:`typemax` 関数により取得が可能です。:

.. doctest::

    julia> (typemin(Int32), typemax(Int32))
    (-2147483648,2147483647)

    julia> for T in [Int8,Int16,Int32,Int64,Int128,UInt8,UInt16,UInt32,UInt64,UInt128]
             println("$(lpad(T,7)): [$(typemin(T)),$(typemax(T))]")
           end
       Int8: [-128,127]
      Int16: [-32768,32767]
      Int32: [-2147483648,2147483647]
      Int64: [-9223372036854775808,9223372036854775807]
     Int128: [-170141183460469231731687303715884105728,170141183460469231731687303715884105727]
      UInt8: [0,255]
     UInt16: [0,65535]
     UInt32: [0,4294967295]
     UInt64: [0,18446744073709551615]
    UInt128: [0,340282366920938463463374607431768211455]    

.. 
  The values returned by :func:`typemin` and :func:`typemax` are always of the
  given argument type. (The above expression uses several features we have
  yet to introduce, including :ref:`for loops <man-loops>`,
  :ref:`man-strings`, and :ref:`man-string-interpolation`,
  but should be easy enough to understand for users with some existing
  programming experience.)
  
:func:`typemin` および :func:`typemax` の戻り値は常に指定された引数の型となります。
（上記の式は、まだ紹介されていない :ref:`for loops <man-loops>`、:ref:`man-strings`、:ref:`man-string-interpolation`
などの機能が含まれていますが、プログラミング経験者にとっては難しいものではないはずです。）

.. 
  Overflow behavior
  ~~~~~~~~~~~~~~~~~

オーバーフロー処理
~~~~~~~~~~~~~~~~~

.. 
  In Julia, exceeding the maximum representable value of a given type results in
  a wraparound behavior:

  .. doctest::

      julia> x = typemax(Int64)
      9223372036854775807

      julia> x + 1
      -9223372036854775808

      julia> x + 1 == typemin(Int64)
      true

Juliaでは、指定された型の表現可能な最大値を超えた場合、ワードラップが発生します。:

.. doctest::

    julia> x = typemax(Int64)
    9223372036854775807

    julia> x + 1
    -9223372036854775808

    julia> x + 1 == typemin(Int64)
    true

.. 
  Thus, arithmetic with Julia integers is actually a form of `modular arithmetic
  <https://en.wikipedia.org/wiki/Modular_arithmetic>`_. This reflects the
  characteristics of the underlying arithmetic of integers as implemented on
  modern computers. In applications where overflow is possible, explicit checking
  for wraparound produced by overflow is essential; otherwise, the ``BigInt`` type
  in :ref:`man-arbitrary-precision-arithmetic` is recommended instead.

このように、Juliaにおける整数の演算は、`合同算術<https://en.wikipedia.org/wiki/Modular_arithmetic>`_ の
形をとります。これは現代のコンピュータで実行される基本演算の特性を反映しています。
オーバーフローを許容するアプリケーションでは、オーバーフローにより発生したワードラップの明示的なチェックは不可欠です。
チェックが難しい場合は、 :ref:`man-arbitrary-precision-arithmetic` の ``BigInt`` 型を使用することをお勧めします。

.. 
  Division errors
  ~~~~~~~~~~~~~~~

除算処理エラー
~~~~~~~~~~~~~~~

.. 
  Integer division (the ``div`` function) has two exceptional cases: dividing by
  zero, and dividing the lowest negative number (:func:`typemin`) by -1. Both of
  these cases throw a :exc:`DivideError`. The remainder and modulus functions
  (``rem`` and ``mod``) throw a :exc:`DivideError` when their second argument is
  zero.

整数の除算（``div`` 関数）には、0で割る、最小の負の数（ :func:`typemin` ）を−1で割るの2つの
例外的ケースがあります。どちらの結果も :exc:`DivideError` となります。余りおよび絶対値の
関数（``rem`` および ``mod`` ）についても、その第二引数が0の際に :exc:`DivideError` となります。

.. 
  Floating-Point Numbers
  ----------------------

浮動小数点数
----------------------

.. 
  Literal floating-point numbers are represented in the standard formats:

  .. doctest::

      julia> 1.0
      1.0

      julia> 1.
      1.0

      julia> 0.5
      0.5

      julia> .5
      0.5

      julia> -1.23
      -1.23

      julia> 1e10
      1.0e10

      julia> 2.5e-4
      0.00025

浮動小数点数リテラルは標準的な形式で表されます。:

.. doctest::

    julia> 1.0
    1.0

    julia> 1.
    1.0

    julia> 0.5
    0.5

    julia> .5
    0.5

    julia> -1.23
    -1.23

    julia> 1e10
    1.0e10

    julia> 2.5e-4
    0.00025    

.. 
  The above results are all ``Float64`` values. Literal ``Float32`` values can
  be entered by writing an ``f`` in place of ``e``:

  .. doctest::

      julia> 0.5f0
      0.5f0

      julia> typeof(ans)
      Float32

      julia> 2.5f-4
      0.00025f0

上記の結果は、全て ``Float64`` 値です。リテラル ``Float32`` 値は、 ``f`` の代わりに ``e`` を使用することで入力が可能です。:

.. doctest::

    julia> 0.5f0
    0.5f0

    julia> typeof(ans)
    Float32

    julia> 2.5f-4
    0.00025f0

.. 
  Values can be converted to ``Float32`` easily:

  .. doctest::

      julia> Float32(-1.5)
      -1.5f0

      julia> typeof(ans)
      Float32

値は簡単に ``Float32`` に変換することが可能です。:

.. doctest::

    julia> Float32(-1.5)
    -1.5f0

    julia> typeof(ans)
    Float32

.. 
  Hexadecimal floating-point literals are also valid, but only as ``Float64`` values:

  .. doctest::

      julia> 0x1p0
      1.0

      julia> 0x1.8p3
      12.0

      julia> 0x.4p-1
      0.125

      julia> typeof(ans)
      Float64

16進数の浮動小数点リテラルも有効ですが、 ``Float64`` 値としてのみ使用が可能です。:

.. doctest::

    julia> 0x1p0
    1.0

    julia> 0x1.8p3
    12.0

    julia> 0x.4p-1
    0.125

    julia> typeof(ans)
    Float64    

.. 
  Half-precision floating-point numbers are also supported (``Float16``), but
  only as a storage format. In calculations they'll be converted to ``Float32``:

  .. doctest::

      julia> sizeof(Float16(4.))
      2

      julia> 2*Float16(4.)
      Float16(8.0)

半制度浮動小数点数もサポートされていますが（ ``Float16`` ）、保存形式のみとして使用が可能です。
計算時には、それらは ``Float32`` に変換されます。:

.. doctest::

    julia> sizeof(Float16(4.))
    2

    julia> 2*Float16(4.)
    Float16(8.0)    

.. 
  The underscore ``_`` can be used as digit separator:

  .. doctest::

      julia> 10_000, 0.000_000_005, 0xdead_beef, 0b1011_0010
      (10000,5.0e-9,0xdeadbeef,0xb2)

アンダースコア（ ``_`` ）は桁区切り文字として使用することができます。

.. doctest::

    julia> 10_000, 0.000_000_005, 0xdead_beef, 0b1011_0010
    (10000,5.0e-9,0xdeadbeef,0xb2)    

.. 
  Floating-point zero
  ~~~~~~~~~~~~~~~~~~~

浮動小数点における0
~~~~~~~~~~~~~~~~~~~

.. 
  Floating-point numbers have `two zeros
  <https://en.wikipedia.org/wiki/Signed_zero>`_, positive zero and negative zero.
  They are equal to each other but have different binary representations, as can
  be seen using the ``bits`` function: :

  .. doctest::

      julia> 0.0 == -0.0
      true

      julia> bits(0.0)
      "0000000000000000000000000000000000000000000000000000000000000000"

      julia> bits(-0.0)
      "1000000000000000000000000000000000000000000000000000000000000000"

  .. _man-special-floats:

浮動小数点数には`2つの０<https://en.wikipedia.org/wiki/Signed_zero>`_　（正の0と負の0）があります。
この2つの0は同じ値ですが、 ``bits`` 関数を使用した際に見られるように、それぞれ異なるバイナリ表現を持ちます。::

.. doctest::

    julia> 0.0 == -0.0
    true

    julia> bits(0.0)
    "0000000000000000000000000000000000000000000000000000000000000000"

    julia> bits(-0.0)
    "1000000000000000000000000000000000000000000000000000000000000000"

.. _man-special-floats:

.. 
  Special floating-point values
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

特殊な浮動小数点値
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. 
  There are three specified standard floating-point values that do not
  correspond to any point on the real number line:
  
  =========== =========== ===========  ================= =================================================================
  Special value                        Name              Description
  -----------------------------------  ----------------- -----------------------------------------------------------------
  ``Float16`` ``Float32`` ``Float64``
  =========== =========== ===========  ================= =================================================================
  ``Inf16``   ``Inf32``    ``Inf``     positive infinity a value greater than all finite floating-point values
  ``-Inf16``  ``-Inf32``   ``-Inf``    negative infinity a value less than all finite floating-point values
  ``NaN16``   ``NaN32``    ``NaN``     not a number      a value not ``==`` to any floating-point value (including itself)
  =========== =========== ===========  ================= =================================================================

実数直線上に対応しない3つの浮動小数点値が存在します。

=========== =========== ===========  ================= =================================================================
特殊な値                              名称               概要
-----------------------------------  ----------------- -----------------------------------------------------------------
``Float16`` ``Float32`` ``Float64``
=========== =========== ===========  ================= =================================================================
``Inf16``   ``Inf32``    ``Inf``     正の無限大         全ての有限の浮動小数点値よりも大きい値
``-Inf16``  ``-Inf32``   ``-Inf``    負の無限大         全ての有限の浮動小数点数値よりも小さい値
``NaN16``   ``NaN32``    ``NaN``     非数値            浮動小数点値以外の値
=========== =========== ===========  ================= =================================================================

.. 
  For further discussion of how these non-finite floating-point values are
  ordered with respect to each other and other floats, see
  :ref:`man-numeric-comparisons`. By the
  `IEEE 754 standard <https://en.wikipedia.org/wiki/IEEE_754-2008>`_, these
  floating-point values are the results of certain arithmetic operations:

  .. doctest::

      julia> 1/Inf
      0.0

      julia> 1/0
      Inf

      julia> -5/0
      -Inf

      julia> 0.000001/0
      Inf

      julia> 0/0
      NaN

      julia> 500 + Inf
      Inf

      julia> 500 - Inf
      -Inf

      julia> Inf + Inf
      Inf

      julia> Inf - Inf
      NaN

      julia> Inf * Inf
      Inf

      julia> Inf / Inf
      NaN

      julia> 0 * Inf
      NaN

どのように非有限浮動小数点値がお互いに、およびその他の浮動値に対して順序付けられているかについては、
:ref:`man-numeric-comparisons` を参照してください。`IEEE 754規格 <https://en.wikipedia.org/wiki/IEEE_754-2008>`_
では、これらの浮動小数点値は特定の演算処理の結果として取得されます。:

.. doctest::

    julia> 1/Inf
    0.0

    julia> 1/0
    Inf

    julia> -5/0
    -Inf

    julia> 0.000001/0
    Inf

    julia> 0/0
    NaN

    julia> 500 + Inf
    Inf

    julia> 500 - Inf
    -Inf

    julia> Inf + Inf
    Inf

    julia> Inf - Inf
    NaN

    julia> Inf * Inf
    Inf

    julia> Inf / Inf
    NaN

    julia> 0 * Inf
    NaN    

.. 
  The :func:`typemin` and :func:`typemax` functions also apply to floating-point
  types:

  .. doctest::

      julia> (typemin(Float16),typemax(Float16))
      (-Inf16,Inf16)

      julia> (typemin(Float32),typemax(Float32))
      (-Inf32,Inf32)

      julia> (typemin(Float64),typemax(Float64))
      (-Inf,Inf)
      
:func:`typemin` および :func:`typemax` 関数は浮動小数点型に対しても使用が可能です。

.. doctest::

    julia> (typemin(Float16),typemax(Float16))
    (-Inf16,Inf16)

    julia> (typemin(Float32),typemax(Float32))
    (-Inf32,Inf32)

    julia> (typemin(Float64),typemax(Float64))
    (-Inf,Inf)

.. 
  Machine epsilon
  ~~~~~~~~~~~~~~~

計算機イプシロン
~~~~~~~~~~~~~~~

.. 
  Most real numbers cannot be represented exactly with floating-point numbers,
  and so for many purposes it is important to know the distance between two
  adjacent representable floating-point numbers, which is often known as
  `machine epsilon <https://en.wikipedia.org/wiki/Machine_epsilon>`_.
  
ほとんどの実数は、浮動小数点数で正確に表現することができないため、`計算機イプシロン <https://en.wikipedia.org/wiki/Machine_epsilon>`_
として知られる隣接する２つの浮動小数点数の距離を理解することは、多くの用途のために重要となります。

.. 
  Julia provides :func:`eps`, which gives the distance between ``1.0``
  and the next larger representable floating-point value:

  .. doctest::

      julia> eps(Float32)
      1.1920929f-7

      julia> eps(Float64)
      2.220446049250313e-16

      julia> eps() # same as eps(Float64)
      2.220446049250313e-16

Juliaは、 ``1.0`` と次に大きい浮動小数点値の間の距離を取得する :func:`eps` 関数を提供しています。:

.. doctest::

    julia> eps(Float32)
    1.1920929f-7

    julia> eps(Float64)
    2.220446049250313e-16

    julia> eps() # same as eps(Float64)
    2.220446049250313e-16    

.. 
  These values are ``2.0^-23`` and ``2.0^-52`` as ``Float32`` and ``Float64``
  values, respectively. The :func:`eps` function can also take a
  floating-point value as an argument, and gives the absolute difference
  between that value and the next representable floating point value. That
  is, ``eps(x)`` yields a value of the same type as ``x`` such that
  ``x + eps(x)`` is the next representable floating-point value larger
  than ``x``:

  .. doctest::

      julia> eps(1.0)
      2.220446049250313e-16

      julia> eps(1000.)
      1.1368683772161603e-13

      julia> eps(1e-27)
      1.793662034335766e-43

      julia> eps(0.0)
      5.0e-324

これらはそれぞれ ``Float32`` および ``Float64`` 値として ``2.0^-23`` および ``2.0^-52`` と
なります。 :func:`eps` 関数は、浮動小数点値を引数として使用したり、ある値と次の浮動小数点値との絶対差を
得ることができます。つまり、 ``eps(x)`` は ``x`` と同様の型の値を生成し、``x + eps(x)`` は ``x`` よりも
大きい次の浮動小数点値を生成します。:

.. doctest::

    julia> eps(1.0)
    2.220446049250313e-16

    julia> eps(1000.)
    1.1368683772161603e-13

    julia> eps(1e-27)
    1.793662034335766e-43

    julia> eps(0.0)
    5.0e-324    

.. 
  The distance between two adjacent representable floating-point numbers is not
  constant, but is smaller for smaller values and larger for larger values. In
  other words, the representable floating-point numbers are densest in the real
  number line near zero, and grow sparser exponentially as one moves farther away
  from zero. By definition, ``eps(1.0)`` is the same as ``eps(Float64)`` since
  ``1.0`` is a 64-bit floating-point value.

隣接する2つの浮動小数点数の距離は一定ではありませんが、小さい値ではその距離はより小さく、
大きい値ではより大きくなります。言い換えれば、浮動小数点数は実数線上で0に近い場合に最も高密度になり、
0から離れるにつれてまばらになっていきます。定義上、 ``1.0`` は64ビットの浮動小数点値であるため、
 ``eps(1.0)`` は``eps(Float64)`` と同一です。

.. 
  Julia also provides the :func:`nextfloat` and :func:`prevfloat` functions which return
  the next largest or smallest representable floating-point number to the
  argument respectively: :

  .. doctest::

      julia> x = 1.25f0
      1.25f0

      julia> nextfloat(x)
      1.2500001f0

      julia> prevfloat(x)
      1.2499999f0

      julia> bits(prevfloat(x))
      "00111111100111111111111111111111"

      julia> bits(x)
      "00111111101000000000000000000000"

      julia> bits(nextfloat(x))
      "00111111101000000000000000000001"

Juliaは、次の最大または最小の浮動小数点数を引数に戻り値として与える :func:`nextfloat` 
および :func:`prevfloat` 関数を提供しています。:

.. doctest::

    julia> x = 1.25f0
    1.25f0

    julia> nextfloat(x)
    1.2500001f0

    julia> prevfloat(x)
    1.2499999f0

    julia> bits(prevfloat(x))
    "00111111100111111111111111111111"

    julia> bits(x)
    "00111111101000000000000000000000"

    julia> bits(nextfloat(x))
    "00111111101000000000000000000001"

.. 
  This example highlights the general principle that the adjacent representable
  floating-point numbers also have adjacent binary integer representations.

この例では、隣接する浮動小数点数は隣接する二進整数を持つという一般的な原則を表しています。

.. 
  Rounding modes
  ~~~~~~~~~~~~~~

端数処理モード
~~~~~~~~~~~~~~

.. 
  If a number doesn't have an exact floating-point representation, it must be
  rounded to an appropriate representable value, however, if wanted, the manner
  in which this rounding is done can be changed according to the rounding modes
  presented in the `IEEE 754 standard <https://en.wikipedia.org/wiki/IEEE_754-2008>`_.

  .. doctest::

      julia> x = 1.1; y = 0.1;

      julia> x + y
      1.2000000000000002

      julia> setrounding(Float64,RoundDown) do
                 x + y
             end
      1.2

値が割り切れる浮動小数点数ではない場合、適切な表現可能な値に繰り上げる必要があります。
しかし、必要な場合は、どのように繰り上げするかは `IEEE 754規格 <https://en.wikipedia.org/wiki/IEEE_754-2008>`_ の
端数処理モードに準拠して変更が可能です。

.. doctest::

    julia> x = 1.1; y = 0.1;

    julia> x + y
    1.2000000000000002

    julia> setrounding(Float64,RoundDown) do
               x + y
           end
    1.2   

.. 
  The default mode used is always :const:`RoundNearest`, which rounds to the nearest
  representable value, with ties rounded towards the nearest value with an even
  least significant bit.

デフォルトでは、偶数の最下位ビットに最も近い値になるよう値を
繰り上げる :const:`RoundNearest` が常に使用されます。

.. 
  .. warning:: Rounding is generally only correct for basic arithmetic functions
         (:func:`+`, :func:`-`, :func:`*`, :func:`/` and :func:`sqrt`) and
         type conversion operations. Many other functions assume the
         default :const:`RoundNearest` mode is set, and can give erroneous
         results when operating under other rounding modes.
         
.. warning::端数処理は通常基本的な算術関数（:func:`+`、 :func:`-`、 :func:`*`、 :func:`/`、 :func:`sqrt`）および
        型変換処理においてのみ正常に機能します。その他の多くの関数は、デフォルトの :const:`RoundNearest` モードが
        設定されている前提で機能し、その他の端数処理モードでの演算処理では誤った結果を出力する場合があります。       

Background and References
~~~~~~~~~~~~~~~~~~~~~~~~~

Floating-point arithmetic entails many subtleties which can be surprising to
users who are unfamiliar with the low-level implementation details. However,
these subtleties are described in detail in most books on scientific
computation, and also in the following references:

- The definitive guide to floating point arithmetic is the `IEEE 754-2008
  Standard <http://standards.ieee.org/findstds/standard/754-2008.html>`_;
  however, it is not available for free online.
- For a brief but lucid presentation of how floating-point numbers are
  represented, see John D. Cook's `article
  <http://www.johndcook.com/blog/2009/04/06/anatomy-of-a-floating-point-number/>`_
  on the subject as well as his `introduction
  <http://www.johndcook.com/blog/2009/04/06/numbers-are-a-leaky-abstraction/>`_
  to some of the issues arising from how this representation differs in
  behavior from the idealized abstraction of real numbers.
- Also recommended is Bruce Dawson's `series of blog posts on floating-point
  numbers <https://randomascii.wordpress.com/2012/05/20/thats-not-normalthe-performance-of-odd-floats/>`_.
- For an excellent, in-depth discussion of floating-point numbers and issues of
  numerical accuracy encountered when computing with them, see David Goldberg's
  paper `What Every Computer Scientist Should Know About Floating-Point
  Arithmetic
  <http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.22.6768&rep=rep1&type=pdf>`_.
- For even more extensive documentation of the history of, rationale for,
  and issues with floating-point numbers, as well as discussion of many other
  topics in numerical computing, see the `collected writings
  <https://people.eecs.berkeley.edu/~wkahan/>`_ of `William Kahan
  <https://en.wikipedia.org/wiki/William_Kahan>`_, commonly known as the "Father
  of Floating-Point". Of particular interest may be `An Interview with the Old
  Man of Floating-Point
  <https://people.eecs.berkeley.edu/~wkahan/ieee754status/754story.html>`_.

.. _man-arbitrary-precision-arithmetic:

Arbitrary Precision Arithmetic
------------------------------

To allow computations with arbitrary-precision integers and floating point numbers,
Julia wraps the `GNU Multiple Precision Arithmetic Library (GMP) <https://gmplib.org>`_ and the `GNU MPFR Library <http://www.mpfr.org>`_, respectively.
The :class:`BigInt` and :class:`BigFloat` types are available in Julia for arbitrary precision
integer and floating point numbers respectively.

Constructors exist to create these types from primitive numerical types, and
:func:`parse` can be use to construct them from :class:`AbstractString`\ s.  Once
created, they participate in arithmetic with all other numeric types thanks to
Julia's
:ref:`type promotion and conversion mechanism <man-conversion-and-promotion>`:

.. doctest::

    julia> BigInt(typemax(Int64)) + 1
    9223372036854775808

    julia> parse(BigInt, "123456789012345678901234567890") + 1
    123456789012345678901234567891

    julia> parse(BigFloat, "1.23456789012345678901")
    1.234567890123456789010000000000000000000000000000000000000000000000000000000004

    julia> BigFloat(2.0^66) / 3
    2.459565876494606882133333333333333333333333333333333333333333333333333333333344e+19

    julia> factorial(BigInt(40))
    815915283247897734345611269596115894272000000000

However, type promotion between the primitive types above and
:class:`BigInt`/:class:`BigFloat` is not automatic and must be explicitly stated.

.. doctest::

    julia> x = typemin(Int64)
    -9223372036854775808

    julia> x = x - 1
    9223372036854775807

    julia> typeof(x)
    Int64

    julia> y = BigInt(typemin(Int64))
    -9223372036854775808

    julia> y = y - 1
    -9223372036854775809

    julia> typeof(y)
    BigInt

The default precision (in number of bits of the significand) and
rounding mode of :class:`BigFloat` operations can be changed globally
by calling :func:`setprecision` and
:func:`setrounding`, and all further calculations will take
these changes in account.  Alternatively, the precision or the
rounding can be changed only within the execution of a particular
block of code by using the same functions with a ``do`` block:

.. doctest::

    julia> setrounding(BigFloat, RoundUp) do
           BigFloat(1) + parse(BigFloat, "0.1")
           end
    1.100000000000000000000000000000000000000000000000000000000000000000000000000003

    julia> setrounding(BigFloat, RoundDown) do
           BigFloat(1) + parse(BigFloat, "0.1")
           end
    1.099999999999999999999999999999999999999999999999999999999999999999999999999986

    julia> setprecision(40) do
           BigFloat(1) + parse(BigFloat, "0.1")
           end
    1.1000000000004


.. _man-numeric-literal-coefficients:

Numeric Literal Coefficients
----------------------------

To make common numeric formulas and expressions clearer, Julia allows
variables to be immediately preceded by a numeric literal, implying
multiplication. This makes writing polynomial expressions much cleaner:

.. doctest::

    julia> x = 3
    3

    julia> 2x^2 - 3x + 1
    10

    julia> 1.5x^2 - .5x + 1
    13.0

It also makes writing exponential functions more elegant:

.. doctest::

    julia> 2^2x
    64

The precedence of numeric literal coefficients is the same as that of unary
operators such as negation. So ``2^3x`` is parsed as ``2^(3x)``, and
``2x^3`` is parsed as ``2*(x^3)``.

Numeric literals also work as coefficients to parenthesized
expressions:

.. doctest::

    julia> 2(x-1)^2 - 3(x-1) + 1
    3

Additionally, parenthesized expressions can be used as coefficients to
variables, implying multiplication of the expression by the variable:

.. doctest::

    julia> (x-1)x
    6

Neither juxtaposition of two parenthesized expressions, nor placing a
variable before a parenthesized expression, however, can be used to
imply multiplication:

.. doctest::

    julia> (x-1)(x+1)
    ERROR: MethodError: objects of type Int64 are not callable
    ...

    julia> x(x+1)
    ERROR: MethodError: objects of type Int64 are not callable
    ...

Both expressions are interpreted as function application: any
expression that is not a numeric literal, when immediately followed by a
parenthetical, is interpreted as a function applied to the values in
parentheses (see :ref:`man-functions` for more about functions).
Thus, in both of these cases, an error occurs since the left-hand value
is not a function.

The above syntactic enhancements significantly reduce the visual noise
incurred when writing common mathematical formulae. Note that no
whitespace may come between a numeric literal coefficient and the
identifier or parenthesized expression which it multiplies.

Syntax Conflicts
~~~~~~~~~~~~~~~~

Juxtaposed literal coefficient syntax may conflict with two numeric literal
syntaxes: hexadecimal integer literals and engineering notation for
floating-point literals. Here are some situations where syntactic
conflicts arise:

-  The hexadecimal integer literal expression ``0xff`` could be
   interpreted as the numeric literal ``0`` multiplied by the variable
   ``xff``.
-  The floating-point literal expression ``1e10`` could be interpreted
   as the numeric literal ``1`` multiplied by the variable ``e10``, and
   similarly with the equivalent ``E`` form.

In both cases, we resolve the ambiguity in favor of interpretation as a
numeric literals:

-  Expressions starting with ``0x`` are always hexadecimal literals.
-  Expressions starting with a numeric literal followed by ``e`` or
   ``E`` are always floating-point literals.

Literal zero and one
--------------------

Julia provides functions which return literal 0 and 1 corresponding to a
specified type or the type of a given variable.

====================== =====================================================
Function               Description
====================== =====================================================
:func:`zero(x) <zero>` Literal zero of type ``x`` or type of variable ``x``
:func:`one(x) <one>`   Literal one of type ``x`` or type of variable ``x``
====================== =====================================================

These functions are useful in :ref:`man-numeric-comparisons` to avoid overhead
from unnecessary :ref:`type conversion <man-conversion-and-promotion>`.

Examples:

.. doctest::

    julia> zero(Float32)
    0.0f0

    julia> zero(1.0)
    0.0

    julia> one(Int32)
    1

    julia> one(BigFloat)
    1.000000000000000000000000000000000000000000000000000000000000000000000000000000

