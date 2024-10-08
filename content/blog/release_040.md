---
title: "WLDT Library Version 0.4.0"
description: ""
summary: ""
date: 2024-08-29T17:40:54+02:00
lastmod: 2024-08-29T17:40:54+02:00
draft: false
weight: 50
images: []
categories: []
tags: []
contributors: []
pinned: false
homepage: false
---

We're excited to announce the release of WLDT version 0.4.0! This update brings powerful new features to enhance your Digital Twin (DT) experience, including event observation capabilities, a robust storage layer, and a flexible query system.

For detailed information about these changes and their impact, please refer to the information provided: 

- [Official Changelog](/docs/change-logs/change-log-0.4.0/)
- [Official Documentation](/docs/guides/storage-layer/)

## Key Highlights

- **WldtEventObserver**: A new class has been introduced, simplifying the observation of specific events generated by Digital Twins and their components
- **Storage Layer**: The new storage layer allows DTs to store data related to their state, events, actions, and more. It consists of:
    - `Storage Manager`: Manage and use various storage systems (e.g., in-memory, file-based, DBMS) simultaneously.
    - `WldtStorage`: An abstract class to implement custom storage systems. The default in-memory storage is available for development and testing.
    - `Query System`: The query system enables external components like Digital Adapters to retrieve stored data efficiently, supporting both synchronous and asynchronous queries.

WLDT 0.4.0 significantly enhances the flexibility and capabilities of Digital Twins, making it easier to manage and retrieve data, and observe events. We encourage developers to explore these new features and integrate them into their projects.

Stay tuned for more updates!