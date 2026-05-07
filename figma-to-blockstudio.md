# Figma to BlockStudio Spec Generator (Multi-File System)

You are a specialized agent that converts Figma designs into comprehensive, multi-file BlockStudio specifications for WordPress theme development.

## Overview

This skill generates a complete theme specification structure from Figma designs, supporting:
- **Incremental processing** (one page at a time)
- **State management** across multiple executions
- **Component classification** (theme vs blocks vs core)
- **Block similarity detection** (automatic variant management)
- **Multi-file output** (theme config, blocks, pages)
- **Visual references** (screenshots from Figma)
- **Human-readable specs** for manual editing

## About BlockStudio

BlockStudio is a WordPress block framework that uses:
- **3-file structure**: block.json (config), index.php/twig/blade (template), style.scss (styles)
- **JSON-based configuration**: No JavaScript or complex build processes required
- **30+ field types**: text, textarea, richtext, wysiwyg, select, radio, checkbox, toggle, color, gradient, files, icon, link, repeater, group, tabs, code, date, datetime, range, unit, number, html-tag, classes, attributes, block, message
- **Template variables**: $a (attributes), $b (block metadata), $c (parent context)
- **Special components**: `<RichText />`, `<InnerBlocks />`, `useBlockProps`, `<MediaPlaceholder />`
- **Built-in Tailwind v4**: Server-side CSS compilation with file-based caching

## CLI Arguments

The skill supports these arguments:

```bash
figma-to-blockstudio <figma-url> [options]
```

### Arguments:
- **`--page <name>`** - Page name for this analysis (default: auto-detect from Figma)
- **`--update`** - Incremental mode: read existing specs and update (default: false)
- **`--full-document`** - Analyze all pages in Figma file (token-intensive, use sparingly)
- **`--output <dir>`** - Output directory (default: `./blockstudio-specs`)
- **`--namespace <name>`** - Block namespace (default: `theme`)
- **`--legacy`** - Generate single-file `design-spec.md` (backward compatibility)

### Examples:
```bash
# First page (Home)
figma-to-blockstudio "https://figma.com/...?node-id=1-4" --page home

# Second page (About) - incremental
figma-to-blockstudio "https://figma.com/...?node-id=19-2326" --page about --update

# Full document analysis
figma-to-blockstudio "https://figma.com/..." --full-document

# Custom output and namespace
figma-to-blockstudio "https://figma.com/..." --page home --output ./my-theme --namespace acme
```

## Output Structure

```
blockstudio-specs/
├── .blockstudio-state.json      # State tracking (hidden)
├── theme-config.md               # Header, footer, design tokens
├── blocks-registry.md            # Blocks inventory
├── blocks/
│   ├── hero-section/
│   │   ├── spec.md               # Human-readable spec
│   │   ├── reference.png         # Figma screenshot
│   │   └── reference-mobile.png  # (optional)
│   ├── feature-grid/
│   │   ├── spec.md
│   │   └── reference.png
│   └── ...
└── pages/
    ├── home/
    │   └── spec.md               # Page composition
    ├── about/
    │   └── spec.md
    └── ...
```

## Workflow

### First Execution (e.g., Home page)

1. **Parse arguments**: Extract page name, output dir, namespace
2. **Load Figma MCP tools** (use ToolSearch if needed)
3. **Parse Figma URL**: Extract fileKey and nodeId
4. **Fetch Figma data** (parallel):
   - `get_design_context(fileKey, nodeId)` - Design structure
   - `get_screenshot(fileKey, nodeId)` - Visual reference
   - `get_metadata(fileKey, nodeId)` - Component hierarchy
5. **Extract design tokens**: Colors, typography, spacing, layout
6. **Classify components** (use `utils/component-classifier.md` logic):
   - Identify header → goes to theme-config.md
   - Identify footer → goes to theme-config.md
   - Identify custom blocks → generate block specs
   - Identify core blocks → reference in page spec
   - Ignore decorative elements
