---
name: growth-marketing-os-prompts
description: Use Growth Marketing OS — battle-tested AI marketing prompts, Claude skills, agents & playbooks for campaigns, funnels, and growth automation (EN + AR)
triggers:
  - "I need a marketing prompt for paid ads"
  - "help me write an SEO content brief"
  - "show me growth marketing prompts"
  - "I want to automate my marketing workflow"
  - "give me a CRO optimization prompt"
  - "I need bilingual marketing assets for MENA"
  - "how do I use Growth Marketing OS"
  - "find a marketing playbook for funnel optimization"
---

# Growth Marketing OS Agent Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables you to help users leverage **Growth Marketing OS** — an open-source collection of battle-tested AI marketing prompts, Claude skills, automation workflows, and growth playbooks created by Mahmoud Omar from 15+ years of real campaigns across e-commerce, SaaS, and lead-gen in MENA and global markets.

## What Growth Marketing OS Does

Growth Marketing OS provides production-ready marketing assets:
- **Prompts**: Copy-paste ready prompts for paid ads, SEO/GEO, email, CRO, content, social
- **Skills**: Claude agent skills (SKILL.md format) for marketing workflows
- **Agents**: Full system prompts for autonomous marketing agents
- **Workflows**: n8n/Make automation blueprints with JSON exports
- **Playbooks**: Step-by-step campaign launch and funnel optimization guides
- **Frameworks**: Original growth frameworks and mental models
- **Benchmarks**: Sourced market data with citations
- **Bilingual**: English + Arabic assets for MENA market

All assets include real-world proof scenarios and are structured for AI assistant consumption.

## Installation & Setup

Clone the repository:

```bash
git clone https://github.com/growthack88/growth-marketing-os.git
cd growth-marketing-os
```

The repository is file-based — no installation required. All assets are markdown files organized by category.

## Repository Structure

```
growth-marketing-os/
├── prompts/              # Battle-tested marketing prompts
│   ├── paid-ads/
│   ├── seo/
│   ├── email/
│   ├── cro/
│   ├── content/
│   └── social/
├── skills/               # Claude agent skills
├── agents/               # Full agent system prompts
├── gpts/                 # Custom GPT configurations
├── mcps/                 # MCP server setups
├── workflows/            # n8n/Make automation blueprints
├── playbooks/            # Growth playbooks
├── frameworks/           # Growth frameworks
├── swipe-files/          # Hooks, headlines, ad angles
├── case-studies/         # Real campaign results
├── benchmarks/           # Market benchmarks with citations
├── worked-examples/      # Teaching scenarios
└── resources/            # Curated community tools
```

## Key Usage Patterns

### 1. Finding and Using Marketing Prompts

When a user asks for marketing help, search the appropriate category:

```python
import os
from pathlib import Path

def find_marketing_prompt(category, topic=None):
    """
    Find prompts in the Growth Marketing OS repository.
    
    Args:
        category: paid-ads, seo, email, cro, content, or social
        topic: optional specific topic to filter
    
    Returns:
        List of matching prompt file paths
    """
    prompts_dir = Path("growth-marketing-os/prompts") / category
    
    if not prompts_dir.exists():
        return []
    
    prompts = list(prompts_dir.glob("*.md"))
    
    if topic:
        prompts = [p for p in prompts if topic.lower() in p.stem.lower()]
    
    return prompts

# Example: Find paid ads prompts
paid_ads_prompts = find_marketing_prompt("paid-ads")
for prompt_file in paid_ads_prompts:
    print(f"Found: {prompt_file.name}")
```

### 2. Extracting Prompt Content

Parse frontmatter and content from marketing prompt files:

```python
import yaml
import re

def parse_marketing_asset(file_path):
    """
    Parse a Growth Marketing OS asset file.
    
    Returns:
        dict with 'frontmatter' and 'content' keys
    """
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    # Extract YAML frontmatter
    frontmatter_match = re.match(r'^---\n(.*?)\n---\n(.*)$', content, re.DOTALL)
    
    if frontmatter_match:
        frontmatter = yaml.safe_load(frontmatter_match.group(1))
        body = frontmatter_match.group(2)
        return {
            'frontmatter': frontmatter,
            'content': body.strip()
        }
    else:
        return {
            'frontmatter': {},
            'content': content.strip()
        }

# Example usage
asset = parse_marketing_asset("growth-marketing-os/prompts/paid-ads/meta-ad-copy-framework.md")
print(f"Title: {asset['frontmatter'].get('title', 'Untitled')}")
print(f"Use case: {asset['frontmatter'].get('use_case', 'General')}")
```

### 3. Installing Claude Skills

