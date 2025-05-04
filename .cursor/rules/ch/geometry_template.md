### Geometry Template and Instance Encoding

FlatCityBuf supports CityJSON's Geometry Templates for efficient representation of repeated geometries.

**Template Definition (in `header.fbs`)**

Geometry templates are defined globally within the `Header` table:

```
// Within header.fbs, inside the Header table:
table Header {
  // ... other fields ...
  templates: [Geometry];             // Array of template Geometry definitions
  templates_verteces: [DoubleVertex]; // Array of all vertices used by templates (f64 precision)
  // ... other fields ...
}

struct DoubleVertex {
  x: double;
  y: double;
  z: double;
}

```

- `templates`: An array of standard `Geometry` tables (defined in `geometry.fbs`), each representing a reusable geometry shape. The encoding of boundaries, semantics, etc., within these template geometries follows the standard `Geometry` encoding rules.
- `templates_verteces`: A single flat array containing all vertices for *all* templates. Vertices are stored as `DoubleVertex` (using `f64`) to maintain precision, as templates are often defined in a local coordinate system. The indices used within a template's `boundaries` refer to its specific block of vertices within this global array.

**Instance Definition (in `feature.fbs` via `geometry.fbs`)**

Individual CityObjects use `GeometryInstance` tables to reference and place templates:

```
// Within feature.fbs, inside the CityObject table:
table CityObject {
  // ... other fields ...
  geometry_instances: [GeometryInstance]; // Array of instances using templates
  // ... other fields ...
}

// Defined in geometry.fbs:
table GeometryInstance {
  transformation: TransformationMatrix; // 4x4 transformation matrix
  template: uint;                     // 0-based index into Header.templates array
  boundaries: [uint];                 // MUST contain exactly one index into the *feature's* vertices array (the reference point)
}

struct TransformationMatrix {
  m00:double; m01:double; m02:double; m03:double; // Row 1
  m10:double; m11:double; m12:double; m13:double; // Row 2
  m20:double; m21:double; m22:double; m23:double; // Row 3
  m30:double; m31:double; m32:double; m33:double; // Row 4
}

```

- `geometry_instances`: An array within a `CityObject` holding references to templates.
- `GeometryInstance`:
  - `template`: The index of the template geometry in the `Header.templates` array.
  - `boundaries`: Contains **exactly one** `uint` index. This index refers to a vertex within the *containing `CityFeature`'s* `vertices` array (which uses `int` coordinates). This vertex serves as the reference point for the instance.
  - `transformation`: A `TransformationMatrix` struct defining the 4x4 matrix (rotation, translation, scaling) applied to the template geometry relative to the reference point. The 16 `f64` values are stored row-major.

This separation allows defining complex shapes once in the header and instantiating them multiple times within features using only an index, a reference point index, and a transformation matrix.