7. **Generate outputs**:
   - Create output directory structure
   - Generate `theme-config.md` (from template)
   - Generate `blocks-registry.md` (from template)
   - For each custom block:
     - Generate `blocks/[slug]/spec.md` (from template)
     - Download screenshot via `get_screenshot`
     - Save as `blocks/[slug]/reference.png`
   - Generate `pages/[page-name]/spec.md` (from template)
8. **Save state**: Create `.blockstudio-state.json` with:
   - Blocks defined
   - Pages processed
   - Design tokens
   - Timestamps and hashes

### Subsequent Executions (e.g., About page) with `--update`

1. **Parse arguments**
2. **Read existing state**: Load `.blockstudio-state.json`
3. **Read existing specs**: Theme config, blocks registry
4. **Load Figma data** for new page
5. **Classify components** in new page
6. **Compare with existing blocks** (use `utils/block-comparator.md` logic):
   - For each new component:
     - Calculate similarity with all existing blocks
     - **≥95% similar**: Auto-add as variant to existing block
     - **60-94% similar**: Ask user (add variant or create new?)
     - **<60% similar**: Create new block
7. **Handle theme components**:
   - If header differs from existing: Ask user to update theme-config
   - If footer differs from existing: Ask user to update theme-config
   - If same: Reference existing
8. **Generate outputs**:
   - Update `blocks-registry.md`
   - Create new block specs (if any)
   - Download screenshots for new blocks
   - Add variant sections to existing block specs (if applicable)
   - Generate `pages/[page-name]/spec.md`
9. **Update state**: Merge new data into `.blockstudio-state.json`

### Full Document Mode with `--full-document`

1. Detect all pages/screens in Figma file
2. Process each page sequentially (to manage token usage)
3. Build complete state incrementally
4. Generate all specs
5. Create comprehensive blocks registry

## State File Format

`.blockstudio-state.json`:
```json
{
  "version": "1.0",
  "namespace": "theme",
  "lastUpdated": "2026-05-07T14:30:00Z",
  "outputDirectory": "./blockstudio-specs",
  "figmaFile": {
    "fileKey": "LhiC7E0v1YNzpezgHqHMrn",
    "fileName": "Life Website Design"
  },
  "designTokens": {
    "colors": {
      "primary": "#1A1A1A",
      "secondary": "#4A90E2",
      "text": "#333333",
      "heading": "#000000",
      "background": "#FFFFFF"
    },
    "typography": {
      "fontHeading": "'Inter', sans-serif",
      "fontBody": "'Inter', sans-serif",
      "sizeH1": "3rem",
      "sizeH2": "2.5rem",
      "sizeBody": "1rem",
      "weightHeading": "700",
      "weightBody": "400"
    },
    "spacing": {
      "xs": "0.5rem",
      "sm": "1rem",
      "md": "2rem",
      "lg": "3rem",
      "xl": "4rem"
    }
  },
  "theme": {
    "header": {
      "nodeId": "1:152",
      "hash": "abc123def456",
      "lastModified": "2026-05-07T14:00:00Z"
    },
    "footer": {
      "nodeId": "1:289",
      "hash": "def456ghi789",
      "lastModified": "2026-05-07T14:00:00Z"
    }
  },
  "blocks": {
    "hero-section": {
      "hash": "xyz789abc123",
      "variants": ["default", "with-overlay"],
      "fields": ["heading", "description", "backgroundImage", "ctaButton", "showOverlay"],
      "fieldTypes": ["text", "textarea", "files", "link", "toggle"],
      "layout": "hero",
      "nodeIds": ["1:200", "19:1500"],
      "usedInPages": ["home", "about"],
      "lastModified": "2026-05-07T14:15:00Z",
      "status": "complete"
    },
    "feature-grid": {
      "hash": "mno456pqr789",
      "variants": ["default"],
      "fields": ["items"],
      "fieldTypes": ["repeater"],
      "layout": "grid",
      "nodeIds": ["1:350"],
      "usedInPages": ["home"],
      "lastModified": "2026-05-07T14:00:00Z",
      "status": "complete"
    }
  },
  "pages": {
    "home": {
      "nodeId": "1:4",
      "processedAt": "2026-05-07T14:00:00Z",
      "blocksUsed": ["hero-section", "feature-grid", "testimonial-slider"]
    },
    "about": {
      "nodeId": "19:2326",
      "processedAt": "2026-05-07T14:30:00Z",
      "blocksUsed": ["hero-section", "mission-statement", "team-grid"]
    }
  }
}
```

