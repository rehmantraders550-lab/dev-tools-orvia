# ORVIA Baseline vs Candidate Design Diff Engine

**Record ID:** ORVIA-ARCH-004  
**Source model:** Flutter DevTools App Size comparison architecture  
**Source commit:** `405689bc49f9c7ae0d7ca77b3d9b0f698745989b`  
**Status:** Source-grounded architectural conversion

## Purpose

Convert the DevTools App Size diff model into an ORVIA engine that compares a **baseline** capture against a **candidate** capture using normalized evidence rather than subjective visual review alone.

## Source-derived comparison model

The App Size controller supports both single-artifact analysis and OLD-vs-NEW comparison.

Before generating a diff, the controller verifies that the two input files are compatible types. It also detects an empty diff and reports identical inputs.

The comparison path produces a hierarchical diff map and then generates three views:

- `combined`
- `increaseOnly`
- `decreaseOnly`

Each resulting tree node represents a change. Positive values can be filtered into the increase-only tree, negative values into the decrease-only tree, while the combined tree preserves both.

The controller can also preserve meaningful structural segmentation—for example main/deferred application units—before diffing and can associate call-graph context when source data contains that information.

### Architectural rule extracted

A useful diff engine does more than compute `candidate - baseline`. It:

1. validates input compatibility;
2. normalizes both inputs to a comparable model;
3. preserves hierarchy;
4. computes signed changes;
5. supports directional filters;
6. retains context that helps explain where the change came from.

## ORVIA conversion

### Input contract

```yaml
comparison:
  baseline_capture_id:
  candidate_capture_id:
  required_schema:
  comparison_profile:
  dimensions:
```

### Compatibility gate

Before diffing, verify:

- same evidence schema family;
- compatible page/route identity;
- compatible viewport or an explicitly allowed responsive comparison;
- compatible collector capabilities;
- compatible unit definitions;
- compatible normalization version, or migrate both to a common normalized schema.

If incompatible:

```
DO NOT COMPUTE A MISLEADING DIFF
→ return structured incompatibility reason
```

### Normalized diff object

```json
{
  "metric_id": "",
  "node_id": "",
  "baseline": null,
  "candidate": null,
  "delta": null,
  "delta_ratio": null,
  "direction": "increase|decrease|unchanged",
  "unit": null,
  "children": [],
  "evidence": []
}
```

### Directional views

ORVIA should preserve the App Size model's three-way comparison idea:

```
COMBINED
  all meaningful changes

REGRESSIONS / INCREASE-OF-COST
  changes moving an undesirable metric upward
  or a desirable metric downward

IMPROVEMENTS / DECREASE-OF-COST
  changes moving an undesirable metric downward
  or a desirable metric upward
```

Important: "increase" is not automatically negative in design diagnostics. Direction must be interpreted through each metric's semantics.

### Proposed comparison domains

ORVIA may compare normalized evidence for:

- DOM node count and depth;
- element geometry;
- spacing/rhythm measurements;
- typography metrics;
- contrast measurements;
- accessible names/roles/states;
- focus order;
- asset/request counts;
- transfer size;
- interaction latency;
- animation duration/displacement;
- frame/runtime behavior;
- layout-shift measurements;
- responsive state changes;
- component hierarchy;
- visual-density measurements;
- categorical design-control outputs.

These are ORVIA targets, not Flutter DevTools App Size metrics.

## Hierarchical diff principle

Whenever possible, preserve the diagnostic hierarchy:

```
PAGE
 ├─ SECTION
 │   ├─ COMPONENT
 │   │   ├─ ELEMENT
 │   │   └─ ELEMENT
 │   └─ COMPONENT
 └─ SECTION
```

A top-level regression should be traceable to the child nodes that contributed to it.

## Comparison result classes

```yaml
comparison_result:
  compatibility:
  summary:
  combined:
  regressions:
  improvements:
  unchanged_significant:
  unresolved:
```

## Required invariants

- Compare normalized evidence, not presentation-layer strings.
- Preserve both original measured values.
- Preserve signed delta.
- Never erase hierarchy when aggregating.
- Identical inputs should be detected explicitly.
- Incompatible evidence should fail closed rather than produce a fabricated comparison.
- Directional classification must be metric-aware.
- AI explanation is downstream of the deterministic diff.

## Source files

- `packages/devtools_app/lib/src/screens/app_size/app_size_controller.dart`
- supporting comparison/treemap logic from the `vm_snapshot_analysis` integration used by that controller

## Boundary

OLD/NEW validation, hierarchical comparison, signed node changes, and combined/increase/decrease trees are source-derived. The web-design metrics, regression semantics, and ORVIA schemas are project-specific synthesis.
