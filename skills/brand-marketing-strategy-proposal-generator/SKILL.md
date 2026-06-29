---
name: brand-marketing-strategy-proposal-generator
description: Generate comprehensive brand marketing strategy proposals with automated research, insights, and PPT script generation
triggers:
  - generate a brand marketing strategy proposal
  - create a marketing strategy document for a client
  - build a brand positioning deck
  - produce a marketing proposal with research and insights
  - generate PPT scripts for brand strategy presentation
  - create a comprehensive marketing strategy report
  - develop a brand marketing proposal workflow
  - automate brand strategy document generation
---

# Brand Marketing Strategy Proposal Generator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An agent skill for generating comprehensive brand marketing strategy proposals. This tool automates the entire workflow from client briefing through research analysis, opportunity insights, content structuring, PPT script generation, and final Word document output.

## What It Does

This skill orchestrates a multi-stage process to create professional brand marketing strategy proposals:

1. **Client Brief Understanding** — Parse client materials and define project scope
2. **Research & Analysis** — Conduct market, competitor, and brand analysis
3. **Opportunity Identification** — Generate strategic insights and recommendations
4. **Content Structuring** — Build client-facing outline and internal analysis framework
5. **PPT Script Generation** — Create slide-by-slide content scripts
6. **Quality Assurance** — Review and validate generated content
7. **Document Export** — Output final Word document

## Installation

Clone this repository into your project workspace:

```bash
git clone https://github.com/yuleiwang156-a11y/brand-marketing-strategy-proposal-skill.git
cd brand-marketing-strategy-proposal-skill
```

Set up the directory structure:

```bash
mkdir -p inputs/客户资料
mkdir -p outputs
mkdir -p agent_memory
mkdir -p backups
```

Add to `.gitignore`:

```
inputs/客户资料/
outputs/
agent_memory/
backups/
MEMORY.md
*.docx
*.pptx
```

## Project Structure

```
brand-marketing-strategy-proposal-skill/
├── inputs/
│   └── 客户资料/
│       └── [客户名称]/
│           ├── brief.pdf
│           ├── brand_assets.zip
│           └── market_data.xlsx
├── outputs/
│   └── [客户名称]_strategy_proposal.docx
├── agent_memory/
│   ├── research_findings.md
│   ├── insights.md
│   └── outline.md
├── backups/
├── MEMORY.md
└── README.md
```

## Core Workflow

### Step 1: Prepare Client Materials

Place all client materials in the designated folder:

```bash
mkdir -p "inputs/客户资料/AcmeBrand"
# Add client files: brief, brand guidelines, market research, etc.
```

### Step 2: Initialize the Workflow

Trigger the skill with:

```
请按照本项目 Skill 执行。本次客户资料在 inputs/客户资料/AcmeBrand/ 中，请从确认点 1 开始。
```

### Step 3: Checkpoint 1 — Needs & Problem Definition

The agent will:
- Read all client materials
- Summarize business context
- Define core marketing challenges
- Present findings for confirmation

Respond with confirmation or request revisions:

```
确认，请继续到研究分析层
```

### Step 4: Research & Analysis Layer

The agent conducts:
- **Market Analysis** — Industry trends, size, growth, dynamics
- **Competitor Analysis** — Key players, positioning, strategies
- **Consumer Analysis** — Target segments, behaviors, pain points
- **Brand Analysis** — Current positioning, equity, perception

### Step 5: Checkpoint 2 — Opportunity Insights

The agent presents:
- Strategic opportunities identified
- Recommended positioning angles
- Differentiation strategies
- Growth levers

Confirm or iterate:

```
洞察2和3需要深化，请补充消费者痛点数据支持
```

### Step 6: Checkpoint 3 — Outline Approval

Review the proposed structure:
- Client-facing table of contents
- Internal analysis framework
- Slide count and flow

Approve or modify:

```
目录结构确认。请开始生成 PPT 内容脚本。
```

### Step 7: PPT Script Generation

The agent generates page-by-page scripts with:
- Slide title
- Key messages
- Supporting data/visuals
- Speaker notes

### Step 8: Quality Check & Export

Final review and Word document generation:

```
质量检查通过，请生成 Word 文档并保存到 outputs/
```

## Configuration

### Environment Variables

Set configuration via environment variables:

```bash
# Language model settings
export PROPOSAL_MODEL="gpt-4"
export PROPOSAL_TEMPERATURE="0.7"

# Output preferences
export PROPOSAL_OUTPUT_FORMAT="docx"
export PROPOSAL_LANGUAGE="zh-CN"

# Analysis depth
export RESEARCH_DEPTH="comprehensive"  # or "standard", "brief"
```

### Memory Management

The skill uses `MEMORY.md` to track workflow state:

