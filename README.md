# Holochain Entry-type Patterns
Every Holochain app developer will make decisions regarding the patterns described in this
specification.  The purpose of this document is to describe the concepts behind each pattern and
provide a shared vocabulary.

These pattern decisions apply to each entry type.  A zome may employ more than 1 pattern but each
entry type is expected to use 1 pattern consistently.


## Overview

|    | Purpose                     | Description                       | Options                                                        | Flag      |
|----|-----------------------------|-----------------------------------|----------------------------------------------------------------|-----------|
| 1. | [Chronology](#1-chronology) | the action relationship framework | [Flattened](#flattened) vs [Chained](#chained) Updating        | `F` / `C` |
| 2. | [Provenance](#2-provenance) | the lifecycle identifier          | `EntryHash` vs `HeaderHash` Entity IDs                         | `E` / `H` |
| 3. | [Navigation](#3-navigation) | the linking model                 | [Permalink](#permalink) vs [Replacing Links](#replacing-links) | `P` / `R` |
| 4. | [Congruence](#4-congruence) | the conflict resolution type      | [Operation](#operation-based) vs [State](#state-based) based   | `O` / `S` |


### What is an Entity and an Entity ID?

- *Entity* - a word that represents a conceptual object that is defined by its life cycle
- *Entity ID* - a word that represents the entity's create hash

For example, an entry defines some immutable data.  A header defines an action where a series of
actions can be used to define some thing.  What name can we use for that thing?  Entity - *a thing
with distinct and independent existence*


### 16 Pattern Combinations
The pattern expression could be defined as `(F|C)(E|H)(P|R)(O|S)` and all 16 potential pattern
combinations are featured below.

|        | Chronology | Provenance | Navigation      | Congruence | Note                                            |
|--------|------------|------------|-----------------|------------|-------------------------------------------------|
| `FEPO` | Flattened  | EntryHash  | Permalink       | Operation  |                                                 |
| `FEPS` | Flattened  | EntryHash  | Permalink       | State      |                                                 |
| `FERO` | Flattened  | EntryHash  | Replacing Links | Operation  | `__RO` are [counterintuitive](#pros-and-cons-1) |
| `FERS` | Flattened  | EntryHash  | Replacing Links | State      |                                                 |
| `FHPO` | Flattened  | HeaderHash | Permalink       | Operation  |                                                 |
| `FHPS` | Flattened  | HeaderHash | Permalink       | State      |                                                 |
| `FHRO` | Flattened  | HeaderHash | Replacing Links | Operation  | `__RO` are [counterintuitive](#pros-and-cons-1) |
| `FHRS` | Flattened  | HeaderHash | Replacing Links | State      |                                                 |
| `CEPO` | Chained    | EntryHash  | Permalink       | Operation  |                                                 |
| `CEPS` | Chained    | EntryHash  | Permalink       | State      |                                                 |
| `CERO` | Chained    | EntryHash  | Replacing Links | Operation  | `__RO` are [counterintuitive](#pros-and-cons-1) |
| `CERS` | Chained    | EntryHash  | Replacing Links | State      |                                                 |
| `CHPO` | Chained    | HeaderHash | Permalink       | Operation  |                                                 |
| `CHPS` | Chained    | HeaderHash | Permalink       | State      |                                                 |
| `CHRO` | Chained    | HeaderHash | Replacing Links | Operation  | `__RO` are [counterintuitive](#pros-and-cons-1) |
| `CHRS` | Chained    | HeaderHash | Replacing Links | State      |                                                 |



## The 4 Pattern Vectors

### 1. Chronology

> *the arrangement of events in the order of their occurrence*

This pattern option relates to updating entries; more specifically, which header is chosen for the
update's base.  The options are a flattened (`F`) or chained (`C`) approach to basing.

#### Flattened
Flattened chronology means that all updates are based off of the Create header.  The order of
updates would be determined by the header timestamps.

*Example of multi-agent flattened updates*

![](https://drive.google.com/a/webheroes.ca/thumbnail?sz=w1000&id=1-PTXNUo_oFF-imgbVPTqJLzYpwJLYXXK)

#### Chained
Chained chronology means that updates can be based off of other Update headers.  The order of
updates is determined by the chain reference order.

*Example of multi-agent chained updates*

![](https://drive.google.com/a/webheroes.ca/thumbnail?sz=w1000&id=1zPjc-0smNeEsB_yReqs-0-NYRu5m8Bf_)

#### Pros and Cons
When only 1 Agent can make updates to a particular Entity, flattened seems to have no cons.  The
order of updates is inherently preserved by the Agent's chain.

When there are multiple Agent's who can make updates, the flattened approach leaves opportunity for
updates to overwrite each other.  Whereas chaining the updates would preserve the intention and
intrinsically allow the deduction of deltas.

If paired with operation-based congruency, the flattened approach would not suffer from overwrites
as easily because the probability of 2 Agent's overwriting the same field is lesser.

Chaining will result in dead ends sometimes which means some branch in the chain of updates was
orphaned.  The resolution of orphaned updates depends on the congruency vector.

The chained approach cannot use `get_details` to retrieve the list of all updates.  It would require
recursive `get_details` to build the full chain from header references.  The simple solution for
this is to add additional links so that all updates can be seen with a single `get_links` request.



### 2. Provenance

> *the earliest known history of something*

This pattern option relates to the Entity ID.  The options are the `EntryHash` or the `HeaderHash`
of the Create event.

> **DISCLAIMER** As of October 2021, Holochain does not support links to or from header-hashes.
> This is not because of architectural limitations and it may be supported in the future.  This
> means that any patterns using `_H__` are not natively supported.

Since this is not natively supported at the moment, further analysis of this pattern will be found
under [Thought Experiments](#thought-experiments).



### 3. Navigation

> *the process of ascertaining one's position and following a route.*

This pattern option relates to the discovery of child Entities from some parent Entity.  The options
are "Permalink" or "Replacing links".

#### Permalink
The links from a parent to child are always from parent Entity ID to child Entity ID.

#### Replacing-Links
The links from a parent to child are updated when the child Entity is updated.

#### Pros and Cons
Replacing links would not be a suitable model when the Agent updating the child Entity does not have
permission to update the links from the parent.  This would make links unreliable and disrupt the
conveniences of this model.

Pairing "replacing links" with operation-based congruency seems to be counterintuitive.

Some readers may realize that "replacing links" could be extended to replacing every time the parent
is updated.  This design greatly increases the complexity with no clear advantage.  If anyone can
describe a scenario where that design would be beneficial, please [submit an
issue](https://github.com/mjbrisebois/hc-4-pattern-vectors/issues).



### 4. Congruence

> *agreement or harmony; compatibility*

This pattern option relates to
[CRDT](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type).  The options are state or
operation based resolution strategies.

#### Operation-based
The Entity's most current state is determined by applying all update operations to the initial state.

#### State-based
The Entity's most current state is determined by the most current update.

#### Pros and Cons
Operation-based changes will require more computation as more changes are made.  One way to resolve
this is to pair some operation-based entry type, with a state-based entry type where the operations
are collected at some interval to produce a full-state entry.  This prevents the operational changes
from being infinite.

When only 1 Agent can make updates to a particular Entity, there appear to be no pros and cons for
the Agent.  Most of the reasoning of each option relates to collaborative scenarios.  Since we can
deterministically prevent overwrites within the same chain, the rationale for each option would
reside with the developers preferences.

In scenarios where the goal is real-time collaboration, state-based updating is more likely to
result in Agent's overwriting other Agent's updates.  Since there is no central source to determine
which update crossed the finish line first, the feedback loop for overwritten changes is slower and
not as obvious.

Support for offline usage will be a problem for either option when there are multiple Agent's who
can make updates.  Offline support must be designed at a higher level.



## Comparing Patterns

Coming soon...


## Thought Experiments

Coming soon...


## Contributing
There may be many extraordinary or less common scenarios that this framework does not directly
address.  If you have a scenario that does not fit into any of these patterns, it would be great to
learn about it.  Please [submit an
issue](https://github.com/mjbrisebois/hc-4-pattern-vectors/issues) explaining your scenario.
