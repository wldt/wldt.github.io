---
title: "WLDT Library Version 0.8.0"
description: ""
summary: ""
date: 2026-10-08T12:00:00+02:00
lastmod: 2026-10-08T12:00:00+02:00
draft: false
weight: 50
images: []
categories: []
tags: []
contributors: []
pinned: false
homepage: false
---

Version 0.8.0 introduces three major additions. First, a complete **WLDT Monitoring System**, a new `it.wldt.monitoring` package providing a pluggable, handler-based metric infrastructure that is automatically injected into every DT component and tracks a rich set of framework-native metrics out of the box. Second, **Augmentation Function refinements**: a unified result-handling type, explicit state tracking for stateful functions, and full monitoring integration at the augmentation layer. Third, a **WldtEventBus thread-safety fix** that eliminates a `ConcurrentModificationException` under concurrent publish/subscribe.

Key Updates:

- **New `MonitoringInterface` system**: One per `DigitalTwin`, automatically injected into the Model, Adapters, Augmentation Functions, and Storage. It is opt-in per component and extensible through the `MonitoringInterfaceHandler`
- **5 metric types**: `WldtCounter`, `WldtUpDownCounter`, `WldtGauge`, `WldtTimer`, and `WldtHistogram`, all with a fluent mutation API and snapshot semantics for callbacks
- **Framework-native metrics**: Automatic tracking of latency, success/error counts, and state transitions for the DT Model, Physical Adapters, Digital Adapters, Storage, and the DT life cycle, without any developer action
- **Augmentation monitoring integration**: The `AugmentationFunction` base class gains built-in monitoring support, with the `handleMetricsRegistration()` hook for custom function-level metrics
- **`AugmentationFunctionResultList`**: A unified return type for augmentation function executions, carrying either a list of results or an error
- **Stateful function state tracking**: `isRunning()` on `StatefulAugmentationFunction` and new query methods to check if a stateful function is running
- **Event Bus fix**: Thread-safety improvement of the `WldtEventBus` under concurrent publish/subscribe

This release also introduces breaking changes in the Augmentation Function API (e.g., the removal of `StatelessAugmentationListener`). Please check the changelog before upgrading.

For detailed information about these changes and their impact, please refer to the information provided:

- [Official Changelog](/docs/change-logs/change-log-0.8.0/)

Details about the new features:

- [Monitoring Interface](/docs/guides/monitoring-interface/)
- [Augmentation Functions](/docs/guides/augmentation-functions/)
- [DT Events](/docs/introduction/dt-events/)

Stay tuned for more updates!
