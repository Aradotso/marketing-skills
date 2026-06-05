---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - generate AI content from research to video
  - automate content creation pipeline with AI
  - create marketing content with Claude and OpenAI
  - build automated content workflow
  - generate videos from text content automatically
  - scrape trending topics and create content
  - use marketing pipeline for content automation
  - setup AI content generation system
---

# Marketing Pipeline Share AI Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share**, a complete automated content creation system that handles research, scriptwriting, and video generation. The pipeline uses Claude 3/OpenAI for content generation and Remotion for video rendering.

## What This Project Does

Marketing Pipeline Share is an all-in-one content automation pipeline that:

- **Auto-scrapes** trending content from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Provides Next.js interface** for content management and scheduling

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Setup environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPID_API_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── lib/
│   │   ├── ai/              # AI integration (Claude, OpenAI)
│   │   ├── scraper/         # Content scraping modules
│   │   ├── video/           # Remotion video generation
│   │   └── utils/           # Utility functions
│   ├── pages/               # Next.js pages
│   ├── components/          # React components
│   └── remotion/            # Remotion video compositions
├── config/
│   └── content-templates.ts # Content format templates
└── scripts/                 # CLI automation scripts
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
import { researchTopic } from '@/lib/scraper';

// Scrape trending content from multiple sources
const research = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  maxResults: 10
});

console.log(research.insights); // Array of trending insights
console.log(research.data);     // Raw scraped data
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate content using Claude or OpenAI
const content = await generateContent({
  model: 'claude-3-opus', // or 'gpt-4'
  topic: 'AI automation trends',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  tone: 'professional', // 'friendly', 'humorous'
  language: 'bilingual', // 'en', 'vi'
  researchData: research.insights
});

console.log(content.title);
console.log(content.body);
console.log(content.vietnamese); // If bilingual
```

### 3. Content Format Templates

```typescript
import { ContentFormat } from '@/lib/types';

// Available content formats
const formats: ContentFormat[] = [
  {
    type: 'toplist',
    structure: {
      intro: true,
      items: 5-10,
      conclusion: true
    }
  },
  {
    type: 'pov',
    structure: {
      hook: true,
      perspective: 'expert',
      arguments: 3-5,
      cta: true
    }
  },
  {
    type: 'case-study',
    structure: {
      problem: true,
      solution: true,
      results: true,
      dataPoints: true
    }
  }
];
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/render';
import { VideoComposition } from '@/remotion/compositions';

// Render video from content
const video = await renderVideo({
  composition: VideoComposition.Infographic,
  content: {
    title: content.title,
    points: content.keyPoints,
    style: 'modern'
  },
  format: {
    platform: 'reels', // 'tiktok', 'shorts', 'instagram'
    aspectRatio: '9:16',
    duration: 30 // seconds
  },
  outputPath: './output/video.mp4'
});

console.log(video.url);
```

### 5. Complete Pipeline Execution

```typescript
import { runContentPipeline } from '@/lib/pipeline';

// Execute full pipeline: research → content → video
const result = await runContentPipeline({
  keyword: 'AI marketing automation',
  contentFormat: 'toplist',
  generateVideo: true,
  publishTo: ['draft'], // 'facebook', 'instagram', etc.
  schedule: new Date('2026-06-10T10:00:00')
});

console.log(result.contentId);
console.log(result.videoUrl);
console.log(result.status);
```

## CLI Commands

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends" --format toplist

# Research only (no content generation)
npm run research -- --keyword "marketing automation" --sources techcrunch,a16z

# Render video from existing content
npm run render-video -- --content-id abc123 --platform reels

# Run full pipeline
npm run pipeline -- --keyword "AI tools" --format pov --video true

# Start development server
npm run dev

# Build for production
npm run build
```

## Configuration

### Content Templates Configuration

```typescript
// config/content-templates.ts
export const contentConfig = {
  defaultModel: 'claude-3-opus',
  fallbackModel: 'gpt-4',
  maxTokens: 4000,
  temperature: 0.7,
  
  formats: {
    toplist: {
      minItems: 5,
      maxItems: 10,
      includeData: true
    },
    pov: {
      tone: 'expert',
      includeCounterArguments: true
    }
  },
  
  languages: {
    bilingual: {
      primary: 'en',
      secondary: 'vi',
      separateOutput: true
    }
  }
};
```