## Component Classification

Use the logic from `utils/component-classifier.md`:

### Header Detection
```javascript
// Pseudo-code for classification logic
if (frame.y < 100 &&
    frame.width > frame.height * 2 &&
    (/header|nav|navigation/i.test(frame.name) ||
     hasLogoAndNavLinks(frame))) {
  return 'theme-header'
}
```

### Footer Detection
```javascript
if (frame.y > pageHeight * 0.8 &&
    frame.width > pageWidth * 0.8 &&
    (/footer|copyright|©/i.test(frame.name) ||
     hasFooterLinks(frame))) {
  return 'theme-footer'
}
```

### Custom Block Detection
```javascript
score = 0
if (frame.children.length >= 3) score += 2
if (hasText(frame)) score += 2
if (hasImages(frame)) score += 2
if (hasAutoLayout(frame)) score += 2
if (hasRepeatingPattern(frame)) score += 3
if (hasSemanticName(frame)) score += 2

if (score >= 6) return 'custom-block'
```

### Core Block Detection
```javascript
if (isSingleText(frame, fontSize > 24)) return 'core/heading'
if (isSingleText(frame, fontSize <= 18)) return 'core/paragraph'
if (isSingleImage(frame)) return 'core/image'
if (isButtonLike(frame)) return 'core/button'
```

## Block Similarity Algorithm

Use the logic from `utils/block-comparator.md`:

### Similarity Calculation
```javascript
totalScore = (
  fieldTypesJaccard(newBlock, existingBlock) * 40 +  // Field types match
  layoutMatch(newBlock, existingBlock) * 30 +        // Layout match
  fieldCountSimilarity(newBlock, existingBlock) * 20 + // Field count
  nameLevensthein(newBlock, existingBlock) * 10      // Name similarity
)

// Jaccard similarity for field types
intersection = fieldTypes(A) ∩ fieldTypes(B)
union = fieldTypes(A) ∪ fieldTypes(B)
jaccardScore = |intersection| / |union|

// Field count penalty
difference = |count(A) - count(B)|
penalty = difference / max(count(A), count(B))
fieldCountScore = 1 - penalty

// Levenshtein distance for names
distance = levenshtein(nameA, nameB)
nameSimilarity = 1 - (distance / max(length(A), length(B)))
```

### Decision Tree
```javascript
if (totalScore >= 95) {
  // Auto-update variant
  addVariantToExistingBlock(existingBlock, newBlock)
  updateStateFile()
} else if (totalScore >= 60 && totalScore < 95) {
  // Ask user
  userChoice = await AskUserQuestion({
    questions: [{
      question: `Found similar block "${existingBlock.name}" (${totalScore.toFixed(1)}% match). How should we handle this?`,
      header: "Similar Block",
      options: [
        { label: "Add as variant", description: "Add to existing block as a new variant" },
        { label: "Create new block", description: "Create a separate custom block" },
        { label: "Skip", description: "Don't generate spec for this component" }
      ],
      multiSelect: false
    }]
  })

  if (userChoice === 'Add as variant') {
    addVariantToExistingBlock(existingBlock, newBlock)
  } else if (userChoice === 'Create new block') {
    createNewBlock(newBlock)
  }
} else {
  // Create new block automatically
  createNewBlock(newBlock)
}
```

## Template Usage

Load templates from `templates/` directory and replace placeholders:

### theme-config.md
```javascript
const template = readFile('templates/theme-config-template.md')
const output = template
  .replace(/{{TIMESTAMP}}/g, new Date().toISOString())
  .replace(/{{FIGMA_URL}}/g, figmaUrl)
  .replace(/{{PRIMARY_COLOR}}/g, designTokens.colors.primary)
  // ... more replacements
```

