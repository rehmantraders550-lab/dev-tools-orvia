# Diagnostic Tool Architecture Record

**Record ID:** ORVIA-DTAR-001  
**Status:** Source-grounded extraction + ORVIA translation layer  
**Source project:** Flutter DevTools (`flutter/devtools`)  
**Source branch:** `master` at extraction time  
**Destination:** ORVIA development knowledge base  
**Extraction date:** 2026-09-25

---

## 1. Purpose

This record extracts reusable diagnostic-tool architecture patterns from Flutter DevTools for application inside ORVIA.

The objective is **not** to reproduce Flutter DevTools or copy its product-specific implementation. The objective is to preserve the architectural ideas that make a mature diagnostic system reliable:

- capability-gated tools;
- explicit operating modes;
- evidence acquisition separated from interpretation;
- offline analysis where possible;
- domain-specific measurement panes;
- structured comparison and diagnosis;
- extension/plugin boundaries;
- accessibility inspection;
- strong engineering constraints around dependencies, reuse, testing, and maintainability.

Anything under **Source-derived architecture** is grounded in the referenced Flutter DevTools repository files. Anything under **ORVIA translation** is an architectural synthesis for this project rather than a claim about Flutter DevTools itself.

---

## 2. Canonical diagnostic surfaces

Flutter DevTools defines top-level diagnostic screens through centralized metadata rather than treating each screen as an isolated route.

Observed tool metadata includes:

| Tool | Key capability requirements / modes |
|---|---|
| Home | no connection required; web-server-device support |
| Flutter Inspector | Flutter app; debug build; web-server-device support |
| Performance | no connection required; offline data supported |
| CPU Profiler | Dart VM; no connection required; offline data supported |
| Memory | Dart VM; no connection required; offline data supported |
| Debugger | debug build |
| Network | Dart VM; no connection required; offline-data capability controlled by feature configuration |
| Logging | web-server-device support |
| Accessibility | Flutter app |
| Provider | requires `package:provider/`; debug build |
| App Size | no connection required; Dart VM |
| Deep Links | no connection required; Dart VM; Flutter app |
| VM Tools | advanced developer mode |
| DTD Tools | advanced developer mode; no connection required |
| Simple | minimal metadata entry |

The important reusable pattern is not the individual Flutter tool list. It is the existence of a **formal applicability contract** for every diagnostic surface.

---

## 3. Source-derived architecture

### 3.1 Capability metadata as an applicability envelope

The `ScreenMetaData` / `Screen` system models explicit eligibility dimensions including:

- `requiresConnection`
- `requiresDartVm`
- `requiresFlutter`
- `requiresDebugBuild`
- `requiresAdvancedDeveloperMode`
- `supportsWebServerDevice`
- `worksWithOfflineData`
- `requiresLibrary`

The runtime then checks these constraints before showing a screen.

This creates a general architectural principle:

> A diagnostic tool should declare the environment in which its results are valid before the tool is allowed to run or present itself as applicable.

This avoids false diagnostics produced by applying a measurement method outside its valid operating conditions.

### 3.2 Three operating states

The DevTools `Screen` abstraction explicitly supports three broad modes:

1. connected to an application;
2. working from offline diagnostic data;
3. not connected to an application.

A screen may support any combination of those modes.

This separation is highly reusable. It implies that **data acquisition and analysis do not need to be permanently coupled**.

### 3.3 Eligibility and disabled-reason logic

The screen-selection layer evaluates why a diagnostic surface can or cannot be shown. Examples include:

- offline data not supported;
- advanced developer mode required;
- service not ready;
- required Dart library unavailable;
- Dart VM required;
- Flutter runtime required;
- debug build required.

Reusable principle:

> Ineligibility should be represented as a machine-readable reason, not merely as hidden UI behavior.

That reason can later power UI explanations, audit logs, AI interpretation, fallbacks, and automated test assertions.

---

## 4. ORVIA applicability envelope

ORVIA should generalize the same concept into a domain-independent diagnostic contract.

Recommended control schema:

