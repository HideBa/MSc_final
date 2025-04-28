# Chapter 3: Methodology

## 3.1 Overview

- FlatBuffers overview
- File structure
- FlatBuffer schema
- Header
- Spatial index
  - Hilbert sort
  - Packed Hilbert R-tree
- Attribute index
  - requirements for attribute index
  - Eytzinger layout
  - Static B+Tree
  - Payload and offset byte management
- Feature encoding
  - Geometry encoding
    - Boundaries
    - Semantic surfaces
  - Attribute encoding
  - Appearance/Material/Texture encoding
  - Geometry template/instance encoding
  - Extension encoding
  - Feature
- HTTP range requests
- Works on browser (WASM build)

## 3.2 FlatBuffers Overview

- Brief introduction to FlatBuffers and zero-copy parsing
- General workflow of FlatBuffers schema and compilation
- Data types and restrictions

## 3.3 File Structure

- Magic bytes
- Header
- Spatial index
- Attribute index
- Feature encoding

## 3.4 Spatial index

### 3.4.1 Hilbert sort

### 3.4.2 Packed Hilbert R-tree

### 3.4.3 Attribute index

- requirements for attribute index
- Eytzinger layout
- Static B+Tree
- Payload and offset byte management

## 3.5 Feature encoding

- Geometry encoding
  - Boundaries
  - Semantic surfaces
- Attribute encoding
- Appearance/Material/Texture encoding
- Geometry template/instance encoding
- Extension encoding
- Feature

## 3.6 HTTP range requests

- Partial fetch with HTTP range requests
- Works on browser (WASM build)