### Video Rendering Configuration

```typescript
// config/video-config.ts
export const videoConfig = {
  platforms: {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    youtube: { width: 1920, height: 1080, fps: 60 }
  },
  
  rendering: {
    codec: 'h264',
    quality: 'high',
    concurrency: 4
  },
  
  storage: {
    provider: 's3', // or 'local'
    bucket: process.env.S3_BUCKET
  }
};
```

## Common Patterns

### Pattern 1: Multi-Source Research with Aggregation

```typescript
import { aggregateResearch } from '@/lib/scraper/aggregator';

const multiSourceResearch = async (keyword: string) => {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await Promise.all(
    sources.map(source => 
      researchTopic({ keyword, sources: [source], maxResults: 5 })
    )
  );
  
  return aggregateResearch(results, {
    deduplicate: true,
    scoreRelevance: true,
    topN: 10
  });
};
```

### Pattern 2: Batch Content Generation

```typescript
import { batchGenerate } from '@/lib/ai/batch-generator';

const createContentSeries = async (topics: string[]) => {
  return await batchGenerate({
    topics,
    format: 'toplist',
    model: 'claude-3-opus',
    parallel: 3, // Generate 3 at a time
    onProgress: (completed, total) => {
      console.log(`Generated ${completed}/${total}`);
    }
  });
};
```

### Pattern 3: Scheduled Content Pipeline

```typescript
import { scheduleContent } from '@/lib/scheduler';

const setupAutomation = async () => {
  await scheduleContent({
    keywords: ['AI trends', 'marketing automation', 'content creation'],
    frequency: 'daily',
    time: '09:00',
    pipeline: {
      research: true,
      generate: true,
      video: true,
      publish: 'draft'
    }
  });
};
```

### Pattern 4: Custom Video Composition

```typescript
import { Composition } from 'remotion';

// Create custom video template
export const CustomInfograp: React.FC<{content: Content}> = ({content}) => {
  return (
    <Composition
      id="custom-infographic"
      component={InfograpTemplate}
      durationInFrames={900} // 30 seconds at 30fps
      fps={30}
      width={1080}
      height={1920}
      defaultProps={{
        title: content.title,
        points: content.keyPoints,
        theme: 'dark'
      }}
    />
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { withRetry } from '@/lib/utils/retry';

// Automatic retry with exponential backoff
const generateWithRetry = withRetry(
  () => generateContent({...}),
  {
    maxRetries: 3,
    backoff: 'exponential',
    onRetry: (error, attempt) => {
      console.log(`Retry attempt ${attempt}: ${error.message}`);
    }
  }
);
```

### Content Quality Issues

```typescript
// Use content validation
import { validateContent } from '@/lib/utils/validator';

const content = await generateContent({...});

const validation = validateContent(content, {
  minLength: 800,
  maxLength: 2000,
  requireData: true,
  checkReadability: true
});

if (!validation.valid) {
  // Regenerate with adjustments
  const improvedContent = await generateContent({
    ...options,
    instructions: validation.suggestions.join('. ')
  });
}
```

### Video Rendering Failures

```typescript
// Check Remotion logs and use local fallback
try {
  const video = await renderVideo({...});
} catch (error) {
  console.error('Cloud render failed:', error);
  
  // Fallback to local rendering
  const localVideo = await renderVideo({
    ...options,
    renderMode: 'local',
    outputDir: './local-renders'
  });
}
```

### Scraper Blocking

```typescript
// Use rotation and delays
import { ScraperConfig } from '@/lib/scraper/config';

const safeScraperConfig: ScraperConfig = {
  userAgentRotation: true,
  proxyEnabled: true,
  requestDelay: 2000, // 2 seconds between requests
  respectRobotsTxt: true,
  maxConcurrent: 2
};
```

## Best Practices

1. **Always validate API keys** before running pipeline
2. **Cache research data** to avoid redundant API calls
3. **Use bilingual mode** sparingly (doubles token usage)
4. **Monitor video rendering costs** (AWS/cloud charges)
5. **Test content formats** with small batches first
6. **Set up error notifications** for production pipelines
7. **Version control your templates** separately from code