```markdown
# Current Project: AcmeBrand

## Status
- Checkpoint: 2 (Opportunity Insights)
- Last Updated: 2026-06-15

## Key Decisions
- Target audience: Premium urban millennials
- Core positioning: Sustainable luxury
- Primary channel: Digital-first

## Pending Actions
- [ ] Validate consumer pain points with additional data
- [ ] Deep-dive on competitor X's recent campaign
```

## Code Examples

### Reading Client Brief

```python
import os
from pathlib import Path

def load_client_materials(client_name):
    """Load all client materials from inputs directory"""
    client_dir = Path(f"inputs/客户资料/{client_name}")
    
    materials = {
        "brief": [],
        "data": [],
        "assets": []
    }
    
    for file in client_dir.glob("**/*"):
        if file.suffix in [".pdf", ".docx"]:
            materials["brief"].append(file)
        elif file.suffix in [".xlsx", ".csv", ".json"]:
            materials["data"].append(file)
        elif file.suffix in [".zip", ".png", ".jpg"]:
            materials["assets"].append(file)
    
    return materials

# Usage
client_files = load_client_materials("AcmeBrand")
print(f"Found {len(client_files['brief'])} brief documents")
```

### Generating Research Analysis

```python
def generate_market_analysis(client_data):
    """Generate market analysis section"""
    
    analysis = {
        "market_size": extract_market_size(client_data),
        "growth_rate": calculate_cagr(client_data),
        "key_trends": identify_trends(client_data),
        "competitive_landscape": map_competitors(client_data)
    }
    
    # Structure findings
    report = f"""
# Market Analysis

## Market Size & Growth
- Current market size: {analysis['market_size']}
- CAGR (5yr): {analysis['growth_rate']}%

## Key Trends
{format_trends(analysis['key_trends'])}

## Competitive Landscape
{format_competitors(analysis['competitive_landscape'])}
"""
    
    return report
```

### Building Content Outline

```python
def create_proposal_outline(research, insights):
    """Generate proposal table of contents"""
    
    outline = {
        "sections": [
            {
                "title": "Executive Summary",
                "slides": 3,
                "content": ["Problem statement", "Solution overview", "Expected impact"]
            },
            {
                "title": "Market Context",
                "slides": 5,
                "content": research["market_analysis"]
            },
            {
                "title": "Strategic Opportunities",
                "slides": 4,
                "content": insights["opportunities"]
            },
            {
                "title": "Recommended Strategy",
                "slides": 8,
                "content": ["Positioning", "Messaging", "Channel strategy", "Activation plan"]
            },
            {
                "title": "Implementation Roadmap",
                "slides": 3,
                "content": ["Phases", "Timeline", "KPIs"]
            }
        ]
    }
    
    return outline
```

### Generating PPT Scripts

```python
def generate_slide_script(slide_number, section, content):
    """Generate detailed script for a single slide"""
    
    script = {
        "slide_number": slide_number,
        "section": section,
        "title": content["title"],
        "key_message": content["message"],
        "supporting_points": content["points"],
        "visual_suggestion": suggest_visual(content),
        "speaker_notes": generate_notes(content)
    }
    
    return format_script(script)

def format_script(script):
    """Format script as markdown"""
    return f"""
## Slide {script['slide_number']}: {script['title']}

**Key Message:** {script['key_message']}

**Supporting Points:**
{chr(10).join(f"- {point}" for point in script['supporting_points'])}

**Visual:** {script['visual_suggestion']}

**Speaker Notes:**
{script['speaker_notes']}
---
"""
```

### Exporting to Word

```python
from docx import Document
from docx.shared import Inches, Pt

def export_to_word(proposal_data, output_path):
    """Export proposal to Word document"""
    
    doc = Document()
    
    # Add title page
    doc.add_heading(proposal_data["title"], 0)
    doc.add_paragraph(f"Client: {proposal_data['client']}")
    doc.add_paragraph(f"Date: {proposal_data['date']}")
    doc.add_page_break()
    
    # Add table of contents
    doc.add_heading("Table of Contents", 1)
    for section in proposal_data["outline"]:
        doc.add_paragraph(
            f"{section['number']}. {section['title']}", 
            style='List Number'
        )
    doc.add_page_break()
    
    # Add content sections
    for section in proposal_data["sections"]:
        doc.add_heading(section["title"], 1)
        
        for slide in section["slides"]:
            doc.add_heading(slide["title"], 2)
            doc.add_paragraph(slide["content"])
            
            if slide.get("visual"):
                doc.add_paragraph(f"[Visual: {slide['visual']}]")
    
    # Save
    doc.save(output_path)
    print(f"Proposal exported to {output_path}")

# Usage
export_to_word(
    proposal_data,
    "outputs/AcmeBrand_strategy_proposal.docx"
)
```

