---
name: cohort-de-marketing-claude-skills
description: Marketing research and offer creation toolkit with 6 Claude Code skills for avatar research, competitor analysis, trend hunting, swipe files, and offer creation.
triggers:
  - how do I use the cohort de marketing skills
  - setup cohort de marketing research tools
  - run avatar research with claude code
  - create competitor analysis with espiao skill
  - generate offerbook with marketing cohort
  - use academia lendaria marketing skills
  - marketing research skills claude code
  - build offer with cohort de marketing
---

# Cohort de Marketing Skills

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Marketing research and offer creation toolkit built by Academia Lendária. Provides 6 Claude Code skills for avatar research, competitor intelligence, trend hunting, swipe file organization, and offer creation. Clone-and-run system that loads skills automatically into Claude Code.

## Installation

```bash
git clone https://github.com/marketingLendario/cohort-de-marketing.git
cd cohort-de-marketing
```

Copy environment template for optional API keys:

```bash
cp .env.example .env
```

Open in Claude Code:

```bash
claude
```

Skills in `.claude/skills/` load automatically. Type `/` to see available skills.

## Core Philosophy

**Research → Offer → Copy → Ads** (never skip this order)

- Pesquisa antes da oferta (research before offer)
- Voz do cliente sempre verbatim (customer voice always verbatim)
- Mark assumptions as `[SUPOSIÇÃO]` when no real data exists
- Offerbook approval before any copy/LP/ads

## The 6 Skills (Aula 01)

### `/avatar-funil`

Avatar research across 7 dimensions + synthetic focus group. Outputs MD + HTML + PDF.

**Usage in Claude Code:**

```
/avatar-funil

Produto: Curso de copywriting para coaches
Mercado: Coaches de vida e carreira Brasil
Público: Mulheres 30-45 anos com negócio online
```

**Dimensions covered:**
- Demographics
- Psychographics
- Pain points & desires
- Current solutions & frustrations
- Buying triggers
- Objections
- Language patterns (verbatim phrases)

**Output:** `avatar-[produto].md`, `avatar-[produto].html`, `avatar-[produto].pdf`

### `/espiao-do-concorrente`

Multi-source competitor intelligence dossier.

**Usage:**

```
/espiao-do-concorrente

Concorrente: [Nome ou URL]
Fontes: Meta Ad Library, Google Ads, Instagram, site, reviews
```

**Sources analyzed:**
- Meta Ad Library (active ads)
- Google Ads (search ads)
- Social media presence
- Website copy & structure
- Customer reviews
- Pricing & positioning

**Output:** `espiao-[concorrente].md` with structured intel

### `/trend-hunting`

Identify emerging trends before saturation across 4 sources.

**Usage:**

```
/trend-hunting

Nicho: Marketing digital para psicólogos
Período: últimos 3 meses
```

**Sources:**
- Google Trends
- Reddit discussions
- Twitter/X conversations
- Industry publications

**Output:** `trends-[nicho]-[data].md` with trend score and timing

### `/swipe-file`

Organize winning creatives by type/format/pattern.

**Usage:**

```
/swipe-file

Tipo: Ads do Facebook
Nicho: Infoprodutos
Padrão: Problem-Agitate-Solve
```

**Categories:**
- Headlines
- Hooks (video/carousel)
- Body copy patterns
- CTA formulas
- Visual patterns

**Output:** `swipe-[tipo]-[data].md` with categorized examples

### `/offerbook`

The Offer Book in 7 blocks. Outputs MD + DOCX using official template.

**Usage:**

```
/offerbook

Produto: [nome do produto]
Avatar: [referência ao avatar-funil gerado]
Concorrentes: [referência ao espiao gerado]
```

**7 Blocks:**
1. **Contexto** - Market positioning
2. **Avatar** - Who buys (from avatar research)
3. **Problema** - Pain landscape
4. **Solução** - Your mechanism
5. **Oferta** - What they get (stack)
6. **Garantia** - Risk reversal
7. **Preço** - Pricing strategy & logic

**Output:** 
- `offerbook-[produto].md`
- `offerbook-[produto].docx` (uses `templates/Template-Offerbook.docx`)

### `/design-md`

Generate DESIGN.md with brand identity (logo, colors, fonts). Used by `/avatar-funil` and `/offerbook` to render HTML with your branding.

**Usage:**

```
/design-md

Brand: Nome da Marca
Cores primárias: #HEX1, #HEX2
Fonte principal: Font Name
Logo: [URL ou caminho]
```

**Optional:** If not run, outputs default to Academia Lendária branding.

**Output:** `DESIGN.md` at project root

## Configuration

Optional API keys in `.env`:

```bash
# Meta/Facebook (for Ad Library access)
META_ACCESS_TOKEN=your_token_here

# Google Ads (for competitor ad intel)
GOOGLE_ADS_DEVELOPER_TOKEN=your_token_here
GOOGLE_ADS_CLIENT_ID=your_client_id
GOOGLE_ADS_CLIENT_SECRET=your_secret

# OpenAI (for enhanced synthesis)
OPENAI_API_KEY=your_key_here

# Serper (for search/trend data)
SERPER_API_KEY=your_key_here
```

All keys are **optional**. Skills work in manual mode without keys (you provide data manually).

## Typical Workflow

### 1. Research Phase

```bash
# Start with avatar research
/avatar-funil

# Then competitor intel (run for 2-3 competitors)
/espiao-do-concorrente

# Check trends
/trend-hunting

# Gather swipes
/swipe-file
```

### 2. Offer Creation Phase

