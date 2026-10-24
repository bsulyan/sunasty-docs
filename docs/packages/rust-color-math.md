# sunasty-color-math (Rust)

Official documentation for the `sunasty-color-math` crate published on [crates.io](https://crates.io/crates/sunasty-color-math). A zero-dependency, high-performance Rust library for sRGB color manipulation, IEC 61966-2-1 gamma linearization, relative luminance, and WCAG 2.1 accessibility contrast calculation.

Engineered for native performance in CLI tools, WebAssembly runtimes, server-side design token processors, and rendering pipelines. Developed for the [Sunasty Color Picker](https://sunasty.com/color-picker/) and the [Sunasty Developer Utilities](https://sunasty.com/dev/).

---

## Package Overview

- **crates.io**: [`sunasty-color-math` on crates.io](https://crates.io/crates/sunasty-color-math)
- **API Docs**: [docs.rs/sunasty-color-math](https://docs.rs/sunasty-color-math)
- **Source Code**: [GitHub bsulyan/sunasty-color-math-rs](https://github.com/bsulyan/sunasty-color-math-rs)
- **License**: MIT
- **Dependencies**: 0 external dependencies (pure standard library)
- **Edition**: Rust 2021

---

## Installation

Add to your `Cargo.toml`:

```toml
[dependencies]
sunasty-color-math = "1.0.0"
```

Or install via the `cargo` command line:

```bash
cargo add sunasty-color-math
```

---

## Complete API Reference

### 1. `Rgb` Struct

Represents an 8-bit sRGB color tuple with `Copy` and `Eq` semantics.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct Rgb {
    pub r: u8,
    pub g: u8,
    pub b: u8,
}
```

#### `Rgb::new(r: u8, g: u8, b: u8) -> Self`
Constant constructor creating a new `Rgb` instance.

```rust
use sunasty_color_math::Rgb;

const BRAND_TEAL: Rgb = Rgb::new(15, 118, 110);
```

#### `Rgb::from_hex(hex: &str) -> Result<Self, &'static str>`
Parses a 3-character (`#fff` or `fff`) or 6-character (`#0f766e` or `0f766e`) hexadecimal color string. Automatically trims whitespace and leading `#`.

Returns `Err("Invalid hex length")` or `Err("Invalid hex characters")` on failure.

```rust
use sunasty_color_math::Rgb;

let color = Rgb::from_hex("#0F766E").expect("valid hex");
assert_eq!(color, Rgb::new(15, 118, 110));
```

#### `Rgb::to_hex(&self) -> String`
Formats the color into a lowercase 6-character hex string with `#`.

```rust
use sunasty_color_math::Rgb;

let color = Rgb::new(15, 118, 110);
assert_eq!(color.to_hex(), "#0f766e");
```

---

### 2. Linearization & Luminance

#### `Rgb::linearize(val: u8) -> f64`
Transforms an 8-bit sRGB channel integer ($0 \le v \le 255$) into a linear light intensity value ($0.0 \le L \le 1.0$) based on IEC 61966-2-1:

$$\text{linear}(V) = \begin{cases} \frac{V}{12.92} & \text{if } V \le 0.04045 \\ \left(\frac{V + 0.055}{1.055}\right)^{2.4} & \text{if } V > 0.04045 \end{cases}$$

```rust
use sunasty_color_math::Rgb;

let lin = Rgb::linearize(118);
println!("Linear green: {:.4}", lin);
// => Linear green: 0.1784
```

#### `Rgb::relative_luminance(&self) -> f64`
Computes the WCAG 2.1 relative luminance ($0.0 \le Y \le 1.0$) using standard ITU-R BT.709 spectral weights:

$$Y = 0.2126 \cdot R_{\text{lin}} + 0.7152 \cdot G_{\text{lin}} + 0.0722 \cdot B_{\text{lin}}$$

```rust
use sunasty_color_math::Rgb;

let color = Rgb::new(15, 118, 110);
println!("Luminance: {:.4}", color.relative_luminance());
```

---

### 3. Contrast Calculation & Conformance

#### `Rgb::contrast_ratio(&self, other: &Self) -> f64`
Calculates the WCAG 2.1 contrast ratio between two colors ($1.0 \le \text{ratio} \le 21.0$). Automatically identifies the lighter and darker color:

$$\text{Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05} \quad (L_1 \ge L_2)$$

Returns the contrast ratio rounded to 2 decimal places.

```rust
use sunasty_color_math::Rgb;

let fg = Rgb::from_hex("#0F766E").unwrap();
let bg = Rgb::from_hex("#FFFFFF").unwrap();

let ratio = fg.contrast_ratio(&bg);
println!("Contrast ratio: {}:1", ratio);
// => Contrast ratio: 5.61:1
```

#### `WcagConformance` Struct

Holds the full evaluation report for WCAG 2.1 AA and AAA conformance:

```rust
#[derive(Debug, Clone, Copy, PartialEq)]
pub struct WcagConformance {
    pub ratio: f64,
    pub aa_normal: bool,  // >= 4.5
    pub aa_large: bool,   // >= 3.0
    pub aaa_normal: bool, // >= 7.0
    pub aaa_large: bool,  // >= 4.5
}
```

#### `WcagConformance::evaluate(ratio: f64) -> Self`
Evaluates a calculated contrast ratio against WCAG thresholds.

```rust
use sunasty_color_math::{Rgb, WcagConformance};

let fg = Rgb::from_hex("#0F766E").unwrap();
let bg = Rgb::from_hex("#FFFFFF").unwrap();

let conf = WcagConformance::evaluate(fg.contrast_ratio(&bg));
println!("Meets AA normal text: {}", conf.aa_normal);
println!("Meets AAA normal text: {}", conf.aaa_normal);
```

---

## Complete Example

```rust
use sunasty_color_math::{Rgb, WcagConformance};

fn main() {
    let text = Rgb::from_hex("#0f766e").expect("valid hex");
    let background = Rgb::new(255, 255, 255);

    let ratio = text.contrast_ratio(&background);
    let report = WcagConformance::evaluate(ratio);

    println!("Background: {}", background.to_hex());
    println!("Text:       {}", text.to_hex());
    println!("Contrast:   {}:1", report.ratio);
    println!("WCAG AA:    {}", if report.aa_normal { "PASS" } else { "FAIL" });
    println!("WCAG AAA:   {}", if report.aaa_normal { "PASS" } else { "FAIL" });
}
```

---

## Unit Testing with `cargo test`

```rust
#[cfg(test)]
mod tests {
    use sunasty_color_math::{Rgb, WcagConformance};

    #[test]
    fn test_perfect_contrast() {
        let black = Rgb::new(0, 0, 0);
        let white = Rgb::new(255, 255, 255);
        let ratio = black.contrast_ratio(&white);

        assert!((ratio - 21.0).abs() < 0.05);

        let conf = WcagConformance::evaluate(ratio);
        assert!(conf.aa_normal);
        assert!(conf.aaa_normal);
    }

    #[test]
    fn test_hex_normalization() {
        let short = Rgb::from_hex("fff").unwrap();
        let full = Rgb::from_hex("#ffffff").unwrap();
        assert_eq!(short, full);
        assert_eq!(full.to_hex(), "#ffffff");
    }
}
```

---

## Online Playground & Utilities

Explore color science and browser-side accessibility tools:
- [Sunasty Color Picker](https://sunasty.com/color-picker/)
- [Sunasty Contrast Checker](https://sunasty.com/contrast-checker/)
- [Sunasty Developer Utilities](https://sunasty.com/dev/)
- [Ulysan Studio](https://ulysan.fr/)
