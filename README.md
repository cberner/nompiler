# Nompiler

**Nompiler** (**Nom**), short for **Neural Compiler**, is a coding-assistant workflow orchestrator. It turns
high-level product intent into working software through a structured multi-agent process.

The user describes what they want in a human-authored Charter. Nom generates the architecture and task plan, implements
the software, tests and reviews the result, fixes problems, and iterates toward a stable release.

```text
Charter  ->  Design  ->  Code  ->  Release
human        generated   generated  stabilized
source       IR          software   output
```

Nom is intended to keep people focused on product goals and important decisions rather than implementation
micromanagement. Generated artifacts remain available for inspection, debugging, and auditing. Nom is delivered as a
single executable named `nom`.

## Status

Nom is currently in the pre-implementation bootstrap stage. This repository contains the project Charter and the
instructions for producing the first self-developing version of Nom.

## Project Documents

- `charter.md`: durable product intent and long-term behavior.
- `ui.md`: durable product requirements for the Nom user interface.
- `bootstrap.md`: short-term scope, acceptance criteria, and prompts for the initial bootstrap.
- `decisions/`: temporary human decisions that have not yet been incorporated into the canonical documents.
- `AGENTS.md`: instructions for agents working in this repository.

Agents should begin with `AGENTS.md`, then read `charter.md`, `ui.md`, and `bootstrap.md`. This README is an overview,
not a normative specification.

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as
defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.
