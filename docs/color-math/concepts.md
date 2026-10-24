# Color Mathematics & Accessibility Concepts

The `sunasty-color-math` family of libraries provides mathematically rigorous color space conversions and accessibility calculations. This section covers the physical and mathematical foundations behind sRGB linearization, WCAG 2.1 relative luminance, and the OKLCH perceptual color model.

## 1. sRGB Gamma Linearization

Digital displays store colors in the standard RGB (sRGB) color space using an 8-bit non-linear transfer function (gamma curve). To perform physically accurate optical calculations—such as computing light luminance or blending colors—the non-linear sRGB values must first be linearized.

Given an 8-bit channel value $C_{\text{srgb}} \in [0, 255]$:

1. Normalize to the unit interval:
   $$c = \frac{C_{\text{srgb}}}{255.0}$$

2. Apply the inverse sRGB electro-optical transfer function (EOTF) as specified in IEC 61966-2-1:
   $$c_{\text{linear}} = \begin{cases}
   \frac{c}{12.92} & \text{if } c \le 0.04045 \\
   \left(\frac{c + 0.055}{1.055}\right)^{2.4} & \text{if } c > 0.04045
   \end{cases}$$

## 2. WCAG 2.1 Relative Luminance

The Web Content Accessibility Guidelines (WCAG 2.1, Success Criterion 1.4.3 and 1.4.6) define relative luminance $L$ as the relative brightness of any point in a colorspace, normalized to $0.0$ for the darkest black and $1.0$ for the lightest white.

Using the linearized components $R_{\text{linear}}, G_{\text{linear}}, B_{\text{linear}}$ and standard ITU-R BT.709 chromaticity coefficients:

$$L = 0.2126 \cdot R_{\text{linear}} + 0.7152 \cdot G_{\text{linear}} + 0.0722 \cdot B_{\text{linear}}$$

Human vision is disproportionately sensitive to green light ($71.52\%$) compared to red ($21.26\%$) and blue ($7.22\%$).

## 3. WCAG 2.1 Contrast Ratio

Given two colors with relative luminances $L_1$ and $L_2$ (where $L_1$ is the lighter color and $L_2$ is the darker color):

$$\text{Contrast Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05}$$

The constant offset $+0.05$ accounts for ambient light reflecting off the display surface. The contrast ratio ranges strictly from $1.0:1$ (identical colors) to $21.0:1$ (pure black on pure white).

### WCAG Conformance Levels

| Standard | Element Type | Minimum Ratio | Description |
| :--- | :--- | :---: | :--- |
| **WCAG AA** | Normal Text | **4.5:1** | Body text under 18pt (or 14pt bold) |
| **WCAG AA** | Large Text | **3.0:1** | Headings 18pt+ or bold text 14pt+ |
| **WCAG AA** | UI Components & Graphics | **3.0:1** | Input borders, icons, focus indicators |
| **WCAG AAA** | Normal Text | **7.0:1** | Enhanced contrast for high readability |
| **WCAG AAA** | Large Text | **4.5:1** | Enhanced contrast for large headings |

Test these calculations live with the [Sunasty Contrast Checker](https://sunasty.com/contrast-checker/) and the [Sunasty Color Picker](https://sunasty.com/color-picker/).

## 4. The OKLCH Perceptual Color Model

While HSL and sRGB are hardware-oriented models, they fail to represent human perception. In HSL, pure yellow (`hsl(60, 100%, 50%)`) and pure blue (`hsl(240, 100%, 50%)`) have the exact same lightness value ($50\%$), yet yellow has a perceived luminance of $\approx 0.93$ while blue has $\approx 0.07$.

**OKLCH** (designed by Björn Ottosson in 2020) solves this problem by providing a cylindrical representation of the Oklab color space with:
- **$L$ (Perceived Lightness)**: Normalized from $0.0$ (black) to $1.0$ (white). Colors with the same $L$ value appear equally bright to the human eye, regardless of hue.
- **$C$ (Chroma)**: Purity or saturation of the color, ranging from $0.0$ (achromatic gray) to $\approx 0.4$.
- **$H$ (Hue)**: Angle in degrees from $0^\circ$ to $360^\circ$ ($0^\circ$ is pink/red, $90^\circ$ is yellow, $140^\circ$ is green, $240^\circ$ is blue).

```
   sRGB (Linear) ───[ M1 ]───> LMS cone responses
                                   │
                               [ x^(1/3) ] non-linear compression
                                   │
   Oklab (L, a, b) <───[ M2 ]──────┘
        │
   [ Polar transform: C = sqrt(a² + b²), H = atan2(b, a) ]
        │
   OKLCH (L, C, H)
```

Because of its perceptual uniformity, OKLCH allows automated palette generation where accessible contrast ratios can be guaranteed programmatically simply by manipulating the $L$ coordinate.

Explore OKLCH conversions in your browser on the [Sunasty Color Picker & Palette Extractor](https://sunasty.com/color-picker/) and read the developer notes on the [Sunasty Developer Suite](https://sunasty.com/dev/).
