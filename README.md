# Figma to BlockStudio Specification Generator

A comprehensive skill for converting Figma designs into production-ready BlockStudio (WordPress block framework) specifications with state management, incremental processing, and automatic variant detection.

## Features

- ✅ **Multi-file output** structure for better organization
- ✅ **Incremental processing** (one page at a time)
- ✅ **State management** across multiple executions
- ✅ **Automatic component classification** (theme vs blocks vs core)
- ✅ **Block similarity detection** with automatic variant management
- ✅ **Visual references** (screenshots from Figma)
- ✅ **Human-readable specs** for manual editing
- ✅ **Design token extraction** (colors, typography, spacing)
- ✅ **Backward compatibility** with legacy single-file output

## Quick Start

### First Page (Home)

```bash
figma-to-blockstudio "https://figma.com/design/...?node-id=1-4" --page home
```

This will create:
```
blockstudio-specs/
├── .blockstudio-state.json
├── theme-config.md
├── blocks-registry.md
├── blocks/
│   ├── hero-section/
│   │   ├── spec.md
│   │   └── reference.png
│   └── feature-grid/
│       ├── spec.md
│       └── reference.png
└── pages/
    └── home/
        └── spec.md
```

### Second Page (About) - Incremental

```bash
figma-to-blockstudio "https://figma.com/design/...?node-id=19-2326" --page about --update
```

The `--update` flag:
- Reads existing state and specs
- Detects similar blocks (adds variants automatically if ≥95% similar)
- Asks for confirmation if 60-94% similar
- Creates new blocks if <60% similar
- Updates blocks registry
- Preserves existing theme config (unless header/footer changed)

## CLI Arguments

| Argument | Description | Default | Example |
|----------|-------------|---------|---------|
| `--page <name>` | Page name for this analysis | Auto-detect from Figma | `--page home` |
| `--update` | Incremental mode (read existing specs) | `false` | `--update` |
| `--full-document` | Analyze all pages in Figma file | `false` | `--full-document` |
| `--output <dir>` | Output directory | `./blockstudio-specs` | `--output ./my-theme` |
| `--namespace <name>` | Block namespace | `theme` | `--namespace acme` |
| `--legacy` | Generate single-file output | `false` | `--legacy` |

## Output Structure

### theme-config.md
Global theme configuration including:
- Design tokens (colors, typography, spacing, layout)
- Header structure and fields
- Footer structure and fields
- Global CSS variables

### blocks-registry.md
Inventory of all custom blocks:
- Block list with status, variants, and usage
- Categorization (hero, content, features, etc.)
- Implementation priority
- Statistics

### blocks/[slug]/spec.md
Individual block specifications with:
- YAML frontmatter (metadata, variants, status)
- Visual reference (screenshot)
- BlockStudio configuration (block.json)
- Template code (index.php)
- Styling (style.scss)
- Field mapping and hierarchy
- Responsive behavior
- Implementation checklist

### pages/[page-name]/spec.md
Page composition specifications:
- Visual reference
- Block composition in order
- Layout structure
- Content requirements
- SEO considerations
- Implementation steps

### .blockstudio-state.json
Hidden state file tracking:
- Processed pages
- Defined blocks (with hashes, variants, fields)
- Design tokens
- Theme components
- Timestamps

## Workflow

### Typical Multi-Page Project

```bash
# 1. Process home page
figma-to-blockstudio "https://figma.com/design/ABC123/...?node-id=1-4" --page home

# Review output, implement blocks

# 2. Process about page (incremental)
figma-to-blockstudio "https://figma.com/design/ABC123/...?node-id=19-2326" --page about --update

# System detects that header, footer, and hero are similar
# Automatically adds variants or asks for confirmation

# 3. Process services page
figma-to-blockstudio "https://figma.com/design/ABC123/...?node-id=35-1500" --page services --update

# Continue for all pages...
```

### Full Document Analysis

```bash
# Analyze entire Figma file at once (token-intensive)
figma-to-blockstudio "https://figma.com/design/ABC123/..." --full-document
```

This will:
- Detect all pages/screens in the file
- Process each page sequentially
- Build complete state incrementally
- Generate comprehensive blocks registry

## Block Similarity Detection

The system automatically compares new components with existing blocks using a weighted similarity algorithm:

### Similarity Score = 100%

**Components:**
- Field types match (40% weight) - Jaccard similarity
- Layout match (30% weight) - Exact layout type
- Field count (20% weight) - Similar number of fields
- Name similarity (10% weight) - Levenshtein distance

### Decision Tree

| Score | Action | Confirmation Required |
|-------|--------|----------------------|
| ≥95% | Auto-add variant to existing block | No |
| 60-94% | Ask user (variant vs new block) | Yes |
| <60% | Create new block | No |

### Example

```
Existing Block: "Hero Section"
- Fields: heading, description, image, cta
- Layout: hero

New Component: "Hero with Overlay"
- Fields: heading, description, image, cta, overlay
- Layout: hero

Similarity Score: 96%
Action: Auto-add "with-overlay" variant to hero-section
```

## Component Classification

The system automatically classifies Figma components:

