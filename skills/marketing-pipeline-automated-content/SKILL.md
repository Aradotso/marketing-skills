---
name: marketing-pipeline-automated-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - create automated content pipeline with AI
  - generate videos from text content automatically
  - crawl news and research for content creation
  - set up AI marketing content workflow
  - automate social media content generation
  - build content pipeline with Claude and OpenAI
  - create multi-format content with AI research
  - generate video content from blog posts
---

# Marketing Pipeline Automated Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the **Ultimate AI Content Pipeline** - a TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation. The system integrates Claude 3, OpenAI, and Remotion to create a complete content production pipeline.

## What This Project Does

The Marketing Pipeline automates:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for recent data
2. **Multi-Format Content**: Generates articles in various formats (toplist, POV, case study, how-to)
3. **Bilingual Output**: Creates content in both English and Vietnamese
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Multi-Platform Export**: Optimizes content for Reels, TikTok, Shorts

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Remotion License (if applicable)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video compositions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNews } from '@/lib/crawlers/news-crawler';
import { analyzeResearch } from '@/lib/ai/research-analyzer';

async function gatherResearch(keyword: string) {
  // Crawl recent news from multiple sources
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze with Claude for insights
  const insights = await analyzeResearch(newsData, {
    model: 'claude-3-opus-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    rawData: newsData,
    insights: insights,
    statistics: insights.statistics
  };
}
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language, Tone } from '@/lib/content/types';

async function createArticle(research: any, options: {
  format: ContentFormat;
  language: Language;
  tone: Tone;
}) {
  const content = await generateContent({
    research,
    format: options.format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: options.language, // 'en' | 'vi'
    tone: options.tone, // 'professional' | 'friendly' | 'humorous'
    aiProvider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    seoKeywords: content.keywords
  };
}

// Example: Generate bilingual content
async function generateBilingualContent(keyword: string) {
  const research = await gatherResearch(keyword);
  
  const [english, vietnamese] = await Promise.all([
    createArticle(research, {
      format: 'toplist',
      language: 'en',
      tone: 'professional'
    }),
    createArticle(research, {
      format: 'toplist',
      language: 'vi',
      tone: 'friendly'
    })
  ]);

  return { english, vietnamese };
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

async function generateVideo(content: {
  title: string;
  points: string[];
  images?: string[];
}) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      images: content.images || []
    }
  });

  // Render video
  const outputLocation = `./output/video-${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps
  });

  return outputLocation;
}

// Generate platform-specific videos
async function generatePlatformVideos(content: any) {
  const platforms = [
    { name: 'reels', width: 1080, height: 1920 },
    { name: 'tiktok', width: 1080, height: 1920 },
    { name: 'youtube-shorts', width: 1080, height: 1920 }
  ];

  const videos = await Promise.all(
    platforms.map(platform => 
      generateVideo({
        ...content,
        dimensions: { width: platform.width, height: platform.height }
      })
    )
  );

  return videos;
}
```

### 4. Complete Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runCompletePipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    claudeApiKey: process.env.ANTHROPIC_API_KEY,
    openaiApiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  // Execute full pipeline
  const result = await pipeline.execute({
    keyword,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    generateVideo: true,
    platforms: ['reels', 'tiktok', 'youtube-shorts']
  });

  return {
    research: result.research,
    articles: result.articles, // Array of generated articles
    videos: result.videos,     // Array of rendered videos
    metadata: result.metadata
  };
}

// Usage
const output = await runCompletePipeline('AI marketing automation 2024');
console.log(`Generated ${output.articles.length} articles`);
console.log(`Generated ${output.videos.length} videos`);
```

## Common Patterns

### Custom Content Format

```typescript
import { defineContentFormat } from '@/lib/content/formats';

const customFormat = defineContentFormat({
  name: 'comparison',
  structure: {
    introduction: { required: true },
    comparisonTable: { required: true },
    pros: { required: true },
    cons: { required: true },
    conclusion: { required: true }
  },
  prompt: `
    Create a detailed comparison article about {topic}.
    Include a comparison table, pros and cons for each option,
    and a clear conclusion with recommendations.
  `
});

const article = await generateContent({
  research: researchData,
  format: customFormat,
  language: 'en',
  tone: 'professional'
});
```

### Scheduled Content Generation

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

// Schedule daily content generation
scheduleContentGeneration({
  keywords: ['AI trends', 'marketing automation', 'content strategy'],
  schedule: '0 9 * * *', // 9 AM daily
  formats: ['toplist', 'pov'],
  languages: ['en', 'vi'],
  onComplete: async (results) => {
    // Auto-publish or save to CMS
    await publishToWordPress(results.articles);
    await uploadToYouTube(results.videos);
  }
});
```

### Custom Video Template

```typescript
// src/remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const CustomVideoTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: '#fff', fontSize: 60 }}>{title}</h1>
      </Sequence>
      {points.map((point, i) => (
        <Sequence key={i} from={60 + i * 90} durationInFrames={90}>
          <div style={{ color: '#fff', fontSize: 40 }}>{point}</div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## CLI Commands

If the project includes CLI tools:

```bash
# Generate content from command line
npm run generate -- --keyword "AI marketing" --format toplist --lang en

# Crawl news sources
npm run crawl -- --sources techcrunch,a16z --timeframe 24h

# Render video
npm run render-video -- --input ./content.json --output ./video.mp4

# Run complete pipeline
npm run pipeline -- --keyword "marketing trends 2024" --video
```

## Development Server

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMilliseconds: 60000 // 50 requests per minute
});

async function crawlWithRateLimit(urls: string[]) {
  const results = [];
  for (const url of urls) {
    await limiter.wait();
    const data = await fetch(url);
    results.push(data);
  }
  return results;
}
```

### Video Rendering Errors

```typescript
// Ensure ffmpeg is installed
// Linux/Mac: sudo apt-get install ffmpeg
// Windows: Download from ffmpeg.org

// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  outputLocation,
  timeoutInMilliseconds: 120000 // 2 minutes
});
```

### Claude API Errors

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

try {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }]
  });
} catch (error) {
  if (error.status === 429) {
    // Rate limit - implement exponential backoff
    await new Promise(resolve => setTimeout(resolve, 5000));
  } else if (error.status === 400) {
    // Invalid request - check prompt length
    console.error('Invalid request:', error.message);
  }
}
```

### Memory Issues with Large Content

```typescript
// Process in chunks for large datasets
async function processLargeDataset(items: any[], chunkSize = 10) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(item => processItem(item))
    );
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Best Practices

1. **Always validate API keys** before starting long-running pipelines
2. **Cache research data** to avoid redundant crawling
3. **Use queues** for video rendering to prevent memory overflow
4. **Implement retry logic** for API calls with exponential backoff
5. **Monitor costs** when using paid AI APIs (Claude, OpenAI)
6. **Store generated content** with proper versioning and metadata
