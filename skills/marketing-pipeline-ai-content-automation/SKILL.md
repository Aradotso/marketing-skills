---
name: marketing-pipeline-ai-content-automation
description: Automate end-to-end content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline for marketing
  - generate blog posts and videos automatically
  - use Claude and OpenAI for content automation
  - create automated content workflow with research
  - build AI-powered marketing content system
  - automate video generation from written content
  - set up content automation with Remotion
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline is a comprehensive TypeScript-based content automation system that creates a complete content production pipeline. It automatically:

1. **Research** - Scrapes and analyzes real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn
2. **Content Generation** - Uses Claude 3 and OpenAI to create content in multiple formats (Top lists, POV, Case Studies, How-to)
3. **Multi-language Output** - Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering** - Automatically converts content to videos and infographics using Remotion

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm >= 9.0.0
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_bearer_token_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key_here
```

## Key Architecture

### Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── research/          # Web scraping and data collection
│   ├── ai/                # AI content generation (Claude/OpenAI)
│   ├── video/             # Remotion video rendering
│   ├── utils/             # Shared utilities
│   └── app/               # Next.js application
├── remotion/              # Remotion video templates
└── public/                # Static assets
```

## Core Workflows

### 1. Research & Data Collection

#### Scraping Recent Content

```typescript
import { ResearchEngine } from './src/research/engine';

// Initialize research engine
const researcher = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  keywords: ['AI', 'marketing automation']
});

// Fetch and analyze data
const insights = await researcher.gather();

console.log(insights);
// {
//   articles: [...],
//   trends: [...],
//   keyInsights: [...],
//   dataPoints: [...]
// }
```

#### Custom Source Configuration

```typescript
import { SourceConfig } from './src/research/types';

const customSource: SourceConfig = {
  name: 'custom-blog',
  url: 'https://example.com/blog',
  selectors: {
    title: '.post-title',
    content: '.post-content',
    date: '.post-date'
  },
  rateLimit: 1000 // ms between requests
};

const researcher = new ResearchEngine({
  customSources: [customSource]
});
```

### 2. AI Content Generation

#### Using Claude for Content Creation

```typescript
import { ClaudeContentGenerator } from './src/ai/claude';

const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content from research data
const content = await generator.create({
  research: insights,
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  targetAudience: 'marketers',
  wordCount: 1500
});

console.log(content);
// {
//   title: "Top 10 AI Marketing Trends in 2024",
//   body: "...",
//   metadata: { ... }
// }
```

#### OpenAI Integration

```typescript
import { OpenAIContentGenerator } from './src/ai/openai';

const openaiGen = new OpenAIContentGenerator({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

// Generate multiple format variations
const variations = await openaiGen.createVariations({
  baseContent: content,
  formats: ['how-to', 'case-study', 'pov'],
  count: 3
});
```

#### Multi-language Generation

```typescript
import { MultiLanguageGenerator } from './src/ai/multilang';

const mlGenerator = new MultiLanguageGenerator({
  claudeKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY
});

// Generate parallel English and Vietnamese content
const bilingualContent = await mlGenerator.generateParallel({
  research: insights,
  languages: ['en', 'vi'],
  format: 'toplist',
  maintainTone: true
});

console.log(bilingualContent.en.title);
console.log(bilingualContent.vi.title);
```

### 3. Video Generation with Remotion

#### Basic Video Rendering

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { ContentToVideoConverter } from './src/video/converter';

// Convert content to video props
const converter = new ContentToVideoConverter();
const videoProps = converter.transform(content);

// Bundle Remotion composition
const bundled = await bundle({
  entryPoint: './remotion/index.ts',
  webpackOverride: (config) => config
});

// Get composition
const composition = await selectComposition({
  serveUrl: bundled,
  id: 'ContentVideo',
  inputProps: videoProps
});

// Render video
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: `out/${content.title}.mp4`,
  inputProps: videoProps
});
```

#### Custom Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  branding: BrandConfig;
}> = ({ title, points, branding }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: branding.bgColor }}>
      <div style={{ padding: 60 }}>
        <h1 style={{ 
          fontSize: 60, 
          color: branding.textColor,
          opacity: Math.min(1, frame / 30)
        }}>
          {title}
        </h1>
        {points.map((point, idx) => (
          <p 
            key={idx}
            style={{
              fontSize: 40,
              opacity: frame > (idx + 1) * fps ? 1 : 0,
              transition: 'opacity 0.5s'
            }}
          >
            {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

#### Platform-Specific Export

```typescript
import { PlatformOptimizer } from './src/video/optimizer';

const optimizer = new PlatformOptimizer();

// Render for multiple platforms
const platforms = ['tiktok', 'reels', 'shorts'];

