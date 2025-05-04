## extension support

flatcitybuf implements almost full support for the cityjson extension mechanism, allowing customization of the data model while maintaining compatibility with standard tools.

### extension mechanism background

cityjson extensions enable users to:

1. **add new attributes** to existing cityobjects
2. **create new cityobject types** beyond the standard types
3. **add new properties** at the root level of a cityjson file
4. **define new semantic surface types**

extensions are identified using a "+" prefix (e.g., "+noise") and are defined in json schema files hosted at urls referenced in the cityjson file.

### extension schema implementation

flatcitybuf supports extensions through specialized schema components in three main areas:

### 1. extension definition in `extension.fbs`

While CityJSON handles extensions by referencing external schema files (e.g., noise.city.json and noise.ext.json), FlatCityBuf takes a different approach. One of its key goals is to provide a fully self-contained binary representation of a CityJSON file. Therefore, instead of relying on external links, FlatCityBuf embeds the extension schema information directly into the header section of the file.

```
table Extension {
  name: string;                    // Extension name (e.g., "+Noise")
  description: string;             // Description of the extension
  url: string;                     // URL to the extension schema
  version: string;                 // Extension version
  version_cityjson: string;        // Compatible CityJSON version
  extra_attributes: string;        // Stringified JSON schema for attributes
  extra_city_objects: string;      // Stringified JSON schema for city objects
  extra_root_properties: string;   // Stringified JSON schema for root properties
  extra_semantic_surfaces: string; // Stringified JSON schema for semantic surfaces
}

```

### 2. extended cityobjects in `feature.fbs`

```
enum CityObjectType:ubyte {
  // ... standard types ...
  ExtensionObject
}

table CityObject {
  type: CityObjectType;
  extension_type: string; // e.g. "+NoiseCityFurnitureSegment"
  // ... other fields ...
}

```

### 3. extended semantic surfaces in `geometry.fbs`

```
enum SemanticSurfaceType:ubyte {
  // ... standard types ...
  ExtraSemanticSurface
}

table SemanticObject {
  type: SemanticSurfaceType;
  extension_type: string; // e.g. "+ThermalSurface"
  // ... other fields ...
}

```

### 4. extension references in header

extensions are referenced in the header, allowing applications to understand which extensions are used in the file:

```
table Header {
  // ... other fields ...
  extensions: [Extension];  // List of extensions used
  // ... other fields ...
}

```

### encoding and decoding strategy

the encoding and decoding of extensions follows these principles:

1. **self-contained extensions**: extension schemas are embedded directly in the file as stringified json, making the file self-contained and usable without external references.
2. **enum with extension marker**: special enum values (`extensionobject`, `extrasemanticssurface`) combined with a string field (`extension_type`) handle extended types. this approach maintains enum efficiency while supporting unlimited extension types.
3. **unified attribute storage**: extension attributes are treated the same as core attributes, both encoded in the attributes byte array. this simplifies implementation and maintains query performance.
4. **root properties**: extension properties at the root level are stored in the header's attributes field.

when encoding:

- if a cityobject type starts with "+", it's encoded as `extensionobject` with the full type name stored in the `extension_type` field
- if a semantic surface type starts with "+", it's encoded as `extrasemanticssurface` with the full type name stored in the `extension_type` field
- extension-specific attributes are encoded as regular attributes in the binary attribute array

when decoding:

- if a cityobject has type `extensionobject` and a non-null `extension_type`, the extension type name is used
- if a semantic surface has type `extrasemanticssurface` and a non-null `extension_type`, the extension type name is used
- all attributes are decoded, regardless of whether they belong to the core schema or extensions
