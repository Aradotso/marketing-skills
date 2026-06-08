---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate content pipeline from keyword to video
  - set up automated marketing content system
  - create content using Claude and OpenAI for research
  - automate video generation from written content
  - build AI content workflow with Remotion
  - research and generate marketing content automatically
  - use marketing pipeline for automated posts
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a complete content automation system that transforms a single keyword into fully-researched articles and auto-generated videos. It combines:

- **Auto-research**: Crawls fresh data from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI writing**: Generates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Video rendering**: Automatically creates videos from content using Remotion
- **Multi-language**: Produces both English and Vietnamese content
- **Multi-format**: Optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Services
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Functionality

### 1. Research Module

Automatically crawls and analyzes content from multiple sources:

```typescript
import { researchKeyword } from '@/lib/research/crawler';

// Research a keyword
const research = await researchKeyword({
  keyword: 'AI marketing trends',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

// Returns structured data
console.log(research.insights);    // Key insights
console.log(research.statistics);  // Data points
console.log(research.sources);     // Source URLs
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate article
const content = await generateContent({
  keyword: 'AI marketing automation',
  format: 'toplist',           // 'toplist' | 'pov' | 'case-study' | 'how-to'
  tone: 'expert',              // 'expert' | 'friendly' | 'humorous'
  language: 'vi',              // 'en' | 'vi'
  provider: 'claude',          // 'claude' | 'openai'
  researchData: research       // From research module
});

// Returns structured content
console.log(content.title);
console.log(content.body);
console.log(content.sections);
console.log(content.metadata);
```

### 3. Multi-Language Generation

Generate content in both languages simultaneously:

```typescript
import { generateMultiLanguage } from '@/lib/ai/multilang';

const contents = await generateMultiLanguage({
  keyword: 'content marketing',
  format: 'how-to',
  languages: ['en', 'vi'],
  researchData: research
});

// Access both versions
console.log(contents.en.title);  // English version
console.log(contents.vi.title);  // Vietnamese version
```

### 4. Video Generation with Remotion

Convert content to video automatically:

```typescript
import { renderVideo } from '@/lib/video/renderer';

// Generate video from content
const video = await renderVideo({
  content: content,
  template: 'infographic',     // 'infographic' | 'text-animation' | 'slides'
  format: 'vertical',          // 'vertical' | 'square' | 'horizontal'
  duration: 30,                // seconds
  platform: 'tiktok'          // 'tiktok' | 'reels' | 'shorts'
});

// Returns video URL and metadata
console.log(video.url);
console.log(video.duration);
console.log(video.resolution);
```

## API Routes

The Next.js app provides API endpoints for the pipeline:

### Research Endpoint

```typescript
// POST /api/research
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'digital marketing',
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  })
});

const data = await response.json();
```

### Content Generation Endpoint

```typescript
// POST /api/generate
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'social media strategy',
    format: 'toplist',
    language: 'vi',
    includeResearch: true
  })
});

const content = await response.json();
```

### Video Rendering Endpoint

```typescript
// POST /api/render-video
const response = await fetch('/api/render-video', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    contentId: '123',
    template: 'infographic',
    format: 'vertical'
  })
});

const video = await response.json();
```

## Full Pipeline Example

Complete workflow from keyword to video:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize pipeline
const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY!,
  openaiKey: process.env.OPENAI_API_KEY!,
  rapidApiKey: process.env.RAPIDAPI_KEY!
});

// Run full pipeline
async function createContent(keyword: string) {
  try {
    // 1. Research
    const research = await pipeline.research(keyword, {
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });

    // 2. Generate content (both languages)
    const content = await pipeline.generate({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      researchData: research,
      tone: 'expert'
    });

    // 3. Render video
    const video = await pipeline.renderVideo({
      content: content.en,
      template: 'infographic',
      format: 'vertical',
      platform: 'tiktok'
    });

    // 4. Save to database
    await pipeline.save({
      research,
      content,
      video
    });

    return {
      research,
      content,
      video
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute
const result = await createContent('AI automation tools');
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

## Common Patterns

### Custom Research Sources

```typescript
import { addCustomSource } from '@/lib/research/sources';

// Add custom RSS feed or API
addCustomSource({
  name: 'custom-blog',
  type: 'rss',
  url: 'https://example.com/feed.xml',
  parser: (data) => ({
    title: data.title,
    content: data.content,
    publishedAt: data.pubDate
  })
});
```

### Custom Content Templates

```typescript
import { registerTemplate } from '@/lib/ai/templates';

// Register custom content format
registerTemplate('product-review', {
  structure: [
    { section: 'introduction', maxWords: 100 },
    { section: 'pros', format: 'list' },
    { section: 'cons', format: 'list' },
    { section: 'verdict', maxWords: 150 }
  ],
  tone: 'balanced',
  cta: true
});
```

### Custom Video Templates

```typescript
import { Composition } from 'remotion';

// Create custom Remotion composition
export const CustomVideoTemplate: React.FC<{
  content: ContentData;
  duration: number;
}> = ({ content, duration }) => {
  return (
    <div>
      <h1>{content.title}</h1>
      {content.sections.map((section, i) => (
        <div key={i}>{section.text}</div>
      ))}
    </div>
  );
};

// Register in remotion/index.ts
<Composition
  id="custom-template"
  component={CustomVideoTemplate}
  durationInFrames={900}
  fps={30}
  width={1080}
  height={1920}
/>
```

## Scheduling & Automation

### Schedule Content Generation

```typescript
import { scheduleContentJob } from '@/lib/scheduler';

// Schedule daily content generation
scheduleContentJob({
  keywords: ['AI trends', 'marketing automation'],
  frequency: 'daily',
  time: '09:00',
  formats: ['toplist', 'how-to'],
  autoPublish: false
});
```

## Troubleshooting

### AI API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

// Implement rate limiting
const limiter = new RateLimiter({
  maxRequests: 50,
  timeWindow: 60000 // 1 minute
});

await limiter.execute(async () => {
  return await generateContent({...});
});
```

### Research Data Quality

```typescript
// Validate research data before generation
import { validateResearch } from '@/lib/research/validator';

const research = await researchKeyword('keyword');
const validation = validateResearch(research);

if (!validation.isValid) {
  console.warn('Research quality issues:', validation.issues);
  // Fallback to cached data or retry
}
```

### Video Rendering Timeouts

```typescript
// Configure timeout for long renders
const video = await renderVideo({
  content,
  template: 'infographic',
  options: {
    timeout: 300000, // 5 minutes
    quality: 'high',
    retries: 2
  }
});
```

### Memory Issues with Large Content

```typescript
// Stream large content generation
import { streamContent } from '@/lib/ai/stream';

const stream = await streamContent({
  keyword: 'comprehensive guide',
  format: 'case-study'
});

for await (const chunk of stream) {
  process.stdout.write(chunk);
}
```

## Best Practices

1. **Always validate API keys** before starting pipeline
2. **Use research cache** to avoid redundant API calls
3. **Implement error handling** for each pipeline stage
4. **Monitor API usage** to stay within limits
5. **Test video templates** before bulk generation
6. **Use webhooks** for long-running video renders
7. **Store content versions** for A/B testing

## Performance Optimization

```typescript
// Parallel processing for multiple keywords
import { parallelPipeline } from '@/lib/pipeline/parallel';

const results = await parallelPipeline({
  keywords: ['AI', 'marketing', 'automation'],
  concurrency: 3,
  format: 'toplist'
});
```

This skill enables AI coding agents to effectively use the Ultimate AI Content Pipeline for automated marketing content creation, from research through video generation.
