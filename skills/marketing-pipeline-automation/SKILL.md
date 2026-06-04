---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content pipeline"
  - "create AI content workflow with research and video"
  - "automate content creation from research to video"
  - "build marketing automation with Claude and Remotion"
  - "generate videos from AI-written content automatically"
  - "scrape news and create content pipeline"
  - "set up multi-language content automation"
  - "integrate OpenAI with video rendering pipeline"
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a comprehensive TypeScript/Next.js system that automates content creation from research to video generation. It combines web scraping, AI content generation (Claude/OpenAI), and automated video rendering (Remotion) into a single workflow.

## What This Project Does

The marketing pipeline automates:
1. **Research Phase**: Auto-crawls news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
2. **Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages
3. **Video Rendering**: Automatically generates videos and infographics from written content using Remotion
4. **Publishing**: Ready-to-post content optimized for Reels, TikTok, and Shorts

## Installation

### Prerequisites
```bash
# Node.js 18+ required
node --version

# Install pnpm (recommended) or npm
npm install -g pnpm
```

### Setup Steps

```bash
# Clone repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install
# or
npm install

# Copy environment template
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with these required variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media APIs
FACEBOOK_PAGE_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

### Run Development Server

```bash
pnpm dev
# or
npm run dev
```

Access at `http://localhost:3000`

## Key Components & API

### 1. Research Module

**Scraping News Sources**

```typescript
import { researchNews } from '@/lib/research/scraper';

// Scrape news from multiple sources
const research = await researchNews({
  keyword: 'artificial intelligence',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxResults: 50
});

console.log(research.articles); // Array of scraped articles
console.log(research.insights); // Extracted insights
```

**Custom Source Configuration**

```typescript
// lib/research/config.ts
export const researchConfig = {
  sources: {
    techcrunch: {
      enabled: true,
      url: 'https://techcrunch.com/feed/',
      selector: '.post-block',
      rateLimit: 10 // requests per minute
    },
    a16z: {
      enabled: true,
      url: 'https://a16z.com/feed/',
      selector: 'article'
    }
  },
  filters: {
    minWordCount: 300,
    excludeKeywords: ['sponsored', 'advertisement']
  }
};
```

### 2. Content Generation with AI

**Using Claude (Anthropic)**

```typescript
import { generateContent } from '@/lib/ai/claude';

const content = await generateContent({
  prompt: 'Write a toplist article about AI trends',
  context: research.insights,
  format: 'toplist',
  language: 'en',
  tone: 'expert', // 'friendly', 'humorous', 'professional'
  wordCount: 1500
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

**Using OpenAI**

```typescript
import { generateWithOpenAI } from '@/lib/ai/openai';

const variants = await generateWithOpenAI({
  baseContent: content.body,
  model: 'gpt-4',
  variants: 3, // Generate 3 versions
  optimize: 'engagement' // or 'seo', 'conversion'
});
```

**Multi-Language Generation**

```typescript
import { translateContent } from '@/lib/ai/translate';

const vietnameseVersion = await translateContent({
  content: content.body,
  targetLanguage: 'vi',
  preserveFormatting: true,
  adaptCulturally: true // Adapts idioms and cultural references
});
```

### 3. Format Templates

**Define Content Formats**

```typescript
// lib/templates/formats.ts
export const contentFormats = {
  toplist: {
    structure: [
      'introduction',
      'item_1_with_details',
      'item_2_with_details',
      // ... more items
      'conclusion',
      'cta'
    ],
    itemCount: 10,
    includeImages: true
  },
  
  pov: {
    structure: [
      'hook',
      'personal_perspective',
      'supporting_evidence',
      'counterargument',
      'conclusion'
    ],
    tone: 'personal'
  },
  
  caseStudy: {
    structure: [
      'background',
      'challenge',
      'solution',
      'results',
      'takeaways'
    ],
    includeMetrics: true
  }
};
```

**Apply Format**

```typescript
import { applyFormat } from '@/lib/templates/formatter';

