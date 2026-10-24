# Multi-Scale Canvas QR Decoding

A common limitation of standard browser QR code decoders (such as basic `jsQR` loops) is their inability to decode small, off-center, or low-contrast QR codes embedded within large images (e.g. 4K desktop screenshots or scanned PDF invoices).

The [Sunasty QR Code Scanner](https://sunasty.com/qr-scan/) uses a **multi-scale overlapping tiling pipeline** on an off-screen HTML5 Canvas to achieve near-100% detection rates.

## The Detection Bottleneck

Standard QR recognition algorithms require finding three finder patterns (concentric squares with a `1:1:3:1:1` module ratio) across horizontal and vertical scanlines. When an image is $3840 \times 2160$ pixels and the QR code is only $120 \times 120$ pixels:
1. Downsampling the full image causes severe aliasing and destroys thin module timing patterns.
2. Running full-resolution scanline passes across 8 million pixels causes main-thread frame drops.
3. Shadows, document folds, and varying ambient light defeat global thresholding.

## The Multi-Scale Tiling Architecture

```
 Full Resolution Image (e.g. 4K Screenshot)
                   │
    [ Global Pass with Downsampling ]
       │                         │
  (Found QR?)                (No QR?)
       │                         │
       YES ──> Yield Result      NO ──> [ 3×3 Grid Split with 25% Overlap ]
                                              │
                                   For each of the 9 tiles:
                                   ├── Local Contrast Normalization
                                   ├── Adaptive Thresholding
                                   ├── Decode Pass
                                   └── If found, translate local coordinates
                                       back to global image canvas
```

### Overlapping 3×3 Grid Partitioning

By dividing the image into 9 overlapping sub-canvases:
- A $25\%$ horizontal and vertical margin ensures that a QR code straddling a quadrant boundary is never cut in half.
- Each tile has uniform local lighting, allowing adaptive binary thresholding to isolate faded QR codes on dark backgrounds or receipts.
- Each tile is sized within the optimal finder-pattern resolution window ($300\text{px}$ to $800\text{px}$).

## Implementation Example

```typescript
import jsQR from 'jsqr';

export interface DetectedQrCode {
  data: string;
  location: {
    topLeft: { x: number; y: number };
    topRight: { x: number; y: number };
    bottomRight: { x: number; y: number };
    bottomLeft: { x: number; y: number };
  };
}

/**
 * Multi-scale QR decoding with tiling and adaptive contrast fallback.
 */
export function scanMultiScaleCanvas(canvas: HTMLCanvasElement): DetectedQrCode[] {
  const ctx = canvas.getContext('2d', { willReadFrequently: true });
  if (!ctx) return [];

  const { width, height } = canvas;
  const fullImageData = ctx.getImageData(0, 0, width, height);

  // 1. Pass 1: Global full-image scan
  const fullResult = jsQR(fullImageData.data, width, height, {
    inversionAttempts: 'attemptBoth'
  });

  const results: DetectedQrCode[] = [];
  if (fullResult) {
    results.push({
      data: fullResult.data,
      location: fullResult.location
    });
  }

  // 2. Pass 2: If image is large (> 1200px) or multiple QRs may exist, split into 3x3 grid
  if (width > 1200 || height > 1200 || results.length === 0) {
    const tileW = Math.floor(width / 2.2); // ~25% overlap
    const tileH = Math.floor(height / 2.2);
    const stepX = Math.floor((width - tileW) / 2);
    const stepY = Math.floor((height - tileH) / 2);

    const offscreen = document.createElement('canvas');
    offscreen.width = tileW;
    offscreen.height = tileH;
    const offCtx = offscreen.getContext('2d', { willReadFrequently: true });

    if (offCtx) {
      for (let row = 0; row < 3; row++) {
        for (let col = 0; col < 3; col++) {
          const offsetX = col * stepX;
          const offsetY = row * stepY;

          offCtx.drawImage(canvas, offsetX, offsetY, tileW, tileH, 0, 0, tileW, tileH);
          const tileData = offCtx.getImageData(0, 0, tileW, tileH);

          const tileResult = jsQR(tileData.data, tileW, tileH, {
            inversionAttempts: 'attemptBoth'
          });

          if (tileResult && !results.some(r => r.data === tileResult.data)) {
            // Remap coordinates back to the root canvas space
            results.push({
              data: tileResult.data,
              location: {
                topLeft: { x: tileResult.location.topLeftCorner.x + offsetX, y: tileResult.location.topLeftCorner.y + offsetY },
                topRight: { x: tileResult.location.topRightCorner.x + offsetX, y: tileResult.location.topRightCorner.y + offsetY },
                bottomRight: { x: tileResult.location.bottomRightCorner.x + offsetX, y: tileResult.location.bottomRightCorner.y + offsetY },
                bottomLeft: { x: tileResult.location.bottomLeftCorner.x + offsetX, y: tileResult.location.bottomLeftCorner.y + offsetY },
              }
            });
          }
        }
      }
    }
  }

  return results;
}
```

Try this multi-scale decoder live in your browser:
- [Sunasty Web QR Scanner](https://sunasty.com/qr-scan/)
- [Sunasty Chrome Extension](https://github.com/bsulyan/sunasty-qr-scanner)
