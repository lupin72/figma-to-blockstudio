# Block Similarity Comparator

This utility provides algorithms for comparing blocks to determine if a new block is similar enough to an existing one to be treated as a variant rather than a new block.

## Similarity Algorithm

The algorithm uses a weighted scoring system (0-100%) based on four key factors:

### 1. Field Types Match (40% weight)

Uses **Jaccard similarity coefficient** on the set of field types:

```
Jaccard(A, B) = |A ∩ B| / |A ∪ B|
```

**Example:**
```javascript
Block A: ['text', 'textarea', 'files', 'link', 'color']
Block B: ['text', 'textarea', 'files', 'toggle']

Intersection: ['text', 'textarea', 'files'] = 3 items
Union: ['text', 'textarea', 'files', 'link', 'color', 'toggle'] = 6 items

Jaccard = 3/6 = 0.50
Field Types Score = 0.50 * 40 = 20 points
```

### 2. Layout Match (30% weight)

Compares layout structure (exact match or not):

**Layout patterns:**
- `stack` - Vertical stacking
- `grid` - Grid layout
- `flex` - Flexbox layout
- `split` - Two-column split
- `hero` - Hero banner style
- `card` - Card layout
- `list` - List layout

**Scoring:**
- Exact match: 30 points
- No match: 0 points

**Example:**
```javascript
Block A: layout = 'grid'
Block B: layout = 'grid'
Layout Score = 30 points

Block A: layout = 'stack'
Block B: layout = 'flex'
Layout Score = 0 points
```

### 3. Field Count Similarity (20% weight)

Penalizes blocks with significantly different field counts:

```
Difference = |count_A - count_B|
Max Count = max(count_A, count_B)
Penalty = Difference / Max Count

Field Count Score = (1 - Penalty) * 20
```

**Example:**
```javascript
Block A: 5 fields
Block B: 6 fields

Difference = |5 - 6| = 1
Max Count = 6
Penalty = 1/6 = 0.167

Field Count Score = (1 - 0.167) * 20 = 16.66 points
```

### 4. Name Similarity (10% weight)

Uses **Levenshtein distance** normalized by length:

```
Levenshtein Distance = minimum edits to transform string A to B
Similarity = 1 - (Distance / max(length_A, length_B))
Name Score = Similarity * 10
```

**Example:**
```javascript
Block A: "hero-section"
Block B: "hero-banner"

Distance = 5 (replace "section" with "banner")
Max Length = 12
Similarity = 1 - (5/12) = 0.583

Name Score = 0.583 * 10 = 5.83 points
```

## Total Score Calculation

```
Total Score = Field Types Score + Layout Score + Field Count Score + Name Score
```

**Maximum possible score: 100 points**

## Decision Tree

Based on the total similarity score:

### ≥95% Similarity → **Update Variant Automatically**
The blocks are nearly identical. Automatically add as a variant to the existing block.

**Action:**
- Add variant name to existing block spec
- Document differences in variants section
- Update `.blockstudio-state.json`
- No user confirmation needed

**Example:**
```
Existing: "Hero Section" (score: 97%)
- Same fields: heading, description, image, cta
- Same layout: hero
- Only difference: overlay toggle added

Action: Add "with-overlay" variant to hero-section block
```

### 60-94% Similarity → **Ask User**
The blocks are similar but have meaningful differences. User should decide.

