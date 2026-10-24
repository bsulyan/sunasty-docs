# sunasty-color-math (Python)

Official documentation for the `sunasty-color-math` PyPI package. A zero-dependency, pure Python utility library for sRGB, OKLCH color space conversions, relative luminance calculation, and WCAG 2.1 accessibility contrast evaluation.

Designed for server-side image processing pipelines, CLI color generators, web accessibility audits, and UI design token automation. Developed for the [Sunasty Color Picker](https://sunasty.com/color-picker/) and the [Sunasty Developer Utilities](https://sunasty.com/dev/).

---

## Package Overview

- **PyPI Registry**: [`sunasty-color-math` on PyPI](https://pypi.org/project/sunasty-color-math/)
- **Source Code**: [GitHub bsulyan/sunasty-color-math](https://github.com/bsulyan/sunasty-color-math)
- **License**: MIT
- **Requirements**: Python `>= 3.8` (no external dependencies, standard library only)
- **Typing**: Fully typed with PEP 484 type annotations

---

## Installation

```bash
# pip
pip install sunasty-color-math

# poetry
poetry add sunasty-color-math

# uv
uv add sunasty-color-math

# pdm
pdm add sunasty-color-math
```

---

## Complete API Reference

All functions are exported directly at the top level of `sunasty_color_math`.

### 1. Hex and RGB Parsing

#### `hex_to_rgb(hex_str: str) -> Tuple[int, int, int]`
Parses a 3-character (`#fff`) or 6-character (`#ffffff`) hexadecimal color string. Strips leading `#` and whitespace. Returns an `(r, g, b)` tuple where each channel is an integer in $[0, 255]$.

Raises `ValueError` if the input is not a valid 3- or 6-character hex string.

```python
from sunasty_color_math import hex_to_rgb

r, g, b = hex_to_rgb("#0F766E")
print(r, g, b)
# => (15, 118, 110)
```

#### `rgb_to_hex(r: int, g: int, b: int) -> str`
Formats 8-bit integer RGB channels into a lowercase 6-digit hex string with a leading `#`. Channels are automatically clamped to $[0, 255]$.

```python
from sunasty_color_math import rgb_to_hex

hex_code = rgb_to_hex(15, 118, 110)
print(hex_code)
# => "#0f766e"
```

---

### 2. sRGB Gamma Linearization

#### `linearize_srgb(c_8bit: float) -> float`
Converts an 8-bit sRGB color channel ($0 \le c \le 255$) to linear light intensity ($0.0 \le L \le 1.0$) using the official IEC 61966-2-1 transfer function:

$$\text{linear}(V) = \begin{cases} \frac{V}{12.92} & \text{if } V \le 0.04045 \\ \left(\frac{V + 0.055}{1.055}\right)^{2.4} & \text{if } V > 0.04045 \end{cases}$$

where $V = \frac{c}{255}$.

```python
from sunasty_color_math import linearize_srgb

lin_green = linearize_srgb(118)
print(f"{lin_green:.4f}")
# => 0.1784
```

#### `delinearize_srgb(lin: float) -> int`
Converts a linear light value ($0.0 \le \text{lin} \le 1.0$) back to an 8-bit gamma-encoded integer channel ($0 \le c \le 255$).

---

### 3. OKLCH Perceptual Color Space

The OKLCH color space offers uniform perceptual linearity, separating perceived lightness from chroma and hue without the distortion artifacts seen in HSL.

#### `rgb_to_oklch(r: int, g: int, b: int) -> Tuple[float, float, float]`
Converts standard 8-bit sRGB channels into cylindrical OKLCH coordinates $(L, C, h)$:

- `L` (Lightness): Perceived brightness in $[0.0, 1.0]$.
- `C` (Chroma): Saturation/purity in $[0.0, \approx 0.4]$.
- `h` (Hue): Angle in degrees in $[0.0, 360.0)$.

```python
from sunasty_color_math import rgb_to_oklch

l, c, h = rgb_to_oklch(15, 118, 110)
print(f"Lightness: {l:.4f}, Chroma: {c:.4f}, Hue: {h:.2f}°")
# => Lightness: 0.4851, Chroma: 0.0882, Hue: 184.21°
```

#### `oklch_to_rgb(l_val: float, c_val: float, h_deg: float) -> Tuple[int, int, int]`
Converts cylindrical OKLCH coordinates back into standard 8-bit sRGB `(r, g, b)`. Output values are clamped to the displayable sRGB gamut ($[0, 255]$).

```python
from sunasty_color_math import oklch_to_rgb, rgb_to_hex

r, g, b = oklch_to_rgb(0.4851, 0.0882, 184.21)
print(rgb_to_hex(r, g, b))
# => "#0f766e"
```

---

### 4. WCAG 2.1 Contrast & Accessibility

#### `relative_luminance(r: int, g: int, b: int) -> float`
Calculates the WCAG 2.1 relative luminance of an sRGB color ($0.0$ for darkest black to $1.0$ for lightest white) using ITU-R BT.709 coefficients:

$$Y = 0.2126 \cdot R_{\text{lin}} + 0.7152 \cdot G_{\text{lin}} + 0.0722 \cdot B_{\text{lin}}$$

```python
from sunasty_color_math import relative_luminance

lum = relative_luminance(15, 118, 110)
print(f"Relative luminance: {lum:.4f}")
```

#### `contrast_ratio(color_a: Tuple[int, int, int], color_b: Tuple[int, int, int]) -> float`
Calculates the WCAG 2.1 contrast ratio between two sRGB tuples $(1.0 \le \text{ratio} \le 21.0)$:

$$\text{Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05} \quad (L_1 \ge L_2)$$

Returns the ratio rounded to 2 decimal places.

```python
from sunasty_color_math import hex_to_rgb, contrast_ratio

fg = hex_to_rgb("#0F766E")
bg = hex_to_rgb("#FFFFFF")

ratio = contrast_ratio(fg, bg)
print(f"Contrast ratio: {ratio}:1")
# => Contrast ratio: 5.61:1
```

#### `check_wcag(contrast: float) -> Dict[str, Any]`
Evaluates a contrast ratio against WCAG 2.1 AA and AAA thresholds for normal and large text.

Returns a dictionary:
```python
{
    "ratio": 5.61,
    "aa_normal": True,   # >= 4.5
    "aa_large": True,    # >= 3.0
    "aaa_normal": False, # >= 7.0
    "aaa_large": True    # >= 4.5
}
```

---

## Practical Examples

### Automated Batch WCAG Verification

```python
from sunasty_color_math import hex_to_rgb, contrast_ratio, check_wcag

pairs = [
    ("#0F766E", "#FFFFFF", "Primary Teal on White"),
    ("#94A3B8", "#FFFFFF", "Muted Gray on White"),
    ("#1E293B", "#F8FAFC", "Slate Dark on Slate Light"),
]

for fg_hex, bg_hex, label in pairs:
    fg = hex_to_rgb(fg_hex)
    bg = hex_to_rgb(bg_hex)
    ratio = contrast_ratio(fg, bg)
    report = check_wcag(ratio)
    
    status = "PASS" if report["aa_normal"] else "FAIL"
    print(f"[{status}] {label}: {ratio}:1 (AA: {report['aa_normal']}, AAA: {report['aaa_normal']})")
```

### Lightness Binary Search for Accessible Palette

```python
from sunasty_color_math import hex_to_rgb, rgb_to_hex, rgb_to_oklch, oklch_to_rgb, contrast_ratio

def make_accessible(fg_hex: str, bg_hex: str, target: float = 4.5) -> str:
    fg = hex_to_rgb(fg_hex)
    bg = hex_to_rgb(bg_hex)
    
    if contrast_ratio(fg, bg) >= target:
        return fg_hex
        
    l, c, h = rgb_to_oklch(*fg)
    bg_l, _, _ = rgb_to_oklch(*bg)
    
    # Decide direction: darken if background is light, lighten if dark
    darken = bg_l > 0.5
    low, high = (0.0, l) if darken else (l, 1.0)
    
    best_hex = fg_hex
    for _ in range(16):  # 16 binary search iterations
        mid = (low + high) / 2.0
        candidate_rgb = oklch_to_rgb(mid, c, h)
        candidate_ratio = contrast_ratio(candidate_rgb, bg)
        
        if candidate_ratio >= target:
            best_hex = rgb_to_hex(*candidate_rgb)
            if darken:
                low = mid
            else:
                high = mid
        else:
            if darken:
                high = mid
            else:
                low = mid
                
    return best_hex

print(make_accessible("#38BDF8", "#FFFFFF", 4.5))
# Automatically outputs an accessible darker cyan while preserving hue and chroma
```

---

## Unit Testing with Pytest

```python
# test_colors.py
import pytest
from sunasty_color_math import hex_to_rgb, rgb_to_hex, contrast_ratio, check_wcag

def test_hex_conversion():
    assert hex_to_rgb("#FFFFFF") == (255, 255, 255)
    assert hex_to_rgb("#000") == (0, 0, 0)
    assert rgb_to_hex(15, 118, 110) == "#0f766e"

def test_wcag_black_white():
    black = hex_to_rgb("#000000")
    white = hex_to_rgb("#FFFFFF")
    ratio = contrast_ratio(black, white)
    assert ratio == 21.0
    
    report = check_wcag(ratio)
    assert report["aa_normal"] is True
    assert report["aaa_normal"] is True

def test_invalid_hex():
    with pytest.raises(ValueError):
        hex_to_rgb("invalid")
```

---

## Interactive Playground & Web Utilities

Experiment with color spaces and accessibility calculations online:
- [Sunasty Color Picker & Palette Extractor](https://sunasty.com/color-picker/)
- [Sunasty Contrast Checker](https://sunasty.com/contrast-checker/)
- [Sunasty Developer Suite](https://sunasty.com/dev/)
- [Ulysan Studio](https://ulysan.fr/)