### Theme Components (→ theme-config.md)
- **Header**: Top frame (<100px) with nav/logo
- **Footer**: Bottom frame (>80% page height) with copyright/links

### Custom Blocks (→ blocks/[slug]/spec.md)
- 3+ child elements
- Editable content (text, images)
- Complex layout (grid/flex)
- Clear semantic purpose

### Core Blocks (→ referenced in page spec)
- Single heading → `core/heading`
- Single paragraph → `core/paragraph`
- Single image → `core/image`
- Button-like element → `core/button`

### Ignored
- Background shapes
- Decorative elements
- Hidden layers
- Reference/notes frames

## Template System

The skill uses templates from `templates/` directory:
- `theme-config-template.md`
- `blocks-registry-template.md`
- `block-spec-template.md`
- `page-spec-template.md`

Templates use `{{PLACEHOLDER}}` syntax for variable replacement.

## Utility Modules

### utils/block-comparator.md
Algorithm for calculating block similarity:
- Jaccard similarity for field types
- Levenshtein distance for names
- Field count penalty
- Layout matching

### utils/component-classifier.md
Logic for classifying Figma components:
- Header/footer detection
- Custom block scoring system
- Core block identification
- Pattern recognition (hero, cards, features, etc.)

## Field Type Mapping

Figma elements are automatically mapped to BlockStudio field types:

| Figma Element | BlockStudio Type | Notes |
|---------------|------------------|-------|
| Short text | `text` | Character limits |
| Multi-line text | `textarea` | Row count |
| Formatted text | `richtext` | Inline formatting |
| Rich content | `wysiwyg` | Full editor |
| Single image | `files` (multiple: false) | Media library |
| Gallery | `files` (multiple: true) | Reorderable |
| Icon | `icon` | SVG icon picker |
| Color | `color` | Palette + picker |
| Link/Button | `link` | Internal/external |
| Number input | `number` | Min/max/step |
| Slider | `range` | Visual slider |
| Dropdown | `select` | Single/multiple |
| Radio buttons | `radio` | Mutually exclusive |
| Checkboxes | `checkbox` | Multiple selection |
| Toggle | `toggle` | Boolean on/off |
| Repeated items | `repeater` | Nested fields |
| Grouped fields | `group` | Collapsible |

## Design Token Extraction

Automatically extracts and organizes:

### Colors
- Primary, secondary, text, heading, background, border, accent

### Typography
- Font families (heading, body)
- Font sizes (h1-h4, body, small)
- Font weights (heading, body, bold)
- Line heights

### Spacing
- Scale: xs, sm, md, lg, xl, xxl
- Container max-width
- Padding values

### Layout
- Grid columns and gap
- Breakpoints (mobile, tablet, desktop)

Tokens are defined as CSS custom properties in `theme-config.md` and referenced in all block specs.

## Best Practices

### For First Run
1. Start with the most important page (usually home)
2. Review generated specs before implementing
3. Adjust design tokens in `theme-config.md` if needed
4. Implement 1-2 blocks to test the workflow

### For Incremental Updates
1. Always use `--update` flag
2. Review similarity notifications
3. Accept variant additions when appropriate
4. Create new blocks when components are truly different
5. Keep state file in version control

### For Implementation
1. Follow the implementation checklist in each block spec
2. Use BEM naming convention (`.block-name__element--modifier`)
3. Reference design tokens from theme-config
4. Test responsive behavior at all breakpoints
5. Validate accessibility (contrast, keyboard nav, ARIA)

## Troubleshooting

### State File Corrupted
```bash
# Remove and start fresh
rm blockstudio-specs/.blockstudio-state.json
figma-to-blockstudio <url> --page <name>
```

### Can't Access Figma File
- Check Figma file sharing settings
- Ensure file URL is correct
- Try opening in browser first

### Similarity Detection Not Working
- Check that state file exists
- Verify block hashes are present
- Use `--update` flag

### Too Many Blocks Created
- Review classification logic
- Manually merge similar blocks in specs
- Adjust similarity thresholds in future (planned feature)

## Limitations

- **Token usage**: Full document mode can be expensive
- **Manual review needed**: Specs are starting points, not final implementation
- **Complex layouts**: Absolute positioning may need manual adjustment
- **Custom animations**: Figma animations need manual translation

## Roadmap

- [ ] Visual diff between variants
- [ ] Export directly to WordPress (skip specs)
- [ ] Support for Twig/Blade templates
- [ ] Auto-sync with file system watching
- [ ] Configurable similarity thresholds
- [ ] Machine learning for better pattern recognition

## Examples

See `examples/` directory for sample outputs:
- `examples/home-page/` - Complete home page analysis
- `examples/multi-page-update/` - Incremental update workflow
- `examples/full-document/` - Full site analysis

## Support

- [BlockStudio Documentation](https://www.blockstudio.dev/)
- [BlockStudio Field Types Reference](https://www.blockstudio.dev/docs/fields)
- [WordPress Block Editor Handbook](https://developer.wordpress.org/block-editor/)

## License

MIT License - See LICENSE file for details

## Credits

Created for figma-to-blockstudio project
Designed for WordPress theme development with BlockStudio framework
