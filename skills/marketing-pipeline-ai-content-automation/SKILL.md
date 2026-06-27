---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scripting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - generate automated content with research and video using this pipeline
  - configure Claude and OpenAI for content automation
  - create automated blog posts with AI research
  - render videos automatically from content scripts
  - set up marketing content automation with remotion
  - automate content research and video generation
  - build AI-powered content creation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides a complete automated content pipeline that handles research, content generation, and video rendering. It crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls fresh content from major news sources and social platforms
- **AI Content Generation**: Creates blog posts, case studies, how-tos, and POV pieces in multiple languages
- **Video Rendering**: Automatically converts written content into videos optimized for TikTok, Reels, and Shorts
- **Multi-Format Support**: Generates content in various formats (toplist, case study, how-to, POV)
- **Bilingual Output**: Simultaneous English and Vietnamese content generation

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```env
# AI Models
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Content Sources
TECHCRUNCH_API_KEY=your_key_here
TWITTER_BEARER_TOKEN=your_token_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion Config
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Key Components

### 1. Research Module

Automatically crawls and aggregates content from multiple sources:

```typescript
import { ResearchEngine } from './src/research/engine';

const research = new ResearchEngine({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch recent content by keyword
const insights = await research.crawl({
  keyword: 'AI marketing automation',
  timeframe: '24h',
  minRelevance: 0.7
});

console.log(insights);
// Returns: { articles: [...], tweets: [...], insights: [...] }
```

### 2. Content Generation with Claude

Generate content in multiple formats using Anthropic's Claude:

```typescript
import { ContentGenerator } from './src/content/generator';

const generator = new ContentGenerator({
  provider: 'claude',
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate blog post from research data
const content = await generator.generate({
  type: 'blog-post',
  format: 'how-to',
  research: insights,
  tone: 'professional',
  language: 'en',
  length: 'medium'
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Bilingual Content Generation

Create content in both English and Vietnamese simultaneously:

```typescript
import { BilingualGenerator } from './src/content/bilingual';

const bilingualGen = new BilingualGenerator({
  claudeKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY
});

const bilingualContent = await bilingualGen.generatePair({
  topic: 'Marketing Automation Trends 2024',
  format: 'toplist',
  sourceData: insights
});

console.log(bilingualContent.english);
console.log(bilingualContent.vietnamese);
```

### 4. Video Rendering with Remotion

Convert written content to video automatically:

```typescript
import { VideoRenderer } from './src/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer({
  compositionId: 'MarketingVideo',
  outputFormat: 'mp4'
});

// Render video from content
const videoPath = await renderer.render({
  content: content,
  style: 'modern',
  duration: 60, // seconds
  aspectRatio: '9:16', // For TikTok/Reels
  outputPath: './output/video.mp4'
});

console.log(`Video rendered: ${videoPath}`);
```

### 5. Complete Pipeline Execution

Run the entire pipeline from research to video:

```typescript
import { ContentPipeline } from './src/pipeline';

const pipeline = new ContentPipeline({
  openaiKey: process.env.OPENAI_API_KEY,
  claudeKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Execute full pipeline
const result = await pipeline.execute({
  keyword: 'AI content marketing',
  contentType: 'case-study',
  includeVideo: true,
  languages: ['en', 'vi'],
  platforms: ['tiktok', 'reels', 'shorts']
});

console.log(`Generated: ${result.articles.length} articles`);
console.log(`Rendered: ${result.videos.length} videos`);
```

## Common Patterns

### Pattern 1: Daily Research Automation

```typescript
import { ScheduledResearch } from './src/automation/scheduler';

const scheduler = new ScheduledResearch({
  interval: '24h',
  keywords: ['AI marketing', 'content automation', 'video marketing'],
  autoGenerate: true
});

// Start automated daily research
await scheduler.start();
```

### Pattern 2: Custom Content Formats

```typescript
import { CustomFormat } from './src/content/formats';

const customFormat = new CustomFormat({
  name: 'product-review',
  structure: {
    intro: { tone: 'engaging', length: 'short' },
    pros: { format: 'bullet-points' },
    cons: { format: 'bullet-points' },
    verdict: { tone: 'authoritative' }
  }
});

const review = await generator.generate({
  format: customFormat,
  research: insights
});
```

### Pattern 3: Batch Video Generation

```typescript
import { BatchRenderer } from './src/video/batch';

const batchRenderer = new BatchRenderer({
  concurrent: 3,
  outputDir: './output/videos'
});

const articles = [content1, content2, content3];

const videos = await batchRenderer.renderBatch({
  contents: articles,
  variants: [
    { aspectRatio: '9:16', platform: 'tiktok' },
    { aspectRatio: '16:9', platform: 'youtube' },
    { aspectRatio: '1:1', platform: 'instagram' }
  ]
});
```

### Pattern 4: Content Quality Control

```typescript
import { QualityChecker } from './src/content/quality';

const checker = new QualityChecker({
  minScore: 0.8,
  checks: ['grammar', 'readability', 'seo', 'factuality']
});

const qualityReport = await checker.evaluate(content);

if (qualityReport.score < 0.8) {
  console.log('Needs improvement:', qualityReport.issues);
  
  // Auto-improve content
  const improved = await generator.improve({
    content: content,
    issues: qualityReport.issues
  });
}
```

## CLI Commands

If the project includes CLI tools:

```bash
# Research command
npm run research -- --keyword "AI marketing" --timeframe 24h

# Generate content
npm run generate -- --type blog-post --format how-to --lang en

# Render video
npm run render -- --input content.json --output video.mp4 --aspect 9:16

# Run full pipeline
npm run pipeline -- --keyword "content automation" --with-video
```

## API Integration Examples

### OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT4(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a content marketing expert.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return completion.choices[0].message.content;
}
```

### Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].text;
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
import { RateLimiter } from './src/utils/rateLimiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perWindow: '1m',
  provider: 'openai'
});

// Use with rate limiting
const safeGenerate = limiter.wrap(async (prompt: string) => {
  return await generator.generate({ prompt });
});
```

### Issue: Video Rendering Failures

```typescript
// Add retry logic
import { RetryRenderer } from './src/video/retry';

const retryRenderer = new RetryRenderer({
  maxAttempts: 3,
  backoff: 'exponential'
});

try {
  const video = await retryRenderer.render(content);
} catch (error) {
  console.error('Rendering failed after retries:', error);
  // Fallback to image generation
  const image = await renderer.renderStatic(content);
}
```

### Issue: Research Data Quality

```typescript
// Filter and validate research data
import { DataValidator } from './src/research/validator';

const validator = new DataValidator({
  minRelevance: 0.7,
  requireSources: true,
  excludeDuplicates: true
});

const validatedInsights = await validator.validate(insights);
```

### Issue: Memory Issues with Large Batches

```typescript
// Process in chunks
async function processInChunks<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize: number = 5
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(processor)
    );
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Development Workflow

```bash
# Start development server
npm run dev

# Run tests
npm test

# Build for production
npm run build

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Advanced Configuration

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    refreshInterval: '6h',
    cacheExpiry: '24h'
  },
  content: {
    providers: {
      primary: 'claude',
      fallback: 'openai'
    },
    formats: ['blog-post', 'case-study', 'how-to', 'toplist'],
    languages: ['en', 'vi'],
    tones: ['professional', 'casual', 'humorous']
  },
  video: {
    defaultAspectRatio: '9:16',
    fps: 30,
    bitrate: '5M',
    codec: 'h264'
  }
};
```

This skill enables AI agents to help developers implement complete automated content marketing pipelines with research, generation, and video rendering capabilities.
