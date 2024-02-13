+++
title = "Xenoblade 3: Gimmick breakdown (Part 1)"
date = 2024-01-12
draft = true

[extra]
enable_toc = true

[taxonomies]
tags = ["xenoblade", "re"]
+++

# What are gimmicks?
# Where are gimmicks defined?

## The LVB file

## The `SYS_GimmickLocation` table

However, adding rows to this table may not be effective for some gimmick types. The game uses another structure to only
load gimmicks that are actually needed from the table.

### Bounding volume hierarchies

A [bounding volume hierarchy](https://en.wikipedia.org/wiki/Bounding_volume_hierarchy) (BVH) is a tree 
for *n*-dimensional bounding volumes for geometric objects. A property of this tree is that each child of 
a node is fully contained inside the region of that node.

The game uses this tree to search for gimmicks that overlap a region centered on the player's position. The minimap
(and by extension, the world map) is one application of this search.

In a BVH tree, nodes can either be containers or leaf nodes (with data). Each node has a bounding volume, which in our
case is axis-aligned. To search for a leaf node, all (in our case it is a binary tree, so both) 
children of a node must be queried, but when a node is found that does not fully contain
the searched region, the entire downstream branch can be skipped. This allows for an efficient search, if the tree
is balanced and with regions of optimal size.

In Xenoblade 3, a BVH for gimmicks is defined in the `/gmk_r/misc/bvh.dat` file. Unlike LVB files, the BVH does not support
overlays, so each DLC pack overwrites this file with an updated one. This file contains one BVH tree for each map.
Each tree has the following format:

| Field | Type/Value |
| ----- | ----------- |
| Magic | "GBVH" |
| Section size | u32 |
| Version | 0x0100 |
| Map ID | u16 |
| Padding | (4 bytes) |
| Nodes | list of nodes |

Each node follows this format:

| Field | Type/Value |
| ----- | ----------- |
| Min X | f32 |
| Min Y | f32 |
| Min Z | f32 |
| Padding | (4 bytes) |
| Max X | f32 |
| Max Y | f32 |
| Max Z | f32 |
| Padding | (4 bytes) |
| Parent | Pointer |
| Left child | Pointer |
| Right child | Pointer |
| BDAT row ID | u32 |
| Has BDAT ID | bool |
| Padding | (3 bytes) |

The Pointer type is a 64-bit integer relative to the first node: 0 is node 0, 64 is node 1, 128 is node 2, etc. 
If the node is missing (e.g., no parent or left/right child), the value is -1.  
The BDAT row ID matches the `ID` field in `SYS_GimmickLocation`. (not `GimmickID`!)

I've also released a tool to convert the game's BVH trees from and to JSON as part of the 
[xeno-lvb](https://github.com/RoccoDev/xeno-lvb) repository.