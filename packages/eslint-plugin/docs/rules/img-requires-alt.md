# img-requires-alt

Enforces `alt` attribute on `<img>` elements for accessibility (WCAG 2.1 Level A compliance).

⚠️ This rule **_warns_** by default in the `recommended` config.

## Rule Details

Screen readers rely on `alt` text to describe images to visually impaired users. Missing or empty `alt` attributes create accessibility barriers and violate WCAG 2.1 guidelines.

### Why This Matters

| Issue | Impact | Standard |
|-------|--------|----------|
| ♿ **Accessibility** | Screen readers can't describe image | WCAG 2.1 Level A |
| 🔍 **SEO** | Search engines can't index images | Google Guidelines |
| 📱 **Mobile** | Broken images show no context | UX Best Practice |
| ⚖️ **Legal** | ADA/Section 508 compliance risk | Legal Requirement |

## Examples

### ❌ Incorrect

```tsx
// Missing alt attribute
<img src="logo.png" />

// Empty alt (when image has meaning)
<img src="chart.png" alt="" />

// Non-descriptive alt
<img src="product.jpg" alt="image" />
<img src="photo.jpg" alt="picture" />

// Alt same as filename
<img src="screenshot.png" alt="screenshot.png" />
```

### ✅ Correct

```tsx
// Descriptive alt text
<img src="logo.png" alt="Company Logo" />

// Informative description
<img src="chart.png" alt="Sales data showing 20% growth in Q4 2024" />

// Context-specific
<img src="user-avatar.jpg" alt="Profile photo of John Doe" />

// Decorative images (no meaning)
<img src="decorative-line.svg" alt="" role="presentation" />

// Background pattern
<img src="pattern.png" alt="" aria-hidden="true" />
```

## Configuration

```javascript
{
  rules: {
    '@forge-js/llm-optimized/img-requires-alt': ['error', {
      allowAriaLabel: true,          // Accept aria-label as alternative
      allowAriaLabelledby: true      // Accept aria-labelledby as alternative
    }]
  }
}
```

## Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `allowAriaLabel` | `boolean` | `true` | Accept `aria-label` as alternative to `alt` |
| `allowAriaLabelledby` | `boolean` | `true` | Accept `aria-labelledby` as alternative to `alt` |

### Strict Alt-Only Mode

```javascript
{
  rules: {
    '@forge-js/llm-optimized/img-requires-alt': ['error', {
      allowAriaLabel: false,       // Require alt attribute
      allowAriaLabelledby: false   // Don't accept aria alternatives
    }]
  }
}
```

```tsx
// ❌ Fails with strict mode
<img src="logo.png" aria-label="Company Logo" />

// ✅ Passes with strict mode
<img src="logo.png" alt="Company Logo" />
```

### Allow ARIA Alternatives

```javascript
{
  rules: {
    '@forge-js/llm-optimized/img-requires-alt': ['error', {
      allowAriaLabel: true,
      allowAriaLabelledby: true
    }]
  }
}
```

```tsx
// ✅ All valid with ARIA alternatives enabled
<img src="logo.png" alt="Company Logo" />
<img src="logo.png" aria-label="Company Logo" />
<img src="logo.png" aria-labelledby="logo-description" />
```

## Best Practices

### Decorative vs. Informative Images

```tsx
// ✅ Decorative (empty alt is correct)
<img src="border.svg" alt="" role="presentation" />

// ✅ Informative (descriptive alt required)
<img src="chart.svg" alt="Revenue increased 30% year over year" />

// ❌ Decorative treated as informative
<img src="border.svg" alt="decorative border" />
```

### Alt Text Guidelines

| Scenario | Alt Text Example |
|----------|------------------|
| **Logo** | "Company Name Logo" or "Company Name" |
| **Photo** | "Person/place/thing doing action" |
| **Chart** | "Chart type showing key insight" |
| **Icon** | Function of the icon, not description |
| **Button** | Action the button performs |
| **Decorative** | Empty alt (`alt=""`) |