**Action:**
- Present comparison table to user
- Show differences clearly
- Offer three options:
  1. Add as variant to existing block
  2. Create new block
  3. Skip (don't generate spec)

**Example:**
```
Existing: "Hero Section" (score: 78%)
Differences:
- New block adds video field
- Layout changed from 'hero' to 'split'
- 2 new fields, 1 removed field

Question: "Found similar block 'hero-section'. Add as variant or create new block?"
```

### <60% Similarity → **Create New Block**
The blocks are significantly different. Create a new block automatically.

**Action:**
- Generate new block spec
- Add to blocks registry
- Update state file
- No user confirmation needed

**Example:**
```
Existing: "Hero Section" (score: 42%)
- Different layout: hero vs card-grid
- Different field types: single image vs repeater
- Different purpose: banner vs content grid

Action: Create new block "feature-grid"
```

## Implementation Pseudo-Code

```javascript
function calculateSimilarity(newBlock, existingBlock) {
  // 1. Field Types Similarity (40%)
  const fieldTypesA = new Set(newBlock.fields.map(f => f.type))
  const fieldTypesB = new Set(existingBlock.fields.map(f => f.type))
  const intersection = [...fieldTypesA].filter(x => fieldTypesB.has(x)).length
  const union = new Set([...fieldTypesA, ...fieldTypesB]).size
  const jaccardScore = intersection / union
  const fieldTypesScore = jaccardScore * 40

  // 2. Layout Similarity (30%)
  const layoutScore = newBlock.layout === existingBlock.layout ? 30 : 0

  // 3. Field Count Similarity (20%)
  const countA = newBlock.fields.length
  const countB = existingBlock.fields.length
  const difference = Math.abs(countA - countB)
  const maxCount = Math.max(countA, countB)
  const penalty = difference / maxCount
  const fieldCountScore = (1 - penalty) * 20

  // 4. Name Similarity (10%)
  const distance = levenshteinDistance(newBlock.name, existingBlock.name)
  const maxLength = Math.max(newBlock.name.length, existingBlock.name.length)
  const nameSimilarity = 1 - (distance / maxLength)
  const nameScore = nameSimilarity * 10

  // Total
  const totalScore = fieldTypesScore + layoutScore + fieldCountScore + nameScore

  return {
    totalScore,
    breakdown: {
      fieldTypes: fieldTypesScore,
      layout: layoutScore,
      fieldCount: fieldCountScore,
      name: nameScore
    }
  }
}

function levenshteinDistance(str1, str2) {
  const matrix = []

  for (let i = 0; i <= str2.length; i++) {
    matrix[i] = [i]
  }

  for (let j = 0; j <= str1.length; j++) {
    matrix[0][j] = j
  }

  for (let i = 1; i <= str2.length; i++) {
    for (let j = 1; j <= str1.length; j++) {
      if (str2.charAt(i - 1) === str1.charAt(j - 1)) {
        matrix[i][j] = matrix[i - 1][j - 1]
      } else {
        matrix[i][j] = Math.min(
          matrix[i - 1][j - 1] + 1, // substitution
          matrix[i][j - 1] + 1,     // insertion
          matrix[i - 1][j] + 1      // deletion
        )
      }
    }
  }

  return matrix[str2.length][str1.length]
}

function decideSimilarityAction(score, newBlock, existingBlock) {
  if (score >= 95) {
    return {
      action: 'update_variant',
      confidence: 'high',
      message: `Auto-updating ${existingBlock.slug} with new variant`,
      requiresConfirmation: false
    }
  } else if (score >= 60) {
    return {
      action: 'ask_user',
      confidence: 'medium',
      message: `Found similar block "${existingBlock.name}" (${score.toFixed(1)}% match). Add as variant or create new block?`,
      requiresConfirmation: true,
      options: [
        'Add as variant',
        'Create new block',
        'Skip'
      ]
    }
  } else {
    return {
      action: 'create_new',
      confidence: 'high',
      message: `Creating new block (only ${score.toFixed(1)}% similar to existing)`,
      requiresConfirmation: false
    }
  }
}
```

## Usage Example

```javascript
// When processing a new block from Figma:
const newBlock = {
  name: "hero-banner",
  layout: "hero",
  fields: [
    { type: "text", name: "heading" },
    { type: "textarea", name: "description" },
    { type: "files", name: "backgroundImage" },
    { type: "link", name: "ctaButton" },
    { type: "toggle", name: "showOverlay" }  // New field
  ]
}

const existingBlocks = readStateFile().blocks

for (const [slug, existingBlock] of Object.entries(existingBlocks)) {
  const similarity = calculateSimilarity(newBlock, existingBlock)
  const decision = decideSimilarityAction(similarity.totalScore, newBlock, existingBlock)

  console.log(`Comparing with ${slug}: ${similarity.totalScore.toFixed(1)}%`)
  console.log(`Action: ${decision.action}`)

  if (similarity.totalScore >= 60) {
    // This block is similar enough to consider
    if (decision.requiresConfirmation) {
      // Ask user via AskUserQuestion tool
      const userChoice = await askUser(decision.message, decision.options)
      if (userChoice === 'Add as variant') {
        addVariantToExistingBlock(slug, newBlock)
      } else if (userChoice === 'Create new block') {
        createNewBlock(newBlock)
      }
    } else {
      // Auto-execute based on confidence
      if (decision.action === 'update_variant') {
        addVariantToExistingBlock(slug, newBlock)
      }
    }
    break  // Found a match, stop comparing
  }
}
```

## Optimization Notes

### Performance
- Run comparisons in parallel for multiple existing blocks
- Cache Levenshtein distances for frequently compared names
- Short-circuit if field count difference > 50% (can't be >60% similar)

### Accuracy Improvements
- Consider field names, not just types (future enhancement)
- Weight recent blocks higher (temporal relevance)
- Learn from user corrections (machine learning enhancement)

### Edge Cases
- Empty blocks: always create new
- Identical blocks: 100% score, update variant
- Single-field blocks: rely more on name similarity

## Testing

### Test Case 1: High Similarity (≥95%)
```
Block A: Hero Section
- Fields: heading, description, image, cta
- Layout: hero

Block B: Hero with Overlay
- Fields: heading, description, image, cta, overlay
- Layout: hero

Expected Score: ~96%
Expected Action: Update variant
```

### Test Case 2: Medium Similarity (60-94%)
```
Block A: Feature Grid
- Fields: [repeater of icon, title, text]
- Layout: grid

Block B: Service Cards
- Fields: [repeater of image, title, text, link]
- Layout: grid

Expected Score: ~72%
Expected Action: Ask user
```

### Test Case 3: Low Similarity (<60%)
```
Block A: Hero Banner
- Layout: hero
- Fields: heading, image, cta

Block B: Testimonial Slider
- Layout: carousel
- Fields: [repeater of quote, author, avatar]

Expected Score: ~25%
Expected Action: Create new block
```

## State File Integration

When a variant is added, update `.blockstudio-state.json`:

```json
{
  "blocks": {
    "hero-section": {
      "hash": "abc123",
      "variants": ["default", "with-overlay", "centered"],  // Added "centered"
      "fields": ["heading", "description", "image", "cta", "alignment"],
      "lastModified": "2026-05-07T14:30:00Z"
    }
  }
}
```

## References

- [Jaccard Similarity Coefficient](https://en.wikipedia.org/wiki/Jaccard_index)
- [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance)
- Threshold values (95%, 60%) are based on practical testing and can be adjusted
