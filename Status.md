# Memo to myself

## DOT Context [11/24/2018]

After much resistance, I finally caved in and decided to add "Context"
to DOT. There are two motivating factors:

1. It would be nice if when a change is being applied, it has some
context available (such as the "logical time" when the change occured
or the "user" or "session" where the change occured). Since values are
also used in the context of "merging", this gets awkward as changes
are required to be free of sideeffects (both input and output). The
current solution is to pass nil everywhere when values are used within
the context of merging.

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

