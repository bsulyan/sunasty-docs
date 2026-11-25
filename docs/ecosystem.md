# Sunasty Ecosystem & Registries

The Sunasty developer ecosystem spans 233 client-side web tools across 32 languages, open-source MIT packages on 7 registries, Manifest V3 browser extensions, technical articles, and verified directory profiles.

---

## Package Registries

All core mathematical, sanitization, barcode, color, and QR parsing engines are open-sourced under the MIT license:

| Platform | Package Identifier | Direct Link | Notes / Docs |
| :--- | :--- | :--- | :--- |
| **npm (HAR Sanitizer)** | `har-clean` | [npmjs.com/package/har-clean](https://www.npmjs.com/package/har-clean) | [Technical Note](https://bsulyan.github.io/notes/client-side-har-sanitization/) |
| **JSR (HAR Sanitizer)** | `@sunasty/har-sanitizer` | [jsr.io/@sunasty/har-sanitizer](https://jsr.io/@sunasty/har-sanitizer) | [Web Tool](https://sunasty.com/har-sanitizer/) |
| **npm (Tournament Math)** | `swiss-math` | [npmjs.com/package/swiss-math](https://www.npmjs.com/package/swiss-math) | [Technical Note](https://bsulyan.github.io/notes/swiss-tournament-tiebreak-math/) |
| **JSR (Tournament Math)** | `@sunasty/tournament-math` | [jsr.io/@sunasty/tournament-math](https://jsr.io/@sunasty/tournament-math) | [Web Tool](https://sunasty.com/tournament-calculator/) |
| **npm (Code 128 SVG)** | `barcode128-svg` | [npmjs.com/package/barcode128-svg](https://www.npmjs.com/package/barcode128-svg) | [Technical Note](https://bsulyan.github.io/notes/code128-vector-svg-encoding/) |
| **JSR (Code 128 SVG)** | `@sunasty/code128-svg` | [jsr.io/@sunasty/code128-svg](https://jsr.io/@sunasty/code128-svg) | [Web Tool](https://sunasty.com/barcode-generator/) |
| **npm (Color Math)** | `oklch-wcag` | [npmjs.com/package/oklch-wcag](https://www.npmjs.com/package/oklch-wcag) | [API Reference](color-math/quickstart.md#1-javascript-typescript) |
| **JSR (Color Math)** | `@sunasty/color-math` | [jsr.io/@sunasty/color-math](https://jsr.io/@sunasty/color-math) | [Web Tool](https://sunasty.com/color-picker/) |
| **npm (QR Payloads)** | `qr-payload-parser` | [npmjs.com/package/qr-payload-parser](https://www.npmjs.com/package/qr-payload-parser) | [API Reference](qr/payload-schemas.md) |
| **JSR (QR Payloads)** | `@sunasty/qr-payload` | [jsr.io/@sunasty/qr-payload](https://jsr.io/@sunasty/qr-payload) | [Web Tool](https://sunasty.com/qr-scan/) |
| **pub.dev (Dart / Flutter)** | `sunasty_color_math` | [pub.dev/packages/sunasty_color_math](https://pub.dev/packages/sunasty_color_math) | [Dart Guide](packages/dart-color-math.md) |
| **PyPI (Python)** | `sunasty-color-math` | [pypi.org/project/sunasty-color-math/](https://pypi.org/project/sunasty-color-math/) | [Python Guide](color-math/quickstart.md#2-python) |
| **crates.io (Rust)** | `sunasty-color-math` | [crates.io/crates/sunasty-color-math](https://crates.io/crates/sunasty-color-math) | [docs.rs/sunasty-color-math](https://docs.rs/sunasty-color-math/latest/sunasty_color_math/) |
| **Go Packages** | `sunasty-color-math-go` | [pkg.go.dev/github.com/bsulyan/sunasty-color-math-go](https://pkg.go.dev/github.com/bsulyan/sunasty-color-math-go) | [Go Reference](color-math/quickstart.md#4-go) |
| **Packagist (PHP)** | `sunasty/color-math` | [packagist.org/packages/sunasty/color-math](https://packagist.org/packages/sunasty/color-math) | [PHP Guide](color-math/quickstart.md#5-php) |
| **Read the Docs** | `sunasty-docs` | [sunasty-docs.readthedocs.io](https://sunasty-docs.readthedocs.io/) | [Documentation Hub](index.md) |

---

## Browser Extensions (Manifest V3)

Engineered for Chrome, Firefox, and Edge with zero external network requests:

- **Sunasty QR Code Scanner & Generator**: Capture visible tabs, scan multi-QR codes with overlapping canvas tiles, and generate vector codes:
  - [Chrome Web Store](https://chromewebstore.google.com/detail/sunasty-qr-code-scanner-g/fbiighpklpogdbnhdapbiijcadhlijcb)
  - [Firefox Add-ons (AMO)](https://addons.mozilla.org/en-US/firefox/addon/sunasty-qr-code-scanner-genera/) ([FR](https://addons.mozilla.org/fr/firefox/addon/sunasty-qr-code-scanner-genera/))
  - [Microsoft Edge Add-ons](https://microsoftedge.microsoft.com/addons/detail/sunasty-qr-code-scanner-generator/npbppaegpbeefpfaffjgphjecmgbngab)
- **[Sunasty Color Picker & OKLCH Eyedropper](https://github.com/bsulyan/sunasty-color-picker)**: Pick screen pixels, convert sRGB to OKLCH, and test WCAG 2.1 contrast live.
  - [Microsoft Edge Add-ons](https://microsoftedge.microsoft.com/addons/detail/jcjfhbbmbnehefadhmhhegoccbdcfkgm)

---

## Web Applications, Directories & Tool Suites

Explore all 233 client-side utilities at [sunasty.com](https://sunasty.com/) or on [AlternativeTo](https://alternativeto.net/software/sunasty/about/) and [SaaSHub](https://www.saashub.com/sunasty):

- **[Color & Palette Tools](https://sunasty.com/color-picker/)** & **[Contrast Checker](https://sunasty.com/contrast-checker/)**
- **[QR Code](https://sunasty.com/qr-scan/)** & **[Barcode Generator](https://sunasty.com/barcode-generator/)**
- **[PDF Utilities](https://sunasty.com/pdf/)** & **[TCG Proxy Print](https://sunasty.com/tcg-proxy-print/)**
- **[Image Privacy, EXIF & C2PA Inspector](https://sunasty.com/exif-reader/)**
- **[HAR Sanitizer](https://sunasty.com/har-sanitizer/)**, **[Parquet Viewer](https://sunasty.com/parquet-viewer/)**, & **[SQLite Viewer](https://sunasty.com/sqlite-viewer/)**
- **[Swiss Tournament Calculator](https://sunasty.com/tournament-calculator/)** & **[Audio / Video Suite](https://sunasty.com/audio/)**

---

## Authorship & Studio

Developed and maintained by **Bastien Sulyan** at **[Ulysan Studio](https://ulysan.fr/)** (Strasbourg, France). Technical articles and implementation notes are published on [bsulyan.github.io](https://bsulyan.github.io/) and [dev.to/bsulyan](https://dev.to/bsulyan).
