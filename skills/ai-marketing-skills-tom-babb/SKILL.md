---
name: ai-marketing-skills-tom-babb
description: Documented AI marketing workflows for content, distribution, brand systems, and campaign execution
triggers:
  - how do I use Tom Babb's AI marketing skills
  - show me the marketing skills workflows
  - how to scrape content for marketing
  - build a brand system with AI
  - create anti-slop prompts for content
  - set up AI video production workflow
  - implement GEO optimization strategy
  - use reverse prompting for images
---

# AI Marketing Skills by Tom Babb

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

`ai-marketing-skills-with-by-tom-babb` is a collection of battle-tested, repeatable AI marketing workflows built from real agency and in-house experience. Each skill is a documented process that collapses traditional multi-person marketing workflows into single-operator systems using AI tools.

The repository contains workflows for:
- **Distribution**: Content scraping, lead generation, and multi-format content pipelines
- **Brand**: Visual system creation and brand operating systems (prompts as brand guides)
- **Prompting**: Constraint-based prompting, anti-slop techniques, and reverse prompting
- **Content & Video**: AI video production pipelines and personal content engines
- **Paid Advertising**: Google Ads campaign generation workflows
- **Discoverability**: GEO (Generative Engine Optimization) strategies

## Installation

This is a documentation repository, not an installable package. Clone it to access the skills:

```bash
git clone https://github.com/Tom-Babb/ai-marketing-skills-with-by-tom-babb.git
cd ai-marketing-skills-with-by-tom-babb
```

Each skill lives in `/skills/<skill-name>/SKILL.md` with its own documentation, tools list, and step-by-step process.

## Repository Structure

```
/skills/
  /knowledge-scrape-to-content/
    SKILL.md
  /people-scrape-to-list/
    SKILL.md
  /long-form-to-content-pipeline/
    SKILL.md
  /brand-from-references/
    SKILL.md
  /brand-operating-system/
    SKILL.md
  /constraint-prompting/
    SKILL.md
  /anti-slop/
    SKILL.md
  /reverse-prompting-for-image-gen/
    SKILL.md
  /ai-video-production/
    SKILL.md
  /personal-content-engine/
    SKILL.md
  /reve-world-builder/
    SKILL.md
  /google-ads-with-ai/
    SKILL.md
  /music-distribution-strategy/
    SKILL.md
  /geo-optimization/
    SKILL.md
```

## Key Workflows

### Anti-Slop Prompting

The anti-slop workflow eliminates AI content patterns. Add this to the end of any content generation prompt:

```
ANTI-SLOP CONSTRAINTS:

1. SENTENCE OPENINGS: Do not start more than one sentence per paragraph with the same word. Vary structure.

2. CONTRAST FRAMING: Avoid "X, not Y" or "It's not about X, it's about Y" constructions unless absolutely necessary for clarity.

3. COLON TITLES: Do not use colons in titles or headlines unless listing items or introducing a direct quote.

4. BUZZWORD FILLER: Remove words like "ecosystem," "paradigm," "holistic," "synergy," "leverage," "robust," "revolutionary," "game-changing," "cutting-edge," "disruptive" unless they have specific technical meaning in context.

5. VAGUE CLAIMS: Every claim needs evidence. Replace "studies show" with the actual study. Replace "experts say" with the expert's name. Replace "significantly" with the number.

Output format: [your original format request]
```

**Usage Example:**

```
Write a 500-word blog post about email marketing automation for B2B SaaS companies.

ANTI-SLOP CONSTRAINTS:
[paste constraints above]

Output format: markdown with H2 sections
```

### Brand Operating System Prompt Template

Convert a brand guide into reusable AI prompts. Structure:

```markdown
# Brand Operating System for [Brand Name]

## Voice Prompt
Use this for all written content:

You are writing as [Brand Name]. 
Voice characteristics:
- [Trait 1]: [Specific example]
- [Trait 2]: [Specific example]
- [Trait 3]: [Specific example]

Never use: [specific words/phrases to avoid]
Always include: [specific elements that must appear]

## Design Prompt
Use this for all visual generation:

Create designs for [Brand Name] following this system:
- Color palette: [hex codes with usage rules]
- Typography: [font families, sizes, hierarchy]
- Layout principles: [spacing, alignment, grid]
- Visual style: [specific descriptors with reference examples]

## Messaging Prompt
Use this for positioning and key messages:

[Brand Name] positioning:
- Core message: [one sentence]
- Value propositions: [3-5 bullets]
- Proof points: [specific evidence]
- Differentiation: [vs. competitors]

## Identity Prompt
Use this for brand consistency checks:

[Brand Name] is: [3 adjectives]
[Brand Name] is not: [3 anti-adjectives]

We talk about: [topics list]
We do not talk about: [topics to avoid]
```

### Constraint Prompting Pattern

