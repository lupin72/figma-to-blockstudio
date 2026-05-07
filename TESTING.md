# Testing Guide for Figma to BlockStudio v2.0

This guide provides step-by-step testing procedures to validate the multi-file system implementation.

## Prerequisites

- Access to Figma files for testing
- figma-remote-mcp tools configured
- Test Figma file with multiple pages (recommended structure):
  - Home page (node-id: 1-4)
  - About page (node-id: 19-1403 or similar)
  - Services page (optional, node-id: 35-xxxx)

## Test Suite

### Test 1: First Execution (Home Page)

#### Objective
Verify that the skill creates the complete directory structure and generates all necessary files.

#### Command
```bash
figma-to-blockstudio "https://figma.com/design/LhiC7E0v1YNzpezgHqHMrn/Life?node-id=1-4" --page home --output ./test-output
```

#### Expected Results

✅ **Directory Structure Created**
```
test-output/
├── .blockstudio-state.json
├── theme-config.md
├── blocks-registry.md
├── blocks/
│   ├── [block-1]/
│   │   ├── spec.md
│   │   └── reference.png
│   ├── [block-2]/
│   │   ├── spec.md
│   │   └── reference.png
│   └── ...
└── pages/
    └── home/
        └── spec.md
```

✅ **File Validations**

**theme-config.md:**
- [ ] Contains YAML frontmatter
- [ ] Has Design Tokens section with CSS variables
- [ ] Has Header Structure section (if header detected)
- [ ] Has Footer Structure section (if footer detected)
- [ ] Design tokens are complete (colors, typography, spacing, layout)

**blocks-registry.md:**
- [ ] Contains YAML frontmatter
- [ ] Has table of all blocks with status
- [ ] Lists variants for each block
- [ ] Shows pages where blocks are used
- [ ] Has statistics section

**blocks/[slug]/spec.md:**
- [ ] Contains valid YAML frontmatter
- [ ] Has visual reference section
- [ ] Has complete block.json configuration
- [ ] Has template code (index.php)
- [ ] Has SCSS with BEM naming
- [ ] Uses CSS custom properties (var(--))
- [ ] Has implementation checklist

**blocks/[slug]/reference.png:**
- [ ] Screenshot file exists
- [ ] Image is readable (not corrupted)
- [ ] Shows the component from Figma

**pages/home/spec.md:**
- [ ] Contains YAML frontmatter
- [ ] Lists all blocks in order
- [ ] Each block has namespace, variant, attributes
- [ ] Has page structure diagram
- [ ] Has implementation checklist

**.blockstudio-state.json:**
- [ ] Valid JSON (parse without errors)
- [ ] Contains `version`, `namespace`, `lastUpdated`
- [ ] Has `designTokens` object
- [ ] Has `theme` object with header/footer
- [ ] Has `blocks` object with block definitions
- [ ] Has `pages` object with home page entry
- [ ] Each block has: hash, variants, fields, fieldTypes, layout, nodeIds

✅ **Component Classification**

Console output should show:
```
Classification complete:
- Header: Found
- Footer: Found
- Custom Blocks: X
- Core Blocks: Y
- Ignored: Z
```

Verify:
- [ ] Header was correctly identified (if exists)
- [ ] Footer was correctly identified (if exists)
- [ ] Custom blocks have 3+ fields
- [ ] Core blocks are simple (single text/image)

✅ **Design Tokens Extracted**

In theme-config.md, verify:
- [ ] At least 5 colors defined
- [ ] Typography scales (h1-h4, body) defined
- [ ] Spacing scale (xs-xl) defined
- [ ] Layout values (container, grid, breakpoints) defined

---

### Test 2: Incremental Update (About Page)

#### Objective
Verify that the skill reads existing state, detects similar blocks, and updates appropriately.

#### Command
```bash
figma-to-blockstudio "https://figma.com/design/LhiC7E0v1YNzpezgHqHMrn/Life?node-id=19-1403" --page about --update --output ./test-output
```

