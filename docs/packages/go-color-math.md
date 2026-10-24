# sunasty-color-math-go (Go)

Official documentation for the `github.com/bsulyan/sunasty-color-math-go` package. A zero-dependency Go module for sRGB and OKLCH color space conversions, relative luminance calculation, and WCAG 2.1 accessibility contrast verification.

Engineered for high-throughput microservices, CLI applications, automated accessibility linters, and design token pipelines. Developed for the [Sunasty Color Picker](https://sunasty.com/color-picker/) and the [Sunasty Developer Utilities](https://sunasty.com/dev/).

---

## Package Overview

- **Go Package**: `github.com/bsulyan/sunasty-color-math-go`
- **Documentation**: [pkg.go.dev/github.com/bsulyan/sunasty-color-math-go](https://pkg.go.dev/github.com/bsulyan/sunasty-color-math-go)
- **Source Code**: [GitHub bsulyan/sunasty-color-math-go](https://github.com/bsulyan/sunasty-color-math-go)
- **License**: MIT
- **Dependencies**: 0 external dependencies (Go standard library only)
- **Go Version**: `>= 1.21`

---

## Installation

```bash
go get github.com/bsulyan/sunasty-color-math-go
```

Import in your Go files:

```go
import "github.com/bsulyan/sunasty-color-math-go"
```

---

## Complete API Reference

### 1. Data Structures

#### `type RGB struct`
Represents an 8-bit sRGB color tuple.

```go
type RGB struct {
    R uint8
    G uint8
    B uint8
}
```

#### `type OKLCH struct`
Represents a cylindrical OKLCH color coordinate.

```go
type OKLCH struct {
    L float64 // Lightness [0.0, 1.0]
    C float64 // Chroma [0.0, ~0.4]
    H float64 // Hue in degrees [0.0, 360.0)
}
```

#### `type WCAGConformance struct`
Holds conformance flags for WCAG 2.1 criteria, with built-in JSON tags.

```go
type WCAGConformance struct {
    Ratio     float64 `json:"ratio"`
    AANormal  bool    `json:"aa_normal"`
    AALarge   bool    `json:"aa_large"`
    AAANormal bool    `json:"aaa_normal"`
    AAALarge  bool    `json:"aaa_large"`
}
```

---

### 2. Color Parsing & Formatting

#### `ParseHex(hexStr string) (RGB, error)`
Parses a 3-character (`#fff` or `fff`) or 6-character (`#0f766e` or `0f766e`) hexadecimal color string. Strips leading `#` and whitespace.

```go
package main

import (
    "fmt"
    "github.com/bsulyan/sunasty-color-math-go"
)

func main() {
    color, err := colormath.ParseHex("#0F766E")
    if err != nil {
        panic(err)
    }
    fmt.Printf("RGB: (%d, %d, %d)\n", color.R, color.G, color.B)
    // => RGB: (15, 118, 110)
}
```

#### `(c RGB) ToHex() string`
Formats an `RGB` struct as a lowercase 6-digit hex string prefixed with `#`.

```go
c := colormath.RGB{R: 15, G: 118, B: 110}
fmt.Println(c.ToHex())
// => "#0f766e"
```

---

### 3. Gamma Linearization & Relative Luminance

#### `Linearize(val uint8) float64`
Applies the IEC 61966-2-1 inverse transfer function to transform an 8-bit sRGB channel into linear light intensity:

$$\text{linear}(V) = \begin{cases} \frac{V}{12.92} & \text{if } V \le 0.04045 \\ \left(\frac{V + 0.055}{1.055}\right)^{2.4} & \text{if } V > 0.04045 \end{cases}$$

```go
lin := colormath.Linearize(118)
fmt.Printf("Linear: %.4f\n", lin)
// => Linear: 0.1784
```

#### `RelativeLuminance(c RGB) float64`
Calculates the WCAG 2.1 relative luminance using ITU-R BT.709 coefficients:

$$Y = 0.2126 \cdot R_{\text{lin}} + 0.7152 \cdot G_{\text{lin}} + 0.0722 \cdot B_{\text{lin}}$$

```go
c := colormath.RGB{R: 15, G: 118, B: 110}
lum := colormath.RelativeLuminance(c)
fmt.Printf("Relative luminance: %.4f\n", lum)
```

---

### 4. WCAG 2.1 Contrast Calculation

#### `ContrastRatio(a, b RGB) float64`
Calculates the exact contrast ratio between two sRGB colors $(1.0 \le \text{ratio} \le 21.0)$. Automatically determines the lighter and darker luminance:

$$\text{Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05} \quad (L_1 \ge L_2)$$

Returns the contrast ratio rounded to 2 decimal places.

```go
fg, _ := colormath.ParseHex("#0F766E")
bg, _ := colormath.ParseHex("#FFFFFF")

ratio := colormath.ContrastRatio(fg, bg)
fmt.Printf("Contrast: %.2f:1\n", ratio)
// => Contrast: 5.61:1
```

#### `CheckWCAG(ratio float64) WCAGConformance`
Evaluates a contrast ratio against WCAG 2.1 AA and AAA thresholds:
- Normal text AA: $\ge 4.5$
- Large text AA: $\ge 3.0$
- Normal text AAA: $\ge 7.0$
- Large text AAA: $\ge 4.5$

```go
conf := colormath.CheckWCAG(ratio)
fmt.Printf("Passes AA Normal: %t\n", conf.AANormal)
fmt.Printf("Passes AAA Normal: %t\n", conf.AAANormal)
```

---

## Practical Example: HTTP Handler for Contrast Audits

```go
package main

import (
    "encoding/json"
    "net/http"
    "github.com/bsulyan/sunasty-color-math-go"
)

type AuditRequest struct {
    Foreground string `json:"foreground"`
    Background string `json:"background"`
}

func handleAudit(w http.ResponseWriter, r *http.Request) {
    var req AuditRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    fg, err := colormath.ParseHex(req.Foreground)
    if err != nil {
        http.Error(w, "invalid foreground hex", http.StatusBadRequest)
        return
    }

    bg, err := colormath.ParseHex(req.Background)
    if err != nil {
        http.Error(w, "invalid background hex", http.StatusBadRequest)
        return
    }

    ratio := colormath.ContrastRatio(fg, bg)
    report := colormath.CheckWCAG(ratio)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(report)
}

func main() {
    http.HandleFunc("/api/contrast", handleAudit)
    http.ListenAndServe(":8080", nil)
}
```

---

## Unit Testing with `go test`

```go
// colormath_test.go
package main

import (
    "testing"
    "github.com/bsulyan/sunasty-color-math-go"
)

func TestContrastCalculation(t *testing.T) {
    white := colormath.RGB{R: 255, G: 255, B: 255}
    black := colormath.RGB{R: 0, G: 0, B: 0}

    ratio := colormath.ContrastRatio(white, black)
    if ratio != 21.0 {
        t.Fatalf("expected 21.0, got %f", ratio)
    }

    conf := colormath.CheckWCAG(ratio)
    if !conf.AANormal || !conf.AAANormal {
        t.Fatalf("expected pass for AA and AAA, got %+v", conf)
    }
}

func TestHexParsing(t *testing.T) {
    c, err := colormath.ParseHex("#0f766e")
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
    if c.R != 15 || c.G != 118 || c.B != 110 {
        t.Fatalf("mismatched RGB channels: %+v", c)
    }
    if c.ToHex() != "#0f766e" {
        t.Fatalf("expected #0f766e, got %s", c.ToHex())
    }
}
```

---

## Online Playground & Utilities

Explore color tools in the browser:
- [Sunasty Color Picker](https://sunasty.com/color-picker/)
- [Sunasty Contrast Checker](https://sunasty.com/contrast-checker/)
- [Sunasty Developer Utilities](https://sunasty.com/dev/)
- [Ulysan Studio](https://ulysan.fr/)
