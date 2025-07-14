# SOPs Language (Planning Phase)

> You know how hard it is to make humans, businesses, machines, and AIs speak all the same language?  
> SOPs brings that common language to the table — simple, clear, and ready to use, no matter who or what you are.

## Background and Motivation

Defining clear, executable procedures remains challenging across organizations:

* **Humans** write SOPs in free-form text or flowcharts.
* **Businesses** use multiple siloed tools and formats.
* **Machines** and **AIs** require strict, unambiguous schemas to automate or reason.

**SOPs** (Standard Operation Procedure standard) bridges these gaps with a simple, YAML-based DSL. Anyone—managers, developers, RPA bots, or LLMs—can read, validate, and execute the same SOPs file.

## OpenAPI Inspiration

SOPs draws on the clarity and tooling ecosystem of OpenAPI:

* **Versioned, namespaced definitions** for reusability and import/extend patterns.
* **Schema-based validation** of inputs and outputs.
* **Consistent referencing** via `$ref`-style pointers.

By leveraging these proven patterns, SOPs benefits from existing validators, code generators, and editor support.

## Core Design Decisions

1. **YAML Foundation**
   Leverage human familiarity and existing tooling; avoid new syntax.

2. **Minimal Concept Base**
   Every entity extends a `Concept` with only:

    * `id` (machine identifier)
    * `name` (human label)
    * `description`

3. **Actor-Centric Model**

    * **Actor** containers define `_actions` (contracts) and optional `_steps` (internal workflows).
    * Each action declares `_expects`, `_returns`, and either `_call` (external delegation) or `_step` (internal entry).

4. **Reserved Keys with `_` Prefix**
   All engine/system fields begin with `_` to avoid business-data collisions.

5. **Flowchart-Inspired Step Primitives**
   Each **Step** uses exactly one of:

    * `_operation`
    * `_decision`
    * `_transform`
    * `_terminate`
    * `_fork` / `_merge`
    * `_wait`
    * `_noop`

## File Structure

Place the base schema at the project root:

```
base.sops
```

Domain-specific files extend this schema:

```yaml
version: 1.0
namespace: example.my_domain
extends:
  - ./base.sops

concepts:
  # Domain extensions

Actor:
  # _actions, optional _steps, _default
```

View the foundation here: [base.sops](./base.sops)

## Example SOPs File
View a sample SOPs file that defines a simple coffee shop workflow: [coffee-shop.sops](./coffee-shop.sops)


## Roadmap & Next Steps

* **Prototype Workflows**
  Build real-world examples (e.g., coffee shop, support ticket).

* **Validation & Interpreter**
  Create a CLI and library to validate and execute SOPs files.

* **Editor Integration**
  Develop VSCode support with autocomplete and linting.

* **OpenAPI Alignment**
  Offer converter tools to generate OpenAPI fragments for SOPs actions.

* **Visualization**
  Build a graphical flowchart renderer from SOPs definitions.

## Join Us

This project is in its early planning phase. We welcome volunteers to help with:

* Schema refinement and edge-case validation
* Prototype implementation (CLI, interpreter)
* Editor integration and documentation
* Community outreach and examples

If you’re interested in shaping the future of a universal process language, please get in touch or open an issue. Let’s make SOPs the common language for people, businesses, and machines!
