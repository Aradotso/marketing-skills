---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from articles automatically
  - create content with Claude and OpenAI APIs
  - auto-research and write blog posts with AI
  - build content automation workflow
  - use Remotion to render marketing videos
  - scrape news and generate content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation system that handles research (web scraping), content generation (Claude/OpenAI), and video rendering (Remotion). It crawls news from sources like TechCrunch, a16z, Twitter, and LinkedIn, then generates multi-format content (articles, videos, infographics) in multiple languages.

## What It Does

- **Auto-Research**: Scrapes real-time news from major sources within the last 24 hours
- **AI Content Generation**: Creates articles in various formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-Language**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically creates videos and infographics using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
```

Required environment variables:

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Web Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── services/
│   │   ├── scraper/        # Web scraping logic
│   │   ├── ai/             # AI content generation
│   │   └── video/          # Remotion video rendering
│   ├── lib/
│   │   ├── claude.ts       # Claude API integration
│   │   ├── openai.ts       # OpenAI API integration
│   │   └── crawler.ts      # News crawling utilities
│   ├── app/                # Next.js app directory
│   └── remotion/           # Remotion video compositions
├── public/
└── package.json
```

## Key API Usage

### 1. Content Research & Scraping

```typescript
import { NewsScraperService } from '@/services/scraper';

const scraper = new NewsScraperService({
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Scrape recent news from multiple sources
const researchData = await scraper.scrapeTopics({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h',
  maxResults: 50
});

console.log(researchData);
// {
//   articles: [...],
//   insights: [...],
//   trends: [...]
// }
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(researchData: any, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `
        Based on this research data: ${JSON.stringify(researchData)}
        
        Create a ${format} article with:
        - Compelling headline
        - Data-backed insights
        - Clear structure
        - Actionable takeaways
        
        Language: Vietnamese and English
        Tone: Professional yet engaging
      `
    }]
  });

  return message.content[0].text;
}

// Usage
const article = await generateArticle(researchData, 'case-study');
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and trend analysis.'
      },
      {
        role: 'user',
        content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// src/remotion/compositions/ArticleVideo.tsx
import { Composition } from 'remotion';
import { ArticleScene } from './scenes/ArticleScene';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ArticleVideo"
        component={ArticleScene}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // Vertical for Reels/TikTok
        defaultProps={{
          title: '',
          content: '',
          insights: []
        }}
      />
    </>
  );
};
```

```typescript
// src/services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderArticleVideo(articleData: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: {
      title: articleData.title,
      content: articleData.summary,
      insights: articleData.keyPoints
    },
  });

  const outputLocation = `./output/video-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
  });

  return outputLocation;
}
```

## Complete Workflow Example

```typescript
// src/workflows/content-pipeline.ts
import { NewsScraperService } from '@/services/scraper';
import { ContentGenerator } from '@/services/ai/generator';
import { renderArticleVideo } from '@/services/video/renderer';

export async function runContentPipeline(keyword: string) {
  // Step 1: Research
  const scraper = new NewsScraperService({
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  const researchData = await scraper.scrapeTopics({
    keyword,
    sources: ['techcrunch', 'a16z', 'linkedin'],
    timeRange: '24h'
  });

  // Step 2: Generate Content
  const generator = new ContentGenerator({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY
  });

  const article = await generator.createArticle({
    research: researchData,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional'
  });

  // Step 3: Render Video
  const videoPath = await renderArticleVideo({
    title: article.title,
    summary: article.summary,
    keyPoints: article.insights
  });

  return {
    article,
    videoPath,
    metadata: {
      keyword,
      sources: researchData.sources,
      generatedAt: new Date()
    }
  };
}

// Usage
const result = await runContentPipeline('AI marketing automation');
console.log('Content created:', result);
```

## Common Patterns

### Multi-Format Content Generation

```typescript
const formats = ['toplist', 'case-study', 'how-to', 'pov'];

async function generateAllFormats(researchData: any) {
  const articles = await Promise.all(
    formats.map(format => 
      generator.createArticle({
        research: researchData,
        format,
        languages: ['en', 'vi']
      })
    )
  );

  return articles;
}
```

### Scheduling Content Publishing

```typescript
import { CronJob } from 'cron';

// Run pipeline daily at 6 AM
const dailyContentJob = new CronJob('0 6 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'content strategy'];
  
  for (const keyword of keywords) {
    await runContentPipeline(keyword);
  }
});

dailyContentJob.start();
```

### Custom Remotion Compositions

```typescript
// src/remotion/scenes/ArticleScene.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ArticleScene: React.FC<{
  title: string;
  content: string;
  insights: string[];
}> = ({ title, content, insights }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ color: 'white', fontSize: 64, marginBottom: 40 }}>
          {title}
        </h1>
        <div style={{ color: '#ccc', fontSize: 32, lineHeight: 1.6 }}>
          {insights.map((insight, i) => (
            <p key={i}>• {insight}</p>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (preview)
npm run remotion:preview

# Render specific composition
npm run remotion:render ArticleVideo output.mp4
```

## Configuration

### Content Generation Settings

```typescript
// config/content.config.ts
export const contentConfig = {
  ai: {
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022',
    temperature: 0.7,
    maxTokens: 4096
  },
  
  scraping: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxArticles: 50
  },
  
  video: {
    fps: 30,
    width: 1080,
    height: 1920, // Vertical
    codec: 'h264',
    quality: 'high'
  },
  
  languages: ['en', 'vi'],
  
  formats: [
    'toplist',
    'case-study',
    'how-to',
    'pov'
  ]
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Scraping Failures

```typescript
// Fallback to cached data or alternative sources
async function scrapeWithFallback(keyword: string) {
  try {
    return await scraper.scrapeTopics({ keyword });
  } catch (error) {
    console.warn('Primary scraping failed, using cache');
    return getCachedResearch(keyword);
  }
}
```

### Video Rendering Memory Issues

```typescript
// Render in chunks for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  // Optimize memory usage
  chromiumOptions: {
    gl: 'angle',
  },
  concurrency: 1, // Reduce if memory issues persist
});
```

### Missing Environment Variables

```typescript
// Validate environment at startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
    'OPENAI_API_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing env vars: ${missing.join(', ')}`);
  }
}

validateEnv();
```

This skill enables AI agents to help developers set up and use a complete AI-powered content automation pipeline, from research through video generation.
