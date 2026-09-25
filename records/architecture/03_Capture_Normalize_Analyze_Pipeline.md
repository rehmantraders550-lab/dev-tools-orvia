# ORVIA Capture → Normalize → Analyze Pipeline

**Record ID:** ORVIA-ARCH-003  
**Source model:** Flutter DevTools offline diagnostic-data architecture  
**Source commit:** `405689bc49f9c7ae0d7ca77b3d9b0f698745989b`  
**Status:** Source-grounded architectural conversion

## Purpose

Convert DevTools offline-data support into an ORVIA architecture where evidence can be captured once, serialized, reloaded, normalized, re-analyzed, compared, and inspected independently of the live website session.

## Source-derived architecture

DevTools has a global `OfflineDataController` and a reusable `OfflineScreenControllerMixin<T>`.

The global controller tracks:

- whether offline data is being shown;
- the imported raw JSON;
- the previously connected application;
- whether offline data exists for a given screen.

A screen that supports offline data must explicitly declare that capability.

Each supporting screen controller is responsible for:

1. preparing serializable screen data;
2. exporting it;
3. parsing imported JSON into its own typed model;
4. deciding whether imported data is usable;
5. restoring its own models/notifiers for offline viewing.

`OfflineScreenData` binds exported data to a screen identifier and requires JSON-serializable primitive data.

The source also preserves live and offline concerns separately: an offline controller instance can operate while another controller instance remains associated with the live connection.

## Architectural rule extracted

```
LIVE SYSTEM
    ↓
CAPTURE
    ↓
SERIALIZABLE DIAGNOSTIC ARTIFACT
    ↓
IMPORT / LOAD
    ↓
DOMAIN PARSER
    ↓
DOMAIN MODEL
    ↓
ANALYSIS
    ↓
VIEW / REPORT
```

The diagnostic artifact becomes an interface boundary between acquisition and analysis.

## ORVIA conversion

### Stage 1 — Capture

Capture source evidence without interpretation.

Each capture should declare:

```yaml
capture:
  capture_id:
  schema_version:
  created_at:
  source:
    url:
    page_id:
  environment:
    viewport:
    browser:
    input_mode:
    pixel_ratio:
    reduced_motion:
  capabilities:
  collectors:
  raw_evidence:
```

### Stage 2 — Validate

Before normalization:

- validate schema version;
- validate required fields;
- verify capture completeness;
- verify evidence references;
- identify unsupported or missing collectors;
- preserve unknown fields where possible.

### Stage 3 — Normalize

Convert collector-specific formats into canonical ORVIA evidence objects.

```
collector output
      ↓
domain parser
      ↓
canonical evidence schema
```

Normalization must not apply pass/fail judgments.

### Stage 4 — Analyze

Controls consume normalized evidence.

```
NORMALIZED EVIDENCE
      ↓
APPLICABILITY
      ↓
MEASUREMENT
      ↓
CONTROL ENVELOPE
      ↓
CLASSIFICATION
      ↓
DIAGNOSTIC CORRELATION
```

### Stage 5 — Persist derived results separately

A capture may be re-analyzed under a newer rule set. Therefore:

```
capture artifact version
≠
control-set version
≠
analysis-result version
```

## Recommended artifact split

```
captures/
  <capture-id>/
    manifest.json
    structure.json
    styles.json
    accessibility.json
    runtime.json
    network.json
    screenshots/
    traces/

analyses/
  <capture-id>/
    <control-set-version>/
      results.json
      summary.json
```

## Required invariants

- Raw evidence is immutable after capture.
- Imported evidence retains its original schema/version metadata.
- Normalization is deterministic for a given normalizer version.
- Analysis must not mutate source artifacts.
- A capture can be evaluated by multiple control-set versions.
- Missing data produces explicit applicability/degradation states.
- UI state is not part of diagnostic truth unless intentionally stored as evidence.
- Offline analysis must never pretend to have capabilities that were not captured.

## Performance precedent

The DevTools Performance model serializes trace/frame-related information into `OfflinePerformanceData`, then reconstructs feature-controller state through `setOfflineData`. This demonstrates that a rich diagnostic view can be recreated from a saved artifact when the required evidence was captured.

## Source files

- `packages/devtools_app/lib/src/shared/offline/offline_data.dart`
- `packages/devtools_app/lib/src/shared/framework/screen.dart`
- `packages/devtools_app/lib/src/screens/performance/performance_controller.dart`
- `packages/devtools_app/lib/src/screens/performance/performance_model.dart`

## Boundary

The global offline controller, per-screen serialization responsibility, JSON export model, and explicit offline-support flag are source-derived. The ORVIA directory layout, capture manifest, normalization contract, and version-separation scheme are ORVIA synthesis.
