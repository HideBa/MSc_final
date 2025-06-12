# Research Paper Summary

## Chapter 1: Introduction

### 1.1 Problem Statement

Three-dimensional city models have evolved from visualization tools to fundamental components in urban planning, environmental simulation, and emergency response. Major national initiatives like the Netherlands' 3DBAG, Japan's PLATEAU, and Switzerland's SwissBUILDINGS3D demonstrate widespread adoption. However, the shift from desktop-based to cloud-native GIS introduces technical challenges including network latency, bandwidth limitations, and concurrent user support. Traditional text-based formats like CityJSON and CityJSONSeq inherit performance limitations that hinder efficient cloud deployment. While cloud-optimized geospatial formats exist for 2D data, they don't comprehensively address 3D city models, creating a significant gap in cloud-native solutions for urban digital twin applications.

### 1.2 Research Objectives

The research investigates FlatBuffers as an encoding mechanism for CityJSONSeq to optimize cloud-native 3D city model access. The main research question asks: "How can CityJSONSeq encoding be optimized for faster access, lower memory consumption, and flexible feature querying in web environments?" Sub-questions address FlatBuffers schema design for all CityJSONSeq components, achieving logarithmic time complexity for spatial and attribute queries, and efficient web-based data subset retrieval. The proposed FlatCityBuf format combines FlatBuffers' binary serialization with HTTP Range Requests, prioritizing read performance over update capabilities while maintaining CityJSON's semantic richness.

### 1.3 Scope of the Research

The research focuses on defining FlatCityBuf data specification, implementing a Rust library with WebAssembly bindings, supporting spatial and attribute querying with logarithmic complexity, demonstrating HTTP Range Request capabilities, and evaluating performance against other formats. Secondary aspects outside the scope include implementing libraries in other programming languages, exploring alternative serialization frameworks like Parquet or Protocol Buffers, and optimizing for write operations. The thesis is structured across seven chapters covering theoretical background, related work, methodology, results, discussion, future work, and conclusions.

## Chapter 2: Theoretical Background

### 2.1 Strategies for Cloud-native GIS

Cloud-native GIS employs four primary optimization strategies to address performance challenges while leveraging cloud advantages. Tiling and partitioning divide large datasets into manageable chunks, exemplified by Tile Map Service (TMS) standards and implementations in 3DBAG and PLATEAU. Spatial indexing using R-trees, quadtrees, and KD-trees enables efficient spatial queries through pre-built indices. Data simplification and level-of-detail systems reduce complexity for visualization, often combined with tiling as seen in Cesium 3D Tiles. Binary encoding and compression reduce file sizes for faster downloads, though requiring client-side decompression trade-offs. These strategies collectively form "cloud-optimized" formats that enable on-demand geospatial data access.

### 2.2 Binary Files

Binary files store data as bit sequences optimized for machine processing rather than human-readable text. They offer superior storage efficiency and faster execution but present challenges including lack of human readability, debugging complexity, and potential portability issues. In geospatial domains, text-based formats include GeoJSON, GML, CityGML, and CityJSON, while binary formats include GeoPackage and GeoTIFF. Notably, no widely adopted standard binary formats exist for 3D city model data, representing a gap this research addresses. The Unix philosophy advocates plain text for universal interfaces, but binary encoding provides significant performance advantages for large-scale data processing applications.

### 2.3 WebAssembly

WebAssembly (WASM) is a low-level assembly-like language with compact binary format enabling near-native performance in web browsers. Standardized by W3C, it serves as a compilation target for C/C++, C#, and Rust, allowing these languages to run efficiently on web platforms. WASM complements JavaScript rather than replacing it, enabling developers to leverage performance characteristics for computationally intensive tasks while maintaining JavaScript's flexibility for application logic. Key advantages include code reuse across platforms and elimination of rewrites for performance-critical components. In this research, WASM enables Rust library compilation for 3D city model processing with near-native browser performance.

### 2.4 Row-based and Column-based Data Storage

