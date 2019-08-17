# Bidirectional Convergent Programming

Bidirectional convergent programming is an effort to formulate a
functional reactive approach to distributed computation that
eventually converges (using operational transformation or CRDT to
achieve this).

This particular formulation has a few other properties:

1. Strong rich types
2. A simple and readable implementation, particularly for mutation
3. Branch/merge/undo semantics builtin.

## Immutable values and changes

The backbone of the system is a small set of composable immutaable
types which support a small set of composable changes.  The result of
applying a change to a value is a new value, leaving the original
value intact.

The changes, when applied in parallel, can be merged using operational
transformation yielding convergent values but a framework is  still
needed to track changes

## Convergent Streams

Streams are an almost *immutable* structure that forms the basis of a
highly functional, reactive convergent setup.

Streams can be thought of as a linked-list of versions.  Each version
has a reference to the next version.

Streams are almost immutable:  mutations on a single version do not
mutate the underlying value but return the new version.  It is not
quite immutable: the next reference is updated but in reality, this is
a write-once type situation: once the next field is initialized to a
non-null value, it is never modified after.

Streams converge: the latest value can be found by walking the next
refereence. If the latest value is mutated, its next reference is set
to the updated value.  If an earlier value is mutated, the change is
transformed and then tacked on to the latest.  But the mutation
function does not return this latest value.  Doing so will lead to
unexpected behavior that is also hard to reason as the code which
mutates a value can have an expectation that it appear to have worked
in isolation.

Instead, the result of mutating an older version is a new stream.  The
value reflects only the mutation on  top of the old value but the
next references of this chain are transformations of the next
references of the older version.  The transformations are such that
both chains converge  to the same value.

A picture is worth a thousand words, so:

```
   Initial: S0 -> S1 -> S2
   Mutate S0. result = Cx

   S0 --> S1 --> S2 --> Cx
    \               /
     C -> S1x -> S2x
```

Operational transformation is used to find S1x, S2x and Cx to make
this work.

As the diagram above shows, streams can be thought of as all
converging on the latest value.  This abstraction holds even if the
mutations are on separate clients -- the implementation of stream
hides OT and its complexities.

### Properties

1. Every stream implements a next property that converges to the same
"latest" value.
2. The next property provides both the change (which is useful for OT
impleementations but also to implement dervied data later) as well as
the next value (which can be lazily computed if needed).
3. The ability to watch for changes is limited.  Changes notifications
implicitly introduce order of execution, scheduling etc but also end
up with difficulties with reasoning as they break the immutable nature
of a single stream instance.
4. Mutable methods do not mutate the underlying value but return a new
instance in the stream. Following the next chain of any instance of a
stream is guaranteed to converge to the same value.
5. Streams can be strongly typed. A Text stream, for instance can have
strong methods for dealing with text values (though the next field may
be weakly typed because a text field can be mutated to something other
than text).
6. Streams track the future, not the past. So, garbage collection
works very effectively.

## Reactive streams

Consider a computation that is a simple concatenation of two string
streams where the underlying values are joined together:

```js

// javascript example though works in any language
function join(s1, s2) {
   // assume toString converts text streams to actual strings
   const joined := s1.toString() + s2.toString();
   const nextf = func() {
      let s1n = s1.next;
      let s2n = s2.next;
      if (s1n || s2n) {
          s1n = s1n || {change: null, version: s1};
          s2n = s2n || {change: null, version: s2};
          return {
            change: /* not important here */,
            version: join(s1n.version, s2n.version),
          }
      }
   }
   // make a value with a custom next property
   return makeTextInstance(joined, nextf);
}
```

This derived computation is reactive in the `pull FRP` sense. The
result of the computation is a stream that tracks the two underlying
streams and recomputes as needed.

## Stream changes

In fact the example above can be replaced with any pure functional
computation and much of the code remains the same.  It is easy to
think of an API which abstracts much of this:

```js

function join(s1, s2) {
   return reactive(s1, s2, (x1, x2) => x1.toString() + x2.toString())
}

```

The main difficulty with that approach lies in the piece of code not
shown in the earlier example, particularly with the line where the
change is calculated:

```js
      return {
        change: /* not imporant here */,
        version: join(s1n.version, ss2n.version),
      }
```

This change computation is often not trivial.  It is possible though,
to build a set of reactivee primitives (array concatenation, splicing,
object zipping, extending etc) that handle this well and then compose
these to get the right results.

## Bidirectionality

Yet another problem is how to mutate the derived streams. The derived
`join` stream above should probably behave like a text stream for all
practical purposes, so the question naturally arises if we can define
mutation methods on it.

There are a few variations possible:

1. Make derived values be read-only (i.e. a specialization of the
mutable stream type).  For some cases, this is reasonable. For
instance a computation that yields the size of the string is best
considered a readonly value.

2. Proxy mutations over.  This is valuable for things like `fields`:
if the computation simply yields the field of an object, the mutation
can very naturally be passed through.  But the `join` case is  a bit
trickier: if someone replaces  the whole derived string, what should
the underlying implementation do? There are two obvious choices:
replace the first or second and empty the other. In some cases, these
mutations can simply fail -- maybe only mutations that affect just one
underlying string can be considered.  The main issue with proxying in
ambiguous cases is the fact that the behavior is often a matter of
policy.

3.  Yet another choice is branching: mutating a calculated value is
possible and even acts coherently but the mutations are simply
considered a branch that never gets committed.  The elegance of this
choice is that code written to work with writeable streams can be used
on derived read-only streams without having to modify the code.

