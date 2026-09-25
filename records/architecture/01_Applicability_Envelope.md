# ORVIA Applicability Envelope

**Record ID:** ORVIA-ARCH-001  
**Source model:** Flutter DevTools screen/tool metadata  
**Source commit:** `405689bc49f9c7ae0d7ca77b3d9b0f698745989b`  
**Status:** Source-grounded architectural conversion

## Purpose

Convert Flutter DevTools' screen eligibility model into a general-purpose ORVIA **Applicability Envelope**. The envelope determines whether a diagnostic control is valid for the current evidence, runtime, device, capture mode, or project context before ORVIA evaluates that control.

## Source-derived pattern

Flutter DevTools centralizes tool requirements in `ScreenMetaData` and carries those requirements into `Screen`.

Observed source fields include:

- `requiresConnection`
- `requiresDartVm`
- `requiresFlutter`
- `requiresDebugBuild`
- `requiresAdvancedDeveloperMode`
- `supportsWebServerDevice`
- `worksWithOfflineData`
- `requiresLibrary`

The screen system also distinguishes three operating states:

1. showing offline data;
2. connected to an application;
3. not connected to an application.

The screen eligibility path checks requirements before exposing a tool as usable and can associate a disabled reason with the failed requirement.

### Architectural rule extracted

A diagnostic control must declare the conditions under which its output is valid. ORVIA should determine applicability **before measurement or classification**.

## ORVIA conversion

### Applicability contract

```yaml
control:
  id:
  domain:
  applicability:
    runtime_required: false
    dom_required: false
    cssom_required: false
    accessibility_tree_required: false
    screenshot_required: false
    interaction_trace_required: false
    network_trace_required: false
    computed_layout_required: false
    offline_evidence_supported: false
    supported_viewports: []
    supported_input_modes: []
    supported_rendering_contexts: []
    feature_dependencies: []
```

### Result contract

```yaml
applicability_result:
  control_id:
  status: applicable | not_applicable | degraded | unsupported
  failed_requirements: []
  evidence_available: []
  evidence_missing: []
  reason_code:
  explanation:
```

### Evaluation order

```
CONTROL REGISTRATION
        ↓
CAPABILITY DISCOVERY
        ↓
EVIDENCE INVENTORY
        ↓
APPLICABILITY CHECK
        ↓
┌──────────────────────────┐
│ applicable               │ → continue to measurement
│ degraded                 │ → run with declared limitations
│ not_applicable           │ → stop and return reason
│ unsupported              │ → stop and return capability gap
└──────────────────────────┘
```

## ORVIA invariants

- No control may silently run when a required evidence source is absent.
- `not_applicable` is not a failure classification.
- Applicability reasoning must be machine-readable.
- A degraded run must disclose which evidence or capability is missing.
- Offline evidence must be explicitly declared compatible by the control.
- UI visibility and control validity should both derive from the same applicability result.
- AI interpretation occurs after applicability, never instead of it.

## Recommended reason-code families

```
CAPABILITY_MISSING
EVIDENCE_MISSING
RUNTIME_REQUIRED
OFFLINE_UNSUPPORTED
VIEWPORT_UNSUPPORTED
INPUT_MODE_UNSUPPORTED
DEPENDENCY_MISSING
CAPTURE_INCOMPLETE
SCHEMA_UNSUPPORTED
CONTROL_EXPERIMENTAL
```

## Source files

- `packages/devtools_app/lib/src/shared/framework/screen.dart`

## Boundary

The specific Flutter/Dart requirement flags are source-derived. The web/UI capability fields and ORVIA status taxonomy are ORVIA architectural synthesis.