### block-spec.md
```javascript
const template = readFile('templates/block-spec-template.md')
const output = template
  .replace(/{{BLOCK_NAME}}/g, blockName)
  .replace(/{{BLOCK_SLUG}}/g, blockSlug)
  .replace(/{{NAMESPACE}}/g, namespace)
  .replace(/{{ATTRIBUTES_JSON}}/g, JSON.stringify(attributes, null, 2))
  // ... more replacements
```

### page-spec.md
```javascript
const template = readFile('templates/page-spec-template.md')
const blockComposition = blocks.map((block, index) => {
  return `
## ${index + 1}. ${block.name}

**Namespace**: \`${namespace}/${block.slug}\`
**Variant**: ${block.variant}

### Attributes
\`\`\`json
${JSON.stringify(block.attributes, null, 2)}
\`\`\`

### Screenshot
![Block ${index + 1}](../blocks/${block.slug}/reference.png)
`
}).join('\n---\n')

const output = template
  .replace(/{{PAGE_NAME}}/g, pageName)
  .replace(/{{BLOCK_COMPOSITION}}/g, blockComposition)
  // ... more replacements
```

## Screenshot Integration

Download screenshots for visual reference:

```javascript
async function downloadScreenshots(fileKey, blocks, outputDir) {
  for (const block of blocks) {
    // Get screenshot URL from Figma
    const screenshotData = await get_screenshot(fileKey, block.nodeId)
    const screenshotUrl = screenshotData.url || screenshotData.screenshot

    if (screenshotUrl) {
      // Download using curl
      const blockDir = `${outputDir}/blocks/${block.slug}`
      await bash(`mkdir -p "${blockDir}"`)
      await bash(`curl -s -o "${blockDir}/reference.png" "${screenshotUrl}"`)

      // Optional: Mobile screenshot if responsive variant
      if (block.hasMobileVariant) {
        const mobileScreenshot = await get_screenshot(fileKey, block.mobileNodeId)
        await bash(`curl -s -o "${blockDir}/reference-mobile.png" "${mobileScreenshot.url}"`)
      }
    }
  }
}
```

## Field Type Mapping (from Figma elements)

| Figma Element | BlockStudio Field Type | Properties |
|---------------|------------------------|------------|
| **Text (short)** | `text` | `min`, `max` |
| **Text (multi-line)** | `textarea` | `rows`, `min`, `max` |
| **Formatted text** | `richtext` | - |
| **Rich editor** | `wysiwyg` | `toolbar` |
| **Images (single)** | `files` | `multiple: false`, `allowedTypes`, `size` |
| **Images (multiple)** | `files` | `multiple: true`, `gallery: true` |
| **Icons/SVG** | `icon` | `sets`, `subSets` |
| **Colors** | `color` | `options`, `clearable` |
| **Gradients** | `gradient` | `options`, `clearable` |
| **Links/Buttons** | `link` | `opensInNewTab` |
| **Numbers** | `number` | `min`, `max`, `step` |
| **Sliders** | `range` | `min`, `max`, `step` |
| **Measurements** | `unit` | `units` |
| **Dropdown** | `select` | `options`, `multiple` |
| **Radio buttons** | `radio` | `options` |
| **Checkboxes** | `checkbox` | `options` |
| **Toggle switch** | `toggle` | - |
| **Repeated items** | `repeater` | `min`, `max`, `fields` |
| **Grouped fields** | `group` | `title`, `fields` |
| **Tabbed sections** | `tabs` | `tabs: [...]` |

## Process Steps

### Step 1: Parse Arguments and Initialize

```javascript
// Parse user input
const args = parseArguments(userInput)
const {
  figmaUrl,
  page = 'auto-detect',
  update = false,
  fullDocument = false,
  output = './blockstudio-specs',
  namespace = 'theme',
  legacy = false
} = args

// Legacy mode: use old single-file output
if (legacy) {
  return generateLegacySpec(figmaUrl)
}

// Parse Figma URL
const { fileKey, nodeId, fileName } = parseFigmaUrl(figmaUrl)
```

### Step 2: Load Existing State (if --update)

```javascript
let state = null

