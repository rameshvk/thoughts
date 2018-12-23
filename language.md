# A new language family: X

## Principles

1. Simplicity, ease of use and productivity for beginners,
non-professionals as well as professionals.

2. Tools are part of the language yet written to be customized. Tools
include both static tools (compilers, transpilers, type checkers,
performance optimizers) and runtime tools (debuggers, log/tracers,
crash dumps, hot-reloaders).

3. Package management and deployment are part of the language --
packages, distribution, versioning, snippets.


## Big bets

0. No explict allocation/free. The language uses a combination of ref
counting and garbate collection.

1. All values are immutable. The compiler can choose to implement
mutability for performance but the language provides no way to access
it.

2. Use a process model with RPC for concurrency.  The compiler can
implement these with either message passing or simply rewrite
everything to be asynchronous as it prefers.  The process abstraction
also can be use for IPC as well as networked communication.  The
process boundary is a good place to handle errors (exceptions)

3. Use lexical scoping for all the common cases but support dynamic
scoping for dependency injection with explicit namespaced syntax
(i.e. dynamically scoped variable names are namespaced to avoid
collisions).  The compiler can implement dynamic scopes via implicit
parameters or using  process local variables.  Dynamic scopes cannot
be passed across process boundaries.

4. The syntax is based on expressions (everthing is an expression with
a return value). Records (structs) can be created by json-like syntax
(with equals sign instead of colon). Infix operators are mapped to
methods on the left operand. The dot operator works as expected though
records are not  the same as JS objects in that the keys must always
be constants (no variable field access) and there  is no way to
enumerate the fields. Unlike JSON, functions can be part of the record
with the key being the name plus the args list and the value being any
expression (which can use the args in the args list). The fields of a
record can also access other fields as part of their definition. There
is no difference between  functions with no args and fields. An
unnamed function will effectively make the record callable.  The names
of records are for documentation purposes only and anonymous records
are allowed.

5. No classes or other higher level types are needed. All methods are
type checked using structural equivalence (same methods). In
particular, there are no arrays or pointers as far as syntax goes but
there are functions which return objects with methods/properties that
look like arrays/lists etc. The underlying implementation is compiler
dependent.

6. Compile-time methods are prefixed with the @ sign. These are run by
the compiler but its parameters are not evaluated. Instead, the AST
for the parameters are provided to the given @ function. This allows
very rich macro-functionality. In particular, it is expected that
transpilers will be built using this.

7. The  @ macros also provide the way to implement  annotations which
allow performance hints (mutable vs immutable) etc to be attached to
the file. All parts of the AST have semi-permanent paths used with
revision control so the hints can be translated to  later versions.

(which are internally mapped to methods on the operands). 
language eposes contranspiled
to a message queue based set of primitives (

The X language has a core philosophy of being able to simply read and
write correct code or beginners and advanced developers alike.

This often takes the form of *Concise stable form, diverse varying
function*

Having a stable form is crucial for those unfamiliar with the language
or with the particular codebase to navigate well.  Its like having
familiar road signs.

Some of the interesting ways this plays out for X is how X deals with
mutability or documentation.  X specifies immutable values (and hence
types) as the basis for ease of writing correct code as well as
reading code knowing about possible side-effects immediately. But
immutable code often performs poorly and to achieve the performance,
the developer has to provide "hints" that can be used by the compiler
to automatically choose mutable implementations.  The form of the code
remains immutable but it functions as mutable and only those who care
need to know.

Another interesting case is documentation.  Documentation needs are as
rich as with applications themselves.  The X langauge defines a form
for documentation (islands of code within documentation with some code
islands marked as test, examples etc).  It specifies very little about
the non-code parts.  In particular, it allows blocks of compiler
instructions  to transform code and documentation blocks in ways that
are meaningful to developers.  The form of the whole
documentation/code ensemble is relatively fixed but the
interpretations of each island allow diverse varying behavior

## Immutable values

Immutable values are easier to code with as they prevent a lot of
mistakes that arise with mutability.  They are also often helpful when
debugging.

But they often lead to awkwardess that are mitigated as follows:

1. Modifying a field has a syntax which allows the immutable "clone
with new field" to be expressed simply and clearly
2. Loop expressions accept naked closures (i.e. no need to qualify
with  "function()" prefix) so they read very similar to non-functional
languages
3. Side-effects allowed in system operations

A common complaint against immutable types is their performance
impact. This is mitigated by compiler techniques to rewrite code that
does not require immutability to use mutable fast types.  By moving
this to the tools (and maybe to hint/optimization files), the core
logic is not polluted with complex type/performance choices that can
lead to errors.

## Inferred simple types

Type systems perform three functions: compiler memory layout
integration, correctness/type validation, documentation/readability.
The X language is mostly concerned with the latter two: the ability to
type check code for validity and to document what is expected when
someone calls a piece of code.

Towards this end, all types are inferred in X and the compiler chooses
a default (possibly sub-optimal w.r.t performance) layout.  And
validation tools kick-in validating whether the shape of the provided
parameter matches whatever is expected  of that function.

Without any type information, some complex validation is not possible
(such as type categories of Haskell). TODO: find simpler ways to do
this (possibly with parameterized constraints or separate hints)

Tools can similarly provide the ability to look at inferred
types. This does leave out the ability to document what a particular
field means in a particular context. TODO: find a way to do this.

The actual memory layout is chosen by the compiler.
