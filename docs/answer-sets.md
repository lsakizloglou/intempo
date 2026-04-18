# Answer Sets

This file refers to chapters of the [FAC 2025 article](https://doi.org/10.1145/3744714).

InTempo supports three answer-set semantics and the pruning option, each combination a different trade-off between the precision of answers and amenability to change-driven evaluation. The three answer sets are selected via `--answer-set` (plain, effective, definite); enabling the pruning option `--pruning` creates a *projected variant* of each answer set that is evaluated over a pruned runtime model.

### Plain

The plain answer set contains every match witnessed given the events seen so far. It is produced by the change-driven TVGDN evaluation and is updated incrementally as each event is processed. The plain answer set may contain matches that will be affected when more events are processed; in particular, matches based on a property with a future-bounded operator whose temporal constraint has not yet elapsed. These are false positives over incomplete traces.

Plain is appropriate for offline analysis over complete logs or for self-adaptation scenarios, where premature results from the plain answer set may start an adaptation while there is still time for the system to fulfill its requirement (see §6.5 of the article).

### Definite

The definite answer set contains only those matches that no future extension of the log will affect. Formally, it contains triples (m, V^d, IV^d): the *definite temporal validity* V^d, i.e., time points where m definitely satisfies the formula, and the *definite temporal invalidity* IV^d, i.e., time points where m definitely falsifies the formula. Both are emitted directly in the match stream, in the format `{S:<V^d-ranges>, U:<IV^d-ranges>}`: S shows definite satisfaction ranges, whereas U the definite falsification ranges.

Because definiteness depends on the time point at which a query is evaluated, and that time point advances with each event, the definite answer set cannot be produced by incremental updates alone; every version of the RTMH requires re-computation of all definite spans. This makes the definite answer set expensive over longer event traces.

### Effective

The effective answer set recovers change-driven evaluation while returning only definite answers. It relies on the plain answer set and applies a definiteness check based on the *non-definiteness window* of the property: the interval after a match is first observed during which a future event could still affect it. A match is admitted to the effective answer set only for the portion of its temporal validity that falls within the definite period, i.e., the time points τ satisfying τ ≤ c − w(ψ), where c is the current timestamp.

For a property with a future-bounded operator `U[0,b]`, the non-definiteness window for a match first observed at time *t* is the interval [*t*, *t* + *b*]. A match for patient 1 in the quickstart example is observed at t=1000 with bound b=3600; t=1000 becomes a definite time point once c ≥ 4600. A match for patient 2 would first appear at t=10000; t=10000 becomes a definite time point only once c ≥ 13600, by which time patient 2 has already complied, so no effective match for patient 2 is ever emitted.

Effective is the default recommendation for online monitoring: it is change-driven, scales to large traces, and returns only answers that are definite.

### Projected (pruning)

When `--pruning` is enabled, InTempo deletes vertices from the runtime model once they age past the query's effective lookback window, keeping memory bounded over long traces. Projected answer sets are restricted to matches that involve only vertices still present in the pruned model.

When pruning is enabled, InTempo becomes competitive with the state-of-the-art in terms of performance while retaining its richer output, i.e., temporal validity intervals rather than point-based decisions---see §6 of the article. Specifically, the combination `--answer-set effective --pruning` is the recommended configuration for online monitoring over large logs: it is change-driven, returns only definite answers, and may keep memory bounded.
