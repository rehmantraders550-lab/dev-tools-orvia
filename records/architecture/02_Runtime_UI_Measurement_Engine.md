# ORVIA Runtime UI Measurement Engine

**Record ID:** ORVIA-ARCH-002  
**Source model:** Flutter DevTools Performance pipeline  
**Source commit:** `405689bc49f9c7ae0d7ca77b3d9b0f698745989b`  
**Status:** Source-grounded architectural conversion

## Purpose

Convert the Flutter DevTools performance architecture into an ORVIA **Runtime UI Measurement Engine** that keeps raw runtime evidence, normalized measurements, classification, and diagnosis separate.

## Source-derived pipeline

The DevTools Performance controller coordinates multiple feature controllers rather than placing all performance logic in a single view.

Observed feature controllers:

- Flutter frames;
- timeline events;
- rebuild statistics.

The controller listens for runtime extension events. For frame events, it parses `FlutterFrame` data and forwards the resulting frame model into the frame controller. Rebuild events are processed separately by the rebuild-count model.

A `FlutterFrame` preserves several distinct measurements:

- frame identifier;
- frame time range;
- build time;
- raster time;
- vsync overhead;
- timeline event data;
- optional enhanced-tracing state.

A frame can then derive diagnostic properties such as UI jank, raster jank, shader-related duration, and frame analysis.

`FrameAnalysis` decomposes timeline evidence into:

- build phase;
- layout phase;
- paint phase;
- raster phase;
- longest UI phase;
- expensive operation counts such as save-layer and intrinsic-layout operations.

The source also explicitly handles overlapping event structures instead of blindly summing nested phases.

## Architectural rule extracted

```
RUNTIME SIGNAL
    ↓
PARSE INTO STABLE MODEL
    ↓
ATTACH RELATED TRACE EVIDENCE
    ↓
DERIVE MEASUREMENTS
    ↓
CLASSIFY AGAINST CONTEXTUAL BUDGET
    ↓
ANALYZE CONTRIBUTING PHASES
    ↓
ATTACH CAUSAL EVIDENCE
```

The critical lesson is that **measurement precedes diagnosis**.

## ORVIA conversion

### Engine layers

#### 1. Runtime collectors

Collect observations only.

Potential evidence types:

- animation frames;
- long tasks;
- interaction timestamps;
- requestAnimationFrame traces;
- layout shifts;
- style/layout invalidations where observable;
- scroll samples;
- event timing;
- resource timing;
- network timing;
- input mode;
- viewport and refresh context.

#### 2. Normalized runtime events

```json
{
  "event_id": "",
  "capture_id": "",
  "domain": "runtime",
  "type": "",
  "start_us": 0,
  "duration_us": 0,
  "target": null,
  "parent_event_id": null,
  "metadata": {}
}
```

#### 3. Measurement models

Measurements remain factual and independent from thresholds.

Examples:

- frame duration;
- dropped-frame count;
- animation duration;
- interaction latency;
- scroll velocity;
- layout-shift magnitude;
- long-task duration;
- main-thread occupancy;
- repeated layout/paint work.

#### 4. Budget/envelope evaluator

Thresholds must be stored as policy, not embedded in capture code.

```yaml
measurement:
  id: motion.frame_duration
  value:
  unit: ms

envelope:
  policy_id:
  policy_version:
  context:
  min:
  max:
  target:
```

#### 5. Diagnostic decomposition

A violation should be decomposed into contributing phases/signals where evidence allows it.

```
VIOLATION
   ↓
TIME / EVENT CORRELATION
   ↓
CONTRIBUTING PHASES
   ↓
DOM / STYLE / NETWORK / SCRIPT CORRELATION
   ↓
EVIDENCE PACKAGE
```

## Context-sensitive budgets

Flutter DevTools computes the frame target from the display refresh rate instead of assuming a universal fixed frame budget.

ORVIA should preserve this principle:

> thresholds may depend on capture context.

Examples of ORVIA contexts:

- refresh rate;
- viewport;
- input mode;
- reduced-motion preference;
- device class when known;
- interaction type;
- animation role;
- page lifecycle state.

## Offline compatibility

Performance data can be serialized and restored, including frame data, trace data, display refresh rate, selected frame, rebuild-count data, and selected feature tab.

ORVIA should therefore design runtime evidence as reusable capture artifacts rather than transient UI state.

## Engine invariant

```
RAW EVENT ≠ MEASUREMENT ≠ THRESHOLD ≠ CLASSIFICATION ≠ EXPLANATION
```

These layers must remain independently inspectable.

## Source files

- `packages/devtools_app/lib/src/screens/performance/performance_controller.dart`
- `packages/devtools_app/lib/src/screens/performance/performance_model.dart`
- `packages/devtools_app/lib/src/screens/performance/panes/flutter_frames/flutter_frame_model.dart`
- `packages/devtools_app/lib/src/screens/performance/panes/frame_analysis/frame_analysis_model.dart`

## Boundary

Flutter frame/build/raster semantics are source-derived. Browser/UI runtime collectors and ORVIA measurement schemas are architectural translation, not claims about Flutter DevTools.
