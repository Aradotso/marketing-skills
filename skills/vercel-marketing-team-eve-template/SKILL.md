---
name: vercel-marketing-team-eve-template
description: Team of AI marketing agents built on eve framework for content, social, email, SEO, and product marketing tasks
triggers:
  - "set up the eve marketing team template"
  - "deploy marketing agents with eve framework"
  - "create ai marketing team with notion and resend"
  - "configure eve marketing specialists"
  - "build content and social media agents"
  - "set up typefully and resend integration for marketing"
  - "add custom marketing agent to eve team"
  - "customize marketing team brand context"
---

# vercel-marketing-team-eve-template

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A team of specialized marketing AI agents built on the [eve](https://eve.dev) framework. The team includes a lead agent that delegates to five specialists: product-marketer, content-marketer, social-media-coordinator, seo, and email. Each specialist owns specific deliverables and shares a common brand context document.

## What It Does

The marketing team accepts requests like "write a blog post," "plan a launch," or "audit this page's SEO" and routes them to the appropriate specialist. The lead agent maintains shared product context, delegates to specialists, and returns completed work. All specialists read from a shared brand context document and user preferences.

**Key Features:**
- One-level delegation tree: lead routes to specialists, specialists don't delegate further
- Subagents inherit nothing: each runs in a fresh session
- Shared brand context: all specialists read the same positioning/messaging document
- Human-in-the-loop: publishing actions require approval via Slack or TUI
- Skills load on demand: pulled in when frontmatter description matches the task

## Installation

### Deploy with Vercel

Use the one-click deploy button to provision all connections:

```bash
# Or clone and deploy manually
git clone https://github.com/vercel-labs/marketing-team-eve-template.git
cd marketing-team-eve-template
npm install
```

The deploy provisions:
- Notion connector (`NOTION_CONNECTOR`)
- Resend connector (`RESEND_CONNECTOR`)
- Slack connector with trigger at `/eve/v1/slack` (`SLACK_CONNECTOR`)
- Vercel Blob store
- Typefully API key prompt (`TYPEFULLY_API_KEY`)

### Manual Setup Required

**Resend sending domain:** Verify a domain and create at least one segment in the [Resend dashboard](https://resend.com/domains). The agent cannot create or verify domains.

## Project Structure

```
agent/
  agent.ts                    # Lead agent configuration
  instructions.md             # Lead routing behavior
  channels/
    eve.ts                    # TUI and custom frontend route
    slack.ts                  # Slack integration route
  connections/notion.ts
  tools/                      # Brand context, preferences, artifacts
  lib/                        # Shared utilities
    vercel-blob/              # Blob storage helpers
    brand-context/            # Brand document tools
    user-preferences/         # User-scoped preferences
    content/                  # Linting and artifacts
  subagents/
    product-marketer/         # Positioning, messaging, brand context
    content-marketer/         # Blog posts, landing pages, newsletters
    social-media-coordinator/ # Social drafts in Typefully
    seo/                      # Audits, schema, site architecture
    email/                    # Email campaigns via Resend
```

Each specialist directory contains:
- `agent.ts` - Agent configuration and routing description
- `instructions.md` - Specialist behavior and guidelines
- `sandbox.ts` - Sandbox configuration
- `tools/` - Specialist-specific tools plus shared tools
- `skills/` - Skill files for that specialist

## Core Concepts

### Lead Agent (agent/agent.ts)

The lead loads brand context and user preferences, then delegates to one specialist:

```typescript
import { agent } from 'eve';
import { openai } from '@ai-sdk/openai';

export default agent({
  model: openai('gpt-4o-mini'),
  compactionThreshold: 100_000,
});
```

### Brand Context

The shared state document that all specialists read. Owned by the product-marketer:

```typescript
// In any specialist's tools/get_brand_context.ts
import { tool } from 'eve';
import { z } from 'zod';
import { getBrandContext } from '#lib/brand-context/get.js';

export default tool({
  description: 'Load the shared brand context document',
  parameters: z.object({}),
  execute: async () => {
    const context = await getBrandContext();
    return context || 'No brand context saved yet. Ask the product marketer to create one.';
  },
});
```

### Subagent Configuration

Each specialist is a complete agent with its own configuration:

```typescript
// subagents/content-marketer/agent.ts
import { agent } from 'eve';
import { openai } from '@ai-sdk/openai';

export default agent({
  description: 'Writes blog posts, landing pages, case studies, newsletters, and documentation',
  model: openai('gpt-4o'),
  compactionThreshold: 150_000,
});
```

## Creating a New Specialist

Add a new specialist by creating a directory under `subagents/`:

```typescript
// subagents/podcast-producer/agent.ts
import { agent } from 'eve';
import { openai } from '@ai-sdk/openai';

export default agent({
  description: 'Produces podcast scripts, show notes, and promotion materials',
  model: openai('gpt-4o'),
  compactionThreshold: 100_000,
});
```

```markdown
<!-- subagents/podcast-producer/instructions.md -->
You are the podcast producer for this marketing team.

## Your deliverables

- Episode scripts with timestamps
- Show notes with links and references
- Social promotion snippets
- Email announcement copy

## Every task starts the same way

1. Load the brand context with `get_brand_context`
2. Load this user's preferences with `get_user_preferences`
3. Ask what they need: episode topic, format, guest if any

## Research and drafting

- Use `web_search` and `web_fetch` for guest background and topic research
- Save the script as an artifact with `save_artifact`
- Draft in passes: outline, script, show notes, then edit each
```

Add required tools:

```typescript
// subagents/podcast-producer/tools/get_brand_context.ts
export { default } from '#lib/brand-context/tool.js';

// subagents/podcast-producer/tools/save_artifact.ts
export { default } from '#lib/content/save-artifact-tool.js';
```

## Adding Skills

Skills are markdown files that load on demand when their description matches the task:

```markdown
<!-- subagents/content-marketer/skills/tutorial-writing.md -->
---
description: Writing technical tutorials and how-to guides
---

# Tutorial Writing

## Structure

1. **What you'll build** - Show the end result upfront
2. **Prerequisites** - Tools, accounts, knowledge needed
3. **Step-by-step instructions** - Numbered, tested steps
4. **Troubleshooting** - Common issues and fixes
5. **Next steps** - What to explore after

## Code examples

- Always test code before publishing
- Show complete, runnable examples
- Explain why, not just what
- Use real file paths and structure
```

The skill loads automatically when the agent's task matches "tutorial" or "how-to guide."

## Working with Connections

### Notion Integration

```typescript
// connections/notion.ts
import { connect } from 'eve';
import { mcp } from 'eve/mcp';

export default connect({
  notion: mcp('npx -y @modelcontextprotocol/server-notion'),
});
```

Specialists use Notion tools to create and update pages:

```typescript
// The content-marketer uses Notion to publish blog posts
// Tools are auto-discovered from the MCP server
```

### Typefully (Social Media)

Set the API key in environment variables:

```bash
TYPEFULLY_API_KEY=your_key_here
```

The social-media-coordinator uses Typefully to create and manage social drafts.

### Resend (Email Campaigns)

Per-user OAuth via Vercel Connect. The email specialist uses Resend tools with restricted scope:

```typescript
// Only campaign, list, and diagnostic tools are exposed
// Account administration is never available
```

## Blob Storage for Assets

Upload files that specialists can reference:

```typescript
// lib/vercel-blob/upload-asset-tool.ts
import { tool } from 'eve';
import { z } from 'zod';
import { put } from '@vercel/blob';

export default function uploadAssetTool() {
  return tool({
    description: 'Upload a file (image, PDF, etc.) and get a public URL',
    parameters: z.object({
      filename: z.string(),
      content: z.string().describe('Base64 encoded file content'),
      contentType: z.string(),
    }),
    execute: async ({ filename, content, contentType }, { principal }) => {
      const buffer = Buffer.from(content, 'base64');
      const blob = await put(`${principal}/${filename}`, buffer, {
        access: 'public',
        contentType,
      });
      return {
        url: blob.url,
        message: `Uploaded ${filename} successfully`,
      };
    },
  });
}
```

## User Preferences

Each user can save preferences that persist across sessions:

```typescript
// tools/save_user_preferences.ts
import { tool } from 'eve';
import { z } from 'zod';
import { saveUserPreferences } from '#lib/user-preferences/save.js';

export default tool({
  description: 'Save this user's standing preferences for tone, style, or priorities',
  parameters: z.object({
    preferences: z.string().describe('The preferences text to save'),
  }),
  execute: async ({ preferences }, { principal }) => {
    await saveUserPreferences(principal, preferences);
    return 'Preferences saved. I'll load them at the start of every conversation.';
  },
});
```

## Slack Integration

The Slack channel opens with suggested prompts:

```typescript
// channels/slack.ts
import { channel } from 'eve';

export default channel('slack', {
  suggestedPrompts: [
    { title: 'Sharpen our positioning', message: 'Help me refine our product positioning' },
    { title: 'Write a blog post', message: 'I need a blog post about [topic]' },
    { title: 'Draft social posts', message: 'Create social media posts for [announcement]' },
    { title: 'Review a page's SEO', message: 'Audit the SEO for [URL]' },
  ],
});
```

Requires these Slack scopes:
- `assistant:write` under Bot Scopes
- `assistant_thread_started` and `app_home_opened` event subscriptions

## Human-in-the-Loop Approvals

Actions that cannot be undone pause for approval:

```typescript
// These actions always require approval:
// - Send Resend broadcast, email, or batch
// - Delete Resend resources
// - Change contact topic subscriptions
// - Delete Typefully drafts
// - Publish Typefully drafts
// - Move Notion pages or change views
// - delete_asset, clear_user_preferences

// These do NOT require approval:
// - Create/update Notion pages (drafting is normal flow)
// - save_brand_context (model is told to agree with user first)
```

## Development Workflow

```bash
# Install dependencies
npm install

# Run locally with the TUI
npm run dev

# Deploy to Vercel
vercel deploy

# Lint and format
npm run lint
npm run format
```

## Environment Variables

```bash
# Vercel Connect connector UIDs
NOTION_CONNECTOR=conn_xxx
RESEND_CONNECTOR=conn_xxx
SLACK_CONNECTOR=conn_xxx

# Typefully static key
TYPEFULLY_API_KEY=your_key_here

# Vercel Blob and AI Gateway use OIDC - no manual config needed
```

## Common Patterns

### Delegating from Lead to Specialist

The lead's instructions include complete briefing:

```markdown
## When you delegate

Write a complete brief. The specialist gets no conversation history.

Include:
- What the user asked for
- Relevant brand context
- Any constraints or preferences mentioned
- Expected deliverable format

Then hand back what the specialist returns.
```

### Specialist Research Pattern

```markdown
## Research budget

- Start with `web_search` for 3-5 sources
- Use `web_fetch` to read the most relevant 2-3 pages
- Cite sources in your deliverable
- If you can't find enough, ask the user for more direction
```

### Multi-Pass Editing

```markdown
## Editing passes

1. **Structure** - Does the outline serve the goal?
2. **Clarity** - Can a reader follow the logic?
3. **Precision** - Are claims specific and defensible?
4. **Voice** - Does it match the brand context?
5. **Polish** - Fix typos, smooth awkward phrases
```

## Troubleshooting

### Specialist Not Found

If the lead can't route to a specialist, check:
- Directory exists under `subagents/`
- `agent.ts` has a `description` field
- Directory name matches the description's domain

### Brand Context Not Loading

```typescript
// Check the Blob key
import { BRAND_CONTEXT_KEY } from '#lib/brand-context/key.js';
console.log(BRAND_CONTEXT_KEY); // Should be 'brand-context.md'

// Verify Blob storage permissions
// The project OIDC token should have access
```

### Notion Pages Not Creating

- User must authorize Notion connection (OAuth)
- Notion workspace must have pages the integration can access
- Check connection permissions in Vercel dashboard

### Typefully Drafts Not Appearing

- Verify `TYPEFULLY_API_KEY` is set
- Check the MCP server is running: `npx -y @modelcontextprotocol/server-typefully`
- Ensure the API key has write permissions

### Resend Sending Fails

- User must authorize Resend (per-user OAuth)
- Verify domain in Resend dashboard
- Check that at least one segment exists
- Confirm sending is approved (human-in-the-loop)

## Example: Complete Content Marketing Flow

```markdown
User: "Write a blog post about our new API versioning feature"

Lead agent:
1. Loads brand context with get_brand_context
2. Loads user preferences with get_user_preferences
3. Delegates to content-marketer with brief:
   "Write a blog post about API versioning. Brand context attached. 
    User prefers technical depth with code examples. 
    Target audience: developers integrating our API."

Content marketer:
1. Loads brand context again (fresh session)
2. Loads blog-style skill (matches "blog post")
3. Researches API versioning best practices (web_search, web_fetch)
4. Drafts outline, shares for approval
5. Writes full post in Notion
6. Edits in passes: structure, clarity, precision, voice, polish
7. Returns Notion page link

Lead agent:
Returns to user: "I've published the blog post here: [Notion link]"
```

## Best Practices

1. **Keep the brand context current** - The product-marketer should update it as positioning evolves
2. **One specialist per request** - Don't try to route to multiple specialists in one turn
3. **Complete briefs** - Specialists have no conversation history, so the lead must pass everything
4. **Research before drafting** - Use web tools to gather evidence and examples
5. **Edit in passes** - Structure, then clarity, then precision, then voice, then polish
6. **Test publishing flows** - Always verify Notion/Typefully/Resend integrations in dev first
7. **Respect approval gates** - Never try to bypass human-in-the-loop for irreversible actions
