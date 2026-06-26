---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from blog posts automatically
  - crawl news sources and create content
  - build AI content pipeline with Remotion
  - automate marketing content research and writing
  - create multilingual content with Claude API
  - generate social media videos from articles
  - setup automated content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

A complete TypeScript-based automated content creation system that handles research (web crawling), content generation (Claude/OpenAI), and video rendering (Remotion). Transforms a single keyword into fully formatted articles and social media videos in multiple languages.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-Research**: Crawls TechCrunch, a16z, X (Twitter), LinkedIn for recent news (24h)
- **Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 or OpenAI
- **Multilingual**: Generates English and Vietnamese content simultaneously
- **Video Rendering**: Converts articles into infographics and short-form videos using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Setup environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_TOKEN=your_linkedin_token

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── lib/
│   │   ├── research/        # Web crawling & data extraction
│   │   ├── ai/              # Claude & OpenAI integrations
│   │   ├── content/         # Content generation logic
│   │   └── video/           # Remotion video rendering
│   ├── app/                 # Next.js app router
│   ├── components/          # React components
│   └── types/               # TypeScript definitions
├── remotion/                # Video templates
└── public/                  # Static assets
```

## Core Modules & Usage

### 1. Research Module (Web Crawling)

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Crawl news sources for a topic
const researchData = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxResults: 20
});

// Returns structured data
// {
//   articles: Article[],
//   insights: Insight[],
//   trends: Trend[],
//   statistics: Statistic[]
// }
```

**Crawler Configuration**:

```typescript
// src/lib/research/config.ts
export const crawlerConfig = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com',
      selector: 'article.post-block',
      rateLimit: 2000 // ms between requests
    },
    a16z: {
      url: 'https://a16z.com/posts',
      selector: '.post-card'
    }
  },
  headers: {
    'User-Agent': 'Mozilla/5.0...',
    'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
  }
};
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate article using Claude
const article = await generateContent({
  topic: 'AI automation trends 2024',
  format: 'listicle', // 'pov', 'case-study', 'how-to'
  language: 'en', // 'vi' for Vietnamese
  tone: 'professional', // 'friendly', 'humorous'
  researchData: researchData,
  provider: 'claude', // or 'openai'
  model: 'claude-3-5-sonnet-20241022'
});

// Returns formatted article
// {
//   title: string,
//   content: string,
//   metadata: {
//     wordCount: number,
//     readingTime: number,
//     seo: SEOData
//   }
// }
```

**Claude Integration**:

```typescript
// src/lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

export async function generateWithClaude(prompt: string, options?: Options) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const message = await anthropic.messages.create({
    model: options?.model || 'claude-3-5-sonnet-20241022',
    max_tokens: options?.maxTokens || 4096,
    temperature: options?.temperature || 0.7,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}
```

**OpenAI Integration**:

```typescript
// src/lib/ai/openai.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(prompt: string, options?: Options) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const completion = await openai.chat.completions.create({
    model: options?.model || 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are a professional content writer.' },
      { role: 'user', content: prompt }
    ],
    temperature: options?.temperature || 0.7,
    max_tokens: options?.maxTokens || 4096
  });

  return completion.choices[0].message.content;
}
```

### 3. Multilingual Content

```typescript
import { generateMultilingual } from '@/lib/content/multilingual';

// Generate both English and Vietnamese versions
const content = await generateMultilingual({
  topic: 'AI automation',
  format: 'listicle',
  researchData: researchData,
  languages: ['en', 'vi']
});

// Returns:
// {
//   en: { title, content, metadata },
//   vi: { title, content, metadata }
// }
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';

// Render video from article
const videoUrl = await renderVideo({
  articleId: article.id,
  template: 'infographic', // 'talking-head', 'slideshow'
  format: 'vertical', // 'square', 'horizontal'
  platform: 'tiktok', // 'reels', 'shorts'
  duration: 60 // seconds
});
```

