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
  _"How can we bring cloud optimization to 3D city models?"_

---

## Solution: FlatCityBuf: A Cloud-Optimised CityJSON Format

#### **10-20× faster performance with cloud-native streaming**

  <video width="800" controls style="display: block; margin: 0 auto;">
    <source src="https://storage.googleapis.com/flatcitybuf/demo_1k.mov" type="video/mp4">
  </video>

**Live Demo**: [https://fcb-web-prototype.netlify.app/](https://fcb-web-prototype.netlify.app/)

---

## Performance Results: Speed

### **Read time up to 256× faster**

![w:1100px center](./figures/results/bench_cjseq.png)

**Key achievements:**

- CityJSONSeq: 37-84ms → FlatCityBuf: 0.3-0.6ms
- Memory usage: 70-75MB → 5-12MB (6× reduction)
- Zero-copy deserialization eliminates parsing overhead

---

## Performance Results: Web Queries

<!-- _class: columns -->

- ### ID Query Performance

  ![w:550px](./figures/results/webbench_id.png)

  **2-10× faster** than 3DBAG API for single feature retrieval

- ### Bounding Box Query

  ![w:550px](./figures/results/webbench_bbox.png)

  **Consistent sub-second** response for spatial queries

---

## Technical Innovation: Zero-Copy Architecture

![w:1100px center](./figures/methodology/file_structure.png)

**Key Components:**

1. **Magic Bytes**: Format identification
2. **Header**: Metadata & schema (single HTTP request)
3. **Spatial Index**: Packed Hilbert R-tree for bbox queries
4. **Attribute Index**: B+Tree for property queries
5. **Features**: Direct memory-mapped access

---

## How It Works: HTTP Range Requests

<!-- _class: columns -->

- ### Traditional Approach

  ```text
  GET /data.json
  → Download entire file
  → Parse JSON
  → Filter features
  ```

  **Problem**: Downloads unnecessary data

- ### FlatCityBuf Approach

  ```text
  GET /data.fcb
  Range: bytes=0-1024     → Header
  Range: bytes=1024-2048  → Index
  Range: bytes=5000-6000  → Features
  ```

  **Solution**: Only fetch required bytes

  ![w:700px center](./figures/methodology/http_range.png)

---

## Spatial Indexing: Packed Hilbert R-tree

<!-- _class: columns -->

- ### Construction

  1. Calculate bounding boxes
  2. Map to Hilbert curve
  3. Sort by spatial locality
  4. Build R-tree bottom-up
  5. Pack into linear array

  ![w:400px center](./figures/tb_rw/hilbert_sort.png)

- ### Query Support

  - **Bounding box queries** in O(log n)
  - **Nearest neighbor** search
  - **Streaming retrieval** via partial reads

  ![w:500px center](./figures/methodology/bbox_query.png)

---

## Real-World Scale: Netherlands Dataset

### **10 million buildings in a single 70.4 GB file**

<!-- _class: columns -->

- ![h:400px center](./figures/results/netherlands.png)

- **File remains queryable:**
  - All attributes indexed
  - Spatial queries supported
  - Partial retrieval enabled

  **Comparison:**
  - CityJSONSeq: 65.2 GB
  - With indices: +7.5 GB overhead
  - Better compression with more attributes

---

## Implementation & Adoption

<!-- _class: columns -->

- ### Available Tools

  **Core Library (Rust)**

  ```bash
  cargo install fcb_cli
  fcb ser -i data.jsonl -o data.fcb
  ```

  **WASM Bindings (TypeScript)**

  ```javascript
  import { FlatCityBuf } from 'fcb_wasm';
  const features = await fcb.query(bbox);
  ```

  Published on [crates.io](https://crates.io/crates/fcb_core) & [npm](https://www.npmjs.com/package/fcb_wasm)

- ### Server Architecture

  **Simplified infrastructure:**

  ![w:500px center](./figures/discussion/server_architecture_fcb.png)

  - Static file hosting only
  - No database required
  - CDN-friendly

---

## Use Cases & Applications

### **1. Flexible Data Access**

- Query only features of interest
- Export to multiple formats (CityJSON, OBJ, glTF)
- Progressive loading for web viewers

### **2. High-Performance Processing**

- I/O intensive operations (3DBAG generation)
- Batch processing pipelines
- Real-time analysis applications

### **3. Web-Native Distribution**

- Direct browser access
- No server-side processing
- Bandwidth-efficient streaming

---

## Limitations & Trade-offs

<!-- _class: columns -->

- ### Design Choices

  ✅ **Optimized for:**
  - Read performance
  - Web streaming
  - Static hosting

  ❌ **Not suitable for:**
  - Frequent updates
  - Complex SQL queries
  - Small datasets

- ### Technical Constraints

  - **Immutable format**: Requires regeneration for updates
  - **Client complexity**: WASM/binary processing needed
  - **Query flexibility**: Limited to predefined indices

  **Best for read-heavy, web-based applications**

---

## Conclusions

### **FlatCityBuf enables efficient 3D city model streaming**

**Key Contributions:**

- **60× performance improvement** through zero-copy deserialization
- **Cloud-native design** with HTTP Range Requests
- **Practical implementation** with Rust/WASM libraries

**Impact:**

- Simplifies server architecture (static files only)
- Reduces bandwidth requirements
- Enables web-based 3D city applications

**Future Work:**

- Python bindings for broader adoption
- Column-oriented format exploration (Parquet)
- Progressive 3D visualization

---

## Thank you

**Questions?**

![bg right](./figures/logos/logo.gif)

**Resources:**

- Demo: [fcb-web-prototype.netlify.app](https://fcb-web-prototype.netlify.app/)
- Code: [github.com/TUDelft-3D/FlatCityBuf](https://github.com/TUDelft-3D/FlatCityBuf)
- Thesis: Available at TU Delft repository

**Contact:**
<h.baba@student.tudelft.nl>