Data storage systems use either row-based or column-based approaches. Row-based storage stores consecutive rows sequentially, performing well for single-row searches and OLTP systems requiring frequent lookups and updates. Column-based storage stores consecutive columns sequentially, excelling at analytical queries involving aggregation or filtering. For geospatial data, column-based storage suits analytical queries like calculating average building heights or filtering by construction year, while row-based storage is preferable for accessing entire records or frequently updated datasets. The choice depends on query patterns and use case requirements.

### 2.5 CPU Caches

Modern computing faces performance bottlenecks due to CPU speed exceeding memory access speeds. CPU caches use fast SRAM to store temporary copies of likely-to-be-reused data, leveraging temporal locality (recently accessed data) and spatial locality (nearby data). Cache hierarchies (L1, L2, L3) balance speed and size, with data loaded in 64-byte cache lines to amortize memory access latency. Cache performance depends on hit rates - hits provide fast access while misses require slower memory retrieval. Understanding cache behavior is crucial for designing binary data formats, as cache-friendly layouts can significantly improve application performance through better memory access patterns.

### 2.6 Serialization and Deserialization

Serialization translates data structures into bit-strings for storage or transmission, while deserialization reverses this process. Terminology varies across ecosystems (serialization, pickling, marshalling, flattening) but the core concept remains consistent. The process involves converting in-memory data structures to storable/transmittable formats and back. This fundamental concept underlies all data persistence and network communication, making efficient serialization crucial for performance-critical applications, especially those handling large datasets like 3D city models.

### 2.7 Zero-copy

Zero-copy techniques avoid copying data between memory locations, addressing significant overhead in conventional I/O operations where data traverses multiple memory regions (storage to kernel buffer to user-space to network buffers). Each copy consumes CPU cycles, memory bandwidth, and increases latency. Zero-copy approaches include memory-mapped I/O using mmap(), in-place parsing without intermediate copies, zero-copy system calls like sendfile(), and shared memory for inter-process communication. Modern serialization formats like FlatBuffers implement zero-copy through memory layouts allowing direct access to serialized data without separate deserialization steps.

### 2.8 Endianness

Endianness refers to byte storage order for multi-byte values. Little-endian stores least significant bytes at lowest memory addresses (Intel processors), while big-endian stores most significant bytes first (network byte order for Internet protocols). For 32-bit integer 0x12345678, little-endian stores as 0x78, 0x56, 0x34, 0x12, while big-endian stores as 0x12, 0x34, 0x56, 0x78. Understanding endianness is crucial for binary format design, especially for cross-platform compatibility and network data exchange.

### 2.9 Binary Search and Indexing Algorithms

#### Binary Search

Binary search provides logarithmic time complexity for sorted arrays but suffers from poor memory access patterns on modern hardware. Each comparison potentially causes cache misses, with worst-case memory reads proportional to tree height. This inefficiency is problematic for external memory or HTTP access where each operation incurs significant latency.

#### Eytzinger Layout

The Eytzinger layout rearranges array elements in level-order traversal sequence of a complete binary tree, improving memory access patterns while preserving binary search algorithms. Adjacent accesses often refer to elements in the same cache lines, enabling effective hardware prefetching and reducing network roundtrips for HTTP-based access.

#### S+Tree (Static B+Tree)

S+Trees optimize for static datasets by using implicit structure with mathematically calculated child positions. With block size B, trees achieve log_B(n) instead of log_2(n) memory accesses. The layout maximally fills blocks with no empty slots, eliminating explicit pointers and improving cache efficiency. S+Trees achieve up to 15× performance improvement over standard binary search while requiring only 6-7% additional memory, making them valuable for frequent searches on large, static datasets over high-latency connections.

## Chapter 2: Related Work

### 2.1 Cloud-Optimised Geospatial Formats

Cloud-optimised geospatial formats enable efficient on-demand access to geospatial data, offering four key advantages: reduced latency through partial data retrieval, scalability via parallel operations, flexibility with advanced query capabilities, and cost-effectiveness through optimised access patterns. Examples include Cloud Optimised GeoTIFF, GeoParquet, PMTiles, FlatGeobuf, 3D Tiles, and Mapbox Vector Tiles. These formats address limitations of traditional GIS formats that require complete file downloads and lack efficient cloud-native access patterns. The development of these formats represents a paradigm shift from desktop-based to cloud-native geospatial data processing.

### 2.2 CityGML, CityJSON and Enhancements

