---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, auto-posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - set up AI marketing content workflow
  - generate videos from research automatically
  - create content pipeline with Claude and OpenAI
  - automate research to video production
  - build AI content automation system
  - integrate Remotion for video generation
  - scrape news and generate marketing content
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a complete automated content creation system that handles research, scriptwriting, auto-posting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh content from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates multi-format content (toplists, POV, case studies, how-tos) in multiple languages
3. **Auto-Posting**: Publishes to social platforms automatically
4. **Video Generation**: Renders infographics and short-form videos using Remotion

Perfect for content creators, marketers, and businesses looking to scale content production by 90%.

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or pnpm
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Create environment file
cp .env.example .env
```

### Environment Configuration

Edit `.env` file with required API keys:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license

# Social Media (Optional)
FACEBOOK_PAGE_TOKEN=your_fb_page_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
pnpm dev
```

Access at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── scraper/     # Content crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── publisher/   # Auto-posting modules
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── remotion/            # Video templates
├── public/              # Static assets
└── .env                 # Environment variables
```

## Key Features & Usage

### 1. Content Research & Scraping

```typescript
import { ResearchEngine } from '@/lib/scraper/research-engine';

// Initialize research engine
const researcher = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Execute research
const insights = await researcher.scrape();

console.log(insights);
// {
//   articles: [...],
//   trends: [...],
//   keyInsights: [...],
//   dataPoints: [...]
// }
```

### 2. AI Content Generation

#### Using Claude 3

```typescript
import { ClaudeContentGenerator } from '@/lib/ai/claude-generator';

const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content from research
const content = await generator.generateContent({
  research: insights,
  format: 'toplist',
  tone: 'expert',
  language: 'vi',
  targetAudience: 'marketers'
});

console.log(content);
// {
//   title: "Top 10 AI Marketing Tools...",
//   body: "...",
//   seo: {...},
//   socialPosts: {...}
// }
```

#### Using OpenAI

```typescript
import { OpenAIContentGenerator } from '@/lib/ai/openai-generator';

const openaiGenerator = new OpenAIContentGenerator({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

// Generate bilingual content
const bilingualContent = await openaiGenerator.generateBilingual({
  topic: 'AI Content Automation',
  formats: ['case-study', 'how-to'],
  languages: ['en', 'vi']
});
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

// Render video from content
async function generateContentVideo(content: ContentData) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      bullets: content.keyPoints,
      branding: content.brandAssets
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: composition.inputProps,
  });
  
  return `out/${content.id}.mp4`;
}

// Usage
const videoPath = await generateContentVideo(content);
console.log('Video generated:', videoPath);
```

### 4. Auto-Publishing

```typescript
import { ContentPublisher } from '@/lib/publisher/publisher';

const publisher = new ContentPublisher({
  platforms: ['facebook', 'linkedin', 'twitter'],
  credentials: {
    facebook: { token: process.env.FACEBOOK_PAGE_TOKEN },
    linkedin: { token: process.env.LINKEDIN_ACCESS_TOKEN },
    twitter: { apiKey: process.env.TWITTER_API_KEY }
  }
});

// Publish content
const results = await publisher.publish({
  content: content,
  video: videoPath,
  schedule: new Date('2024-06-20T10:00:00Z')
});

console.log('Published to:', results.platforms);
```

## Complete Workflow Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  try {
    // Step 1: Research
    console.log('🔍 Researching...');
    const research = await pipeline.research({
      keyword,
      sources: ['techcrunch', 'twitter'],
      depth: 'deep'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      research,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional'
    });

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const video = await pipeline.renderVideo({
      content,
      template: 'reels',
      duration: 60
    });

    // Step 4: Publish
    console.log('📤 Publishing...');
    const published = await pipeline.publish({
      content,
      video,
      platforms: ['facebook', 'linkedin'],
      schedule: 'immediate'
    });

    console.log('✅ Pipeline complete!', published);
    
    return {
      research,
      content,
      video,
      published
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run pipeline
runContentPipeline('AI marketing automation')
  .then(result => console.log('Success:', result))
  .catch(err => console.error('Failed:', err));
```

