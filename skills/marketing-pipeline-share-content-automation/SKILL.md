---
name: marketing-pipeline-share-content-automation
description: AI-powered content pipeline that automates research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from text content automatically
  - create content pipeline with Claude and OpenAI
  - build automated marketing content system
  - set up AI content research and generation
  - integrate Remotion for video rendering
  - automate social media content workflow
  - create multilingual content with AI
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research to video generation. The pipeline uses Claude 3, OpenAI for content generation, and Remotion for video rendering.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-crawls** news and insights from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using AI
- **Supports multilingual** output (English and Vietnamese)
- **Renders videos** and infographics automatically using Remotion
- **Optimizes for platforms** like Reels, TikTok, Shorts

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

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
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video templates
│   └── utils/           # Utility functions
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core APIs and Usage

### 1. Content Research Module

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });
  
  return {
    articles: research.articles,
    insights: research.keyInsights,
    trends: research.trendingTopics,
    data: research.statistics
  };
}

// Usage
const insights = await gatherInsights('AI marketing automation');
console.log(insights.keyInsights);
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, ToneOfVoice } from '@/types/content';

async function createArticle(topic: string) {
  const content = await generateContent({
    topic,
    format: ContentFormat.CASE_STUDY,
    tone: ToneOfVoice.PROFESSIONAL,
    languages: ['en', 'vi'],
    aiProvider: 'claude', // or 'openai'
    research: {
      includeData: true,
      includeTrends: true,
      sources: ['recent_news', 'social_media']
    }
  });
  
  return {
    title: content.title,
    body: content.body,
    translations: content.translations,
    metadata: content.metadata
  };
}

// Usage with Claude
const article = await createArticle('Future of Content Marketing');
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

async function generateVideoFromContent(content: any) {
  const video = await renderVideo({
    template: VideoTemplate.INFOGRAPHIC,
    content: {
      title: content.title,
      keyPoints: content.keyPoints,
      statistics: content.data,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    outputFormat: {
      platform: 'reels', // 'tiktok', 'shorts', 'youtube'
      resolution: '1080x1920',
      duration: 30
    }
  });
  
  return video.url;
}

// Usage
const videoUrl = await generateVideoFromContent(article);
```

## Content Format Examples

### POV (Point of View) Format

```typescript
import { generatePOVContent } from '@/lib/content/formats/pov';

const povArticle = await generatePOVContent({
  topic: 'AI replacing content creators',
  perspective: 'contrarian',
  tone: 'thought-provoking',
  includeData: true,
  aiProvider: 'claude'
});

// Returns structured POV article with:
// - Hook
// - Main argument
// - Supporting evidence
// - Counter-arguments
// - Conclusion
```

### Toplist Format

```typescript
import { generateToplist } from '@/lib/content/formats/toplist';

const toplist = await generateToplist({
  topic: 'AI marketing tools',
  count: 10,
  criteria: ['features', 'pricing', 'ease-of-use'],
  includeRatings: true,
  research: {
    sources: ['product_hunt', 'g2', 'recent_news']
  }
});

// Returns ranked list with descriptions and ratings
```

## API Routes

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, languages } = await req.json();
    
    const content = await generateContent({
      topic: keyword,
      format,
      languages,
      aiProvider: process.env.AI_PROVIDER || 'claude'
    });
    
    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/crawler/research';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeRange } = await req.json();
  
  const research = await researchTopic({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeRange: timeRange || '24h'
  });
  
  return NextResponse.json({ research });
}
```

## Common Patterns

### Full Pipeline Execution

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoRenderer: 'remotion',
    languages: ['en', 'vi']
  });
  
  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content
  const content = await pipeline.generateContent({
    research,
    format: 'case-study',
    tone: 'professional'
  });
  
  // Step 3: Generate video
  const video = await pipeline.renderVideo({
    content,
    template: 'infographic',
    platform: 'reels'
  });
  
  // Step 4: Schedule posting (if integrated)
  await pipeline.schedule({
    content,
    video,
    platforms: ['facebook', 'instagram'],
    publishAt: new Date(Date.now() + 3600000)
  });
  
  return { content, video };
}
```

### Batch Content Generation

```typescript
async function generateBatch(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const content = await generateContent({
        topic: keyword,
        format: 'how-to',
        languages: ['en'],
        aiProvider: 'openai'
      });
      
      return { keyword, content };
    })
  );
  
  return results;
}

// Usage
const batch = await generateBatch([
  'AI content marketing',
  'Video automation tools',
  'Social media scheduling'
]);
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm start

# Render Remotion video locally
npm run remotion

# Type checking
npm run type-check

# Linting
npm run lint
```

## Remotion Video Templates

### Custom Video Template

```typescript
// src/remotion/templates/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const Infographic: React.FC<{
  title: string;
  data: Array<{ label: string; value: number }>;
}> = ({ title, data }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#fff' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ fontSize: 48, marginBottom: 30 }}>{title}</h1>
        {data.map((item, i) => (
          <div key={i} style={{ marginBottom: 20 }}>
            <span>{item.label}: {item.value}</span>
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

### Rendering Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setCodec('h264');
Config.setConcurrency(4);
Config.setQuality(90);

export default Config;
```

## Troubleshooting

### AI API Rate Limits

```typescript
import { retry } from '@/utils/retry';

async function generateWithRetry(topic: string) {
  return retry(
    async () => {
      return await generateContent({ topic });
    },
    {
      maxAttempts: 3,
      delayMs: 2000,
      backoff: 'exponential'
    }
  );
}
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrent renders
Config.setConcurrency(2);

// Or render with lower quality
const video = await renderVideo({
  content,
  options: {
    quality: 70,
    resolution: '720x1280' // Lower resolution
  }
});
```

### Crawler Timeout Issues

```typescript
const research = await researchTopic({
  keyword,
  sources: ['techcrunch'],
  timeout: 30000, // Increase timeout to 30s
  retries: 2
});
```

### Environment Variable Not Loading

```typescript
// Check if variables are loaded
if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY not found in environment');
}

// Use fallback for development
const apiKey = process.env.OPENAI_API_KEY || 
  (process.env.NODE_ENV === 'development' ? 'dev-key' : undefined);
```

## Best Practices

1. **Always validate research data** before content generation
2. **Cache research results** to avoid redundant API calls
3. **Use environment-specific AI providers** (Claude for long-form, OpenAI for quick generation)
4. **Batch video renders** during off-peak hours to save resources
5. **Implement rate limiting** for external APIs
6. **Store generated content** with metadata for future reference
