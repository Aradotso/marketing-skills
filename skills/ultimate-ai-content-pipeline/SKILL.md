---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion for multi-format marketing content
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate marketing videos automatically from text
  - scrape news and create content with AI
  - build automated content workflow with Remotion
  - create multi-language marketing content pipeline
  - automate TechCrunch and Twitter content research
  - render videos from AI-generated scripts
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Automated content production system that handles research (scraping TechCrunch, Twitter, LinkedIn), AI-powered script generation (Claude/OpenAI), multi-format content creation, and automatic video rendering using Remotion. Supports Vietnamese and English content with customizable tones.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Architecture

The pipeline consists of three main stages:

1. **Research Stage**: Auto-scrape news sources
2. **Generation Stage**: AI-powered content creation
3. **Render Stage**: Video/image generation with Remotion

## Research Module (Auto-Scan)

### Scraping News Sources

```typescript
import { scrapeLatestNews } from '@/lib/research/scraper';

// Scrape TechCrunch articles
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  
  const newsData = await scrapeLatestNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 10
  });
  
  // Returns structured data
  return newsData.map(article => ({
    title: article.title,
    url: article.url,
    summary: article.summary,
    publishedAt: article.publishedAt,
    source: article.source
  }));
}
```

### Extracting Insights

```typescript
import { extractInsights } from '@/lib/research/analyzer';

async function analyzeResearch(articles: Article[]) {
  const insights = await extractInsights({
    articles,
    model: 'claude-3-sonnet-20240229',
    language: 'en'
  });
  
  return {
    keyTrends: insights.trends,
    dataPoints: insights.statistics,
    expertQuotes: insights.quotes,
    actionableInsights: insights.recommendations
  };
}
```

## Content Generation Module

### Generate Multi-Format Content

```typescript
import { generateContent } from '@/lib/generation/content-generator';

async function createContent(params: {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
}) {
  const content = await generateContent({
    keyword: params.keyword,
    research: await researchTopic(params.keyword),
    format: params.format,
    tone: params.tone,
    language: params.language,
    aiProvider: 'claude' // or 'openai'
  });
  
  return {
    title: content.title,
    body: content.body,
    meta: content.meta,
    keywords: content.keywords,
    cta: content.callToAction
  };
}
```

### Dual Language Generation

```typescript
import { generateDualLanguage } from '@/lib/generation/multi-lang';

async function createBilingualContent(keyword: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ keyword, format: 'toplist', tone: 'expert', language: 'en' }),
    generateContent({ keyword, format: 'toplist', tone: 'expert', language: 'vi' })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    syncedAt: new Date().toISOString()
  };
}
```

## Video Rendering Module (Remotion)

### Setup Remotion Composition

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  branding: {
    logo: string;
    colors: string[];
  };
}> = ({ title, points, branding }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors[0] }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ fontSize: 72, color: '#fff' }}>{title}</h1>
        {points.map((point, i) => (
          <p key={i} style={{ fontSize: 36, color: '#fff', marginTop: 20 }}>
            {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

### Register Compositions

```typescript
// src/remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920} // 9:16 for Reels/TikTok
        defaultProps={{
          title: 'Default Title',
          points: [],
          branding: { logo: '', colors: ['#000'] }
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

### Render Video Programmatically

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: GeneratedContent) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#1a1a1a', '#3b82f6']
      }
    }
  });
  
  const outputPath = path.join(process.cwd(), 'output', `${content.id}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });
  
  return outputPath;
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/generation/content-generator';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, language } = await request.json();
    
    // Step 1: Research
    const research = await researchTopic(keyword);
    
    // Step 2: Generate Content
    const content = await generateContent({
      keyword,
      research,
      format,
      tone,
      language,
      aiProvider: 'claude'
    });
    
    return NextResponse.json({
      success: true,
      content,
      research: research.slice(0, 3) // Preview
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/render/video-renderer';

export async function POST(request: NextRequest) {
  try {
    const { contentId } = await request.json();
    
    // Fetch content from database
    const content = await fetchContentById(contentId);
    
    // Render video
    const videoPath = await renderContentVideo(content);
    
    return NextResponse.json({
      success: true,
      videoUrl: `/output/${path.basename(videoPath)}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY,
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  // Full automation
  const result = await pipeline.execute({
    keyword,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    generateVideo: true,
    videoAspectRatio: '9:16', // Reels/TikTok
    autoPublish: false // Set true to auto-post
  });
  
  return {
    articles: result.articles, // Array of generated content
    videos: result.videos,     // Array of rendered video URLs
    insights: result.research  // Research insights
  };
}

// Usage
const output = await runFullPipeline('AI Marketing Automation 2026');
console.log(`Generated ${output.articles.length} articles`);
console.log(`Rendered ${output.videos.length} videos`);
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      createContent({
        keyword,
        format: 'toplist',
        tone: 'expert',
        language: 'en'
      })
    )
  );
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Custom Tone Configuration

```typescript
const tonePresets = {
  expert: {
    systemPrompt: 'You are a seasoned industry expert...',
    temperature: 0.7,
    maxTokens: 2000
  },
  friendly: {
    systemPrompt: 'You are a helpful friend sharing insights...',
    temperature: 0.8,
    maxTokens: 1500
  },
  humorous: {
    systemPrompt: 'You are witty and entertaining...',
    temperature: 0.9,
    maxTokens: 1500
  }
};

await generateContent({
  keyword: 'startup funding',
  format: 'pov',
  tone: 'humorous',
  customTone: tonePresets.humorous
});
```

### Scheduling Content

```typescript
import { scheduleContent } from '@/lib/scheduler';

async function scheduleWeeklyContent() {
  const keywords = ['AI trends', 'Marketing automation', 'Growth hacking'];
  
  for (const [index, keyword] of keywords.entries()) {
    await scheduleContent({
      keyword,
      format: 'toplist',
      publishAt: new Date(Date.now() + (index * 24 * 60 * 60 * 1000)),
      platforms: ['facebook', 'linkedin', 'twitter']
    });
  }
}
```

## CLI Commands

```bash
# Run development server
npm run dev

# Generate content from CLI
npm run generate -- --keyword "AI trends" --format toplist --lang en

# Render video
npm run render -- --content-id abc123 --ratio 9:16

# Batch process
npm run batch -- --keywords-file ./keywords.txt

# Preview Remotion compositions
npm run remotion:preview
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMilliseconds: 60000 // 10 requests per minute
});

await limiter.schedule(() => generateContent({...}));
```

### Video Rendering Memory Issues

```typescript
// Reduce video quality for large batches
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  scale: 0.5, // Reduce resolution
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-dev-shm-usage']
  }
});
```

### Research Scraping Failures

```typescript
try {
  const news = await scrapeLatestNews({ keyword, sources });
} catch (error) {
  console.error('Scraping failed, using fallback');
  // Fallback to cached data or alternative sources
  const fallbackNews = await getCachedNews(keyword);
  return fallbackNews;
}
```

### Claude/OpenAI Timeout

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 60000, // 60 seconds
  maxRetries: 3
});

const response = await anthropic.messages.create({
  model: 'claude-3-sonnet-20240229',
  max_tokens: 2000,
  messages: [{ role: 'user', content: prompt }]
});
```

## Performance Optimization

```typescript
// Use streaming for large content generation
import { OpenAI } from 'openai';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

async function generateWithStreaming(prompt: string) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{ role: 'user', content: prompt }],
    stream: true
  });
  
  let fullContent = '';
  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || '';
    fullContent += content;
    process.stdout.write(content); // Real-time output
  }
  
  return fullContent;
}
```