```yaml
control:
  id:
  domain:
  applicability:
    requires_runtime_capture:
    requires_dom:
    requires_cssom:
    requires_visual_capture:
    requires_interaction_trace:
    requires_network_trace:
    requires_accessibility_tree:
    supports_offline_evidence:
    supported_viewports:
    supported_input_modes:
  inputs:
  measurements:
  thresholds:
  classifications:
  evidence:
  remediation:
  confidence:
```

The central rule is:

```
CAN THIS CONTROL VALIDLY RUN?
        ↓
YES → acquire / load evidence
NO  → return explicit applicability reason
```

This should become a first-class ORVIA behavior, not an implementation detail.

---

## 5. Evidence pipeline

The reusable diagnostic flow extracted from DevTools is:

```
RAW EVENT / STATE
        ↓
MEASUREMENT
        ↓
NORMALIZATION
        ↓
DOMAIN CLASSIFICATION
        ↓
THRESHOLD / RULE EVALUATION
        ↓
ROOT-CAUSE INSPECTION
        ↓
EVIDENCE PACKAGE
        ↓
INTERPRETATION / REMEDIATION
```

For ORVIA, this should remain deliberately split into separate responsibilities.

### Acquisition layer

Captures facts without deciding whether they are good or bad.

Possible web evidence classes:

- DOM snapshot;
- CSS/computed-style snapshot;
- accessibility tree;
- viewport dimensions;
- rendered screenshots;
- frame/runtime timings;
- resource timings;
- request/response metadata;
- layout-shift data;
- interaction traces;
- scroll traces;
- animation timing;
- font metrics;
- element geometry;
- contrast values;
- focus order.

### Normalization layer

Transforms heterogeneous capture formats into stable internal schemas.

### Control layer

Evaluates categorical rules and numerical envelopes.

### Diagnostic layer

Connects violations to evidence and plausible causes.

### Interpretation layer

Produces explanations for humans or downstream AI without mutating the underlying measurements.

---

## 6. Performance-tool pattern

The Flutter DevTools performance area is decomposed into distinct panes including:

- controls;
- Flutter frames;
- frame analysis;
- rebuild statistics;
- timeline events.

The reusable lesson is architectural decomposition.

A mature performance diagnostic system should not expose only one global "performance score." It should separate:

```
CAPTURE CONTROL
    ↓
FRAME / EVENT DATA
    ↓
ANALYSIS
    ↓
SECONDARY CAUSAL SIGNALS
    ↓
TIMELINE / EVIDENCE VIEW
```

### ORVIA translation

For website/UI analysis, this suggests modules such as:

```
Runtime Capture
    ↓
Frame & Long-Task Measurements
    ↓
Interaction / Animation Analysis
    ↓
Layout / Paint / Reflow Evidence
    ↓
Root-Cause Correlation
```

Numerical envelopes must be stored separately from raw measurement. This allows threshold policies to evolve without recollecting evidence.

---

## 7. Offline diagnostic architecture

Several DevTools surfaces explicitly support offline data. The architectural consequence is important:

```
LIVE SYSTEM
   ↓ capture
DIAGNOSTIC ARTIFACT
   ↓ normalize
ANALYSIS ENGINE
   ↓
REPORT / UI / AI
```

ORVIA should therefore treat every meaningful capture as a reusable artifact rather than a transient UI state.

Recommended properties for evidence artifacts:

- schema version;
- capture timestamp;
- source URL / page identifier;
- viewport;
- browser/runtime context;
- capture capabilities;
- tool/control versions;
- raw measurements;
- derived measurements;
- evidence references;
- applicability state;
- integrity hash when practical.

This enables deterministic re-analysis and baseline comparisons.

---

## 8. Baseline-vs-candidate architecture

The App Size class of diagnostic thinking demonstrates the value of inspecting a current artifact as data rather than only observing a live process.

ORVIA should generalize this into:

```
BASELINE EVIDENCE
       ↘
        NORMALIZED DIFF → CONTROL EVALUATION → EXPLANATION
       ↗
CANDIDATE EVIDENCE
```

Useful comparison domains may include:

