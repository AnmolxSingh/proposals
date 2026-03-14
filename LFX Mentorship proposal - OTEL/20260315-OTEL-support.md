# Propose OpenTelemetry support  in CAPA

## Table of Content

- [Summary](#Summary)
- [Motivation](#motivation)
  - [Problem Statement](#problem-statement)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories](#user-stories)
  - [Implementation Details](#implementation-details)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Alternatives](#alternatives)
- [Implementation Plan](#implementation-plan)
- [Resources](#resources)

## Summary
With the increasing adoption of CAPA we want to improve the supportability of CAPA especially in production. CAPA currently lacks OpenTelemetry support for tracing in controllers. This proposal suggests adding OpenTelemetry support to its controllers enabling tracing for better observability.

## Motivation
### Problem Statement
With the increasing adoption of CAPA we want to improve the supportability of CAPA especially in production. CAPA currently lacks tracing support for its controllers making it difficult to track failures in controllers during reconciliation.

### Goals
- OpenTelemetry SDK intialization and configuration in CAPA
- Root spans at reconciliation boundaries for core CAPA controllers
- Enable tracing for debugging and performance analysis
- Add sampling to control tracing levels which is to be accessible via the flags

## Proposal
### User Stories
#### 1. Controller Observability

As a CAPA operator, I want to trace controller reconciliation workflows so that I can diagnose cluster provisioning issues and understand controller execution behavior.

#### 2. Debugging Infrastructure Failures

As a developer working on CAPA, I want visibility into AWS API calls and controller operations so that I can quickly identify the source of failures during cluster lifecycle events.

### Implementation Details
#### Telemetry Initialization
OpenTelemetry will be initialized in the CAPA controller entry point located in the controller manager.

Example location:

`./main.go`

During controller startup:

1. Create an OpenTelemetry TracerProvider.

2. Define a service-level resource describing the CAPA controller.

3. Configure exporters (such as OTLP).

4. Register the tracer provider globally.

#### Tracing Controller Reconciliation

Each controller's reconciliation loop will be instrumented using spans.

Example controllers:

- AWSClusterReconciler

- AWSMachineReconciler

- AWSMachinePoolReconciler

- AWSManagedClusterReconciler

Tracing will be added at the beginning of the Reconcile() method.

Conceptual example:
``` 
   Reconcile()
       ↓
  start tracing span
       ↓
  perform reconciliation logic
       ↓
   end span
  ```

Spans will capture:

- reconciliation request metadata

- controller execution duration

- reconciliation result status

#### Shared Telemetry Utilities

To avoid duplication across multiple controllers, a shared telemetry helper package may be introduced.

Example location:

`pkg/observability/`


Responsibilities:

- initialize tracing

- provide reusable tracer helpers

- configure exporters and resources

This approach ensures maintainability and avoids repetitive instrumentation code in each controller.

#### Trace Export
Tracing data will be exported using OpenTelemetry exporters.

Possible export mechanisms include:

- OTLP exporters

- OpenTelemetry collectors

- compatible tracing backends such as Jaeger or Tempo

The exporter configuration should remain configurable via controller flags.

## Risks and Mitigations
### Performance Overhead

Tracing introduces additional runtime overhead.

Mitigation:

- use configurable sampling

- allow tracing to be disabled through configuration

- ensure spans remain lightweight

### Complexity of Integration

Adding telemetry may increase code complexity.

Mitigation:

- isolate tracing logic into a dedicated observability package

- minimize modifications to existing controller logic