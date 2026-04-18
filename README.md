# InTempo

InTempo is a runtime monitoring prototype for systems whose states are amenable to a graph-based representation.

The prototype was developed at Hasso Plattner Institute in the context of this PhD thesis [4], and it is built on the Eclipse Modeling Framework (EMF) [11]. InTempo requires Java 17+ and is available as a CLI (see [cli/intempo.jar](intempo.jar)) and as an EMF plugin. The CLI was developed based on [https://picocli.info/](picocli).

## In a Nutshell

InTempo captures properties of interest as *temporal graph queries*, comprising a graph pattern whose matches satisfy a temporal graph logic formula; the system's evolving state is captured by transforming a sequence of events into a graph that encodes a trace of timestamped snapshots. InTempo evaluates queries over such graphs in a change-driven and incremental manner, and pairs each query match with its *temporal validity*, i.e., the period for which the match satisfies the property.

The approach targets modern safety-critical systems such as smart healthcare and autonomous transportation, which are human-in-the-loop, operate in dynamic environments, and are real-time, i.e., their safe operation depends on the timing of their actions. 

## InTempo by Example: A Smart Hospital System for the Treatment of Sepsis

A Smart Hospital System (SHS) is a smart medical environment that supports clinicians in medical treatments by automating tasks via smart devices. The example is based on an SHS for the treatment of sepsis, where medical actions such as triage or the administration of antibiotics are represented by events emitted by the Hospital Information System, sensors, or smart pumps.

A basic instruction in a medical guideline on the treatment of sepsis [9,10] reads:

> Within one hour of a sepsis triage in the Emergency Room (ER), intravenous (IV) antibiotics should be administered to the diagnosed patient. 

The input language of InTempo captures this instruction as a temporal graph query: The *ER Sepsis Triage* and *IV Antibiotics* events are captured as graph patterns; the query comprises the ER Sepsis Triage pattern for a patient and a formula that states that matches for the ER Sepsis Triage pattern must be followed by matches for the IV Antibiotics pattern (for the same patient) within 3600 seconds. 

The graph-based system description of the SHS is based on EMF and is in [`quickstart/shs.ecore`](quickstart/shs.ecore). InTempo introduces two Domain-specific Languages (DSLs) for the formulation of queries and mappings between events and graph modifications, respectively. The query that encodes the property from above is in [`quickstart/sepsis-antibiotics-1h.itql`](quickstart/sepsis-antibiotics-1h.itql); the mapping from events to model modifications based on shs.ecore for this example is in [`quickstart/shs.e2p`](quickstart/shs.e2p).

The quickstart.log contains four events covering two patient trajectories: patient 1 falsifies the 1-hour constraint; patient 2 satisfies the constraint. The InTempo JAR can monitor the execution in this log using the following command:

```
java -jar intempo.jar run \
    --metamodel quickstart/shs.ecore \
    --mapping quickstart/shs.e2p \
    --query quickstart/sepsis-antibiotics-1h.itql \
    --log quickstart/quickstart.log \
    --answer-set effective \
    --profile shs
```

 Using a query searching for *falsifications*, i.e., patients who have not been treated timely, InTempo returns the patient 1 trajectory. See [`quickstart/README.md`](quickstart/README.md) for a detailed explanation.

## Technical documentation

Answer sets of InTempo can be parameterized to target different application scenarios. Moreover, InTempo can delete vertices from the state representation that are not temporally relevant to the query evaluation, thus performing efficiently even at the face of increasing execution sizes. InTempo introduces two domain-specific languages: ITQL for temporal graph queries and E2P for mappings between events and graph modifications. Both are implemented with [Xtext](https://eclipse.dev/Xtext/). For more technical information, see:

- [`docs/cli-reference.md`](docs/cli-reference.md): full parameter reference, output format, emission modes, and profiles.
- [`docs/architecture.md`](docs/architecture.md): brief look "under the hood:" the RTMH, the TGDN, and the change-driven evaluation pipeline.
- [`docs/answer-sets-and-performance.md`](docs/answer-sets-and-performance.md): the four types of answer sets.
- [`docs/itql-grammar.md`](docs/itql-grammar.md): the ITQL grammar.
- [`docs/e2p-grammar.md`](docs/e2p-grammar.md): the E2P grammar.

## For MDE researchers

InTempo is built on the Eclipse Modeling Framework (EMF) and is also available as an EMF plugin. The EMF plugin version used in [1] is available at the [HPI MDELab project page](https://www.hpi.uni-potsdam.de/giese/public/mdelab/mdelab-projects/intempo/) and, for demonstration, as part of the article supplement on Zenodo [2] (`fac24.ova`).

## Assumptions and Limitations

InTempo is intended for complex systems with large, dynamic structures, i.e., large event sequences comprising heavily interrelated data. In small systems or over simple structures, the overhead of the approach would probably outweigh its advantages. 

Additionally, InTempo, as well as similar online runtime monitoring tools, rely on the existence of a stream of (very) frequently occurring events that affect only parts of a large representation of an execution: if events do not occur frequently enough, then large delays could be observed in the returning of answers; on the other hand, if events always affect a large part of the state, the effect of change-driven, incremental evaluation would be mitigated.

InTempo relies on an available Ecore file; even when present, i.e., in systems developed based on EMF, InTempo requires changes to the Ecore, i.e., the introduction of a creation and deletion timestamp for all vertices. Thereby, the approach is not applicable to model-based systems for which such a change is not possible. In the future, we plan to render InTempo capable of adding timestamps to elements of a model seamlessly.

## Papers and artifact

- [1] **Foundations for Change-driven Query-based Runtime Monitoring of Temporal Properties.** L. Sakizloglou, H. Giese, L. Lambers. *Formal Aspects of Computing*, 2025. [DOI](https://doi.org/10.1145/3744714)
- [2] **Supplement to "Foundations for Change-driven Query-based Runtime Monitoring of Temporal Properties" (Journal: Formal Aspects of Computing, 2025).** L. Sakizloglou, H. Giese, L. Lambers. Zenodo. [DOI](https://doi.org/10.5281/zenodo.15697935).
- [3] **Foundations for Query-based Runtime Monitoring of Temporal Properties over Runtime Models.** L. Sakizloglou, H. Giese, L. Lambers. FASE 2024. [DOI](https://doi.org/10.1007/978-3-031-57259-3_2)
- [4] **Evaluating Temporal Queries over History-aware Architectural Runtime Models.** L. Sakizloglou Doctoral thesis. Potsdam University. [DOI](https://doi.org/10.25932/publishup-60439)
- [5] **Incremental execution of temporal graph queries over runtime models with history and its applications.** L. Sakizloglou, S. Ghahremani, M. Barkowsky, H. Giese. *Software and Systems Modeling*, 2022. [DOI](https://doi.org/10.1007/s10270-021-00950-6)
- [6] **Keeping Pace with the History of Evolving Runtime Models.** L. Sakizloglou, M. Barkowsky, H. Giese. FASE 2021. [DOI](https://link.springer.com/chapter/10.1007/978-3-030-71500-7_13?)
- [7] **A scalable querying scheme for memory-efficient runtime models with history.** L. Sakizloglou, S. Ghahremani, M. Barkowsky, H. Giese. MoDELS 2020. [DOI](https://doi.org/10.1145/3365438.3410961)
- [8] **Towards Highly Scalable Runtime Models with History.** L. Sakizloglou, S. Ghahremani, T. Brand, M. Barkowsky, H. Giese. SEAMS 2019. [DOI](https://doi.org/10.1145/3387939.3388614) 

## References

- [9] **Analyzing the Trajectories of Patients with Sepsis using Process Mining.** Mannhardt et al. RADAR+EMISA@CAiSE, 2017. [DOI](http://ceur-ws.org/Vol-1859/bpmds-08-paper.pdf)
- [10] **Surviving Sepsis Campaign: International Guidelines for Management of Sepsis and Septic Shock: 2016.** Rhodes et al. Intensive Care Medicine, 2017. [DOI](https://doi.org/10.1007/s00134-017-4683-6)
- [11] **Eclipse Modeling Framework (EMF).** [Eclipse Foundation](https://eclipse.dev/emf/)

## Status and license

Active research prototype. InTempo is developed based on the Eclipse Modeling Framework and Xtext and licensed under the [Eclipse Public License v1.0](LICENSE.txt). Contributions, issues, and questions welcome.

InTempo is bundled with Google Guava, Google Guice, and log4j, which are licensed under the [APACHE 2.0 license](LICENSE-3rd-PARTY.txt).
