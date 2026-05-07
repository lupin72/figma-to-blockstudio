# Component Classifier

This utility classifies Figma components into categories to determine how they should be handled in the BlockStudio specification generation.

## Classification Categories

1. **Theme Components** - Header, Footer, Navigation
2. **Custom Blocks** - Unique content blocks requiring custom implementation
3. **Core Blocks** - Standard WordPress blocks (heading, paragraph, image, button)
4. **Ignored Components** - Background elements, decorative shapes, overlays

## Classification Algorithm

### 1. Header Detection

**Criteria (ALL must be true):**
- Frame is in top 100px of page (`frame.y < 100`)
- Contains text matching navigation keywords: `nav`, `menu`, `navigation`, `header`
- OR contains logo-like element: small image in top-left area
- Has horizontal layout (width > 2× height)
- Contains 3+ clickable/link elements OR has repeating link pattern

**Additional Signals:**
- Frame named "Header", "Navigation", "Nav Bar", etc.
- Contains "Login", "Sign Up", "Get Started" buttons
- Has hamburger icon (mobile menu indicator)

**Output:**
- **Type**: `theme-header`
- **Action**: Add to `theme-config.md`, not `blocks/`
- **Fields**: Logo, navigation items, CTA buttons

**Example Detection:**
```javascript
function isHeader(frame, pageHeight) {
  // Position check
  if (frame.y > 100) return false

  // Name check
  if (/header|nav|navigation|menu/i.test(frame.name)) return true

  // Layout check
  if (frame.width < frame.height * 2) return false

  // Content check
  const hasNavLinks = frame.children.filter(c =>
    c.type === 'TEXT' && /home|about|services|contact|products/i.test(c.characters)
  ).length >= 3

  const hasLogo = frame.children.some(c =>
    c.type === 'IMAGE' && c.x < 200 && c.width < 150
  )

  return hasNavLinks || hasLogo
}
```

---

### 2. Footer Detection

**Criteria (ALL must be true):**
- Frame is in bottom 20% of page (`frame.y > pageHeight * 0.8`)
- Contains text matching footer keywords: `footer`, `copyright`, `©`, `privacy`, `terms`
- OR has multi-column layout with many links
- Width ≥ 80% of page width

**Additional Signals:**
- Frame named "Footer"
- Contains social media icons
- Contains "All rights reserved" or year (e.g., "2025")
- Has dark background (common pattern)

**Output:**
- **Type**: `theme-footer`
- **Action**: Add to `theme-config.md`, not `blocks/`
- **Fields**: Copyright text, footer columns, social links

**Example Detection:**
```javascript
function isFooter(frame, pageHeight, pageWidth) {
  // Position check
  if (frame.y < pageHeight * 0.8) return false

  // Width check
  if (frame.width < pageWidth * 0.8) return false

  // Name check
  if (/footer/i.test(frame.name)) return true

  // Content check
  const hasCopyright = frame.children.some(c =>
    c.type === 'TEXT' && /copyright|©|all rights reserved|\d{4}/i.test(c.characters)
  )

  const hasFooterLinks = frame.children.filter(c =>
    c.type === 'TEXT' && /privacy|terms|contact|about/i.test(c.characters)
  ).length >= 2

  const hasSocialIcons = frame.children.filter(c =>
    c.type === 'IMAGE' && c.width < 40 && c.height < 40
  ).length >= 2

  return hasCopyright || hasFooterLinks || hasSocialIcons
}
```

---

### 3. Custom Block Detection

**Criteria (2+ must be true):**
- Has 3+ child elements
- Contains editable content (text, images, or both)
- Has complex layout (grid, flex, or nested containers)
- Has clear semantic purpose (not just decoration)
- Is not header/footer
- Is not a single core block

**Scoring System:**
```
Score = 0
+ Has 3-10 children: +2 points
+ Has 10+ children: +3 points
+ Contains text layers: +2 points
+ Contains images: +2 points
+ Has auto-layout (Figma): +2 points
+ Has repeating pattern: +3 points (good candidate for repeater field)
+ Named semantically (hero, feature, testimonial): +2 points
+ Has clear hierarchy (heading + body): +2 points

If Score ≥ 6: Custom Block
```

**Common Custom Block Patterns:**

#### Hero Section
```
Pattern:
- Large heading (font-size > 32px)
- Descriptive text below
- CTA button(s)
- Background image or color
- Usually first section after header

Detection: heading + (button OR large-text) + background
```

#### Card Grid
```
Pattern:
- Repeating items (3-6 typically)
- Each item: image + title + text
- Grid or flex layout
- Equal-sized cards

Detection: 3+ similar components with image+text pattern
```

#### Feature Section
```
Pattern:
- Icon/image + heading + text
- Repeated 2-4 times
- Horizontal or grid layout

Detection: repeated (icon + heading + text) pattern
```

