# SOPs Language Specification

## 1. Introduction

**SOPs** (Standard Operation Procedure standard) is a YAML-based language for defining executable workflows that can be understood equally by humans, applications, and AI. It combines the clarity of OpenAPI-style schemas with the familiar primitives of flowcharts to model actors, actions, data contexts, and step-by-step logic in a single, versioned format.

## 2. Goals

1. **Human Readability**
   Simple YAML syntax; minimal keywords; reserved fields prefixed with `_`.

2. **Machine Interpretability**
   Strict, schema-driven definitions; JSON-Schema validation of inputs, outputs, and process structure.

3. **Extensibility**
   Versioning, namespacing, and `$ref`-style reuse for large enterprises or cross-team sharing.

4. **Interoperability**
   Inspired by OpenAPI to leverage existing tools (validators, code generators, editors).

## 3. File Layout

```text
<root>.sops
  version: <semver>
  namespace: <string>
  extends:
    - ./base.sops

  paths:      # Optional API-style entry points
  concepts:   # Domain extensions (optional)
  <Actor>…    # Actor and process definitions
```

* **version**: language version (e.g. `1.0`)
* **namespace**: logical grouping (e.g. `example.coffee_shop`)
* **extends**: import base schema(s) before parsing

## 4. Core Concepts

### 4.1 Concept

Base type for all definitions. Properties:

* `id` (string)
* `name` (string)
* `description` (string)

### 4.2 Action

Callable contract (atomic or process entry). Fields:

* `_expects` (object) – JSON-Schema of input context.
* `_returns` (object) – JSON-Schema of output context.
* `_call` (array of strings) – `[actor, action, context]` delegate.
* `_step` (array of strings) – `[self, step, context]` internal entry; takes precedence over `_call`.

If neither `_call` nor `_step` is set and the actor defines `_steps`, the action begins at the first key in `_steps`.

### 4.3 Actor

Container for actions and optional workflow steps. Fields:

* `_actions` (map of Action) – public entry points.
* `_steps` (map of Step) – internal workflow steps.
* `_default` (string) – fallback step when a decision flow omits a default.

### 4.4 Context

Runtime data and system state. Fields:

* `_payload` (object) – business data.
* `_requester` (string) – initiator identifier.
* `_operations` (object) – intermediate results by step.
* `_trace` (array of objects) – audit log of executions.
* `_state` (object) – overall state flags.
* `_debug` (object) – diagnostic messages.
* `_meta` (object) – auxiliary metadata.

Fields `_payload` and `_requester` are required.

## 5. Step Primitives

A **Step** extends Concept and must define exactly one of the following reserved keys:

1. **_operation**
   Invoke an action on any actor.

    * `_call` (array)
    * `_next`, `_on_success`, `_on_error` (strings)
    * `_retry` (attempts, delay)
    * `_timeout` (string)

2. **_decision**
   Conditional branching.

    * `_inputs` (map from var to context pointer)
    * `_cases` (array of objects with `_operator`, `_conditions`, `_value`, `_reference`, `_compare`, `_type`, `_next`)
    * `_default` (string)

3. **_transform**
   Data transformation.

    * `_from`, `_op`, `_to`, `_next` (strings)

4. **_terminate**
   End process and return.

    * `_status` (string)
    * `_result` (object)

5. **_fork** / **_merge**
   Parallel branch & join.

    * `_branches` / `_from` (array of step names)
    * `_op`, `_to`, `_next` (strings)

6. **_wait**
   Pause until time/event.

    * `_until`, `_next` (strings)

7. **_noop**
   No operation placeholder.

    * `_next` (string)

## 6. Paths (Optional)

You may declare API-style entry points under the `paths` key:

```yaml
paths:
  /resource:
    action_name:
      _expects: { … }
      _returns: { … }
      _call: [Actor, action, $]
```

This maps HTTP-like routes to SOPs actions for integration with gateways or webhooks.

## 7. Validation Rules

* Every file must declare `version` and `namespace`.
* The base schema (`base.sops`) must be imported via `extends`.
* Actor IDs and step names must be unique within a namespace.
* Steps must define exactly one reserved primitive.
* Actions must supply at least `_call` or `_step`, or rely on `_steps`.
* Decisions must list `_cases` and may specify `_default`.
* Context requires `_payload` and `_requester`.

## 8. Example

A minimal coffee shop entry:

```yaml
version: 1.0
namespace: example.coffee_shop
extends:
  - ./base.sops

paths:
  /order:
    place:
      _expects: { $ref: "#/concepts/OrderContext" }
      _returns: { $ref: "#/concepts/OrderContext" }
      _call: [Cashier, take_order, $]
```

See `examples/coffee-shop.sops` for the full end-to-end workflow.

## 9. Roadmap

* Implement reference CLI for validation and execution.
* Develop editor plugins (VSCode).
* Provide OpenAPI-SOPs converters.
* Build visualization tools.

## 10. Contribution

This specification is production-ready. We welcome:

* Feedback on edge cases or missing primitives
* Pull requests for tooling or examples
* Volunteers to help shape the next milestones

Please open issues or pull requests against this repository.
