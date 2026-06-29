---
name: brand-marketing-strategy-proposal-skill
description: Generate comprehensive brand marketing strategy proposals with AI-driven analysis, insights, and PPT-ready content scripts
triggers:
  - generate a brand marketing strategy proposal
  - create a marketing strategy document for a client
  - build a brand strategy presentation
  - analyze client data and create marketing recommendations
  - generate PPT content for brand strategy
  - create a marketing strategy proposal workflow
  - develop brand positioning strategy documents
  - analyze market opportunities and create strategy deck
---

# Brand Marketing Strategy Proposal Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An Agent Skill for generating professional brand marketing strategy proposals. This project automates the workflow from client data analysis through PPT content script generation and Word document output, following a structured consultation methodology.

## What This Project Does

This skill implements a multi-stage brand marketing strategy development process:

1. **Client Needs Understanding** - Analyzes client materials and defines project scope
2. **Research & Analysis** - Conducts market, competitor, and brand analysis
3. **Opportunity Insights** - Identifies strategic opportunities from research
4. **Structure Building** - Creates client-facing table of contents and internal analysis framework
5. **Content Generation** - Produces slide-by-slide PPT content scripts
6. **Quality Control** - Reviews and validates output quality
7. **Document Export** - Generates final Word document deliverable

## Installation

Clone the repository and set up the directory structure:

```bash
git clone https://github.com/yuleiwang156-a11y/brand-marketing-strategy-proposal-skill.git
cd brand-marketing-strategy-proposal-skill

# Create required directories
mkdir -p inputs/客户资料
mkdir -p outputs
mkdir -p agent_memory
mkdir -p backups
```

## Directory Structure

```
project/
├── inputs/
│   └── 客户资料/
│       └── [客户名称]/          # Client-specific folder
│           ├── 品牌资料.pdf
│           ├── 市场数据.xlsx
│           └── 需求说明.docx
├── outputs/                     # Generated proposals
├── agent_memory/                # Workflow state and memory
├── backups/                     # Backup copies
├── MEMORY.md                    # Session memory
└── SKILL.md                     # This file
```

## Key Workflow Stages

### Stage 1: Client Needs Understanding (确认点 1)

Analyze client materials and define the project problem statement.

**Agent Command:**
```text
请按照本项目 Skill 执行。本次客户资料在 inputs/客户资料/[客户名称]/ 中，请从确认点 1 开始。
```

**Expected Output:**
- Client background summary
- Core business challenges
- Marketing objectives
- Project scope definition
- Key questions to address

### Stage 2: Research & Analysis Layer (研究分析层)

Conduct comprehensive analysis across multiple dimensions.

**Analysis Components:**
- Market environment analysis
- Competitive landscape mapping
- Consumer insights
- Brand equity assessment
- Category trends
- Distribution channel analysis

**Agent Command:**
```text
已确认需求理解，请继续研究分析层。
```

### Stage 3: Opportunity Insights (确认点 2)

Synthesize research into actionable strategic opportunities.

**Output Format:**
- 3-5 key opportunity areas
- Supporting evidence from research
- Strategic implications
- Priority ranking

**Agent Command:**
```text
研究分析完成，请提炼机会点洞察。
```

### Stage 4: Structure Building (确认点 3)

Create dual-layer structure:
- **Client-facing TOC** - Presentation outline
- **Internal analysis structure** - Research framework

**Example Structure:**
```markdown
## 客户可见目录
1. 执行摘要
2. 市场洞察
3. 品牌现状诊断
4. 战略机会点
5. 核心策略建议
6. 实施路线图
7. 预期成果

## 内部分析结构
- 数据源清单
- 分析模型应用
- 假设与验证
- 风险评估
```

**Agent Command:**
```text
机会点已确认，请搭建目录结构。
```

### Stage 5: PPT Content Script Generation (逐页生成)

Generate slide-by-slide content scripts with:
- Slide title
- Key message
- Supporting points (3-5 bullets)
- Data/evidence
- Visual suggestions

