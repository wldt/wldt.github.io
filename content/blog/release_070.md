---
title: "WLDT Library Version 0.7.0"
description: ""
summary: ""
date: 2026-04-01T12:39:19+02:00
lastmod: 2026-04-01T12:39:19+02:00
draft: false
weight: 50
images: []
categories: []
tags: []
contributors: []
pinned: false
homepage: false
---

Version 0.7.0 introduces the **Augmentation Function** framework, a major feature addition that extends Digital Twin capabilities beyond basic shadowing. This release adds a fully event-driven augmentation layer, comprehensive storage support for augmentation data, new event types on the WLDT Event Bus, and native augmentation-aware callbacks inside the `DigitalTwinModel`.

Key Updates:

- **Augmentation Function Framework**: A hierarchical, modular architecture enabling Stateless and Stateful augmentation functions to be registered, executed, and managed independently from the core shadowing logic
- **Storage Layer Extension**: `WldtStorage` now defines a full set of abstract methods to persist and query augmentation function requests, results, errors, registrations, and unregistrations
- **WLDT Event Bus — Augmentation Events**: Eight new event types added to `WldtEventTypes` covering the complete augmentation lifecycle (registration, execution, result, error)
- **`DigitalTwinModel` Augmentation Integration**: The `DigitalTwinModel` now subscribes to augmentation events automatically and exposes optional callback methods and invocation helpers for both Stateless and Stateful functions

For detailed information about these changes and their impact, please refer to the information provided: 

- [Official Changelog](/docs/change-logs/change-log-0.7.0/)

Details about the new features:

- [Augmentation Functions](/docs/guides/augmentation-functions/)
- [Storage Update](/docs/guides/storage-layer/)

Stay tuned for more updates!
