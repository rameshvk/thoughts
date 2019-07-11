# Memo to myself

## Values 07/11/2019

A pure functional value system can be constructed with simple composition operations:

1. Sequence(val1, ...)
2. Map(key1 = val1, ....)

Most languages associate a "static type"  with values, but this can be considered the first element of sequence (or a special key "type").

Ideally, the map should support any value as its key type.  Most languages restrict this but with a good hash code function, it seems this is doable.

### Hashcode for primitive types

Lots of standard algorithms but simplest is possible 32-bit CRC of binnary data

### Hashcode for sequences

See: [How to Hash a Set RIchard O'Keefe](https://www.preprints.org/manuscript/201710.0192/v1)

Use Robert Jenkins' [NewHash](http://burtleburtle.net/bob/hash/evahash.html) for strings and array sequences:

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

