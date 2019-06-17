# STOMP architecture for DOT

Stomp is about splitting a DOT application into three hierarchies:

## Contents
1. [Storage Hierarchy](#storage-hierarchy)
2. [Model hierarchy](#model-hierarchy)
3. [Presentation Hierarchy](#presentation-hierarchy)

## Storage Hierarchy

The storage layer is mostly concerned with representation of the full state. This lowest layer is generally designed with the following in mind:

1. Portability across runtimes and languages
2. Convergence of state (so complex graphs are represented as trees with references)
3. Ease of maintenanace (shapes changing over time)

A simple way to go about this is to treat the app state as a set of top-level object collections (users, messages etc) with each object within the collection having an unique ID used for reference elsewhere.  The objects themselves can be thought of as a map of fields, most of which are likely to be simple primitive types.

This approach leads to some tradeoffs: objects get orphaned and so may need to be garbage collected. In general, the flexibility of having a flat hierarchy is that it allows deep virtual hiearchies to be built on top and shifted around with ease.

One particular issue with loose constraints is what happens when a product requirement changes the shape of an object in conflicting ways. This approach can be used to do explicit migrations (like databases) or copy-on-write migrations (where the object can be migrated to a new location with the old location updated with a pointer) or even schedule user-driven migrations.

## Model hierarchy

* Meaningful representation matching product/business understanding
    * reference IDs replaced with actual live reference objects, for example
    * multiple views of the same content (using secondary indices or joins)
    * native idomatic richer types with augmented methods (formatted name vs raw etc)
* Most business policy and constraints are implicitly expressed via the model APIs
    * Ordered elements availalbe via lists
    * Unordered elements returned as maps
    * ReadOnly lists for things that cannot be mutated etc
* Declarative computation from storage to model layer
    * Composed with bi-directional stream transformation primitives
    * Lazy vs automatic computation
* Most schema changes here do not break older clients and don't require storage schema changes.
    * Backwards compatibilty may still require versioning approaches
    * Newer schema may still cause trouble with older models.
    * Zombie objects are one solution (object locked down because it has been migrated to another version with all edits only possible on clients that understand that version).
    * Another solution is *merge* (edits to old are migrated to new, edits to new are migrated to old) when there is no information loss. And eventually garbage collected.
* Snapshots tend to be of this layer
* Dynamically derived models
    * These are like "user configured views": `table.filter(x = z)`
    * Not all apps have these but having the ability to do these is very neat
    * These are easiest done on top of the weakly-typed storage layer with the computed result mapped to model layer
    * The derivation ("code") is itself a tree and can arguably be represented/edited with OT.

## Presentation Hierarchy

* Focused on ease of UI composition and less on Model mapping.
* Changes very frequently with marginal impact on model hierarchy
* Declarative mapping between model and presentation is declarative but using UI terms
    * Ideally, UI components are composable that the dynamic derivation approach for models also works here
    * Since derivation is based on streams, the same derivation may work on server to produce HTML as on client to produce interactive UX
