# ORVIA Modular Diagnostic Plugin Architecture

**Record ID:** ORVIA-ARCH-005  
**Source model:** Flutter DevTools Extensions  
**Source commit:** `405689bc49f9c7ae0d7ca77b3d9b0f698745989b`  
**Status:** Source-grounded architectural conversion

## Purpose

Use the DevTools Extensions system as a reference for ORVIA **modular diagnostic plugins** so new diagnostic domains can be added without coupling every module to the ORVIA core.

## Source-derived extension model

DevTools Extensions are independent Flutter web applications integrated into the DevTools environment.

The extension configuration specification declares required metadata:

- `name`
- `issueTracker`
- `version`
- `materialIconCodePoint`

and an optional capability field:

- `requiresConnection` (defaults to true)

The `DevToolsExtension` wrapper initializes and exposes managed framework services:

- `extensionManager`
- `serviceManager`
- `dtdManager`

The wrapper owns connection/lifecycle initialization rather than requiring each extension to recreate it.

The extension environment can also be simulated during development, allowing extension behavior to be exercised without embedding it in the full production host.

### Architectural rule extracted

A plugin should declare metadata and requirements, then receive access to host capabilities through stable managed interfaces.

It should **not** reach arbitrarily into host internals.

## ORVIA conversion

### Plugin manifest

```yaml
id:
name:
version:
category:
description:
entrypoint:

requires:
  runtime_capture: false
  dom: false
  cssom: false
  accessibility_tree: false
  screenshot: false
  network_trace: false
  project_files: false

supports:
  offline_evidence: false
  baseline_diff: false
  candidate_diff: false

inputs: []
outputs: []

compatibility:
  orvia_api:
  evidence_schema:

stability:
  status: experimental | beta | stable
```

### Host-managed services

Equivalent ORVIA services should be provided through public interfaces such as:

```
pluginContext.evidence
pluginContext.capture
pluginContext.controls
pluginContext.diff
pluginContext.reporting
pluginContext.storage
pluginContext.projectFiles
pluginContext.events
pluginContext.logging
```

A plugin only receives services permitted by its manifest/capability contract.

### Lifecycle

```
DISCOVER
   ↓
VALIDATE MANIFEST
   ↓
CHECK VERSION COMPATIBILITY
   ↓
CHECK REQUIRED CAPABILITIES
   ↓
INITIALIZE PLUGIN CONTEXT
   ↓
REGISTER CONTROLS / COLLECTORS / ANALYZERS
   ↓
RUN
   ↓
DISPOSE
```

### Plugin responsibilities

A diagnostic plugin may contribute one or more of:

- evidence collectors;
- normalizers;
- measurement models;
- control definitions;
- analyzers;
- comparison strategies;
- visualization panels;
- remediation knowledge.

These should remain explicit registrations rather than implicit side effects.

## Isolation principle

Recommended dependency direction:

```
ORVIA CORE API
      ↑
PLUGIN SDK
      ↑
DIAGNOSTIC PLUGIN
```

Plugins should not import private ORVIA core implementation modules.

## Capability-gated services

The DevTools model's `requiresConnection` is a compact example of capability declaration. ORVIA should generalize it.

For example:

```yaml
requires:
  runtime_capture: true
  accessibility_tree: true
```

The host resolves those requirements through the Applicability Envelope before activation.

## Simulated plugin environment

Inspired by the DevTools simulated extension environment, ORVIA should provide a plugin harness with:

- synthetic evidence fixtures;
- mocked host services;
- simulated capability availability;
- event injection;
- deterministic logging;
- manifest validation;
- schema validation;
- baseline/candidate fixture pairs.

This allows plugin development without repeatedly running full website captures.

## Validation gates

Before a plugin is accepted:

```
manifest valid
AND API version compatible
AND declared dependencies available
AND output schema valid
AND deterministic fixture tests pass
AND no private-core dependency
```

## Example ORVIA plugin families

```
plugins/
  accessibility-semantics/
  motion-runtime/
  responsive-layout/
  typography-metrics/
  contrast-analysis/
  network-budget/
  design-diff/
  focus-navigation/
```

These are proposed ORVIA modules, not DevTools extension types.

## Source files

- `packages/devtools_extensions/README.md`
- `packages/devtools_extensions/extension_config_spec.md`
- `packages/devtools_extensions/lib/src/template/devtools_extension.dart`

## Boundary

Extension metadata, connection requirement, managed extension/service/DTD managers, wrapper-owned lifecycle, and simulated development environment are source-derived. ORVIA plugin services, permissions, manifests, module families, and validation gates are architectural synthesis.
