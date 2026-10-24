# Client-Side PDF Architecture

Most web-based PDF editors upload user documents to remote cloud servers to run headless binaries (like `qpdf` or `pdftk`). This introduces severe security and regulatory risks for sensitive legal, medical, and financial documents.

The [Sunasty PDF Toolkit](https://sunasty.com/pdf/) eliminates server-side processing entirely. PDF parsing, page extraction, merging, and compression execute inside the client's browser memory via Web Workers and typed byte buffers.

## In-Memory Stream Architecture

```
 User Selects PDF (File input / Drag & Drop)
                    │
         [ File.arrayBuffer() ]
                    │
         Uint8Array In-Memory Buffer
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
   [ Header Check ]      [ Cross-Reference Table ]
   Verify %PDF-1.x       Read xref table & trailer
         │                     │
         └──────────┬──────────┘
                    ▼
     Web Worker Document Graph Construction
     (Catalog, Pages, Info Dict, Metadata Stream)
                    │
         ┌──────────┼──────────┐
         ▼          ▼          ▼
      Merge      Compress    Redact / Sign
    (Combine)  (Deflate)    (Vector Stamps)
                    │
                    ▼
     Uint8Array Output Buffer
                    │
     [ URL.createObjectURL(Blob) ]
                    │
     Instant On-Device Download
```

## Key Technical Invariants

### 1. Isolated Web Worker Execution
Processing multi-megabyte PDFs on the main UI thread causes frame drops and unresponsive browser tabs. Sunasty offloads all decompression, cross-reference table parsing, and binary stream concatenation to dedicated `Worker` threads via transferable `ArrayBuffer` objects:

```typescript
// Main thread dispatch
const worker = new Worker(new URL('./pdf-worker.ts', import.meta.url), { type: 'module' });

const buffer = await file.arrayBuffer();
// Transfer ownership without memory copying
worker.postMessage({ action: 'MERGE', buffers: [buffer] }, [buffer]);
```

### 2. Stream-Level Compression Without Visual Degeneracy
When compressing PDFs using [Sunasty PDF Compress](https://sunasty.com/pdf-compress/), the engine performs:
- **FlateDecode re-compression**: Maximizes Deflate zlib compression levels on text content streams.
- **Deduplication of Fonts and XObjects**: Merges duplicate embedded font descriptors across combined document pages.
- **Orphan Object Pruning**: Traverses the document graph starting from the `Root` catalog and strips unreferenced indirect objects.

### 3. Metadata Sanitation
Before sharing documents, users can inspect and sanitize embedded document history:
- Removing author names, software identifiers, and editing timestamps from the `/Info` dictionary.
- Purging serialized Adobe XMP XML packets (`/Metadata` streams).
- Flattening annotations and signature appearances to prevent post-export modification.

## Web Tools in the PDF Suite

- [Sunasty PDF Hub](https://sunasty.com/pdf/)
- [Sunasty PDF Merge](https://sunasty.com/pdf-merge/)
- [Sunasty PDF Compress](https://sunasty.com/pdf-compress/)
- [Sunasty PDF Metadata Inspector](https://sunasty.com/pdf-metadata/)
- [Sunasty PDF Deskew & Straighten](https://sunasty.com/pdf-deskew/)
- [Guide: How to Compress a PDF for Email](https://sunasty.com/guides/compress-pdf-for-email/)
- [Sunasty PDF Android Mobile App](https://play.google.com/store/apps/details?id=com.sunasty.sunasty_pdf)