Every prompt should separate context from constraints:

```
CONTEXT:
[Everything the AI needs to understand the situation]
- Audience: [specific description]
- Goal: [measurable outcome]
- Format: [where this will be used]
- Prior work: [what's been tried]

CONSTRAINTS:
[Rules that prevent known failure modes]
- Length: [exact count or range]
- Structure: [required elements]
- Prohibitions: [what not to do]
- Requirements: [what must appear]
- Quality bar: [specific standard]

TASK:
[The actual request]
```

**Example:**

```
CONTEXT:
- Audience: B2B SaaS founders, 30-45, technical background, burning $50k+/month on paid ads
- Goal: 500 qualified demo bookings in 90 days
- Format: LinkedIn post, will be boosted with $2k budget
- Prior work: Last 3 posts got 2k impressions, 15 clicks, 0 conversions

CONSTRAINTS:
- Length: 150-200 words
- Structure: Hook + problem + insight + CTA, no bullets
- Prohibitions: No rhetorical questions in first sentence, no em dashes, no "imagine if"
- Requirements: Must include specific number or stat in hook, CTA must be single action
- Quality bar: Should read like a peer wrote it, not a marketer

TASK:
Write a LinkedIn post about CAC payback period optimization
```

### Reverse Prompting for Images

Extract prompts from existing images you want to replicate:

```python
# Conceptual workflow (requires vision model API access)
import anthropic
import os

client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

def reverse_prompt_image(image_path):
    """Extract detailed prompt from an image"""
    
    with open(image_path, "rb") as image_file:
        image_data = image_file.read()
    
    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "image",
                        "source": {
                            "type": "base64",
                            "media_type": "image/jpeg",
                            "data": image_data,
                        },
                    },
                    {
                        "type": "text",
                        "text": """Analyze this image and extract a detailed prompt that would recreate it.

Format your response as:

SUBJECT: [main subject with specific details]
SETTING: [environment, lighting, atmosphere]
COMPOSITION: [framing, angle, focus]
STYLE: [artistic style, render quality, visual treatment]
COLOR: [palette, mood, contrast]
TECHNICAL: [camera settings equivalent, depth of field]

Then provide a single consolidated prompt."""
                    }
                ],
            }
        ],
    )
    
    return message.content[0].text

# Usage
prompt = reverse_prompt_image("reference_image.jpg")
print(prompt)
```

### Knowledge Scrape to Content Workflow

Process for turning online discussions into ranked content:

1. **Scrape Sources** (Reddit, forums, YouTube comments):
```python
# Example using Reddit API
import praw
import os

reddit = praw.Reddit(
    client_id=os.environ.get("REDDIT_CLIENT_ID"),
    client_secret=os.environ.get("REDDIT_CLIENT_SECRET"),
    user_agent="marketing_scraper"
)

def scrape_pain_points(subreddit_name, keyword, limit=100):
    """Extract pain points from subreddit discussions"""
    subreddit = reddit.subreddit(subreddit_name)
    pain_points = []
    
    for submission in subreddit.search(keyword, limit=limit):
        pain_points.append({
            "title": submission.title,
            "body": submission.selftext,
            "score": submission.score,
            "num_comments": submission.num_comments,
            "url": submission.url
        })
        
        # Get top comments
        submission.comments.replace_more(limit=0)
        for comment in submission.comments.list()[:10]:
            pain_points.append({
                "title": f"Comment on: {submission.title}",
                "body": comment.body,
                "score": comment.score,
                "url": submission.url
            })
    
    return pain_points

# Example usage
pain_points = scrape_pain_points("saas", "customer churn", limit=50)
```

2. **Synthesize into content brief**:
```
Take these pain points and create a blog post outline:

[paste scraped pain points]

CONSTRAINTS:
- Title must match a real search query (provide 3 options with search volume estimate)
- H2s must address specific pain points found in the data
- Include exact quotes from the scrape (anonymized)
- Each section needs a specific solution, not generic advice
- End with a CTA that matches the reader's stage (awareness, not purchase)

Output: Title options, full outline with H2/H3 structure, key points for each section
```

### AI Video Production Pipeline

The correct order: **Prompt → Image → Video → Edit**

```bash
# Step 1: Generate base image with Reve (or Midjourney, Flux)
# Use detailed prompt from reverse prompting or original creation

# Step 2: Image-to-video with Veo 3 via Google Flow
# Upload image, specify camera movement and duration

# Step 3: Download and edit in Premiere Pro or DaVinci Resolve
# Add transitions, audio, text overlays

# Decision framework:
# Stop at image if: Static social post, thumbnail, presentation slide
# Stop at video if: Short-form content (<30s), no dialogue, simple message
# Go to edit if: Long-form (>30s), multiple scenes, audio sync required
```