- element count;
- DOM depth;
- CSS complexity;
- asset weight;
- typography metrics;
- layout density;
- contrast changes;
- accessible-name changes;
- animation durations;
- motion displacement;
- frame behavior;
- interaction latency;
- focus-order differences;
- responsive breakpoint behavior;
- content hierarchy;
- visual rhythm.

These are ORVIA design targets, not Flutter DevTools metrics.

---

## 9. Accessibility architecture

The current DevTools accessibility controller exposes a meaningful split between **implemented inspection** and **partially implemented environment overrides**.

Source-grounded observations:

- brightness override is wired through a service extension;
- accessibility state includes text scale, bold text, screen reader, and high-contrast controls;
- several of those override handlers remain TODOs;
- semantics inspection is implemented;
- the controller can enable semantics, obtain a semantics tree, parse nodes, and arrange child order using traversal/hit-test information.

Important engineering lesson:

> A diagnostic product must distinguish an inspector that observes real system state from a simulator that claims to modify accessibility conditions.

ORVIA must not label a control "supported" merely because its UI exists.

Recommended capability state:

```yaml
capability:
  status: implemented | partial | experimental | unavailable
  inspection_supported: true|false
  simulation_supported: true|false
  validation_supported: true|false
  evidence_type:
  limitations:
```

For ORVIA accessibility work, the semantic/accessibility tree should be treated as evidence, not inferred solely from rendered pixels.

---

## 10. Extension architecture

Flutter DevTools Extensions demonstrates a strong model for third-party / modular diagnostic tooling.

Source-derived properties:

- extension UI is a Flutter web application;
- it is embedded into DevTools;
- extensions may work with or without a running application;
- extensions may act as companion tools for packages or as standalone tools;
- extensions may interact with project files through tooling integration;
- extension configuration declares metadata such as name, issue tracker, version, icon, and whether a live connection is required;
- the extension framework exposes managers for the DevTools framework, VM service, and Dart Tooling Daemon when available;
- a simulated environment is provided for extension development;
- build-and-copy and validation commands enforce packaging expectations.

### ORVIA translation

ORVIA diagnostic modules should be independently registerable.

Suggested manifest:

```yaml
id: motion-runtime
name: Motion Runtime Diagnostics
version: 0.1.0
category: runtime
requires:
  runtime_capture: true
  screenshot: false
  accessibility_tree: false
supports:
  offline_evidence: true
inputs:
  - frame_trace
  - interaction_trace
outputs:
  - measurements
  - violations
  - evidence_links
  - remediation_context
```

A module should not need privileged access to the entire ORVIA internals. It should consume stable service contracts.

---

## 11. Simulated diagnostic environments

DevTools Extensions explicitly supports a simulated environment for development.

This suggests an ORVIA testing principle:

> Diagnostic modules should be testable against synthetic evidence without requiring a real website capture every time.

ORVIA should maintain fixtures for:

- ideal cases;
- threshold-edge cases;
- clear failures;
- missing capabilities;
- malformed evidence;
- responsive variants;
- reduced-motion variants;
- keyboard-only interaction;
- touch interaction;
- screen-reader / semantics cases where evidence is available.

The same fixture should produce deterministic classifications across builds unless the rule version changes.

---

## 12. Engineering governance extracted from DevTools

The repository's AI/contributor guidance establishes several useful maintainability principles.

Source-derived examples:

- dependency boundaries are explicit;
- published packages must not silently depend on unpublished internal packages;
- public members should be documented;
- duplicated implementations should be identified;
- meaningful naming is expected;
- code should be formatted and analysis-clean;
- reusable shared components/utilities are preferred;
- UI magic numbers should be avoided in favor of named constants;
- existing themes and shared styles should be reused;
- smaller composable widgets are preferred to excessively long build functions;
- tests are required using the appropriate package test runner.

### ORVIA translation

Adopt equivalent constraints:

```
collectors must not depend on report UI
normalizers must not depend on AI interpretation
controls must not mutate source evidence
reports must not alter measurement results
plugins must depend on public service contracts
thresholds must be named/versioned data
shared schemas must remain UI-independent
```

---

## 13. Proposed ORVIA diagnostic module lifecycle

