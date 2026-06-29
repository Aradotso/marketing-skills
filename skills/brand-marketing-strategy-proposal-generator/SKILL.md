---
name: brand-marketing-strategy-proposal-generator
description: Generate comprehensive brand marketing strategy proposals with research, insights, and structured PPT scripts
triggers:
  - generate a brand marketing strategy proposal
  - create a marketing strategy document for a client
  - build a brand strategy presentation
  - analyze client data and create marketing recommendations
  - produce a strategic marketing proposal
  - develop a brand positioning strategy document
  - create a multi-page marketing strategy PPT script
  - generate marketing insights from client research
---

# Brand Marketing Strategy Proposal Generator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An agent-based workflow system for generating comprehensive brand marketing strategy proposals. Takes client data through a multi-stage process including needs analysis, research, opportunity identification, outline creation, and slide-by-slide script generation with Word document output.

## What It Does

This skill automates the creation of professional marketing strategy proposals by:

- Analyzing client background materials and requirements
- Defining project problems and objectives
- Conducting research and competitive analysis
- Identifying strategic opportunities
- Building structured presentation outlines
- Generating detailed slide-by-slide scripts
- Outputting formatted Word documents

The workflow includes multiple confirmation checkpoints to ensure accuracy and alignment with client needs.

## Installation

Clone or download the project into your workspace:

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
│       └── [客户名称]/          # Client-specific folders
│           ├── brief.txt        # Client brief
│           ├── background.pdf   # Company background
│           └── research.docx    # Market research
├── outputs/                     # Generated proposals
├── agent_memory/                # Session state (gitignored)
├── backups/                     # Workflow backups (gitignored)
└── MEMORY.md                    # Project memory (gitignored)
```

## Core Workflow

The skill follows a six-stage process with three confirmation checkpoints:

### Stage 1: Client Input
Place all client materials in `inputs/客户资料/[客户名称]/`

### Stage 2: Checkpoint 1 - Needs Understanding & Problem Definition
```text
请按照本项目 Skill 执行。本次客户资料在 inputs/客户资料/[客户名称]/ 中，请从确认点 1 开始。
```

The agent will:
- Read and analyze all client materials
- Extract key business objectives
- Define core marketing challenges
- Propose initial problem statements
- Wait for user confirmation

### Stage 3: Research & Analysis Layer
After checkpoint 1 approval, the agent conducts:
- Industry landscape analysis
- Competitive positioning research
- Consumer insights synthesis
- Market trends identification
- SWOT analysis

### Stage 4: Checkpoint 2 - Opportunity Insights
The agent presents:
- Strategic opportunity areas
- Positioning recommendations
- Target audience refinements
- Key differentiators
- Wait for user confirmation

### Stage 5: Checkpoint 3 - Outline Structure
The agent provides:
- Client-facing table of contents
- Internal analysis structure
- Slide sequence and topics
- Narrative flow
- Wait for user confirmation

### Stage 6: Script Generation & Output
The agent generates:
- Slide-by-slide content scripts
- Visual direction notes
- Supporting data and evidence
- Quality checks
- Final Word document output

## Usage Patterns

### Basic Workflow

```text
User: 请为客户"ABC品牌"生成营销战略建议书，资料在 inputs/客户资料/ABC品牌/ 中

Agent: 
1. 读取客户资料...
2. 【确认点 1】需求理解与问题定义：
   - 客户目标：提升品牌年轻化认知
   - 核心问题：品牌老化，年轻消费者流失
   - 项目范围：品牌重塑战略
   
是否确认继续？

User: 确认

Agent:
3. 执行研究分析...
4. 【确认点 2】机会点洞察：
   - 机会1：Z世代文化共鸣
   - 机会2：数字化体验升级
   - 机会3：可持续发展叙事
   