**Example Script:**
```markdown
### 第 3 页：市场规模与增长趋势

**核心信息：**
目标市场呈现双位数增长，年轻消费群体驱动需求升级

**要点：**
- 2023-2025 年市场 CAGR 达 12.5%
- 25-35 岁消费者占比从 32% 增至 41%
- 高端化趋势明显，客单价提升 18%
- 线上渠道占比突破 45%

**数据来源：**
[客户资料]/市场数据.xlsx, Sheet2

**视觉建议：**
柱状图展示市场规模增长 + 饼图展示消费者年龄分布变化
```

**Agent Command:**
```text
目录已确认，请开始逐页生成 PPT 内容脚本。
```

### Stage 6: Quality Check (质量检查)

Validate output against quality criteria:
- Logic consistency
- Evidence support
- Actionability
- Client alignment
- Completeness

**Agent Command:**
```text
内容生成完成，请执行质量检查。
```

### Stage 7: Word Document Export (文档输出)

Generate final Word document with:
- Cover page
- Table of contents
- All slide scripts formatted as sections
- Appendices (data sources, methodology)

**Agent Command:**
```text
质量检查通过，请生成 Word 文档。
```

**Output Location:**
```
outputs/[客户名称]_品牌营销战略建议书_[日期].docx
```

## Configuration

### Environment Variables

```bash
# Optional: Configure AI model preferences
export ANALYSIS_MODEL="gpt-4"
export CONTENT_MODEL="gpt-4"

# Optional: Set language preference
export OUTPUT_LANGUAGE="zh-CN"

# Optional: Quality threshold
export QUALITY_SCORE_THRESHOLD="0.85"
```

### Memory Management

The skill uses `MEMORY.md` and `agent_memory/` to maintain context across sessions:

```markdown
## 当前项目状态
- 客户：[客户名称]
- 阶段：确认点 2 - 机会点洞察
- 最后更新：2026-06-15

## 已确认内容
- 核心需求：提升品牌认知度和市场份额
- 主要挑战：竞争激烈，品牌差异化不足
- 目标受众：25-40 岁都市中产

## 待办事项
- [ ] 完成机会点优先级排序
- [ ] 用户确认前三大机会点
- [ ] 继续目录搭建
```

## Code Examples

### Example 1: Initiating Analysis Workflow

```python
# workflow.py - Example structure (not in original repo)
class BrandStrategyWorkflow:
    def __init__(self, client_name):
        self.client_name = client_name
        self.input_path = f"inputs/客户资料/{client_name}"
        self.output_path = f"outputs/{client_name}"
        self.stage = "init"
    
    def load_client_materials(self):
        """Load all client materials from input directory"""
        materials = []
        import os
        for file in os.listdir(self.input_path):
            if file.endswith(('.pdf', '.docx', '.xlsx', '.pptx')):
                materials.append(os.path.join(self.input_path, file))
        return materials
    
    def checkpoint_1_needs_analysis(self, materials):
        """Stage 1: Understand client needs and define problem"""
        analysis = {
            "client_background": "",
            "core_challenges": [],
            "marketing_objectives": [],
            "project_scope": "",
            "key_questions": []
        }
        # AI agent populates this structure
        return analysis
    
    def checkpoint_2_opportunity_insights(self, research_data):
        """Stage 3: Extract strategic opportunities"""
        opportunities = []
        # AI agent identifies 3-5 key opportunities
        return opportunities
    
    def generate_ppt_scripts(self, toc_structure):
        """Stage 5: Generate slide-by-slide content"""
        scripts = []
        for section in toc_structure:
            for slide_num in section['slides']:
                script = {
                    "slide_number": slide_num,
                    "title": "",
                    "key_message": "",
                    "bullets": [],
                    "data_source": "",
                    "visual_suggestion": ""
                }
                scripts.append(script)
        return scripts
```

### Example 2: Quality Check Implementation

