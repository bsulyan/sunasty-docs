# sunasty-color-math (JavaScript & TypeScript)

Official documentation for the `sunasty-color-math` npm package. A zero-dependency, tree-shakeable utility library for sRGB, HSL, and OKLCH color space conversions, relative luminance, automated accessible palette generation, and WCAG 2.1 contrast calculations.

Originally developed for the [Sunasty Color Picker](https://sunasty.com/color-picker/) client-side tool and the [Sunasty Developer Suite](https://sunasty.com/dev/).

---

## Package Overview

- **npm Registry**: [`sunasty-color-math` on npmjs.com](https://www.npmjs.com/package/sunasty-color-math)
- **CDN (ESM)**: `https://cdn.jsdelivr.net/npm/sunasty-color-math@1.0.1/+esm`
- **Source Code**: [GitHub bsulyan/sunasty-color-math](https://github.com/bsulyan/sunasty-color-math)
- **License**: MIT
- **Bundle Size**: Zero external dependencies (< 4 kB minified + gzipped)
- **TypeScript**: First-class type definitions included (`index.d.ts`)

---

## Installation

```bash
# npm
npm install sunasty-color-math

# pnpm
pnpm add sunasty-color-math

# yarn
yarn add sunasty-color-math

# bun
bun add sunasty-color-math
```

### Direct Browser Import (ESM CDN)

```html
<script type="module">
  import { hexToRgb, computeContrastRatio } from 'https://cdn.jsdelivr.net/npm/sunasty-color-math@1.0.1/+esm';
  
  const contrast = computeContrastRatio('#0F766E', '#FFFFFF');
  console.log(`Contrast ratio: ${contrast.ratio}:1`);
</script>
```

---

## Complete API Reference

### 1. Color Parsing & Normalization

#### `hexToRgb(hex: string): RgbColor | null`
Parses a 3-character (`#fff`) or 6-character (`#ffffff`) hexadecimal color string. Tolerates leading whitespace and optional leading `#`. Returns `{ r: number, g: number, b: number }` with values between `0` and `255`, or `null` if invalid.

```typescript
import { hexToRgb } from 'sunasty-color-math';

const rgb = hexToRgb('#0F766E');
// => { r: 15, g: 118, b: 110 }
```

#### `rgbToHex(r: number, g: number, b: number): string`
Converts 8-bit RGB integers to a lowercase 6-digit hex string with `#`. Automatically clamps values to `[0, 255]`.

```typescript
import { rgbToHex } from 'sunasty-color-math';

const hex = rgbToHex(15, 118, 110);
// => "#0f766e"
```

#### `parseColorInput(input: string): ParsedColor | null`
Flexible parser supporting multiple color string notations:
- Hex: `#0F766E`, `0f766e`, `#fff`
- Functional RGB: `rgb(15, 118, 110)`
- Functional HSL: `hsl(175, 77%, 26%)`

---

### 2. Color Space Conversions

#### `rgbToOklch(r: number, g: number, b: number): OklchColor`
Transforms an 8-bit sRGB color into the cylindrical OKLCH perceptual space using Björn Ottosson's Oklab matrices.

Returns:
- `L`: Perceived lightness in $[0.0, 1.0]$.
- `C`: Chroma (saturation) in $[0.0, \approx 0.4]$.
- `h`: Hue angle in degrees in $[0.0, 360.0)$.

```typescript
import { rgbToOklch } from 'sunasty-color-math';

const oklch = rgbToOklch(15, 118, 110);
console.log(`L=${oklch.L.toFixed(3)}, C=${oklch.C.toFixed(3)}, h=${oklch.h.toFixed(1)}°`);
// => L=0.485, C=0.088, h=184.2°
```

#### `oklchToRgb(L: number, C: number, h: number): RgbColor`
Converts cylindrical OKLCH coordinates back into sRGB `{ r, g, b }` integers. Output channels are clamped to standard `0-255` display gamut.

#### `rgbToHsl(r: number, g: number, b: number): HslColor`
Converts 8-bit RGB to `{ h: 0..360, s: 0..100, l: 0..100 }`.

#### `hslToRgb(h: number, s: number, l: number): RgbColor`
Converts HSL back into 8-bit RGB channels.

---

### 3. Accessibility & WCAG 2.1 Contrast

#### `relativeLuminance(r: number, g: number, b: number): number`
Computes the WCAG 2.1 relative luminance $L \in [0.0, 1.0]$ using IEC 61966-2-1 inverse sRGB transfer functions and ITU-R BT.709 coefficients:

$$L = 0.2126 \cdot R_{\text{lin}} + 0.7152 \cdot G_{\text{lin}} + 0.0722 \cdot B_{\text{lin}}$$

```typescript
import { relativeLuminance } from 'sunasty-color-math';

const lum = relativeLuminance(15, 118, 110);
console.log(`Luminance: ${lum.toFixed(4)}`);
```

#### `computeContrastRatio(fgHex: string, bgHex: string): ContrastResult`
Calculates the exact contrast ratio between two hex colors and evaluates WCAG AA and AAA conformance.

```typescript
import { computeContrastRatio } from 'sunasty-color-math';

const result = computeContrastRatio('#0F766E', '#FFFFFF');
console.log(result);
// => {
//   ratio: 5.61,
//   aaNormal: true,  (>= 4.5)
//   aaLarge: true,   (>= 3.0)
//   aaaNormal: false, (>= 7.0)
//   aaaLarge: true   (>= 4.5)
// }
```

#### `autoFixContrastForAA(fgHex: string, bgHex: string, targetRatio: number = 4.5): string`
Performs an automated binary search in OKLCH perceptual lightness space to adjust the foreground color until it achieves the required contrast ratio against the background, preserving the original hue and saturation.

```typescript
import { autoFixContrastForAA } from 'sunasty-color-math';

// Automatically darken cyan on white to guarantee 4.5:1
const fixedColor = autoFixContrastForAA('#38BDF8', '#FFFFFF', 4.5);
console.log(`Accessible foreground: ${fixedColor}`);
```

---

### 4. Palette Generation & Tailwind Matching

#### `generateShadesScale(r: number, g: number, b: number): ShadeStop[]`
Generates a 10-step perceptual tonal palette (`50`, `100`, `200`, `300`, `400`, `500`, `600`, `700`, `800`, `900`, `950`) centered around the input color.

```typescript
import { generateShadesScale } from 'sunasty-color-math';

const scale = generateShadesScale(15, 118, 110);
scale.forEach(s => console.log(`${s.step}: ${s.hex}`));
```

#### `findClosestTailwindShade(r: number, g: number, b: number): TailwindMatch`
Computes the minimum Euclidean distance in Oklab space to find the closest match in the Tailwind CSS default palette:

```typescript
import { findClosestTailwindShade } from 'sunasty-color-math';

const match = findClosestTailwindShade(15, 118, 110);
console.log(`Closest Tailwind class: ${match.name} (${match.hex})`);
// => "teal-700 (#0f766e)"
```

#### `generateHarmonies(r: number, g: number, b: number): ColorHarmonies`
Generates color wheel harmonies (complementary $180^\circ$, analogous $\pm 30^\circ$, triadic $\pm 120^\circ$) using constant OKLCH lightness:

```typescript
import { generateHarmonies } from 'sunasty-color-math';

const harmonies = generateHarmonies(15, 118, 110);
console.log(`Complementary: ${harmonies.complementary}`);
```

---

## Interactive Playground

Test all functions interactively in your browser:
- [Sunasty Color Picker & Palette Extractor](https://sunasty.com/color-picker/)
- [Sunasty Contrast Checker](https://sunasty.com/contrast-checker/)
- [Sunasty Developer Utilities](https://sunasty.com/dev/)
- [Sunasty Color Picker Chrome Extension](https://github.com/bsulyan/sunasty-color-picker)
