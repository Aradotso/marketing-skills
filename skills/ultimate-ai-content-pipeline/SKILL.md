```markdown
---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content research and generation
  - generate video content from text automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - build content automation with Remotion video rendering
  - configure multi-language content generation pipeline
  - automate content from research to video generation
  - use AI for content research and scriptwriting
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from web scraping and research, to AI-powered scriptwriting (using Claude 3 and OpenAI), to automatic video generation with Remotion. It supports multiple content formats, bilingual content (English/Vietnamese), and automated social media publishing.

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

# Research & Data Collection
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Optional: Social Media Publishing
FACEBOOK_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── research/     # Web scraping & data collection
│   │   ├── ai/           # AI content generation (Claude/OpenAI)
│   │   ├── video/        # Remotion video rendering
│   │   └── utils/        # Helper functions
│   └── types/            # TypeScript type definitions
├── public/               # Static assets
└── remotion/             # Remotion video templates
```

## Core Modules

### 1. Research & Data Collection

```typescript
import { ResearchEngine } from '@/lib/research/engine';

// Initialize research engine
const researcher = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h'
});

// Perform automated research
const research = await researcher.scan({
  keyword: 'AI marketing automation',
  depth: 'deep', // 'quick' | 'deep' | 'comprehensive'
  language: 'en'
});

// Extract insights
const insights = research.extractInsights({
  minRelevance: 0.7,
  maxResults: 10
});

console.log(insights);
// {
//   trends: [...],
//   statistics: [...],
//   quotes: [...],
//   sources: [...]
// }
```

### 2. AI Content Generation

#### Using Claude (Anthropic)

```typescript
import { ClaudeContentGenerator } from '@/lib/ai/claude';

const claudeGen = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content from research
const content = await claudeGen.generateContent({
  research: insights,
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  tone: 'expert', // 'expert' | 'friendly' | 'humorous'
  language: 'vi', // 'en' | 'vi' | 'both'
  wordCount: 1500
});

console.log(content);
// {
//   title: "Top 10 AI Marketing Trends...",
//   body: "...",
//   meta: { ... },
//   language: 'vi'
// }
```

#### Using OpenAI

```typescript
import { OpenAIContentGenerator } from '@/lib/ai/openai';

const openaiGen = new OpenAIContentGenerator({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

// Generate bilingual content
const bilingualContent = await openaiGen.generateBilingual({
  topic: 'AI content automation',
  research: insights,
  format: 'how-to',
  primaryLanguage: 'en',
  secondaryLanguage: 'vi'
});

console.log(bilingualContent);
// {
//   en: { title: "...", body: "..." },
//   vi: { title: "...", body: "..." }
// }
```

### 3. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const videoGen = new VideoRenderer({
  composition: 'ContentVideo',
  fps: 30,
  durationInFrames: 300
});

// Render video from content
const videoPath = await videoGen.render({
  content: content.body,
  title: content.title,
  style: 'infographic', // 'infographic' | 'talking-head' | 'text-animation'
  platform: 'reels', // 'reels' | 'tiktok' | 'shorts' | 'youtube'
  outputFormat: 'mp4'
});

console.log(`Video rendered: ${videoPath}`);
```

#### Custom Remotion Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'infographic' | 'talking-head' | 'text-animation';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  content, 
  style 
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        opacity, 
        color: '#fff',
        padding: 40,
        fontSize: 48
      }}>
        <h1>{title}</h1>
        <p style={{ fontSize: 24 }}>{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h'
  },
  generation: {
    provider: 'claude', // 'claude' | 'openai'
    model: 'claude-3-opus-20240229',
    format: 'toplist',
    tone: 'expert',
    language: 'both'
  },
  video: {
    enabled: true,
    platform: 'reels',
    style: 'infographic'
  },
  publishing: {
    autoPublish: false,
    platforms: ['facebook', 'linkedin']
  }
});

// Execute full pipeline
const result = await pipeline.execute({
  keyword: 'AI marketing automation 2024'
});

console.log(result);
// {
//   research: { ... },
//   content: {
//     en: { ... },
//     vi: { ... }
//   },
//   video: {
//     path: '/videos/output.mp4',
//     thumbnail: '/thumbnails/output.jpg'
//   },
//   publishStatus: { ... }
// }
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchEngine } from '@/lib/research/engine';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeframe } = await req.json();
  
  const researcher = new ResearchEngine({ sources, timeframe });
  const results = await researcher.scan({ keyword });
  
  return NextResponse.json(results);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ClaudeContentGenerator } from '@/lib/ai/claude';

export async function POST(req: NextRequest) {
  const { research, format, tone, language } = await req.json();
  
  const generator = new ClaudeContentGenerator({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });
  
  const content = await generator.generateContent({
    research,
    format,
    tone,
    language
  });
  
  return NextResponse.json(content);
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { content, title, style, platform } = await req.json();
  
  const renderer = new VideoRenderer({
    composition: 'ContentVideo'
  });
  
  const videoPath = await renderer.render({
    content,
    title,
    style,
    platform
  });
  
  return NextResponse.json({ 
    success: true, 
    videoPath 
  });
}
```

