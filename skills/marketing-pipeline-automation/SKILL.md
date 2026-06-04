---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to script writing to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline
  - generate marketing content with AI
  - create videos from blog posts automatically
  - research and write content using Claude
  - build automated marketing workflow
  - generate multilingual content with AI
  - create social media videos from articles
  - set up content automation system
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use an automated content pipeline that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is a full-stack TypeScript/Next.js system that automates content creation from end to end:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, X, LinkedIn) for fresh data within 24 hours
2. **AI Content Generation**: Uses Claude/OpenAI to write articles in multiple formats (toplist, POV, case study, how-to)
3. **Multilingual Output**: Generates content in both English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographics and short videos using Remotion
5. **Multi-Platform Export**: Outputs video in formats optimized for Reels, TikTok, and YouTube Shorts

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
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion for video rendering
REMOTION_LICENSE_KEY=your_remotion_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Application settings
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
│   │   ├── scraper/     # News scraping logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core API Usage

### 1. Research & Scraping

```typescript
import { NewsScraperService } from '@/lib/scraper/news-scraper';
import { ResearchAggregator } from '@/lib/scraper/research-aggregator';

// Initialize scraper
const scraper = new NewsScraperService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch recent news by keyword
const research = await scraper.fetchNews({
  keyword: 'AI automation',
  timeframe: '24h',
  limit: 10
});

// Aggregate and analyze insights
const aggregator = new ResearchAggregator();
const insights = await aggregator.analyze(research);

console.log(insights);
// {
//   trends: [...],
//   dataPoints: [...],
//   keyInsights: [...],
//   sources: [...]
// }
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { ContentGenerator } from '@/lib/generator/content-generator';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const generator = new ContentGenerator(anthropic);

// Generate content in multiple formats
const content = await generator.generate({
  keyword: 'Marketing Automation',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'en', // 'en' | 'vi'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  researchData: insights,
  includeDataPoints: true
});

console.log(content);
// {
//   title: "Top 10 Marketing Automation Tools in 2024",
//   body: "...",
//   metadata: { ... },
//   sources: [...]
// }
```

### 3. Generate Vietnamese Content

```typescript
// Generate parallel Vietnamese version
const contentVi = await generator.generate({
  keyword: 'Marketing Automation',
  format: 'toplist',
  language: 'vi',
  tone: 'friendly',
  researchData: insights,
  includeDataPoints: true
});

console.log(contentVi.title);
// "Top 10 Công Cụ Tự Động Hóa Marketing Năm 2024"
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

// Prepare video data from article
const videoData = {
  title: content.title,
  keyPoints: content.body.split('\n').slice(0, 5),
  theme: 'modern',
  duration: 30, // seconds
  format: 'vertical' // 'vertical' | 'square' | 'horizontal'
};

// Bundle Remotion composition
const bundleLocation = await bundle({
  entryPoint: './remotion/index.ts',
  webpackOverride: (config) => config,
});

// Select composition
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
  inputProps: videoData,
});

// Render video
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: `out/${content.title.replace(/\s+/g, '-')}.mp4`,
  inputProps: videoData,
});

console.log('Video rendered successfully!');
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

// Initialize pipeline with all services
const pipeline = new ContentPipeline({
  openaiKey: process.env.OPENAI_API_KEY!,
  anthropicKey: process.env.ANTHROPIC_API_KEY!,
  rapidApiKey: process.env.RAPIDAPI_KEY!,
});

// Run complete automation
const result = await pipeline.execute({
  keyword: 'AI Marketing Tools',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormats: ['vertical', 'square'],
  autoPublish: false
});

console.log(result);
// {
//   articles: [
//     { id: '...', title: '...', url: '...', language: 'en' },
//     { id: '...', title: '...', url: '...', language: 'vi' }
//   ],
//   videos: [
//     { id: '...', format: 'vertical', path: '...' },
//     { id: '...', format: 'square', path: '...' }
//   ],
//   research: { ... }
// }
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// app/api/content/create/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, formats, languages, generateVideo } = body;

    const pipeline = new ContentPipeline({
      openaiKey: process.env.OPENAI_API_KEY!,
      anthropicKey: process.env.ANTHROPIC_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });

    const result = await pipeline.execute({
      keyword,
      formats,
      languages,
      generateVideo,
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { NewsScraperService } from '@/lib/scraper/news-scraper';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const timeframe = searchParams.get('timeframe') || '24h';

  const scraper = new NewsScraperService({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const research = await scraper.fetchNews({
    keyword: keyword!,
    timeframe,
    limit: 20
  });

  return NextResponse.json(research);
}
```

