# The Y language, Version 0

## Literal types

The basic literal values are *numbers* and *strings*: **1.2**, **-2**,
**"hello"**

Numbers do not support scientific notation or other forms. All numbers
are logically treated ass if they were 64-bit floating point numbers
(though physical representation may be different).

Strings can use either single quotes or double quotes but the open
quote should match the close quote.  String constants are
automatically concatenated: **"Hello " "world"** is the same as
**"Hello world**.  String constants do not have any escape sequences,
including newlines and such.

## Simple value composition: records

A richer value can be created from the basic types using value
composition using **Records**:

``` Complex{x = 1.2, y = 5} ```

In the example above, **Complex** is the name while the fields are
**x** and **y**.

Fields can be accessed via the dot notation: **Complex{x = 1.2, y =
5}.y** for instance.

The name of the records does not generally play a significant role and
is primarily to make the code readable. In particular, it can
altogether be omitted: **{x = 5}** and can be reused with different
fields without any error.

## Infix expressions

Standard operators like **+**, **-**, ***** and **/** can be used to
form infix expressions: **5 + 3.2**.

Everything is an expression in Y with a value.

## Record fields as variables and methods

Record fields can be defined using other fields: **Complex{x = 1.2, y
= 2 * x}**.  These can also be defined in any order as cyclical
dependencies are not allowed for fields: **Complex{x = 2 * y, y = 5}**

Methods can also be defined: **Complex{x = 1, y = 2, Distance(o) =
sqrt(square(o.x-x) + square(o.y-y))}**

## Let records

Sometimes it is useful to define local variables in more complex
expressions. To do this, one can use an anonymous record and simply
access a particular field. For example:

``` { result = Complex{x = 1, y = 2, Distance(o) = distance(x-o.x,
    y-o.y)}, distance(dx, dy) = import("math").sqrt(dx*dx + dy*dy),
    }.result ```

## Immutable values

All values are immutable. Modifying a field in a record can be done
using the **update** function: **update(Complex{x = 1, y = 2}, {y =
5})**.  This always returns a new value leaving the original value
unmodified (though compilers may implement them more efficiently if
they detect the old value is not used anywhere).

It is an error to use extend with a field that does not exist in the
base. It is an error to use **update** with a variable second
parameter: the fields being updated must explicitly specified (or
easily inferred lexically)

## Collections

Collections (such as arrays or lists) are implemented with the
**collection(example, count)** function which returns a object with
methods: **first(), rest(), item(index), withItem(index, item),
prepend(item), append(item), concat(iteem), find(fn)**

Collections can be used like lists using **first() and rest()** or via
random access methods. The compiler and runtime will optimize the
underlying implementation based on the actual access patterns.

## Equality

Equality has a lot of different meanings: are things structurally
equally? or do they have the same physical representation? etc.

Y only defines equality for strings and numbers. All other types
require custom equals implementations.  Numbers cannot be compared to
strings for equality and vice-versa.

## Maps/Dictionaries

Y provides the **dict(isEqual)** constructor for dictionaries which
takes ane equality method to test for equality of keys. All **set(key,
value)** calls then use the provided equality method.  To create a
dictionary of numbers, one would have to do: **dict({isEqual(x, y) = x
== y})**

Note that the above form also allows string dictionaries (though one
can do zero additions to force **x** and **y** to be numbers)

## Inline Functions

Y allows a method to have no name: **Foo{x = 23, (y) = x + y}**.  In
this case, the whole record is callable with other fields accessible
as they are normally.  The main purpose of this is to pass callables
as parameters: **x.filter({(item) = item < 5})**. In the previous
expression, the **filter** method accepts a function that is called on
each element of the array. The anonymous record + anonyous function
call allows creating such an expression inline.

    TODO: Is this needed? I think it is better to simply make such
    functions be objects and then one can write
    **x.filter({condition(item) = item < 5})**

## Rendering

All Y values can be rendered into Text, HTML or Widget. This is done
by special methods on all objects: **Render(context)**.

## Context and imports

Y supports contexts which is a form of dependency injection or dynamic
scoping: **context({search: default}).search**.  Such context
variables an be provided dynamically via: **withContext({search: ...},
fn)** -- any use of this context variable in fn will effectively have
the value provided by the **withContext** call.

Context can be thought of lexical shorthand for extra parameters. Each
context variable is effectively a parameter that is passed through all
parents until the call to withContext of a matching namespace.

In particular, contexts are expected to be strongly type checked. It
is an error to use contexts with variable context name (i.e the
argument to **context** is required to be evaluatable at compile time,
preferably a lexical constant).

External imports are done via the **import(moduleName)** function and
the modules can be overridden via **withModule(moduleName,
value)**. In this sense, modules act like context variables and
similar to context variables, it is expected that the moduleName
parameter is a lexical constant.

## Packages and files

There is no explicit concept of packages. The **import** function
works with either fixed well known strings (like "math", "os") or with
URLs in which case, the provided URL is expected to contain the source
code.  It is also possible to use relative file paths for other module
elements via **import("./other_code_file.y")** in which case the
relative path is used with reference to the curreent source file.

Note that when using source in github, one should use the specific
paths to files (like so:
**import("https://github.com/github/hubot/blob/master/README.md")**)
rather than a path to a repository.

Versioning is done at the git level using this method.


