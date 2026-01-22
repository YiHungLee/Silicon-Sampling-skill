# CLAUDE.md - AI Assistant Development Guide

**Repository**: Silicon-Sampling-skill
**Purpose**: Claude Agent Skill for Silicon Sampling psychological research methodology
**Last Updated**: 2026-01-22
**Current Branch**: claude/add-claude-documentation-BiGwx

---

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Codebase Structure](#codebase-structure)
3. [Architecture & Key Concepts](#architecture--key-concepts)
4. [Development Workflows](#development-workflows)
5. [Critical Conventions](#critical-conventions)
6. [File Format Specifications](#file-format-specifications)
7. [Testing & Validation](#testing--validation)
8. [Common Tasks](#common-tasks)
9. [Troubleshooting](#troubleshooting)

---

## Repository Overview

### What is Silicon Sampling?

Silicon Sampling is a research methodology that uses AI models to simulate human respondents filling out psychological questionnaires. This repository implements a Claude Agent Skill that enables:

- **Pre-testing questionnaires** before formal data collection
- **Generating simulated data** for teaching or methodological research
- **Exploring AI psychological characteristics** on standardized scales
- **Supplementing samples** when specific demographic groups are difficult to recruit

### Key Features

- **Interactive workflow**: Automatically collects missing information (population, sample size, questionnaire)
- **Population-based persona generation**: Creates diverse, realistic personas based on demographic distributions
- **NEO-PI-R 30 facets model**: Advanced personality model with 30 sub-dimensions across Big Five domains
- **Psychologically coherent responses**: Simulates human response patterns based on personality and demographics
- **Structured CSV output**: Ready for statistical analysis in R, Python, SPSS, etc.

### Primary Language

- **Skill instructions**: Traditional Chinese (zh-TW)
- **Code/Documentation**: English and Traditional Chinese
- **Target users**: Taiwan-based psychological researchers

---

## Codebase Structure

```
Silicon-Sampling-skill/
├── CLAUDE.md                               # This file - AI assistant guide
├── SKILL.md                                # ⭐ Main skill definition (Anthropic format)
├── README.md                               # User-facing documentation
├── LICENSE                                 # MIT License
│
├── examples/                               # Example files for users
│   ├── population_tw_college.json          # Taiwan college student population
│   ├── population_general_adult.json       # Taiwan general adult population
│   ├── big_five_bfi10.json                 # BFI-10 personality questionnaire
│   ├── phq9.json                           # PHQ-9 depression screening
│   └── sample_output.csv                   # Example output format
│
└── references/                             # Taiwan population statistics
    ├── 114-12_Age_composition_by_city.csv  # Age/gender distribution (ROC 114/Dec)
    └── 113_education.csv                   # Education distribution (ROC 113)
```

### Critical Files

#### 1. SKILL.md (Main Skill Definition)

**Location**: `/SKILL.md`
**Format**: Anthropic Skill specification (YAML frontmatter + Markdown instructions)

**Structure**:
```yaml
---
name: psychology-survey
description: |
  Silicon Sampling skill for simulating human respondents...
---

# [Skill Instructions Follow]
```

**Purpose**:
- Defines the skill interface and behavior
- Contains complete workflow instructions
- Includes NEO-PI-R 30 facets reference
- Specifies critical rules (NO SCRIPTS, consistency requirements)

**When to modify**:
- Updating skill behavior or workflow
- Adding new personality models
- Changing output formats
- Modifying population handling

#### 2. README.md (User Documentation)

**Location**: `/README.md`
**Audience**: End users (researchers, students)
**Language**: Traditional Chinese

**When to modify**:
- User-facing feature changes
- Usage examples updates
- Adding new example populations or questionnaires

#### 3. examples/ (Reference Files)

**Purpose**: Provide users with ready-to-use templates

**Key files**:
- `population_tw_college.json`: Well-documented population definition schema
- `big_five_bfi10.json`: Complete questionnaire format example
- `sample_output.csv`: Expected output format

**When to add new examples**:
- New validated population definitions
- Commonly used psychological scales
- Different questionnaire formats (e.g., visual analog scales)

#### 4. references/ (Data Sources)

**Purpose**: Official Taiwan population statistics for default persona generation

**Data sources**:
- Ministry of the Interior, Department of Household Registration
- Directorate-General of Budget, Accounting and Statistics, Executive Yuan

**Usage**: When user doesn't specify a population, skill uses this data to generate realistic personas

---

## Architecture & Key Concepts

### 1. Skill Execution Model

The skill follows Anthropic's Skills specification:

```
User Request → Skill Activation → Interactive Workflow → Output Generation
```

**Workflow stages**:
1. **Information Collection** (interactive if missing):
   - Population definition (or use Taiwan default)
   - Sample size (recommended 1-100)
   - Questionnaire items and scale

2. **Persona Generation**:
   - Create N unique personas following population distributions
   - Assign NEO-PI-R 30 facets scores
   - Ensure demographic and personality diversity

3. **Response Simulation**:
   - For each persona, simulate responses to each questionnaire item
   - Apply personality facet influences
   - Maintain cross-item consistency

4. **Output Generation**:
   - `responses.csv`: Wide-format response data
   - `personas.csv`: Complete persona demographics + 30 facets
   - Summary statistics in YAML format

### 2. NEO-PI-R 30 Facets Model

**Core innovation**: This skill uses the comprehensive 30-facet model instead of simple Big Five scores.

**Structure**:
- **5 Domains**: Neuroticism (N), Extraversion (E), Openness (O), Agreeableness (A), Conscientiousness (C)
- **6 Facets per domain**: Total 30 sub-dimensions
- **Scoring**: Percentile rank 1-100 (50 = population mean)

**Example**:
```
【神經質 Neuroticism (N)】
N1: 焦慮 (Anxiety) - 45
N2: 憤怒敵意 (Angry Hostility) - 38
N3: 憂鬱 (Depression) - 42
N4: 自我意識 (Self-Consciousness) - 55
N5: 衝動 (Impulsiveness) - 52
N6: 脆弱 (Vulnerability) - 40
```

**Why this matters**:
- **Fine-grained response patterns**: N3 (Depression) directly influences PHQ-9 responses differently than N1 (Anxiety)
- **Psychological realism**: Someone high in E2 (Gregariousness) but low in E3 (Assertiveness) creates believable social patterns
- **Cross-scale consistency**: Same facets predict responses across multiple questionnaires

**Reference**: See SKILL.md lines 380-440 for complete facet definitions

### 3. Population Definition Schema

Populations define target demographic groups for sampling.

**Required attributes**:
```json
{
  "population": {
    "name": "台灣大學生",
    "age_range": {
      "min": 18,
      "max": 24,
      "mean": 20.5,
      "sd": 1.5,
      "distribution": "normal"
    },
    "gender_distribution": {
      "male": 0.48,
      "female": 0.50,
      "non_binary": 0.02
    },
    "education_distribution": { ... },
    "cultural_background": "台灣",
    "personality_tendencies": {
      "extraversion": { "mean": 55, "sd": 18 },
      // ... other Big Five domains
    }
  }
}
```

**Default population**: If user doesn't specify, use Taiwan general adult population (see SKILL.md lines 45-75)

### 4. Response Generation Logic

**Critical**: Responses MUST be generated through deliberate reasoning, NOT scripts or random number generators.

**Process for each item**:
1. **Identify relevant personality facets**:
   - Example: "I feel nervous" → primarily influenced by N1 (Anxiety), also N4 (Self-Consciousness)

2. **Consider demographics**:
   - Age: Older respondents may have different perspectives
   - Education: Affects comprehension and response style
   - Cultural background: Affects social desirability bias

3. **Apply response tendencies**:
   - Central tendency bias: Most responses near middle
   - Acquiescence: Some personas tend to agree more
   - Social desirability: Sensitive topics show bias

4. **Ensure consistency**:
   - Related items should correlate (e.g., multiple anxiety items)
   - Reverse-scored items should show opposite patterns
   - Cross-scale consistency (high N3 persona should score high on PHQ-9)

**Example reasoning**:
```
Persona R001: 21F, N1_anxiety=75, N3_depression=68, E6_positive_emotions=35

PHQ-9 Item 2: "Feeling down, depressed, or hopeless"
Scale: 0=Not at all, 1=Several days, 2=More than half, 3=Nearly every day

Reasoning:
- Very high depression facet (68) → strong influence toward higher scores
- Low positive emotions (35) → reinforces depressive symptoms
- Young adult (21) → may have less coping experience
- College student → academic stress may exacerbate
→ Response: 2 (More than half the days)
```

---

## Development Workflows

### Workflow 1: Modifying the Skill

**When**: Updating skill behavior, adding features, fixing bugs

**Steps**:
1. **Read SKILL.md** to understand current behavior
2. **Identify the section to modify**:
   - Workflow changes → Lines 23-33 (workflow section)
   - Personality model → Lines 150-212 (persona generation), 380-440 (facet reference)
   - Response logic → Lines 218-285 (response simulation)
   - Output format → Lines 289-329 (output CSV)
3. **Make targeted edits** using Edit tool
4. **Test the changes** (see Testing section)
5. **Update README.md** if user-facing behavior changed
6. **Commit with descriptive message**:
   ```bash
   git add SKILL.md README.md
   git commit -m "feat: Add support for 7-point Likert scales"
   git push -u origin claude/add-claude-documentation-BiGwx
   ```

### Workflow 2: Adding New Example Files

**When**: Providing users with new population definitions or questionnaires

**Steps**:
1. **Determine file type**:
   - Population → `examples/population_*.json`
   - Questionnaire → `examples/questionnaire_*.json`
   - Output example → `examples/sample_output_*.csv`

2. **Follow existing schema**:
   - For populations: See `examples/population_tw_college.json`
   - For questionnaires: See `examples/big_five_bfi10.json`

3. **Validate JSON structure**:
   ```bash
   python -m json.tool examples/new_file.json
   ```

4. **Add reference in README.md**:
   ```markdown
   ## 範例量表

   3. **新量表名稱**
      - 描述
      - 用途
   ```

5. **Commit**:
   ```bash
   git add examples/new_file.json README.md
   git commit -m "docs: Add example for [description]"
   git push -u origin claude/add-claude-documentation-BiGwx
   ```

### Workflow 3: Updating Population Statistics

**When**: New official Taiwan statistics are released

**Steps**:
1. **Obtain official data**:
   - Age/gender: https://www.ris.gov.tw/app/portal/346 (內政部戶政司)
   - Education: https://www.stat.gov.tw/ (主計總處)

2. **Update CSV files** in `references/`:
   ```bash
   # Keep naming convention: [year]-[month]_description.csv
   cp new_data.csv references/115-06_Age_composition_by_city.csv
   ```

3. **Update SKILL.md** default population (lines 45-75):
   - Gender distribution
   - Age distribution
   - Education distribution
   - Data source citations

4. **Update README.md** (lines 203-212)

5. **Commit**:
   ```bash
   git add references/ SKILL.md README.md
   git commit -m "data: Update Taiwan population statistics to ROC 115/June"
   git push -u origin claude/add-claude-documentation-BiGwx
   ```

### Workflow 4: Git Operations

**Branch naming convention**: `claude/[description]-[session-id]`

**Current branch**: `claude/add-claude-documentation-BiGwx`

**Critical git rules**:
- ✅ Always push to branch: `git push -u origin claude/add-claude-documentation-BiGwx`
- ❌ Never push to main/master directly
- ⚠️ Branch must start with `claude/` and end with matching session ID
- 🔄 Retry network failures up to 4 times with exponential backoff (2s, 4s, 8s, 16s)

**Common operations**:
```bash
# Check status
git status

# Stage and commit changes
git add SKILL.md README.md
git commit -m "feat: Add new feature description"

# Push to feature branch
git push -u origin claude/add-claude-documentation-BiGwx

# View recent commits
git log --oneline -10

# View diff
git diff HEAD~1
```

---

## Critical Conventions

### 🚫 Rule 1: NO SCRIPTS FOR PERSONA GENERATION

**NEVER** use code, scripts, or programmatic methods to generate personas.

**Why**: Scripts produce artificial patterns lacking psychological coherence. Real personas have meaningful trait combinations (e.g., high anxiety often correlates with high depression, not random independent values).

**Correct approach**:
```
1. Consider population characteristics
2. Sample demographics from distributions
3. Generate personality facets with realistic correlations:
   - N facets often correlate positively with each other
   - E1 (Warmth) and A3 (Altruism) often correlate
   - C2 (Order) and C5 (Self-Discipline) often correlate
4. Create coherent background narrative
5. Think through how this person would respond to surveys
```

**Reference**: SKILL.md lines 144-148, 335-342

### 🚫 Rule 2: NO SCRIPTS FOR RESPONSE SIMULATION

**NEVER** use random number generators or scripts to fill in responses.

**Why**: Random generation creates statistically inconsistent data. Real human responses show:
- **Within-scale consistency**: Related items correlate (all anxiety items should be similar)
- **Between-scale consistency**: Personality predicts symptom scales (high N3 → high PHQ-9)
- **Response patterns**: Central tendency, acquiescence, social desirability

**Correct approach**:
```
For each item:
1. Identify relevant personality facets (e.g., "I feel sad" → N3 Depression)
2. Consider facet score (N3=75 → high depression → endorses sad items)
3. Apply demographics (age, culture affect interpretation)
4. Apply response tendencies (central tendency, social desirability)
5. Check consistency with previous responses
→ Deliberate response: 4 (Agree)
```

**Reference**: SKILL.md lines 218-223, 335-346

### ✅ Rule 3: ENSURE CROSS-ITEM CONSISTENCY

**Critical**: Each persona must show coherent response patterns.

**Within-scale consistency**:
```
PHQ-9 has multiple depression items:
- "Little interest or pleasure"
- "Feeling down, depressed"
- "Trouble sleeping"

A highly depressed persona (N3=75) should endorse ALL of these,
not randomly scatter responses.
```

**Cross-scale consistency** (SKILL.md lines 268-285):

| Personality Facet | Affected Scales | Direction |
|-------------------|-----------------|-----------|
| N1 (Anxiety) | GAD-7, STAI | Positive |
| N3 (Depression) | PHQ-9, BDI-II | Positive |
| E6 (Positive Emotions) | PHQ-9, SWLS | Negative/Positive |
| E2 (Gregariousness) | Loneliness scales | Negative |
| C5 (Self-Discipline) | Procrastination scales | Negative |

**Example**:
```
Persona with N1=80, N3=75, E6=25 should show:
- High scores on anxiety scales (GAD-7)
- High scores on depression scales (PHQ-9)
- Low scores on positive affect scales
- Consistent pattern across all mental health measures
```

### ✅ Rule 4: FOLLOW POPULATION DISTRIBUTIONS

When generating personas, strictly follow population distributions:

**Gender distribution**:
```json
"gender_distribution": {
  "male": 0.48,
  "female": 0.50,
  "non_binary": 0.02
}
```
→ Out of 100 personas: ~48 male, ~50 female, ~2 non-binary

**Age distribution**:
```json
"age_range": {
  "min": 18,
  "max": 24,
  "mean": 20.5,
  "sd": 1.5,
  "distribution": "normal"
}
```
→ Most personas around 20-21, fewer at extremes

**Personality tendencies**:
```json
"extraversion": {
  "mean": 55,
  "sd": 18
}
```
→ Domain mean is 55, facet scores should average to ~55 with reasonable SD

### ✅ Rule 5: MAINTAIN PSYCHOLOGICAL REALISM

**Facet correlations**: Personality facets aren't independent

**Typical correlations**:
- N1 (Anxiety) ↔ N3 (Depression): r ≈ 0.6
- E1 (Warmth) ↔ A3 (Altruism): r ≈ 0.5
- C2 (Order) ↔ C5 (Self-Discipline): r ≈ 0.4

**Anti-correlations**:
- N (Neuroticism) ↔ E6 (Positive Emotions): r ≈ -0.5
- A4 (Compliance) ↔ E3 (Assertiveness): r ≈ -0.3

**Example realistic persona**:
```
✅ GOOD:
N1_anxiety: 75, N3_depression: 70, N6_vulnerability: 72, E6_positive_emotions: 30
(Coherent high-neuroticism profile)

❌ BAD:
N1_anxiety: 75, N3_depression: 25, N6_vulnerability: 80, E6_positive_emotions: 90
(Incoherent - high anxiety but low depression and very high positive emotions)
```

### ✅ Rule 6: INTERACTIVE WORKFLOW

**Always follow the 6-step workflow** (SKILL.md lines 23-33):

```
Step 1: Collect Population Definition
  ↓ If missing → Ask user or use Taiwan default
Step 2: Collect Sample Size
  ↓ If missing → Ask user
Step 3: Collect Questionnaire Items
  ↓ If missing → Ask user
Step 4: Generate Personas
  ↓ Deliberate reasoning based on population
Step 5: Simulate Responses
  ↓ Deliberate reasoning based on personas
Step 6: Output CSV Results
```

**Never skip steps**. If information is missing, ask the user.

**Example**:
```
User: "請幫我填 PHQ-9"

❌ BAD: Immediately start generating without knowing sample size
✅ GOOD: Ask "請問您需要生成多少個模擬樣本？"
```

---

## File Format Specifications

### Population Definition Format

**File**: `examples/population_*.json`

**Schema**:
```json
{
  "population": {
    "name": "string",
    "description": "string (optional)",

    "age_range": {
      "min": number,
      "max": number,
      "mean": number,
      "sd": number,
      "distribution": "normal" | "uniform"
    },

    "gender_distribution": {
      "male": number (0-1),
      "female": number (0-1),
      "non_binary": number (0-1)
      // Must sum to 1.0
    },

    "education_distribution": {
      "[level]": number (0-1),
      // Must sum to 1.0
    },

    "cultural_background": "string",

    "personality_tendencies": {
      "extraversion": { "mean": number (1-100), "sd": number },
      "agreeableness": { "mean": number (1-100), "sd": number },
      "conscientiousness": { "mean": number (1-100), "sd": number },
      "neuroticism": { "mean": number (1-100), "sd": number },
      "openness": { "mean": number (1-100), "sd": number }
    },

    "additional_characteristics": {
      // Optional: any relevant attributes
    }
  }
}
```

**Validation**:
- All distribution probabilities must sum to 1.0
- Means and SDs should be realistic (SD typically 15-20 for personality)
- Age range should be reasonable for population

### Questionnaire Format

**File**: `examples/questionnaire_*.json`

**Schema**:
```json
{
  "questionnaire": {
    "name": "string",
    "description": "string",
    "language": "zh-TW" | "en-US" | ...,
    "citation": "string (optional)",

    "scale": {
      "type": "likert" | "frequency" | "visual_analog",
      "min": number,
      "max": number,
      "labels": {
        "1": "string",
        "2": "string",
        ...
      }
    },

    "instruction": "string",

    "items": [
      {
        "item_id": "string",
        "text": "string",
        "dimension": "string (optional)",
        "reverse_scored": boolean
      },
      ...
    ],

    "scoring": {
      "[dimension]": ["item_id1", "item_id2", ...],
      ...
    }
  }
}
```

### Output Format 1: responses.csv

**Format**: Wide format, one row per persona

```csv
respondent_id,ITEM1,ITEM2,ITEM3,...
R001,2,4,5,...
R002,4,3,3,...
R003,3,4,4,...
```

**Specifications**:
- `respondent_id`: R001, R002, ... R{N}
- Item columns: Named by `item_id` from questionnaire
- Values: Raw responses (before reverse-scoring)

### Output Format 2: personas.csv

**Format**: One row per persona with demographics + 30 facets

```csv
respondent_id,age,gender,education,occupation,background,N1_anxiety,N2_hostility,N3_depression,N4_self_consciousness,N5_impulsiveness,N6_vulnerability,E1_warmth,E2_gregariousness,E3_assertiveness,E4_activity,E5_excitement_seeking,E6_positive_emotions,O1_fantasy,O2_aesthetics,O3_feelings,O4_actions,O5_ideas,O6_values,A1_trust,A2_straightforwardness,A3_altruism,A4_compliance,A5_modesty,A6_tender_mindedness,C1_competence,C2_order,C3_dutifulness,C4_achievement_striving,C5_self_discipline,C6_deliberation
R001,21,female,bachelor_3,student_parttime,"台北，心理系",45,38,42,55,52,40,68,72,58,65,60,70,75,78,72,68,70,65,62,70,75,58,55,68,60,52,58,62,48,55
```

**Columns**:
- Demographics: `respondent_id`, `age`, `gender`, `education`, `occupation`, `background`
- Facets: All 30 facets as `[DOMAIN][N]_[name]` (e.g., `N1_anxiety`, `E2_gregariousness`)

### Output Format 3: Summary

**Format**: YAML

```yaml
summary:
  population: "string"
  sample_size: number
  questionnaire: "string"

  demographics:
    age_mean: number
    age_sd: number
    gender_ratio: "string"

  response_statistics:
    mean_response_time_ms: number
    completion_rate: number

  notes:
    - "string"
    - "string"
```

---

## Testing & Validation

### Test Cases for Skill Modifications

When modifying SKILL.md, test these scenarios:

#### Test 1: Interactive Workflow
```
User: "請幫我填問卷"
Expected: Ask for population → sample size → questionnaire
```

#### Test 2: Partial Information
```
User: "請幫我填 PHQ-9，30 個樣本"
Expected: Ask for population or offer Taiwan default
```

#### Test 3: Complete Information
```
User: "台灣大學生，50 個樣本，PHQ-9"
Expected: Generate immediately without asking
```

#### Test 4: Default Population
```
User: "使用預設母群，100 個樣本，BFI-10"
Expected: Use Taiwan adult population from references/
```

### Validation Checklist

When generating output, verify:

**Demographics**:
- [ ] Gender distribution matches population (±5%)
- [ ] Age distribution follows specified distribution
- [ ] Education distribution matches population (±5%)

**Personality**:
- [ ] Mean facet scores near population mean (±10)
- [ ] Facet SD reasonable (not all identical or wildly variable)
- [ ] Realistic facet correlations (N facets correlate, etc.)

**Responses**:
- [ ] No impossible values (e.g., 6 on 5-point scale)
- [ ] Internal consistency for related items
- [ ] High-neuroticism personas show high symptom scores
- [ ] Low-neuroticism personas show low symptom scores
- [ ] Responses show central tendency (not all extremes)

**Output files**:
- [ ] responses.csv has correct number of rows and columns
- [ ] personas.csv has all 36+ columns (demographics + 30 facets)
- [ ] Summary statistics match data

### Statistical Validation (For Advanced Users)

Researchers can validate output quality:

```r
# Load data
responses <- read.csv("responses.csv")
personas <- read.csv("personas.csv")

# Check internal consistency (Cronbach's alpha)
library(psych)
alpha(responses[, c("PHQ9_1", "PHQ9_2", "PHQ9_3", ...)])
# Expected: α > 0.80 for most scales

# Check personality-symptom correlations
cor.test(personas$N3_depression, rowMeans(responses))
# Expected: r > 0.5 for depression scales

# Check demographic distributions
table(personas$gender)
# Should match population distribution
```

---

## Common Tasks

### Task 1: Add a New Personality Facet Influence

**Scenario**: You want to document how O5 (Ideas) influences creativity scales.

**Steps**:
1. Read SKILL.md lines 227-251 (Personality Facet Influence section)
2. Add new entry:
```markdown
**開放性 (Openness) 構面影響：**
- 高想像/理念 (O1/O5) → 創造力量表得分較高
- 高理念 (O5) → 智力好奇量表、科學興趣量表得分較高  # NEW
- 高審美/感受 (O2/O3) → 藝術興趣、情感敏感度題目得分較高
```
3. Also add to cross-scale consistency table (lines 268-285):
```markdown
| O5 (Ideas) | Creativity scales, NFC | Positive |  # NEW
```
4. Test with a creativity questionnaire
5. Commit: `git commit -m "docs: Add O5 influence on creativity scales"`

### Task 2: Update Default Taiwan Population

**Scenario**: New population data released for ROC 115.

**Steps**:
1. Download data from official sources
2. Update `references/115-xx_*.csv`
3. Calculate new distributions
4. Edit SKILL.md lines 49-70:
```markdown
### Gender Distribution (性別分布)
| 性別 | 比例 |
|------|------|
| 男 | 49.1% |  # UPDATED
| 女 | 50.9% |  # UPDATED
```
5. Update data source line 72-74:
```markdown
- 性別與年齡: 內政部戶政司 民國115年6月人口統計  # UPDATED
```
6. Also update README.md lines 207-211
7. Commit: `git commit -m "data: Update to ROC 115 population statistics"`

### Task 3: Add Support for New Scale Type

**Scenario**: Add visual analog scale (VAS) support.

**Steps**:
1. Read SKILL.md lines 442-466 (Likert Scale Reference)
2. Add new section:
```markdown
**Visual Analog Scale (VAS):**
- Range: 0-100 continuous
- Labels: Text anchors at endpoints (e.g., "No pain" to "Worst pain")
- Response: Integer 0-100
```
3. Update response simulation section (lines 218-266) to handle VAS logic:
```markdown
### Visual Analog Scale Responses
- Consider the full range 0-100
- Most responses will cluster in meaningful ranges (not uniform)
- Apply same personality influences but map to 0-100 scale
```
4. Add example questionnaire: `examples/vas_pain.json`
5. Update README.md to mention VAS support
6. Commit: `git commit -m "feat: Add visual analog scale (VAS) support"`

### Task 4: Fix Inconsistent Response Generation

**Scenario**: Skill is generating inconsistent responses across related items.

**Diagnosis**:
1. Check if skill is following deliberate reasoning (not using scripts)
2. Check recent commits: `git log --oneline -10`
3. Look for lines that might have broken consistency rules

**Fix**:
1. Review SKILL.md lines 218-285 (Response simulation + consistency)
2. Strengthen language about consistency:
```markdown
### Response Consistency (CRITICAL)

Before finalizing a persona's responses, verify:
1. Related items show correlation (r > 0.6 for similar items)
2. Reverse-scored items show opposite patterns
3. High facet scores → high item endorsement consistently
4. No contradictory patterns (high anxiety but low stress)
```
3. Add explicit check in workflow:
```markdown
Step 5: Simulate Responses
  ...
  After generating all responses for a persona:
  → Review for consistency
  → Adjust any contradictory responses
  → Ensure personality-symptom alignment
```
4. Test with multiple personas on multiple questionnaires
5. Commit: `git commit -m "fix: Strengthen response consistency requirements"`

---

## Troubleshooting

### Issue 1: Personas Lack Diversity

**Symptoms**:
- All personas have very similar personality profiles
- Demographics don't match population distribution

**Causes**:
- Not sampling from distributions properly
- Using simplistic "average person" approach

**Solution**:
1. Emphasize in SKILL.md persona generation section (lines 150-212):
```markdown
3. **Individual Variation**: Each persona MUST be unique
   - Sample age from distribution (not just mean)
   - Vary all facets (use population SD as guide)
   - Create diverse backgrounds within population constraints

Example:
Population mean N1=50, SD=18
→ Persona 1: N1=35 (below average)
→ Persona 2: N1=72 (above average)
→ Persona 3: N1=51 (near average)
NOT all personas with N1=50
```

### Issue 2: Responses Look Too Random

**Symptoms**:
- Related items don't correlate
- High-anxiety personas don't consistently endorse anxiety items
- Responses lack psychological coherence

**Causes**:
- Using scripts or random generation (violates Rule 1 & 2)
- Not considering personality facets when responding

**Solution**:
1. Review critical rules in SKILL.md (lines 335-354)
2. Add explicit worked example in response simulation section:
```markdown
### Worked Example: Deliberate Response Reasoning

Persona R001: N1_anxiety=75 (high), N3_depression=42 (average)

GAD-7 Item 1: "Feeling nervous, anxious, or on edge"
Scale: 0-3 (Not at all → Nearly every day)

Step 1: Identify facets → N1 (Anxiety) - PRIMARY
Step 2: Check facet score → 75 (high, 75th percentile)
Step 3: Apply demographics → 21-year-old college student (high stress)
Step 4: Determine response → 2 or 3 likely, choose 2 (More than half the days)

Result: 2 ✅

Do this for EVERY item.
```

### Issue 3: Output Files Have Formatting Errors

**Symptoms**:
- CSV has misaligned columns
- Missing facet columns in personas.csv
- Respondent IDs inconsistent

**Causes**:
- Not following exact column order
- Missing facets
- Typos in facet names

**Solution**:
1. Create template row in SKILL.md (around line 302):
```csv
# Template row - ALWAYS use this exact order:
respondent_id,age,gender,education,occupation,background,N1_anxiety,N2_hostility,N3_depression,N4_self_consciousness,N5_impulsiveness,N6_vulnerability,E1_warmth,E2_gregariousness,E3_assertiveness,E4_activity,E5_excitement_seeking,E6_positive_emotions,O1_fantasy,O2_aesthetics,O3_feelings,O4_actions,O5_ideas,O6_values,A1_trust,A2_straightforwardness,A3_altruism,A4_compliance,A5_modesty,A6_tender_mindedness,C1_competence,C2_order,C3_dutifulness,C4_achievement_striving,C5_self_discipline,C6_deliberation
```
2. Add validation checklist:
```markdown
Before outputting personas.csv, verify:
- [ ] Exactly 36 columns (6 demo + 30 facets)
- [ ] All facets present: N1-N6, E1-E6, O1-O6, A1-A6, C1-C6
- [ ] Respondent IDs: R001, R002, ... R{sample_size}
- [ ] No missing values
```

### Issue 4: Large Sample Sizes Cause Problems

**Symptoms**:
- Skill struggles with 100+ samples
- Output becomes inconsistent for later personas
- Process seems overwhelming

**Causes**:
- Not using batch processing approach
- Trying to keep all personas in working memory

**Solution**:
1. Review SKILL.md lines 358-377 (Large Sample Handling)
2. Emphasize batch approach:
```markdown
For sample_size > 50:

1. Ask user: "建議分批處理，每批 25 個。您希望如何處理？"
2. Generate in batches:
   - Batch 1: R001-R025
   - Batch 2: R026-R050
   - Batch 3: R051-R075
   - Batch 4: R076-R100
3. Maintain population distributions across batches
4. Output cumulative files after each batch
```

### Issue 5: Cross-Scale Consistency Problems

**Symptoms**:
- High-depression persona scores low on PHQ-9
- High-anxiety persona scores low on GAD-7
- Personality doesn't predict symptoms

**Causes**:
- Not checking cross-scale consistency table
- Treating each questionnaire independently

**Solution**:
1. Always consult SKILL.md lines 268-285 (Cross-Scale Consistency)
2. Before generating responses, identify relationships:
```markdown
Step 5: Simulate Responses (Enhanced)

For each persona:
1. Review personality profile (all 30 facets)
2. For each questionnaire:
   a. Identify which facets should influence responses
   b. Check cross-scale consistency table
   c. Generate responses aligned with facets
3. After ALL questionnaires:
   → Verify personality-symptom alignment
   → Check for contradictions
   → Adjust if needed
```

---

## Recent Changes & Migration Notes

### Upgrade to NEO-PI-R 30 Facets (2025-01-22)

**Commit**: `356d686 feat: 升級人格模型至 NEO-PI-R 30 構面並優化輸出格式`

**Changes**:
- **OLD**: Simple Big Five scores (5 values)
- **NEW**: NEO-PI-R 30 facets (30 values: 6 per domain)

**Migration for existing skills/scripts**:
- Persona generation now includes all 30 facets
- personas.csv has 30 additional columns
- Response simulation uses fine-grained facets instead of domain scores

**Benefits**:
- More realistic personality profiles
- Better response prediction (e.g., N1 vs N3 have different effects)
- Richer data for researchers

### Script Prohibition (2025-01-22)

**Commit**: `2dd14bb 增加腳本禁用`

**Changes**:
- Added explicit rules against using scripts or code for persona/response generation
- Emphasized deliberate reasoning requirement

**Reason**: Scripts produce statistically invalid patterns lacking psychological coherence

### Population-Based Workflow (Earlier)

**Commit**: `bb911ad refactor: Update skill to use population-based workflow`

**Changes**:
- **OLD**: Single persona per request
- **NEW**: Population-based sampling with multiple personas

**Benefits**:
- Simulates real research sampling
- Generates diverse, representative samples
- Supports demographic and personality distribution specifications

---

## Additional Resources

### External Documentation

- **Anthropic Skills Specification**: https://docs.anthropic.com/claude/docs/skills
- **Claude API Docs**: https://docs.anthropic.com/
- **NEO-PI-R Manual**: Costa, P. T., & McCrae, R. R. (1992). Revised NEO Personality Inventory (NEO-PI-R)

### Research References

- **Silicon Sampling**: Argyle, L. P., et al. (2023). Out of One, Many: Using Language Models to Simulate Human Samples. *Political Analysis*.
- **BFI-10**: Rammstedt, B., & John, O. P. (2007). Measuring personality in one minute or less. *Journal of Research in Personality*, 41(1), 203-212.
- **PHQ-9**: Kroenke, K., Spitzer, R. L., & Williams, J. B. (2001). The PHQ-9. *Journal of General Internal Medicine*, 16(9), 606-613.

### Taiwan Data Sources

- **人口統計**: https://www.ris.gov.tw/ (內政部戶政司)
- **教育統計**: https://www.stat.gov.tw/ (行政院主計總處)

---

## Contributing Guidelines

When making changes to this repository:

1. **Read first**: Understand SKILL.md and this CLAUDE.md before modifying
2. **Test thoroughly**: Verify changes with multiple test cases
3. **Update documentation**: Keep README.md and CLAUDE.md in sync
4. **Commit clearly**: Use conventional commit messages (feat/fix/docs/data)
5. **Push to feature branch**: Never push directly to main

**Questions or Issues**: Open an issue on GitHub or consult the SKILL.md for detailed specifications.

---

**Document Version**: 1.0
**Last Updated**: 2026-01-22
**Maintained By**: Repository contributors
**For**: Claude AI assistants working with Silicon-Sampling-skill