#### Expected Results

✅ **State Loaded**

Console output should show:
```
Loaded existing state: X blocks defined
```

✅ **Similarity Detection**

If the about page has a hero similar to home:
- [ ] Console shows: `Found similar block: hero-section (XX%)`
- [ ] If ≥95%: Auto-adds variant with message
- [ ] If 60-94%: Asks user for confirmation
- [ ] If <60%: Creates new block

**For ≥95% similarity:**
```
Auto-adding variant to hero-section
Updated hero-section with variant: centered
```

**For 60-94% similarity:**
```
User prompt appears with options:
- Add as variant (Recommended)
- Create new block
- Skip
```

✅ **Files Updated**

**blocks-registry.md:**
- [ ] Updated with new blocks (if any)
- [ ] Variant count increased for updated blocks
- [ ] "Used In Pages" includes "about" for shared blocks

**blocks/hero-section/spec.md** (if variant added):
- [ ] Frontmatter `variants` array includes new variant
- [ ] Frontmatter `lastModified` timestamp updated
- [ ] New variant section added at bottom
- [ ] Variant section documents differences from default

**pages/about/spec.md:**
- [ ] New file created
- [ ] Lists blocks used in about page
- [ ] References correct block slugs and variants

**.blockstudio-state.json:**
- [ ] `blocks.hero-section.variants` array updated (if variant added)
- [ ] `blocks.hero-section.usedInPages` includes "about"
- [ ] `pages.about` entry added
- [ ] `lastUpdated` timestamp is current

**theme-config.md:**
- [ ] If header/footer unchanged: file not modified (check timestamp)
- [ ] If header/footer changed: user was prompted
- [ ] If updated: lastModified timestamp is current

✅ **New Blocks Created**

For any blocks unique to about page:
- [ ] New directory in `blocks/[slug]/`
- [ ] New spec.md file
- [ ] New reference.png screenshot
- [ ] Added to blocks-registry.md
- [ ] Added to state file

---

### Test 3: Similarity Detection Edge Cases

#### Objective
Test the similarity algorithm with controlled examples.

#### Test Case 3.1: Very Similar Block (≥95%)

**Setup:**
Create or find a component that differs only by one toggle field from an existing block.

**Expected:**
- [ ] Similarity score ≥95%
- [ ] Automatic variant addition
- [ ] No user prompt
- [ ] Console: "Auto-adding variant to [block-slug]"

#### Test Case 3.2: Moderately Similar Block (60-94%)

**Setup:**
Create or find a component with same layout but different field types (e.g., icon vs image).

**Expected:**
- [ ] Similarity score 60-94%
- [ ] User prompt appears
- [ ] Options: Add variant / Create new / Skip
- [ ] User choice is respected

#### Test Case 3.3: Different Block (<60%)

**Setup:**
Create or find a completely different component (e.g., hero vs testimonial slider).

**Expected:**
- [ ] Similarity score <60%
- [ ] Automatic new block creation
- [ ] No user prompt
- [ ] Console: "Creating new block (only XX% similar)"

---

### Test 4: Component Classification

#### Objective
Verify that components are classified correctly.

#### Test Case 4.1: Header Detection

**Setup:**
Page with header at top (y < 100px), containing nav links.

**Expected:**
- [ ] Classified as `theme-header`
- [ ] Added to theme-config.md (not blocks/)
- [ ] Console: "Header: Found"

#### Test Case 4.2: Footer Detection

**Setup:**
Page with footer at bottom (y > 80% page height), containing copyright.

**Expected:**
- [ ] Classified as `theme-footer`
- [ ] Added to theme-config.md (not blocks/)
- [ ] Console: "Footer: Found"

#### Test Case 4.3: Core Block Detection

**Setup:**
Single heading layer, single paragraph, or single image.

**Expected:**
- [ ] Classified as `core-block`
- [ ] Type identified: `core/heading`, `core/paragraph`, or `core/image`
- [ ] Referenced in page spec (not in blocks/)
- [ ] Console: "Core Blocks: X"