### Dynamic Images

```tsx
// ✅ Alt from data
<img 
  src={user.avatar} 
  alt={`Profile photo of ${user.name}`} 
/>

// ✅ Fallback for missing data
<img 
  src={product.image} 
  alt={product.description || `${product.name} product image`} 
/>

// ❌ Generic alt for dynamic content
<img src={photos[index]} alt="photo" />
```

## Framework-Specific Examples

### React

```tsx
// ✅ Component with required alt prop
interface ImageProps {
  src: string;
  alt: string;  // Required prop
}

const Image: React.FC<ImageProps> = ({ src, alt }) => (
  <img src={src} alt={alt} />
);

// Usage
<Image src="logo.png" alt="Company Logo" />
```

### Next.js

```tsx
import Image from 'next/image';

// ✅ Next.js Image component (alt is required)
<Image 
  src="/hero.jpg" 
  alt="Hero showing product in use" 
  width={800} 
  height={600} 
/>
```

### Vue

```vue
<!-- ✅ Vue with computed alt -->
<template>
  <img :src="imageSrc" :alt="imageDescription" />
</template>

<script>
export default {
  props: ['imageSrc'],
  computed: {
    imageDescription() {
      return this.imageSrc ? `Image: ${this.imageSrc}` : 'No image';
    }
  }
}
</script>
```

## WCAG 2.1 Compliance

This rule helps satisfy:

- **1.1.1 Non-text Content (Level A)**: All non-text content needs text alternative
- **Success Criterion**: Images must have alt text or be marked as decorative

### WCAG Testing

```tsx
// ✅ Passes WCAG 1.1.1
<img src="diagram.png" alt="Flowchart showing user registration process" />

// ❌ Fails WCAG 1.1.1
<img src="diagram.png" />
```

## Auto-Fix Capability

This rule provides suggestions but **cannot auto-fix** (meaningful alt text requires human judgment):

```
⚠️ Missing alt attribute on <img> element

Suggestions:
  1. Add descriptive alt text based on image content
  2. If decorative, use alt="" and role="presentation"
  3. Consider using aria-label for complex scenarios
```

## Common Patterns

### Image Galleries

```tsx
// ✅ Gallery with descriptive alt
{photos.map((photo, index) => (
  <img 
    key={photo.id}
    src={photo.url} 
    alt={`${photo.title}: ${photo.description}`} 
  />
))}
```

### Icons with Text

```tsx
// ✅ Icon with adjacent text (alt="" is appropriate)
<button>
  <img src="save-icon.svg" alt="" />
  <span>Save</span>
</button>

// ✅ Icon without text (alt describes action)
<button>
  <img src="save-icon.svg" alt="Save" />
</button>
```

### Background Images

```tsx
// ✅ CSS background images (not checked by this rule)
<div style={{ backgroundImage: 'url(hero.jpg)' }} role="img" aria-label="Hero image" />

// Or use semantic HTML
<img src="hero.jpg" alt="Hero showing product features" />
```

## When Not To Use It

- When using a framework that enforces alt attributes at the type level (e.g., Next.js Image)
- For non-web projects (Node.js backends, CLIs)
- In component libraries where images are always decorative

## Related Rules

- `jsx-a11y/alt-text` - React-specific alt text checking
- `img-redundant-alt` - Detects redundant words in alt text

## Further Reading

- [WCAG 2.1 - Non-text Content](https://www.w3.org/WAI/WCAG21/Understanding/non-text-content.html)
- [WebAIM: Alternative Text](https://webaim.org/techniques/alttext/)
- [MDN: The Image Embed element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#accessibility_concerns)
- [Alt Text Decision Tree](https://www.w3.org/WAI/tutorials/images/decision-tree/)

## Version

This rule is available in `@forge-js/eslint-plugin-llm-optimized` v0.0.1+

