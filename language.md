# A new language family: X

1. Simple: only functions, immutable types
2. Mutability for performance: compiler tools can identify variables
that can be made mutable and automatically generate code to treat them
as such. Developers can provide "hints" via annotations
3. Collection types (arrays, lists, trees, heaps etc) are treated like
mutability: language does not require those but hints and profiling
tools can generate *profiles* to enable these
4. Structural typing: all types are inferred. Compiler tools with
provide the initial type *profile* that can be modified for
performance reasons.
5. Profiles (mutability, types etc) have *continuity* -- changing code
would not break the profiles for code that has not changed and
three-way merges of profiles should be posssible
6. Custom DSL for math etc
7. Unicode from day-1
8. Readability: Code documentation can include embedded snippets or
references which are compiled for validation. Inferred types are
available for documentation
9. Transpilable: the langauge is designed so any DSL transpiled to it
can be debugged with as much facility
10. Debugging can be remote; snapshots are available etc.
11. UI framework wired in
12. Dynamic composition: at some point all languages need some form of
injection. Most simply skip over this problem leading  to awkward
contexts and some form of type-less setups.  Instead, dynamic scoping
is better being an explict part of the language so all the type-safety
tools can access and validate this.
