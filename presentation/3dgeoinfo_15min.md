---
marp: true
theme: hideba
paginate: true
---

# FlatCityBuf: a new cloud-optimised CityJSON format

<!-- _class: columns -->

- ![w:500px](./figures/logos/logo.png)

- **Hidemichi Baba**

  Co-authors: Hugo Ledoux, and Ravi Peters

  ![bg contrast:280%](./figures/tb_rw/3dbag.png)

  Delft University of Technology, Netherlands

  **3DGeoInfo 2025**

---

# Motivation and Problem Statement

---

## Traditional vs Cloud-Optimized Geospatial Formats

<!-- _class: columns -->

- ### Traditional Formats

  **Download → Process → Use**

  - Download entire dataset
  - Process locally
  - High bandwidth usage
  - Storage requirements

  Examples: Shapefile, GeoJSON, GeoTIFF

- ### Cloud-Optimized Formats

  **Query → Stream → Use**

  - Partial data retrieval
  - Direct access via HTTP
  - Reduced latency
  - No local storage

  Examples: COG, FlatGeobuf, GeoParquet

---

## The 2D Revolution: Cloud-Optimized Formats

### **A paradigm shift in geospatial data delivery**

- **Cloud Optimized GeoTIFF (COG)**: Raster data with internal tiling
- **FlatGeobuf**: Vector features with spatial indexing
- **GeoParquet**: Columnar storage for analytics
- **PMTiles**: Single-file map tiles

**Key Innovation**:

- **Object storage instead of databases** - Static files on S3/CDN
- **Web-optimized delivery** - HTTP Range Requests for partial access
- **No backend infrastructure** - Direct client-to-storage queries

<!-- ---

## Example: FlatGeobuf in Action

<video width="1000" controls style="display: block; margin: 0 auto;">
  <source src="https://storage.googleapis.com/flatcitybuf/flatgeobuf_demo.mp4" type="video/mp4">
</video>

**Static file + HTTP Range Requests = Dynamic queries** -->

---

## The Gap: 3D City Models Still Behind

<!-- _class: columns -->

- ### Current 3D Distribution

  **3DBAG Example:**
  - Tile-based downloads
  - Choose tiles manually
  - Download complete tiles
  - No partial access

  ![h:250px center](./figures/tb_rw/3dbag_tile.png)

- ### The Problem

  **No cloud-optimized format for 3D city models**

  - CityJSON: Text-based, no indexing
  - CityJSONSeq: Streaming, but no spatial queries
  - 3DCityDB: Database-centric, complex setup

  **Research Question:**
  **_"How can we bring cloud optimization to 3D city models?"_**

---

## Solution: FlatCityBuf: A Cloud-Optimised CityJSON Format

### **10-20× faster performance with cloud-native streaming**

  <video width="800" controls style="display: block; margin: 0 auto;">
    <source src="https://storage.googleapis.com/flatcitybuf/demo_1k.mov" type="video/mp4">
  </video>