#### Testimonial
```
Pattern:
- Quote text (often styled differently)
- Author name
- Author role/company
- Optional: avatar, rating stars

Detection: text + attribution + optional-image
```

#### CTA Block
```
Pattern:
- Attention-grabbing heading
- Short description
- Button(s)
- Often has colored/image background

Detection: heading + button + background-treatment
```

**Output:**
- **Type**: `custom-block`
- **Action**: Generate full spec in `blocks/[slug]/spec.md`
- **Fields**: Map content to appropriate field types

**Example Detection:**
```javascript
function isCustomBlock(frame) {
  let score = 0
  const children = frame.children.length

  // Child count
  if (children >= 3 && children <= 10) score += 2
  if (children > 10) score += 3

  // Content types
  const hasText = frame.children.some(c => c.type === 'TEXT')
  const hasImages = frame.children.some(c => c.type === 'IMAGE')
  if (hasText) score += 2
  if (hasImages) score += 2

  // Layout
  if (frame.layoutMode !== 'NONE') score += 2  // Has auto-layout

  // Repeating pattern
  const hasRepeating = detectRepeatingPattern(frame.children)
  if (hasRepeating) score += 3

  // Semantic naming
  if (/hero|feature|testimonial|cta|card|grid/i.test(frame.name)) score += 2

  // Hierarchy
  const hasHeading = frame.children.some(c =>
    c.type === 'TEXT' && c.fontSize > 24
  )
  const hasBody = frame.children.some(c =>
    c.type === 'TEXT' && c.fontSize <= 18
  )
  if (hasHeading && hasBody) score += 2

  return score >= 6
}

function detectRepeatingPattern(children) {
  // Group similar components
  const groups = {}
  children.forEach(child => {
    const signature = `${child.type}-${child.width}-${child.height}`
    groups[signature] = (groups[signature] || 0) + 1
  })

  // If any group has 3+ items, it's repeating
  return Object.values(groups).some(count => count >= 3)
}
```

---

### 4. Core Block Detection