## CLI Usage

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run remotion:render -- --composition=ContentVideo --props='{"title":"Test"}'

# Run research job
npm run research -- --keyword="AI trends" --sources=techcrunch,a16z

# Generate content from research
npm run generate -- --input=research.json --format=toplist --language=both
```

## Common Patterns

### Pattern 1: Quick Content Creation

```typescript
import { quickContent } from '@/lib/shortcuts';

// One-liner content generation
const result = await quickContent({
  keyword: 'AI marketing',
  language: 'vi',
  includeVideo: true
});
```

### Pattern 2: Scheduled Content Pipeline

```typescript
import { scheduleContentPipeline } from '@/lib/scheduler';

// Schedule daily content generation
scheduleContentPipeline({
  keywords: ['AI trends', 'marketing automation'],
  schedule: '0 9 * * *', // Daily at 9 AM
  autoPublish: true,
  platforms: ['facebook', 'linkedin']
});
```

### Pattern 3: Batch Processing

```typescript
import { batchProcess } from '@/lib/batch';

// Process multiple keywords
const results = await batchProcess({
  keywords: [
    'AI marketing',
    'content automation',
    'video generation'
  ],
  format: 'toplist',
  generateVideos: true,
  parallel: 3 // Process 3 at a time
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 50 requests per minute
});

await limiter.throttle(async () => {
  return await claudeGen.generateContent({ ... });
});
```

### Video Rendering Memory Issues

```typescript
// Reduce video quality for large batches
const videoGen = new VideoRenderer({
  composition: 'ContentVideo',
  fps: 30,
  durationInFrames: 300,
  quality: 'medium', // 'low' | 'medium' | 'high'
  concurrency: 1 // Render one at a time
});
```

### Research Data Quality

```typescript
// Filter low-quality sources
const insights = research.extractInsights({
  minRelevance: 0.8, // Higher threshold
  maxResults: 5,
  excludeSources: ['low-quality-blog.com'],
  requireStatistics: true // Only include data-backed insights
});
```

### Bilingual Content Consistency

```typescript
// Ensure consistent translation
const content = await openaiGen.generateBilingual({
  topic: 'AI trends',
  research: insights,
  translationStrategy: 'parallel', // Generate both simultaneously
  ensureConsistency: true,
  reviewTranslation: true
});
```

## Best Practices

1. **Always validate research data** before content generation
2. **Use environment variables** for all API keys
3. **Implement rate limiting** for AI API calls
4. **Cache research results** to avoid redundant API calls
5. **Test video compositions** with sample data before batch rendering
6. **Monitor API costs** with usage tracking
7. **Implement error recovery** for long-running pipelines

## Advanced Configuration

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    cacheEnabled: true,
    cacheDuration: 3600, // 1 hour
    maxSources: 10,
    timeout: 30000
  },
  ai: {
    retryAttempts: 3,
    retryDelay: 1000,
    temperature: 0.7,
    maxTokens: 4000
  },
  video: {
    maxConcurrent: 2,
    outputDir: './public/videos',
    cleanupOldVideos: true,
    maxAge: 604800 // 7 days
  }
};
```

This skill covers the essential usage patterns for the Ultimate AI Content Pipeline, enabling AI coding agents to assist developers in building automated content workflows from research to video generation.
```
