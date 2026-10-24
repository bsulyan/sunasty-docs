# sunasty_color_math (Dart & Flutter)

Official documentation for the `sunasty_color_math` package published on [pub.dev](https://pub.dev/packages/sunasty_color_math) under the Verified Publisher **`sunasty.com`**. A zero-dependency, high-performance Dart and Flutter library for sRGB, HSL, and OKLCH color space conversions, relative luminance calculation, and WCAG 2.1 accessibility contrast verification.

Originally developed for the [Sunasty Color Picker](https://sunasty.com/color-picker/) client-side tool and the [Sunasty Developer Suite](https://sunasty.com/dev/).

---

## Package Overview

- **pub.dev Registry**: [`sunasty_color_math` on pub.dev](https://pub.dev/packages/sunasty_color_math)
- **Publisher**: Verified Publisher **`sunasty.com`** (160 / 160 pub points)
- **Source Code**: [GitHub bsulyan/sunasty-color-math-dart](https://github.com/bsulyan/sunasty-color-math-dart)
- **License**: MIT
- **Dependencies**: 0 external runtime dependencies
- **Platforms**: Flutter, Dart VM, Web (WASM-ready), Android, iOS, Windows, Linux, macOS
- **Null Safety**: Sound null safety enabled (Dart 3+ compatible)

---

## Installation

Add `sunasty_color_math` to your `pubspec.yaml`:

```bash
dart pub add sunasty_color_math
```

Or for Flutter projects:

```bash
flutter pub add sunasty_color_math
```

---

## Quickstart Example

```dart
import 'package:sunasty_color_math/sunasty_color_math.dart';

void main() {
  // 1. Parse HEX to RGB
  final teal = hexToRgb('#0F766E');
  print('RGB: ${teal.r}, ${teal.g}, ${teal.b}');

  // 2. Convert to OKLCH perceptual space
  final oklch = rgbToOklch(teal.r, teal.g, teal.b);
  print('OKLCH: L=${oklch.l.toStringAsFixed(3)}, C=${oklch.c.toStringAsFixed(3)}, H=${oklch.h.toStringAsFixed(1)}°');

  // 3. Compute WCAG 2.1 contrast ratio against white
  final contrast = contrastRatioHex('#0F766E', '#FFFFFF');
  print('Contrast ratio: ${contrast.toStringAsFixed(2)}:1');
  print('Passes WCAG AA normal text: ${passesWcagAA(contrast)}');

  // 4. Auto-fix lightness to guarantee AA compliance
  final fixedHex = autoFixContrastForAA('#38BDF8', '#FFFFFF');
  print('Adjusted accessible color: $fixedHex');
}
```

---

## API Reference

### Color Conversions

- `RgbColor hexToRgb(String hex)`: Converts 3-digit or 6-digit hex string into RGB integers (`0..255`).
- `String rgbToHex(int r, int g, int b)`: Converts RGB integers into standard `#RRGGBB` format.
- `OklchColor rgbToOklch(int r, int g, int b)`: Converts sRGB into Oklab perceptual color space using Björn Ottosson's matrices.
- `RgbColor oklchToRgb(double l, double c, double h)`: Converts OKLCH coordinates back to sRGB with gamuting.

### Contrast & Accessibility

- `double relativeLuminance(int r, int g, int b)`: Implements IEC 61966-2-1 gamma linearization.
- `double contrastRatio(RgbColor foreground, RgbColor background)`: Returns contrast ratio from `1.0` to `21.0`.
- `double contrastRatioHex(String fgHex, String bgHex)`: Computes contrast ratio directly between two hex strings.
- `bool passesWcagAA(double ratio, {bool isLargeText = false})`: Checks 4.5:1 (normal) or 3.0:1 (large) threshold.
- `bool passesWcagAAA(double ratio, {bool isLargeText = false})`: Checks 7.0:1 (normal) or 4.5:1 (large) threshold.
- `String autoFixContrastForAA(String fgHex, String bgHex)`: Binary search optimizer on OKLCH lightness $L$ preserving hue and chroma.