#### CityGML

CityGML is an OGC standard defining a comprehensive data model for 3D city models with geometric and semantic information through modular structure. Version 3.0 separates conceptual model from encoding, featuring Core module plus eleven thematic extensions (Building, Bridge, Tunnel, etc.) and five specialized modules (Appearance, PointCloud, Generics, etc.). The modular design allows implementations to support specific subsets based on application requirements while maintaining standard compliance.

#### CityJSON

CityJSON is a JSON-based encoding implementing a subset of the CityGML conceptual model, currently at version 2.0.1. Key features include flattened city objects architecture using unique identifiers with parent references, shared vertex arrays for geometry efficiency, separate semantic surface objects, geometry templates for reusing structures, coordinate quantisation for size reduction, and JSON Schema-based extension mechanism. These optimisations achieve up to 7× compression compared to CityGML while maintaining semantic richness.

#### CityJSONSeq

CityJSONSeq optimises CityJSON for streaming by decomposing objects into independent CityJSONFeature units representing complete city objects with hierarchical children. Each feature maintains local vertex lists and appearance data for self-containment, following Newline Delimited JSON specification. While providing improved compression and memory efficiency, the text-based format still exhibits limitations including lack of explicit data typing, complete parsing requirements, and absence of built-in indexing mechanisms.

### 2.3 Non-Geospatial Formats in Cloud Environments

Modern cloud-optimised geospatial formats leverage established non-geospatial structures: GeoParquet uses Parquet, FlatGeobuf uses FlatBuffers, and Mapbox Vector Tiles use Protocol Buffers. FlatBuffers outperforms alternatives in deserialisation efficiency and memory utilisation. Protocol Buffers requires complete dataset loading and data parsing, while FlatBuffers enables zero-copy access. Apache Parquet employs columnar storage with record shredding algorithm for nested data and hierarchical organisation (File, Row Group, Column Chunk, Page levels). Performance comparisons show FlatBuffers excelling in deserialisation and memory efficiency, while Protocol Buffers achieve better compression but with higher processing overhead.

### 2.4 Cloud-Optimised Geospatial Implementations

Contemporary implementations include Mapbox Vector Tiles (Protobuf-based with tile pyramid structure), PMTiles (Z/X/Y coordinate tiles with HTTP Range Requests), FlatGeobuf (FlatBuffers with packed Hilbert R-tree spatial index), GeoParquet (columnar storage for cloud data warehouses), and 3D Tiles (spatial partitioning for streaming 3D models with GLTF). Comparative analysis shows varying strengths: FlatGeobuf excels in deserialisation and spatial indexing, MVT in serialisation and storage efficiency, GeoParquet in compression and read operations, while formats like GeoJSON lag in performance metrics. Each format targets specific use cases with different optimisation strategies.

### 2.5 Research Gaps

Despite advances in CityJSON optimisation through various encoding techniques, approaches specifically tailored for 3D city models in cloud environments remain insufficient. While cloud-native optimisations have succeeded for 2D geospatial data (like FlatGeobuf for Simple Features), their application to 3D city models lacks thorough investigation. Previous work on binary encoding of CityJSON (CBOR, zlib, Draco) and experimental FlatBuffers implementation showed potential but didn't address cloud-native requirements including spatial indexing, attribute indexing, extensions, and partial data retrieval capabilities.

## Chapter 3: Methodology

### 3.1 Overview

FlatCityBuf addresses limitations of current 3D city model formats in cloud environments through three interconnected objectives: binary encoding using FlatBuffers for faster read performance while preserving semantic richness, dual indexing mechanisms (spatial packed Hilbert R-tree and attribute-based S+Tree) for accelerated queries, and cloud-native data access through HTTP Range Requests enabling partial retrieval. Research outcomes include data format specification, Rust library implementation with CLI tools, web demonstration prototype, and comprehensive performance evaluation. The file structure comprises magic bytes identifier, header with metadata, spatial index, attribute index, and features section, designed for incremental access through HTTP Range Requests.

### 3.2 Magic Bytes and Header Section

