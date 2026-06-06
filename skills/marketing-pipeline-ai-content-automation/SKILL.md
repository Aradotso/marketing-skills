---
name: marketing-pipeline-ai-content-automation
description: Automated content creation pipeline using AI (Claude/OpenAI) to research, generate scripts, and produce videos with Remotion
triggers:
  - "how do I automate content creation with AI research"
  - "generate video content from text using Remotion"
  - "set up AI marketing pipeline for social media"
  - "create automated content workflow with Claude API"
  - "build AI content generator with video rendering"
  - "automate blog posts and video creation pipeline"
  - "integrate OpenAI and Claude for content automation"
  - "use Remotion to render marketing videos automatically"
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that researches trending topics, generates multi-format content (blog posts, scripts), and renders videos automatically. It combines web scraping, AI content generation (Claude 3, OpenAI), and video rendering (Remotion) into a unified TypeScript pipeline.

## What It Does

- **Auto-Research**: Crawls news sources (TechCrunch, Twitter, LinkedIn) for real-time data
- **AI Content Generation**: Creates content in multiple formats (listicles, POV articles, case studies, how-tos) using Claude/OpenAI
- **Multi-Language Support**: Generates English and Vietnamese content simultaneously
- **Video Rendering**: Automatically converts content to videos using Remotion
- **Social Media Optimization**: Exports videos in formats optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using persistence)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Web scraping & research
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Utilities
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core APIs and Usage

### 1. Content Research Pipeline

```typescript
import { researchTopic } from '@/lib/research/auto-scan';

interface ResearchResult {
  keyword: string;
  sources: Array<{
    title: string;
    url: string;
    excerpt: string;
    publishedAt: string;
  }>;
  insights: string[];
  trendingScore: number;
}

async function runResearch(keyword: string): Promise<ResearchResult> {
  const result = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });
  
  return result;
}

// Example usage
const research = await runResearch('AI automation tools');
console.log(research.insights);
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, VoiceTone } from '@/types/content';

interface ContentRequest {
  keyword: string;
  format: ContentFormat; // 'toplist' | 'pov' | 'case-study' | 'how-to'
  tone: VoiceTone; // 'expert' | 'friendly' | 'humorous'
  language: 'en' | 'vi';
  researchData?: ResearchResult;
}

async function createContent(request: ContentRequest) {
  const content = await generateContent({
    keyword: request.keyword,
    format: request.format,
    tone: request.tone,
    language: request.language,
    aiProvider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    researchContext: request.researchData
  });
  
  return {
    title: content.title,
    body: content.body,
    meta: content.metadata,
    images: content.suggestedImages
  };
}

// Generate bilingual content
const englishPost = await createContent({
  keyword: 'AI marketing automation',
  format: 'how-to',
  tone: 'expert',
  language: 'en'
});

const vietnamesePost = await createContent({
  keyword: 'AI marketing automation',
  format: 'how-to',
  tone: 'friendly',
  language: 'vi'
});
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/render';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

interface VideoConfig {
  contentId: string;
  title: string;
  script: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

async function generateVideo(config: VideoConfig) {
  // Bundle Remotion composition
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  // Render video
  const { outputPath } = await renderMedia({
    composition: {
      id: config.platform,
      width: config.platform === 'reels' ? 1080 : 1080,
      height: config.platform === 'reels' ? 1920 : 1920,
      fps: 30,
      durationInFrames: config.duration * 30
    },
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${config.contentId}.mp4`,
    inputProps: {
      title: config.title,
      script: config.script
    }
  });
  
  return outputPath;
}

// Example: Create video from content
const videoPath = await generateVideo({
  contentId: 'post-123',
  title: englishPost.title,
  script: englishPost.body.split('\n\n').slice(0, 5),
  platform: 'reels',
  duration: 30
});
```

### 4. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    researchEnabled: true,
    videoEnabled: true,
    languages: ['en', 'vi']
  });
  
  // Execute full pipeline
  const result = await pipeline.execute({
    keyword,
    formats: ['how-to', 'toplist'],
    videoPlatforms: ['reels', 'tiktok'],
    schedule: {
      autoPost: false,
      platforms: ['facebook', 'linkedin']
    }
  });
  
  return {
    research: result.research,
    content: result.generatedContent,
    videos: result.renderedVideos,
    status: result.status
  };
}

// Usage
const output = await runFullPipeline('AI content automation 2024');
console.log(`Generated ${output.content.length} posts`);
console.log(`Rendered ${output.videos.length} videos`);
```

