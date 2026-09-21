# External weight format

A `GraphJson` constant with `ConstInit::Weights { ref }`, rendered in `.webnn` as
`@weights("ref")`, obtains its bytes from a sidecar. The graph declaration remains authoritative for
the logical data type and shape.

## Discovery

An explicit weights path is resolved relative to the graph. Without one, discovery checks, in order:

1. `<graph-stem>.safetensors`
2. `<graph-stem>.weights`
3. `model.safetensors`
4. `model.weights`

A SafeTensors file is self-describing and ignores a manifest argument. A raw `.weights` file requires
an explicit manifest or a discovered `<graph-stem>.manifest.json` or `manifest.json`. Manifest byte ranges
are validated before they are copied.

Files are memory-mapped read-only during resolution. Each selected tensor is copied once into the owned
`InlineBytes` representation used by `GraphJson` consumers.

## SafeTensors mapping

Logical types use the corresponding SafeTensors type: `float32`/F32, `float16`/F16, `int32`/I32,
`uint32`/U32, `int64`/I64, `uint64`/U64, `int8`/I8, and `uint8`/U8. A BF16 tensor may satisfy a
`float32` declaration; it is converted to F32 while loading. Shapes must otherwise match exactly.

SafeTensors has no native 4-bit type. Logical `int4` and `uint4` declarations use the versioned extension:

- archive metadata: `rustnn.webnn.packed4=1`
- physical SafeTensors type: U8
- physical shape: `[ceil(logical_element_count / 2)]`
- byte layout: the first logical element is the low nibble, followed by the high nibble

The loader requires the marker when a graph references an external 4-bit tensor and rejects unknown
versions, non-U8 storage, incorrect physical shapes, length overflow, or missing references. Archives that
contain only ordinary tensors need no marker and remain compatible with earlier files.

## Names

Exact tensor names take precedence. As a compatibility fallback, archive and manifest names are normalized
by replacing `::` with `__` and `.` with `_`. Resolution fails if multiple archive names normalize to the
same reference.

## Writing

`write_external_weights_safetensors` accepts the graph, bytes keyed by weight reference, and a destination.
It applies the same logical dtype, shape, packed-length, and marker rules as the reader. The archive is
streamed to a uniquely named temporary file in the destination directory and renamed only after successful
serialization, so an incomplete archive is never installed as the final path.
# External weight format

A `GraphJson` constant with `ConstInit::Weights { ref }`, rendered in `.webnn` as
`@weights("ref")`, obtains its bytes from a sidecar. The graph declaration remains authoritative for
the logical data type and shape.

## Discovery

An explicit weights path is resolved relative to the graph. Without one, discovery checks, in order:

1. `<graph-stem>.safetensors`
2. `<graph-stem>.weights`
3. `model.safetensors`
4. `model.weights`

A SafeTensors file is self-describing and ignores a manifest argument. A raw `.weights` file requires
an explicit manifest or a discovered `<graph-stem>.manifest.json` or `manifest.json`. Manifest byte ranges
are validated before they are copied.

Files are memory-mapped read-only during resolution. Each selected tensor is copied once into the owned
`InlineBytes` representation used by `GraphJson` consumers.

## SafeTensors mapping

Logical types use the corresponding SafeTensors type: `float32`/F32, `float16`/F16, `int32`/I32,
`uint32`/U32, `int64`/I64, `uint64`/U64, `int8`/I8, and `uint8`/U8. A BF16 tensor may satisfy a
`float32` declaration; it is converted to F32 while loading. Shapes must otherwise match exactly.

SafeTensors has no native 4-bit type. Logical `int4` and `uint4` declarations use the versioned extension:

- archive metadata: `rustnn.webnn.packed4=1`
- physical SafeTensors type: U8
- physical shape: `[ceil(logical_element_count / 2)]`
- byte layout: the first logical element is the low nibble, followed by the high nibble

The loader requires the marker when a graph references an external 4-bit tensor and rejects unknown
versions, non-U8 storage, incorrect physical shapes, length overflow, or missing references. Archives that
contain only ordinary tensors need no marker and remain compatible with earlier files.

## Names

Exact tensor names take precedence. As a compatibility fallback, archive and manifest names are normalized
by replacing `::` with `__` and `.` with `_`. Resolution fails if multiple archive names normalize to the
same reference.

## Writing

`write_external_weights_safetensors` accepts the graph, bytes keyed by weight reference, and a destination.
It applies the same logical dtype, shape, packed-length, and marker rules as the reader. The archive is
streamed to a uniquely named temporary file in the destination directory and renamed only after successful
serialization, so an incomplete archive is never installed as the final path.
