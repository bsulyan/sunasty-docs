# sunasty/color-math (PHP)

Official documentation for the `sunasty/color-math` Composer package published on [Packagist](https://packagist.org/packages/sunasty/color-math). A zero-dependency PHP 8.1+ library for sRGB hex parsing, IEC 61966-2-1 gamma linearization, relative luminance, and WCAG 2.1 contrast calculation.

Engineered for server-side template engines (Blade, Twig), CMS accessibility audits (WordPress, Drupal), and automated design system token generators. Developed for the [Sunasty Color Picker](https://sunasty.com/color-picker/) and the [Sunasty Developer Utilities](https://sunasty.com/dev/).

---

## Package Overview

- **Packagist**: [`sunasty/color-math` on Packagist](https://packagist.org/packages/sunasty/color-math)
- **Source Code**: [GitHub bsulyan/sunasty-color-math-php](https://github.com/bsulyan/sunasty-color-math-php)
- **License**: MIT
- **Dependencies**: 0 external dependencies
- **PHP Requirements**: PHP `>= 8.1` (supports PHP 8.1, 8.2, 8.3, 8.4)
- **PSR-4 Autoloading**: `Sunasty\ColorMath\` mapped to `src/`

---

## Installation

```bash
composer require sunasty/color-math
```

---

## Complete API Reference

All methods are static members of the `Sunasty\ColorMath\ColorMath` class.

### 1. Hex and RGB Conversions

#### `ColorMath::hexToRgb(string $hex): array{0: int, 1: int, 2: int}`
Parses a 3-character (`#fff` or `fff`) or 6-character (`#0f766e` or `0f766e`) hexadecimal color string. Automatically trims whitespace and leading `#`.

Returns an indexed 3-element integer array `[red, green, blue]` in $[0, 255]$.

Throws `\InvalidArgumentException` if the hex string is malformed.

```php
use Sunasty\ColorMath\ColorMath;

[$r, $g, $b] = ColorMath::hexToRgb('#0F766E');
echo "R: $r, G: $g, B: $b\n";
// => R: 15, G: 118, B: 110
```

#### `ColorMath::rgbToHex(int $r, int $g, int $b): string`
Formats 8-bit integer RGB channels into a lowercase 6-digit hex string with leading `#`. Channels are automatically clamped to $[0, 255]$.

```php
use Sunasty\ColorMath\ColorMath;

$hex = ColorMath::rgbToHex(15, 118, 110);
echo $hex;
// => "#0f766e"
```

---

### 2. Gamma Linearization & Luminance

#### `ColorMath::linearize(float $channel8bit): float`
Converts an 8-bit sRGB color channel value ($0 \le c \le 255$) into a linear light intensity value ($0.0 \le L \le 1.0$) based on the official IEC 61966-2-1 specification:

$$\text{linear}(V) = \begin{cases} \frac{V}{12.92} & \text{if } V \le 0.04045 \\ \left(\frac{V + 0.055}{1.055}\right)^{2.4} & \text{if } V > 0.04045 \end{cases}$$

```php
use Sunasty\ColorMath\ColorMath;

$lin = ColorMath::linearize(118);
echo round($lin, 4);
// => 0.1784
```

#### `ColorMath::relativeLuminance(int $r, int $g, int $b): float`
Calculates the WCAG 2.1 relative luminance ($0.0 \le Y \le 1.0$) using standard ITU-R BT.709 spectral weights:

$$Y = 0.2126 \cdot R_{\text{lin}} + 0.7152 \cdot G_{\text{lin}} + 0.0722 \cdot B_{\text{lin}}$$

```php
use Sunasty\ColorMath\ColorMath;

$lum = ColorMath::relativeLuminance(15, 118, 110);
echo round($lum, 4);
```

---

### 3. Contrast Ratio & WCAG Evaluation

#### `ColorMath::contrastRatio(array $colorA, array $colorB): float`
Computes the WCAG 2.1 contrast ratio between two `[r, g, b]` arrays ($1.0 \le \text{ratio} \le 21.0$). Automatically orders the lighter and darker luminance:

$$\text{Ratio} = \frac{L_1 + 0.05}{L_2 + 0.05} \quad (L_1 \ge L_2)$$

Returns the contrast ratio rounded to 2 decimal places.

```php
use Sunasty\ColorMath\ColorMath;

$fg = ColorMath::hexToRgb('#0F766E');
$bg = ColorMath::hexToRgb('#FFFFFF');

$ratio = ColorMath::contrastRatio($fg, $bg);
echo "Contrast ratio: {$ratio}:1\n";
// => Contrast ratio: 5.61:1
```

#### `ColorMath::checkWcag(float $contrast): array`
Evaluates a contrast ratio against WCAG 2.1 AA and AAA thresholds.

Returns an associative array:
```php
[
    'ratio' => 5.61,
    'aa_normal' => true,   // >= 4.5
    'aa_large' => true,    // >= 3.0
    'aaa_normal' => false, // >= 7.0
    'aaa_large' => true,   // >= 4.5
]
```

```php
use Sunasty\ColorMath\ColorMath;

$report = ColorMath::checkWcag($ratio);
if (!$report['aa_normal']) {
    throw new \Exception('Foreground color does not satisfy WCAG AA contrast.');
}
```

---

## Framework Integration Examples

### Blade / Laravel View Helper

```php
namespace App\Support;

use Sunasty\ColorMath\ColorMath;

class PaletteHelper
{
    public static function getAccessibleTextColor(string $bgHex): string
    {
        $bg = ColorMath::hexToRgb($bgHex);
        $white = [255, 255, 255];
        $dark = [17, 24, 39]; // Gray 900

        $contrastWhite = ColorMath::contrastRatio($bg, $white);
        $contrastDark = ColorMath::contrastRatio($bg, $dark);

        return $contrastWhite >= $contrastDark ? '#ffffff' : '#111827';
    }
}
```

In your Blade template:

```blade
<div style="background-color: {{ $badgeColor }}; color: {{ \App\Support\PaletteHelper::getAccessibleTextColor($badgeColor) }};">
    {{ $badgeLabel }}
</div>
```

---

## Unit Testing with PHPUnit

```php
<?php

declare(strict_types=1);

namespace Tests;

use PHPUnit\Framework\TestCase;
use Sunasty\ColorMath\ColorMath;

final class ColorMathTest extends TestCase
{
    public function testPerfectContrast(): void
    {
        $white = ColorMath::hexToRgb('#ffffff');
        $black = ColorMath::hexToRgb('#000000');

        $ratio = ColorMath::contrastRatio($white, $black);
        $this->assertEquals(21.0, $ratio);

        $report = ColorMath::checkWcag($ratio);
        $this->assertTrue($report['aa_normal']);
        $this->assertTrue($report['aaa_normal']);
    }

    public function testHexShorthand(): void
    {
        $this->assertSame([255, 255, 255], ColorMath::hexToRgb('#fff'));
        $this->assertSame('#ffffff', ColorMath::rgbToHex(255, 255, 255));
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
