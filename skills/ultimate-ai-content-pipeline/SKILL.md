---
name: ultimate-ai-content-pipeline
description: Vietnamese-language AI content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video
  - set up Vietnamese AI content pipeline
  - crawl news and generate marketing content automatically
  - create video content from AI-written articles
  - build automated content workflow with Claude and Remotion
  - generate multilingual marketing content with AI
  - automate social media content from research to video
  - set up AI-powered content automation system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive Vietnamese/English AI content automation system that handles the entire content creation pipeline: automated research/crawling from news sources, AI-powered script generation (Claude/OpenAI), and automatic video rendering (Remotion). Designed for marketers and content creators to automate 90% of their workflow.

## What It Does

This TypeScript/Next.js project provides:

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for breaking news (last 24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Support**: Generates Vietnamese and English versions simultaneously
- **Video Automation**: Renders infographics and short-form videos via Remotion
- **Multi-Platform Optimization**: Exports video ratios for Reels, TikTok, Shorts

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

### Required Environment Variables

```bash
# AI Service Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database if using persistent storage
DATABASE_URL=your_database_connection_string

# Remotion Configuration (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations (Claude, OpenAI)
│   │   ├── crawler/     # News crawling modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video templates
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── package.json
```

## Core APIs and Usage

### 1. Research & Data Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-scraper';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

// Crawl news from multiple sources
const researchData = await crawlNews({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

// Extract insights using AI
const insights = await analyzeInsights({
  data: researchData,
  provider: 'claude', // or 'openai'
  model: 'claude-3-opus-20240229'
});
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';

// Generate article with Claude or OpenAI
const article = await generateContent({
  topic: 'Top 10 AI Tools for Marketers in 2024',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  insights: insights,
  languages: ['vi', 'en'],
  tone: 'professional', // 'friendly', 'humorous'
  provider: 'claude',
  model: 'claude-3-opus-20240229'
});

// Output structure
interface GeneratedContent {
  title: {
    vi: string;
    en: string;
  };
  content: {
    vi: string;
    en: string;
  };
  metadata: {
    keywords: string[];
    summary: string;
    readingTime: number;
  };
  sources: Array<{
    title: string;
    url: string;
    publishedAt: string;
  }>;
}
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

// Render video from article content
const videoConfig = {
  compositionId: 'ArticleToVideo',
  inputProps: {
    title: article.title.vi,
    content: article.content.vi,
    style: 'infographic', // 'slideshow', 'animated-text'
    duration: 60, // seconds
  },
  outputFormat: 'mp4',
  aspectRatio: '9:16', // TikTok/Reels format
};

const videoPath = await renderVideo(videoConfig);
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automation: research → content → video
const pipeline = new ContentPipeline({
  aiProvider: 'claude',
  crawlSources: ['techcrunch', 'a16z'],
  outputLanguages: ['vi', 'en'],
});

const result = await pipeline.execute({
  keyword: 'marketing automation',
  contentFormat: 'toplist',
  generateVideo: true,
  videoFormat: 'reels',
});

// Result structure
interface PipelineResult {
  research: {
    articles: Array<any>;
    insights: string[];
  };
  content: GeneratedContent;
  video?: {
    path: string;
    duration: number;
    format: string;
  };
  metadata: {
    timestamp: string;
    processingTime: number;
  };
}
```

## Configuration Patterns

### Custom AI Prompts

```typescript
// src/lib/ai/prompts.ts
export const customPrompts = {
  vietnamese: {
    toplist: `Bạn là một chuyên gia marketing. Viết bài Top 10 về {topic}.
    Sử dụng dữ liệu: {insights}
    Giọng văn: {tone}
    Yêu cầu: Dễ đọc, có số liệu, kết thúc bằng CTA.`,
    
    pov: `Viết bài quan điểm cá nhân về {topic}...`,
  },
  english: {
    toplist: `You are a marketing expert. Write a Top 10 article about {topic}...`,
  }
};

// Use in content generation
const article = await generateContent({
  topic: 'AI Tools',
  customPrompt: customPrompts.vietnamese.toplist,
  // ...other options
});
```

### Crawler Configuration

```typescript
// src/lib/crawler/config.ts
export const crawlerConfig = {
  techcrunch: {
    baseUrl: 'https://techcrunch.com',
    apiEndpoint: '/wp-json/wp/v2/posts',
    rateLimit: 100, // requests per hour
  },
  twitter: {
    useRapidAPI: true,
    endpoint: process.env.RAPIDAPI_TWITTER_ENDPOINT,
    searchParams: {
      count: 20,
      lang: 'en',
    }
  }
};
```

### Remotion Video Templates

```typescript
// src/remotion/ToplistVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ToplistVideo: React.FC<{
  items: string[];
  title: string;
  style: 'minimal' | 'vibrant';
}> = ({ items, title, style }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <h1 style={{ 
        fontSize: 60, 
        color: '#fff',
        opacity: Math.min(1, frame / fps)
      }}>
        {title}
      </h1>
      {items.map((item, i) => (
        <div key={i} style={{ 
          opacity: frame > (i + 1) * fps ? 1 : 0,
          transition: 'opacity 0.5s'
        }}>
          {i + 1}. {item}
        </div>
      ))}
    </AbsoluteFill>
  );
};
```

## CLI Commands (if applicable)

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion:render

# Run content pipeline from CLI
npm run pipeline -- --keyword "AI marketing" --format toplist --video
```

## Common Workflows

### Daily Content Automation

```typescript
// scripts/daily-content.ts
import { ContentPipeline } from '@/lib/pipeline';
import { schedulePost } from '@/lib/social-media';

async function dailyContentJob() {
  const topics = ['AI trends', 'Marketing automation', 'Social media tips'];
  
  for (const topic of topics) {
    const pipeline = new ContentPipeline({
      aiProvider: 'claude',
      crawlSources: ['techcrunch', 'linkedin'],
      outputLanguages: ['vi', 'en'],
    });
    
    const result = await pipeline.execute({
      keyword: topic,
      contentFormat: 'toplist',
      generateVideo: true,
    });
    
    // Schedule to social media
    await schedulePost({
      platform: 'facebook',
      content: result.content.vi.substring(0, 500),
      video: result.video?.path,
      scheduledTime: new Date(Date.now() + 3600000), // 1 hour later
    });
  }
}

// Run with cron or task scheduler
dailyContentJob();
```

### Multi-Format Content from Single Research

```typescript
async function multiFormatGeneration(keyword: string) {
  const research = await crawlNews({ keyword, sources: ['techcrunch'] });
  const insights = await analyzeInsights({ data: research });
  
  const formats = ['toplist', 'pov', 'how-to'] as const;
  const outputs = await Promise.all(
    formats.map(format => 
      generateContent({
        topic: keyword,
        format,
        insights,
        languages: ['vi', 'en'],
        provider: 'claude',
      })
    )
  );
  
  return outputs; // Returns 3 different article formats
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

const article = await retry(
  () => generateContent({ /* config */ }),
  {
    maxAttempts: 3,
    delayMs: 1000,
    backoff: 'exponential',
  }
);
```

### Claude/OpenAI Token Limits

```typescript
// Chunk large content for processing
import { chunkText } from '@/lib/utils/text';

const largeInsights = crawlResults.map(r => r.content).join('\n');
const chunks = chunkText(largeInsights, 3000); // tokens

const processedChunks = await Promise.all(
  chunks.map(chunk => 
    analyzeInsights({ data: chunk, provider: 'claude' })
  )
);

const combinedInsights = processedChunks.flat();
```

### Remotion Rendering Issues

```typescript
// Use server-side rendering for heavy videos
import { renderMediaOnLambda } from '@remotion/lambda';

const { renderId, bucketName } = await renderMediaOnLambda({
  region: 'us-east-1',
  functionName: process.env.REMOTION_LAMBDA_FUNCTION,
  composition: 'ArticleToVideo',
  inputProps: videoConfig.inputProps,
  codec: 'h264',
  imageFormat: 'jpeg',
});

// Monitor progress
const progress = await getRenderProgress({ renderId, bucketName });
```

### Vietnamese Character Encoding

```typescript
// Ensure proper UTF-8 handling
const article = await generateContent({
  topic: 'Công nghệ AI',
  languages: ['vi'],
  encoding: 'utf-8',
});

// Save with correct encoding
import fs from 'fs/promises';
await fs.writeFile(
  'output.txt', 
  article.content.vi, 
  { encoding: 'utf-8' }
);
```

## Best Practices

1. **Cache Research Data**: Store crawled news for 24h to avoid redundant API calls
2. **Queue Video Rendering**: Use job queue (Bull, BullMQ) for Remotion renders to avoid blocking
3. **Monitor AI Costs**: Track Claude/OpenAI token usage per request
4. **Content Validation**: Run plagiarism checks and fact-verification on generated content
5. **A/B Test Formats**: Track engagement metrics per content format to optimize pipeline

## Integration Examples

### With Facebook Graph API

```typescript
import { postToFacebook } from '@/lib/social-media/facebook';

await postToFacebook({
  pageId: process.env.FACEBOOK_PAGE_ID,
  accessToken: process.env.FACEBOOK_ACCESS_TOKEN,
  message: article.content.vi,
  link: article.metadata.canonicalUrl,
});
```

### With WordPress

```typescript
import { publishToWordPress } from '@/lib/cms/wordpress';

await publishToWordPress({
  endpoint: process.env.WP_REST_API,
  username: process.env.WP_USERNAME,
  password: process.env.WP_APP_PASSWORD,
  post: {
    title: article.title.vi,
    content: article.content.vi,
    status: 'publish',
    categories: [1, 5], // Category IDs
  }
});
```
