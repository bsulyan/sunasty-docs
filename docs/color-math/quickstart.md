# Multi-Language Quickstart

The `sunasty-color-math` algorithm is published in five official language implementations, all sharing identical mathematical results and zero runtime dependencies.

---

## 1. JavaScript & TypeScript

Published on npm as [`sunasty-color-math`](https://www.npmjs.com/package/sunasty-color-math). Compatible with Node.js, Bun, Deno, and modern browsers.

### Installation
```bash
npm install sunasty-color-math
```

### Usage Example
```typescript
import { 
  hexToRgb, 
  rgbToHex, 
  relativeLuminance, 
  contrastRatio, 
  checkWcag, 
  rgbToOklch, 
  oklchToRgb 
} from 'sunasty-color-math';

// 1. Parse Hex Colors
const brandTeal = hexToRgb('#0F766E'); // [15, 118, 110]
const paperWhite = hexToRgb('#F8F7F4'); // [248, 247, 244]

// 2. Relative Luminance
const l1 = relativeLuminance(brandTeal[0], brandTeal[1], brandTeal[2]);
console.log(`Brand luminance: ${l1.toFixed(4)}`);

// 3. WCAG 2.1 Contrast Ratio
const ratio = contrastRatio(brandTeal, paperWhite);
console.log(`Contrast Ratio: ${ratio}:1`);

// 4. Verify Accessibility Compliance
const wcag = checkWcag(ratio);
console.log(`Passes AA Normal Text (>= 4.5): ${wcag.aa_normal}`);
console.log(`Passes AAA Large Text (>= 4.5): ${wcag.aaa_large}`);

// 5. Convert to OKLCH
const [l, c, h] = rgbToOklch(brandTeal[0], brandTeal[1], brandTeal[2]);
console.log(`OKLCH: L=${l.toFixed(3)}, C=${c.toFixed(3)}, H=${h.toFixed(1)}°`);
```

Interactive tool: [Sunasty Color Picker](https://sunasty.com/color-picker/)

---

## 2. Python

Published on PyPI as [`sunasty-color-math`](https://pypi.org/project/sunasty-color-math/). Tested on Python 3.8 through 3.13+.

### Installation
```bash
pip install sunasty-color-math
```

### Usage Example
```python
from sunasty_color_math import (
    hex_to_rgb, 
    rgb_to_hex, 
    relative_luminance, 
    contrast_ratio, 
    check_wcag, 
    rgb_to_oklch
)

# 1. Parse colors
teal = hex_to_rgb("#0F766E")   # (15, 118, 110)
white = hex_to_rgb("#FFFFFF")  # (255, 255, 255)

# 2. Compute WCAG Contrast
ratio = contrast_ratio(teal, white)
print(f"Contrast: {ratio}:1")

# 3. Check WCAG compliance levels
status = check_wcag(ratio)
print(f"WCAG AA Normal Text: {status['aa_normal']}")
print(f"WCAG AAA Normal Text: {status['aaa_normal']}")

# 4. Convert to OKLCH perceptual space
lightness, chroma, hue = rgb_to_oklch(*teal)
print(f"OKLCH Lightness: {lightness:.3f}, Chroma: {chroma:.3f}, Hue: {hue:.1f}°")
```

---

## 3. Rust

Published on crates.io as [`sunasty-color-math`](https://crates.io/crates/sunasty-color-math). Zero external dependencies, `#![no_std]` compatible core.

### Installation
Add to your `Cargo.toml`:
```toml
[dependencies]
sunasty-color-math = "1.0.0"
```

### Usage Example
```rust
use sunasty_color_math::{Rgb, WcagConformance};

fn main() {
    // 1. Parse Hex Strings
    let teal = Rgb::from_hex("#0F766E").expect("Valid hex string");
    let white = Rgb::new(255, 255, 255);

    // 2. Relative Luminance
    let lum = teal.relative_luminance();
    println!("Teal Luminance: {:.4}", lum);

    // 3. Contrast Ratio
    let ratio = teal.contrast_ratio(&white);
    println!("Contrast Ratio: {:.2}:1", ratio);

    // 4. Conformance evaluation
    let conf = WcagConformance::evaluate(ratio);
    println!("WCAG AA Normal: {}", conf.aa_normal);
    println!("WCAG AAA Large: {}", conf.aaa_large);
}
```

Documentation: [docs.rs/sunasty-color-math](https://docs.rs/sunasty-color-math)

---

## 4. Go

Published as [`github.com/bsulyan/sunasty-color-math-go`](https://pkg.go.dev/github.com/bsulyan/sunasty-color-math-go). Standard Go module.

### Installation
```bash
go get github.com/bsulyan/sunasty-color-math-go
```

### Usage Example
```go
package main

import (
	"fmt"
	colormath "github.com/bsulyan/sunasty-color-math-go"
)

func main() {
	// 1. Parse hex color
	teal, err := colormath.HexToRGB("#0F766E")
	if err != nil {
		panic(err)
	}
	white := colormath.RGB{R: 255, G: 255, B: 255}

	// 2. Calculate WCAG 2.1 Contrast
	ratio := colormath.ContrastRatio(teal, white)
	fmt.Printf("Contrast Ratio: %.2f:1\n", ratio)

	// 3. Check Compliance
	conf := colormath.CheckWCAG(ratio)
	fmt.Printf("Passes AA Normal: %v\n", conf.AANormal)
	fmt.Printf("Passes AAA Normal: %v\n", conf.AAANormal)
}
```

Go Documentation: [pkg.go.dev](https://pkg.go.dev/github.com/bsulyan/sunasty-color-math-go)

---

## 5. PHP

Published on Packagist as [`sunasty/color-math`](https://packagist.org/packages/sunasty/color-math). Requires PHP 8.1+.

### Installation
```bash
composer require sunasty/color-math
```

### Usage Example
```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

use Sunasty\ColorMath\ColorMath;

// 1. Parse Hex Colors
$teal = ColorMath::hexToRgb('#0F766E'); // [15, 118, 110]
$white = [255, 255, 255];

// 2. Relative Luminance
$lum = ColorMath::relativeLuminance($teal[0], $teal[1], $teal[2]);
echo "Relative Luminance: " . round($lum, 4) . "\n";

// 3. Contrast Ratio
$ratio = ColorMath::contrastRatio($teal, $white);
echo "Contrast Ratio: {$ratio}:1\n";

// 4. Accessibility Conformance
$wcag = ColorMath::checkWcag($ratio);
echo "Passes WCAG AA: " . ($wcag['aa_normal'] ? 'Yes' : 'No') . "\n";
```

Package link: [packagist.org/packages/sunasty/color-math](https://packagist.org/packages/sunasty/color-math)