Magic bytes comprise eight bytes starting with 'FCB' followed by semantic versioning (currently '01000'). The header section encapsulates metadata as size-prefixed FlatBuffers table, maintaining CityJSON compatibility while adding FlatCityBuf-specific extensions. Core fields include CityJSON metadata (version, transform, reference system, geographical extent), appearance information (materials, textures, UV coordinates), geometry templates for reusable structures, extension support with embedded schemas for self-containment, and attribute schema with indexing metadata. The design ensures compact representation while providing all necessary information for file interpretation, enabling efficient header skipping when accessing specific features.

### 3.3 Spatial Indexing

FlatCityBuf implements packed Hilbert R-tree spatial indexing adapted from FlatGeobuf, combining Hilbert curve-based spatial ordering, maximally filled tree structure, bottom-up construction, and flattened storage format. Feature sorting uses Hilbert space-filling curve encoding of 2D centroid coordinates to ensure spatial locality, optimising both disk access and HTTP range requests. Index structure uses fixed-size binary nodes containing bounding box coordinates and byte offsets, built level-by-level in Eytzinger layout for cache optimisation. The implementation deliberately uses 2D rather than 3D indexing based on horizontal distribution patterns of city models, typical query patterns, standards compatibility, and implementation efficiency considerations. The approach prioritises read performance over update capabilities for cloud-optimised data delivery.

### 3.4 Attribute Indexing

FlatCityBuf implements a modified S+Tree (Static B+Tree) for efficient attribute-based queries, supporting standard comparison operators (equality, inequality, less/greater than, between) and logical combinations (AND operations). Key modifications include duplicate key handling through dedicated payload sections, multi-type support for various CityJSON attribute types, explicit node offsets for simplified implementation, and tag-bit mechanisms to distinguish between direct feature references and payload pointers. The serialisation strategy uses fixed-length encoding for numeric and temporal types, and fixed-width strings (50 bytes maximum) with padding for variable-length data. The structure optimises for HTTP Range Requests through streaming tree traversal, payload prefetching, request batching, and block alignment, achieving logarithmic time complexity for attribute queries while minimising network overhead.

### 3.5 Feature Encoding

Feature encoding preserves CityJSON's semantic structure while leveraging FlatBuffers' binary efficiency. Core structures include CityFeature (root container with ID, objects, vertices, appearance), CityObject (individual features with type, geometry, attributes, hierarchical relationships), and comprehensive geometry encoding supporting boundaries, semantics, materials/textures, and geometry templates. The implementation addresses FlatBuffers' limitation with nested arrays through dimensional hierarchy encoding using parallel flattened arrays (boundaries → strings → surfaces → shells → solids). Semantic surfaces use parallel structure with semantics_boundaries and semantics_values arrays. Materials and textures follow CityJSON's appearance model with external file references for performance efficiency. Attributes are encoded as binary data with schema-defined types, while extensions are supported through enum markers combined with string type names, maintaining compatibility with CityJSON's extension mechanism while ensuring self-contained files.

### 3.6 HTTP Range Requests and Cloud Optimisation

FlatCityBuf enables selective data retrieval through HTTP Range Requests following RFC 7233 standards. The workflow involves initial header retrieval (magic bytes, header size, header content), followed by selective index navigation (spatial R-tree or attribute S+Tree traversal), targeted feature resolution using byte offsets, and progressive processing enabling immediate data use. Optimisation techniques include request batching to group nearby features, payload prefetching for attribute indices, streaming index traversal loading only necessary nodes, and buffered HTTP client caching to avoid redundant requests. The implementation integrates seamlessly with cloud infrastructure (AWS S3, Google Cloud Storage, Azure Blob Storage) enabling serverless architectures without specialised application servers, significantly reducing deployment complexity and operational costs while maintaining high performance for partial data access.

## Chapter 4: Results and Analysis

### 4.1 Web Prototype and Cross-Platform Implementation

A functional web prototype demonstrates FlatCityBuf's capabilities using a 3.4GB dataset covering 20km × 20km of South Holland, served from Google Cloud Storage. The browser-based application supports spatial queries (bounding box, point intersection), attribute queries (filtering by conditions), and data export functionality, showcasing responsive performance despite large file sizes. Cross-platform implementation provides native Rust library and WebAssembly modules for browser environments, with HTTP Range Request capabilities across platforms. Integration with cloud infrastructure enables serverless deployment using standard object storage services, eliminating traditional database and application server requirements through client-side filtering approaches.