## API Routes

### Next.js API Endpoints

```typescript
// app/api/research/route.ts
export async function POST(request: Request) {
  const { keyword, sources } = await request.json();
  
  const researcher = new ResearchEngine({ sources });
  const data = await researcher.scrape(keyword);
  
  return Response.json(data);
}

// app/api/generate/route.ts
export async function POST(request: Request) {
  const { research, format, language } = await request.json();
  
  const generator = new ClaudeContentGenerator({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const content = await generator.generateContent({
    research,
    format,
    language
  });
  
  return Response.json(content);
}

// app/api/render-video/route.ts
export async function POST(request: Request) {
  const { contentId } = await request.json();
  
  const videoPath = await generateContentVideo(contentId);
  
  return Response.json({ videoPath });
}
```

## Common Patterns

### Custom Content Format

```typescript
// Define custom format template
const customFormat = {
  name: 'success-story',
  structure: {
    hook: 'attention-grabbing opening',
    problem: 'pain point description',
    solution: 'how product solved it',
    results: 'metrics and outcomes',
    cta: 'call to action'
  },
  tone: 'inspiring'
};

// Use in generation
const content = await generator.generateContent({
  research: insights,
  customFormat,
  language: 'vi'
});
```

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline(config);
  
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await pipeline.research({ keyword });
      const content = await pipeline.generateContent({ research });
      return { keyword, content };
    })
  );
  
  return results;
}

// Generate for multiple topics
const batch = await batchGenerate([
  'AI tools',
  'marketing automation',
  'content creation'
]);
```

### Schedule Publishing Queue

```typescript
import { ScheduleQueue } from '@/lib/publisher/queue';

const queue = new ScheduleQueue({
  database: process.env.DATABASE_URL
});

// Add to queue
await queue.add({
  content,
  video,
  platform: 'facebook',
  scheduledFor: new Date('2024-06-21T14:00:00Z')
});

// Process queue (run as cron job)
await queue.processScheduled();
```

## Configuration

### Customizing AI Models

```typescript
// config/ai-models.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.8
  }
};
```

### Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <h1 style={{ opacity: Math.min(1, frame / 30) }}>
        {title}
      </h1>
      {points.map((point, i) => (
        <p key={i} style={{ 
          opacity: Math.min(1, (frame - (i * 30)) / 30) 
        }}>
          {point}
        </p>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

// Wrap API calls
await limiter.execute(async () => {
  return await generator.generateContent(params);
});
```

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

try {
  await pipeline.run(params);
} catch (error) {
  if (error instanceof PipelineError) {
    switch (error.stage) {
      case 'research':
        console.error('Research failed:', error.message);
        // Retry with different sources
        break;
      case 'generation':
        console.error('Content generation failed:', error.message);
        // Try fallback model
        break;
      case 'video':
        console.error('Video rendering failed:', error.message);
        // Skip video, publish text only
        break;
    }
  }
}
```

### Debugging Remotion Renders

```bash
# Preview composition locally
npm run remotion preview

# Check specific composition
npm run remotion preview src/remotion/index.ts -- --props='{"title":"Test"}'

# Render with debug info
npm run remotion render src/remotion/index.ts ContentVideo out/test.mp4 --log=verbose
```

### Memory Issues with Large Batches

```typescript
// Process in chunks to avoid memory overflow
async function processBatches(items: any[], batchSize = 5) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => pipeline.process(item))
    );
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Performance Optimization

```typescript
// Cache research results
import { CacheManager } from '@/lib/utils/cache';

const cache = new CacheManager({
  ttl: 3600 // 1 hour
});

async function getCachedResearch(keyword: string) {
  const cached = await cache.get(`research:${keyword}`);
  if (cached) return cached;
  
  const fresh = await researcher.scrape(keyword);
  await cache.set(`research:${keyword}`, fresh);
  
  return fresh;
}
```

This skill provides comprehensive coverage for working with the Marketing Pipeline AI Content Automation system, enabling AI agents to effectively assist developers in automating their entire content creation workflow.
