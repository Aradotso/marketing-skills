---
name: marketing-content-pipeline-automation
description: Automated AI content pipeline for research, scripting, video generation and multi-platform publishing using Claude, OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up an AI marketing content pipeline
  - crawl news and create content with Claude
  - build automated content workflow with video rendering
  - create multi-format content from keywords using AI
  - automate social media content generation and posting
  - research to video content pipeline setup
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation and publishing.

## What This Project Does

The Marketing Content Pipeline is an end-to-end automation system that:

- **Auto-crawls** trending news from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for Reels, TikTok, Shorts
- **Schedules publishing** to multiple platforms via Next.js interface

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if using)
DATABASE_URL=your_database_connection

# Content Publishing
FACEBOOK_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

### Start Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content crawling & analysis
│   │   ├── video/       # Remotion video generation
│   │   └── publisher/   # Platform publishing logic
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Video templates
└── content/             # Generated content cache
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { ResearchEngine } from '@/lib/research/engine';

// Initialize research engine
const research = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  apiKey: process.env.RAPIDAPI_KEY
});

// Crawl content by keyword
async function gatherResearch(keyword: string) {
  const results = await research.scan({
    keyword,
    limit: 50,
    filters: {
      minEngagement: 100,
      language: ['en', 'vi']
    }
  });

  // Extract insights
  const insights = await research.analyzeInsights(results);
  
  return {
    rawData: results,
    insights,
    trending: insights.filter(i => i.score > 0.8)
  };
}
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import { Anthropic } from '@anthropic-ai/sdk';

// Using Claude 3
const generator = new ContentGenerator({
  provider: 'claude',
  model: 'claude-3-sonnet-20240229',
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate multi-format content
async function generateContent(research: any, format: string) {
  const content = await generator.create({
    topic: research.keyword,
    format: format, // 'toplist', 'pov', 'case-study', 'how-to'
    insights: research.insights,
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly', 'humorous', 'expert'
    length: 'medium' // 'short', 'long'
  });

  return content;
}

// Example: Generate POV article
const povContent = await generateContent(researchData, 'pov');
console.log(povContent.en); // English version
console.log(povContent.vi); // Vietnamese version
```

### 3. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer({
  licenseKey: process.env.REMOTION_LICENSE_KEY
});

// Render video from content
async function createVideo(content: any, platform: string) {
  const videoConfig = {
    template: 'infographic', // 'talking-head', 'text-animation'
    platform, // 'tiktok', 'reels', 'shorts'
    content: {
      headline: content.title,
      keyPoints: content.bulletPoints,
      stats: content.dataPoints,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    }
  };

  const composition = await renderer.prepare(videoConfig);
  const video = await renderer.render({
    composition,
    outputPath: `./output/${platform}_${Date.now()}.mp4`,
    aspectRatio: platform === 'tiktok' ? '9:16' : '1:1'
  });

  return video;
}
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'twitter'],
    apiKey: process.env.RAPIDAPI_KEY
  },
  ai: {
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  },
  video: {
    enabled: true,
    platforms: ['tiktok', 'reels']
  },
  publishing: {
    autoPost: false, // Set to true for auto-publishing
    schedule: '09:00' // UTC time
  }
});

// Execute full pipeline
async function runPipeline(keyword: string) {
  const job = await pipeline.execute({
    keyword,
    formats: ['toplist', 'pov'],
    generateVideo: true,
    targetPlatforms: ['facebook', 'linkedin', 'tiktok']
  });

  // Monitor progress
  job.on('progress', (stage) => {
    console.log(`Stage: ${stage.name}, Progress: ${stage.percent}%`);
  });

  // Wait for completion
  const results = await job.complete();
  
  return {
    articles: results.content,
    videos: results.videos,
    published: results.publishedLinks
  };
}

// Usage
const output = await runPipeline('AI marketing automation');
```

## Configuration Patterns

### Custom Content Templates

```typescript
// src/config/templates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'hook',
      'introduction',
      'items', // 5-10 items
      'conclusion',
      'cta'
    ],
    itemFormat: {
      title: true,
      description: true,
      data: true,
      visual: true
    }
  },
  
  pov: {
    structure: [
      'opinion_statement',
      'context',
      'arguments', // 3-5 points
      'counterpoints',
      'conclusion'
    ],
    tone: 'thought-leadership'
  },
  
  caseStudy: {
    structure: [
      'challenge',
      'solution',
      'implementation',
      'results',
      'takeaways'
    ],
    dataRequired: true
  }
};
```

### Multi-Language Support

```typescript
// src/lib/ai/translator.ts
import { ContentGenerator } from '@/lib/ai/generator';

async function generateBilingual(topic: string, format: string) {
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  // Generate in both languages simultaneously
  const [english, vietnamese] = await Promise.all([
    generator.create({
      topic,
      format,
      language: 'en',
      culturalContext: 'international'
    }),
    generator.create({
      topic,
      format,
      language: 'vi',
      culturalContext: 'vietnamese',
      localizeExamples: true
    })
  ]);

  return { en: english, vi: vietnamese };
}
```

### Video Template Customization

```typescript
// remotion/templates/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface InfographicProps {
  headline: string;
  stats: Array<{ label: string; value: string }>;
  colors: string[];
}

export const Infographic: React.FC<InfographicProps> = ({
  headline,
  stats,
  colors
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: colors[0] }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ fontSize: 72, color: colors[1] }}>
          {headline}
        </h1>
        <div style={{ marginTop: 40 }}>
          {stats.map((stat, i) => (
            <div key={i} style={{ marginBottom: 30 }}>
              <div style={{ fontSize: 96, fontWeight: 'bold' }}>
                {stat.value}
              </div>
              <div style={{ fontSize: 36, opacity: 0.8 }}>
                {stat.label}
              </div>
            </div>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

## Common Workflows

### Daily Trending Content Generation

```typescript
import { pipeline } from '@/lib/pipeline';
import { scheduleJob } from 'node-schedule';

// Schedule daily at 6 AM
scheduleJob('0 6 * * *', async () => {
  const trendingTopics = await pipeline.research.getTrending({
    sources: ['techcrunch', 'twitter'],
    limit: 5
  });

  for (const topic of trendingTopics) {
    await pipeline.execute({
      keyword: topic.keyword,
      formats: ['toplist'],
      generateVideo: true,
      autoPublish: true
    });
  }
});
```

### Batch Content Creation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      pipeline.execute({
        keyword,
        formats: ['pov', 'how-to'],
        generateVideo: false,
        saveToDatabase: true
      })
    )
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  return successful;
}

// Generate 10 articles
const batch = await batchGenerate([
  'AI content marketing',
  'video automation tools',
  'social media scheduling',
  // ... more keywords
]);
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

const results = await Promise.all(
  keywords.map(keyword =>
    limit(() => pipeline.execute({ keyword }))
  )
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for complex videos
const video = await renderer.render({
  composition,
  timeout: 300000, // 5 minutes
  concurrency: 2 // Reduce concurrent renders
});
```

### Claude API Errors

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  maxRetries: 3,
  timeout: 60000
});

