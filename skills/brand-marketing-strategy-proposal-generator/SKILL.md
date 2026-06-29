---
name: brand-marketing-strategy-proposal-generator
description: Generate comprehensive brand marketing strategy proposals with automated research, insights, and PPT script generation
triggers:
  - generate a brand marketing strategy proposal
  - create a marketing strategy document for a client
  - build a brand strategy presentation
  - analyze client data and create marketing recommendations
  - generate PPT content script for brand strategy
  - create a marketing proposal from client materials
  - develop a comprehensive brand strategy deck
  - automate brand marketing proposal generation
---

# Brand Marketing Strategy Proposal Generator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An automated workflow system for generating professional brand marketing strategy proposals. This tool takes client materials through a structured process: needs analysis, problem definition, research, opportunity insights, content structuring, and PPT script generation with Word document output.

## Installation

Clone the repository:

```bash
git clone https://github.com/yuleiwang156-a11y/brand-marketing-strategy-proposal-skill.git
cd brand-marketing-strategy-proposal-skill
```

Create the required directory structure:

```bash
mkdir -p inputs/客户资料
mkdir -p outputs
mkdir -p agent_memory
mkdir -p backups
```

## Project Structure

```
brand-marketing-strategy-proposal-skill/
├── inputs/
│   └── 客户资料/
│       └── [客户名称]/          # Client-specific materials
├── outputs/                      # Generated proposals
├── agent_memory/                 # Process state and memory
├── backups/                      # Backup files
├── MEMORY.md                     # Agent memory log
└── README.md
```

## Core Workflow

The proposal generation follows a structured 7-step process:

1. **Client Data Input** — Place all client materials in the appropriate folder
2. **Confirmation Point 1** — Client needs understanding and project problem definition
3. **Research Analysis Layer** — Deep dive into market, competitors, and brand positioning
4. **Confirmation Point 2** — Specific opportunity insights identification
5. **Confirmation Point 3** — Client-visible outline and internal analysis structure
6. **PPT Content Script Generation** — Slide-by-slide content creation
7. **Quality Check & Word Document Output** — Final review and document generation

## Usage

### Basic Workflow

1. **Prepare client materials:**

```bash
# Create client folder
mkdir -p "inputs/客户资料/客户名称"

# Add client materials (briefs, data, research, etc.)
cp /path/to/client/materials/* "inputs/客户资料/客户名称/"
```

2. **Initiate the proposal generation:**

Provide this prompt to your AI agent:

```text
请按照本项目 Skill 执行。本次客户资料在 inputs/客户资料/客户名称/ 中，请从确认点 1 开始。
```

3. **Follow the confirmation points:**

The agent will guide you through each confirmation point, requiring your approval before proceeding to the next stage.

### Stage-by-Stage Process

#### Stage 1: Needs Understanding

The agent analyzes client materials to extract:
- Business objectives
- Target audience
- Market challenges
- Brand positioning needs
- Budget and timeline constraints

**Example interaction:**

```text
Agent: 已完成客户需求分析。发现以下关键点：
1. 目标市场：25-40岁都市白领
2. 核心挑战：品牌认知度低
3. 预算范围：50-100万
4. 期望成果：提升品牌曝光率30%

是否确认继续到研究分析层？
```

#### Stage 2: Research Analysis

The agent conducts:
- Competitive landscape analysis
- Market trend research
- Consumer behavior insights
- Brand gap analysis
- SWOT analysis

#### Stage 3: Opportunity Insights

The agent identifies and prioritizes:
- Market opportunities
- Differentiation points
- Strategic recommendations
- Tactical approaches

**Example confirmation:**

```text
Agent: 识别到3个核心机会点：
1. 社交媒体KOL合作（影响力最大）
2. 内容营销与品牌故事（长期价值）
3. 线下体验活动（转化率高）

建议优先级：2 → 1 → 3
是否确认此洞察框架？
```

#### Stage 4: Outline Creation

The agent creates:
- Executive summary structure
- Main content sections
- Supporting analysis framework
- Appendix organization

**Example structure:**

```markdown
## 客户可见目录
1. 执行摘要
2. 市场机会分析
3. 战略建议
4. 实施路线图
5. 预期成果

## 内部分析结构
- 竞品详细对比表
- 消费者调研数据
- 媒介投放建议
- 预算分配方案
```

#### Stage 5: PPT Script Generation

The agent generates slide-by-slide content:

**Example output format:**

```markdown
# Slide 1: 封面
标题：[客户名称] 品牌营销战略建议书
副标题：驱动增长的三大核心策略
视觉建议：品牌主色调背景，简洁现代设计

# Slide 2: 执行摘要
核心信息：
- 当前挑战：品牌认知度不足，市场份额3%
- 战略目标：12个月内提升至8%
- 三大策略：内容营销、KOL合作、体验活动
- 预期投资回报率：1:4.2

视觉建议：信息图表展示ROI预期
```

