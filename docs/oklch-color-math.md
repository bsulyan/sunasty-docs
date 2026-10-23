# OKLCH & WCAG 2.1 Color Math

The `sunasty-color-math` library provides zero-dependency color space conversions and contrast calculations for web applications and CLI utilities.

## Core Concepts

### sRGB Gamma Linearization

The Web Content Accessibility Guidelines (WCAG 2.1) require relative luminance $L$ calculated from linearized sRGB color components:

$$
c_{\text{linear}} = \begin{cases}
\frac{c}{12.92} & \text{if } c \le 0.04045 \\
\left(\frac{c + 0.055}{1.055}\right)^{2.4} & \text{otherwise}
\end{cases}
$$

Relative luminance is computed using standard ITU-R BT.709 coefficients:

$$
L = 0.2126 \cdot R_{\text{linear}} + 0.7152 \cdot G_{\text{linear}} + 0.0722 \cdot B_{\text{linear}}
$$

### WCAG 2.1 Contrast Ratio

The contrast ratio between two relative luminances $L_1$ and $L_2$ (where $L_1 \ge L_2$) is:

$$
\text{Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05}
$$

- **WCAG AA Normal Text**: Minimum `4.5:1`
- **WCAG AA Large Text**: Minimum `3.0:1`
- **WCAG AAA Normal Text**: Minimum `7.0:1`
- **WCAG AAA Large Text**: Minimum `4.5:1`

## Implementations

- **JavaScript / TypeScript**: [`sunasty-color-math` on npm](https://www.npmjs.com/package/sunasty-color-math)
- **Python**: [`sunasty-color-math` on PyPI](https://pypi.org/project/sunasty-color-math/)
- **Rust**: [`sunasty-color-math` on crates.io](https://crates.io/crates/sunasty-color-math)
- **Go**: [`sunasty-color-math-go` on pkg.go.dev](https://pkg.go.dev/github.com/bsulyan/sunasty-color-math-go)
- **PHP**: [`sunasty/color-math` on Packagist](https://packagist.org/packages/sunasty/color-math)

## Interactive Tools

- [Sunasty Color Picker & Palette Extractor](https://sunasty.com/color-picker/)
- [Sunasty Developer Utilities](https://sunasty.com/dev/)
- [Ulysan Studio](https://ulysan.fr/)