try {
  const content = await generator.create(params);
} catch (error) {
  if (error.status === 429) {
    // Rate limited - wait and retry
    await new Promise(resolve => setTimeout(resolve, 5000));
    return generator.create(params);
  }
  throw error;
}
```

### Memory Issues with Large Datasets

```typescript
// Stream large research results
async function* streamResearch(keyword: string) {
  const pageSize = 10;
  let page = 0;
  
  while (true) {
    const results = await research.scan({
      keyword,
      limit: pageSize,
      offset: page * pageSize
    });
    
    if (results.length === 0) break;
    
    yield results;
    page++;
  }
}

// Process in chunks
for await (const chunk of streamResearch('AI marketing')) {
  await processChunk(chunk);
}
```

## Testing

```typescript
// __tests__/pipeline.test.ts
import { ContentPipeline } from '@/lib/pipeline';

describe('Content Pipeline', () => {
  it('should generate content from keyword', async () => {
    const pipeline = new ContentPipeline({
      research: { sources: ['techcrunch'] },
      ai: { provider: 'claude', apiKey: process.env.ANTHROPIC_API_KEY }
    });

    const result = await pipeline.execute({
      keyword: 'test keyword',
      formats: ['toplist']
    });

    expect(result.content).toBeDefined();
    expect(result.content.en).toHaveProperty('title');
  });
});
```

Run tests:

```bash
npm test
# or
npm run test:watch
```

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm start

# Or deploy to Vercel
vercel deploy --prod
```

This skill provides comprehensive guidance for AI agents to effectively utilize the Marketing Content Pipeline for automated content creation, from research through video generation to multi-platform publishing.
