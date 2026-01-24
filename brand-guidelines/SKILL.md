---
name: brand-guidelines
description: Brand guidelines for Cosmonic and wasmCloud. Use this skill when creating visual content, presentations, websites, or any materials that need to follow Cosmonic or wasmCloud brand standards.
license: Apache-2.0
tags:
  - branding
  - design
  - visual-identity
  - cosmonic
  - wasmcloud
---

# Cosmonic & wasmCloud Brand Guidelines

This skill provides official brand colors and typography for Cosmonic and wasmCloud projects.

## Cosmonic Brand

### Typography

- **Primary Font:** Work Sans
- **Fallback:** Arial, sans-serif

### Colors

**Primary Colors:**

| Color Name    | Hex Code  | Usage                              |
|---------------|-----------|-------------------------------------|
| Slate Purple  | `#685BC7` | Primary brand color, CTAs, accents |
| Light Gray    | `#768692` | Secondary text, subtle elements    |

**Secondary Colors:**

| Color Name | Hex Code  | Usage                                |
|------------|-----------|--------------------------------------|
| Yellow     | `#FFB600` | Highlights, warnings, attention      |
| Space Blue | `#002E5D` | Dark backgrounds, headers            |
| Gunmetal   | `#253746` | Dark text, secondary backgrounds     |
| Gainsboro  | `#D9E1E2` | Light backgrounds, borders, dividers |

## wasmCloud Brand

### Typography

- **Primary Font:** Lexend
- **Fallback:** Arial, sans-serif

### Colors

**Primary Colors:**

| Color Name  | Hex Code  | Usage                              |
|-------------|-----------|-------------------------------------|
| Green Aqua  | `#00C389` | Primary brand color, CTAs, accents |
| Light Gray  | `#768692` | Secondary text, subtle elements    |

**Secondary Colors:**

wasmCloud shares the same secondary color palette as Cosmonic:

| Color Name | Hex Code  | Usage                                |
|------------|-----------|--------------------------------------|
| Yellow     | `#FFB600` | Highlights, warnings, attention      |
| Space Blue | `#002E5D` | Dark backgrounds, headers            |
| Gunmetal   | `#253746` | Dark text, secondary backgrounds     |
| Gainsboro  | `#D9E1E2` | Light backgrounds, borders, dividers |

## CSS Variables

For web projects, use these CSS custom properties:

```css
/* Cosmonic Brand */
:root {
  --cosmonic-slate-purple: #685BC7;
  --cosmonic-light-gray: #768692;
  --cosmonic-yellow: #FFB600;
  --cosmonic-space-blue: #002E5D;
  --cosmonic-gunmetal: #253746;
  --cosmonic-gainsboro: #D9E1E2;
}

/* wasmCloud Brand */
:root {
  --wasmcloud-green-aqua: #00C389;
  --wasmcloud-light-gray: #768692;
  --wasmcloud-yellow: #FFB600;
  --wasmcloud-space-blue: #002E5D;
  --wasmcloud-gunmetal: #253746;
  --wasmcloud-gainsboro: #D9E1E2;
}
```

## Tailwind Configuration

For Tailwind CSS projects:

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        cosmonic: {
          purple: '#685BC7',
          gray: '#768692',
          yellow: '#FFB600',
          blue: '#002E5D',
          gunmetal: '#253746',
          gainsboro: '#D9E1E2',
        },
        wasmcloud: {
          aqua: '#00C389',
          gray: '#768692',
          yellow: '#FFB600',
          blue: '#002E5D',
          gunmetal: '#253746',
          gainsboro: '#D9E1E2',
        },
      },
      fontFamily: {
        cosmonic: ['Work Sans', 'Arial', 'sans-serif'],
        wasmcloud: ['Lexend', 'Arial', 'sans-serif'],
      },
    },
  },
};
```

## Usage Guidelines

### When to Use Cosmonic Branding
- Cosmonic product interfaces and dashboards
- Cosmonic marketing materials
- Cosmonic documentation and websites
- Enterprise and commercial content

### When to Use wasmCloud Branding
- wasmCloud open-source project materials
- wasmCloud documentation and tutorials
- Community content and contributions
- Technical demonstrations

### Color Combinations

**Cosmonic:**
- Primary: Slate Purple (`#685BC7`) on white or Gainsboro backgrounds
- Dark mode: Light text on Space Blue (`#002E5D`) or Gunmetal (`#253746`)
- Accents: Yellow (`#FFB600`) for highlights and CTAs

**wasmCloud:**
- Primary: Green Aqua (`#00C389`) on white or Gainsboro backgrounds
- Dark mode: Light text on Space Blue (`#002E5D`) or Gunmetal (`#253746`)
- Accents: Yellow (`#FFB600`) for highlights and CTAs

### Accessibility

Ensure sufficient color contrast for text readability:
- Use Gunmetal (`#253746`) or Space Blue (`#002E5D`) for body text on light backgrounds
- Use white or Gainsboro (`#D9E1E2`) for text on dark backgrounds
- Test color combinations with WCAG contrast checkers
