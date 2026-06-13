---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do i generate content automatically with AI pipeline
  - set up automated content creation with Claude and OpenAI
  - create videos from content using Remotion
  - build AI-powered content workflow from research to video
  - automate content research and script generation
  - use ultimate ai content pipeline for marketing
  - generate multilingual content with AI automation
  - create social media videos automatically from articles
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the **Ultimate AI Content Pipeline** - an end-to-end automated content creation system that handles research, script writing, and video generation using Claude 3, OpenAI, and Remotion.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Scan Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Multi-Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install pnpm if not available
npm install -g pnpm
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create `.env` file with the following variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for Research Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key APIs and Usage

### 1. Research Crawler

```typescript
import { ResearchCrawler } from '@/lib/crawler/research-crawler';

// Initialize crawler
const crawler = new ResearchCrawler({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch recent news by keyword
const research = await crawler.fetchRecentNews({
  keyword: 'AI marketing automation',
  timeframe: '24h',
  maxResults: 10
});

// Extract insights
const insights = await crawler.extractInsights(research);
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';

// Using Claude
const claudeGenerator = new ContentGenerator({
  provider: 'anthropic',
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content with specific format
const content = await claudeGenerator.generate({
  topic: 'AI in Marketing',
  format: 'toplist', // Options: 'toplist', 'pov', 'case-study', 'how-to'
  language: 'vi', // 'en' or 'vi'
  tone: 'professional', // 'professional', 'friendly', 'humorous'
  researchData: insights
});

// Using OpenAI
const openAIGenerator = new ContentGenerator({
  provider: 'openai',
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});
```

### 3. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

// Create video from content
const renderer = new VideoRenderer({
  licenseKey: process.env.REMOTION_LICENSE_KEY
});

const video = await renderer.createFromContent({
  content: content,
  template: 'infographic', // 'infographic', 'shorts', 'reels'
  aspectRatio: '9:16', // TikTok/Reels format
  duration: 30 // seconds
});

// Render video
const bundled = await bundle({
  entryPoint: './remotion/index.ts',
  webpackOverride: (config) => config
});

await renderMedia({
  composition: video.composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: `out/${video.id}.mp4`
});
```

## Common Patterns

### Full Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
  remotionLicense: process.env.REMOTION_LICENSE_KEY
});

// Run complete pipeline
const result = await pipeline.run({
  keyword: 'AI Marketing Tools 2026',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormats: ['reels', 'shorts']
});

// Result contains:
// - research: { sources, insights, data }
// - content: { en: {...}, vi: {...} }
// - videos: [{ url, format, platform }]
```

### Batch Content Generation

```typescript
import { BatchProcessor } from '@/lib/batch-processor';

const batchProcessor = new BatchProcessor({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  concurrency: 3 // Process 3 at a time
});

const keywords = [
  'AI Marketing Automation',
  'Social Media Tools 2026',
  'Content Creation AI'
];

const results = await batchProcessor.processKeywords({
  keywords,
  format: 'pov',
  language: 'vi',
  onProgress: (keyword, progress) => {
    console.log(`${keyword}: ${progress}%`);
  }
});
```

### Custom Video Template

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  content: string[];
  theme: 'dark' | 'light';
}> = ({ title, content, theme }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ 
      backgroundColor: theme === 'dark' ? '#000' : '#fff',
      opacity
    }}>
      <h1 style={{ 
        color: theme === 'dark' ? '#fff' : '#000',
        fontSize: 60
      }}>
        {title}
      </h1>
      {content.map((item, i) => (
        <p key={i}>{item}</p>
      ))}
    </AbsoluteFill>
  );
};
```

## Running the Application

### Development Mode

```bash
# Start Next.js dev server
pnpm dev

# Access at http://localhost:3000
```

### Build for Production

```bash
# Build the application
pnpm build

# Start production server
pnpm start
```

### Remotion Studio

```bash
# Open Remotion visual editor
pnpm remotion:studio

# Render specific composition
pnpm remotion:render
```

## Configuration

### Content Generator Settings

```typescript
// src/config/content.ts
export const contentConfig = {
  formats: {
    toplist: {
      minItems: 5,
      maxItems: 10,
      includeIntro: true,
      includeConclusion: true
    },
    pov: {
      perspective: 'first-person',
      includePersonalExperience: true
    },
    caseStudy: {
      includeMetrics: true,
      includeTimeline: true
    },
    howTo: {
      stepByStep: true,
      includeImages: true
    }
  },
  tones: {
    professional: 'Expert, data-driven, authoritative',
    friendly: 'Conversational, approachable, helpful',
    humorous: 'Light-hearted, engaging, entertaining'
  }
};
```

### Video Rendering Settings

```typescript
// src/config/video.ts
export const videoConfig = {
  templates: {
    infographic: {
      fps: 30,
      durationInFrames: 900, // 30 seconds
      width: 1080,
      height: 1920
    },
    reels: {
      fps: 30,
      durationInFrames: 450, // 15 seconds
      width: 1080,
      height: 1920
    },
    shorts: {
      fps: 30,
      durationInFrames: 1800, // 60 seconds
      width: 1080,
      height: 1920
    }
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

const content = await retry(
  () => generator.generate(params),
  {
    maxAttempts: 3,
    delayMs: 1000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" pnpm remotion:render
```

### Research Crawler Timeouts

```typescript
// Adjust timeout settings
const crawler = new ResearchCrawler({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  timeout: 30000, // 30 seconds
  retries: 2
});
```

### Missing Environment Variables

```typescript
// src/lib/utils/env-check.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
    'REMOTION_LICENSE_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Claude API Errors

```typescript
// Handle rate limiting and errors
try {
  const content = await claudeGenerator.generate(params);
} catch (error) {
  if (error.status === 429) {
    // Rate limit - wait and retry
    await new Promise(resolve => setTimeout(resolve, 60000));
    return claudeGenerator.generate(params);
  }
  throw error;
}
```

## Advanced Usage

### Scheduling Content Generation

```typescript
import cron from 'node-cron';

// Generate content daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const result = await pipeline.run({
    keyword: 'Daily Tech News',
    formats: ['toplist'],
    languages: ['en', 'vi'],
    generateVideo: true
  });
  
  // Auto-publish to social media
  await publishToSocialMedia(result);
});
```

### Custom Research Sources

```typescript
// Add custom crawler for specific sources
import { BaseCrawler } from '@/lib/crawler/base-crawler';

class CustomSourceCrawler extends BaseCrawler {
  async fetch(keyword: string) {
    const response = await this.httpClient.get(
      `https://api.customsource.com/search?q=${keyword}`,
      { headers: { 'X-API-Key': process.env.CUSTOM_SOURCE_KEY } }
    );
    
    return this.parseResults(response.data);
  }
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline for automated content creation, research, and video generation workflows.
