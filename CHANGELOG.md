# Changelog

All notable changes to the Figma to BlockStudio Specification Generator.

## [2.0.0] - 2026-05-07

### 🎉 Major Refactor - Multi-File System

This is a complete overhaul of the skill, transforming it from a single-file generator to a sophisticated multi-file system with state management and incremental processing.

### Added

#### Core Features
- **Multi-file output structure** with organized directory hierarchy
- **State management system** (`.blockstudio-state.json`) for tracking blocks, pages, and design tokens across executions
- **Incremental processing** with `--update` flag to process pages one at a time
- **Automatic component classification** (theme components, custom blocks, core blocks, ignored elements)
- **Block similarity detection** with automatic variant management
  - ≥95% similarity: Auto-add variant
  - 60-94% similarity: Ask user for confirmation
  - <60% similarity: Create new block
- **Visual reference screenshots** downloaded from Figma for each block
- **Full document mode** (`--full-document`) to analyze entire Figma file at once

#### CLI Arguments
- `--page <name>` - Specify page name for analysis
- `--update` - Incremental mode (read existing specs and update)
- `--full-document` - Analyze all pages in Figma file
- `--output <dir>` - Custom output directory (default: `./blockstudio-specs`)
- `--namespace <name>` - Custom block namespace (default: `theme`)
- `--legacy` - Generate single-file output for backward compatibility

#### New Files & Templates
- `templates/theme-config-template.md` - Theme configuration template
- `templates/blocks-registry-template.md` - Blocks inventory template
- `templates/block-spec-template.md` - Individual block spec template
- `templates/page-spec-template.md` - Page composition template
- `utils/block-comparator.md` - Block similarity algorithm
- `utils/component-classifier.md` - Component classification logic

#### Output Files
- `theme-config.md` - Global theme configuration with design tokens, header, footer
- `blocks-registry.md` - Complete inventory of all blocks with status, variants, usage
- `blocks/[slug]/spec.md` - Individual block specifications with YAML frontmatter
- `blocks/[slug]/reference.png` - Visual reference screenshot from Figma
- `pages/[page-name]/spec.md` - Page composition and block usage
- `.blockstudio-state.json` - Hidden state file for tracking

#### Component Classification
- **Header detection**: Identifies header/navigation components → adds to theme-config
- **Footer detection**: Identifies footer components → adds to theme-config
- **Custom block detection**: Scores components to determine if they should be custom blocks
- **Core block mapping**: Maps simple components to WordPress core blocks
- **Pattern recognition**: Identifies common patterns (hero, cards, features, testimonials, CTA)

#### Similarity Algorithm
- **Weighted scoring system** (0-100%)
  - Field types match (40% weight) using Jaccard similarity
  - Layout match (30% weight)
  - Field count similarity (20% weight)
  - Name similarity (10% weight) using Levenshtein distance
- **Automatic variant detection** with configurable thresholds
- **User confirmation** for medium-similarity matches

#### Enhanced Specs
- **YAML frontmatter** in all spec files for machine-parsability
- **Variant sections** documenting differences from default
- **Component hierarchy trees** showing nested structure
- **Field mapping tables** linking Figma elements to BlockStudio fields
- **Responsive behavior documentation** for mobile/tablet/desktop
- **Implementation checklists** for each block
- **Accessibility considerations** section
- **Troubleshooting sections** for common issues

### Changed

#### Major Changes
- **Output format**: From single `design-spec.md` to multi-file structure
- **Workflow**: From one-shot generation to incremental processing
- **State tracking**: Added persistent state between executions
- **Template system**: Introduced template files for consistency

#### Improvements
- **Better organization**: Separate files for theme, blocks, pages, registry
- **Human-readable**: All specs remain editable markdown
- **Developer-friendly**: Clear implementation paths and checklists
- **Scalable**: Can handle large projects with many pages and blocks
- **Intelligent**: Detects similarities and avoids duplication

### Maintained

#### Backward Compatibility
- `--legacy` flag generates original single-file `design-spec.md`
- All original field type mappings preserved
- Original template examples maintained
- Original best practices documentation included

#### Core Functionality
- Figma MCP tool integration
- Design token extraction
- BlockStudio field type mapping (30+ types)
- BEM naming convention
- CSS custom properties usage
- Template structure (block.json, index.php, style.scss)

### Documentation

- **README.md**: Comprehensive usage guide with examples
- **CHANGELOG.md**: This file documenting all changes
- **.blockstudio-state-example.json**: Example state file structure
- **Inline documentation**: Detailed comments throughout templates and utils

### Migration Guide

#### For Existing Users

**Continue using v1.0 (single-file):**
```bash
figma-to-blockstudio <url> --legacy
```

**Migrate to v2.0 (multi-file):**
```bash
# Start fresh with new structure
figma-to-blockstudio <url> --page home

# Continue incrementally
figma-to-blockstudio <url> --page about --update
```

**Benefits of upgrading:**
- Better organization for multi-page projects
- Automatic variant detection saves time
- State tracking prevents duplicate work
- Visual references make implementation easier
- Easier to maintain and update specs

### Known Issues

- Full document mode can be token-intensive for large Figma files
- Similarity detection requires manual review for edge cases
- Complex Figma animations need manual translation
- Absolute positioning may need adjustment

### Future Enhancements

Planned for v2.1+:
- [ ] Visual diff viewer for variants
- [ ] Direct WordPress export (skip specs)
- [ ] Twig/Blade template support
- [ ] File system watching for auto-sync
- [ ] Configurable similarity thresholds
- [ ] Machine learning for pattern recognition
- [ ] Component library management
- [ ] Design system integration

---

## [1.0.0] - 2025-XX-XX

### Initial Release

- Single-file spec generation (`design-spec.md`)
- Figma MCP integration
- Design token extraction
- BlockStudio field type mapping
- Template examples (block.json, index.php, style.scss)
- Common pattern recognition
- Best practices documentation

---

**Note**: This project uses [Semantic Versioning](https://semver.org/).

- **Major version** (2.x.x): Breaking changes to output structure or CLI
- **Minor version** (x.1.x): New features, backward compatible
- **Patch version** (x.x.1): Bug fixes and small improvements