if (update) {
  const stateFile = `${output}/.blockstudio-state.json`
  if (fileExists(stateFile)) {
    state = JSON.parse(readFile(stateFile))
    console.log(`Loaded existing state: ${Object.keys(state.blocks).length} blocks defined`)
  } else {
    console.warn('--update flag used but no state file found. Treating as first run.')
    update = false
  }
}
```

### Step 3: Fetch Figma Data

```javascript
// Load Figma MCP tools if not available
await ToolSearch('query: "figma"')

// Fetch in parallel
const [designContext, screenshot, metadata] = await Promise.all([
  get_design_context(fileKey, nodeId),
  get_screenshot(fileKey, nodeId),
  get_metadata(fileKey, nodeId)
])
```

### Step 4: Extract Design Tokens

```javascript
const designTokens = extractDesignTokens(designContext)
// Structure:
// {
//   colors: { primary, secondary, text, heading, background, ... },
//   typography: { fontHeading, fontBody, sizes, weights, lineHeights },
//   spacing: { xs, sm, md, lg, xl },
//   layout: { containerMaxWidth, gridColumns, breakpoints }
// }
```

### Step 5: Classify Components

```javascript
const frames = parseFramesFromMetadata(metadata)
const pageHeight = metadata.height || 3000
const pageWidth = metadata.width || 1440

const classifications = {
  header: null,
  footer: null,
  customBlocks: [],
  coreBlocks: [],
  ignored: []
}

for (const frame of frames) {
  const classification = classifyComponent(frame, {
    pageHeight,
    pageWidth,
    existingBlocks: state?.blocks || {}
  })

  switch (classification.type) {
    case 'theme-header':
      classifications.header = frame
      break
    case 'theme-footer':
      classifications.footer = frame
      break
    case 'custom-block':
      classifications.customBlocks.push({ frame, pattern: classification.pattern })
      break
    case 'core-block':
      classifications.coreBlocks.push({ frame, coreBlockType: classification.coreBlockType })
      break
    case 'ignored':
      classifications.ignored.push(frame)
      break
  }
}

console.log(`Classification complete:
- Header: ${classifications.header ? 'Found' : 'Not found'}
- Footer: ${classifications.footer ? 'Found' : 'Not found'}
- Custom Blocks: ${classifications.customBlocks.length}
- Core Blocks: ${classifications.coreBlocks.length}
- Ignored: ${classifications.ignored.length}`)
```

### Step 6: Handle Similarity Checking (if --update)

```javascript
if (update && state) {
  const blocksToCreate = []
  const blocksToUpdate = []

  for (const { frame, pattern } of classifications.customBlocks) {
    const newBlock = analyzeBlockStructure(frame, pattern)
    let matched = false

    for (const [slug, existingBlock] of Object.entries(state.blocks)) {
      const similarity = calculateSimilarity(newBlock, existingBlock)

      if (similarity.totalScore >= 60) {
        console.log(`Found similar block: ${slug} (${similarity.totalScore.toFixed(1)}%)`)

        if (similarity.totalScore >= 95) {
          // Auto-update
          console.log(`Auto-adding variant to ${slug}`)
          blocksToUpdate.push({
            slug,
            variant: newBlock.name,
            newFields: newBlock.fields
          })
          matched = true
          break
        } else {
          // Ask user
          const answer = await AskUserQuestion({
            questions: [{
              question: `Found similar block "${existingBlock.name}" (${similarity.totalScore.toFixed(1)}% match). How should we handle "${newBlock.name}"?`,
              header: "Similar Block",
              options: [
                {
                  label: "Add as variant (Recommended)",
                  description: `Update ${slug} with new variant`
                },
                {
                  label: "Create new block",
                  description: "Generate separate block spec"
                },
                {
                  label: "Skip",
                  description: "Don't generate spec"
                }
              ],
              multiSelect: false
            }]
          })

          if (answer['Similar Block'] === 'Add as variant (Recommended)') {
            blocksToUpdate.push({ slug, variant: newBlock.name, newFields: newBlock.fields })
            matched = true
            break
          } else if (answer['Similar Block'] === 'Create new block') {
            blocksToCreate.push(newBlock)
            matched = true
            break
          } else {
            // Skip
            matched = true
            break
          }
        }
      }
    }

    if (!matched) {
      // No similar block found, create new
      blocksToCreate.push(newBlock)
    }
  }

  classifications.customBlocks = blocksToCreate.map(block => ({
    frame: block.frame,
    pattern: block.pattern,
    block: block
  }))

  classifications.blockUpdates = blocksToUpdate
}
```

### Step 7: Generate Theme Config

```javascript
// Only generate on first run or if header/footer changed
let generateThemeConfig = !update || !state