const formattedContent = await applyFormat({
  rawContent: content.body,
  format: 'toplist',
  research: research.insights,
  customization: {
    itemCount: 7,
    addSourceLinks: true
  }
});
```

### 4. Video Generation with Remotion

**Basic Video Render**

```typescript
import { renderVideo } from '@/lib/video/remotion-render';

const video = await renderVideo({
  content: formattedContent,
  template: 'infographic', // 'talking-head', 'slideshow', 'animated-text'
  duration: 60, // seconds
  aspectRatio: '9:16', // for Reels/TikTok, use '16:9' for YouTube
  style: {
    theme: 'modern',
    primaryColor: '#FF6B6B',
    font: 'Inter'
  }
});

console.log(video.outputPath); // Path to rendered video
console.log(video.thumbnail); // Auto-generated thumbnail
```

**Remotion Composition Setup**

```typescript
// remotion/compositions/ContentVideo.tsx
import { Composition } from 'remotion';
import { InfographicScene } from './scenes/Infographic';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={InfographicScene}
        durationInFrames={1800} // 60 seconds at 30fps
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: '',
          points: [],
          style: {}
        }}
      />
    </>
  );
};
```

**Custom Video Scene**

```typescript
// remotion/scenes/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const InfographicScene: React.FC<{
  title: string;
  points: string[];
  style: any;
}> = ({ title, points, style }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: style.bgColor }}>
      <div style={{ opacity }}>
        <h1 style={{ color: style.primaryColor }}>{title}</h1>
        {points.map((point, idx) => (
          <div key={idx} className="point-item">
            {point}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

**Render Configuration**

```typescript
// lib/video/config.ts
export const videoConfig = {
  codec: 'h264',
  outputFormat: 'mp4',
  quality: 80,
  concurrency: 4, // Parallel rendering threads
  
  presets: {
    reels: {
      width: 1080,
      height: 1920,
      fps: 30,
      duration: 60
    },
    youtube: {
      width: 1920,
      height: 1080,
      fps: 60,
      duration: 600 // 10 minutes max
    },
    tiktok: {
      width: 1080,
      height: 1920,
      fps: 30,
      duration: 180 // 3 minutes max
    }
  }
};
```

### 5. Full Pipeline Orchestration

**Complete Workflow**

```typescript
import { runPipeline } from '@/lib/pipeline/orchestrator';

const result = await runPipeline({
  keyword: 'AI automation tools 2026',
  
  research: {
    sources: ['techcrunch', 'a16z', 'twitter'],
    depth: 'comprehensive' // 'quick', 'standard', 'comprehensive'
  },
  
  content: {
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'expert',
    length: 'long' // 'short', 'medium', 'long'
  },
  
  media: {
    generateVideo: true,
    videoTemplate: 'infographic',
    platforms: ['reels', 'tiktok', 'youtube-shorts']
  },
  
  publish: {
    autoPost: false, // Set true to auto-publish
    schedule: new Date('2026-06-10T10:00:00Z'),
    platforms: ['facebook', 'twitter', 'linkedin']
  }
});

console.log(result.contentId);
console.log(result.videoUrls);
console.log(result.publishStatus);
```

**API Route Example**

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    
    const result = await runPipeline({
      keyword: body.keyword,
      research: body.research || {},
      content: body.content || {},
      media: body.media || {}
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Common Patterns

### Pattern 1: Quick Content Generation

```typescript
// For rapid content creation without video
import { quickGenerate } from '@/lib/shortcuts';

const content = await quickGenerate('AI marketing trends', {
  format: 'toplist',
  language: 'en'
});
```

### Pattern 2: Batch Processing

```typescript
// Generate content for multiple keywords
import { batchProcess } from '@/lib/pipeline/batch';

const keywords = [
  'AI automation',
  'content marketing tools',
  'social media trends'
];

const results = await batchProcess(keywords, {
  concurrent: 3, // Process 3 at a time
  format: 'pov',
  generateVideos: true
});
```

### Pattern 3: Custom AI Model Selection

```typescript
// Mix different AI models for optimal results
const content = await generateContent({
  prompt: 'Write about AI',
  modelStrategy: {
    research: 'claude-3-opus', // Best for analysis
    writing: 'gpt-4-turbo', // Best for creative writing
    editing: 'claude-3-sonnet' // Fast refinement
  }
});
```

### Pattern 4: Scheduling & Automation

```typescript
// lib/automation/scheduler.ts
import { scheduleContent } from '@/lib/automation/scheduler';

await scheduleContent({
  frequency: 'daily', // 'hourly', 'daily', 'weekly'
  time: '09:00',
  keywords: ['rotating', 'topic', 'list'],
  autoPublish: true,
  platforms: ['facebook', 'twitter']
});
```

## Configuration Files

### Main Config

```typescript
// config/pipeline.config.ts
export default {
  ai: {
    defaultProvider: 'claude', // 'openai', 'claude'
    fallbackProvider: 'openai',
    maxTokens: 4000,
    temperature: 0.7
  },
  
  research: {
    cacheDuration: 3600, // Cache research for 1 hour
    maxSources: 10,
    minRelevanceScore: 0.7
  },
  
  content: {
    defaultLanguage: 'en',
    supportedLanguages: ['en', 'vi'],
    minWordCount: 800,
    maxWordCount: 2500
  },
  
  video: {
    defaultAspectRatio: '9:16',
    maxDuration: 180, // 3 minutes
    outputDirectory: './public/videos'
  },
  
  publishing: {
    defaultScheduleOffset: 3600000, // 1 hour in ms
    retryAttempts: 3
  }
};
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 50 requests per minute
});

await limiter.throttle(async () => {
  return await researchNews({ keyword: 'AI' });
});
```

### Issue: Video Rendering Fails

```typescript
// Add error handling and retry logic
import { renderWithRetry } from '@/lib/video/render-retry';

try {
  const video = await renderWithRetry({
    content: formattedContent,
    template: 'infographic',
    maxRetries: 3,
    timeout: 300000 // 5 minutes
  });
} catch (error) {
  console.error('Video rendering failed:', error);
  // Fallback: generate static image instead
  const fallback = await generateStaticImage(formattedContent);
}
```

### Issue: Memory Leaks During Batch Processing

```typescript
// Use streaming and chunking
import { streamBatchProcess } from '@/lib/pipeline/stream-batch';

for await (const result of streamBatchProcess(keywords)) {
  console.log('Processed:', result.keyword);
  // Results processed one at a time, reducing memory usage
}
```

### Issue: Content Quality Issues

```typescript
// Add validation and refinement
import { validateContent } from '@/lib/validation/content-validator';

const content = await generateContent({ prompt: 'AI trends' });

const validation = validateContent(content, {
  minReadabilityScore: 60,
  requireSources: true,
  checkPlagiarism: true
});

if (!validation.passed) {
  // Refine content
  const refined = await refineContent(content, validation.issues);
}
```

### Issue: Research Returns Irrelevant Data

```typescript
// Add filtering and relevance scoring
import { filterResearch } from '@/lib/research/filter';

const research = await researchNews({ keyword: 'AI' });

const filtered = filterResearch(research, {
  minRelevanceScore: 0.8,
  excludeDomains: ['spam-site.com'],
  requireKeywords: ['artificial intelligence', 'machine learning'],
  dateRange: { start: '2026-06-01', end: '2026-06-04' }
});
```

## Testing

```typescript
// tests/pipeline.test.ts
import { describe, it, expect } from 'vitest';
import { runPipeline } from '@/lib/pipeline/orchestrator';

describe('Content Pipeline', () => {
  it('should generate content from keyword', async () => {
    const result = await runPipeline({
      keyword: 'test keyword',
      content: { format: 'toplist', languages: ['en'] },
      media: { generateVideo: false }
    });
    
    expect(result.contentId).toBeDefined();
    expect(result.content.title).toContain('test keyword');
  });
});
```

## CLI Commands

```bash
# Generate content
pnpm generate --keyword "AI tools" --format toplist --language en

# Run full pipeline
pnpm pipeline --keyword "marketing automation" --video true

# Schedule content
pnpm schedule --keyword "trends" --time "09:00" --daily

# Render video only
pnpm render:video --input content.json --template infographic
```

This skill provides comprehensive guidance for using the marketing automation pipeline effectively.
