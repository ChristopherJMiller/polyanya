# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Polyanya is a Rust implementation of the [Polyanya any-angle pathfinding algorithm](https://www.ijcai.org/proceedings/2017/0070.pdf) for navigation meshes. It provides efficient pathfinding on 2D navigation meshes with support for multi-layer navigation (overlapping floors, bridges, etc.).

## Build and Test Commands

```bash
# Build
cargo build

# Run all tests
cargo test

# Run tests with release optimizations (slower to compile, faster to run)
cargo test --release --examples

# Run a single test
cargo test test_name

# Format code (required for PRs)
cargo fmt --all

# Lint (required for PRs)
cargo clippy -- -D warnings

# Run benchmarks (not run in CI, use locally to check for regressions)
cargo bench
```

## Key Features

Enable features in Cargo.toml or via `--features`:
- `async` - Async pathfinding support via `FuturePath`
- `recast` - Integration with Recast navigation mesh format
- `serde` - Serialization support
- `stats` - Output benchmark statistics similar to the original C++ implementation
- `verbose` - Detailed debug output for algorithm behavior
- `detailed-layers` - Track layer transitions in paths
- `no-default-baking` - Disable automatic BVH baking on mesh creation

## Architecture

### Core Types

- **`Mesh`** (`src/lib.rs`): Main navigation mesh container holding one or more layers. Entry point for pathfinding via `mesh.path(from, to)`.

- **`Layer`** (`src/layers.rs`): A single navigation layer containing vertices and polygons. Layers can be offset and stitched together for multi-level navigation.

- **`Vertex`** / **`Polygon`** (`src/primitives.rs`): Navigation mesh primitives. Vertices track which polygons they belong to. Polygons can be marked as one-way.

- **`Path`** / **`Coords`** (`src/lib.rs`): Path results and coordinate types with optional layer information.

### Mesh Construction (`src/input/`)

Multiple ways to build a mesh:
- **`Triangulation`**: Build from outer edges and inner obstacles (Delaunay triangulation via `spade`)
- **`Trimesh`**: Build from raw triangle mesh data
- **`PolyanyaFile`**: Load from the original C++ implementation's file format
- **`RecastPolyMesh`** / **`RecastPolyMeshDetail`**: Import from Recast navigation mesh format

### Search Implementation

- **`SearchInstance`** (`src/instance.rs`): Implements the Polyanya search algorithm using a priority queue of `SearchNode`s
- **`SearchNode`** (`src/lib.rs`): Represents a search state with root point, visibility interval, and heuristic costs

### Multi-Layer Support

- **`src/stitching.rs`**: Connects layers at shared edges
- **`src/merger.rs`**: Merges overlapping meshes into a single layer
- Layers are indexed using a packed u32 format (8 bits layer, 24 bits polygon index)

### Optimizations

- **BVH** (`bvh2d`): Spatial indexing for fast point-in-polygon queries
- **Island detection**: Pre-compute disconnected regions to fast-fail unreachable paths
- Call `mesh.bake()` after construction (automatic unless `no-default-baking` feature enabled)