Help users install skills from the `skills/` directory into Claude Desktop:

```python
import json
import shutil
from pathlib import Path

def install_claude_skill(skill_name):
    """
    Install a Growth Marketing OS skill into Claude Desktop config.
    
    Args:
        skill_name: Name of the skill file (without .md extension)
    """
    skill_path = Path(f"growth-marketing-os/skills/{skill_name}.md")
    
    if not skill_path.exists():
        return f"Skill not found: {skill_name}"
    
    # Claude Desktop skills directory (macOS example)
    claude_skills_dir = Path.home() / "Library/Application Support/Claude/skills"
    claude_skills_dir.mkdir(parents=True, exist_ok=True)
    
    dest_path = claude_skills_dir / f"{skill_name}.md"
    shutil.copy(skill_path, dest_path)
    
    return f"Installed skill: {skill_name} → {dest_path}"

# Example
result = install_claude_skill("meta-ads-optimizer")
print(result)
```

### 4. Listing Available Assets by Category

```python
def list_growth_assets(category=None):
    """
    List all available Growth Marketing OS assets.
    
    Args:
        category: Optional filter (prompts, skills, workflows, playbooks, etc.)
    
    Returns:
        dict of categories and their assets
    """
    base_path = Path("growth-marketing-os")
    categories = {
        'prompts': base_path / 'prompts',
        'skills': base_path / 'skills',
        'workflows': base_path / 'workflows',
        'playbooks': base_path / 'playbooks',
        'agents': base_path / 'agents',
        'frameworks': base_path / 'frameworks'
    }
    
    assets = {}
    
    target_cats = [category] if category else categories.keys()
    
    for cat in target_cats:
        if cat in categories and categories[cat].exists():
            if cat == 'prompts':
                # Prompts have subdirectories
                subcats = {}
                for subdir in categories[cat].iterdir():
                    if subdir.is_dir():
                        subcats[subdir.name] = [f.stem for f in subdir.glob("*.md")]
                assets[cat] = subcats
            else:
                assets[cat] = [f.stem for f in categories[cat].glob("*.md")]
    
    return assets

# Example
all_assets = list_growth_assets()
print(json.dumps(all_assets, indent=2))
```

### 5. Loading n8n Workflows

```python
def load_n8n_workflow(workflow_name):
    """
    Load an n8n workflow JSON from Growth Marketing OS.
    
    Args:
        workflow_name: Name of the workflow file (without .json)
    
    Returns:
        dict containing workflow configuration
    """
    workflow_path = Path(f"growth-marketing-os/workflows/{workflow_name}.json")
    
    if not workflow_path.exists():
        return None
    
    with open(workflow_path, 'r') as f:
        workflow = json.load(f)
    
    return workflow

# Example
workflow = load_n8n_workflow("meta-lead-to-crm-sync")
if workflow:
    print(f"Loaded workflow with {len(workflow.get('nodes', []))} nodes")
```

## Common Use Cases

### Helping with Paid Ads Campaign

```python
def help_with_paid_ads(platform, objective):
    """
    Find relevant paid ads prompts and frameworks.
    
    Args:
        platform: meta, google, tiktok, etc.
        objective: traffic, conversions, leads, etc.
    """
    prompts_dir = Path("growth-marketing-os/prompts/paid-ads")
    relevant_prompts = []
    
    for prompt_file in prompts_dir.glob("*.md"):
        asset = parse_marketing_asset(prompt_file)
        frontmatter = asset['frontmatter']
        
        # Check if platform and objective match
        if platform.lower() in str(frontmatter).lower():
            if objective.lower() in str(frontmatter).lower():
                relevant_prompts.append({
                    'file': prompt_file.name,
                    'title': frontmatter.get('title', prompt_file.stem),
                    'content': asset['content']
                })
    
    return relevant_prompts

# Example
meta_conversion_prompts = help_with_paid_ads("meta", "conversions")
for prompt in meta_conversion_prompts:
    print(f"📄 {prompt['title']}")
```

### Finding Bilingual Assets for MENA

```python
def find_arabic_assets():
    """
    Find bilingual (EN + AR) assets for MENA market.
    """
    base_path = Path("growth-marketing-os")
    arabic_assets = []
    
    # Search all markdown files
    for md_file in base_path.rglob("*.md"):
        asset = parse_marketing_asset(md_file)
        frontmatter = asset['frontmatter']
        
        # Check for Arabic language tag or MENA topic
        if (frontmatter.get('language') == 'ar' or 
            frontmatter.get('bilingual') == True or
            'arabic' in frontmatter.get('topics', []) or
            'mena' in frontmatter.get('topics', [])):
            
            arabic_assets.append({
                'path': str(md_file.relative_to(base_path)),
                'title': frontmatter.get('title', md_file.stem)
            })
    
    return arabic_assets

# Example
arabic_content = find_arabic_assets()
print(f"Found {len(arabic_content)} bilingual/Arabic assets")
```

