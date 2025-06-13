# Summary of Revisions – Thesis Update

## 1. Introduction

* **Expanded background**:

  * Added definitions and explanation of *cloud-native systems* and specifically *cloud-native GIS*, referencing key literature (e.g., Cloud-Native Geospatial Foundation).
* **More detailed problem statement**:

  * Elaborated on technical challenges such as network latency, bandwidth limitations, and performance bottlenecks of text-based formats.
  * Positioned CityJSONSeq limitations in a broader context, emphasizing the need for more efficient 3D geospatial data formats for web delivery.

---

## 2. Theoretical Background

* **New sections introduced**:

  * *Common strategies for cloud-native GIS* (e.g., tiling, spatial indexing, data simplification, etc.)
  * *Binary file formats* – added context to binary encoding advantages.
  * *WebAssembly (WASM)* – discussed for cross-platform deployment and client-side performance.
  * *Row-based vs column-based storage* –
  * *CPU caches* – discussed in relation to data layout and memory access optimization.
* These additions provide better motivation for the technical design choices in later chapters.

---

## 3. Related Work

* **Expanded literature review**:

  * Included a more comprehensive discussion of *CityGML*, *CityJSON*, and *CityJSONSeq*.
  * Improved comparison of cloud-optimised geospatial formats with a more structured evaluation table.

---

## 4. Methodology

* **Minor revisions**:

  * No major structural changes; small edits for clarity.
  * Figures were improved (e.g., clearer illustration of file layout, packed R-trees).
  * Added more explanation to support reproducibility and implementation.

---

## 5. Results

* **Expanded benchmarks**:

  * Added detailed results for *local environment benchmarking* including multi-format comparisons (CityJSONSeq, CBOR, BSON).
  * Added *Web benchmarking* – includes performance for bounding box and feature ID queries, as well as comparison with 3DBAG API.
  * Summarized both local and web results for clarity.

---

## 6. Discussion

* **Added more content**:

  * Expanded discussion on *server architecture*, comparing traditional server-client models with FlatCityBuf's static file architecture.
  * Discussed implications for *scalability*, *cost*, and *client-side complexity* in more depth.
  * Included architectural diagrams and literature references