#### Test Case 4.4: Custom Block Detection

**Setup:**
Component with 3+ children, text + image, complex layout.

**Expected:**
- [ ] Classified as `custom-block`
- [ ] Block spec generated in blocks/
- [ ] Console: "Custom Blocks: X"

#### Test Case 4.5: Ignored Elements

**Setup:**
Background shapes, decorative elements, hidden layers.

**Expected:**
- [ ] Classified as `ignored`
- [ ] No spec generated
- [ ] Console: "Ignored: X"

---

### Test 5: Full Document Mode

#### Objective
Verify that full document analysis works for multi-page files.

#### Command
```bash
figma-to-blockstudio "https://figma.com/design/LhiC7E0v1YNzpezgHqHMrn/Life" --full-document --output ./test-full
```

#### Expected Results

✅ **All Pages Processed**
- [ ] Console shows: "Detecting all pages in Figma file..."
- [ ] Lists all pages found
- [ ] Processes each page sequentially

✅ **Complete Output**
- [ ] All pages have specs in `pages/[page-name]/`
- [ ] All blocks across all pages are in blocks-registry.md
- [ ] Shared blocks are identified (not duplicated)
- [ ] State file includes all pages and blocks

✅ **Performance**
- [ ] Process completes without errors
- [ ] Token usage is logged
- [ ] Large files may take time (expected)

---

### Test 6: Custom Arguments

#### Test Case 6.1: Custom Output Directory

**Command:**
```bash
figma-to-blockstudio <url> --page home --output ./my-custom-theme
```

**Expected:**
- [ ] Output is created in `./my-custom-theme/`
- [ ] State file path is correct in state.json

#### Test Case 6.2: Custom Namespace

**Command:**
```bash
figma-to-blockstudio <url> --page home --namespace acme
```

**Expected:**
- [ ] All blocks have namespace `acme/block-name`
- [ ] State file has `"namespace": "acme"`
- [ ] Blocks-registry shows correct namespace

---

### Test 7: Legacy Mode

#### Objective
Verify backward compatibility with v1.0 single-file output.

#### Command
```bash
figma-to-blockstudio <url> --legacy
```

#### Expected Results

✅ **Single File Output**
- [ ] Generates `design-spec.md` (not multi-file structure)
- [ ] File contains all sections from original format
- [ ] No state file created
- [ ] No directory structure created

---

### Test 8: Error Handling

#### Test Case 8.1: Invalid Figma URL

**Command:**
```bash
figma-to-blockstudio "https://invalid-url.com"
```

**Expected:**
- [ ] Error message: "Invalid Figma URL"
- [ ] Suggests correct format
- [ ] Process stops gracefully

#### Test Case 8.2: Update Without State File

**Command:**
```bash
figma-to-blockstudio <url> --page test --update --output ./empty-dir
```

**Expected:**
- [ ] Warning: "No state file found. Treating as first run."
- [ ] Proceeds as first execution (not incremental)

#### Test Case 8.3: Corrupted State File

**Setup:**
Manually corrupt `.blockstudio-state.json` (invalid JSON).

**Command:**
```bash
figma-to-blockstudio <url> --page test --update
```

**Expected:**
- [ ] Error: "Corrupted state file"
- [ ] Suggests removing and starting fresh
- [ ] Process stops gracefully

---

### Test 9: Screenshot Integration

#### Objective
Verify that screenshots are downloaded correctly.

#### Expected Results

✅ **For Each Block**
- [ ] `blocks/[slug]/reference.png` exists
- [ ] File size > 0 (not empty)
- [ ] Image can be opened and viewed
- [ ] Shows correct component from Figma

✅ **Console Output**
- [ ] Shows: "Downloading screenshot for [block-name]..."
- [ ] No curl errors

---

### Test 10: Template Substitution

#### Objective
Verify that template placeholders are replaced correctly.

