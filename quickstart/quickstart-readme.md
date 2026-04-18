# Quickstart

This example demonstrates the basic functionality of InTempo, and moreover how different answer sets treat an uncertain verdict. The example is based on one of the use-cases used to evaluate the performance of InTempo in the [FAC 2025 article](https://doi.org/10.1145/3744714).

## Files

- `shs.e2p`: maps SHS event types to graph modifications. Taken verbatim from the FAC 2025 replication package.
- `sepsis-antibiotics-1h.itql`: the temporal query. Encodes the instruction: *"Within one hour of a sepsis triage, there should be an IV antibiotics event for the same patient."* This is a simplified version of `MG1` from the replication.
- `quickstart.log`: four timestamped events covering two patients.

## The Log

```
ER Sepsis Triage;1000;1
IV Antibiotics;5000;1
ER Sepsis Triage;10000;2
IV Antibiotics;13000;2
```

Format: `event type ; timestamp (seconds) ; patient ID`. Patient IDs are integers (`shs.ecore` declares `pID` as `EInt`); we refer to them as patient 1 and patient 2.

- **Patient 1:** triage at t=1000, antibiotics at t=5000 → 4000-second gap → **falsifies** the 3600 s (one hour) bound. A definite violation.
- **Patient 2:** triage at t=10000, antibiotics at t=13000 → 3000-second gap → ultimately satisfies. Between t=10000 and t=13600, however, no definite decision can be made: t=10000 falls within the non-definiteness window w(ψ) = 3600 s of the property, which requires the current timestamp c to reach c ≥ 10000 + 3600 = 13600 before t=10000 becomes a definite time point. Patient 2 exits this window satisfying the property.

## Running InTempo

Place `intempo.jar` in the parent directory and run from `quickstart/`:

```
java -jar ../intempo.jar run \
    --metamodel shs.ecore \
    --mapping shs.e2p \
    --query sepsis-antibiotics-1h.itql \
    --log quickstart.log \
    --answer-set effective \
    --profile shs
```

Swap `--answer-set effective` for `--answer-set plain` to see how the two answer sets differ on this log.

## Output

The query searches for falsifications; each reported line is a violation. The two answer sets agree on patient 1 and disagree on patient 2.

**Plain** (`--answer-set plain`):

```
4600    PMonitoringService#1    [1000,inf)
11800   PMonitoringService#2    [10000,inf)
```

**Effective** (`--answer-set effective`):

```
4600    PMonitoringService#1    [1000,inf)
```

Both answer sets report patient 1 at round 4600: this is the time point the first analysis round occurs, and coincindetanlly the first time point on which the one-hour temporal constraint is falsified. The match is accompanied by the temporal validity [1000, inf], which in this case states that the property is violated from the time point 1000 onward---until the end of the execution. 

The plain answer set additionally reports patient 2 at the round occurring at time point 11800: the prefix seen so far falsifies the query, so the match is emitted immediately. The effective answer set suppresses that line because at round 11800 the start of patient 2's potential match (at time point 10000) is not yet a definite time point: the non-definiteness window w(ψ) = 3600 s requires the current timestamp to reach c ≥ 10000 + 3600 = 13600 before a decision at t=10000 can be definite. By the time t=10000 becomes a definite time point, patient 2 has already received antibiotics, so no falsification match for patient 2 is ever emitted in the effective answer set.

This is the central trade-off: plain reports matches as soon as the observed prefix falsifies the query, at the cost of possible false positives over incomplete traces; effective waits for definiteness and returns only true violations. As a consequence of waiting for definiteness, the effective answer set may report some matches with a delay compared to the plain answer set (see §4.3 of the FAC 2025 article).

## See also

- **Definite answer set** — `--answer-set definite` reports only matches for which a definite answer has been reached, emitting both the definite temporal validity and temporal invalidity intervals `{S:<ranges>, U:<ranges>}`: S shows the definite satisfaction ranges (V^d) and U shows the definite falsification ranges (IV^d), corresponding to the (m, V^d, IV^d) triples of the definite answer set. Unlike effective, definite recomputes all definite spans at every analysis round rather than building incrementally on the plain answer set, and is therefore more expensive over large logs. The definite answer set is meant as a formal means to establish the soundness of the effective answer set (see §4 of the FAC 2025 article). On this log it reports the same matches as effective.
- **Pruning** — The flag `--pruning` enables the deletion of vertices from the runtime model once they are not temporally relevant to matches, i.e., they cannot affect matches any more. Pruning mimics the behavior of other runtime monitoring tools and thus renders InTempo competitive with the state-of-the-art. See `docs/answer-sets-and-performance.md` for the full comparison.
- **Scalability** — the full replication package (Zenodo artifact, `shs.zip`) includes a real hospital log (`real.log`) with 1049 sepsis patient trajectories, plus synthetic logs at 10× and 100× density.
