# Memo to myself

## Bi-directional (or 2-way) declarative building blocks 07/11/2019

Instead of writing `z = x + y`, consider this a constraint: `z - x - y = 0`.  This allows the system to automatically compute change of x if z changed (via `x = z - y`).

It makes even more sense if we consider multiple constraints:

```js
   z - x - y = 0;
   x < 10;
```

Now, if `z` changes, `x = z - y; x < 10` results in updating x so long as it honors the `x < 10` constratint at  which point, the update  of `z` can be transferred to `y` (via `y = z - x`).

But constraints are notoriously difficult to build mental models, so the **form** of expression should probably remain `z = x  + y` with the **intent** being understood as a bi-directional declaration.

The general constraint solving problem is hard.  Where the constraint cannot be solved, we end up with a one-way declarative expression as in the current world with the output being (readonly).

### GROUP BY and UNION

Consider `Union("SourceTable",  TableA, TableB)` to be an operation that simpy combines all rows from tableA and tableB (they can be heterogenous).  This is similar to SQL Union statement with the one distinction that the "SourceTable" is a new column in the output specifying which table (TableA or TableB) the output row originates from.

Now notice that `GROUPBY("SourceTable", Output)` can take the output of the `Union` and produce a collection of sub-tables that mirrors `tableA`, `tableB` etc. 

These are symmetric operations and so we can treat each as effectively a 2-way declarative statement with edits in one side being meaningnfully translated to edits on the other side and vice-versa.

### Two-way building blocks combat organic complexity

All data layers typically get more and more complex with time with a lot of dependencies that get tacked on.  Two-way building blocks like these provide a neat escape hatch: If one starts with an individual table, a `UNION()` can create a view that starts out as a `dependent` but since it is 2-way, this can be converted to being the `master` at any point (with all the original tables now becoming the `dependent` versions) without changing anything in the overall data schema.

This type of using of 2-way declarative statements to reshape the schema is effectively an extremely safe way to reshape data.

### A lot of calculations are not two-way declarative.

Calculations like `COUNT(X)` etc happen frequently and are not 2-way.  But a lot of expressions that appear to be one-way are actually *partial* two-way: `FILTER(Table, Column1 = "boo")` only has part of the information.  But we consider the rest  a *residue* (ie `Rest = FILTER(Table, Column1 != "boo")`) that is hidden.  Now the original table can be considered a `dependent` version of the `UNION` of the filtered and the hidden table.   Even in this case, we can easily make schema changes possible.

###  Multiple hierarchies

Most data participate in multiple hierarchies.   The ability to construct 2-way bindings allows constructing mulitple hierarchies of the same data.


## Values 07/11/2019

A pure functional value system can be constructed with simple composition operations:

1. Sequence(val1, ...)
2. Map(key1 = val1, ....)

Most languages associate a "static type"  with values, but this can be considered the first element of sequence (or a special key "type").

Ideally, the map should support any value as its key type.  Most languages restrict this but with a good hash code function, it seems this is doable.

A side effect of having hash-codes is the ability to implement relatively fast (a < b) and (a == b) **deep value** comparisons using the hash codes.  Obviously, the hash code calculations have to be memoized or precomputed which sacrifices performance for space.

### Hashcode for primitive types

Lots of standard algorithms but simplest is possible 32-bit CRC of binnary data

### Hashcode for sequences

See: [How to Hash a Set RIchard O'Keefe](https://www.preprints.org/manuscript/201710.0192/v1)

Use Robert Jenkins' [NewHash](http://burtleburtle.net/bob/hash/evahash.html) for strings and array sequences.

Note: [Blake2](https://blake2.net/blake2.pdf) is probably a better function but it is likely to perform poorly, particularly in interpreted languages.

### Hashcode for maps

Based on [How to Hash a Set RIchard O'Keefe](https://www.preprints.org/manuscript/201710.0192/v1):

1. Use Xor(4) if performance is super critial

```js
const hashes = [0, 0, 0, 0];
for (elt of seq) {
    const h = hash(elt);
    hashes[h%4] = hashes[h%4] xor h;
}
return h[0] xor h[1] xor h[2] xor h[3];
```

2. Use Fold (or a symmetric polynomial) if perf is not super critical:

```js
const result = 0;
for (let elt of seq) {
  const h = hash(elt);
  result = 3860031 + (result+h)*2779 + (result*h*2);
}
return result;
```

#### Do not export hash functions to applications

It is tempting to provide `value.hash()` as in C# but this is a mistake. It locks in an implementation to this.

Instead export `blake2.hash(val)` or something like that, so applications can rely on the stability provided by that.

## Ylang 12/23/2018

Current thinking of functional reactive language captured in
[ylang0.md](ylang0.md):
  - Does not capture processes/IPC
  - Does not include type inference/debugging

I go back and forth between building a prototype and explaining the
languaage. This week I seem to be more on the side of building a
prototype to test viability. 

## DOT Context 11/24/2018

After much resistance, I finally caved in and decided to add "Context"
to DOT. There are two motivating factors:

1. It would be nice if when a change is being applied, it has some
context available (such as the "logical time" when the change occured
or the "user" or "session" where the change occured). Since values are
also used in the context of "merging", this gets awkward as changes
are required to be free of sideeffects (both input and output). The
current solution is to pass nil everywhere when values are used within
the context of merging. A good example to work through is one where a
text maintains the "last modified user" for each segment and whether
that can be done in a convergent way with merges with the current nil
scheme. TBD

2. A weak form of injection: some value types (like Refs) may be
simpler if they had access to an interface (such as a "resolver"),
particularly in their query methods (i.e. no mutation). Today, values
are primarily meant to be sent on the wire and so should not stash
globals of any kind. This gives an out.

Both can lead to major abuse and are really a form of dynamic
scoping.  It would be nice if this were a language facility with
static typechecking and access to tools to detect abuse..

## DOT Composition [11/24/2018]

I'm looking into building an universal collaborative editor ground up
as a way to validate the [DOT](https://github.com/dotchain/dot)
prototype.  Some of the issues I'm running into with bad composition:

1. Several UI components need the ability to query the current layout
and change the rendering of dependent components based on that. A
simple example is rendering collaborative cursors (particularly when
two such cursors overlap) in the presence of text  with varying font
sizes (the issue with using span/background-color is that  the
overlapped region would need to have two background with differing
heights).  Another case of this is rendering custom overlays for
tables (such as fixed headers, a la instagram). There are two issues
hidden here: the HTML needed to make this work effectively makes
components  aware of rendering details of different parts of the
code. And: the code rendering needs to be awkardly staged so that some
random subset needs  to be rendered before other parts can re-render
and this gets more complicated in the  absence of change
notifications.

2. Focus management is another "global" system where there is no
simple composition. There is  good prior work here though: WPF has a
very clean system for
[Focus](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/focus-overview)
that seems like a good model for apps -- if they can be implemented
well in the subsytem of choice.

Both of the above problems seems to be greatly diminished if we
consider a system where the layout engine was cleanly separated from
rendering: the app uses specific layout APIs to produce a "laid out"
set of state, preferably with the ability to layout parts of the
system and compose the "laid out  parts" and finally emit something
that can be rendered. That is, all the current parts of the layout
engine would then work within the context of each widget rather than
working globally.  This would allow apps to do a better job of stuff
like infinite scrolls etc.  Similarly, if focus management was managed
by the app by using helper methods for standard behavior rather than
having to "plug" into a specific engine, it would allow apps to
customize their behavior more naturally.

### Specific outcomes:

1. Plan on explicit layout APIs eventually
2. Focus is effectively app level with its own internal delegation to
deal with routing stuff appropriately

