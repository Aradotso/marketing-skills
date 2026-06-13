---
name: ai-content-pipeline-automation
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline
  - generate videos from text automatically
  - crawl news and create content with AI
  - use remotion for automated video creation
  - build a content automation system
  - create multi-language content with Claude
  - automate social media content production
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive content automation system that handles research, scriptwriting, multi-language content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

The AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scrapes research** from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
- **Generates multi-format content** (toplists, POV articles, case studies, how-tos) in multiple languages
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

The pipeline transforms a single keyword input into complete content pieces with data-backed insights, ready for publication.

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

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_API_KEY=your_linkedin_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
DATABASE_URL=your_database_connection_string
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawlers
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript types
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research Crawling

```typescript
import { ResearchCrawler } from '@/lib/crawlers/research-crawler';

// Initialize crawler
const crawler = new ResearchCrawler({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl for keyword
const researchData = await crawler.crawl({
  keyword: 'AI automation',
  timeRange: '24h',
  maxResults: 50,
  filterQuality: true
});

console.log(researchData.insights); // Extracted insights
console.log(researchData.dataPoints); // Statistics and metrics
```

### 2. Content Generation with Claude

```typescript
import { ClaudeContentGenerator } from '@/lib/ai/claude-generator';

const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-5-sonnet-20241022'
});

// Generate multi-language content
const content = await generator.generate({
  topic: 'AI Content Automation Trends',
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  languages: ['en', 'vi'],
  tone: 'expert', // or 'friendly', 'humorous'
  researchData: researchData,
  wordCount: 1500
});

// Content structure
console.log(content.en.title);
console.log(content.en.body);
console.log(content.en.keyPoints);
console.log(content.vi.title);
```

### 3. OpenAI Alternative

```typescript
import { OpenAIContentGenerator } from '@/lib/ai/openai-generator';

const generator = new OpenAIContentGenerator({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

const content = await generator.generate({
  topic: 'Marketing Automation',
  format: 'how-to',
  language: 'en',
  context: researchData
});
```

### 4. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { ContentToVideoComposition } from '@/remotion/compositions/ContentVideo';

// Render video from content
const videoResult = await renderVideo({
  composition: ContentToVideoComposition,
  inputProps: {
    title: content.en.title,
    keyPoints: content.en.keyPoints,
    duration: 60, // seconds
    style: 'modern',
    platform: 'reels' // or 'tiktok', 'shorts'
  },
  outputPath: './output/video.mp4',
  format: 'mp4',
  codec: 'h264',
  fps: 30
});

console.log(videoResult.outputFile);
```

## Common Patterns

### Full Pipeline Execution

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
  remotionLicense: process.env.REMOTION_LICENSE_KEY
});

// Execute complete pipeline
const result = await pipeline.execute({
  keyword: 'SaaS Marketing Trends 2024',
  contentFormats: ['toplist', 'case-study'],
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['reels', 'tiktok']
});

// Access all outputs
result.research;     // Crawled data
result.content;      // Generated articles
result.videos;       // Rendered videos
result.scheduleData; // Publishing metadata
```

### Batch Content Creation

```typescript
const keywords = [
  'AI Marketing Tools',
  'Content Automation',
  'Video Marketing Trends'
];

const batchResults = await Promise.all(
  keywords.map(keyword => 
    pipeline.execute({
      keyword,
      contentFormats: ['toplist'],
      languages: ['en'],
      generateVideo: true
    })
  )
);

// Process results
batchResults.forEach((result, index) => {
  console.log(`Content ${index + 1}: ${result.content.en.title}`);
  console.log(`Video: ${result.videos[0].path}`);
});
```

### Custom Research Sources

```typescript
import { CustomSourceCrawler } from '@/lib/crawlers/custom-source';

const customCrawler = new CustomSourceCrawler();

// Add custom RSS feed
customCrawler.addSource({
  type: 'rss',
  url: 'https://your-blog.com/feed',
  parser: 'default'
});

// Add custom API endpoint
customCrawler.addSource({
  type: 'api',
  endpoint: 'https://api.yourservice.com/articles',
  headers: {
    'Authorization': `Bearer ${process.env.CUSTOM_API_KEY}`
  },
  transform: (data) => ({
    title: data.headline,
    content: data.body,
    publishedAt: data.date
  })
});

const data = await customCrawler.crawl('AI trends');
```