#### Checks for theme-config.md
- [ ] No `{{PLACEHOLDER}}` syntax remaining
- [ ] All design token values filled
- [ ] Timestamps are valid ISO format
- [ ] Figma URLs are correct

#### Checks for block-spec.md
- [ ] No `{{PLACEHOLDER}}` syntax remaining
- [ ] Block name and slug are correct
- [ ] Namespace is correct
- [ ] Attributes JSON is valid JSON
- [ ] Template code has no placeholders
- [ ] SCSS has no placeholders

#### Checks for page-spec.md
- [ ] No `{{PLACEHOLDER}}` syntax remaining
- [ ] Page name is correct
- [ ] Block composition is filled
- [ ] All referenced blocks exist

---

### Test 11: YAML Frontmatter Validation

#### Objective
Verify that all YAML frontmatter is valid and parsable.

#### Tools
```bash
# Install yaml-lint if not available
npm install -g yaml-lint

# Validate frontmatter in each file
for file in test-output/blocks/*/spec.md; do
  echo "Validating: $file"
  head -20 "$file" | yaml-lint
done
```

#### Expected
- [ ] All YAML frontmatter parses without errors
- [ ] All required fields present (name, slug, namespace, version, etc.)
- [ ] Arrays are valid YAML syntax
- [ ] Dates are ISO 8601 format

---

## Validation Checklist

After running all tests, verify:

### File Structure
- [ ] All expected directories created
- [ ] All expected files created
- [ ] No unexpected files or directories
- [ ] File permissions are correct

### Content Quality
- [ ] All specs are human-readable
- [ ] No broken markdown formatting
- [ ] Code blocks have proper syntax highlighting
- [ ] Tables are formatted correctly
- [ ] Links work (internal references)

### Technical Correctness
- [ ] State file is valid JSON
- [ ] YAML frontmatter is valid
- [ ] Block.json configurations are valid JSON
- [ ] PHP template code is syntactically valid
- [ ] SCSS is syntactically valid

### Functional Requirements
- [ ] State management works across executions
- [ ] Similarity detection works accurately
- [ ] Component classification is correct
- [ ] Design tokens are complete
- [ ] Screenshots are downloaded

### User Experience
- [ ] Console output is clear and helpful
- [ ] Error messages are actionable
- [ ] User prompts are clear
- [ ] Progress is indicated
- [ ] Summary is comprehensive

---

## Performance Testing

### Token Usage
Monitor token usage for different scenarios:

```
| Scenario | Approximate Tokens | Time |
|----------|-------------------|------|
| Single page, 5 blocks | ~10K tokens | 30s |
| Single page, 10 blocks | ~15K tokens | 45s |
| Update mode, similar blocks | ~5K tokens | 20s |
| Update mode, new blocks | ~12K tokens | 40s |
| Full document, 3 pages | ~30K tokens | 90s |
```

- [ ] Token usage is reasonable
- [ ] No unnecessary MCP calls
- [ ] Parallel calls are used where possible

---

## Regression Testing

After any changes to the skill:

1. Run Test 1 (First execution)
2. Run Test 2 (Incremental update)
3. Run Test 3.1 (High similarity)
4. Run Test 7 (Legacy mode)

All four should pass to ensure no regressions.

---

## Bug Reporting

If tests fail, report with:
- Test case number
- Command used
- Expected result
- Actual result
- State file content (if relevant)
- Console output
- File structure (tree output)

---

## Success Criteria

The implementation is considered successful if:
- [ ] 100% of Test 1 (First execution) passes
- [ ] 100% of Test 2 (Incremental update) passes
- [ ] 90%+ of Test 3 (Similarity detection) passes
- [ ] 90%+ of Test 4 (Classification) passes
- [ ] 100% of Test 7 (Legacy mode) passes
- [ ] All YAML frontmatter is valid
- [ ] All JSON files are valid
- [ ] All screenshots download successfully

---

**Testing completed on**: [DATE]
**Tested by**: [NAME]
**Version**: 2.0.0
**Result**: [PASS/FAIL with notes]
