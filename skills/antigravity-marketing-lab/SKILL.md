---
name: antigravity-marketing-lab
description: Workshop project demonstrating Google Antigravity CLI for agent-first marketing workflows with local-marketing-agent skill for generating campaign headlines
triggers:
  - "how do I use Antigravity CLI for marketing content generation"
  - "create marketing headlines with Antigravity agent skills"
  - "set up local-marketing-agent skill for campaign briefs"
  - "generate localized marketing copy using Google Antigravity"
  - "use agent-first workflow for content creation"
  - "integrate AI agents into marketing website projects"
  - "build marketing briefs for Antigravity agents"
  - "troubleshoot Antigravity CLI agent tasks"
---

# Antigravity Marketing Lab

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a hands-on workshop demonstrating **Google Antigravity CLI** and **agent-first workflows** for marketing content generation. It includes a pre-built `local-marketing-agent` skill that generates localized marketing headlines from structured briefs, designed for events, products, or campaigns targeting Taiwan/Chinese-speaking audiences.

## What This Project Does

- Provides a complete Antigravity CLI workflow example for marketing tasks
- Includes a production-ready agent skill (`.agents/skills/local-marketing-agent/SKILL.md`)
- Demonstrates brief → agent → deliverable pipeline
- Shows multi-stage content generation (headline, LINE copy, Facebook posts)
- Serves as template for building custom marketing agent skills

## Installation

### Install Antigravity CLI

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
export PATH="$HOME/.local/bin:$PATH"
```

Verify installation:

```bash
agy --version
```

### Clone the Project

```bash
git clone https://github.com/jarsing/antigravity-marketing-lab.git
cd antigravity-marketing-lab
```

### Authenticate

On first run:

```bash
agy
```

Follow the prompts to authenticate with your Google account.

## Project Structure

```
.agents/
  skills/
    local-marketing-agent/
      SKILL.md              # Agent skill definition
challenge/
  my-brief.md             # Your marketing brief (user fills)
outputs/
  my-headline.md          # Generated headline output
website/
  index.html              # Demo integration site
```

## Key Concepts

### Agent Skills

Skills are markdown files in `.agents/skills/<skill-name>/SKILL.md` that define:
- Agent behavior and constraints
- Output format requirements
- Stage-by-stage workflows
- Language and tone guidelines

### Brief Format

The `challenge/my-brief.md` template:

```markdown
# Brief

- 宣傳主題：彰化在地美食節
- 目標對象：25-45歲在地居民與遊客
- 最大特色或亮點：超過50家老店首次聯合優惠
- 希望對方採取的行動：報名參加導覽、購買聯合票券
- 可以使用的地名或關鍵字：彰化、肉圓、八卦山、古蹟
- 不可以自行添加的資訊：不得捏造折扣金額、名額限制、截止日期
```

## Core Workflow

### 1. Prepare Your Brief

Edit `challenge/my-brief.md` with your campaign details:

```bash
# View current brief
cat challenge/my-brief.md

# Edit in Cloud Shell editor
cloudshell edit challenge/my-brief.md
```

### 2. Launch Antigravity CLI

```bash
cd antigravity-marketing-lab
agy
```

### 3. Execute Agent Task

In the Antigravity CLI, enter:

```text
請讀取 challenge/my-brief.md，
使用 local-marketing-agent Skill 執行「Stage 1：主標題產生」。

請自行檢查標題是否符合 Skill 規則；
若不符合，請修正後再交付。

最後只將通過檢查的最終主標題
儲存到 outputs/my-headline.md。
```

### 4. Review Output

```bash
cat outputs/my-headline.md
```

Expected format:

```markdown
# 主標題

「彰化美食節｜50家老店聯合優惠，跟著導覽吃遍古城」
```

## Local Marketing Agent Skill

### Viewing the Skill

```bash
cat .agents/skills/local-marketing-agent/SKILL.md
```

### Key Skill Features

- **Language**: Traditional Chinese (Taiwan)
- **Stage 1**: Main headline (≤24 Chinese characters)
- **Stage 2**: LINE push notification copy
- **Stage 3**: Facebook post opening
- **Constraints**: No fabricated discounts, dates, quotas, or results
- **Self-validation**: Agent checks compliance before delivery

### Example Skill Usage Pattern

```text
# Request specific stage
使用 local-marketing-agent 執行 Stage 2：LINE 推播文案

# Multi-stage request
依序執行 Stage 1-3，將所有輸出整合至 outputs/full-campaign.md

# Custom constraints
使用 local-marketing-agent，但將主標題限制改為 16 字以內
```

## Advanced Usage

### Check Available Skills

```bash
find .agents -name "SKILL.md" -type f
```

### Iterate on Output

In `agy` CLI:

```text
請改寫 outputs/my-headline.md 中的標題，
加強在地特色感，但仍需符合原 Skill 規則。
更新後儲存回同一檔案。
```

### Explain Integration Plan

```text
根據 outputs/my-headline.md 與 website/ 現有程式，
請只說明如果要把這句主標題整合進首頁 Hero 區塊，
預計需要修改哪些檔案與原因。

請不要實際修改檔案。
```

### Generate Multiple Variations

```text
使用 local-marketing-agent，
根據同一份 brief 產生 3 個不同風格的主標題變化，
儲存至 outputs/headline-variations.md
```

## Configuration

### Environment Variables

No API keys required for basic CLI usage. Authentication handled via Google OAuth.

For CI/CD or advanced setups:

```bash
export ANTIGRAVITY_PROJECT_ID=your-project-id
export ANTIGRAVITY_REGION=asia-east1
```

### Custom Skill Installation

To add your own marketing skill:

```bash
mkdir -p .agents/skills/my-custom-skill
# Create your SKILL.md with frontmatter and instructions
```

Example custom skill structure:

```markdown
---
name: email-subject-line-generator
version: 1.0
---