## Common Patterns

### Pattern 1: Research-First Content

```typescript
async function researchDrivenContent(topic: string) {
  // 1. Research phase
  const research = await researchTopic({
    keyword: topic,
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  });
  
  // 2. Generate content with research context
  const content = await generateContent({
    keyword: topic,
    format: 'case-study',
    tone: 'expert',
    language: 'en',
    researchContext: research
  });
  
  // 3. Include sources in content
  const enrichedContent = {
    ...content,
    sources: research.sources.map(s => ({
      title: s.title,
      url: s.url
    }))
  };
  
  return enrichedContent;
}
```

### Pattern 2: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        const content = await generateContent({
          keyword,
          format: 'toplist',
          tone: 'friendly',
          language: 'en',
          aiProvider: 'openai'
        });
        return { keyword, success: true, content };
      } catch (error) {
        return { keyword, success: false, error };
      }
    })
  );
  
  const successful = results.filter(r => r.success);
  console.log(`Generated ${successful.length}/${keywords.length} posts`);
  
  return successful;
}
```

### Pattern 3: Multi-Format Video Creation

```typescript
async function createMultiPlatformVideos(contentId: string, script: string[]) {
  const platforms = ['reels', 'tiktok', 'shorts'] as const;
  
  const videos = await Promise.all(
    platforms.map(platform => 
      generateVideo({
        contentId: `${contentId}-${platform}`,
        title: script[0],
        script: script.slice(1),
        platform,
        duration: platform === 'reels' ? 30 : 60
      })
    )
  );
  
  return videos.map((path, i) => ({
    platform: platforms[i],
    path,
    ready: true
  }));
}
```

## CLI Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (if separate CLI)
npm run render -- --props='{"contentId":"123"}' --output=out/video.mp4

# Run research script
npm run research -- --keyword="AI tools" --sources=twitter,linkedin

# Generate content only
npm run generate -- --keyword="marketing" --format=how-to --lang=en
```

## Configuration Files

### `remotion.config.ts`

```typescript
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

### Custom AI Provider Configuration

```typescript
// src/lib/ai/config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4000,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

async function generateWithRetry(params: ContentRequest) {
  return retry(
    async () => await generateContent(params),
    {
      maxAttempts: 3,
      delayMs: 1000,
      backoff: 'exponential'
    }
  );
}
```

### Issue: Remotion Render Fails

```bash
# Check Remotion dependencies
npx remotion versions

# Increase memory limit for large videos
NODE_OPTIONS=--max-old-space-size=8192 npm run render
```

### Issue: Research Scraping Blocked

```typescript
// Add user agent and delays
const researchConfig = {
  headers: {
    'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)'
  },
  delayBetweenRequests: 2000, // ms
  maxRetries: 3
};
```

### Issue: Content Quality Problems

```typescript
// Adjust AI parameters for better output
const betterContent = await generateContent({
  keyword: 'topic',
  format: 'how-to',
  tone: 'expert',
  language: 'en',
  aiProvider: 'claude',
  options: {
    temperature: 0.5, // Lower = more focused
    maxTokens: 6000,  // Longer content
    systemPrompt: 'You are an expert marketing content writer...'
  }
});
```

## Advanced Usage

### Custom Video Template

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  script: string[];
}> = ({ title, script }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <h1 style={{ opacity: Math.min(1, frame / 30) }}>
        {title}
      </h1>
      {/* Add your custom animations */}
    </AbsoluteFill>
  );
};
```

### Webhook Integration for Auto-Publishing

```typescript
// src/app/api/webhook/publish/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { contentId, platforms } = await request.json();
  
  // Publish to social media platforms
  const results = await publishContent(contentId, platforms);
  
  return NextResponse.json({ success: true, results });
}
```

This skill enables AI agents to help developers build and customize automated content creation pipelines with AI research, generation, and video rendering capabilities.