## Common Patterns

### Pattern 1: Iterative Refinement

```
# Initial generation
"请生成市场分析部分"

# Review and refine
"市场趋势部分需要补充数字化转型的影响，请更新"

# Confirm
"市场分析确认，请继续"
```

### Pattern 2: Checkpoint Management

```python
checkpoints = {
    1: "needs_and_problem_definition",
    2: "opportunity_insights",
    3: "outline_approval"
}

def validate_checkpoint(checkpoint_id, user_feedback):
    """Validate checkpoint before proceeding"""
    if user_feedback.contains("确认"):
        return move_to_next_stage()
    else:
        return iterate_current_stage(user_feedback)
```

### Pattern 3: Multi-Client Management

```bash
# Process multiple clients in parallel
for client in "ClientA" "ClientB" "ClientC"; do
    echo "Processing $client..."
    # Trigger workflow for each client
done
```

## Troubleshooting

### Issue: Client materials not found

**Problem:** Agent cannot locate input files

**Solution:**
```bash
# Verify directory structure
ls -la "inputs/客户资料/[客户名称]/"

# Check file permissions
chmod -R 755 inputs/
```

### Issue: Incomplete research analysis

**Problem:** Market analysis lacks depth

**Solution:**
```
请补充以下内容到市场分析：
1. 过去3年的市场增长数据
2. 前5名竞争对手的市场份额
3. 主要消费者细分及规模
```

### Issue: Outline structure too generic

**Problem:** Table of contents doesn't match client needs

**Solution:**
```
目录需要调整：
- 将"市场背景"拆分为"行业趋势"和"消费者洞察"两个独立章节
- 增加"竞争对手分析"章节
- 将实施路线图扩展到5页
```

### Issue: Memory state corruption

**Problem:** Workflow state lost or inconsistent

**Solution:**
```bash
# Backup current state
cp MEMORY.md backups/MEMORY_$(date +%Y%m%d_%H%M%S).md

# Review and manually fix MEMORY.md
# Or restore from backup
cp backups/MEMORY_20260615_143022.md MEMORY.md
```

### Issue: Word export formatting

**Problem:** Generated document has formatting issues

**Solution:**
```python
# Use explicit styling
from docx.enum.style import WD_STYLE_TYPE

def apply_consistent_formatting(doc):
    # Set default font
    style = doc.styles['Normal']
    style.font.name = 'Arial'
    style.font.size = Pt(11)
    
    # Configure heading styles
    for level in range(1, 4):
        heading = doc.styles[f'Heading {level}']
        heading.font.name = 'Arial'
        heading.font.bold = True
```

## Best Practices

1. **Organize client materials systematically** — Use consistent folder structure and naming conventions
2. **Confirm at each checkpoint** — Don't skip validation steps
3. **Keep MEMORY.md updated** — Essential for resuming interrupted workflows
4. **Back up intermediate outputs** — Save research and insights to agent_memory/
5. **Use descriptive client names** — Avoid spaces and special characters in folder names
6. **Version control outputs** — Keep dated backups of generated proposals
7. **Validate data sources** — Ensure research data is current and credible
8. **Review scripts before export** — Quality check PPT content before Word generation

## Advanced Usage

### Custom Analysis Templates

Create reusable analysis templates:

```python
ANALYSIS_TEMPLATES = {
    "tech_brand": {
        "sections": ["Innovation landscape", "Digital ecosystem", "Tech adoption curves"],
        "depth": "technical"
    },
    "consumer_brand": {
        "sections": ["Consumer journey", "Emotional drivers", "Purchase barriers"],
        "depth": "behavioral"
    }
}

def apply_template(client_category):
    template = ANALYSIS_TEMPLATES.get(client_category, "default")
    return generate_analysis(template)
```

### Automated Quality Scoring

```python
def score_proposal_quality(proposal):
    """Evaluate proposal completeness and quality"""
    
    score = 0
    max_score = 100
    
    # Check completeness
    if len(proposal["sections"]) >= 5:
        score += 20
    
    # Check research depth
    if proposal["research"]["data_points"] >= 10:
        score += 20
    
    # Check insight quality
    if proposal["insights"]["uniqueness_score"] >= 0.7:
        score += 20
    
    # Check narrative flow
    if proposal["coherence_score"] >= 0.8:
        score += 20
    
    # Check visual suggestions
    if all(slide.get("visual") for slide in proposal["slides"]):
        score += 20
    
    return score, max_score
```

This skill enables AI coding agents to orchestrate complex, multi-stage brand marketing strategy proposal generation with human-in-the-loop validation at critical decision points.
