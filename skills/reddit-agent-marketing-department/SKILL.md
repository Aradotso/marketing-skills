---
name: reddit-agent-marketing-department
description: Matrix department template for automated Reddit growth, community marketing, and lead scouting with AI agents
triggers:
  - "set up Reddit marketing automation"
  - "create a Reddit growth department"
  - "automate Reddit community research"
  - "build Reddit lead generation system"
  - "deploy AI agents for Reddit marketing"
  - "install Reddit marketing template"
  - "configure Matrix department for Reddit"
  - "set up automated Reddit outreach"
---

# Reddit Agent Marketing Department

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A Matrix department template that provides automated Reddit growth, community marketing, and lead scouting capabilities through AI agents. Built for the Matrix AI Company OS, this template includes research skills, copywriting workflows, anti-AI-slop review, and safety gates for compliant Reddit marketing.

## What It Does

- **Subreddit Research**: Discover and evaluate communities using Reddit's public JSON API
- **Community-Fit Analysis**: Score subreddits by relevance, rules, and promotion risk
- **Lead Scouting**: Find prospects asking questions your product can answer
- **Reddit-Native Drafting**: Generate posts, comments, and DMs that sound human
- **Anti-AI-Slop Review**: Strip telltale AI phrasing before publishing
- **Launch Seeding Plans**: Create staggered posting schedules with cooldown management
- **Approval-Gated Publishing**: All public actions require explicit human approval
- **Outcome Logging**: Track published posts, engagement, and results

## Installation

### Import into Matrix

1. Clone the repository:
```bash
git clone https://github.com/Weike/reddit-agent-marketing-department.git
cd reddit-agent-marketing-department
```