**Remotion Video Template**:

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const Infographic: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const progress = frame / (fps * duration);

  return (
    <AbsoluteFill style={{ backgroundColor: '#fff' }}>
      <h1 style={{ opacity: Math.min(1, frame / 30) }}>
        {title}
      </h1>
      {points.map((point, index) => {
        const startFrame = (index * fps * duration) / points.length;
        const opacity = frame > startFrame ? 1 : 0;
        
        return (
          <div key={index} style={{ opacity }}>
            {point}
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

**Remotion Configuration**:

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');

// Platform presets
export const platformPresets = {
  tiktok: { width: 1080, height: 1920, fps: 30 },
  reels: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
  youtube: { width: 1920, height: 1080, fps: 30 }
};
```

## Complete Workflow Example

```typescript
// src/lib/pipeline/workflow.ts
import { researchTopic } from '@/lib/research/crawler';
import { generateMultilingual } from '@/lib/content/multilingual';
import { renderVideo } from '@/lib/video/renderer';

export async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await generateMultilingual({
    topic: keyword,
    format: 'listicle',
    researchData: research,
    languages: ['en', 'vi'],
    provider: 'claude'
  });

  // Step 3: Render videos
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all(
    ['tiktok', 'reels', 'shorts'].map(platform =>
      renderVideo({
        content: content.en,
        template: 'infographic',
        platform
      })
    )
  );

  return {
    research,
    content,
    videos
  };
}

// Usage
const result = await runContentPipeline('AI automation 2024');
console.log('✅ Pipeline complete!', result);
```

## CLI Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video
npm run render -- --composition=Infographic --output=output.mp4

# Crawl specific source
npm run crawl -- --source=techcrunch --topic="AI"

# Generate content only
npm run generate -- --topic="AI automation" --format=listicle --lang=en
```

## API Routes

The Next.js app includes REST API endpoints:

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();
  
  const data = await researchTopic({ keyword, sources, timeRange });
  
  return NextResponse.json(data);
}
```

**Usage**:
```bash
curl -X POST http://localhost:3000/api/research \
  -H "Content-Type: application/json" \
  -d '{"keyword":"AI automation","sources":["techcrunch"],"timeRange":"24h"}'
```

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
export async function POST(request: NextRequest) {
  const { topic, format, language, researchData } = await request.json();
  
  const article = await generateContent({
    topic,
    format,
    language,
    researchData,
    provider: 'claude'
  });
  
  return NextResponse.json(article);
}
```

### Render Video Endpoint

```typescript
// app/api/render/route.ts
export async function POST(request: NextRequest) {
  const { articleId, template, platform } = await request.json();
  
  const videoUrl = await renderVideo({
    articleId,
    template,
    platform
  });
  
  return NextResponse.json({ videoUrl });
}
```

## Configuration

### Content Format Templates

```typescript
// src/lib/content/templates.ts
export const contentTemplates = {
  listicle: {
    structure: ['intro', 'points', 'conclusion'],
    pointCount: 7,
    prompt: 'Create a listicle with {count} actionable points...'
  },
  pov: {
    structure: ['hook', 'thesis', 'arguments', 'counterarguments', 'conclusion'],
    prompt: 'Write a thought-provoking POV article...'
  },
  'case-study': {
    structure: ['background', 'challenge', 'solution', 'results', 'lessons'],
    prompt: 'Analyze this case study in detail...'
  },
  'how-to': {
    structure: ['intro', 'prerequisites', 'steps', 'tips', 'conclusion'],
    stepCount: 5,
    prompt: 'Create a comprehensive how-to guide...'
  }
};
```

### Tone Presets

```typescript
// src/lib/ai/tone.ts
export const tonePresets = {
  professional: {
    systemPrompt: 'You are a professional business writer with expertise in...',
    temperature: 0.6,
    vocabulary: 'advanced'
  },
  friendly: {
    systemPrompt: 'You are a friendly content creator who explains...',
    temperature: 0.8,
    vocabulary: 'casual'
  },
  humorous: {
    systemPrompt: 'You are a witty writer who uses humor...',
    temperature: 0.9,
    vocabulary: 'playful'
  }
};
```

## Common Patterns

### Error Handling

```typescript
import { AIError, CrawlerError, RenderError } from '@/lib/errors';

try {
  const research = await researchTopic({ keyword });
} catch (error) {
  if (error instanceof CrawlerError) {
    console.error('Crawling failed:', error.source, error.message);
    // Retry with different source
  } else if (error instanceof AIError) {
    console.error('AI generation failed:', error.provider, error.message);
    // Switch to fallback provider
  }
}
```

### Caching Research Data

```typescript
import { cacheResearch, getCachedResearch } from '@/lib/cache';

const cacheKey = `research:${keyword}:${timeRange}`;
let research = await getCachedResearch(cacheKey);

if (!research) {
  research = await researchTopic({ keyword });
  await cacheResearch(cacheKey, research, 3600); // 1 hour TTL
}
```

### Batch Processing

```typescript
import { batchGenerate } from '@/lib/content/batch';

const topics = ['AI automation', 'Machine learning', 'Data analytics'];

const articles = await batchGenerate({
  topics,
  format: 'listicle',
  language: 'en',
  concurrency: 3 // Process 3 at a time
});
```

## Troubleshooting

### API Rate Limiting

If you encounter rate limits:

```typescript
// src/lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

const results = await Promise.all(
  sources.map(source => 
    limit(() => crawlSource(source))
  )
);
```

### Claude API Errors

```typescript
// Handle Claude-specific errors
try {
  const content = await generateWithClaude(prompt);
} catch (error: any) {
  if (error.status === 429) {
    // Rate limited - wait and retry
    await new Promise(resolve => setTimeout(resolve, 5000));
    return generateWithClaude(prompt);
  } else if (error.status === 500) {
    // Server error - fallback to OpenAI
    return generateWithOpenAI(prompt);
  }
  throw error;
}
```

### Remotion Rendering Issues

```bash
# Increase memory for large videos
NODE_OPTIONS=--max_old_space_size=4096 npm run render

# Debug rendering
REMOTION_VERBOSE=true npm run render -- --composition=Infographic
```

### Web Crawling Blocked

If crawlers get blocked:

```typescript
// Use rotating proxies
export const crawlerConfig = {
  proxy: {
    enabled: true,
    list: process.env.PROXY_LIST?.split(',') || []
  },
  retries: 3,
  backoff: 2000
};
```

## Performance Optimization

### Parallel Processing

```typescript
// Process research and content generation in parallel
const [research, template] = await Promise.all([
  researchTopic({ keyword }),
  loadTemplate(format)
]);

const article = await generateContent({ 
  researchData: research,
  template 
});
```

### Streaming Responses

```typescript
// Stream AI responses for faster perceived performance
import { streamText } from '@/lib/ai/stream';

const stream = await streamText({
  prompt,
  provider: 'claude'
});

for await (const chunk of stream) {
  process.stdout.write(chunk);
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline system for automated content creation, from web scraping through AI generation to video rendering.
