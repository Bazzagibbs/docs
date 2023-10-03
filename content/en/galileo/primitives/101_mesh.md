---
title: Mesh asset
linkTitle: Mesh
type: "docs"
description: A primitive asset for storing vertex data.
weight: 1
---

WIP

_This section is almost ready, I am in the process of moving it from the 
[Callisto repository](https://github.com/Bazzagibbs/callisto/blob/master/asset/specification.md)._

## Mesh overview

Mesh assets store an object's vertex data, and how the vertices are connected to form triangles.

## Mesh body
```
+------------------------+-------------------------+--------------------------+
|        40 bytes        |                         |                          |
|      Mesh manifest     | Vertex group info array |   Extension info array   |
|                        |                         |                          |
+------------------------+-------------------------+--------------------------+
|
| Buffer
|
+-----------------------------------------------------------------------------

                                      ...

 -----------------------------------------------------------------------------+
                                                                              |
                                                                              |
                                                                              |
 -----------------------------------------------------------------------------+
```

```c
struct Asset_Body_Mesh {
    Mesh_Manifest           manifest;           // 40 bytes
    Vertex_Group_Info*      vertex_group_infos; // Count provided in manifest. 55 bytes each.
    Extension_Info*         extension_infos;    // Count provided in manifest. 32 bytes each.
    uint8_t*                buffer;             // Size provided in manifest.
};
```

A `mesh` asset body **MUST** contain a 40 byte [`Mesh_Manifest`](#manifest), followed by a variable length 
array of 55 byte [`Vertex_Group_Info`](#vertex-group-info) structs. `Mesh` assets **MAY** include a variable 
length array of 32 byte [`Extension_Info`](../../extensions) structs.

The count of `Vertex_Group_Info` structs and `Extension_Info` structs **MUST** be provided in the manifest.


### Manifest
```
+-----------------------------------------------------------------------------------------------+
|00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31|
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|           Bounds center           |           Bounds extents          | VG count  | Ext count |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
|      Buffer size      |
+-----------------------+
```

```c
struct Mesh_Manifest {
    float32_t[3]    bounds_center;
    float32_t[3]    bounds_extents;         // half of bounding box dimensions
    uint32_t        vertex_group_count;
    uint32_t        extension_count;
    uint64_t        buffer_size;
};
```

A `mesh` **MUST** contain one or more `vertex groups`. 
Each `vertex group` **SHOULD** correspond to one draw call in the renderer, and **MAY** be used to draw different parts of a `mesh` with a different material or shader.

A `vertex group` **MUST** contain information for constructing `vertex attribute` slices. All `vertex attribute` data for a given `vertex group` **MUST** be contiguous and de-interleaved.

### Vertex Group Info

```
+-----------------------------------------------------------------------------------------------+
|00 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31|
+-----------+-----------+-----------+-----------+-----------+-----------+-----------------------+
|           Bounds center           |           Bounds extents          |  Buf slice begin idx  |
+-----------+-----------+-----------+-----------+--+--+--+--+--------+--+-----------------------+
|    Buf slice size     | Idx count | Vert count|TC|CL|JW|Ext attrib.|
+-----------------------+-----------+-----------+--+--+--+-----------+
```

```c
struct Vertex_Group_Info {
    float32_t[3]    bounds_center;
    float32_t[3]    bounds_extents;             // half of bounding box dimensions
    uint64_t        buffer_slice_begin_index;
    uint64_t        buffer_slice_size;

    uint32_t        indices_count;
    uint32_t        vertices_count;

    uint8_t         texcoord_channel_count;
    uint8_t         color_channel_count;
    uint8_t         joint_weight_channel_count;
    
    uint32_t        next_extension;             // index into extension_info array. uint32_max indicates no extensions.
};
```

### Extension Info
```c
struct Extension_Info {
    uint8_t[16]     extension_name;     // ascii string, null terminated or max length 16
    uint32_t        extension_version;
    uint64_t        data_begin_index;   // index into buffer.
    uint32_t        next_extension;     // index into extension_info array. uint32_max indicates no extensions.
};
```

### Buffer


