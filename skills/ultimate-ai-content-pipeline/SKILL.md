---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from text automatically
  - set up AI content pipeline with Claude
  - create marketing content with auto-research
  - build automated content workflow
  - generate social media videos with Remotion
  - scrape news and generate content
  - automate content from research to video
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics across multiple sources (TechCrunch, a16z, Twitter, LinkedIn), to generating articles in multiple formats and languages using Claude/OpenAI, to rendering videos and infographics with Remotion.

## What It Does

- **Auto-Research**: Crawls news sources and extracts trending insights from the last 24 hours
- **Multi-Format Content**: Generates articles in various formats (toplist, POV, case study, how-to)
- **Bilingual Output**: Creates content in both English and Vietnamese
- **Video Rendering**: Automatically converts content to social media videos (Reels, TikTok, Shorts)
- **Next.js Interface**: User-friendly web interface for managing the entire pipeline

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
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Render videos with Remotion
npm run render
```

## Core Architecture

### 1. Research Module

The research module scrapes and analyzes content from multiple sources:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

export async function fetchTrendingTopics(
  keyword: string,
  sources: NewsSource[]
): Promise<ResearchData[]> {
  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const response = await axios.get(source.url, {
          params: { q: keyword, timeRange: '24h' },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          },
        });
        return parseNewsData(response.data, source);
      } catch (error) {
        console.error(`Failed to fetch from ${source.name}:`, error);
        return null;
      }
    })
  );

  return results.filter(Boolean);
}

function parseNewsData(data: any, source: NewsSource): ResearchData {
  return {
    source: source.name,
    articles: data.articles.map((article: any) => ({
      title: article.title,
      summary: article.description,
      url: article.url,
      publishedAt: article.publishedAt,
      insights: extractInsights(article.content),
    })),
  };
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  keywords: string[];
}

export async function generateArticle(
  researchData: ResearchData[],
  config: ContentConfig
): Promise<Article> {
  const prompt = buildPrompt(researchData, config);

  // Using Claude for content generation
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  const content = message.content[0].text;

  return {
    title: extractTitle(content),
    body: content,
    format: config.format,
    language: config.language,
    metadata: {
      generatedAt: new Date(),
      sources: researchData.map((d) => d.source),
    },
  };
}

function buildPrompt(data: ResearchData[], config: ContentConfig): string {
  const formatInstructions = {
    toplist: 'Create a numbered list article with clear rankings',
    pov: 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze real examples with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions',
  };

  return `
You are an expert content creator. Based on the following research:

${JSON.stringify(data, null, 2)}

Create a ${config.format} article in ${config.language} with a ${config.tone} tone.

${formatInstructions[config.format]}

Include:
- Data-backed insights from the research
- Recent examples (within 24 hours)
- Actionable takeaways
- SEO-optimized structure

Keywords to incorporate: ${config.keywords.join(', ')}
  `.trim();
}
```

### 3. Bilingual Content Generation

```typescript
// lib/ai/multilingual.ts
export async function generateBilingualContent(
  researchData: ResearchData[],
  config: Omit<ContentConfig, 'language'>
): Promise<{ en: Article; vi: Article }> {
  const [englishArticle, vietnameseArticle] = await Promise.all([
    generateArticle(researchData, { ...config, language: 'en' }),
    generateArticle(researchData, { ...config, language: 'vi' }),
  ]);

  return {
    en: englishArticle,
    vi: vietnameseArticle,
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// remotion/VideoComposition.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Article Title',
          keyPoints: [],
          style: 'modern',
        }}
      />
    </>
  );
};
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  style: 'modern' | 'minimalist';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  style,
}) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <TitleScene title={title} frame={frame} />
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 60}
          durationInFrames={60}
        >
          <KeyPointScene point={point} index={index} frame={frame} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const TitleScene: React.FC<{ title: string; frame: number }> = ({
  title,
  frame,
}) => {
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <div
      style={{
        opacity,
        fontSize: 64,
        fontWeight: 'bold',
        color: 'white',
        textAlign: 'center',
        padding: 40,
      }}
    >
      {title}
    </div>
  );
};
```

