---
title: Coding Standards
layout: page
---

## CIS 211 Style Guide for Python Programs

   Most software development organizations have coding guidelines
   that constitute a <em>house style</em>.  These may be based on some
   more widely used standard, but often they will also incorporate norms
   and preferences developed by a particular organization.  Students
   who took CIS 210 at UO have used a style guide for that
   course, based on <a
   href="https://www.python.org/dev/peps/pep-0008/">PEP 8</a> and
   extended with some additional rules suited to an introductory
   course.
   
   Guidelines for CIS 211 projects are also based on <a
   href="https://www.python.org/dev/peps/pep-0008/">PEP 8</a>, but
   they differ in some ways from those used in CIS 210. The key
   differences are that CIS 211 style is designed to be more
   concise.
   
   This document represents my best effort at defining style
   guidelines for CIS 211, but it may require
   revision.  We may find that we need some additional rules, or that
   some existing rules are annoyingly strict, or that something is not
   as clear as it should be.  Students are encouraged to suggest
   revisions.  

## Basics: When in doubt, follow PEP 8
   
   <a href="https://www.python.org/dev/peps/pep-0008/">PEP 8</a>
   is a widely used standard for Python code.  It provides guidelines
   for all variety of matters from indentation to spacing in
   expressions.
   
   PEP 8 refers to <a
   href="https://www.python.org/dev/peps/pep-0257/">PEP 257</a> for
   docstring conventions.  We will also follow PEP 257 conventions.
   We will <em>not</em> specify function and method type signatures
   in docstrings (see below for the alternative), but when appropriate
   we may use docstring comments to say more about arguments and
   return values. 
   

## Function headers
   
   We will use Python
   <a href="https://www.python.org/dev/peps/pep-0484/"><em>type annotations</em></a> (also known as
   <em>type hints</em>) to specify the
   signature (argument and return types) of functions and methods.
   Type annotations are a relatively new feature of Python, but they
   are supported well by PyCharm.

   Type annotations make it possible to specify the
   <em>signature</em> of a function very compactly, compared to
describing argument types in a docstring.  Instead of this:


```python
def frobnaz(n, s):
    """frob the naz.
    :n:  integer, frob it this many times
    :s:  string from which we extract the naz
    :returns: string in which the naz has been frobbed
    """
```

we will write

```python
def frobnaz(n: int, s: str) -> str:
    """returns copy s with its frob nazzed n times"""
```

An absent return type is equivalent to specifying that the
function or method returns `None`.

## Method headers

We write type contracts for the methods of classes almost 
exactly as we would for a function, with two differences.  
First, we do not give a type for the `self` argument, as it 
is implied by the class. 

```python
class Point:
    """An (x,y) coordinate pair of integers"""
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y

    def move_to(self, new_x: int, new_y: int):
        """Change the coordinates of this Point"""
        self.x = new_x
        self.y = new_y
```

As with functions, the return type annotation `-> None` may be 
omitted for methods that do not return a value.  Such methods 
will generally be *mutators*, i.e., it may be assumed that they 
modify the value of the `self` object, like the 
`move_to` method above.  

Methods that return a 
value other than `None` (and which therefore have the `-> T` 
annotation for some type `T`) should typically *not* modify 
the value of the `self` object. On rare occasions, concise and 
efficient code may require a method to both return a  
value *and* modify the `self` object.  On those occasions, 
the header comment of the method should explicitly and 
unambiguously indicate that the method modifies the object. 

### Forward type annotations

The second case in which we cannot 
simply write the type of an argument or return value is 
when that type is the class we are defining.  
The type is considered 
undefined until the class is complete.  
Thus we cannot write 

```python
    def __add__(self, other: Point) -> Point:
        """(x,y) + (dx, dy) = (x+dx, y+dy)"""
        return Point(self.x + other.x, self.y + other.y)
```

because the `Point` class does not exist yet.  The workaround 
is to place quotes around the type name to indicate that it 
is a *forward reference*, i.e., a reference to a type that 
we promise will exist soon even though it does not exist 
quite yet. 
 
```python
    def __add__(self, other: "Point") -> "Point":
        """(x,y) + (dx, dy) = (x+dx, y+dy)"""
        return Point(self.x + other.x, self.y + other.y)
```

## Composite type annotations

Types like `int` and `str` can be used directly in 
type annotations.  Types like `tuple` and `list` can also be 
used in type annotations, but typically we will prefer to be 
more specific, e.g., not just a `tuple` but more precisely 
*tuple in which the first element is a string and the second 
element is an int*.  We can make this more specific annotation 
by importing type constructors from the `typing` module: 

```python
from typing import Tuple, List, Dict

def i_eat_tuples(t: Tuple[str, int]) -> Tuple[int, Tuple[int, int]]:
    the_string, the_int = t
    return (the_int, (the_int, the_int))

def to_dict(ls: List[Tuple[str, int]]) -> Dict[str, int]:
    result = { }
    for key, value in ls:
        result[key] = value
    return result
```


<h2>Method headers</h2>
Method headers follow the same rules as function signatures with
the following exceptions:
<ul>
  <li>The <em>self</em> argument of a regular method should not
  have a type annotation.</li>
  <li>Similarly, the <em>cls</em> argument of a
  class method should not have a type annotation.</li>
</ul>

<h2>Mutation</h2>

A function or method that returns a result should not modify its
input arguments.  For example, if I write this function header:

```python
def jazziest(musicians: List[Musician]) -> Musician:
```

it is implicit that the input list of musicians is not
modified.

Occasionally returning a result and modifying an argument in the
same function is helpful for writing concise and efficient code.  The
`pop` method for lists is an example of this.  This is
permissible but must be clearly and explicitly documented in the
header function, and to the extent possible the function name should
suggest that it modifies its input.  For example, if the above
function returned the jazziest musician an also removed it from the
list, the function header and docstring might look like this:


```python
def pull_jazziest(musicians: List[Musician]) -> Musician:
   """Remove and return jazziest musician from musicians"""
```

<h2>Classes: Public and quasi-private attributes</h2>

In accordance with PEP 8, object fields and methods that are meant
to be public (that is, accessed from code outside the class or its
subclasses) should begin with a lowercase letter.  Names of fields and
methods that are <em>not</em> meant to be accessed directly by code
outside a class should begin with a single underscore. (These fields
and methods might be declared <em>private</em> in a language with
encapsulation, such as Java or C++, but Python does not provide
encapsulation.)

In addition to the access categories described in PEP 8, we add
two that can only be specified in docstrings:

* A docstring may describe a class as immutable, in which case
  its public fields may be accessed but not modified by code outside
  the class.
* In a class that is not immutable, individual fields may be
  described as read-only, meaning that outside code may access them
  but must not directly change them.


## Tests
We will not use doctests in CIS 211, and in most cases our
docstrings will not include examples of use.  Instead, we will use
unit tests separate from the module code.

Unit tests may be included in the same source file as the code to
be tested, or it may be in a separate file. If it is in a separate
file, code to test `code.py` should be named
`test_code.py`.

We will use the Python
<a href="https://docs.python.org/3.6/library/unittest.html">unittest</a>
framework whenever practical.  The unittest framework may not be
suitable for testing some features, such as interactive input and
output.  In those cases, tests may be written using whatever means is
appropriate, with as much test automation as feasible.


In many cases I will provide a starter test suite with a project.
The provided test suite is <em>never</em> exhaustive, and passing the
provided test cases is <em>not</em> a guarantee that a project is
correct.  Students are strongly encouraged to add their own test
cases, and may sometimes be required to do so (that is, designing and
implementing good test cases may be part of the project
requirements). 



