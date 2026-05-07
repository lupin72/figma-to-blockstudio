---
type: theme-config
version: 1.0
lastUpdated: {{TIMESTAMP}}
figmaSource: {{FIGMA_URL}}
---

# Theme Configuration

> Generated from Figma: {{FIGMA_URL}}
> Last updated: {{TIMESTAMP}}

## Design Tokens

### Colors
```css
:root {
  /* Primary colors */
  --color-primary: {{PRIMARY_COLOR}};
  --color-secondary: {{SECONDARY_COLOR}};

  /* Text colors */
  --color-text: {{TEXT_COLOR}};
  --color-heading: {{HEADING_COLOR}};

  /* Background colors */
  --color-background: {{BACKGROUND_COLOR}};
  --color-background-alt: {{BACKGROUND_ALT_COLOR}};

  /* UI colors */
  --color-border: {{BORDER_COLOR}};
  --color-accent: {{ACCENT_COLOR}};
}
```

### Typography
```css
:root {
  /* Font families */
  --font-heading: {{FONT_HEADING}};
  --font-body: {{FONT_BODY}};

  /* Font sizes */
  --size-h1: {{SIZE_H1}};
  --size-h2: {{SIZE_H2}};
  --size-h3: {{SIZE_H3}};
  --size-h4: {{SIZE_H4}};
  --size-body: {{SIZE_BODY}};
  --size-small: {{SIZE_SMALL}};

  /* Font weights */
  --weight-heading: {{WEIGHT_HEADING}};
  --weight-body: {{WEIGHT_BODY}};
  --weight-bold: {{WEIGHT_BOLD}};

  /* Line heights */
  --line-height-heading: {{LINE_HEIGHT_HEADING}};
  --line-height-body: {{LINE_HEIGHT_BODY}};
}
```

### Spacing
```css
:root {
  /* Spacing scale */
  --spacing-xs: {{SPACING_XS}};
  --spacing-sm: {{SPACING_SM}};
  --spacing-md: {{SPACING_MD}};
  --spacing-lg: {{SPACING_LG}};
  --spacing-xl: {{SPACING_XL}};
  --spacing-xxl: {{SPACING_XXL}};

  /* Container */
  --container-max-width: {{CONTAINER_MAX_WIDTH}};
  --container-padding: {{CONTAINER_PADDING}};
}
```

### Layout
```css
:root {
  /* Grid */
  --grid-columns: {{GRID_COLUMNS}};
  --grid-gap: {{GRID_GAP}};

  /* Breakpoints */
  --breakpoint-mobile: {{BREAKPOINT_MOBILE}};
  --breakpoint-tablet: {{BREAKPOINT_TABLET}};
  --breakpoint-desktop: {{BREAKPOINT_DESKTOP}};
}
```

---

## Header Structure

### Visual Reference
![Header Reference]({{HEADER_SCREENSHOT}})

### Component Breakdown
{{HEADER_DESCRIPTION}}

### Fields
```json
{
  "headerLayout": {
    "type": "select",
    "label": "Header Layout",
    "options": {{HEADER_LAYOUT_OPTIONS}},
    "default": "{{HEADER_DEFAULT_LAYOUT}}"
  },
  "logo": {
    "type": "files",
    "label": "Logo",
    "multiple": false,
    "allowedTypes": ["image"]
  },
  "navigation": {
    "type": "repeater",
    "label": "Navigation Items",
    "fields": {
      "label": {
        "type": "text",
        "label": "Label"
      },
      "link": {
        "type": "link",
        "label": "Link"
      }
    }
  },
  "ctaButton": {
    "type": "link",
    "label": "CTA Button",
    "opensInNewTab": false
  }
}
```

### Styling
```scss
.site-header {
  {{HEADER_STYLES}}
}
```

---

## Footer Structure

### Visual Reference
![Footer Reference]({{FOOTER_SCREENSHOT}})

### Component Breakdown
{{FOOTER_DESCRIPTION}}

### Fields
```json
{
  "footerLayout": {
    "type": "select",
    "label": "Footer Layout",
    "options": {{FOOTER_LAYOUT_OPTIONS}},
    "default": "{{FOOTER_DEFAULT_LAYOUT}}"
  },
  "copyrightText": {
    "type": "text",
    "label": "Copyright Text",
    "default": "{{COPYRIGHT_DEFAULT}}"
  },
  "socialLinks": {
    "type": "repeater",
    "label": "Social Links",
    "fields": {
      "platform": {
        "type": "select",
        "label": "Platform",
        "options": [
          { "label": "Facebook", "value": "facebook" },
          { "label": "Twitter", "value": "twitter" },
          { "label": "Instagram", "value": "instagram" },
          { "label": "LinkedIn", "value": "linkedin" }
        ]
      },
      "url": {
        "type": "link",
        "label": "URL"
      }
    }
  },
  "footerColumns": {
    "type": "repeater",
    "label": "Footer Columns",
    "fields": {
      "heading": {
        "type": "text",
        "label": "Column Heading"
      },
      "links": {
        "type": "repeater",
        "label": "Links",
        "fields": {
          "label": {
            "type": "text",
            "label": "Link Label"
          },
          "url": {
            "type": "link",
            "label": "URL"
          }
        }
      }
    }
  }
}
```

### Styling
```scss
.site-footer {
  {{FOOTER_STYLES}}
}
```

---

## Global Settings

### Responsive Breakpoints
- **Mobile**: < {{BREAKPOINT_MOBILE}}
- **Tablet**: {{BREAKPOINT_MOBILE}} - {{BREAKPOINT_TABLET}}
- **Desktop**: > {{BREAKPOINT_DESKTOP}}

### Animation/Transition Defaults
```css
:root {
  --transition-fast: 150ms ease;
  --transition-base: 250ms ease;
  --transition-slow: 350ms ease;
}
```

---

## Implementation Notes

{{IMPLEMENTATION_NOTES}}

---

## Theme Setup Checklist

- [ ] Load design tokens in theme.json or functions.php
- [ ] Configure header in Customizer or theme settings
- [ ] Configure footer in Customizer or theme settings
- [ ] Load custom fonts (if not system fonts)
- [ ] Test responsive behavior across all breakpoints
- [ ] Verify accessibility (keyboard navigation, ARIA labels)
- [ ] Test with different content lengths (long menu items, etc.)