2. In [Matrix](https://matrix.build), create a new department or navigate to an existing one

3. Copy the department structure into your Matrix workspace:
```bash
# Copy skills
cp -r skills/ /path/to/matrix/department/skills/

# Copy memory
cp -r memory/ /path/to/matrix/department/memory/

# Copy rules
cp RULES.md /path/to/matrix/department/RULES.md
```

4. Connect Reddit integration in Matrix (for publishing actions)

### Standalone Setup

Use as a reference template without Matrix:

```bash
# Clone and explore
git clone https://github.com/Weike/reddit-agent-marketing-department.git
cd reddit-agent-marketing-department

# Review structure
tree skills/ memory/
```

## Department Structure

```
reddit-agent-marketing-department/
├── RULES.md                              # Department operating rules
├── skills/
│   ├── reddit/                           # Public Reddit JSON API
│   ├── reddit-growth-operator/           # Growth planning workflows
│   └── reddit-marketing-service-guide/   # User intake routing
├── memory/
│   └── knowledge/                        # Safety gates & boundaries
└── artifacts/
    └── marketplace-template-handoff.md
```

## Key Skills

### 1. Reddit Public API Research (`skills/reddit/`)

No API key required. Uses Reddit's public JSON endpoints.

**Available Scripts:**

```python
# Search posts across subreddits
python skills/reddit/scripts/search_posts.py

# Get hot/new/top posts from a subreddit
python skills/reddit/scripts/get_posts.py

# Get subreddit metadata
python skills/reddit/scripts/get_subreddit.py

# Get specific post with comments
python skills/reddit/scripts/get_post.py

# Get user profile and history
python skills/reddit/scripts/get_user.py
```

**Example: Search for product mentions**

```python
from skills.reddit.scripts.reddit_api import RedditAPI

api = RedditAPI()

# Search for "AI marketing tools" discussions
results = api.search_posts(
    query="AI marketing tools",
    subreddit="marketing",
    sort="relevance",
    time_filter="month",
    limit=25
)

for post in results:
    print(f"{post['title']} - {post['score']} upvotes")
    print(f"https://reddit.com{post['permalink']}\n")
```

**Example: Analyze subreddit for product fit**

```python
from skills.reddit.scripts.reddit_api import RedditAPI

api = RedditAPI()

# Get subreddit info
subreddit = api.get_subreddit("SaaS")
print(f"Subscribers: {subreddit['subscribers']}")
print(f"Description: {subreddit['public_description']}")

# Get recent posts to analyze topics
posts = api.get_posts("SaaS", sort="hot", limit=50)

# Analyze for pain points
pain_points = [
    post for post in posts 
    if any(word in post['title'].lower() for word in ['help', 'how', 'problem', 'issue'])
]

print(f"Found {len(pain_points)} potential lead posts")
```

### 2. Reddit Growth Operator (`skills/reddit-growth-operator/`)

Core growth planning skill with reference materials:

- **reddit-account-karma-and-risk.md**: Trust-building and risk management
- **reddit-content-sop.md**: Post/comment SOPs (knowledge-base, hot-topic, resource, story formats)
- **reddit-ai-tool-promotion.md**: Soft-ad patterns and promotion strategies
- **reddit-anti-ai-slop-writing.md**: Banned vocabulary and AI-sounding phrase detection
- **reddit-community-map.md**: Startup, product, design, marketing, AI subreddit directory
- **reddit-github-and-viral-ideas.md**: Technical workflow and viral mechanics

**Example: Generate subreddit research plan**

```markdown
# Task: Find communities for [Your Product]

Use reddit-growth-operator skill to:

1. Review reddit-community-map.md for relevant subreddit categories
2. Use reddit API to fetch subscriber counts and activity levels
3. Analyze recent posts for:
   - Question frequency related to your problem space
   - Subreddit rules on self-promotion
   - Moderator activity and ban risk
4. Generate scored list of top 10 target communities
5. Output research report with engagement strategy per subreddit
```

### 3. Reddit Marketing Service Guide (`skills/reddit-marketing-service-guide/`)

User-facing intake system that routes to service paths:

- **A.** Find relevant subreddits and communities
- **B.** Draft a natural Reddit post or comment
- **C.** Build a full Reddit launch/seeding plan
- **D.** Review draft for spam risk and AI-sounding copy
- **E.** Show handoff guide

**Example: Request a launch plan**

```markdown
I need a Reddit launch plan for my AI writing assistant.

Target audience: Content marketers, bloggers, copywriters
Goal: 50 signups in first week
Current karma: 150 post, 300 comment
```

Agent will:
1. Research relevant subreddits using reddit API
2. Draft 3-5 posts in different formats (story, resource, hot-topic)
3. Review drafts with anti-AI-slop checker
4. Generate staggered posting schedule with cooldowns
5. Stage drafts for approval (nothing published without consent)

## Configuration

### Environment Variables

Set these for Reddit integration (if using publishing features):

```bash
# Reddit OAuth credentials (for posting/commenting)
export REDDIT_CLIENT_ID="your_client_id"
export REDDIT_CLIENT_SECRET="your_client_secret"
export REDDIT_USERNAME="your_username"
export REDDIT_PASSWORD="your_password"
export REDDIT_USER_AGENT="YourApp/0.1 by /u/yourusername"
```

### Matrix Integration

In Matrix, configure the Reddit department:

1. Navigate to **Integrations** → **Reddit**
2. Authorize OAuth connection
3. Set approval gates:
   - ✅ Require approval for all posts
   - ✅ Require approval for all comments
   - ✅ Require approval for all DMs
4. Configure cooldowns:
   - Minimum 2 hours between posts
   - Minimum 30 minutes between comments

## Common Patterns

### Pattern 1: Subreddit Discovery & Scoring

```python
from skills.reddit.scripts.reddit_api import RedditAPI

api = RedditAPI()

# Target subreddits to evaluate
candidates = ["SaaS", "startups", "Entrepreneur", "smallbusiness", "marketing"]

results = []
for sub in candidates:
    info = api.get_subreddit(sub)
    posts = api.get_posts(sub, sort="hot", limit=50)
    
    # Score based on activity and question frequency
    questions = sum(1 for p in posts if '?' in p['title'])
    avg_engagement = sum(p['score'] + p['num_comments'] for p in posts) / len(posts)
    
    results.append({
        'subreddit': sub,
        'subscribers': info['subscribers'],
        'questions_per_50': questions,
        'avg_engagement': avg_engagement,
        'score': (questions * 2) + (avg_engagement / 10)
    })

# Sort by score
results.sort(key=lambda x: x['score'], reverse=True)

for r in results:
    print(f"{r['subreddit']}: {r['score']:.1f} score ({r['subscribers']} subs, {r['questions_per_50']} questions)")
```

### Pattern 2: Lead Scouting in Target Subreddit

```python
from skills.reddit.scripts.reddit_api import RedditAPI

api = RedditAPI()

# Find people asking for tool recommendations
query = "tool for writing content"
leads = api.search_posts(
    query=query,
    subreddit="marketing",
    sort="new",
    time_filter="week",
    limit=100
)

# Filter for genuine questions (not promotional)
qualified_leads = [
    lead for lead in leads
    if lead['score'] < 50  # Not viral (harder to add value)
    and lead['num_comments'] < 20  # Still room for discussion
    and not lead['is_self'] == False  # Text post, not link spam
]

print(f"Found {len(qualified_leads)} qualified leads")

for lead in qualified_leads[:5]:
    print(f"\n{lead['title']}")
    print(f"https://reddit.com{lead['permalink']}")
    print(f"{lead['score']} upvotes, {lead['num_comments']} comments")
```

### Pattern 3: Anti-AI-Slop Review Workflow

Use the anti-AI-slop reference from `skills/reddit-growth-operator/references/reddit-anti-ai-slop-writing.md`:

**Banned vocabulary checklist:**
- "delve", "navigate", "landscape", "leverage", "testament"
- "Furthermore", "Moreover", "In conclusion"
- "game-changer", "revolutionary", "cutting-edge"
- "unlock", "elevate", "empower", "streamline"

**Example review:**

```python
# Draft to review
draft = """
Furthermore, our AI tool is a game-changer that will elevate your content strategy. 
By leveraging cutting-edge technology, you can unlock new levels of productivity 
and streamline your workflow. It's truly revolutionary.
"""

# AI-slop detector (simplified)
banned_words = ["furthermore", "game-changer", "elevate", "leveraging", "cutting-edge", "unlock", "streamline", "revolutionary"]

issues = []
for word in banned_words:
    if word.lower() in draft.lower():
        issues.append(f"❌ Remove '{word}' (AI-slop vocabulary)")

if issues:
    print("AI-SLOP DETECTED:")
    for issue in issues:
        print(issue)
    print("\n🔄 Rewrite required before approval")
```

**Better version:**

```
I built a tool that helps me write faster. It's not perfect, but it cut my 
content time in half. Happy to share if anyone's struggling with the same problem.
```

## Troubleshooting

### Rate Limiting

Reddit's public JSON API has rate limits (~60 requests/min):

```python
import time
from skills.reddit.scripts.reddit_api import RedditAPI

api = RedditAPI()

# Add delays between requests
subreddits = ["SaaS", "startups", "Entrepreneur"]
for sub in subreddits:
    posts = api.get_posts(sub, limit=50)
    # Process posts...
    time.sleep(2)  # 2-second delay between subreddits
```

### Missing Posts in Search Results

Reddit search can be unreliable. Use multiple approaches:

```python
# Approach 1: Direct search
results = api.search_posts(query="AI tools", subreddit="marketing")

# Approach 2: Fetch hot posts and filter locally
posts = api.get_posts("marketing", sort="hot", limit=100)
filtered = [p for p in posts if "AI" in p['title'] or "AI" in p['selftext']]
```

### 403 Errors on Private/Banned Subreddits

Handle gracefully:

```python
from skills.reddit.scripts.reddit_api import RedditAPI

api = RedditAPI()

try:
    subreddit = api.get_subreddit("privatesubreddit")
except Exception as e:
    if "403" in str(e):
        print("Subreddit is private or banned")
    else:
        raise
```

### User-Agent Blocks

If requests fail, ensure proper User-Agent:

```python
# In reddit_api.py, verify headers include:
headers = {
    'User-Agent': 'Mozilla/5.0 (compatible; RedditResearch/1.0)'
}
```

## Safety & Compliance

This template enforces strict safety gates:

1. **No Auto-Publishing**: All posts/comments require explicit approval
2. **Cooldown Management**: Minimum 2-hour gaps between posts
3. **Anti-Spam**: No duplicate content, karma farming, or vote manipulation
4. **Disclosure**: All promotional content must disclose affiliation
5. **Rule Compliance**: Subreddit rules checked before drafting

**Memory cards enforce boundaries:**

- `reddit-template-operating-boundary.md`: What to keep vs. exclude
- `reddit-publishing-safety-gates.md`: Approval gates and risk signals
- `reddit-cooldown-and-retry-deduplication.md`: Cadence management

## Real-World Workflow Example

**Scenario**: Launch a new SaaS product on Reddit

```markdown
# Step 1: Research
Agent uses reddit API to:
- Fetch top 20 SaaS/startup subreddits
- Analyze 50 recent posts per subreddit
- Score by question frequency and engagement
- Output: Top 5 target communities

# Step 2: Lead Scouting
Agent searches for:
- "looking for tool to [problem your product solves]"
- "how do you [use case]"
- "alternatives to [competitor]"
- Filters by recency and comment count
- Output: 15 qualified lead threads

# Step 3: Draft Responses
Agent generates 3 response formats:
- Story: "I had this problem and built [product] to solve it"
- Resource: "Here are 5 tools that do this (including mine)"
- Hot-take: "Unpopular opinion: [relevant insight] + subtle mention"

# Step 4: Anti-AI-Slop Review
Agent checks each draft for:
- Banned vocabulary (delve, leverage, game-changer, etc.)
- Overly formal structure
- Lack of personality/voice
- Output: Flagged issues + rewrite suggestions

# Step 5: Approval Queue
Agent stages drafts with:
- Target post URL
- Proposed response
- Risk assessment (low/medium/high)
- Recommended cooldown
- Human approves or edits before publishing

# Step 6: Publishing & Tracking
After approval:
- Post comment to Reddit
- Log timestamp, URL, and draft version
- Set cooldown timer for next action
- Track engagement (upvotes, replies) after 24h
```

## Integration with Matrix Workflows

Use with other Matrix departments:

```yaml
# In your Matrix company workflow
departments:
  - reddit-marketing:
      skills: [reddit, reddit-growth-operator]
      triggers: ["reddit", "community marketing", "lead gen"]
      
  - content-production:
      skills: [copywriting, editing]
      
  - analytics:
      skills: [engagement-tracking]

# Cross-department flow
1. reddit-marketing finds leads
2. content-production drafts responses
3. reddit-marketing runs anti-AI-slop review
4. Human approves
5. reddit-marketing publishes
6. analytics tracks engagement
```

## Resources

- [Matrix Platform](https://matrix.build)
- [Reddit JSON API Documentation](https://github.com/reddit-archive/reddit/wiki/JSON)
- Project Repository: https://github.com/Weike/reddit-agent-marketing-department

---

**License**: MIT  
**Maintainer**: Weike  
**Matrix Marketplace**: Compatible