for (const platform of platforms) {
  const config = optimizer.getConfig(platform);
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${platform}/${content.title}.mp4`,
    inputProps: videoProps,
    width: config.width,
    height: config.height,
    fps: config.fps
  });
}
```

### 4. Complete Pipeline Automation

```typescript
import { ContentPipeline } from './src/pipeline';

// Initialize full pipeline
const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h'
  },
  ai: {
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    fallbackProvider: 'openai',
    fallbackKey: process.env.OPENAI_API_KEY
  },
  video: {
    enabled: true,
    platforms: ['tiktok', 'reels', 'shorts']
  },
  output: {
    directory: './output',
    formats: ['markdown', 'html', 'json']
  }
});

// Execute full pipeline
const results = await pipeline.execute({
  keyword: 'AI marketing automation',
  contentFormats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideos: true
});

console.log(results);
// {
//   research: { ... },
//   content: [
//     { format: 'toplist', language: 'en', ... },
//     { format: 'toplist', language: 'vi', ... },
//     { format: 'how-to', language: 'en', ... }
//   ],
//   videos: [
//     { platform: 'tiktok', path: '...' },
//     { platform: 'reels', path: '...' }
//   ]
// }
```

## API Reference

### ResearchEngine

```typescript
class ResearchEngine {
  constructor(config: ResearchConfig);
  gather(): Promise<ResearchInsights>;
  addSource(source: SourceConfig): void;
  setTimeframe(timeframe: string): void;
}
```

### ClaudeContentGenerator

```typescript
class ClaudeContentGenerator {
  constructor(config: ClaudeConfig);
  create(params: ContentParams): Promise<GeneratedContent>;
  refine(content: string, instructions: string): Promise<string>;
}
```

### ContentPipeline

```typescript
class ContentPipeline {
  constructor(config: PipelineConfig);
  execute(params: ExecutionParams): Promise<PipelineResults>;
  schedule(params: ScheduleParams): Promise<void>;
}
```

## Common Patterns

### Pattern 1: Daily Content Automation

```typescript
import cron from 'node-cron';

// Schedule daily content generation at 6 AM
cron.schedule('0 6 * * *', async () => {
  const pipeline = new ContentPipeline(defaultConfig);
  
  const results = await pipeline.execute({
    keyword: 'trending tech news',
    contentFormats: ['toplist'],
    languages: ['en'],
    generateVideos: true
  });
  
  // Auto-publish to platforms
  await publishToWordPress(results.content[0]);
  await uploadToYouTube(results.videos[0]);
});
```

### Pattern 2: Multi-Topic Content Generation

```typescript
const topics = [
  'AI marketing trends',
  'Social media automation',
  'Content creation tools'
];

const batchResults = await Promise.all(
  topics.map(topic => 
    pipeline.execute({
      keyword: topic,
      contentFormats: ['how-to'],
      languages: ['en', 'vi']
    })
  )
);
```

### Pattern 3: Custom AI Prompt Engineering

```typescript
import { PromptTemplate } from './src/ai/prompts';

const customTemplate = new PromptTemplate({
  system: `You are an expert marketing content writer specializing in ${industry}.`,
  user: `Create a ${format} article about ${topic} targeting ${audience}.
         Include:
         - Data-backed insights from recent research
         - Actionable takeaways
         - SEO-optimized structure
         Research data: ${researchData}`
});

const content = await generator.create({
  prompt: customTemplate.render({
    industry: 'SaaS',
    format: 'case-study',
    topic: insights.mainTrend,
    audience: 'startup founders',
    researchData: JSON.stringify(insights)
  })
});
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement exponential backoff
import { retry } from './src/utils/retry';

const content = await retry(
  () => generator.create(params),
  {
    maxAttempts: 3,
    delayMs: 1000,
    exponentialBackoff: true
  }
);
```

### Issue: Video Rendering Memory Issues

```typescript
// Use concurrency limits
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent renders

const videoPromises = platforms.map(platform =>
  limit(() => renderForPlatform(platform, content))
);

await Promise.all(videoPromises);
```

### Issue: Scraping Failures

```typescript
// Add fallback sources
const researcher = new ResearchEngine({
  sources: ['techcrunch', 'a16z'],
  fallbackSources: ['medium', 'dev.to'],
  onSourceFail: (source, error) => {
    console.warn(`Source ${source} failed:`, error);
  },
  minSourcesRequired: 1
});
```

### Issue: Content Quality Validation

```typescript
import { ContentValidator } from './src/utils/validator';

const validator = new ContentValidator({
  minWordCount: 1000,
  requireHeadings: true,
  checkReadability: true
});

const content = await generator.create(params);

if (!validator.validate(content)) {
  // Regenerate with refined prompt
  content = await generator.refine(content, validator.getSuggestions());
}
```

## CLI Usage

If the project includes CLI commands:

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends" --format toplist --lang en

# Run full pipeline
npm run pipeline -- --config ./config/production.json

# Render video only
npm run render -- --input ./content/article.json --platform reels

# Schedule automated runs
npm run schedule -- --cron "0 6 * * *" --topics ./topics.json
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** when scraping external sources
3. **Cache research data** to avoid redundant API calls
4. **Validate content quality** before rendering videos
5. **Use concurrency limits** for video rendering to manage resources
6. **Set up error monitoring** for production pipelines
7. **Version control your prompt templates** for reproducible results

## Resources

- GitHub: https://github.com/pennydinh/marketing-pineline-share
- Remotion Docs: https://remotion.dev
- Anthropic Claude API: https://docs.anthropic.com
- OpenAI API: https://platform.openai.com/docs