### Content Formatting & Export

```typescript
import { ContentFormatter } from '@/lib/formatters/content-formatter';

const formatter = new ContentFormatter();

// Export to Markdown
const markdown = formatter.toMarkdown(content.en);
await fs.writeFile('./output/article.md', markdown);

// Export to HTML
const html = formatter.toHTML(content.en, {
  includeCSS: true,
  template: 'blog-post'
});

// Export to JSON for API
const json = formatter.toJSON(content, {
  includeMeta: true,
  includeAnalytics: true
});
```

## Video Templates

### Creating Custom Video Compositions

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomVideoTemplate: React.FC<{
  title: string;
  keyPoints: string[];
}> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ padding: 60 }}>
        <h1 style={{ 
          fontSize: 64, 
          color: 'white',
          opacity: Math.min(1, frame / 30)
        }}>
          {title}
        </h1>
        <ul>
          {keyPoints.map((point, i) => (
            <li key={i} style={{
              fontSize: 32,
              color: '#00ff88',
              opacity: frame > (i + 1) * 30 ? 1 : 0,
              transition: 'opacity 0.5s'
            }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

### Registering Custom Composition

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { CustomVideoTemplate } from './compositions/CustomTemplate';

registerRoot(() => {
  return (
    <>
      <Composition
        id="CustomVideo"
        component={CustomVideoTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
});
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render video (Remotion)
npm run remotion render ContentVideo output/video.mp4

# Preview video composition
npm run remotion preview

# Run content pipeline (custom script)
npm run pipeline -- --keyword "AI Marketing" --format toplist

# Batch process keywords
npm run batch -- --file keywords.txt --output ./content
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();
  
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  const result = await pipeline.execute({
    keyword,
    contentFormats: [format],
    languages,
    generateVideo: false
  });
  
  return NextResponse.json(result);
}
```

### Video Render Endpoint

```typescript
// src/app/api/render-video/route.ts
export async function POST(request: NextRequest) {
  const { contentId, platform } = await request.json();
  
  const content = await getContentById(contentId);
  
  const videoResult = await renderVideo({
    composition: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      platform
    },
    outputPath: `./public/videos/${contentId}.mp4`
  });
  
  return NextResponse.json({
    videoUrl: `/videos/${contentId}.mp4`
  });
}
```

## Configuration Options

### Content Generation Settings

```typescript
// src/config/content.config.ts
export const contentConfig = {
  ai: {
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022',
    temperature: 0.7,
    maxTokens: 4000
  },
  formats: {
    toplist: {
      itemCount: 10,
      includeRatings: true,
      includeImages: true
    },
    caseStudy: {
      sections: ['challenge', 'solution', 'results'],
      includeMetrics: true
    },
    howTo: {
      stepCount: 7,
      includeVisuals: true
    }
  },
  languages: {
    en: {
      tone: 'professional',
      locale: 'en-US'
    },
    vi: {
      tone: 'friendly',
      locale: 'vi-VN'
    }
  }
};
```

### Video Rendering Settings

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);

export const videoConfig = {
  platforms: {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 },
    landscape: { width: 1920, height: 1080, fps: 30 }
  },
  encoding: {
    codec: 'h264',
    quality: 80,
    bitrate: '5M'
  }
};
```

## Troubleshooting

### Rate Limiting Issues

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

// Wrap API calls
await limiter.execute(async () => {
  return await crawler.crawl({ keyword });
});
```

### Video Rendering Fails

```bash
# Check Remotion installation
npx remotion versions

# Clear cache
rm -rf node_modules/.cache/remotion

# Re-install dependencies
npm install @remotion/cli@latest @remotion/renderer@latest
```

### AI Generation Errors

```typescript
import { retryWithBackoff } from '@/lib/utils/retry';

const content = await retryWithBackoff(
  async () => generator.generate({ topic, format }),
  {
    maxRetries: 3,
    initialDelay: 1000,
    backoffMultiplier: 2
  }
);
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processBatch(keywords: string[], chunkSize = 5) {
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    await Promise.all(chunk.map(k => pipeline.execute({ keyword: k })));
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
}
```

This skill provides comprehensive coverage of the AI Content Pipeline automation system, enabling AI coding agents to effectively assist developers in building automated content creation workflows.