```
REGISTER
   ↓
CHECK APPLICABILITY
   ↓
ACQUIRE OR LOAD EVIDENCE
   ↓
VALIDATE EVIDENCE SCHEMA
   ↓
NORMALIZE
   ↓
MEASURE
   ↓
EVALUATE CONTROL ENVELOPES
   ↓
CLASSIFY
   ↓
ATTACH EVIDENCE
   ↓
CORRELATE POSSIBLE CAUSES
   ↓
GENERATE REMEDIATION CONTEXT
   ↓
REPORT / COMPARE / AI INTERPRET
```

Every stage should be independently testable.

---

## 14. Diagnostic result contract

Recommended normalized output:

```json
{
  "control_id": "motion.duration.primary",
  "control_version": "1.0.0",
  "applicability": {
    "status": "applicable",
    "reason": null
  },
  "measurement": {
    "value": null,
    "unit": null
  },
  "envelope": {
    "type": "range",
    "min": null,
    "max": null
  },
  "classification": "pass|warning|fail|informational|not_applicable",
  "confidence": null,
  "evidence": [],
  "causes": [],
  "remediation_context": [],
  "source_capture_id": null
}
```

This contract keeps measurement, threshold, classification, and remediation distinct.

---

## 15. Architectural invariants

ORVIA should preserve the following invariants:

1. **No diagnosis without evidence.**
2. **No control runs outside declared applicability.**
3. **Raw evidence is immutable after capture.**
4. **Derived measurements identify their source evidence.**
5. **Thresholds are versioned independently from measurements.**
6. **A failure classification never replaces the underlying measured value.**
7. **Unavailable capabilities return explicit reasons.**
8. **Offline re-analysis must be possible whenever the diagnostic domain allows it.**
9. **Baseline/candidate comparison uses normalized schemas, not presentation-layer values.**
10. **AI interpretation is downstream of deterministic evidence and rules.**
11. **Partial/experimental capabilities are visibly identified.**
12. **Plugins/modules use defined interfaces rather than internal cross-dependencies.**

---

## 16. ORVIA target architecture

```
                    ORVIA DIAGNOSTIC INTELLIGENCE
                               │
                     CAPABILITY REGISTRY
                               │
                         APPLICABILITY
                               │
             ┌─────────────────┼──────────────────┐
             ↓                 ↓                  ↓
         STRUCTURE          RUNTIME             VISUAL
             │                 │                  │
             ├──────── ACCESSIBILITY ─────────────┤
             │                 │                  │
             └─────────────────┼──────────────────┘
                               ↓
                        EVIDENCE ARTIFACTS
                               ↓
                         NORMALIZATION
                               ↓
                          MEASUREMENT
                               ↓
                    NUMERICAL / CATEGORICAL
                         CONTROL ENVELOPES
                               ↓
                        CLASSIFICATION
                               ↓
                        CAUSE CORRELATION
                               ↓
                   BASELINE ↔ CANDIDATE DIFF
                               ↓
                       AI INTERPRETATION
                               ↓
                       REPORT / REMEDIATION
```

---

## 17. What should be extracted next

This record captures the architecture-level patterns. It does **not** yet exhaustively extract implementation details for every Flutter DevTools screen.

High-value follow-on records should separately study:

- Performance data models and frame analysis;
- CPU profile transformation;
- Memory evidence and leak-oriented workflows;
- Network capture / HAR structures;
- App Size comparison models;
- Inspector tree/property architecture;
- accessibility semantics-tree representation;
- DTD service architecture;
- DevTools Extensions configuration and service interfaces;
- offline-data serialization formats;
- screen eligibility / disabled-reason model.

These should become independent records so ORVIA can reuse the concepts without creating one monolithic diagnostic subsystem.

---

## 18. Source references

Primary source paths used for this record:

- `flutter/devtools/packages/devtools_app/lib/src/shared/framework/screen.dart`
- `flutter/devtools/packages/devtools_app/lib/src/screens/accessibility/accessibility_controller.dart`
- `flutter/devtools/packages/devtools_extensions/README.md`
- `flutter/devtools/AGENTS.md`

Repository:

- https://github.com/flutter/devtools

This record intentionally separates direct source observations from ORVIA-specific architectural synthesis.