```python
# quality_check.py
class QualityChecker:
    def __init__(self, content_scripts):
        self.scripts = content_scripts
        self.quality_criteria = {
            "logic_consistency": 0.0,
            "evidence_support": 0.0,
            "actionability": 0.0,
            "completeness": 0.0
        }
    
    def check_logic_consistency(self):
        """Verify logical flow between slides"""
        score = 0.0
        # Check if each slide builds on previous
        # Verify no contradictions
        return score
    
    def check_evidence_support(self):
        """Ensure all claims have data backing"""
        score = 0.0
        unsupported = []
        for script in self.scripts:
            if not script.get('data_source'):
                unsupported.append(script['slide_number'])
        score = 1.0 - (len(unsupported) / len(self.scripts))
        return score, unsupported
    
    def check_actionability(self):
        """Verify recommendations are concrete"""
        score = 0.0
        # Look for specific, measurable actions
        return score
    
    def run_full_check(self):
        """Execute all quality checks"""
        self.quality_criteria['logic_consistency'] = self.check_logic_consistency()
        self.quality_criteria['evidence_support'], _ = self.check_evidence_support()
        self.quality_criteria['actionability'] = self.check_actionability()
        
        overall_score = sum(self.quality_criteria.values()) / len(self.quality_criteria)
        return overall_score, self.quality_criteria
```

### Example 3: Document Export

```python
# export_docx.py
from docx import Document
from docx.shared import Pt, Inches
from datetime import datetime

class ProposalExporter:
    def __init__(self, client_name, scripts):
        self.client_name = client_name
        self.scripts = scripts
        self.doc = Document()
    
    def add_cover_page(self):
        """Create cover page"""
        self.doc.add_heading(f'{self.client_name} 品牌营销战略建议书', 0)
        self.doc.add_paragraph(f'生成日期：{datetime.now().strftime("%Y-%m-%d")}')
        self.doc.add_page_break()
    
    def add_table_of_contents(self, toc_structure):
        """Insert table of contents"""
        self.doc.add_heading('目录', 1)
        for idx, section in enumerate(toc_structure, 1):
            self.doc.add_paragraph(f'{idx}. {section["title"]}', style='List Number')
        self.doc.add_page_break()
    
    def add_slide_script(self, script):
        """Format single slide script"""
        self.doc.add_heading(f'第 {script["slide_number"]} 页：{script["title"]}', 2)
        
        self.doc.add_heading('核心信息：', 3)
        self.doc.add_paragraph(script['key_message'])
        
        self.doc.add_heading('要点：', 3)
        for bullet in script['bullets']:
            self.doc.add_paragraph(bullet, style='List Bullet')
        
        if script.get('data_source'):
            self.doc.add_heading('数据来源：', 3)
            self.doc.add_paragraph(script['data_source'])
        
        if script.get('visual_suggestion'):
            self.doc.add_heading('视觉建议：', 3)
            self.doc.add_paragraph(script['visual_suggestion'])
        
        self.doc.add_paragraph()  # Spacing
    
    def export(self, output_path):
        """Save document"""
        self.doc.save(output_path)
        return output_path

# Usage
exporter = ProposalExporter("示例客户", ppt_scripts)
exporter.add_cover_page()
exporter.add_table_of_contents(toc_structure)
for script in ppt_scripts:
    exporter.add_slide_script(script)

output_file = f"outputs/{exporter.client_name}_品牌营销战略建议书_{datetime.now().strftime('%Y%m%d')}.docx"
exporter.export(output_file)
```

## Common Patterns

### Pattern 1: Iterative Refinement

Allow user to refine outputs at each checkpoint:

```text
# At Checkpoint 2
User: "机会点 3 需要更聚焦在数字化营销，请调整"
Agent: "明白，我将重新分析数字化渠道机会，包括社交媒体、KOL 合作和内容营销..."
```

### Pattern 2: Partial Regeneration

Regenerate specific sections without restarting:

```text
User: "请重新生成第 15-20 页的内容脚本，增加更多竞品对比数据"
Agent: "收到，我将重点补充竞品对比维度..."
```

### Pattern 3: Memory Persistence

Resume work across sessions:

```text
# New session
User: "继续上次的项目"
Agent: "上次工作在 [客户名称] 项目的确认点 2，已完成机会点洞察。是否继续目录搭建？"
```

