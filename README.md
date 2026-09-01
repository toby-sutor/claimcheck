# claimcheck

Merge two documents with an LLM, then verify that no factual claim was dropped,
contradicted, or invented.

The merge is the easy half and the least interesting one. The verification is
the product. A language model asked to combine two documents will usually
produce something that reads well, and reading well is exactly what makes the
failures hard to see: a dropped sentence leaves no gap, a silently chosen value
looks decided rather than guessed, and an invented detail is written in the same
confident register as the real ones. claimcheck decomposes both sources and the
merge into atomic claims and checks every claim against the other side, in both
directions, so that each of those three failures has a name, a quoted span, and
a line number.

## What it checks

Every claim is verified in one of two directions, and the direction is what
gives the same verdict its meaning.

| direction | a claim that is | finding | what went wrong |
|---|---|---|---|
| source → merge | `MISSING` | **dropped** | a fact in a source did not survive |
| source → merge | `PARTIAL` | **partly dropped** | the merge carries the fact with a detail missing |
| source → merge | `CONTRADICTED` | **contradicted** | the merge states a different value |
| merge → sources | `MISSING` | **invented** | the merge asserts what neither source does |
| merge → sources | `PARTIAL` | **partly invented** | the merge adds a detail to a fact its sources do state |
| merge → sources | `CONTRADICTED` | **contradicted** | the merge disagrees with its own sources |

### Exit codes

| code | meaning |
|---|---|
| **0** | every claim accounted for, no conflicts |
| **1** | at least one claim dropped, contradicted, invented, or carried only in part; or a structural finding - an undeclared absence, a changed invariant token, a written title; or more than 5% of the source segments declared dropped |
| **2** | operational error - bad configuration, unreachable endpoint, or a unit of work the model could not answer usably |

**2 supersedes 1.** If any unit of work errored, the run is inconclusive and
exits 2 whatever the claims that *were* checked happened to say.
