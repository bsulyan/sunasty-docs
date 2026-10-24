# EXIF, GPS & C2PA Metadata Sanitization

Every photograph captured by modern smartphones and digital cameras contains hidden metadata markers. These include precise GPS geographic coordinates, camera serial numbers, device model strings, exposure parameters, and increasingly, **C2PA (Coalition for Content Provenance and Authenticity)** cryptographic provenance credentials.

When users share raw images online, this metadata often reveals home locations, travel patterns, and identity information.

The [Sunasty Privacy Tools](https://sunasty.com/privacy-tools/) and [Sunasty EXIF Reader](https://sunasty.com/exif-reader/) allow users to inspect and strip these markers completely on-device without recompression artifacts.

## JPEG Marker Segment Architecture

JPEG files are organized into variable-length marker segments starting with byte `0xFF`:

```
 [ 0xFF 0xD8 ] Start of Image (SOI)
        │
 [ 0xFF 0xE1 ] APP1 Marker (EXIF & TIFF Header)
        ├── Identifier: "Exif\0\0"
        ├── TIFF Header (Endianness: 'II' Little / 'MM' Big)
        └── Image File Directory (IFD0, SubIFD, GPS IFD)
                ├── GPSLatitudeRef & GPSLatitude
                ├── GPSLongitudeRef & GPSLongitude
                ├── DateTimeOriginal
                └── Make / Model / LensModel
        │
 [ 0xFF 0xEB ] APP11 Marker (C2PA Provenance Manifest)
        ├── JUMBF Superbox (urn:uuid:c2pa...)
        └── Cryptographic Signatures & Claim Generator
        │
 [ 0xFF 0xDB ] DQT (Quantization Tables)
 [ 0xFF 0xC0 ] SOF0 (Start of Frame)
 [ 0xFF 0xDA ] SOS (Start of Scan - Compressed Image Pixels)
        │
 [ 0xFF 0xD9 ] End of Image (EOI)
```

## Lossless Metadata Stripping (Zero Recompression)

Unlike typical web converters that decode images into an HTML5 Canvas and re-encode them as lossy JPEGs (which introduces generational compression artifacts and alters pixel colors), Sunasty's client-side binary parser performs **lossless segment splicing**:

```typescript
/**
 * Strips all EXIF, XMP, IPTC, and C2PA markers from a JPEG ArrayBuffer
 * without re-encoding or degrading image quality.
 */
export function stripJpegMetadata(buffer: ArrayBuffer): ArrayBuffer {
  const view = new DataView(buffer);
  
  // Verify SOI (Start of Image) marker: 0xFFD8
  if (view.getUint16(0, false) !== 0xffd8) {
    throw new Error('Invalid JPEG format: Missing SOI marker');
  }

  const chunks: ArrayBuffer[] = [];
  // Preserve SOI marker
  chunks.push(buffer.slice(0, 2));

  let offset = 2;
  const length = buffer.byteLength;

  while (offset < length) {
    // Read 0xFF marker prefix
    if (view.getUint8(offset) !== 0xff) {
      break;
    }

    const marker = view.getUint8(offset + 1);

    // SOS (Start of Scan: 0xFFDA) marks the beginning of image entropy data
    // EOI (End of Image: 0xFFD9) marks EOF
    if (marker === 0xda || marker === 0xd9) {
      chunks.push(buffer.slice(offset));
      break;
    }

    // Standalone markers without length (RST0..RST7)
    if (marker >= 0xd0 && marker <= 0xd7) {
      chunks.push(buffer.slice(offset, offset + 2));
      offset += 2;
      continue;
    }

    // Read segment length (includes length bytes themselves)
    const segmentLength = view.getUint16(offset + 2, false);
    const nextOffset = offset + 2 + segmentLength;

    // Filter out metadata markers:
    // 0xE1: APP1 (EXIF / XMP)
    // 0xEB: APP11 (C2PA JUMBF manifests)
    // 0xED: APP13 (Photoshop / IPTC)
    const isMetadataMarker = marker === 0xe1 || marker === 0xeb || marker === 0xed;

    if (!isMetadataMarker) {
      chunks.push(buffer.slice(offset, nextOffset));
    }

    offset = nextOffset;
  }

  // Combine preserved binary chunks into single output buffer
  const totalLength = chunks.reduce((acc, c) => acc + c.byteLength, 0);
  const result = new Uint8Array(totalLength);
  let pos = 0;
  for (const chunk of chunks) {
    result.set(new Uint8Array(chunk), pos);
    pos += chunk.byteLength;
  }

  return result.buffer;
}
```

## Security & Privacy Impact

- **Bit-for-Bit Pixel Preservation**: Stripping metadata via binary marker excision guarantees 100% preservation of original DCT coefficients and Huffman tables.
- **Zero Exposure**: Photographs of documents, family members, or confidential locations are sanitized before upload to cloud storage, social networks, or forums.

Explore metadata inspection and cleaning tools:
- [Sunasty EXIF Reader](https://sunasty.com/exif-reader/)
- [Sunasty C2PA Provenance Reader](https://sunasty.com/c2pa-reader/)
- [Sunasty Image Privacy Suite](https://sunasty.com/image/)
- [Sunasty Office Document Metadata Inspector](https://sunasty.com/office-metadata/)
- [Article: The Hidden Privacy Risks of Online File Tools](https://sunasty.com/articles/privacy-risks-online-file-tools/)
