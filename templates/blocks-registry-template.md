---
type: blocks-registry
version: 1.0
namespace: {{NAMESPACE}}
lastUpdated: {{TIMESTAMP}}
---

# Blocks Registry

> Namespace: `{{NAMESPACE}}`
> Total blocks: {{TOTAL_BLOCKS}}
> Last updated: {{TIMESTAMP}}

## Overview

This registry tracks all custom blocks in the theme. Each block is defined in the `blocks/` directory with its spec, templates, and visual references.

---

## Block Inventory

| Slug | Name | Status | Variants | Used In Pages | Last Modified |
|------|------|--------|----------|---------------|---------------|
{{BLOCK_ROWS}}

### Status Legend
- ✅ **Complete**: Fully implemented and tested
- 🚧 **In Progress**: Spec created, implementation pending
- 📝 **Draft**: Initial spec, needs review
- 🔄 **Variant Updated**: New variant added to existing block

---

## Blocks by Category

### Hero/Banner Blocks
{{HERO_BLOCKS}}

### Content Blocks
{{CONTENT_BLOCKS}}

### Feature/Services Blocks
{{FEATURE_BLOCKS}}

### Testimonial/Review Blocks
{{TESTIMONIAL_BLOCKS}}

### CTA Blocks
{{CTA_BLOCKS}}

### Other Blocks
{{OTHER_BLOCKS}}

---

## Block Dependencies

### Shared Components
These elements are used across multiple blocks:
{{SHARED_COMPONENTS}}

### Common Patterns
{{COMMON_PATTERNS}}

---

## Implementation Priority

### High Priority (Core Pages)
{{HIGH_PRIORITY_BLOCKS}}

### Medium Priority (Secondary Pages)
{{MEDIUM_PRIORITY_BLOCKS}}

### Low Priority (Nice to Have)
{{LOW_PRIORITY_BLOCKS}}

---

## Block Relationships

### Parent-Child Relationships
{{PARENT_CHILD_RELATIONSHIPS}}

### Block Patterns
Pre-configured combinations of blocks:
{{BLOCK_PATTERNS}}

---

## Statistics

- **Total Custom Blocks**: {{TOTAL_BLOCKS}}
- **Total Variants**: {{TOTAL_VARIANTS}}
- **Blocks with Repeaters**: {{BLOCKS_WITH_REPEATERS}}
- **Blocks with InnerBlocks**: {{BLOCKS_WITH_INNERBLOCKS}}
- **Average Fields per Block**: {{AVG_FIELDS}}

---

## Development Notes

{{DEVELOPMENT_NOTES}}

---

## Quick Reference

### Creating a New Block
1. Generate spec: `figma-to-blockstudio <url> --page <name>`
2. Review spec: `blocks/<block-slug>/spec.md`
3. Implement: `blocks/<block-slug>/block.json`, `index.php`, `style.scss`
4. Update this registry
5. Test in Block Editor

### Updating an Existing Block
1. Regenerate with `--update` flag
2. Review variant changes in spec
3. Update implementation
4. Test existing pages using the block

---

Last updated: {{TIMESTAMP}}
