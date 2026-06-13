---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing content from trending news
  - create social media videos automatically
  - build AI content pipeline with Remotion
  - scrape and research content with Claude API
  - automate blog post and video generation
  - set up marketing automation workflow
  - create multi-format content with AI
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use **marketing-pipeline-share**, a complete AI-powered content automation system that handles research, scriptwriting, auto-posting, and video generation. The pipeline crawls trending news, generates multi-format content using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending topics (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media Auto-posting
FACEBOOK_PAGE_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── services/         # Core services
│   │   ├── research/     # News scraping & analysis
│   │   ├── generation/   # AI content generation
│   │   └── video/        # Remotion video rendering
│   ├── lib/              # Utilities
│   └── types/            # TypeScript types
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Key API Patterns

### 1. Research Service - Auto News Scraping

```typescript
import { ResearchService } from '@/services/research/ResearchService';

const researchService = new ResearchService({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch trending topics from last 24 hours
const trendingData = await researchService.fetchTrending({
  keyword: 'AI automation',
  timeRange: '24h',
  limit: 10
});

// Analyze and extract insights
const insights = await researchService.analyzeInsights(trendingData);

console.log(insights);
// {
//   mainTopics: ['AI agents', 'automation tools'],
//   dataPoints: [{ stat: '90% time saved', source: 'techcrunch' }],
//   trendingAngles: ['productivity', 'cost reduction']
// }
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/services/generation/ContentGenerator';

const generator = new ContentGenerator({
  claudeApiKey: process.env.ANTHROPIC_API_KEY,
  openaiApiKey: process.env.OPENAI_API_KEY,
  preferredModel: 'claude-3-sonnet'
});

// Generate multi-format content
const content = await generator.generate({
  topic: 'AI Content Automation Trends 2026',
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  tone: 'expert', // or 'friendly', 'humorous'
  languages: ['en', 'vi'],
  insights: insights, // from research step
  includeDataPoints: true
});

console.log(content);
// {
//   en: {
//     title: 'Top 10 AI Content Automation Tools in 2026',
//     body: '...',
//     meta: { wordCount: 1500, readTime: '6 min' }
//   },
//   vi: {
//     title: '10 Công Cụ Tự Động Hóa Nội Dung AI Hàng Đầu 2026',
//     body: '...',
//     meta: { wordCount: 1600, readTime: '7 phút' }
//   }
// }
```

### 3. Video Rendering with Remotion

```typescript
import { VideoRenderer } from '@/services/video/VideoRenderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer();

// Render infographic video
const videoConfig = {
  composition: 'ContentInfographic',
  props: {
    title: content.en.title,
    dataPoints: insights.dataPoints,
    theme: 'modern',
    duration: 30 // seconds
  },
  outputFormat: 'mp4' as const,
  dimensions: {
    width: 1080,
    height: 1920 // Vertical for Reels/TikTok
  }
};

const videoPath = await renderer.render(videoConfig);
console.log(`Video saved to: ${videoPath}`);
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/services/ContentPipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Run full automation pipeline
const result = await pipeline.execute({
  keyword: 'marketing automation',
  contentFormats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormats: ['reels', 'tiktok'],
  autoPost: false // Set true to auto-post to social media
});

console.log(result);
// {
//   research: { insights, sources },
//   content: { en: {...}, vi: {...} },
//   videos: [
//     { format: 'reels', path: '/outputs/reels_001.mp4' },
//     { format: 'tiktok', path: '/outputs/tiktok_001.mp4' }
//   ],
//   status: 'completed'
// }
```

## Development Commands

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview (for video templates)
npm run remotion

# Render specific video composition
npm run render -- --composition=ContentInfographic --props='{"title":"Test"}'

# Type checking
npm run type-check

# Linting
npm run lint
```

## Custom Content Formats

### Creating a New Format Template

```typescript
// src/services/generation/formats/CustomFormat.ts
import { ContentFormat } from '@/types/content';

export class CustomFormatGenerator implements ContentFormat {
  name = 'comparison';
  
  async generate(params: {
    topic: string;
    insights: any;
    language: 'en' | 'vi';
  }): Promise<string> {
    const prompt = `
      Create a comparison article about ${params.topic}.
      Use these insights: ${JSON.stringify(params.insights)}
      Language: ${params.language}
      
      Structure:
      1. Introduction
      2. Option A vs Option B (comparison table)
      3. Pros and Cons
      4. Recommendation
      5. Conclusion
    `;
    
    // Use AI service to generate
    const response = await this.aiService.generate(prompt);
    return response;
  }
}
```

### Register Custom Format

```typescript
// src/services/generation/ContentGenerator.ts
import { CustomFormatGenerator } from './formats/CustomFormat';

const generator = new ContentGenerator({
  claudeApiKey: process.env.ANTHROPIC_API_KEY
});

// Add custom format
generator.registerFormat(new CustomFormatGenerator());

// Use it
const content = await generator.generate({
  topic: 'AI Tools',
  format: 'comparison' // Your custom format
});
```

## Remotion Video Templates

### Basic Infographic Template

```typescript
// remotion/compositions/ContentInfographic.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { interpolate } from 'remotion';

interface Props {
  title: string;
  dataPoints: Array<{ stat: string; label: string }>;
  theme: 'modern' | 'minimal';
}

export const ContentInfographic: React.FC<Props> = ({
  title,
  dataPoints,
  theme
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#0a0a0a' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ 
          opacity: titleOpacity,
          fontSize: 64,
          color: 'white',
          textAlign: 'center',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>
      
      <Sequence from={60}>
        <div style={{ padding: 40 }}>
          {dataPoints.map((point, idx) => (
            <DataPointCard
              key={idx}
              stat={point.stat}
              label={point.label}
              delay={idx * 20}
            />
          ))}
        </div>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### Register Remotion Composition

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { Composition } from 'remotion';
import { ContentInfographic } from './compositions/ContentInfographic';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentInfographic"
        component={ContentInfographic}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'AI Trends 2026',
          dataPoints: [],
          theme: 'modern'
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

## Configuration Options

### Pipeline Configuration

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20,
    enableCache: true,
    cacheDuration: 3600 // 1 hour
  },
  
  generation: {
    defaultModel: 'claude-3-sonnet',
    fallbackModel: 'gpt-4',
    maxTokens: 4000,
    temperature: 0.7,
    formats: ['toplist', 'pov', 'case-study', 'how-to']
  },
  
  video: {
    defaultFormat: 'mp4',
    quality: 'high',
    fps: 30,
    presets: {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      youtube: { width: 1920, height: 1080 }
    }
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  const results = await Promise.all(
    keywords.map(keyword => 
      pipeline.execute({
        keyword,
        contentFormats: ['toplist'],
        languages: ['en'],
        generateVideo: false
      })
    )
  );
  
  return results;
}

// Usage
const keywords = ['AI automation', 'marketing tools', 'content strategy'];
const batchResults = await generateBatchContent(keywords);
```

### Scheduled Content Pipeline

```typescript
import { CronJob } from 'cron';

const dailyContentJob = new CronJob('0 9 * * *', async () => {
  console.log('Starting daily content generation...');
  
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  const result = await pipeline.execute({
    keyword: 'trending marketing news',
    contentFormats: ['toplist', 'pov'],
    languages: ['en', 'vi'],
    generateVideo: true,
    autoPost: true
  });
  
  console.log('Content generated:', result);
});

dailyContentJob.start();
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/retry';

const researchData = await retry(
  () => researchService.fetchTrending({ keyword: 'AI' }),
  {
    maxAttempts: 3,
    delayMs: 1000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
const videoPath = await renderer.render(videoConfig, {
  timeoutInMilliseconds: 300000, // 5 minutes
  verbose: true
});
```

### Memory Issues with Large Content

```typescript
// Stream content generation instead of loading all at once
async function* streamContentGeneration(topics: string[]) {
  for (const topic of topics) {
    const content = await generator.generate({ topic, format: 'toplist' });
    yield content;
  }
}

// Usage
for await (const content of streamContentGeneration(topics)) {
  await saveContent(content);
}
```

### Claude API Errors

```typescript
// Handle Claude-specific errors
try {
  const content = await generator.generate({ topic: 'AI' });
} catch (error) {
  if (error.code === 'overloaded_error') {
    // Fallback to OpenAI
    const content = await generator.generateWithOpenAI({ topic: 'AI' });
  } else if (error.code === 'rate_limit_error') {
    // Wait and retry
    await new Promise(resolve => setTimeout(resolve, 60000));
    return retry();
  }
  throw error;
}
```

## Advanced Usage

### Custom Research Sources

```typescript
import { ResearchSource } from '@/services/research/ResearchSource';

class CustomNewsSource implements ResearchSource {
  async fetch(keyword: string): Promise<Article[]> {
    // Your custom scraping logic
    const response = await fetch(`https://custom-api.com/search?q=${keyword}`);
    const data = await response.json();
    
    return data.articles.map(article => ({
      title: article.title,
      url: article.link,
      excerpt: article.summary,
      publishedAt: new Date(article.date)
    }));
  }
}

// Register custom source
researchService.addSource(new CustomNewsSource());
```

This skill provides comprehensive coverage of the marketing-pipeline-share project, enabling AI agents to help developers automate content creation workflows effectively.