#### Stage 6: Quality Check & Output

The agent:
- Reviews consistency across all sections
- Checks data accuracy
- Validates strategic logic
- Generates Word document

**Example output:**

```bash
outputs/
└── 客户名称_品牌营销战略建议书_20260615.docx
```

## Advanced Configuration

### Custom Workflow Parameters

Create a configuration file for specific requirements:

```python
# config.py
PROPOSAL_CONFIG = {
    "client_name": "示例客户",
    "project_type": "品牌重塑",
    "analysis_depth": "comprehensive",  # basic, standard, comprehensive
    "output_format": ["docx", "pptx"],
    "language": "zh-CN",
    "include_sections": [
        "market_analysis",
        "competitive_landscape",
        "consumer_insights",
        "strategy_recommendations",
        "implementation_roadmap",
        "budget_allocation",
        "kpi_framework"
    ],
    "confirmation_required": True,
    "auto_backup": True
}
```

### Memory Management

The agent maintains state in `MEMORY.md`:

```markdown
# 项目记忆

## 当前项目：示例客户
- 开始时间：2026-06-15 14:30
- 当前阶段：确认点 2
- 上次确认：机会点洞察已通过

## 关键决策记录
1. [14:35] 确认目标受众为25-40岁都市白领
2. [14:42] 选择优先策略：内容营销
```

## Common Patterns

### Pattern 1: Multi-Client Batch Processing

```bash
# Process multiple clients sequentially
for client in inputs/客户资料/*/; do
    echo "Processing $(basename $client)..."
    # Trigger agent with client path
done
```

### Pattern 2: Iterative Refinement

```text
User: 第2个机会点需要调整，请重新分析社交媒体机会
Agent: 了解，将重新评估社交媒体渠道...
[Agent regenerates specific section]
```

### Pattern 3: Template Customization

```python
# custom_template.py
SLIDE_TEMPLATES = {
    "executive_summary": {
        "title": "执行摘要",
        "sections": ["current_state", "objectives", "strategies", "roi"],
        "visual_style": "infographic"
    },
    "market_analysis": {
        "title": "市场分析",
        "sections": ["market_size", "trends", "segments"],
        "visual_style": "charts_and_graphs"
    }
}
```

## Environment Variables

Set these in your `.env` file (never commit):

```bash
# .env
CLIENT_DATA_PATH=inputs/客户资料
OUTPUT_PATH=outputs
BACKUP_PATH=backups
MEMORY_PATH=agent_memory

# Optional: API keys for enhanced research
MARKET_RESEARCH_API_KEY=${YOUR_RESEARCH_API_KEY}
COMPETITOR_ANALYSIS_API_KEY=${YOUR_ANALYSIS_API_KEY}
```

## Troubleshooting

### Issue: Agent stops at confirmation point

**Solution:** Explicitly provide confirmation:

```text
确认继续。请进入下一阶段。
```

### Issue: Missing client materials

**Solution:** Verify directory structure:

```bash
ls -R inputs/客户资料/客户名称/
# Should show all uploaded files
```

### Issue: Output quality inconsistent

**Solution:** Enable comprehensive analysis mode:

```text
请使用 comprehensive 分析深度重新执行。
```

### Issue: Memory file corrupted

**Solution:** Restore from backup:

```bash
cp backups/MEMORY_backup_20260615.md MEMORY.md
```

## Best Practices

1. **Client Data Organization:** Use consistent naming conventions for client folders
2. **Confirmation Discipline:** Review each confirmation point carefully before proceeding
3. **Version Control:** Keep backups of each major milestone
4. **Data Privacy:** Never commit client materials to version control
5. **Iterative Refinement:** Use stage-specific regeneration for targeted improvements

## .gitignore Configuration

Ensure these patterns are in `.gitignore`:

```gitignore
# Client data (sensitive)
inputs/客户资料/*/

# Generated outputs
outputs/
backups/
agent_memory/
MEMORY.md

# Environment
.env
*.log
```

## Integration Examples

### With Document Generators

```python
# Export to PowerPoint
from pptx import Presentation

def generate_pptx_from_script(script_path, output_path):
    prs = Presentation()
    # Parse script and create slides
    with open(script_path, 'r', encoding='utf-8') as f:
        script = f.read()
    # ... slide generation logic
    prs.save(output_path)
```

### With CRM Systems

```python
# Log proposal generation to CRM
import requests

def log_to_crm(client_name, proposal_path):
    response = requests.post(
        f"{CRM_API_URL}/proposals",
        headers={"Authorization": f"Bearer {CRM_API_KEY}"},
        json={
            "client": client_name,
            "document": proposal_path,
            "status": "generated"
        }
    )
    return response.json()
```

This skill enables AI agents to guide users through professional brand marketing strategy proposal generation with structured workflows, quality checkpoints, and automated documentation.
