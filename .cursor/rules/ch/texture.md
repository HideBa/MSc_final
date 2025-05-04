# How CityJSON’s texture looks like?

<https://www.cityjson.org/specs/2.0.1/#appearance-object>

> A Texture Object:
>
> - **must** have one member with the name `"type"`. The value is a string with either "PNG" or "JPG" as value.
> - **must** have one member with the name `"image"`. The value is a string with the name of the file. This file can be a URL (eg `"http://www.someurl.org/filename.jpg"`), a relative path (eg `"appearances/myroof.jpg"`), or an absolute path (eg `"/home/elvis/mycityjson/appearances/myroof.jpg"`).
> - **may** have one member with the name `"wrapMode"`. The value can be any of the following: `"none"`, `"wrap"`, `"mirror"`, `"clamp"`, or `"border"`.
> - **may** have one member with the name `"textureType"`. The value can be any of the following: `"unknown"`, `"specific"`, or `"typical"`.
> - **may** have one member with the name `"borderColor"`. The value is an array with 4 numbers between 0.0 and 1.0 (RGBA colour).

```json
"textures": [
  {
    "type": "PNG",
    "image": "http://www.someurl.org/filename.jpg"
  },
  {
    "type": "JPG",
    "image": "appearances/myroof.jpg",
    "wrapMode": "wrap",
    "textureType": "unknown",
    "borderColor": [0.0, 0.1, 0.2, 1.0]
  }
]
```

# What is the requirement for encoding texture

- Efficient storage & Retrieval: The format should allow rapid deserialization so that the texture can be used by GPU quickly
- Independence from specific format: There are several data formats used such as jpeg, png, dxt, astc, etc. It shouldn’t depend on the specific image format
- Cache friendly over HTTP
- It shouldn’t affect on the read performance when texture isn’t necessary for user

# What is the general/common way to encode texture?

## Case1: glTF/glb

### glTF

JSON file describes the scene graph, materials, mashes, animations, cameras, etc. Binary data (e.g. vertex buffers) and textures are typically stored as separate files referenced by the JSON. Texture can be either reference or data URIs.

<https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#metallic-roughness-material>

<https://github.com/KhronosGroup/glTF-Tutorials/blob/main/gltfTutorial/gltfTutorial_012_TexturesImagesSamplers.md>

- Example

### glb

Embed everything into single file. It uses binary container.

## Case2: OBJ

OBJ files typically reference external texture files (e.g., .mtl and .jpg/.png), relying on a modular approach to manage assets

- Example

## Case3: I3S

It uses reference to image data stored in specified directory.

- Example

<https://github.com/Esri/i3s-spec/blob/master/docs/1.8/textureDefinitionInfo.cmn.md>

# What are pros/cons of encoding strategies

## 1. Containing only metadata and link to image data

```
table Texture {
   format: enum of "PNG", // "PNG", "JPEN", etc
   uri: string
}
```

### Pros

- The FlatBuffer binary data will be small
- It might allow asynchronous or selective loading
- It can support vairous endpoint such as local files, cloud storage, etc. When we encodes CityJSON into FlatCityBuf, we can refer to the endpoint to the
- Caching might work well when it’s used on browser
- We can do lazy loading only for texture

### Cons

- The directory structure including FlatBuffer binary data and image files of texture must be kept together (No single file). It can potentially got broken its structure.

## 2. Containing both metadata and byte array

```
table Texture {
   width: uint;
   height: uint;
   format: enum of "PNG", // "PNG", "JPEN", etc
   data: [uint8]
}
```

### Pros

- Data is portable. It doesn’t need to keep the directory structure with texture data.

### Cons

- Entire FlatBuffer will be larger. Especially since we need to sort features with its geospatial properties, we need to have temporary storage before it writes into single file. It might fail to process huge data as spatially sorting massive data set is another research topic.
- Though FlatBuffers has good design of zero-copy, if texture is part of CityFeature, it needs to read all buffer of CityFeature. If we want to read texture all the time, it’s fine. But for 3D city model, sometimes texture isn’t necessary. e.g. processing only geometries, accessing to only attributes, etc

# Conclusion

Strategy1, having only reference to texture seems better way because

- user can choose to read texture into memory as well or not depending their needs. This method won’t ruins read performance even when texture isn’t needed.
- FlatBuffer file can be kept small which is realistic since we need to sort all features aiming to make single FlatBuffer file containing all buildings of the country
- Caching strategy will work well on the use on browser
- Lazy loading can work. Also we don’t need to implement the code to serialize/deserialize texture data.