```bash
# Render video from CLI
npx remotion render ContentVideo output.mp4 --props='{"title":"Your Title","keyPoints":["Point 1","Point 2"]}'
```

### 5. Complete Pipeline API Route

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { fetchTrendingTopics } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/multilingual';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, sources } = await request.json();

    // Step 1: Research
    const researchData = await fetchTrendingTopics(keyword, sources);

    // Step 2: Generate Content
    const content = await generateBilingualContent(researchData, {
      format,
      tone,
      keywords: [keyword],
    });

    // Step 3: Render Video
    const videoUrl = await renderVideo({
      title: content.en.title,
      keyPoints: extractKeyPoints(content.en.body),
      style: 'modern',
    });

    return NextResponse.json({
      success: true,
      data: {
        english: content.en,
        vietnamese: content.vi,
        videoUrl,
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
// Quick single-language article
import { generateArticle } from '@/lib/ai/content-generator';

const article = await generateArticle(researchData, {
  format: 'how-to',
  tone: 'friendly',
  language: 'en',
  keywords: ['AI', 'automation', 'marketing'],
});
```

### Pattern 2: Scheduled Content Pipeline

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI', 'Marketing', 'SaaS'];

  for (const topic of topics) {
    const researchData = await fetchTrendingTopics(topic, defaultSources);
    const content = await generateBilingualContent(researchData, {
      format: 'toplist',
      tone: 'professional',
      keywords: [topic],
    });

    // Save to database or publish directly
    await publishContent(content);
  }
});
```

### Pattern 3: Custom Video Templates

```typescript
// lib/video/templates.ts
export const videoTemplates = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 450, // 15 seconds
  },
  youtube: {
    width: 1920,
    height: 1080,
    fps: 30,
    durationInFrames: 900, // 30 seconds
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 300, // 10 seconds
  },
};

export async function renderForPlatform(
  platform: keyof typeof videoTemplates,
  content: Article
) {
  const template = videoTemplates[platform];

  return await renderVideo({
    ...template,
    title: content.title,
    keyPoints: extractKeyPoints(content.body),
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pRetry from 'p-retry';

export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  return pRetry(fn, {
    retries: maxRetries,
    onFailedAttempt: (error) => {
      console.log(
        `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
      );
    },
  });
}

// Usage
const article = await withRetry(() =>
  generateArticle(researchData, config)
);
```

### Memory Issues with Large Datasets

```typescript
// Process research data in chunks
async function processLargeDataset(items: any[], chunkSize = 10) {
  const results = [];

  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const processed = await Promise.all(
      chunk.map((item) => processItem(item))
    );
    results.push(...processed);
  }

  return results;
}
```

### Video Rendering Fails

Check Remotion logs and ensure all dependencies are installed:

```bash
# Install Remotion dependencies
npm install @remotion/cli @remotion/renderer

# Test render locally
npx remotion preview
```

Ensure sufficient disk space and memory for video rendering:

```typescript
// lib/video/renderer.ts
import { renderMedia } from '@remotion/renderer';

export async function renderVideo(props: VideoProps) {
  try {
    await renderMedia({
      composition: 'ContentVideo',
      serveUrl: './out',
      codec: 'h264',
      outputLocation: `output/${Date.now()}.mp4`,
      inputProps: props,
      chromiumOptions: {
        // Reduce memory usage
        args: ['--no-sandbox', '--disable-setuid-sandbox'],
      },
    });
  } catch (error) {
    console.error('Render failed:', error);
    throw new Error('Video rendering failed. Check disk space and memory.');
  }
}
```

## Best Practices

1. **Cache Research Data**: Avoid redundant API calls by caching recent research results
2. **Batch Processing**: Generate multiple articles in parallel when possible
3. **Error Handling**: Always wrap API calls in try-catch blocks with proper logging
4. **Rate Limiting**: Implement exponential backoff for external API calls
5. **Video Optimization**: Pre-render video templates to speed up generation
6. **Database Storage**: Store generated content for reuse and analytics
