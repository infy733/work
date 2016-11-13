***********
 Variables
***********

A variable, in Julia, is a name associated (or bound) to a value. It's useful when you want to store a value (that you obtained after some math, for example) for later use. For example:

.. doctest::

    # Assign the value 10 to the variable x
    julia> x = 10
    10

    # Doing math with x's value
    julia> x + 1
    11

    # Reassign x's value
    julia> x = 1 + 1
    2

    # You can assign values of other types, like strings of text
    julia> x = "Hello World!"
    "Hello World!"
