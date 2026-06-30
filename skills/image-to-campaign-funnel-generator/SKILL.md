---
name: image-to-campaign-funnel-generator
description: Turn one product image into a complete German marketing funnel (landing page, 3 Instagram Reels, blog) from a single Brand Brief using Claude orchestration and Higgsfield CLI
triggers:
  - "create a complete marketing campaign from this product image"
  - "generate a landing page and Instagram reels from one photo"
  - "build a full German marketing funnel from this image"
  - "make me a campaign with landing page, reels, and blog"
  - "turn this product photo into a marketing funnel"
  - "generate consistent marketing assets from one image"
  - "create a brand brief and build all marketing materials"
  - "orchestrate a complete campaign from a single product picture"
---

# image-to-campaign-funnel-generator

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Orchestrates the Higgsfield CLI to generate a complete, brand-consistent German marketing funnel from **one product image**: a scroll-driven landing page, 3 Instagram Reels (9:16), and a styled blog article — all derived from a single approved Brand Brief.

## What it does

This is a **brief-first** orchestration skill that:

1. Conducts a short interview about the product
2. Writes a comprehensive Brand Brief (identity, thesis, OKLCH palette, fonts, tone, CTA, Reel angles)
3. Shows you the brief + all asset prompts for approval **before spending credits**
4. Builds landing page → 3 Reels → blog autonomously
5. Reuses the landing page hero video (reframed to 9:16) for one Reel to save costs
6. Runs browser QA on the landing page
7. Delivers everything in a structured campaign folder

## Prerequisites

**Required:**
- Higgsfield account with credits: https://higgsfield.ai/s/mcp-arnold-oberleiter-ecZvus
- Higgsfield CLI installed and authenticated (`higgsfield auth login`)
- Browser + static server (e.g. Python) for landing page preview
- Internet access for CDNs, fonts, and Higgsfield API

**Recommended companion skills:**
- `higgsfield-generate` — smarter model selection
- `higgsfield-product-photoshoot` — prompt enhancer
- `higgsfield-soul-id` — consistent presenter across Reels

## Installation

Clone into your skills directory:

```bash
# Project-level
git clone https://github.com/Arnie936/image-to-campaign .agents/skills/image-to-campaign

# User-level
git clone https://github.com/Arnie936/image-to-campaign ~/.claude/skills/image-to-campaign
```

Authenticate Higgsfield CLI:

```bash
higgsfield auth login
```

## Skill Structure

```
image-to-campaign/
  SKILL.md                  # Main orchestrator
  references/
    brand-brief.md          # Brief derivation + asset prompts
    landing-page.md         # Landing page spec + QA
    reels.md                # 3 Reel angles + caption format
    blog.md                 # German SEO article spec
  assets/
    brief-template.md
    blog-template.html
    campaign-readme-template.md
```

## Core Orchestration Flow

### Phase 1: Bootstrap & Interview

Ask up to 2 questions to gather:
- Product name and core benefit
- Target audience and key differentiation
- Optional: brand voice preference (formal/casual)

### Phase 2: Brand Brief Generation

Create `brief.md` containing:

```markdown
# Brand Brief: [Product]

## Product Identity
- Name: [...]
- Category: [...]
- Core Benefit: [...]
- Target Audience: [...]

## Visual Identity
### Color Palette (OKLCH)
- Primary: oklch(...)
- Secondary: oklch(...)
- Accent: oklch(...)

### Typography
- Headline: [Google Font]
- Body: [Google Font]

## Verbal Identity
- Thesis Line: [single compelling statement]
- Tone: [formal/casual/balanced]
- Key Messages: [3 bullets]

## Call-to-Action
- Primary CTA: [text]
- URL: [demo or real]

## Asset Specifications

### Landing Page
- Hero: [video prompt]
- Middle: [video prompt]
- Gallery: [3 image prompts]

### Instagram Reels (9:16)
1. Hook: [prompt + caption angle]
2. Showcase: [prompt + caption angle]
3. Lifestyle: [prompt + caption angle]

### Blog Article
- SEO Title: [...]
- Focus Keywords: [...]
- Structure: [H2 outline]
```

### Phase 3: Checkpoint

Display the brief and list all asset prompts (hero video, middle video, 3 gallery images, 3 Reels, blog images). **Wait for explicit user approval** before proceeding.

### Phase 4: Build Assets

#### Landing Page First

Generate folder structure:

```bash
mkdir -p kampagne-{slug}/website/assets
mkdir -p kampagne-{slug}/reels
mkdir -p kampagne-{slug}/blog/assets
```