# Task
Generate email subject lines optimized for open rates...

# Rules
- Maximum 50 characters
- Include personalization token: {{name}}
- A/B test variants: 3 versions
...
```

## Common Patterns

### Pattern 1: Brief → Headline → Website Integration

```bash
# 1. Fill brief
cat > challenge/my-brief.md << 'EOF'
# Brief
- 宣傳主題：夏季限定咖啡新品
- 目標對象：18-35歲咖啡愛好者
EOF

# 2. Generate in agy
agy
# > 使用 local-marketing-agent 執行 Stage 1

# 3. Review
cat outputs/my-headline.md

# 4. Preview integration
grep -A 5 "<h1" website/index.html
```

### Pattern 2: Bulk Campaign Generation

```bash
# Generate all stages at once
agy
```

```text
請依序執行：
1. Stage 1 主標題 → outputs/headline.md
2. Stage 2 LINE 文案 → outputs/line-copy.md
3. Stage 3 Facebook 開頭 → outputs/fb-post.md

每階段完成後自我檢查符合 Skill 規則再繼續。
```

### Pattern 3: Localization Workflow

For multiple regions:

```markdown
# In my-brief.md
- 可以使用的地名或關鍵字：台中、彰化、南投
```

```text
使用 local-marketing-agent，
針對台中、彰化、南投三地各產生一版主標題，
分別儲存至 outputs/headline-taichung.md、
outputs/headline-changhua.md、
outputs/headline-nantou.md
```

## Troubleshooting

### Agent Doesn't Follow Skill Rules

**Symptom**: Generated headline exceeds character limit or invents information.

**Solution**: Add explicit validation step:

```text
請先產生主標題草稿，然後依 SKILL.md 第 X 條檢查：
1. 字數是否 ≤24
2. 是否捏造資訊
若不符合，請修正後再輸出至 outputs/
```

### Skill Not Found

**Symptom**: `Error: skill 'local-marketing-agent' not found`

**Solution**:

```bash
# Verify skill exists
find .agents -name "SKILL.md"

# Check you're in project root
pwd  # Should end with antigravity-marketing-lab

# Re-clone if needed
cd .. && rm -rf antigravity-marketing-lab
git clone https://github.com/jarsing/antigravity-marketing-lab.git
cd antigravity-marketing-lab
```

### CLI Not Persisting PATH

**Symptom**: `agy: command not found` after closing shell

**Solution**: Add to `~/.bashrc`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Output File Not Created

**Symptom**: `cat outputs/my-headline.md` returns "No such file"

**Solution**:

```bash
# Create outputs directory if missing
mkdir -p outputs

# Re-run agent task with explicit path
agy
# > 確保輸出檔案完整路徑為：$PWD/outputs/my-headline.md
```

### Brief in Wrong Format

**Symptom**: Agent produces generic output ignoring brief

**Solution**: Validate brief structure:

```bash
# Brief must have these exact fields:
grep -E "^- (宣傳主題|目標對象|最大特色|希望對方採取的行動)" challenge/my-brief.md

# Should return 4+ matches
```

### Rate Limits or Quota Issues

**Symptom**: `429 Too Many Requests` or quota errors

**Solution**:

- Wait 60 seconds between requests
- Use `--dry-run` flag for testing (if available)
- Share an account in workshop settings
- Simplify task to reduce token usage:

```text
# Instead of multi-stage:
使用 local-marketing-agent 只執行 Stage 1

# Instead of variations:
產生 1 個主標題（不要變化版本）
```

## Workshop Extensions

### After Completing Stage 1

Try these progressions:

```bash
# Stage 2: LINE copy
agy
# > 使用 local-marketing-agent 執行 Stage 2，儲存至 outputs/line-push.md

# Stage 3: Facebook post
agy
# > 使用 local-marketing-agent 執行 Stage 3，儲存至 outputs/fb-opening.md
```

### Modify Skill Constraints

```bash
cp .agents/skills/local-marketing-agent/SKILL.md .agents/skills/local-marketing-agent/SKILL.md.bak

# Edit SKILL.md to change character limits, tone, or rules
cloudshell edit .agents/skills/local-marketing-agent/SKILL.md
```

### Create New Skill

```bash
mkdir -p .agents/skills/travel-itinerary-generator
cat > .agents/skills/travel-itinerary-generator/SKILL.md << 'EOF'
---
name: travel-itinerary-generator
version: 1.0
---

# Task
Generate day-by-day travel itineraries for Taiwan destinations...
EOF
```

## Real-World Integration

### Export to Marketing Tools

```bash
# Copy headline for use in campaigns
cat outputs/my-headline.md | pbcopy  # macOS
cat outputs/my-headline.md | xclip   # Linux

# Export as JSON for API integration
cat > outputs/export.json << EOF
{
  "headline": "$(tail -1 outputs/my-headline.md | tr -d '「」')",
  "campaign_id": "${CAMPAIGN_ID}",
  "timestamp": "$(date -Iseconds)"
}
EOF
```

### Automated Campaign Workflow

```bash
#!/bin/bash
# campaign-generator.sh

for city in 台中 彰化 南投; do
  sed "s/彰化/$city/g" challenge/my-brief.md > challenge/brief-$city.md
  
  agy --non-interactive << EOF
請讀取 challenge/brief-$city.md
使用 local-marketing-agent 執行 Stage 1
儲存至 outputs/headline-$city.md
EOF
done
```

## References

- [Antigravity CLI Documentation](https://antigravity.google/cli)
- [Original Workshop Repository](https://github.com/jarsing/antigravity-marketing-lab)
- [GDG Changhua Build with AI 2026](https://gdg.community.dev/gdg-changhua/)
