---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scripting, posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from blog posts automatically
  - crawl news and create content with AI
  - set up automated marketing content workflow
  - build AI content generation pipeline
  - create social media videos from articles
  - automate research to video content flow
  - use Remotion for content video rendering
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a TypeScript-based automated content creation system that handles the complete workflow from research to video generation. It crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos automatically with Remotion.

## What It Does

This system provides an end-to-end content automation pipeline:

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter, LinkedIn for fresh content (last 24h)
- **AI Content Generation**: Creates blog posts, POV pieces, case studies, how-tos in multiple languages
- **Automatic Video Rendering**: Converts written content to videos/infographics using Remotion
- **Multi-Platform Export**: Generates content optimized for Reels, TikTok, Shorts

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

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── components/       # React/Next.js UI components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News/content crawlers
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── pages/
│   │   └── api/         # API routes
│   └── remotion/        # Remotion video templates
├── public/              # Static assets
└── package.json
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
import { NewsCrawler } from '@/lib/crawlers/news-crawler';

// Initialize crawler with sources
const crawler = new NewsCrawler({
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Fetch and analyze content
const research = await crawler.crawl();

// Extract insights
const insights = await crawler.extractInsights(research, {
  minRelevance: 0.7,
  maxResults: 10
});
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about ${topic}. 
                Include data-backed insights and current trends.
                Write in both English and Vietnamese.`
    }],
  });

  return message.content[0].text;
}

// Usage
const article = await generateContent('AI in Marketing', 'case-study');
```

### 3. Multi-Format Content Generation

```typescript
import { ContentGenerator } from '@/lib/generators/content-generator';

const generator = new ContentGenerator({
  aiProvider: 'claude', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate multiple formats from research
const content = await generator.createMultiFormat({
  research: insights,
  formats: ['toplist', 'pov', 'how-to'],
  languages: ['en', 'vi'],
  tone: 'professional', // 'friendly', 'humorous'
  targetAudience: 'marketers'
});

// Returns:
// {
//   toplist: { en: '...', vi: '...' },
//   pov: { en: '...', vi: '...' },
//   howTo: { en: '...', vi: '...' }
// }
```

### 4. OpenAI Integration (Alternative)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing.'
      },
      {
        role: 'user',
        content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/article-video';

async function generateVideo(articleData: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: {
      title: articleData.title,
      content: articleData.content,
      style: 'modern', // 'minimal', 'vibrant'
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${articleData.slug}.mp4`,
    inputProps: composition.inputProps,
  });
}
```

### 6. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoRenderer: 'remotion',
  });

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z'],
    depth: 'comprehensive'
  });

  // Step 2: Generate Content
  const content = await pipeline.generateContent({
    research,
    format: 'case-study',
    languages: ['en', 'vi'],
  });

  // Step 3: Create Videos
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['reels', 'tiktok', 'shorts'],
    aspectRatios: {
      reels: '9:16',
      tiktok: '9:16',
      shorts: '9:16'
    }
  });

  // Step 4: Export
  return {
    articles: content,
    videos: videos,
    metadata: {
      keyword,
      timestamp: new Date(),
      sources: research.sources
    }
  };
}

// Execute
const result = await runPipeline('AI Marketing Automation');
```

## API Routes

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { NewsCrawler } from '@/lib/crawlers/news-crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeRange } = req.body;

  try {
    const crawler = new NewsCrawler({ sources, timeRange, keywords: [keyword] });
    const research = await crawler.crawl();
    
    res.status(200).json({ success: true, data: research });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/lib/generators/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { topic, format, language, tone } = req.body;

  const generator = new ContentGenerator({
    aiProvider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const content = await generator.create({
    topic,
    format,
    language,
    tone
  });

  res.status(200).json({ content });
}
```

## Configuration

### Content Templates

```typescript
// lib/config/templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
    includeData: true
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'conclusion'],
    tone: 'thought-leadership',
    length: 'medium'
  },
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results', 'takeaways'],
    includeMetrics: true,
    visualize: true
  },
  howTo: {
    structure: ['overview', 'steps', 'tips', 'resources'],
    stepFormat: 'numbered',
    includeExamples: true
  }
};
```

### Video Settings

```typescript
// remotion/config.ts
export const videoConfig = {
  fps: 30,
  durationInFrames: 900, // 30 seconds at 30fps
  platforms: {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
    landscape: { width: 1920, height: 1080 }
  },
  styles: {
    modern: { colors: ['#6366f1', '#8b5cf6'], font: 'Inter' },
    minimal: { colors: ['#000000', '#ffffff'], font: 'Helvetica' },
    vibrant: { colors: ['#f59e0b', '#ef4444'], font: 'Poppins' }
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic) => {
      const research = await crawler.crawl({ keywords: [topic] });
      const content = await generator.create({ topic, research });
      const video = await renderVideo(content);
      
      return { topic, content, video };
    })
  );
  
  return results;
}
```

### Content Scheduling

```typescript
import { schedulePost } from '@/lib/social/scheduler';

async function scheduleContent(content: any, platforms: string[]) {
  for (const platform of platforms) {
    await schedulePost({
      platform,
      content: content[platform],
      scheduledTime: new Date(Date.now() + 3600000), // 1 hour from now
      autoPost: true
    });
  }
}
```

### Multilingual Pipeline

```typescript
async function generateMultilingual(topic: string) {
  const languages = ['en', 'vi'];
  const results = {};
  
  for (const lang of languages) {
    results[lang] = await generator.create({
      topic,
      language: lang,
      localizeData: true
    });
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const tasks = keywords.map(keyword => 
  limit(() => generateContent(keyword))
);

const results = await Promise.all(tasks);
```

### Video Rendering Timeouts

```typescript
// Increase timeout for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: output,
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
});
```

### Claude/OpenAI Token Limits

```typescript
// Split long content into chunks
function chunkContent(text: string, maxTokens: number = 3000) {
  const chunks = [];
  const sentences = text.split('. ');
  let currentChunk = '';
  
  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > maxTokens * 4) {
      chunks.push(currentChunk);
      currentChunk = sentence;
    } else {
      currentChunk += sentence + '. ';
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

### Error Handling

```typescript
async function safeGenerate(topic: string) {
  try {
    return await generator.create({ topic });
  } catch (error) {
    if (error.code === 'rate_limit_exceeded') {
      await new Promise(resolve => setTimeout(resolve, 60000));
      return await generator.create({ topic });
    }
    
    if (error.code === 'context_length_exceeded') {
      // Fallback to shorter format
      return await generator.create({ topic, format: 'summary' });
    }
    
    throw error;
  }
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Access at http://localhost:3000
```

## Building for Production

```bash
# Build the application
npm run build

# Start production server
npm start
```

This skill enables AI agents to automate complete content marketing workflows from research through video production using state-of-the-art AI models and rendering technology.