是否确认继续？
```

### Resuming from Checkpoint

```text
请从确认点 2 继续，使用 agent_memory/session_abc.json 中的状态
```

### Regenerating Specific Sections

```text
请重新生成目录结构（确认点 3），保持之前的机会点洞察不变
```

### Custom Analysis Depth

```text
请执行深度竞争分析，包含至少 5 个主要竞品的详细对比矩阵
```

## Configuration

### Client Data Requirements

Organize client materials with clear naming:

```
inputs/客户资料/客户名称/
├── 01_项目简报.docx          # Project brief
├── 02_品牌背景.pdf           # Brand background
├── 03_市场数据.xlsx          # Market data
├── 04_竞品分析.pptx          # Competitive analysis
└── 05_消费者调研.pdf         # Consumer research
```

### Memory Management

The agent uses `MEMORY.md` and `agent_memory/` to maintain state:

```markdown
# MEMORY.md structure
## Current Project: [客户名称]
## Workflow Stage: [确认点 1/2/3 或生成中]
## Key Decisions:
- Problem definition: [...]
- Approved opportunities: [...]
- Outline structure: [...]
```

### Output Formats

Default output structure:

```
outputs/
└── [客户名称]_品牌营销战略建议书_[日期]/
    ├── 01_需求定义.md
    ├── 02_研究分析.md
    ├── 03_机会洞察.md
    ├── 04_提案目录.md
    ├── 05_PPT脚本.md
    └── 最终建议书.docx
```

## Code Examples

### Python: Parsing Client Materials

```python
import os
from pathlib import Path

def load_client_data(client_name):
    """Load all client materials from input directory"""
    client_dir = Path(f"inputs/客户资料/{client_name}")
    
    materials = {
        'brief': None,
        'background': None,
        'research': [],
        'data': []
    }
    
    if not client_dir.exists():
        raise ValueError(f"Client directory not found: {client_dir}")
    
    for file in client_dir.iterdir():
        if file.suffix in ['.txt', '.md']:
            materials['brief'] = file.read_text(encoding='utf-8')
        elif file.suffix == '.pdf':
            materials['background'] = file
        elif file.suffix in ['.docx', '.doc']:
            materials['research'].append(file)
        elif file.suffix in ['.xlsx', '.csv']:
            materials['data'].append(file)
    
    return materials

# Usage
client_data = load_client_data("ABC品牌")
print(f"Found {len(client_data['research'])} research documents")
```

### Python: Generating Structured Output

```python
from dataclasses import dataclass
from typing import List
from datetime import datetime

@dataclass
class OpportunityInsight:
    title: str
    description: str
    evidence: List[str]
    potential_impact: str

@dataclass
class ProposalOutline:
    client_name: str
    sections: List[dict]
    total_slides: int
    
    def to_markdown(self):
        md = f"# {self.client_name} 品牌营销战略建议书\n\n"
        md += f"生成时间: {datetime.now().strftime('%Y-%m-%d')}\n\n"
        md += f"总页数: {self.total_slides}\n\n"
        
        for i, section in enumerate(self.sections, 1):
            md += f"## {i}. {section['title']}\n"
            md += f"页数: {section['slides']}\n"
            md += f"内容: {section['description']}\n\n"
        
        return md

# Usage
outline = ProposalOutline(
    client_name="ABC品牌",
    sections=[
        {
            'title': '品牌现状诊断',
            'slides': 5,
            'description': '市场地位、消费者认知、竞争态势'
        },
        {
            'title': '战略机会洞察',
            'slides': 8,
            'description': 'Z世代文化共鸣、数字化体验升级'
        }
    ],
    total_slides=45
)

output_path = Path(f"outputs/{outline.client_name}_提案目录.md")
output_path.write_text(outline.to_markdown(), encoding='utf-8')
```

### Python: Checkpoint State Management

```python
import json
from enum import Enum

class WorkflowStage(Enum):
    CHECKPOINT_1 = "needs_understanding"
    RESEARCH = "research_analysis"
    CHECKPOINT_2 = "opportunity_insights"
    CHECKPOINT_3 = "outline_structure"
    GENERATION = "script_generation"
    COMPLETE = "complete"