## Common Patterns

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/lib/generator/templates';

// Create custom template
const customTemplate: ContentTemplate = {
  id: 'product-launch',
  name: 'Product Launch Announcement',
  structure: {
    sections: [
      'hook',
      'problem',
      'solution',
      'features',
      'benefits',
      'cta'
    ],
    tone: 'exciting',
    length: 'medium'
  },
  prompts: {
    en: "Create a product launch announcement for {keyword}...",
    vi: "Tạo thông báo ra mắt sản phẩm cho {keyword}..."
  }
};

// Use custom template
const generator = new ContentGenerator(anthropic);
generator.addTemplate(customTemplate);

const content = await generator.generate({
  keyword: 'AI Content Tool',
  format: 'product-launch',
  language: 'en'
});
```

### Batch Processing

```typescript
// Process multiple keywords in parallel
const keywords = ['AI Marketing', 'Content Automation', 'Video Generation'];

const results = await Promise.all(
  keywords.map(keyword =>
    pipeline.execute({
      keyword,
      formats: ['toplist'],
      languages: ['en', 'vi'],
      generateVideo: true
    })
  )
);

console.log(`Processed ${results.length} topics`);
```

### Scheduling Content Generation

```typescript
import { CronJob } from 'cron';

// Schedule daily content generation
const job = new CronJob('0 9 * * *', async () => {
  const trendingTopics = await scraper.getTrendingTopics();
  
  for (const topic of trendingTopics.slice(0, 3)) {
    await pipeline.execute({
      keyword: topic.name,
      formats: ['toplist', 'how-to'],
      languages: ['en'],
      generateVideo: true
    });
  }
});

job.start();
```

## Remotion Video Customization

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
  theme: 'modern' | 'minimal' | 'bold';
}> = ({ title, keyPoints, theme }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: theme === 'modern' ? '#1a1a1a' : '#ffffff',
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <h1 style={{ fontSize: 60, color: theme === 'modern' ? '#fff' : '#000' }}>
        {title}
      </h1>
      <ul>
        {keyPoints.map((point, i) => (
          <li key={i} style={{ fontSize: 30, marginTop: 20 }}>
            {point}
          </li>
        ))}
      </ul>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(params: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generator.generate(params);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Memory Issues with Video Rendering

```typescript
// Process videos sequentially to avoid memory overload
async function renderVideosSequentially(contents: any[]) {
  const videos = [];
  
  for (const content of contents) {
    const video = await renderVideo(content);
    videos.push(video);
    
    // Clear cache between renders
    if (global.gc) {
      global.gc();
    }
  }
  
  return videos;
}
```

### Research Data Quality

```typescript
// Filter and validate scraped data
function validateResearch(data: any[]) {
  return data.filter(item => {
    return (
      item.title &&
      item.content &&
      item.publishedAt &&
      new Date(item.publishedAt) > new Date(Date.now() - 24 * 60 * 60 * 1000)
    );
  });
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run tests
npm run test

# Render Remotion video locally
npm run render

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Best Practices

1. **Always validate environment variables on startup**
2. **Cache research data to reduce API calls**
3. **Use streaming for large content generation**
4. **Implement proper error boundaries in Next.js components**
5. **Monitor API usage and costs**
6. **Store generated content in a database for reuse**
7. **Use queues (Bull, BullMQ) for video rendering tasks**
8. **Implement webhook notifications for pipeline completion**
