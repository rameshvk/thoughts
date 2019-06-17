# STOMP architecture for DOT

Stomp is about splitting a DOT application into three hierarchies:

## Contents
1. [Storage Hierarchy](#storage-hierarchy)
2. [Model hierarchy](#model-hierarchy)
3. [Presentation Hierarchy](#presentation-hierarchy)

## Storage Hierarchy

The storage layer is mostly concerned with the representation for purposes of network transmission and persistence.

* Optimized for convergence and portability
    * Trees, not full graphs
    * Use simple portable primitive types.  Use maps/hashes for objects whenever possible.
* Optimized for ease of schema changes.
    * Avoid deep hierarchies: changing them atomically is difficult
    * Top level entries are heterogenous object collections (per type of object)
    * Use maps for collections (and multi-valued entries) instead of arrays.
         * Changing sort order in the schema is difficult with arrays but easy with sort columns
         * Converting from  single valued to multi-valued is more natural with map-collections
    * References to objects are just by (top-level-collection name, id within collection).
    * Life-cycle issues are simpler but result in garbage collection issues and zombie objects
* Data migration issues still remain, though minimized
    * most typical but still difficult use case: migrate from single value to collection for a field
        * would break old code if it didn't accept multi-valued from day-1 (not difficult to support)
        * old code may try to write single-value and thereby lose stored multi-value.  Probably ok in most cases as users learn.
    * older/newer clients may not understand newer/older data types
    * freeze-migration: migration creates a new object and freezes older object. All old clients only need to understand the concept of freezing even if they don't understand new type being migrated to.  (older object can have a field indicate where the new location is).

## Model Hierarchy

* Meaningful representation matching product/business understanding
    * reference IDs replaced with actual live reference objects, orphaned objects removed
    * multiple views of the same content (using secondary indices or joins or sorts)
    * native idomatic richer types with augmented methods (structs, classes etc)
    * permissive storage types converted to more limited model types (convert multi-valued entities to single value for simplicity, for instance)
* Most business policy and constraints are implicitly expressed via the model APIs
    * Enforcing constraint in model mutations are not enough: merging may result in constraint violation.
    * As constraints change, older clients may still not enforce newer constraints.
    * Need business policy to resolve constraint violations sensibly.
* Declarative computation from storage to model layer
    * Composed with bi-directional stream transformation primitives
    * Bi-directionality is a hard problem in general but composable primitives (filter, map, reduce, etc) address a lot of common cases safely.
    * Lazy vs automatic computation
* Snapshots tend to be of this layer
* Dynamically derived models
    * These are like "user configured views": `table.filter(x = z)`
    * Not all apps have these but having the ability to do these is very neat
    * These are easiest done on top of the weakly-typed storage layer with the computed result mapped to model layer
    * The derivation ("code") is itself a tree and can arguably be represented/edited with OT.

## Presentation Hierarchy

* Focused on ease of UI composition and less on Model mapping.
* Changes very frequently with marginal impact on model hierarchy.
* Declarative mapping between model and presentation is declarative but with UI primitives
    * Ideally, UI components are split into widgets (no reference to specific app model) and top-level UI layout components that bridge models and the widgets
    * Since derivation is based on streams, the same derivation may work on server to produce HTML as on client to produce interactive UX
