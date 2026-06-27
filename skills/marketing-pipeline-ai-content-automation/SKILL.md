---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for marketing workflows
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from text with Remotion
  - crawl news and create content automatically
  - use Claude and OpenAI for content generation
  - build automated content workflow with AI
  - create multilingual marketing content with AI
  - research and write articles automatically
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides a complete automated content pipeline that crawls news, generates multilingual articles using AI (Claude/OpenAI), and renders videos with Remotion. Built with TypeScript and Next.js.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multilingual Support**: Generates content in English and Vietnamese simultaneously
4. **Video Generation**: Automatically renders infographics and short videos using Remotion
5. **Multi-Platform Optimization**: Exports videos for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if used)
DATABASE_URL=your_database_url

# Optional: Video Generation
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI providers (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Crawl News

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';

async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await crawlNews({
    keyword,
    sources,
    timeframe: '24h',
    maxResults: 20
  });
  
  return results.map(article => ({
    title: article.title,
    url: article.url,
    content: article.content,
    publishedAt: article.publishedAt,
    source: article.source
  }));
}

// Usage
const research = await gatherResearch('AI marketing automation');
```

### 2. Generate Content with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  research: any[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `
Based on the following research data, create a ${format} article in ${language}:

${research.map(r => `- ${r.title}: ${r.content}`).join('\n')}

Format requirements:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- ${language === 'vi' ? 'Friendly, professional Vietnamese' : 'Professional English'}
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

// Usage
const article = await generateArticle(research, 'toplist', 'en');
```

### 3. Generate Content with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  research: any[],
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer. Create engaging marketing content based on provided research.`
      },
      {
        role: 'user',
        content: JSON.stringify(research)
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Render Video with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(articleData: {
  title: string;
  points: string[];
  images: string[];
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: articleData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${articleData.title.replace(/\s+/g, '-')}.mp4`,
    inputProps: articleData,
  });
}

// Usage
await generateVideo({
  title: '5 AI Marketing Trends 2024',
  points: [
    'Personalization at scale',
    'Predictive analytics',
    'Content automation',
    'Chatbot evolution',
    'Visual AI'
  ],
  images: ['/img1.jpg', '/img2.jpg']
});
```

### 5. Complete Pipeline Example

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { generateArticle } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Crawling news...');
    const research = await crawlNews({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h',
      maxResults: 10
    });

    // Step 2: Generate Content (Bilingual)
    console.log('✍️ Generating articles...');
    const [englishArticle, vietnameseArticle] = await Promise.all([
      generateArticle(research, 'toplist', 'en'),
      generateArticle(research, 'toplist', 'vi')
    ]);

    // Step 3: Extract video-friendly content
    const videoData = extractVideoPoints(englishArticle);

    // Step 4: Generate Video
    console.log('🎬 Rendering video...');
    await generateVideo(videoData);

    return {
      articles: {
        en: englishArticle,
        vi: vietnameseArticle
      },
      video: `out/${videoData.title.replace(/\s+/g, '-')}.mp4`,
      research: research.length
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractVideoPoints(article: string) {
  // Extract key points from article for video
  const lines = article.split('\n');
  const points = lines
    .filter(line => line.match(/^\d+\.|^-|^•/))
    .slice(0, 5);

  return {
    title: lines[0].replace(/^#\s+/, ''),
    points,
    images: []
  };
}
```

## Next.js API Routes

### Create Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();

  try {
    const results = await crawlNews({
      keyword,
      sources,
      timeframe: '24h',
      maxResults: 20
    });

    return NextResponse.json({ success: true, data: results });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Create Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateArticle } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  const { research, format, language } = await request.json();

  try {
    const article = await generateArticle(research, format, language);
    
    return NextResponse.json({ success: true, article });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Development Workflow

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run type checking
npm run type-check

# Render video locally
npm run remotion:preview

# Build video
npm run remotion:render
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // 3 concurrent requests

async function generateMultipleArticles(topics: string[]) {
  const articles = await Promise.all(
    topics.map(topic =>
      limit(async () => {
        const research = await crawlNews({ keyword: topic });
        return generateArticle(research, 'toplist', 'en');
      })
    )
  );
  
  return articles;
}
```

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached as string);
  }
  
  const results = await crawlNews({ keyword });
  await redis.set(cacheKey, JSON.stringify(results), { ex: 3600 }); // 1 hour
  
  return results;
}
```

### Content Variation Testing

```typescript
async function generateVariations(research: any[], count: number = 3) {
  const temperatures = [0.5, 0.7, 0.9];
  
  const variations = await Promise.all(
    temperatures.slice(0, count).map(temp =>
      anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        temperature: temp,
        messages: [{
          role: 'user',
          content: `Create article from: ${JSON.stringify(research)}`
        }]
      })
    )
  );
  
  return variations.map(v => v.content[0].text);
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
import { sleep } from '@/lib/utils';

async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retrying in ${delay}ms...`);
      await sleep(delay);
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

For large videos, use headless rendering:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  chromiumOptions: {
    headless: true,
    gl: 'swiftshader',
  },
  concurrency: 1, // Reduce concurrency for memory constraints
});
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

// Call at app startup
validateEnv();
```

## Best Practices

1. **Always validate research data** before passing to AI models
2. **Cache API responses** to reduce costs and latency
3. **Use streaming** for long-running content generation
4. **Implement retry logic** for external API calls
5. **Monitor token usage** for cost optimization
6. **Version your prompts** for reproducibility
7. **Test video templates** before bulk rendering

## Resources

- [Anthropic Claude API Docs](https://docs.anthropic.com/)
- [OpenAI API Reference](https://platform.openai.com/docs/)
- [Remotion Documentation](https://www.remotion.dev/docs/)
- [Next.js App Router](https://nextjs.org/docs/app)
