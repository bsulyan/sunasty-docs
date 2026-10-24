# OKLCH Accessible Palette Generation

A common challenge in design systems is generating color tints and shades that are guaranteed to pass WCAG 2.1 accessibility criteria without washing out the brand identity. Traditional HSL lightness scaling produces inconsistent contrast because different hues have drastically different perceived brightness.

The `sunasty-color-math` algorithm solves this by combining the **OKLCH perceptual model** with a **binary search optimization** on lightness $L$.

## The Auto-Fix Algorithm

Given a base foreground color and a target background, the goal is to find the closest lightness coordinate $L'$ in OKLCH such that:

$$\text{ContrastRatio}(\text{OKLCH}(L', C, H), \text{Background}) \ge 4.5$$

Because hue $H$ and chroma $C$ are held constant, the color remains distinctly recognizable as the brand tint while strictly meeting accessibility requirements.

```
                  ┌───────────────────────────────┐
                  │ Input Color: OKLCH(L, C, H)   │
                  │ Target Contrast: Ratio >= 4.5 │
                  └───────────────┬───────────────┘
                                  │
                       Is Ratio >= 4.5 already?
                                 / \
                           YES  /   \  NO
                               v     v
                         Return   Binary Search on Lightness L:
                         Original   - If Background is Dark  -> Increase L
                                    - If Background is Light -> Decrease L
                                    - Convert candidate to sRGB
                                    - Test WCAG Contrast
                                    - Clamp to gamut
                                           │
                                           v
                              Return Optimized OKLCH(L', C, H)
```

## Complete TypeScript Implementation

```typescript
import { 
  rgbToOklch, 
  oklchToRgb, 
  contrastRatio, 
  hexToRgb, 
  rgbToHex 
} from 'sunasty-color-math';

interface AccessibleColorResult {
  hex: string;
  originalRatio: number;
  finalRatio: number;
  adjustedL: number;
  iterations: number;
}

/**
 * Automatically adjusts the lightness of a color in OKLCH space
 * until it achieves the requested WCAG contrast ratio on a background.
 */
export function ensureAccessibleContrast(
  foregroundHex: string,
  backgroundHex: string,
  targetRatio: number = 4.5
): AccessibleColorResult {
  const fgRgb = hexToRgb(foregroundHex);
  const bgRgb = hexToRgb(backgroundHex);

  const initialRatio = contrastRatio(fgRgb, bgRgb);
  if (initialRatio >= targetRatio) {
    return {
      hex: foregroundHex,
      originalRatio: initialRatio,
      finalRatio: initialRatio,
      adjustedL: rgbToOklch(fgRgb[0], fgRgb[1], fgRgb[2])[0],
      iterations: 0
    };
  }

  const [initialL, chroma, hue] = rgbToOklch(fgRgb[0], fgRgb[1], fgRgb[2]);
  const bgL = rgbToOklch(bgRgb[0], bgRgb[1], bgRgb[2])[0];

  // If background is light (L > 0.5), we need a darker foreground (search towards 0.0)
  // If background is dark (L <= 0.5), we need a lighter foreground (search towards 1.0)
  const searchLighter = bgL <= 0.5;

  let low = searchLighter ? initialL : 0.0;
  let high = searchLighter ? 1.0 : initialL;
  let bestRgb = fgRgb;
  let bestRatio = initialRatio;
  let bestL = initialL;

  // 12 iterations gives precision within 0.0002 on Lightness
  for (let i = 0; i < 12; i++) {
    const midL = (low + high) / 2;
    const candidateRgb = oklchToRgb(midL, chroma, hue);
    const candidateRatio = contrastRatio(candidateRgb, bgRgb);

    if (candidateRatio >= targetRatio) {
      bestRgb = candidateRgb;
      bestRatio = candidateRatio;
      bestL = midL;

      // Try to stay as close to original lightness as possible
      if (searchLighter) {
        high = midL;
      } else {
        low = midL;
      }
    } else {
      if (searchLighter) {
        low = midL;
      } else {
        high = midL;
      }
    }
  }

  return {
    hex: rgbToHex(bestRgb[0], bestRgb[1], bestRgb[2]),
    originalRatio: initialRatio,
    finalRatio: bestRatio,
    adjustedL: bestL,
    iterations: 12
  };
}

// Example usage:
const result = ensureAccessibleContrast('#38BDF8', '#FFFFFF', 4.5);
console.log(`Original: #38BDF8 (${result.originalRatio}:1)`);
console.log(`Accessible: ${result.hex} (${result.finalRatio}:1)`);
```

## Interactive Palette Generator

Test this palette engine interactively on the web with the [Sunasty Color Picker & Palette Extractor](https://sunasty.com/color-picker/) and examine contrast matrices on the [Sunasty Contrast Checker](https://sunasty.com/contrast-checker/).