class WorkflowState:
    def __init__(self, client_name):
        self.client_name = client_name
        self.stage = WorkflowStage.CHECKPOINT_1
        self.data = {}
        self.approvals = []
    
    def save(self):
        state_file = Path(f"agent_memory/{self.client_name}_state.json")
        state_file.write_text(json.dumps({
            'client_name': self.client_name,
            'stage': self.stage.value,
            'data': self.data,
            'approvals': self.approvals
        }, ensure_ascii=False, indent=2), encoding='utf-8')
    
    @classmethod
    def load(cls, client_name):
        state_file = Path(f"agent_memory/{client_name}_state.json")
        if not state_file.exists():
            return cls(client_name)
        
        state_data = json.loads(state_file.read_text(encoding='utf-8'))
        instance = cls(state_data['client_name'])
        instance.stage = WorkflowStage(state_data['stage'])
        instance.data = state_data['data']
        instance.approvals = state_data['approvals']
        return instance

# Usage
state = WorkflowState("ABC品牌")
state.data['problem_definition'] = "品牌老化，年轻消费者流失"
state.approvals.append({'checkpoint': 1, 'approved': True})
state.save()

# Resume later
resumed_state = WorkflowState.load("ABC品牌")
print(f"Resuming from: {resumed_state.stage.value}")
```

## Common Patterns

### Pattern 1: Iterative Refinement

```text
User: 请生成建议书，资料在 inputs/客户资料/XYZ/
Agent: [生成确认点1]
User: 问题定义太宽泛，请聚焦在年轻化转型
Agent: [调整后的问题定义]
User: 确认
Agent: [继续到确认点2]
```

### Pattern 2: Parallel Analysis

```text
User: 在执行研究分析时，请同时进行：
1. 5个竞品的详细对比
2. 3个目标人群的深度画像
3. 近3年的市场趋势分析

Agent: [并行执行三项分析任务]
```

### Pattern 3: Template-Based Generation

```text
User: 使用"快消品行业模板"生成建议书框架
Agent: [应用行业特定模板，包含渠道分析、促销策略等章节]
```

## Troubleshooting

### Issue: Client materials not found

```text
Error: Client directory not found: inputs/客户资料/客户名称/

Solution:
1. Verify directory exists and name matches exactly (case-sensitive)
2. Check for special characters or spaces in folder name
3. Ensure materials are not nested in subdirectories
```

### Issue: Memory state corruption

```text
Error: Cannot resume from checkpoint - state file corrupted

Solution:
# Restore from backup
cp backups/[客户名称]_state_backup.json agent_memory/[客户名称]_state.json

# Or start fresh
rm agent_memory/[客户名称]_state.json
```

### Issue: Incomplete output generation

```text
Problem: Word document missing slides 15-20

Solution:
请从第15页开始重新生成，使用已保存的outline结构：
- 保持前14页内容不变
- 从"第三章：战略建议"的第15页继续
- 确保与整体叙事连贯
```

### Issue: Generic or shallow insights

```text
Problem: Generated insights too generic

Solution:
请深化分析，要求：
1. 每个洞察必须引用至少2个具体数据点
2. 包含至少1个真实案例参考
3. 明确说明与客户业务的关联性
4. 提供可量化的预期影响
```

## Best Practices

1. **Organize client materials clearly** - Use numbered prefixes and descriptive names
2. **Review each checkpoint carefully** - The quality of later stages depends on early approvals
3. **Save intermediate states** - Use the backup system for complex projects
4. **Provide context at checkpoints** - Give specific feedback to guide refinement
5. **Use environment-specific configurations** - Keep sensitive client data out of version control

## Environment Variables

```bash
# Optional: Configure output preferences
export PROPOSAL_LANGUAGE="zh-CN"
export PROPOSAL_FORMAT="docx"
export ANALYSIS_DEPTH="detailed"  # standard, detailed, or comprehensive
export OUTPUT_DIR="outputs"
```

## Security Notes

- Never commit client materials to version control
- Add to `.gitignore`:
  ```
  inputs/客户资料/
  outputs/
  agent_memory/
  backups/
  MEMORY.md
  *.docx
  *.pdf
  ```
- Use environment variables for any API keys or credentials
- Sanitize client names in logs and error messages
