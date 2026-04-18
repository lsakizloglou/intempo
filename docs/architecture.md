# Architecture

A brief overview of how InTempo processes a stream of events and evaluates a temporal graph query. For the full formal treatment, see §2–§5 of the [FAC 2025 article](https://doi.org/10.1145/3744714).

## Inputs

1. A **metamodel** describing the types of entities and relationships in the system (e.g., `SHSService`, `PMonitoringService`, `Probe`).
2. A **mapping from events to model modifications** (`.e2p` file): each event type is mapped to a sequence of additions or deletions in the runtime model.
3. A **temporal query** (`.itql` file): a graph pattern together with an MTGL formula that imposes ordering and timing constraints on matches.
4. An **event log** (`.log` file): timestamped events in order of occurrence.

## Runtime Model with History

Events are processed against a *Runtime Model with History* (RTMH): a typed attributed graph where every vertex carries a creation timestamp `cts` and a deletion timestamp `dts`. When an entity is deleted in the modeled system, the value of its `dts` is updated rather than the vertex being removed, so the trace of timestamped snapshots is encoded compactly in a single graph.

## Query operationalization

The query is decomposed into a *Temporal Generalized Discrimination Network* (TGDN): a tree of Graph Transformation (GT) rules where each rule corresponds to an MTGL operator (*exists*, *until*, *negation*, *conjunction*, *since*, *true*). The rules of a TGDN are called *marking rules*; when applied, each creates a *marking node* that marks an intermediate match and stores the satisfaction span of the sub-condition. The root of the TGDN returns matches for the query pattern paired with their satisfaction span.

Extending the TGDN with a *terminal rule* yields a Temporal Validity GDN (TVGDN), whose result intersects the satisfaction span of each match with its lifespan to produce the *temporal validity*, i.e., the answer set returned to the user.

## Change-driven and incremental evaluation

When a new event is processed, only those rules whose LHS is affected by the change are reapplied; answer sets stored by network nodes between evaluations are incrementally updated. These features, together with the efficiency of the underlying graph-pattern matcher, is what allows the plain, effective, and projected answer sets to scale to large traces.

The definite answer set trades efficiency for precision: because the definite semantics depends on the time point at which a query is evaluated, every new version of the RTMH requires a re-computation of all definite spans. The effective answer set recovers change-driven evaluation while still returning definite answers: it relies on the plain answer set and applies a definiteness check based on the non-definiteness window of the property.

## Pipeline summary

```
    event log            .e2p mapping            .itql query
        │                     │                      │
        ▼                     ▼                      ▼
  ┌───────────────────────────────────────────────────────┐
  │                InTempo engine                         │
  │                                                       │
  │     events → RTMH modifications    query → TGDN       │
  │                   │                       │           │
  │                   └────────► TVGDN ◄──────┘           │
  │                                │                      │
  │                                ▼                      │
  │                       (optional) pruning              │
  │                                │                      │
  └────────────────────────────────┼──────────────────────┘
                                   ▼
                             answer set
                      (matches + temporal validity)
```

See Fig. 4 of the paper for an overview and §5 for the rule-by-rule construction of the TGDN.
