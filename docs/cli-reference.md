# CLI Reference

`intempo.jar` is a command-line interface for InTempo. Run `java -jar intempo.jar run --help` for a flag listing.

## Invocation

```
java -jar intempo.jar run [OPTIONS]
```

All flags below are passed to the `run` subcommand.

## Required flags

| Flag | Description |
|---|---|
| `--metamodel <path.ecore>` | Path to the domain Ecore metamodel file. |
| `--mapping <path.e2p>` | Path to event-to-pattern mapping file. |
| `--query <path.itql>` | Path to an `.itql` file containing a metric-temporal graph condition (MTGC). |
| `--log <path.log>` | Path to event log file. |
| `--answer-set <type>` | Answer set type: `plain`, `effective`, or `definite`. |

## Optional flags

| Flag | Default | Description |
|---|---|---|
| `--emit <mode>` | `deltas` | Match emission mode; see Emission modes below. |
| `--profile <name>` | `custom` | Experiment profile: `shs`, `ldbc`, or `custom`; see Profiles below. |
| `--analysis-interval <ms>` | `3600000` | Analysis interval in milliseconds. |
| `--metric <type>` | `none` | Metric to log per analysis round: `time` (ms), `mem` (MB), or `none`. |
| `--matches-output <path>` | stdout | Match stream output path. |
| `--metric-output <path>` | stderr | Metric stream output path. Only written when `--metric` is not `none`. |
| `--pruning` | `false` | Enable model pruning. |
| `--strategy <type>` | auto-selected | Search-plan strategy: `empty-start` or `large-start`. Auto-selected from `--start-model-size` if omitted. |
| `--start-model-size <N>` | `0` | Number of log events (times 10^5) to replay into the model before registering queries, building a start model of the given size. |
| `--time-point <long>` | — | Time point for definite-answer-set evaluation. Only valid with `--answer-set definite`; if omitted, current log timestamp is used. |
| `--skip-events-matching <regex>` | — | Skip events whose first field matches this regex. |
| `--stop-at-timestamp <long>` | — | Stop at the first event whose timestamp is `>=` this value. |
| `--jvm-info` | `false` | Print JVM arguments and GC configuration at startup. |
| `--verbose` | `false` | Print resolved parameters and metamodel registrations to stderr at startup. |
| `--help` | — | Show help message. |
| `--version` | — | Show version. |

## Outputs

InTempo produces two output streams and optional diagnostics.

### Match stream (primary)

Written to stdout by default; redirect with `--matches-output <path>`.

```
<round-timestamp>\t<match-key>\t<intervals-or-SAT>
```

- `<round-timestamp>`: the analysis round at which the match was emitted. Note: rounds start after the first event in the log file.
- `<match-key>`: comma-separated, sorted identifiers of the vertices satisfying or falsifying the property (e.g., `Service#svc-17,Probe#p-42`).
- `<intervals>`: for `plain` and `effective` answer sets, the temporal validity intervals.
- `{S:<ranges>, U:<ranges>}`: for `definite` answer-set matches: S shows the definite satisfaction ranges (V^d), U shows the definite falsification ranges (IV^d).

### Metric stream (secondary)

Written to stderr by default; redirect with `--metric-output <path>`. Only produced when `--metric` is `time` or `mem`.

### Diagnostics

`--verbose`, `--jvm-info`, and startup banners are always written to stderr so they never mix with the match stream on stdout.

## Emission modes

Controlled via `--emit`:

| Mode | Behaviour |
|---|---|
| `deltas` (default) | A given match-key is emitted **once**, the first round it appears. Best for a clean, line-per-match log. |
| `snapshots` | **Every** currently active match is re-emitted in **every** round. Best for offline analysis. |

With `--pruning` enabled, vertices are deleted from the runtime model once they can no longer affect query matches, so a match involving a pruned vertex disappears from subsequent snapshots. Delta mode is pruning-safe; late snapshots under pruning are lossy by design.

## Profiles

Profiles refer to the benchmarks used in the performance evaluation of InTempo in the [FAC 2025 paper](https://doi.org/10.1145/3744714). The two profiles configure the flags that were used in these experiments. Any explicit flag overrides the profile's default, but also diverges from the experimental setting.

| Profile | `--analysis-interval` | `--strategy` | `--stop-at-timestamp` |
|---|---|---|---|
| `shs` | 3600000 (1 hour) | `empty-start` | — |
| `ldbc` | 86400000 (1 day) | `large-start` | 1356994800000 |
| `custom` | 3600000 (1 hour) | auto-selected from `--start-model-size` | — |

`shs` is appropriate for the smart healthcare scenario: the model starts empty and events populate it from scratch. `ldbc` is appropriate for the social network benchmark: the model is pre-populated via `--start-model-size` before queries are registered, and `large-start` strategy exploits runtime model metadata for search plan decisions.

## Strategy

- **`empty-start`**: static cheapest-first search plan. Optimal when the model starts empty and there is no runtime metadata to exploit.
- **`large-start`**: hybrid cost-model strategy that leverages runtime model metadata. Optimal when the model is pre-populated via `--start-model-size`.
- Omitting `--strategy` selects `large-start` if `--start-model-size > 0`, otherwise `empty-start`.