Generate assets using Higgsfield CLI:

```bash
# Hero video (16:9)
higgsfield generate video \
  --prompt "[hero prompt from brief]" \
  --aspect-ratio "16:9" \
  --duration 5 \
  --output kampagne-{slug}/website/assets/hero.mp4

# Middle video (16:9)
higgsfield generate video \
  --prompt "[middle prompt from brief]" \
  --aspect-ratio "16:9" \
  --duration 5 \
  --output kampagne-{slug}/website/assets/middle.mp4

# Gallery images (1:1)
higgsfield generate image \
  --prompt "[gallery-1 prompt]" \
  --aspect-ratio "1:1" \
  --output kampagne-{slug}/website/assets/g1.jpg
```

Build `index.html`:

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Product] – [Thesis Line]</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=[Headline-Font]&family=[Body-Font]&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/ScrollTrigger.min.js"></script>
  <style>
    :root {
      --primary: [oklch from brief];
      --secondary: [oklch from brief];
      --accent: [oklch from brief];
      --headline: '[Headline Font]', sans-serif;
      --body: '[Body Font]', sans-serif;
    }
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: var(--body); background: var(--primary); color: white; }
    
    .hero {
      height: 100vh;
      position: relative;
      overflow: hidden;
    }
    .hero video {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    .hero-overlay {
      position: absolute;
      inset: 0;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      background: linear-gradient(transparent 40%, rgba(0,0,0,0.7));
      text-align: center;
      padding: 2rem;
    }
    .hero h1 {
      font-family: var(--headline);
      font-size: clamp(2rem, 6vw, 4rem);
      margin-bottom: 1rem;
    }
    .hero p {
      font-size: clamp(1rem, 2vw, 1.5rem);
      margin-bottom: 2rem;
      max-width: 600px;
    }
    .cta {
      background: var(--accent);
      color: white;
      padding: 1rem 2rem;
      border: none;
      border-radius: 2rem;
      font-size: 1.1rem;
      font-weight: bold;
      cursor: pointer;
      text-decoration: none;
      display: inline-block;
    }
    
    .section {
      min-height: 100vh;
      padding: 4rem 2rem;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .benefits { background: var(--secondary); }
    .showcase { background: var(--primary); }
    .gallery {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 2rem;
      max-width: 1200px;
    }
    .gallery img {
      width: 100%;
      border-radius: 1rem;
    }
  </style>
</head>
<body>
  <section class="hero">
    <video autoplay muted loop playsinline>
      <source src="assets/hero.mp4" type="video/mp4">
    </video>
    <div class="hero-overlay">
      <h1>[Product Name]</h1>
      <p>[Thesis Line]</p>
      <a href="[CTA URL]" class="cta">[CTA Text]</a>
    </div>
  </section>

  <section class="section benefits">
    <div>
      <h2>[Key Benefit 1]</h2>
      <p>[Explanation from brief]</p>
    </div>
  </section>

  <section class="section showcase">
    <video autoplay muted loop playsinline style="max-width: 800px; width: 100%; border-radius: 1rem;">
      <source src="assets/middle.mp4" type="video/mp4">
    </video>
  </section>

  <section class="section">
    <div class="gallery">
      <img src="assets/g1.jpg" alt="[Gallery 1 alt]">
      <img src="assets/g2.jpg" alt="[Gallery 2 alt]">
      <img src="assets/g3.jpg" alt="[Gallery 3 alt]">
    </div>
  </section>

  <script>
    gsap.registerPlugin(ScrollTrigger);
    gsap.utils.toArray('.section').forEach(section => {
      gsap.from(section.children, {
        opacity: 0,
        y: 50,
        duration: 1,
        scrollTrigger: {
          trigger: section,
          start: "top 80%",
          toggleActions: "play none none reverse"
        }
      });
    });
  </script>
</body>
</html>
```

#### Browser QA

Start local server and verify:

```bash
cd kampagne-{slug}/website
python -m http.server 8000
# Open http://localhost:8000 and verify:
# - Videos load and play
# - Fonts render correctly
# - Colors match brief
# - Scroll animations work
# - CTA is clickable
```

#### Generate Reels

**Reel 1** (Hook) — new generation:

```bash
higgsfield generate video \
  --prompt "[reel-1 hook prompt from brief]" \
  --aspect-ratio "9:16" \
  --duration 7 \
  --output kampagne-{slug}/reels/reel-1-hook.mp4
```

**Reel 2** (Showcase) — **reuse hero video** reframed:

```bash
higgsfield reframe \
  --input kampagne-{slug}/website/assets/hero.mp4 \
  --aspect-ratio "9:16" \
  --output kampagne-{slug}/reels/reel-2-showcase.mp4
```

**Reel 3** (Lifestyle) — new generation:

```bash
higgsfield generate video \
  --prompt "[reel-3 lifestyle prompt from brief]" \
  --aspect-ratio "9:16" \
  --duration 7 \
  --output kampagne-{slug}/reels/reel-3-lifestyle.mp4
```

Create `captions.md`:

```markdown
# Instagram Reel Captions

## Reel 1: Hook
**Hook Text (first 2 sec):** "[attention-grabbing statement]"

**Caption:**
[Engaging first line with emoji]

[Core benefit explanation]

[Call to curiosity or action]

**Hashtags:** #[product] #[category] #[benefit1] #[benefit2] #[targetAudience]

**CTA:** [CTA text] → [URL]

---

## Reel 2: Showcase
[Same structure for Reel 2]

---

## Reel 3: Lifestyle
[Same structure for Reel 3]
```

#### Generate Blog

```bash
higgsfield generate image \
  --prompt "[blog hero image prompt from brief]" \
  --aspect-ratio "16:9" \
  --output kampagne-{slug}/blog/assets/hero.jpg
```

Create `blog.html` using `blog-template.html`:

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[SEO Title from brief]</title>
  <meta name="description" content="[Meta description with focus keywords]">
  <link href="https://fonts.googleapis.com/css2?family=[Body-Font]&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: '[Body Font]', sans-serif;
      line-height: 1.7;
      max-width: 800px;
      margin: 0 auto;
      padding: 2rem;
      color: #333;
    }
    h1 { font-size: 2.5rem; margin-bottom: 1rem; }
    h2 { font-size: 1.8rem; margin-top: 2rem; margin-bottom: 1rem; color: var(--primary); }
    img { width: 100%; border-radius: 1rem; margin: 2rem 0; }
    .meta { color: #777; margin-bottom: 2rem; }
  </style>
</head>
<body>
  <article>
    <h1>[SEO Title]</h1>
    <div class="meta">Von [Brand] | [Date]</div>
    <img src="assets/hero.jpg" alt="[Alt text with keywords]">
    
    <p>[Introduction paragraph with focus keyword in first 100 words]</p>
    
    <h2>[H2 from outline]</h2>
    <p>[Content derived from brief's key messages]</p>
    
    <h2>[H2 from outline]</h2>
    <p>[Content]</p>
    
    <h2>Fazit</h2>
    <p>[Conclusion with CTA: [CTA Text] unter [URL]]</p>
  </article>
</body>
</html>
```

### Phase 5: Deliver

Copy `brief.md` to campaign root and create `README.md`:

```markdown
# Kampagne: [Product Name]

Generiert am [Date]

## Struktur

- `brief.md` – Die genehmigte Brand Brief
- `website/` – Landing Page (index.html + assets/)
- `reels/` – 3 Instagram Reels + captions.md
- `blog/` – Blog-Artikel (blog.html + assets/)

## Landing Page lokal anschauen

```bash
cd website
python -m http.server 8000
# Öffne http://localhost:8000
```

## Reels hochladen

1. Datei auf Handy übertragen
2. In Instagram als Reel hochladen
3. Caption aus `reels/captions.md` kopieren

## Blog deployen

`blog/blog.html` auf deinen Webserver hochladen oder in dein CMS integrieren.
```

Output final summary:

```
✅ Kampagne erstellt: kampagne-[slug]/

📁 Inhalt:
  - Landing Page: 1 HTML + 5 Assets (2 Videos, 3 Bilder)
  - Instagram Reels: 3 Videos (9:16) + Captions
  - Blog: 1 HTML + Hero-Bild

🌐 Landing Page lokal testen:
  cd kampagne-[slug]/website && python -m http.server 8000

📊 Higgsfield Credits verbraucht: ~[estimate based on generations]
```

## Configuration & Customization

### Locked Defaults (change in SKILL.md if needed)

```markdown
## Locked defaults

- **Language:** German (de)
- **CTA default:** "Jetzt Demo anfordern" → https://example.com/demo
- **Reel duration:** 7 seconds (hook/lifestyle), 5 seconds (showcase reframe)
- **Color space:** OKLCH for perceptual uniformity
- **Landing page animations:** GSAP + ScrollTrigger from CDN
```

### Brief Template Customization

Edit `assets/brief-template.md` to change:
- Color palette structure (add tertiary, neutral colors)
- Verbal identity fields (add brand values, tagline)
- Asset prompt sections (add email header, social cards)

### Landing Page Template

Edit the HTML structure in `references/landing-page.md`:
- Add navigation bar
- Add footer with social links
- Change grid layout for gallery
- Swap GSAP for Framer Motion (if building React version)

### Reel Angles

Edit `references/reels.md` to change the 3 default angles:
- Hook (problem/curiosity) → Feature (benefit demo) → Testimonial
- Day-in-life → Quick tip → Product showcase
- Before/After → Process → Result

## Higgsfield CLI Commands Reference

### Generate Video

```bash
higgsfield generate video \
  --prompt "Cinematic shot of [product], [scene details]" \
  --aspect-ratio "16:9" \
  --duration 5 \
  --model "runway-gen3" \
  --output path/to/output.mp4
```

**Aspect ratios:** `16:9` (landscape), `9:16` (portrait), `1:1` (square)  
**Duration:** 5 or 10 seconds  
**Models:** `runway-gen3`, `kling-1.5`, `minimax-video-01` (auto-selected by `higgsfield-generate` skill)

### Generate Image

```bash
higgsfield generate image \
  --prompt "Product photography of [product] on [background]" \
  --aspect-ratio "1:1" \
  --model "flux-1.1-pro" \
  --output path/to/output.jpg
```

**Aspect ratios:** `1:1`, `16:9`, `9:16`, `4:3`, `3:4`  
**Models:** `flux-1.1-pro`, `recraft-v3`, `ideogram-v2`

### Reframe Video

```bash
higgsfield reframe \
  --input path/to/16-9-video.mp4 \
  --aspect-ratio "9:16" \
  --output path/to/9-16-video.mp4
```

Intelligently crops/pans to new aspect ratio while keeping subject in frame.

### Check Credits

```bash
higgsfield credits
```

Returns remaining credits for image and video generations.

## Troubleshooting

### "Higgsfield CLI not found"

Install manually:

```bash
curl -sSL https://install.higgsfield.ai/install.sh | sh
higgsfield auth login
```

On Windows, run the installer in Git Bash or WSL.

### "Generation failed: insufficient credits"

Check credits:

```bash
higgsfield credits
```

Top up at https://higgsfield.ai/settings/billing

### Landing page fonts not loading

Ensure internet connection (fonts from Google Fonts CDN). If deploying offline, download fonts and update CSS:

```css
@font-face {
  font-family: 'CustomHeadline';
  src: url('./fonts/headline.woff2') format('woff2');
}
```

### Videos not playing in browser

- Check video file exists: `ls kampagne-*/website/assets/`
- Verify MIME type: server must serve `.mp4` as `video/mp4`
- Test in different browser (Safari sometimes needs `playsinline` attribute)

### Reel 2 (reframe) looks cropped wrong

The reframe algorithm centers on detected subjects. If output is off:

```bash
# Regenerate instead of reframing
higgsfield generate video \
  --prompt "[same prompt as hero but vertical composition]" \
  --aspect-ratio "9:16" \
  --duration 5 \
  --output kampagne-{slug}/reels/reel-2-showcase.mp4
```

### Brief checkpoint interrupted

Resume by pointing to existing brief:

```bash
# In orchestrator logic:
if exists("kampagne-{slug}/brief.md"):
    brief = read("kampagne-{slug}/brief.md")
    skip to Phase 4 (build assets)
```

## Real-World Usage Patterns

### Pattern 1: E-commerce Product Launch

```
User: "Create campaign for this sneaker image: ./sneaker.jpg"
→ Interview: target (fashion-conscious 18-30), benefit (comfort + style)
→ Brief: bold OKLCH palette, modern sans-serif, casual tone
→ Landing page: hero = product spin, middle = wear test, gallery = 3 colorways
→ Reels: 1) Unboxing hook, 2) Styling showcase, 3) Street lifestyle
→ Blog: "Die perfekten Sneaker für Stadt und Freizeit"
```

### Pattern 2: SaaS Tool

```
User: "Build funnel for this dashboard screenshot: ./app.png"
→ Interview: B2B audience (marketing managers), benefit (save 10h/week)
→ Brief: professional blue palette, clean sans-serif, formal tone
→ Landing page: hero = UI walkthrough, middle = data viz animation, gallery = 3 features
→ Reels: 1) "Verbringst du zu viel Zeit mit Reports?", 2) Feature demo, 3) Customer testimonial
→ Blog: "Wie Marketing-Teams mit [Tool] Zeit sparen"
```

### Pattern 3: Food Product

```
User: "Make campaign for this organic coffee bag: ./coffee.jpg"
→ Interview: health-conscious consumers, benefit (organic + fair trade)
→ Brief: earthy green/brown palette, organic serif, warm tone
→ Landing page: hero = brewing closeup, middle = farm origin, gallery = 3 serving suggestions
→ Reels: 1) Morning ritual hook, 2) Farm-to-cup story, 3) Latte art lifestyle
→ Blog: "Warum Bio-Kaffee besser schmeckt"
```

## Integration with Other Skills

### With `higgsfield-product-photoshoot`

Enhance product image prompts before generation:

```markdown
# In brief asset prompts, add:
"Use higgsfield-product-photoshoot to enhance this prompt before generating."
```

The photoshoot skill adds professional lighting, composition, and material rendering guidance.

### With `higgsfield-soul-id`

Keep the same presenter across all 3 Reels:

```bash
# Generate Reel 1 with UGC presenter
higgsfield generate video --prompt "[...] featuring a young woman with [description]" --output reel-1.mp4

# Extract Soul ID
higgsfield soul-id extract --input reel-1.mp4 --output presenter.soul

# Apply to Reel 2 and 3
higgsfield generate video --prompt "[reel-2 prompt]" --soul-id presenter.soul --output reel-2.mp4
higgsfield generate video --prompt "[reel-3 prompt]" --soul-id presenter.soul --output reel-3.mp4
```

### With `higgsfield-generate`

Automatic model selection based on prompt:

```bash
# Instead of --model flag, let higgsfield-generate choose:
higgsfield generate video --prompt "[complex cinematic shot]"
# → selects runway-gen3 for motion quality

higgsfield generate video --prompt "[UGC style selfie video]"
# → selects kling-1.5 for speed + cost
```

## Best Practices

1. **Always approve the brief checkpoint** — once assets are generated, credits are spent.

2. **Reuse the landing page hero** — the reframe to 9:16 for Reel 2 saves one full video generation.

3. **Test locally before deploying** — browser QA catches font/video issues before going live.

4. **Keep prompts brief-consistent** — reference the brief's visual identity in every asset prompt (colors, style, mood).

5. **Version your briefs** — if iterating, save `brief-v1.md`, `brief-v2.md` to track what changed.

6. **Use environment variables for real CTAs**:

```bash
export CAMPAIGN_CTA_URL="https://shop.example.com/product"
export CAMPAIGN_CTA_TEXT="Jetzt kaufen"
```

Then reference in brief template: `[CTA URL]` → `${CAMPAIGN_CTA_URL}`

## Output Structure

```
kampagne-product-name/
├── brief.md                    # Single source of truth
├── README.md                   # Usage instructions
├── website/
│   ├── index.html
│   └── assets/
│       ├── hero.mp4           # 16:9, 5s
│       ├── middle.mp4         # 16:9, 5s
│       ├── g1.jpg             # 1:1
│       ├── g2.jpg             # 1:1
│       └── g3.jpg             # 1:1
├── reels/
│   ├── reel-1-hook.mp4        # 9:16, 7s, new generation
│   ├── reel-2-showcase.mp4    # 9:16, 5s, reframed from hero
│   ├── reel-3-lifestyle.mp4   # 9:16, 7s, new generation
│   └── captions.md            # Hook, caption, hashtags, CTA per Reel
└── blog/
    ├── blog.html
    └── assets/
        └── hero.jpg           # 16:9
```

**Total Higgsfield cost per campaign:**
- 4 video generations (2 landing, 2 reels) × ~50 credits = 200 credits
- 1 video reframe (reel 2) × ~10 credits = 10 credits
- 4 image generations (3 gallery + 1 blog) × ~5 credits = 20 credits
- **Total: ~230 credits**

## Related Skills

- `higgsfield-generate` — smart model selection
- `higgsfield-product-photoshoot` — prompt enhancement for product visuals
- `higgsfield-soul-id` — consistent presenter identity across videos
- `higgsfield-video-editor` — post-generation trimming, speed adjustments
- `german-seo-copywriter` — SEO-optimized blog content expansion

## Support

For Higgsfield CLI issues: https://docs.higgsfield.ai  
For skill bugs/features: https://github.com/Arnie936/image-to-campaign/issues

---

**Next steps after campaign generation:**

1. `cd kampagne-{slug}/website && python -m http.server` — QA landing page
2. Transfer Reels to phone → upload to Instagram with captions from `reels/captions.md`
3. Deploy `blog/blog.html` to your site or integrate into CMS
4. Track conversions and iterate on brief for next campaign
