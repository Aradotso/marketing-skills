---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, publishing, and video generation using Claude/OpenAI and Remotion
triggers:
  - "create automated content pipeline with AI"
  - "generate video content from research automatically"
  - "build AI content workflow from research to video"
  - "automate content creation with Claude and OpenAI"
  - "set up content automation with Remotion video rendering"
  - "create multi-format content with AI research"
  - "build automated marketing content system"
  - "generate social media videos from AI content"
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

An end-to-end AI-powered content automation system that handles research, scriptwriting, publishing, and video generation. Built with TypeScript, Next.js, Claude 3, OpenAI, and Remotion for rendering videos.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages using Claude/OpenAI
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Export**: Optimized video outputs for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=your_database_url

# Optional: Content Sources
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/                   # Core utilities and services
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── crawlers/         # Web scraping modules
│   ├── video/            # Remotion video rendering
│   └── utils/            # Helper functions
├── public/               # Static assets
└── remotion/             # Remotion video templates
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { AutoResearcher } from '@/lib/crawlers/auto-researcher';

// Initialize researcher
const researcher = new AutoResearcher({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  keywords: ['AI', 'marketing automation']
});

// Fetch and analyze content
const research = await researcher.gather();

console.log(research);
// {
//   articles: [...],
//   insights: [...],
//   trending: [...],
//   dataPoints: [...]
// }
```

### 2. AI Content Generation with Claude

```typescript
import { ClaudeContentGenerator } from '@/lib/ai/claude-generator';

const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: 'claude-3-opus-20240229'
});

// Generate content from research
const content = await generator.generate({
  research: researchData,
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  language: 'vi', // 'en' for English
  tone: 'professional', // 'friendly', 'humorous'
  targetAudience: 'marketers'
});

console.log(content);
// {
//   title: "Top 10 AI Marketing Tools...",
//   body: "...",
//   metadata: {...}
// }
```

### 3. OpenAI Alternative

```typescript
import { OpenAIContentGenerator } from '@/lib/ai/openai-generator';

const generator = new OpenAIContentGenerator({
  apiKey: process.env.OPENAI_API_KEY!,
  model: 'gpt-4-turbo-preview'
});

const content = await generator.generate({
  prompt: "Create a marketing article about...",
  format: 'case-study',
  maxTokens: 2000
});
```

### 4. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { ContentToVideoTemplate } from '@/remotion/templates/ContentTemplate';

// Render video from content
const videoPath = await renderVideo({
  composition: ContentToVideoTemplate,
  props: {
    title: content.title,
    body: content.body,
    highlights: content.highlights,
    style: 'modern'
  },
  outputFormat: 'mp4',
  aspectRatio: '9:16', // For Reels/TikTok/Shorts
  fps: 30,
  duration: 60 // seconds
});

console.log(`Video rendered: ${videoPath}`);
```

## Common Patterns

### Full Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  videoEnabled: true,
  autoPublish: false
});

// Run complete pipeline
const result = await pipeline.run({
  keyword: 'AI automation tools',
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  videoSpecs: {
    platforms: ['tiktok', 'reels', 'shorts']
  }
});

console.log(result);
// {
//   content: {
//     en: { toplist: {...}, howTo: {...} },
//     vi: { toplist: {...}, howTo: {...} }
//   },
//   videos: [
//     { platform: 'tiktok', path: '...' },
//     { platform: 'reels', path: '...' }
//   ]
// }
```

### Custom Research Sources

```typescript
import { CustomCrawler } from '@/lib/crawlers/custom';

// Add custom data source
const customSource = new CustomCrawler({
  url: 'https://your-source.com/api',
  parser: (data) => ({
    title: data.headline,
    content: data.body,
    publishedAt: data.date
  })
});

const researcher = new AutoResearcher({
  customSources: [customSource]
});
```

### Batch Content Generation

```typescript
const topics = ['AI tools', 'Marketing automation', 'Content strategy'];

const batchResults = await Promise.all(
  topics.map(async (topic) => {
    const research = await researcher.gather({ keywords: [topic] });
    const content = await generator.generate({
      research,
      format: 'toplist',
      language: 'en'
    });
    return { topic, content };
  })
);
```

## Next.js API Routes

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { keyword, format, language } = await req.json();

  const pipeline = new ContentPipeline({
    aiProvider: 'claude'
  });

  const result = await pipeline.run({
    keyword,
    formats: [format],
    languages: [language]
  });

  return NextResponse.json(result);
}
```

### Video Generation Endpoint

```typescript
// app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { contentId, platform } = await req.json();

  const content = await fetchContent(contentId);
  
  const aspectRatios = {
    tiktok: '9:16',
    reels: '9:16',
    shorts: '9:16',
    youtube: '16:9'
  };

  const videoPath = await renderVideo({
    composition: ContentToVideoTemplate,
    props: { ...content },
    aspectRatio: aspectRatios[platform]
  });

  return NextResponse.json({ videoPath });
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

# Render Remotion video (standalone)
npm run render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Remotion Video Templates

Create custom video templates in `remotion/` directory:

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1>{title}</h1>
        <p>{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```

Register template:

```typescript
// remotion/Root.tsx
import { Composition } from 'remotion';
import { CustomTemplate } from './templates/CustomTemplate';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="CustomTemplate"
        component={CustomTemplate}
        durationInFrames={150}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
};
```

## Troubleshooting

### AI API Rate Limits

```typescript
import { retry } from '@/lib/utils/retry';

const contentWithRetry = await retry(
  () => generator.generate({ research, format: 'toplist' }),
  {
    maxRetries: 3,
    delay: 2000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Memory Issues

For large video renders, increase Node.js memory:

```bash
NODE_OPTIONS=--max-old-space-size=4096 npm run render
```

### Crawler Blocking

Use proxies or rate limiting:

```typescript
const researcher = new AutoResearcher({
  rateLimit: {
    maxRequests: 10,
    perSeconds: 60
  },
  proxy: process.env.PROXY_URL
});
```

### Content Quality Issues

Fine-tune AI prompts:

```typescript
const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  defaultPromptModifiers: {
    creativity: 0.7,
    factAccuracy: 0.9,
    lengthTarget: 'medium'
  }
});
```

## Best Practices

1. **Cache Research Data**: Save crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use job queues for video generation (e.g., BullMQ)
3. **Content Validation**: Always validate AI-generated content before publishing
4. **Multi-Language Testing**: Test content in both languages for accuracy
5. **Video Preview**: Generate preview frames before full render
6. **Error Handling**: Implement comprehensive error handling for external APIs
