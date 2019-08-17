# Convergence and divergence

Most distributing computing is concerned with convergence where
changes from multiple clients need to be merged. OT helps bring
this about by decomposing the problem into immutable declarative state
and imperative composite mutations. 

But a case can be made for controlled and persistent divergence as a
feature.  And the same underlying technqiue in OT is eminently set up
to deliver this.

## A case for divergence

The basic case for divergence is the github-like pull request model,
where a branch is created.  But there is an expectation of eventual
convergence and there is the sense of working only in the context of a
single branch at a time.  In addition, the **liveness** of a branch is
limited by manual **merge** invocations.

A better case is with text-editor folding of regions of text. Folding
can be thought of as a local operation with further edits being
bidirectional (i.e. local edits and remote edits being synchronized
while the folding itself is maintained).  This example is **live** in
the sense that merges are automatic but the limitation is still that
one works in the conteext of a single branch.

Materialized views can be thought of as forks of tables (i.e. parts of
the whole data set) but which coexist along with the underlying
tables.

## Folding and hiding

With the use of OT, one can consider a branch created with a fixed set
of "folded" changes. All changes to the master can be translated to
the branch and all changes to the branch can be translated to the
master (with some caveats[1](#caveat-1)).

Folding can be thought of as unpublished local changes. The concept of
unaccepted reemote changes is **hiding** and this is complementary.
In fact, **hiding** remote changes can be thought of as accepting them
followed by adding the undo of that remote change to the list of
**folded** changes (with some caveats[2](#caveat-2)).

If the **foldings** and **hidings** are scoped (with some
caveats[3])(#caveat-3)) to parts of the full app state, and the actual
changes are considered part of the definition of a materialized view
of sorts, we end up in a situation of **embedded** branches with a
choice of **manual** vs **automatic** synchronization.

This forms a **divergent** strain of data but one which is intended to
be persistent for expressive power.

## Filters on changes

These **divergent** embedded branches can be defined explicitly as a
set of folded and hidden changes but there is also the ability to
define a function on the whole history of changes.

For example, one can define a mapping which converts currenccy from
dollars to corresponding euros (based on the conversion rate at a
particular time) and arrive at the whole app as it would be in Euros.

More interesting changes are filtering out changes by declarative
constraints (such as `tag = edited_by_user_X`).

While these **derivations** may be compute intensive if implemented in
a generic way, specific derivations may be effectively implemented.

## Divergence is easier than convergence

The set of changes that are compatible with OT is rather narrow. A
much broader set of changes can be used safely with
divergence([4](#caveat-4)) -- the only requirement is to  be able to
translate changes from one side of the branch to the other.

Consider the derivation which groups a set of individual tables into
one mega-table.  There are ways to do this such that changes on
either side can be converted to changes on the other side.

## Equivalent derivations

A class of powerful derivations are those which define equivalent
representations (such as alternate views of the data).  These provide
a convenient hook for refactoring a system: one can at some point view
the embedded branch as the master and invert the relationship by
making the master branch a derivation.

Another way of thinking of this is to view a master and divergent
branches as being in a symmetric relationship (defined by the folded
and local changes). While the concrete representation may use either
as the master, the logical shift to using the other as the master is
feasible ([5](#caveat-4))

## Code itself as data

One can view code itself as data (whether it is basic lisp-expressions
or more AST). The compiled program can be seen as a derivation.
Deployment and other artifacts can be seen as derivations.   A
materialized view of the code is simply another way of putting
together code  to support a situation. For example, a component with
ten features can be implemented with feature flags for each one of
those. A materialized view with the feature turned on/off can be an
expressive way to work with it (such as disabling retries by simply
filtering out that feature).

Feature flags can also be implemented on changesets -- and the same
materialized view can be had by running a filter on the changes.  This
only works if the changes themselves satisfy the conditions described
above which would require the code itself to be more declarative.
None of the existing languages are setup for this directly but in a
way, this requirement is more about how code is edited and its
possible there are ways to create changesets that maintain this
ability.

## Sharing, copying, tracking

The mental model of divergent data is quite daunting.  One approach is
to think of all data as being in one of three categories:

1. **Shared** -- all changes eventually converge to a single master
instance.

2. **Copied** -- all changes stay in silos. Synchronization is
primitive, if at all available (explicit merge).

3. **Tracking** -- the object is viewed as logically being in a silo
but still actively connected to the master. The ability is to
selectively pull (i.e. hide some remote changes) and push (i.e. fold
some local changes).   The tracked object can refer to other shared,
copied or tracked objects.    The synchronization can be automatic or
triggered maanually.  History editing is always available.

## Caveats

### Caveat 1

When changes are made to master, translating them to a folded branch
will likely modify the "folded changes" as well.   For example, for
folded text, if the change happens within the folded text on master,
it will have no effect on the branch but would change the actual
branch definition (i.e the "folding" changes)

### Caveat 2

Relying on **undo** to convert a **hidden** remote change into a
**folded** local change can have some odd behavior as OT struggles
with undo in some edge cases.

### Caveat 3

Scoping changes in general is quite tricky, particularly with derived
computations.  Still, this is a question of implementing performance
techniques (such as introspection of the dependency graphs etc).
Another way to scope things is a **declarative** model where the
branch comes with specification of which part of the branch is to be
derived.

### Caveat 4

OT requires changes to converge but persistent branches live along
with the master with no concept of merging back as they represent an
extra view.  So, the set of change-types are rather broad but they are
still limited to somewhat declarative derivations. Even stateful
computations (such as average delay between edits of a row) can be
defined if we carefully set them up to only run on the sequence of
changes on the master branch (individual client branches may have
different changes which converge).  Note that the embedded branch is
simply seen as a derivation here,

### Caveat 5

Shifing from one as the master to another is simply a question of
snapshot behavior for the most part but the ability to introspect on
the journal of changes  itself imposes a constraint that we track
those changes on the right branch.  That is, if the master branch
changes from one to another, the journal should reflect changes that
make sense on that branch.  This requirement is only **virtual**
though -- i.e. if a derivation introspects the journal on one branch,
the journal itself can be thought of as a computation based on the
branch definition.
