---
title: Extensions
linkTitle: Extensions
weight: 10
type: docs
simple_list: true
description: Provides application-specific data related to an asset.
---

```c
struct Extension_Info {
    uint8_t[16] extension_name;         // Ascii string, null terminated or max length 16.
    uint32_t    extension_version;
    uint64_t    data_begin_index;       // Index into asset's main data buffer.
    uint32_t    next_extension;         // (index + 1) into extension_info array. Zero indicates no extensions.
}
```