**Live Demo**: [https://fcb-web-prototype.netlify.app/](https://fcb-web-prototype.netlify.app/)

---

# Methodology

---

<!-- _class: columns -->

## FlatBuffers: General Idea

- #### FlatBuffers

  FlatBuffers is a binary serialisation framework developed by Google.
  Its main characteristics are:

  - Binary format
  - Access to serialised data without parsing
  - Low memory consumption
  - Strictly typed (Schema driven)

- #### JSON

  Compared with FlatBuffers, JSON is:

  - Text-based (Human-readable and interoperable)
  - Needs to parse (copying data is needed)
  - More code to access data

---

## File Structure Design

The file consists of:

1. Magic Bytes: `FCB10000` (Acronym of FlatCityBuf + Semantic versioning)
2. Header: Common properties of CityJSON features and metadata (FlatBuffers root table)
3. Index:
   - Spatial Index: packed Hilbert R-tree
   - Attribute Index: static B+Tree
4. Features: array of CityJSON features (FlatBuffers root table)

![w:1100px center](./figures/methodology/file_structure.png)

---

## 2.3 Header

The header encodes:

- **Core fields**: CityJSON metadata (version, transform, reference system, geographical extent)
- **Appearance**: Materials, textures, UV coordinates
- **Geometry templates**: Reusable structures for compact representation
- **Extension support**: Embedded schemas for self-containment
- **Attribute schema**: Data structure for attributes
- **Indexing metadata**: Metadata for spatial and attribute indexing (e.g. offset bytes and branching factor)

![w:1100px center](./figures/methodology/header.png)

---

## Spatial Indexing: Packed Hilbert R-tree

<!-- _class: columns -->

- ### Construction

  1. Calculate bounding boxes
  2. Map to Hilbert curve
  3. Sort by spatial locality
  4. Build R-tree bottom-up
  5. Pack into linear array

  ![w:350px center](./figures/tb_rw/hilbert_sort.png)_([m Williams, 2022](https://worace.works/2022/02/23/kicking-the-tires-flatgeobuf/))_

- ### Query Support

  - **Bounding box queries** in O(log n)
  - **Point/Nearest neighbor** search

  ![w:500px center](./figures/methodology/bbox_query.png)

---

## Attribute Indexing: Static B+Tree

<!-- _class: columns -->

- ### Structure

  - **Sort features** by indexed attribute
  - **Build B+Tree** bottom-up
  - **Store keys and pointers** efficiently
  - **Pack into linear array** for disk storage

  ![w:600px center](./figures/methodology/attribute_index.png)

- ### Query Support

  - **Exact Match**: `city = "Tokyo"`
  - **Range Queries**: `year BETWEEN 1990 AND 2000`
  - **Logical Combinations**: `(year > 1990) AND (height > 100)`

---

## 2.6 Feature Encoding: Structure

![w:1100px center](./figures/methodology/file_structure_feature.png)

- **Feature encoding preserves CityJSON structure with FlatBuffers efficiency**
  - **CityFeature**: city objects array, vertices, appearance data
  - **CityObject**: city object type, geometry, attributes, semantics, etc
  - **Flattened arrays**: Parallel structure for nested geometries

---

## HTTP Range Requests: Selective Data Access

<!-- _class: columns -->

- **Traditional Approach**

  ```text
  GET /data.json
  → Download entire file
  → Parse JSON
  → Filter features
  ```

  **Problem**: Downloads whole file, even if only a few features are needed

- **FlatCityBuf Approach**
  ![w:550px center](./figures/methodology/http_range.png)

  **Solution**: Only fetch required bytes

---

## Results & Evaluation

---
<!-- _class: columns -->

## Outcomes: Software and Libraries

-
  - ![w:320px center](./figures/results/fcb_core.png)
    FlatCityBuf core library (Rust) published on [crates.io](https://crates.io/crates/fcb_core)
  - ![w:320px center](./figures/results/fcb_cli.png)
    FlatCityBuf CLI tool (Rust) published on [crates.io](https://crates.io/crates/fcb_cli)
-
  - ![w:320px center](./figures/results/fcb_wasm.png)
    FlatCityBuf WASM bindings for TypeScript published on [npm](https://www.npmjs.com/package/fcb_wasm)
  - ![w:320px center](./figures/results/fcb_py.png)
    FlatCityBuf Python bindings published on [pypi](https://pypi.org/project/flatcitybuf/)

---

## Performance Results: Local environment

**Read time 8.6-256.8x faster and 50-80% lower memory usage on local environemnt**
![w:1100px center](./figures/results/bench_cjseq.png)

---

## Performance Results: over the network vs 3DBAG API

<!-- _class: columns -->

- ### ID Query Performance

  ![w:550px](./figures/results/webbench_id.png)

  **2× faster** than 3DBAG API for single feature retrieval

- ### Bounding Box Query

  ![w:550px](./figures/results/webbench_bbox.png)

  **15× faster** than 3DBAG API for bounding box queries

---

<!-- _class: columns -->

## Real-World Scale: Netherlands Dataset

- ![h:500px center](./figures/results/netherlands.png)

- #### 3DBAG's 10 million buildings are encoded in

  ### **70.4 GB** (single file)

  Data remains accessible for efficient subsetting despite the large file size.

  (Indexed all attributes; without index: 63.9GB. CityJSONSeq: 65.2GB - slightly smaller.)

---

# Discussion & Conclusions

---

<!-- _class: columns -->
## Discussion: Use Cases and Applications

- #### Flexible Data Download

  As the web application showed, users can download only features they want with given queries, even in a data format they prefer e.g. CityJSON, CityJSONSeq, OBJ, etc.

- #### Data Processing Applications

  As performance benchmarks showed, FlatCityBuf is particularly suitable for cases where data processing is I/O intensive such as 3DBAG generation pipeline.

---
<!-- _class: columns -->

## Impact on Server Architecture

- #### Traditional Server Architecture

  Complex, less scalable and expensive

  ![w:600px center](./figures/discussion/server_architecture.png)

- #### FlatCityBuf's Server Architecture

  Simple, scalable and cost-effective

  ![w:500px center](./figures/discussion/server_architecture_fcb.png)

---

## Limitations & Trade-offs

<!-- _class: columns -->

- **Good points:**
  - **Read performance**: Up to 256× faster iteration, 6× lower memory usage
  - **Web streaming**: HTTP Range Requests enable partial data retrieval
  - **Static hosting**: No database servers needed, works with CDN/S3

- **Drawbacks:**
  - **Immutable format**: Updates require rebuilding entire file
  - **Limited query flexibility**: Only spatial and attribute indices supported
  - **Complex client implementation**: client program needs to handle filtering features

  **Best for read-heavy, web-based applications**

---

## Future Work

1. **Supporting multiple languages such as C++**
   - Broader ecosystem adoption
   - Performance-critical applications

2. **Explore other encoding like Parquet**
   - Columnar storage comparison
   - Analytics-optimized formats

3. **Implementing web viewer**
   - Progressive 3D visualization
   - Interactive querying interface

---

## Conclusions

### **FlatCityBuf enables efficient 3D city model streaming**

**Key Contributions:**

- **read performance improvement** through zero-copy deserialization
- **Partial data retrieval** with HTTP Range Requests
- **Practical implementation** with Rust/WASM libraries

**Impact:**
Simplifies server architecture (static files only) and Reduces bandwidth requirements

**Limitations:**
Immutable format, limited query flexibility, and
  complex client implementation

---

## Thank you

**Questions?**

![bg right](./figures/logos/logo.gif)

**Resources:**

- Demo: [fcb-web-prototype.netlify.app](https://fcb-web-prototype.netlify.app/)
- Code: [github.com/TUDelft-3D/FlatCityBuf](https://github.com/cityjson/flatcitybuf)

**Contact:**
<h.b.baba@tudelft.nl>

---

## Appendix

---

## Feature & Geometry Encoding

<!-- _class: columns -->

- ### Preserving CityJSON Structure

  - **CityFeature**: Contains city objects, vertices, appearance
  - **CityObject**: Geometry, attributes, semantics
  - **Flattened arrays**: Handles nested geometries

  **Example geometry encoding:**

  ```text
  boundaries: [0,1,2,3,0,3,7,4...]
  strings: [4,4,4,4,4,4]
  surfaces: [1,1,1,1,1,1]
  ```

- ### Attribute Encoding

  - Binary representation (little endian)
  - Typed attribute schema
  - Compact storage format
  - Self-describing structure

  ![w:500px center](./figures/methodology/attribute_structure.png)