```bash
# Optional: Set your brand
/design-md

# Create the offer book
/offerbook
```

### 3. Approval & Handoff

Review `offerbook-[produto].docx` with stakeholder. **No LP, email, or ads until offerbook is approved.**

## Project Structure

```
cohort-de-marketing/
├── .claude/
│   └── skills/              # 6 skills (Aula 01)
│       ├── avatar-funil.md
│       ├── espiao-do-concorrente.md
│       ├── trend-hunting.md
│       ├── swipe-file.md
│       ├── offerbook.md
│       └── design-md.md
├── aula-01/
│   ├── GUIA-DO-ALUNO.html   # Start here (visual guide)
│   ├── docs/
│   │   ├── SKILLS-INDEX.md  # Name aliases (old → canonical)
│   │   ├── workflow.md
│   │   └── handoff.md
│   └── templates/
│       └── Template-Offerbook.docx
├── .env.example
└── README.md
```

## Real Code Examples

### Calling a skill programmatically (if building automation)

Skills are markdown-based and designed for Claude Code's slash command interface, but can be invoked programmatically:

```javascript
// Example: Parse avatar research output
const fs = require('fs');
const matter = require('gray-matter');

function parseAvatarResearch(filepath) {
  const content = fs.readFileSync(filepath, 'utf8');
  const { data, content: markdown } = matter(content);
  
  // Extract verbatim quotes (customer voice)
  const quotes = markdown.match(/> "(.+?)"/g) || [];
  const cleanQuotes = quotes.map(q => q.replace(/> "/g, '').replace(/"/g, ''));
  
  return {
    meta: data,
    verbatimQuotes: cleanQuotes,
    rawContent: markdown
  };
}

const avatar = parseAvatarResearch('./avatar-curso-copy.md');
console.log(`Found ${avatar.verbatimQuotes.length} customer voice quotes`);
```

### Integrating with Offerbook generation

```javascript
// Example: Pass avatar data to offerbook
const avatarData = parseAvatarResearch('./avatar-produto.md');
const competitorData = JSON.parse(fs.readFileSync('./espiao-data.json', 'utf8'));

// Use in prompt for /offerbook
const offerPrompt = `
/offerbook

Produto: ${productName}
Avatar: ${avatarData.meta.targetAudience}
Pain points principais: ${avatarData.verbatimQuotes.slice(0, 3).join(', ')}
Concorrentes: ${competitorData.competitors.map(c => c.name).join(', ')}
`;
```

## Common Patterns

### Pattern: Research → Synthesis → Offer

```bash
# 1. Gather all research
/avatar-funil
/espiao-do-concorrente  # Run 2-3x for different competitors
/trend-hunting

# 2. Review outputs, highlight key insights

# 3. Create offer
/offerbook  # Reference the research files
```

### Pattern: Verbatim Customer Voice

Always include literal quotes in avatar research. Mark assumptions:

```markdown
## Linguagem do Avatar

### Verbatim (real quotes)
> "Eu não sei como cobrar sem parecer caro"
> "Meus posts não convertem, só dão like"

### [SUPOSIÇÃO] - Needs validation
- Provavelmente usa Instagram como canal principal
```

### Pattern: Competitor Angle Gap

```markdown
## Brechas de Ângulo

| Concorrente | Ângulo | Brecha |
|-------------|---------|--------|
| Concorrente A | "Venda todo dia" | Não fala de posicionamento |
| Concorrente B | "Copy que converte" | Ignora tráfego/audiência |
| **Nossa brecha** | "Posicionamento → Copy → Audiência" | Sistema completo |
```

## Troubleshooting

### Skills don't appear in Claude Code

```bash
# Check skills are in correct location
ls -la .claude/skills/

# Should see:
# avatar-funil.md
# espiao-do-concorrente.md
# trend-hunting.md
# swipe-file.md
# offerbook.md
# design-md.md

# Restart Claude Code
```

### Old skill names (aliases)

Check `aula-01/docs/SKILLS-INDEX.md` for name mappings:

```markdown
# Old name → Current name
/pesquisa-avatar → /avatar-funil
/analise-concorrente → /espiao-do-concorrente
/criar-oferta → /offerbook
```

### DOCX generation fails

Ensure `templates/Template-Offerbook.docx` exists:

```bash
ls -la aula-01/templates/Template-Offerbook.docx
```

### Missing API keys

Skills work without keys in **manual mode**. You provide data when prompted:

```
/espiao-do-concorrente

[Sem META_ACCESS_TOKEN configurado]
Por favor, cole manualmente os ads do Meta Ad Library para [Concorrente]:
```

### Offerbook has no customer quotes

You skipped `/avatar-funil`. **Research before offer** (rule #1). Run avatar research first, then reference in offerbook.

## Skill Name Reference

Canonical names (use these):

- `/avatar-funil` - Avatar research
- `/espiao-do-concorrente` - Competitor intel
- `/trend-hunting` - Trend identification
- `/swipe-file` - Creative organization
- `/offerbook` - Offer book creation
- `/design-md` - Brand identity setup

## Support

- **Technical issues:** Open issue at https://github.com/marketingLendario/cohort-de-marketing
- **Skill name confusion:** Check `aula-01/docs/SKILLS-INDEX.md`
- **Content questions:** Cohort community channel

## Future Aulas

- **Aula 2:** Funil e Páginas (funnel structure, landing pages, creative batches)
- **Aula 3:** Tráfego e Criativos (traffic, ads, creative testing)
- **Aula 4:** Dados e Receita (analytics, revenue optimization)

Skills publish week of each aula.

---

Built by Academia Lendária with Claude Code.
