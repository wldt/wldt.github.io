---
title: "WLDT Library Version 0.8.1"
description: ""
summary: ""
date: 2026-11-06T10:59:40+01:00
lastmod: 2026-11-06T10:59:40+01:00
draft: false
weight: 50
images: []
categories: []
tags: []
contributors: []
pinned: false
homepage: false
---

Version 0.8.1 aligns the **Digital Twin State** with the **Physical Asset Description**: properties, events and actions now carry both a semantic `type` and a `contentType` (the format) in the State too.

Key Updates:

- **`contentType` in the DT State**: `DigitalTwinStateProperty` and `DigitalTwinStateEvent` gain a `contentType` field (`DigitalTwinStateAction` already had it)
- **New constructors** that set both `type` and `contentType`
- **Deprecations**: constructors with only the `type` are deprecated and will not be supported anymore from version 0.9.0
- **Fixes**: `equals`/`hashCode`/`toString` of the State and Physical resources now include `type` and `contentType`, and several `NullPointerException` in `equals` are fixed

No behaviour of the framework changes: the core never interprets `type` or `contentType`, it only stores and forwards them, so existing code keeps compiling and running (with deprecation warnings).

For detailed information about these changes and the migration steps, please refer to the information provided:

- [Official Changelog](/docs/change-logs/change-log-0.8.1/)

Details about the affected topics:

- [Shadowing Function](/docs/guides/shadowing-function/)
- [Physical Adapter](/docs/guides/physical-adapter/)
- [DT Events](/docs/introduction/dt-events/)

Stay tuned for more updates!