**When to use each tool:**
- **Text-to-image**: Reve, Midjourney, Flux (cheapest, most controllable)
- **Image-to-video**: Veo 3, Runway Gen-3 (mid-price, consistent with source)
- **Text-to-video**: Only when you need motion that can't be keyframed (most expensive, least consistent)

### GEO (Generative Engine Optimization)

Optimize content for AI readers, not just human search:

**GEO Principles:**

1. **Authority**: State credentials and sources immediately
```markdown
This analysis uses data from [specific source with date], compiled by [role/credential].
```

2. **Structure**: Use semantic HTML and clear hierarchy
```html
<article>
  <h1>Primary Topic</h1>
  <section>
    <h2>Subtopic</h2>
    <p>Definition: [explicit definition]</p>
    <p>Context: [why it matters]</p>
    <p>Evidence: [specific data]</p>
  </section>
</article>
```

3. **Discoverability**: Include natural language queries as H2s
```markdown
## What is [concept]?
## How do I [action]?
## Why does [thing] happen?
## When should I [decision]?
```

4. **Conciseness**: State conclusions before explanations
```markdown
**Answer**: [Direct answer in first sentence]

**Explanation**: [Supporting detail]

**Source**: [Citation]
```

## Common Patterns

### Pattern: Multi-Format Content from Single Source

```
Input: 1 long-form piece (podcast, video, essay)
Output: 20-30 pieces across formats

Breakdown:
- 1 long-form blog post (2000+ words)
- 5-7 short blog posts (500 words each)
- 10-15 social posts (LinkedIn, X)
- 5-8 short-form videos (30-60s)
- 3-5 quote graphics
- 1 newsletter section

Process:
1. Transcribe source material
2. Extract key points, quotes, data, stories
3. Use constraint prompting to generate each format
4. Apply anti-slop constraints to all written content
5. Use reverse prompting for visual consistency
```

### Pattern: Personalized Outreach at Scale

```
Input: List of prospects with public content
Output: Personalized outreach messages

Process:
1. Scrape recent LinkedIn posts or X threads
2. Summarize each person's current focus
3. Generate personalized first line referencing their content
4. Attach to templated pitch

Example prompt:
CONTEXT:
- Prospect: [Name], [Title] at [Company]
- Recent post: [paste content]
- Our offer: [specific value prop]
- Goal: Book 15-min call

CONSTRAINTS:
- First sentence must reference their specific post with insight
- No generic compliments ("love your content")
- Transition must be natural, not forced
- CTA is single action (reply to book time)
- Length: 80-120 words total

TASK:
Write outreach email
```

### Pattern: Brand-Consistent Generation

```
# Always use two-prompt system:

# Prompt 1: Brand OS prompt (voice/design/messaging)
[Paste brand operating system prompt]

# Prompt 2: Specific task with constraints
Create [asset type] for [use case]

CONSTRAINTS:
[specific rules for this asset]

Ensure output follows the brand system above.
```

## Troubleshooting

**Problem**: Generated content sounds generic/sloppy
- **Solution**: Add anti-slop constraints. Check for repeated sentence openings, contrast framing, vague claims.

**Problem**: Brand inconsistency across outputs
- **Solution**: Build a brand operating system. Use the voice prompt at the start of every content generation request.

**Problem**: AI video output is inconsistent or low quality
- **Solution**: Don't use text-to-video. Use prompt → image → video pipeline. Start with Reve for consistent images, then animate with Veo 3.

**Problem**: Content isn't ranking or getting picked up by AI summaries
- **Solution**: Apply GEO principles. Add explicit definitions, state credentials, structure with semantic HTML, include natural language queries as headers.

**Problem**: Prompts are too long or complex
- **Solution**: Use constraint prompting pattern. Separate context (what AI needs to know) from constraints (rules to follow) from task (what to do).

**Problem**: Can't replicate a visual style
- **Solution**: Use reverse prompting. Feed the reference image to Claude with vision, extract detailed prompt, use that prompt in Reve or Midjourney.

## Additional Resources

- **Project Repository**: https://github.com/Tom-Babb/ai-marketing-skills-with-by-tom-babb
- **Author LinkedIn**: https://www.linkedin.com/in/tbabb02/
- **Author X**: https://x.com/TBabb02
- **Gauntlet AI**: https://www.gauntletai.com

## Key Tools Referenced in Skills

- **Claude**: Anthropic's LLM (vision, artifacts, long context)
- **Reve**: Image generation with persistent world-building
- **Veo 3**: Google's image-to-video model (via Google Flow)
- **Manus**: Marketing research and campaign generation
- **Pipeboard**: Direct integration to ad platforms
- **Reddit API**: Content and pain point scraping
- **YouTube API**: Comment and video analysis
- **Adobe Premiere**: Video editing
- **Google Ads API**: Campaign management

Each skill document contains specific tool setup and usage instructions.