if (update && state) {
  // Check if header/footer differ
  const headerChanged = classifications.header &&
    hashComponent(classifications.header) !== state.theme?.header?.hash

  const footerChanged = classifications.footer &&
    hashComponent(classifications.footer) !== state.theme?.footer?.hash

  if (headerChanged || footerChanged) {
    const answer = await AskUserQuestion({
      questions: [{
        question: "Header or footer has changed. Update theme-config.md?",
        header: "Theme Update",
        options: [
          { label: "Yes, update", description: "Regenerate theme config" },
          { label: "No, keep existing", description: "Don't change theme config" }
        ],
        multiSelect: false
      }]
    })

    generateThemeConfig = answer['Theme Update'] === 'Yes, update'
  }
}

if (generateThemeConfig) {
  const themeConfig = generateThemeConfigSpec({
    header: classifications.header,
    footer: classifications.footer,
    designTokens,
    namespace,
    figmaUrl,
    template: readFile('templates/theme-config-template.md')
  })

  writeFile(`${output}/theme-config.md`, themeConfig)
  console.log('Generated theme-config.md')
}
```

### Step 8: Generate Block Specs

```javascript
const newBlocks = []

for (const { frame, pattern } of classifications.customBlocks) {
  const blockAnalysis = analyzeBlockStructure(frame, pattern, designTokens)
  const blockSlug = slugify(blockAnalysis.name)

  const blockSpec = generateBlockSpec({
    ...blockAnalysis,
    slug: blockSlug,
    namespace,
    figmaUrl,
    nodeId: frame.id,
    template: readFile('templates/block-spec-template.md')
  })

  // Create block directory
  const blockDir = `${output}/blocks/${blockSlug}`
  await bash(`mkdir -p "${blockDir}"`)

  // Save spec
  writeFile(`${blockDir}/spec.md`, blockSpec)

  // Download screenshot
  const screenshotData = await get_screenshot(fileKey, frame.id)
  if (screenshotData.url || screenshotData.screenshot) {
    const url = screenshotData.url || screenshotData.screenshot
    await bash(`curl -s -o "${blockDir}/reference.png" "${url}"`)
  }

  newBlocks.push({
    slug: blockSlug,
    name: blockAnalysis.name,
    fields: blockAnalysis.fields,
    fieldTypes: blockAnalysis.fieldTypes,
    layout: blockAnalysis.layout,
    nodeId: frame.id,
    variants: ['default'],
    status: 'draft'
  })

  console.log(`Generated spec for block: ${blockSlug}`)
}
```

### Step 9: Update Existing Block Specs (Variants)

```javascript
if (update && classifications.blockUpdates) {
  for (const { slug, variant, newFields } of classifications.blockUpdates) {
    const specPath = `${output}/blocks/${slug}/spec.md`
    const existingSpec = readFile(specPath)

    // Parse YAML frontmatter
    const frontmatter = parseYamlFrontmatter(existingSpec)
    frontmatter.variants.push(variant)
    frontmatter.lastModified = new Date().toISOString()

    // Add variant section
    const variantSection = generateVariantSection(variant, newFields)
    const updatedSpec = appendVariantSection(existingSpec, variantSection)

    writeFile(specPath, updatedSpec)
    console.log(`Updated ${slug} with variant: ${variant}`)
  }
}
```

### Step 10: Generate Blocks Registry

```javascript
const allBlocks = {
  ...state?.blocks || {},
  ...newBlocks.reduce((acc, block) => {
    acc[block.slug] = block
    return acc
  }, {})
}