**Single WordPress core block (don't create custom block):**

#### core/heading
- Single text layer
- Font size > 18px
- Short text (< 100 characters)
- No complex styling

#### core/paragraph
- Single text layer
- Font size 14-18px
- Longer text
- Simple formatting

#### core/image
- Single image
- No complex overlay or effects
- Standard aspect ratio

#### core/button
- Small text element
- Button-like shape
- Solid background
- Clear CTA text ("Learn More", "Get Started")

#### core/spacer
- Empty frame
- Used for spacing only
- No content

**Output:**
- **Type**: `core-block`
- **Action**: Mention in page composition, don't generate custom block
- **Reference**: "Use `core/heading` for this element"

**Example Detection:**
```javascript
function identifyCoreBlock(frame) {
  const children = frame.children

  // Single text layer
  if (children.length === 1 && children[0].type === 'TEXT') {
    const text = children[0]
    if (text.fontSize > 24) return 'core/heading'
    if (text.characters.length > 50) return 'core/paragraph'
    if (/button|cta|learn more|get started/i.test(text.characters)) {
      return 'core/button'
    }
    return 'core/paragraph'
  }

  // Single image
  if (children.length === 1 && children[0].type === 'IMAGE') {
    return 'core/image'
  }

  // Empty spacer
  if (children.length === 0) {
    return 'core/spacer'
  }

  return null  // Not a core block
}
```

---

### 5. Ignored Components

**Skip these (don't generate specs):**

- Background shapes/rectangles
- Decorative elements
- Overlay effects
- Grid/guide lines
- Figma component instances used as references
- Frames named "Reference", "Notes", "Backup"

**Detection:**
```javascript
function shouldIgnore(frame) {
  // Name patterns
  if (/background|overlay|decoration|reference|notes|backup|guide/i.test(frame.name)) {
    return true
  }

  // Pure decoration (no text, no images, just shapes)
  if (frame.children.length === 0 ||
      frame.children.every(c => c.type === 'RECTANGLE' || c.type === 'ELLIPSE')) {
    return true
  }

  // Hidden/invisible
  if (frame.visible === false || frame.opacity === 0) {
    return true
  }

  return false
}
```

---

## Classification Workflow

```javascript
function classifyComponent(frame, context) {
  const { pageHeight, pageWidth, existingBlocks } = context

  // 1. Check if should ignore
  if (shouldIgnore(frame)) {
    return { type: 'ignored', reason: 'Decorative or reference element' }
  }

  // 2. Check for header
  if (isHeader(frame, pageHeight)) {
    return { type: 'theme-header', action: 'add-to-theme-config' }
  }

  // 3. Check for footer
  if (isFooter(frame, pageHeight, pageWidth)) {
    return { type: 'theme-footer', action: 'add-to-theme-config' }
  }

  // 4. Check for core block
  const coreBlock = identifyCoreBlock(frame)
  if (coreBlock) {
    return {
      type: 'core-block',
      coreBlockType: coreBlock,
      action: 'reference-in-page-spec'
    }
  }

  // 5. Check for custom block
  if (isCustomBlock(frame)) {
    return {
      type: 'custom-block',
      action: 'generate-block-spec',
      pattern: detectBlockPattern(frame)
    }
  }

  // 6. Default: might be a container or section
  return {
    type: 'container',
    action: 'analyze-children',
    note: 'Not a block itself, but may contain blocks'
  }
}

function detectBlockPattern(frame) {
  const patterns = [
    { name: 'hero', test: isHeroPattern },
    { name: 'card-grid', test: isCardGridPattern },
    { name: 'feature', test: isFeaturePattern },
    { name: 'testimonial', test: isTestimonialPattern },
    { name: 'cta', test: isCtaPattern },
    { name: 'content-image', test: isContentImagePattern }
  ]

  for (const pattern of patterns) {
    if (pattern.test(frame)) {
      return pattern.name
    }
  }

  return 'generic'
}
```

---

## Decision Matrix

| Component Type | Generate Block Spec? | Add to Theme Config? | Reference Only? |
|----------------|---------------------|---------------------|-----------------|
| Header | ✗ | ✓ | ✗ |
| Footer | ✗ | ✓ | ✗ |
| Custom Block | ✓ | ✗ | ✗ |
| Core Block | ✗ | ✗ | ✓ |
| Ignored | ✗ | ✗ | ✗ |
| Container | Analyze Children | ✗ | ✗ |

---

## Integration with Main Workflow

```javascript
// In figma-to-blockstudio main process:
async function processPage(figmaUrl, options) {
  const { nodeId, fileKey } = parseFigmaUrl(figmaUrl)
  const designContext = await get_design_context(fileKey, nodeId)
  const metadata = await get_metadata(fileKey, nodeId)

  // Parse Figma structure
  const frames = extractFrames(metadata)

  const classifications = {
    header: null,
    footer: null,
    customBlocks: [],
    coreBlocks: [],
    ignored: []
  }

  for (const frame of frames) {
    const classification = classifyComponent(frame, {
      pageHeight: metadata.height,
      pageWidth: metadata.width,
      existingBlocks: readStateFile().blocks
    })

    switch (classification.type) {
      case 'theme-header':
        classifications.header = frame
        break
      case 'theme-footer':
        classifications.footer = frame
        break
      case 'custom-block':
        classifications.customBlocks.push({
          frame,
          pattern: classification.pattern
        })
        break
      case 'core-block':
        classifications.coreBlocks.push({
          frame,
          coreBlockType: classification.coreBlockType
        })
        break
      case 'ignored':
        classifications.ignored.push(frame)
        break
    }
  }

  return classifications
}
```

---

## Testing

### Test Header Detection
```
✓ Frame at y=0, contains "Home | About | Services" → Header
✓ Frame at y=50, contains logo + nav → Header
✗ Frame at y=200 → Not header (too low)
```

### Test Footer Detection
```
✓ Frame at y=2800 (page height 3000), contains "© 2025" → Footer
✓ Frame at bottom, 3 columns of links → Footer
✗ Frame at y=1000 → Not footer (not at bottom)
```

### Test Custom Block
```
✓ Frame with heading + text + image + button → Custom block (hero)
✓ Frame with 4 repeating cards → Custom block (card-grid)
✗ Single heading → Core block (core/heading)
```

### Test Core Block
```
✓ Single text "Welcome to our site" (48px) → core/heading
✓ Single paragraph text → core/paragraph
✓ Single image → core/image
✗ Text + image + button → Custom block (too complex)
```

---

## Output Example

When classification is complete:

```javascript
{
  header: {
    type: 'theme-header',
    fields: ['logo', 'navigation', 'ctaButton'],
    screenshot: 'header.png'
  },
  footer: {
    type: 'theme-footer',
    fields: ['copyrightText', 'footerColumns', 'socialLinks'],
    screenshot: 'footer.png'
  },
  customBlocks: [
    {
      name: 'Hero Section',
      slug: 'hero-section',
      pattern: 'hero',
      fields: ['heading', 'description', 'backgroundImage', 'ctaButton']
    },
    {
      name: 'Feature Grid',
      slug: 'feature-grid',
      pattern: 'card-grid',
      fields: ['items[].icon', 'items[].title', 'items[].description']
    }
  ],
  coreBlocks: [
    { type: 'core/heading', text: 'Our Mission' },
    { type: 'core/paragraph', text: 'We strive to...' }
  ],
  stats: {
    total: 15,
    header: 1,
    footer: 1,
    customBlocks: 2,
    coreBlocks: 2,
    ignored: 9
  }
}
```