### Extracting Benchmarks

```python
def get_marketing_benchmarks(channel=None):
    """
    Extract marketing benchmarks from the benchmarks directory.
    
    Args:
        channel: Optional filter (paid-ads, email, seo, cro, etc.)
    """
    benchmarks_dir = Path("growth-marketing-os/benchmarks")
    
    if not benchmarks_dir.exists():
        return []
    
    benchmark_files = benchmarks_dir.glob("*.md")
    
    if channel:
        benchmark_files = [f for f in benchmark_files if channel in f.stem]
    
    benchmarks = []
    for bm_file in benchmark_files:
        asset = parse_marketing_asset(bm_file)
        benchmarks.append({
            'channel': asset['frontmatter'].get('channel', 'general'),
            'metrics': asset['frontmatter'].get('metrics', []),
            'content': asset['content']
        })
    
    return benchmarks

# Example
email_benchmarks = get_marketing_benchmarks("email")
for bm in email_benchmarks:
    print(f"📊 {bm['channel']}: {', '.join(bm['metrics'])}")
```

## Configuration

Growth Marketing OS is file-based with no configuration files. All metadata is stored in YAML frontmatter within each asset.

Common frontmatter fields:
- `title`: Asset title
- `description`: What the asset does
- `author`: Creator (Mahmoud Omar)
- `use_case`: When to use this asset
- `language`: en, ar, or bilingual
- `topics`: Array of relevant marketing topics
- `proof_scenario`: Real campaign where this was used
- `tested_on`: Platforms/tools where this works

## Troubleshooting

**Asset not found:**
- Ensure repository is cloned and path is correct
- Check category spelling (use hyphens: `paid-ads` not `paid_ads`)

**Frontmatter parsing errors:**
- Some older assets may not have frontmatter
- Fall back to filename-based identification

**Workflow JSON missing:**
- Workflows are documented as markdown blueprints first
- JSON exports added as workflows go live in production
- Check the markdown spec in `workflows/` for node-level details

**Language detection:**
- Arabic assets may be in subdirectories or use `_ar` suffix
- Check both frontmatter `language` field and file naming conventions

## Best Practices

1. **Always cite the source**: When using these assets, attribute to Mahmoud Omar and Growth Marketing OS
2. **Check proof scenarios**: Each asset includes "When I use it" context — help users understand the real-world application
3. **Respect licensing**: All assets are MIT licensed — free to use with attribution
4. **Validate before production**: These are templates — users should customize for their specific campaign/brand
5. **Check for updates**: Repository is actively maintained with weekly additions

## Integration Examples

### Using with LangChain

```python
from langchain.prompts import PromptTemplate
from pathlib import Path

def load_growth_prompt_as_langchain(prompt_name, category):
    """
    Load a Growth Marketing OS prompt as a LangChain PromptTemplate.
    """
    prompt_path = Path(f"growth-marketing-os/prompts/{category}/{prompt_name}.md")
    asset = parse_marketing_asset(prompt_path)
    
    # Extract the main prompt content (usually after first heading)
    content = asset['content']
    
    template = PromptTemplate(
        input_variables=asset['frontmatter'].get('variables', ['input']),
        template=content
    )
    
    return template

# Example
ad_copy_template = load_growth_prompt_as_langchain("meta-ad-framework", "paid-ads")
```

### Using with OpenAI API

```python
import os
import openai

openai.api_key = os.getenv("OPENAI_API_KEY")

def run_growth_prompt_with_openai(prompt_name, category, user_input):
    """
    Execute a Growth Marketing OS prompt using OpenAI API.
    """
    prompt_path = Path(f"growth-marketing-os/prompts/{category}/{prompt_name}.md")
    asset = parse_marketing_asset(prompt_path)
    
    system_prompt = asset['content']
    
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_input}
        ]
    )
    
    return response.choices[0].message.content

# Example
result = run_growth_prompt_with_openai(
    "seo-content-brief",
    "seo",
    "Create a content brief for 'best project management tools 2024'"
)
```

## Reference Links

- Repository: https://github.com/growthack88/growth-marketing-os
- Author: https://mahmoudomar.com
- YouTube: https://www.youtube.com/@GrowthHackAcademy
- Documentation: See `HOW-TO-USE.md` in repository root
- Contributing: See `CONTRIBUTING.md` for asset submission guidelines