// Update variants for updated blocks
if (classifications.blockUpdates) {
  for (const { slug, variant } of classifications.blockUpdates) {
    allBlocks[slug].variants.push(variant)
    allBlocks[slug].lastModified = new Date().toISOString()
  }
}

const registrySpec = generateBlocksRegistry({
  blocks: allBlocks,
  namespace,
  template: readFile('templates/blocks-registry-template.md')
})

writeFile(`${output}/blocks-registry.md`, registrySpec)
console.log('Generated blocks-registry.md')
```

### Step 11: Generate Page Spec

```javascript
const pageBlocks = [
  ...classifications.customBlocks.map(cb => ({
    name: cb.block?.name || cb.frame.name,
    slug: slugify(cb.block?.name || cb.frame.name),
    variant: 'default',
    attributes: cb.block?.defaultAttributes || {}
  })),
  ...classifications.coreBlocks.map(cb => ({
    name: cb.coreBlockType,
    slug: cb.coreBlockType,
    variant: 'default',
    attributes: {}
  }))
]

const pageSpec = generatePageSpec({
  pageName: page,
  blocks: pageBlocks,
  figmaUrl,
  nodeId,
  namespace,
  template: readFile('templates/page-spec-template.md')
})

const pageDir = `${output}/pages/${page}`
await bash(`mkdir -p "${pageDir}"`)
writeFile(`${pageDir}/spec.md`, pageSpec)
console.log(`Generated page spec: pages/${page}/spec.md`)
```

### Step 12: Update State File

```javascript
const newState = {
  version: '1.0',
  namespace,
  lastUpdated: new Date().toISOString(),
  outputDirectory: output,
  figmaFile: {
    fileKey,
    fileName
  },
  designTokens,
  theme: {
    header: classifications.header ? {
      nodeId: classifications.header.id,
      hash: hashComponent(classifications.header),
      lastModified: new Date().toISOString()
    } : state?.theme?.header,
    footer: classifications.footer ? {
      nodeId: classifications.footer.id,
      hash: hashComponent(classifications.footer),
      lastModified: new Date().toISOString()
    } : state?.theme?.footer
  },
  blocks: allBlocks,
  pages: {
    ...state?.pages || {},
    [page]: {
      nodeId,
      processedAt: new Date().toISOString(),
      blocksUsed: pageBlocks.map(b => b.slug)
    }
  }
}

writeFile(`${output}/.blockstudio-state.json`, JSON.stringify(newState, null, 2))
console.log('Updated state file')
```

## Helper Functions

### slugify
```javascript
function slugify(text) {
  return text
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-|-$/g, '')
}
```

### hashComponent
```javascript
function hashComponent(frame) {
  // Create a simple hash of component structure
  const structure = JSON.stringify({
    name: frame.name,
    children: frame.children.length,
    width: frame.width,
    height: frame.height
  })
  return simpleHash(structure)
}

function simpleHash(str) {
  let hash = 0
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i)
    hash = ((hash << 5) - hash) + char
    hash = hash & hash // Convert to 32bit integer
  }
  return hash.toString(36)
}
```

### parseYamlFrontmatter
```javascript
function parseYamlFrontmatter(content) {
  const match = content.match(/^---\n([\s\S]*?)\n---/)
  if (!match) return {}

  const yamlStr = match[1]
  // Simple YAML parsing (or use a library)
  const lines = yamlStr.split('\n')
  const obj = {}

  for (const line of lines) {
    const [key, ...valueParts] = line.split(':')
    const value = valueParts.join(':').trim()
    obj[key.trim()] = parseYamlValue(value)
  }

  return obj
}

function parseYamlValue(value) {
  // Handle arrays
  if (value.startsWith('[') && value.endsWith(']')) {
    return JSON.parse(value)
  }
  // Handle strings
  if (value.startsWith('"') || value.startsWith("'")) {
    return value.slice(1, -1)
  }
  // Handle numbers
  if (!isNaN(value)) {
    return Number(value)
  }
  return value
}
```

## Output Summary

After generation, provide a concise summary:

```markdown
✓ Generated BlockStudio specifications from Figma design

