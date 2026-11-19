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

## Using the Extension

Include the Compress Extension schema URI in an Item's `stac_extensions` array when using `compress:*` fields. The example Item shows representative Sentinel-2 metadata for a compressed representation. The values illustrate the field meanings and are not benchmark results.

## Contributing

Contributions are subject to the [STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md). Please follow the [STAC Specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md). Ensure example JSON is valid and that Compress Extension fields conform to [the schema](json-schema/schema.json).