### 4.2 File Size Analysis

Comprehensive file size comparisons across diverse datasets reveal FlatCityBuf's compression efficiency depends on several factors. Level of detail shows minimal impact on compression ratios (consistently 24-25% regardless of geometric complexity). Attribute quantity significantly influences performance, with compression improving from 5.07% (10 attributes) to 44.13% (1000 attributes) due to schema-based storage eliminating redundant key repetition. Geometric complexity enhances compression from 14.94% (simple models) to 26.06% (complex models) through efficient boundary encoding. Coordinate scale creates transition effects, with FlatCityBuf showing inferior compression (-28.65%) for small coordinate values but superior compression (17.79%) for large-scale coordinates due to fixed-size integer representation versus text-based variable-length encoding in CityJSONSeq.

### 4.3 Local Environment Benchmarks

Performance evaluations across multiple formats demonstrate FlatCityBuf's significant advantages. Compared to CityJSONSeq, processing time improvements range from 8.6× to 256.8×, while memory consumption reductions span 2.1× to 6.3×. Against CBOR, processing speeds increase 11.2× to 194.7× with memory efficiency gains of 27.3× to 1011.2×. BSON comparisons show processing improvements of 17.8× to 541.8× and memory reductions of 38.9× to 1409.0×. Generally, smaller datasets exhibit higher performance ratios due to proportionally greater parsing overhead, while larger datasets show substantial absolute time savings. The zero-copy deserialization approach provides consistent advantages across all comparison formats, with particularly dramatic improvements in memory utilisation for formats requiring full dataset loading.

### 4.4 Web Environment Benchmarks

Real-world web performance comparisons with the 3DBAG API demonstrate FlatCityBuf's practical advantages. Feature ID queries show 2.1× faster performance on average (ranging 1.5× to 2.7× across different locations), while bounding box queries achieve 15.1× faster performance. The significant improvement for spatial queries results from FlatCityBuf's Hilbert curve-based feature sorting enabling spatially proximate features to be retrieved in batched operations. Despite architectural differences between static file serving and traditional API-database systems, FlatCityBuf consistently outperforms existing web services, highlighting its potential for web application deployment while acknowledging the comparison limitations due to fundamental architectural differences.

## Chapter 5: Discussion

### 5.1 Use Cases and Applications

FlatCityBuf excels in flexible data download scenarios, enabling users to download precisely filtered datasets based on specific criteria rather than predefined tiles, as demonstrated by the web prototype supporting attribute-based filtering and spatial selection. For data processing applications, the format's superior read performance makes it particularly valuable in I/O-intensive analytical workflows, such as the 3DBAG generation pipeline where multiple processing stages require frequent file access. The unified single-file approach simplifies analytical processes by eliminating complex data chunking and file aggregation requirements common in traditional large-scale processing workflows.

### 5.2 Server Architecture Impact

FlatCityBuf enables significant simplification of server architectures by eliminating traditional database and application server requirements in favour of static file hosting. This approach leverages cloud providers' inherent scalability and high availability infrastructure, offering substantial cost advantages over conventional RDBMS-based systems. For example, Google Cloud Storage costs $0.020 USD per GB per month compared to $0.25 USD per vCPU hour for compute instances, demonstrating the economic benefits of static file hosting for 3D city model delivery. The architecture also provides unlimited scalability through cloud infrastructure without requiring complex sharding or replication strategies typical of traditional database systems.

### 5.3 Limitations and Trade-offs

FlatCityBuf presents certain constraints compared to traditional approaches. Query flexibility remains more limited than specialised spatial databases, supporting primarily attribute-based filtering and basic spatial operations (bounding box, nearest neighbour, point intersection) rather than comprehensive spatial analysis capabilities available in PostGIS-enabled systems. The format shifts computational complexity to client applications, representing an extreme "thick client" architecture that requires platform-specific implementations rather than universal web API access. Update operations present challenges due to the format's immutable structure, requiring complete file rewrites for data modifications, making it less suitable for frequently updated datasets but optimal for analysis and download services.
