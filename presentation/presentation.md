---
marp: true
theme: hideba
paginate: true
---

# Could-Optimised CityJSON
### Hidemichi Baba
student #5967538
h.baba@student.tudelft.nl
**Responsible Supervisor: Hugo Ledoux**
**2nd Supervisor: Ravi Peters**
**Date P2: 2025-01-27**

---

## Index

<div class="grid grid-cols-2 gap-4 index">

<div>

1. Introduction
   - Background
   - Key Motivation

2. Related Work
   - CityJSON and Its Variants
   - Cloud-Optimised Data Formats
   - Peters' Preliminary Work

3. Research Questions
   - Primary Question
   - Objectives
   - Non-Primary Focus Areas

4. Technical Implementation
   - FlatBuffers Overview
   - Encoding Structure
   - Schema Design
   - Spatial Indexing

</div>

<div>

5. Results and Demonstration
   - Performance Comparison
   - Partial Data Retrieval Demo
   - Implementation Status

6. Project Management
   - Time Planning
   - Tools and Datasets

7. Conclusion

</div>

</div>

---

## Introduction

### Background

- 3D city models evolve beyond visualization e.g.
  - PLATEAU (Japan's nationwide 3D city model)
  - 3DBAG (Netherlands' building registration)
- Common encoding formats: CityGML, CityJSON, 3DCityDB
- Growing demand for cloud-native solutions

### Key Motivation
- Current formats lack cloud optimization
- CityJSONSeq needs enhancement for:
  - Efficient data access
  - Reduced memory usage
  - Flexible querying

---

## Related Work

### CityJSON and Its Variants

<div class="grid grid-cols-2 gap-4">

<div>

- **CityJSON:** JSON-based encoding for CityGML, version 2.0.1
  - Flattened architecture with unique identifiers
  - Shared vertex array with quantised coordinates
  - Simplified extension mechanism via JSON Schema

</div>

<div>

![CityJSON logo](./figures/cityjson.png)

</div>

</div>

- **CityJSONSeq:** Optimizes streaming by decomposing objects into independent sequences
  - Reduced file size and faster read time
  - Allows fetching subset of data without parsing entire file

---

## Cloud-Optimised Data Formats

### General Characteristics
According to Cloud-Native Geospatial Foundation:
- Specialized data structures for distributed cloud environments
- **Key Advantages:**
  - Reduced latency through partial data retrieval
  - Scalability via parallel operations
  - Enhanced query flexibility
  - Cost-effectiveness in storage and data transfer

---
## Cloud-Optimised Data Formats
### Geospatial Implementations
- Leverage non-geospatial structures for geospatial efficiency
- **Notable Formats:**
  - **GeoParquet:** Columnar storage for interoperability with cloud warehouses
  - **FlatGeobuf:** Utilizes FlatBuffers for efficient serialization
  - **Mapbox Vector Tiles:** Protobuf-based, tile pyramid structure
  - **PMTiles:** Standardized tile management via HTTP Range Requests

---

### Comparative analysis of Cloud-Optimised Geospatial Formats

Multi-criteria evaluation comparing performance, efficiency, and implementation aspects of cloud-optimised formats.

![Cloud-Optimised Geospatial Formats w:1000](./figures/dataformat_comparison.png)
*Scale: 1 (Poor) → 3 (Moderate) → 5 (Excellent)*

---
## Related Work: Peters' preliminary work

<div class="grid grid-cols-2 gap-4">

<div>

### Key Findings
- **Performance Advantages:**
  - Enhanced feature access speed
  - Reduced memory consumption
  - Decreased storage requirements




</div>

<div>

![Peter's work](./figures/ravi_work.png)

</div>

</div>

*Source: [github.com/3DBAG/CityBuf](https://github.com/3DBAG/CityBuf)*

---

## Research Questions

**Primary Question:**
How can the CityJSONSeq encoding be optimized for faster feature access, lower memory consumption, and flexible querying in web environments?

**Objectives:**
- Enhance decoding strategies for improved read performance
- Optimize memory usage during data processing
- Develop efficient mechanisms for spatial and attribute-based feature queries

**Non-Primary Focus Areas:**
- File size optimization (given efficient partial data retrieval)
- Data update and deletion speed (read performance prioritized)

---

## Methodology

- **FlatBuffers Schema Development:** Define schemas for Header and Feature components
- **Rust Library Implementation:** Encode and decode operations efficiently
- **Spatial Indexing:** Implement Packed Hilbert R-tree for 3D data
- **Attribute Indexing:** Develop B-tree based frameworks for arbitrary attributes
- **Partial Data Retrieval:** Utilize HTTP Range Requests for selective data access
- **Performance Evaluation:** Compare with CityJSONSeq using defined metrics

---

### FlatBuffers Overview: Core Concepts

<div class="grid grid-cols-2 gap-4">

<div>

**Key Features:**
- Zero-copy deserialization
- Memory-efficient binary format
- Schema-based serialization
- Direct memory access without parsing
- Forward/backward compatibility

</div>

<div>

**Supported Data Types:**
- Primitive types (int, float, bool)
- Strings and binary data
- Arrays and vectors
- Tables and structs
- Unions and enums

</div>

</div>

---

### Proposed Encoding Structure

![file structure w:1000](./figures/file_structure.png)

The encoded file structure comprises four distinct segments:
- **Magic Bytes:** Eight-byte identifier representing the file format
- **Header:** Common feature information and CityJSON metadata
- **Index:** Byte offset references to feature data locations
- **Features:** Sequential array of encoded features conforming to the feature schema

---

### FlatBuffers: Schema Structure

<div class="grid grid-cols-2 gap-4">

<div>

**Header Schema:**
```flatbuffers
table Header {
  version: string (required);
  transform: Transform;
  features_count: ulong;
  geographical_extent: GeographicalExtent;
  reference_system: ReferenceSystem;
  columns: [Column];
}

struct Transform {
  scale: Vector;
  translate: Vector;
}
```

</div>

<div>

**Feature Schema:**
```flatbuffers
table CityFeature {
  id: string (key, required);
  objects: [CityObject];
  vertices: [Vertex];
}

table CityObject {
  type: CityObjectType;
  id: string (key, required);
  geographical_extent: GeographicalExtent;
  geometry: [Geometry];
  attributes: [ubyte];
  children: [string];
  parents: [string];
}
```

</div>

</div>

*Note: Simplified schema definitions shown. Complete schemas include additional fields and structures.*

---

### Spatial Indexing

<div class="grid grid-cols-2 gap-4">

<div>

**Implementation Strategy:**
- Packed Hilbert R-tree for efficient spatial queries
- Uses bounding box of each CityFeature in 2D
- Integration with HTTP Range Requests to fetch only required data extents over the web

</div>

<div>

![spatial indexing w:700](./figures/packed_rtree.png)
![spatial indexing w:700](./figures/file_structure_rtree.png)

</div>

</div>

---

## Preliminary Results

**Completed Implementation Components:**
- Schema definitions for Header and CityJSONFeature
- Serialisation and deserialisation functionalities in Rust
- Performance comparison with CBOR, BSON, and CityJSONSeq
- Spatial indexing using 2D bounding boxes
- HTTP Range requests for partial data retrieval
- WebAssembly build pipeline with JavaScript bindings
- Prototype web application demonstrating practical benefits

*Implementation available on [GitHub (2025)](https://github.com/HideBa/flatcitybuf-web-prototype)*

---

### Performance Comparison

**Evaluation Framework:**
- Hardware: Apple MacBook Pro (M1 Max, 32GB RAM)
- Dataset: Subset from Ledoux et al. (2024), 100 iterations

**Key Findings:**
- Example: 3DBV dataset (269MB)
  - FlatCityBuf: 68ms read time
  - CityJSONSeq: 4.08s read time

![alt text w:1000](./figures/performance_comparison.png)

---

### Demonstration of Partial Data Retrieval

<div style="text-align: center;">
    <video controls src="https://storage.googleapis.com/flatcitybuf/demo.mp4" title="Title" width="75%"></video>
</div>

**Resources:**
- Live Demo: [fcb-web-prototype.netlify.app](https://fcb-web-prototype.netlify.app/)
- Source Code: [github.com/HideBa/flatcitybuf-web-prototype](https://github.com/HideBa/flatcitybuf-web-prototype)

---

## Evaluation Framework

### Quantitative Evaluation: Local Environment

**Performance Metrics:**
- Read operation time
- Memory consumption during processing
- Query execution duration

**Dataset:**
- Utilising dataset from Ledoux et al. (2024)
- Enables direct comparison with existing implementations

---

### Quantitative Evaluation: Web Environment

**Web Performance Metrics:**
- Response time and latency
- System throughput (requests/second)
- Network bandwidth utilisation
- Resource consumption (CPU/memory)

**Testing Environment:**
- Load testing using k6 or Locust frameworks
- Datasets: 3DBAG and static server deployment
- Client and server-side measurements

---

### Query Performance Analysis

**Five Fundamental Query Types:**
- Attribute-Based Retrieval
  - *e.g., buildings exceeding height thresholds*
- 2D Spatial Bounding Box
  - *Buildings within geographical boundaries*
- Point Intersection
  - *Buildings intersecting with 2D locations*
- Hierarchical Relationships
  - *Building-part relationship analysis*
- Level of Detail (LoD) Selection
  - *Specific LoD representation retrieval*

---

### Qualitative Assessment

**Key Evaluation Aspects:**
- Query flexibility and extensibility
- Interoperability with existing formats
- System scalability characteristics
- Architectural simplification potential

**Methodology:**
- Literature review
- Use case analysis
- Implementation effectiveness assessment

---

## Time Planning

| Start     | End      | Activity                                        | Status      |
|-----------|----------|------------------------------------------------|-------------|
| 01 Sep    | 10 Nov   | Initial research and topic exploration          | Completed   |
|           |          | P1 - Progress review: Graduation Plan           | Completed   |
| 14 Nov    | 31 Dec   | Literature review and analysis                  | Completed   |
| 14 Nov    | 15 Dec   | Analysis of message encoding formats            | Completed   |
| 14 Nov    | 15 Dec   | Evaluation of cloud-optimised geospatial formats| Completed   |
| 14 Nov    | 10 Jan   | Schema development and prototype implementation | Completed   |
|           |          | P2 - Formal assessment: Graduation Plan         | Completed   |
| 01 Feb    | 28 Feb   | Spatial indexing and sorting implementation    | Completed   |
| 01 Mar    | 31 Mar   | Attribute indexing development                 | In Progress |
| 01 Mar    | 31 Mar   | HTTP Range Request implementation              | Completed   |
| 01 Mar    | 31 Mar   | Encoding CityJSON's extension                  | Not Started |
| 01 Mar    | 31 Mar   | Setting up performance evaluation on server    | Not Started |
| 01 Apr    | 30 Apr   | Texture data encoding optimisation            | Not Started |
| 01 Apr    | 30 Apr   | Performance evaluation and analysis           | In Progress |
| 01 Apr    | 30 Apr   | Use case assessment                           | Not Started |
| 15 Apr    | 15 May   | Thesis composition and revision               | In Progress |

---

## Tools and Datasets

**Core Technologies**

| **Technology**           | **Purpose**                                                                                     |
|--------------------------|-------------------------------------------------------------------------------------------------|
| **Rust**                 | Primary development language for memory safety and performance                                |
| **cjseq**                | Rust library for converting between CityJSON and CityJSONSeq                                   |
| **FlatBuffers**          | Binary serialization framework for efficient encoding and zero-copy deserialization            |
| **WebAssembly**          | Enables selective data retrieval in the browser through compiled Rust implementations           |
| **TypeScript**           | Implements HTTP Range Request functionality                                                   |

Evaluation Datasets

<div class="grid grid-cols-2 gap-4">

<div>

| **Dataset**      | **Size** |
|------------------|-----------|
| **3DBAG**        | 5.9 MB    |
| **3DBV**         | 317 MB    |
| **Helsinki**     | 412 MB    |
| **Helsinki_tex** | 644 MB    |
| **Ingolstadt**   | 3.8 MB    |
| **Montréal**     | 4.6 MB    |

</div>

<div>

| **Dataset**      | **Size** |
|------------------|-----------|
| **NYC**          | 95 MB     |
| **Railway**      | 4.0 MB    |
| **Rotterdam**    | 2.7 MB    |
| **Vienna**       | 4.8 MB    |
| **Zürich**       | 247 MB    |

</div>

</div>

*Datasets are available at:*
[https://github.com/cityjson/paper_cjseq/tree/main/data](https://github.com/cityjson/paper_cjseq/tree/main/data)

---

## Conclusion

- Developed a FlatBuffers-based encoding methodology for CityJSONSeq
- Achieved preliminary outcomes in memory efficiency and feature access speed
- Ongoing work focuses on attribute indexing and texture data encoding
- Anticipated impact on cloud-native 3D city modeling and web-based applications

---

## Thank you for your attention!
