---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - "set up AI content automation pipeline"
  - "create automated content research and video generation"
  - "build content pipeline with Claude and Remotion"
  - "automate blog posts and video creation"
  - "configure content research crawler for articles"
  - "generate multi-format content with AI"
  - "setup marketing content automation system"
  - "create automated social media content pipeline"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive content automation system that handles the entire pipeline from research to video generation. It crawls recent news from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI, and automatically renders videos and infographics using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major sources (24h window)
- **AI Content Generation**: Creates articles in multiple formats and languages (EN/VI)
- **Video Rendering**: Automatically generates videos and infographics from written content
- **Multi-Platform Export**: Optimized output for Reels, TikTok, Shorts

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
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── crawler/     # Research crawler modules
│   │   └── remotion/    # Video generation
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Key Usage Patterns

### 1. Content Research & Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherInsights(topic: string) {
  const results = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });

  return {
    insights: results.insights,
    dataPoints: results.statistics,
    trendingTopics: results.trending
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  topic: string, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Write a ${format} article about "${topic}" in a ${tone} tone. 
        Include data-backed insights and current trends. Format for blog publishing.`
      }
    ],
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateBilingualContent(topic: string, researchData: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content writer creating bilingual content (English and Vietnamese).'
      },
      {
        role: 'user',
        content: `Create an article about "${topic}" using this research: ${JSON.stringify(researchData)}. 
        Provide both English and Vietnamese versions.`
      }
    ],
    response_format: { type: 'json_object' }
  });

  return JSON.parse(completion.choices[0].message.content);
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/templates/VideoTemplate';

async function generateVideo(content: {
  title: string;
  keyPoints: string[];
  images: string[];
}) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.title}-video.mp4`,
    inputProps: content,
  });
}
```

### 5. Full Content Pipeline

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateArticle } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/remotion/render';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate Content
  const article = await generateArticle(
    keyword,
    'toplist',
    'expert',
    research
  );

  // Step 3: Extract Key Points for Video
  const videoContent = {
    title: article.title,
    keyPoints: article.keyPoints || [],
    images: article.featuredImages || []
  };

  // Step 4: Generate Video
  await generateVideo(videoContent);

  return {
    article,
    videoPath: `out/${videoContent.title}-video.mp4`
  };
}
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/crawler/research';

export async function POST(request: Request) {
  const { keyword, sources, timeframe } = await request.json();

  try {
    const results = await researchTopic({
      keyword,
      sources,
      timeframe
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

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateArticle } from '@/lib/ai/claude';

export async function POST(request: Request) {
  const { topic, format, tone, language, research } = await request.json();

  try {
    const content = await generateArticle(topic, format, tone, research);
    
    return NextResponse.json({
      success: true,
      content: {
        english: content.en,
        vietnamese: content.vi
      }
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// app/api/video/route.ts
import { NextResponse } from 'next/server';
import { generateVideo } from '@/lib/remotion/render';

export async function POST(request: Request) {
  const { content } = await request.json();

  try {
    const videoPath = await generateVideo(content);
    
    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${videoPath}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video (Remotion CLI)
npx remotion render VideoTemplate out/video.mp4
```

## Common Patterns

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run content pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline(topic);
  }
});
```

### Batch Processing

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => runContentPipeline(topic))
  );

  return results.map((result, index) => ({
    topic: topics[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Custom Video Templates

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  keyPoints: string[];
}> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ fontSize: 64, color: 'white' }}>{title}</h1>
        <ul>
          {keyPoints.map((point, i) => (
            <li key={i} style={{ fontSize: 32, color: '#ccc' }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### AI API Rate Limits

```typescript
import pLimit from 'p-limit';

// Limit concurrent AI requests
const limit = pLimit(3);

async function generateMultipleArticles(topics: string[]) {
  return Promise.all(
    topics.map(topic => 
      limit(() => generateArticle(topic, 'toplist', 'expert'))
    )
  );
}
```

### Video Rendering Memory Issues

```typescript
// Use smaller compositions or render in chunks
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: `out/video.mp4`,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  },
  // Limit concurrency
  concurrency: 2,
  // Increase timeout
  timeoutInMilliseconds: 120000
});
```

### Research Crawler Failures

```typescript
async function safeResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const results = [];

  for (const source of sources) {
    try {
      const data = await crawlSource(source, keyword);
      results.push(data);
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
      // Continue with other sources
    }
  }

  return results.filter(Boolean);
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in the root directory and restart the dev server:

```bash
# Clear Next.js cache
rm -rf .next

# Restart dev server
npm run dev
```

## Best Practices

1. **API Key Management**: Always use environment variables, never commit keys
2. **Rate Limiting**: Implement delays between API calls to avoid rate limits
3. **Error Handling**: Wrap all AI and crawler calls in try-catch blocks
4. **Caching**: Cache research results to reduce API calls
5. **Video Optimization**: Use appropriate video codecs and resolutions for target platforms
6. **Content Review**: Always review AI-generated content before publishing
