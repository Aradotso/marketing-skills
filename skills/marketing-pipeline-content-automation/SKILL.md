---
name: marketing-pipeline-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation with multi-language support
triggers:
  - automate content creation with AI research
  - generate videos from text content automatically
  - create multi-language marketing content
  - set up AI content pipeline with Claude and OpenAI
  - crawl news and generate blog posts
  - build automated content workflow
  - generate social media videos with Remotion
  - automate research to video pipeline
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a TypeScript-based automation system that transforms keywords into complete content pieces with research, scriptwriting, and video generation. It crawls real-time data from TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats (toplist, POV, case study, how-to) and languages (English/Vietnamese), before automatically rendering videos using Remotion.

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
# AI Provider Keys (use at least one)
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Research/Crawling API
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database/Storage
DATABASE_URL=your_database_connection_string

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # Web scrapers for research
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlTechNews } from '@/lib/crawlers/tech-news';
import { analyzeInsights } from '@/lib/ai/analyzer';

async function gatherResearch(keyword: string) {
  // Crawl recent news from multiple sources
  const newsData = await crawlTechNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeInsights({
    data: newsData,
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229'
  });

  return insights;
}
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/generators/content';

async function createContent(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format: format as 'toplist' | 'pov' | 'case-study' | 'how-to',
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly', 'humorous'
    aiProvider: process.env.ANTHROPIC_API_KEY ? 'claude' : 'openai',
    includeData: true,
    researchData: await gatherResearch(topic)
  });

  return content;
}

// Usage
const article = await createContent(
  'AI automation trends 2024',
  'toplist'
);

console.log(article.en.title);
console.log(article.vi.title);
console.log(article.en.body);
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

async function generateVideoFromContent(content: any) {
  // Prepare video composition
  const composition = {
    id: 'ContentVideo',
    data: {
      title: content.en.title,
      sections: content.en.sections,
      duration: 60, // seconds
      aspectRatio: '9:16' // Reels/TikTok/Shorts
    }
  };

  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  // Render video
  const output = await renderMedia({
    composition: composition.id,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `./output/video-${Date.now()}.mp4`,
    inputProps: composition.data
  });

  return output;
}
```

## Complete Workflow Example

```typescript
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, generateVideo } = await req.json();

    // Step 1: Research
    const research = await gatherResearch(keyword);

    // Step 2: Generate Content
    const content = await generateContent({
      topic: keyword,
      format,
      languages: ['en', 'vi'],
      researchData: research,
      tone: 'professional',
      aiProvider: 'claude'
    });

    // Step 3: Optional Video Generation
    let videoUrl = null;
    if (generateVideo) {
      const video = await generateVideoFromContent(content);
      videoUrl = video.outputLocation;
    }

    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      research: research.summary
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## AI Provider Integration

### Using Claude (Anthropic)

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, research: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Research data: ${JSON.stringify(research)}\n\nTask: ${prompt}`
    }]
  });

  return message.content[0].text;
}
```

### Using OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, research: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a content creation expert.'
      },
      {
        role: 'user',
        content: `Research: ${JSON.stringify(research)}\n\n${prompt}`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Lint code
npm run lint

# Render video (if using Remotion CLI)
npm run remotion render ContentVideo output.mp4
```

## Content Format Templates

### Toplist Format

```typescript
interface ToplistContent {
  title: string;
  introduction: string;
  items: Array<{
    rank: number;
    title: string;
    description: string;
    dataPoints: string[];
    source?: string;
  }>;
  conclusion: string;
}
```

### Case Study Format

```typescript
interface CaseStudyContent {
  title: string;
  problem: string;
  solution: string;
  implementation: string[];
  results: {
    metric: string;
    before: string;
    after: string;
    improvement: string;
  }[];
  takeaways: string[];
}
```

## Multi-Language Configuration

```typescript
const contentConfig = {
  languages: {
    en: {
      locale: 'en-US',
      tone: 'professional',
      lengthTarget: 1500 // words
    },
    vi: {
      locale: 'vi-VN',
      tone: 'friendly',
      lengthTarget: 1800 // words (Vietnamese typically longer)
    }
  }
};

async function generateBilingual(topic: string) {
  const results = await Promise.all(
    Object.entries(contentConfig.languages).map(([lang, config]) =>
      generateContent({
        topic,
        language: lang,
        ...config
      })
    )
  );

  return Object.fromEntries(
    results.map((content, i) => [
      Object.keys(contentConfig.languages)[i],
      content
    ])
  );
}
```

## Video Templates with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  sections: string[];
  duration: number;
}> = ({ title, sections, duration }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={90}>
        <TitleSlide title={title} />
      </Sequence>
      {sections.map((section, i) => (
        <Sequence
          key={i}
          from={90 + i * 120}
          durationInFrames={120}
        >
          <ContentSlide text={section} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### AI API Rate Limits

```typescript
async function generateWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```

### Crawling Failures

```typescript
async function safeCrawl(url: string) {
  try {
    const response = await fetch(url, {
      headers: {
        'User-Agent': 'Mozilla/5.0...',
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
      },
      signal: AbortSignal.timeout(10000)
    });

    if (!response.ok) {
      console.warn(`Crawl failed for ${url}: ${response.status}`);
      return null;
    }

    return await response.json();
  } catch (error) {
    console.error('Crawl error:', error);
    return null;
  }
}
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large projects
const renderConfig = {
  concurrency: 1, // Reduce concurrent renders
  frameRange: [0, 300], // Limit frame range if needed
  quality: 80, // Balance quality vs. size
  enforceAudioTrack: false
};
```

## Best Practices

1. **Always validate research data** before feeding to AI to avoid hallucinations
2. **Cache research results** for 24h to reduce API costs
3. **Use streaming responses** for real-time content generation UI
4. **Implement job queues** (e.g., BullMQ) for video rendering
5. **Store generated content** in database with metadata for analytics
6. **Version control prompts** separately for easy iteration