## Troubleshooting

### Issue: Missing Client Materials

**Problem:** Agent cannot find client data files

**Solution:**
```bash
# Verify directory structure
ls -la inputs/客户资料/[客户名称]/

# Ensure files are in supported formats
# Supported: .pdf, .docx, .xlsx, .pptx, .txt
```

### Issue: Incomplete Analysis

**Problem:** Research layer lacks depth

**Solution:**
```text
User: "研究分析层需要补充以下维度：[具体维度]。请重新执行研究分析。"
```

Provide more specific guidance:
- List required data sources
- Specify analysis frameworks (SWOT, Porter's Five Forces, etc.)
- Define minimum evidence requirements

### Issue: Quality Check Fails

**Problem:** Content doesn't meet quality threshold

**Solution:**
```python
# Lower threshold temporarily for review
export QUALITY_SCORE_THRESHOLD="0.75"

# Or identify specific failing criteria
quality_checker = QualityChecker(scripts)
score, criteria = quality_checker.run_full_check()
print(f"Failing criteria: {[k for k, v in criteria.items() if v < 0.8]}")
```

### Issue: Export Formatting Problems

**Problem:** Word document formatting is broken

**Solution:**
```python
# Install/update python-docx
pip install --upgrade python-docx

# Use template for consistent formatting
from docx import Document
doc = Document('templates/proposal_template.docx')
```

### Issue: Memory Persistence Fails

**Problem:** Agent loses context between sessions

**Solution:**
```bash
# Check memory files exist
ls -la agent_memory/ MEMORY.md

# Manually restore from backup
cp backups/MEMORY_[timestamp].md MEMORY.md

# Explicit context reminder
```
```text
User: "当前项目：[客户名称]，阶段：确认点 2，已完成：[列出已完成内容]"
```

## Best Practices

1. **Organize Client Materials Clearly**
   - Use consistent naming conventions
   - Include README.txt with file descriptions
   - Separate primary vs. reference materials

2. **Confirm Each Checkpoint**
   - Review and approve before proceeding
   - Request modifications if needed
   - Document decisions in MEMORY.md

3. **Maintain Version Control**
   ```bash
   # Backup before major changes
   cp -r outputs/ backups/outputs_$(date +%Y%m%d_%H%M%S)/
   cp MEMORY.md backups/MEMORY_$(date +%Y%m%d_%H%M%S).md
   ```

4. **Use Structured Prompts**
   - Be specific about data sources
   - Reference previous checkpoints
   - Provide examples of desired output

5. **Quality Over Speed**
   - Don't skip quality checks
   - Iterate on weak sections
   - Validate data sources

## Security Notes

- **Never commit client data** to version control
- Add to `.gitignore`:
  ```
  inputs/客户资料/
  outputs/
  agent_memory/
  MEMORY.md
  backups/
  *.docx
  *.pdf
  *.xlsx
  ```

- **Use environment variables** for sensitive config:
  ```bash
  export CLIENT_NAME="client_code_name"  # Not real client name
  export API_KEY="${YOUR_API_KEY}"       # For external data sources
  ```

## Advanced Usage

### Custom Analysis Frameworks

Extend research layer with proprietary frameworks:

```python
# custom_frameworks.py
class CustomAnalysisFramework:
    def __init__(self, framework_name):
        self.name = framework_name
        self.dimensions = []
    
    def add_dimension(self, dimension, weight):
        self.dimensions.append({"name": dimension, "weight": weight})
    
    def apply_to_client(self, client_data):
        results = {}
        for dim in self.dimensions:
            # Apply analysis
            results[dim['name']] = self.analyze_dimension(client_data, dim)
        return results
```

### Multi-Client Comparison

Compare strategies across multiple clients:

```bash
# Generate comparison report
python scripts/compare_strategies.py \
  --clients "客户A,客户B,客户C" \
  --output "outputs/strategy_comparison.xlsx"
```

This skill enables AI agents to guide users through professional brand marketing strategy development with structured methodology, quality controls, and deliverable generation.