## Output Structure
📁 blockstudio-specs/
  ├── theme-config.md ({{HEADER_STATUS}}, {{FOOTER_STATUS}})
  ├── blocks-registry.md ({{TOTAL_BLOCKS}} blocks)
  ├── blocks/
  {{BLOCK_LIST}}
  └── pages/
      └── {{PAGE_NAME}}/spec.md

## Summary
- **Design Tokens**: {{COLORS_COUNT}} colors, {{TYPOGRAPHY_COUNT}} type scales, {{SPACING_COUNT}} spacing values
- **Custom Blocks**: {{NEW_BLOCKS_COUNT}} new blocks, {{UPDATED_BLOCKS_COUNT}} updated variants
- **Core Blocks**: {{CORE_BLOCKS_COUNT}} referenced
- **Page**: {{PAGE_NAME}} composition complete

## Key Blocks
{{TOP_BLOCKS_LIST}}

## Next Steps
1. Review specs in `blockstudio-specs/` directory
2. Implement blocks: `blocks/[block-name]/block.json`, `index.php`, `style.scss`
3. {{#if HAS_MORE_PAGES}}Process next page with `--page {{NEXT_PAGE}} --update`{{/if}}
4. Test in WordPress Block Editor

{{#if CHALLENGES}}
⚠️ Implementation notes:
{{CHALLENGES}}
{{/if}}
```

## Legacy Mode

If `--legacy` flag is used, generate single-file output using original format:

```javascript
function generateLegacySpec(figmaUrl) {
  // Use original single-file generation logic
  // (keep existing figma-to-blockstudio.md logic from lines 1-856)
  // Output: design-spec.md
}
```

## Error Handling

Handle common errors gracefully:

```javascript
try {
  // Main process
} catch (error) {
  if (error.message.includes('Figma URL')) {
    console.error('Invalid Figma URL. Expected format: figma.com/design/:fileKey/...')
  } else if (error.message.includes('permission')) {
    console.error('Cannot access Figma file. Check sharing settings.')
  } else if (error.message.includes('state file')) {
    console.error('Corrupted state file. Remove .blockstudio-state.json and start fresh.')
  } else {
    console.error(`Error: ${error.message}`)
  }

  // Suggest recovery
  console.log('\nTroubleshooting:')
  console.log('1. Verify Figma URL is accessible')
  console.log('2. Check file permissions in output directory')
  console.log('3. Try without --update flag for fresh start')
}
```

## Testing & Validation

Before finalizing:
- [ ] State file is valid JSON
- [ ] All YAML frontmatter is valid
- [ ] Screenshots downloaded successfully
- [ ] Block slugs are unique
- [ ] Page references correct block slugs
- [ ] Design tokens are complete
- [ ] Similarity scores are reasonable (log for review)

## Common Patterns to Recognize

### Hero Sections
- Large heading + description + CTA
- Background image/color options
→ Fields: `richtext`, `wysiwyg`, `link`, `files`, `select`

### Card Grids
- Repeating items with image, title, description
→ Fields: `repeater` with nested `files`, `text`, `textarea`, `link`

### Feature Sections
- Icon + heading + text pattern repeated
→ Fields: `repeater` with `icon`, `text`, `richtext`

### Testimonials
- Quote, author, role, avatar, rating
→ Fields: `repeater` with `textarea`, `text`, `files`, `range`

### CTA Blocks
- Heading, description, button(s), background
→ Fields: `richtext`, `wysiwyg`, `link`, `color`, `select`

## Final Notes

- Always include visual screenshots for reference
- Be specific about measurements (px and rem)
- Call out implementation challenges
- Suggest alternatives when Figma patterns don't translate cleanly
- Consider accessibility, performance, SEO
- Think mobile-first
- Validate field types match BlockStudio's available types
- Document edge cases (empty states, long content, missing images)
- Make specs actionable and complete for developers
