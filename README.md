# Compress Extension Specification

- **Title:** Compress
- **Identifier:** <https://stac-extensions.github.io/compress/v1.0.0/schema.json>
- **Field Name Prefix:** `compress`
- **Scope:** Item
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/README.md#extension-maturity):** Proposal

The STAC Compress Extension describes how an Item's data is compressed and the trade-offs made between storage size, reading speed, and downstream analytical accuracy. It is intended for Earth observation and environmental data, including imagery, gridded fields, and embeddings.

Earth Compress emphasizes that compression choices depend on both the source data and intended use. The extension records the fidelity mode, codec and its settings, any error tolerance, and available measurements of size, speed, and downstream task performance. These fields are optional: producers should include only values that are known and relevant, and should identify the data representation to which they apply.

## Examples

- [Catalog Example](examples/catalog.json)
- [Collection Example](examples/collection.json)
- [Item Example](examples/item.json)
- [JSON Schema](json-schema/schema.json)

## Item Properties

The `compress:*` fields describe the compressed representation of an Item's data. If compression varies by asset, provide asset-specific metadata or describe the applicable assets in `compress:configuration`. Error bounds are expressed in the data's native units; for `lossy` mode, `compress:error_bound` represents an allowed root-mean-square error, while for `near-lossless` it represents a maximum per-value absolute error.

| Field | Type | Units | Description |
| --- | --- | --- | --- |
| `compress:mode` | string | - | Fidelity mode: `lossless`, `near-lossless`, or `lossy`. |
| `compress:codec` | string | - | Compression codec or algorithm. |
| `compress:codec_version` | string | - | Version of the codec or algorithm. |
| `compress:configuration` | string | - | Name or identifier of the complete codec configuration. |
| `compress:objective` | string | - | Trade-off being prioritized, such as `compression_ratio`, `read_speed`, or `downstream_accuracy`. |
| `compress:parameters` | object | - | Codec-specific options used to produce this representation. |
| `compress:components` | array | - | Encoded components, the STAC asset holding each component, its transform bands, fidelity mode, and reconstruction order. |
| `compress:reconstruction` | object | - | Transform, array axes, output representation, and inverse operation needed to recombine components. |
| `compress:error_bound` | number | native units | Maximum absolute error in `near-lossless` mode or maximum RMSE in `lossy` mode. |
| `compress:error_bound_unit` | string | - | Unit for `compress:error_bound`, matching the source data's native unit. |
| `compress:nodata_preserved` | boolean | - | Whether NoData values remain in their original positions and no new NoData values are introduced. |
| `compress:zero_preserved` | boolean | - | Whether exact zeros in native units are preserved. |
| `compress:input_bytes` | integer | bytes | Size of the uncompressed input representation, excluding metadata and container overhead. |
| `compress:compressed_bytes` | integer | bytes | Size of the compressed representation. |
| `compress:compression_ratio` | number | - | Input size divided by compressed size. |
| `compress:bits_per_value` | number | bits/value | Compressed size in bits per input value. |
| `compress:encode_time_seconds` | number | seconds | Time to encode the representation. |
| `compress:decode_time_seconds` | number | seconds | Time to decode the representation. |
| `compress:encode_throughput_mb_s` | number | MB/s | Encoding throughput in decimal megabytes per second. |
| `compress:decode_throughput_mb_s` | number | MB/s | Decoding throughput in decimal megabytes per second. |
| `compress:downstream_metrics` | object | - | Named downstream task metrics measured on the compressed representation, for example `change_iou` or `stream_f1`. |

Compression ratio, bits per value, and throughput should be calculated consistently with the stated input representation. The same `error_bound` has different meanings in `near-lossless` and `lossy` modes; it is not a universal quality score. A codec's parameter names and meanings belong in `compress:parameters` and should be documented by its producer. Downstream metrics should include enough context in the configuration or linked documentation to identify the task and comparison method.

## Layered Components and Reconstruction

A layered codec can separate a transformed array into a base approximation and detail components. A decoder can fetch only the base for a smaller, smoother preview, or fetch the base and detail components for a higher-fidelity reconstruction. `compress:components` identifies each encoded part and the Item asset that stores it. The order and fidelity settings are component-specific; they should not be inferred from asset names.

Each component records:

- `id`: stable identifier within the Item.
- `role`: `base` for the low-frequency approximation or `detail` for detail coefficients.
- `asset`: key in the Item's `assets` object containing the encoded component.
- `reconstruction_order`: order in which the component is supplied to the inverse transform, starting at zero.
- `level` and `subbands`: transform level and subbands represented. In a 2D wavelet transform, `LL` is the approximation; `LH`, `HL`, and `HH` are detail orientations.
- `mode` and `compressed_bytes`: component fidelity mode and encoded byte size.
- Optional `target_bits_per_value`, `quantization_step`, and `error_bound` fields: per-component rate or distortion controls. State error units explicitly. A bound on transform coefficients is not the same as a bound on reconstructed source values.

`compress:reconstruction` defines the operation, transform, wavelet, axes, levels, boundary extension, output shape and dtype, and what to do if a component is omitted. These details matter for reproducibility: two decoders must use the same transform conventions and coefficient ordering to produce the same result.

The Item example demonstrates a three-level 2D CDF 5/3 decomposition of an illustrative 64 x 512 x 512 embedding. It stores the level-3 `LL` base and groups the `LH`, `HL`, and `HH` bands into one detail asset for each level. To reconstruct fully, decode the lossless base, inverse-transform it with the level-3 details to obtain the level-2 approximation, then add level-2 details, followed by level-1 details. If details are omitted, `zero_fill` means their coefficients are set to zero before the inverse transform; this produces a lower-detail approximation, not the original values. Finally apply the stated output conversion. The example's URLs and performance, size, and quality numbers are illustrative, not measurements of an existing codec.

For interoperability, component assets should contain a documented, self-describing stream or have their coefficient type, shape, byte order, and framing specified by the codec. The `asset` references connect the layer description to downloadable data; the inverse operation describes how those assets are recombined.

## Using the Extension

Include the Compress Extension schema URI in an Item's `stac_extensions` array when using `compress:*` fields. The example Item shows a layered compressed AlphaEarth embedding. Its URLs and measurements are illustrative.

## Contributing

Contributions are subject to the [STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md). Please follow the [STAC Specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md). Ensure example JSON is valid and that Compress Extension fields conform to [the schema](json-schema/schema.json).
