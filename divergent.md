# Divergent programming

Most distributing computing is concerned with convergence where
changes from multiple clients need to be merged. OT helps mannage this
complexity by think of state and deltas in composition-friendly
terms.

This same approach also allows thinking about divergence as a
desirable property.

## Folding

An relatively common existing case for divergence is when one user
decides to modify the view locally (such as **folding** regions of
text).  In this situation, we can use OT to maintain the divergence:
changes on top of the folded state can be transformed to be applied on
the unfolded state and vice-versa.

View personalizations also fall into this category. A liberal
interpretation of **folding** is a set of local changes that have
been frozen but which allow further local and remote changes to be
merged with each other.

## Hiding

**Hiding** is the counter-part to **folding** where some remote
changes are held off.  For instance, a remote change may rename some
terms from acryonyms to full names but if one prefers the acryonyms,
this change be **hidden** locally with the ability to continue to work
as if it didn't exist.

Hiding can also be expressed purely as a form of folding: remote
changes that are hidden can be thought of as having been applied
locally immediately follwed by their **undo** being **folded**
locally.

## Embedding branches

Folding and hiding can be thought of as **branching**.  Typically
branches are thought of as both complete and isolated:  the whole
state is duplicated and one works in the context of only one branch at
a time.

But a different and richer picture emerges if branches can be applied
to parts of the data and multiple branches are considered **live** in
the same context with some OT-like mechanism to make both data
available to provide access.

## Derivations

Another way to generalize this idea is to allow more kinds of changes
rather than being restricted to pure OT changes. The main requirement
for changes here is the ability to meaningfully take any new
changes that are applied on one-side and translate them to the
other.  An example is the derivation "group a bunch of tables into one
mega table".  This is not a state-less simple OT operation but it can
safely be used to construct a **folding**.

The set of such derivations is rather large, particularly when these
include the ability to translate from one type of schema to another
equivalent type

## Inverting primary and derived constructs

For all OT-like changes as well as those derivations that are fully
two-way equivalent (i.e. changes on one side can be consistently
converted to changes on the other side), the question of which
representation is primary and which is secondary is completely
arbitrary and the system can safely shift from one to another with
ease.

## More generalizations

Folding and hiding can be thought of as one form of manual editing of
**history** of a branch.  One can think of functions to edit the
history (such as exclude all changes from user XYZ or more usefully as
all changes tagged XYZ). This meta-filter on changes can be
considered a **live** branch (or a special kind of derivation), much
like the other changes.

Other interesting branch derivations could be replacing all changes
from user X to user Y etc.  That is, we can consider the changes
themselves as a domain of data over which to run operations.  A
concrete case here would be to retroactively change Euro-denominated
transactions to Dollar-denominations based on some table.

## Live vs static embedding

The liveness of folding, hiding or derivation can be implicit or
explicit (i.e automatic synchornization vs requiring a user-triggered
synchronization).

## Sharing or single-instancing vs Copying vs Tracking

Sharing is a single-instance that converges.  Copying is multiple
instances that diverge but live in separate worlds. While merging
copied state is possible, selective merging is not possible.  Tracking
is when history is a lot more malleable with local unpublished changes
and hidden remeote changes mingling.

## Code itself as data

The code that runs an app can itself be thought of as state with
folding, hiding and other such possibilities.  For instannce, we can
define a Desktop version of the code base as that without all the
chagnes tagged "Mobile".   Embedding the desktop version into the same
codebase that has the full code-base allows for some powerful ways to
build code (as well as modify code) that traditional codebases can't
support.

## Deployments as code artifacts or derivations

OK, this is crazy extreme but one can view the deployments and running
service as an artifact or derivation of the code.  Unfortunately, the
real world (such as DNS or web-sites etc) have integrity
constraints. So embedding etc will allow us to get far but when
integrity constraints are broken, we may need some clever way to get
around them.
