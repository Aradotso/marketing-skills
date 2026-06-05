---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - generate content automatically from research to video
  - use the marketing pipeline to create social media content
  - automate content creation with Claude and OpenAI APIs
  - create videos from text content using Remotion
  - build an automated content workflow for social media
  - research and generate content scripts automatically
  - set up AI-powered content automation pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation pipeline that handles research, scriptwriting, and video generation. It automatically crawls fresh data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

## What It Does

- **Auto-Research**: Scrapes and analyzes real-time data from major tech/business sources
- **AI Content Generation**: Creates content in multiple formats (toplists, POV, case studies, how-tos) in English and Vietnamese
- **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
- **Multi-Platform**: Optimized for Reels, TikTok, Shorts with proper aspect ratios

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

### Environment Variables

Create a `.env` file with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Research/scraping modules
│   │   └── video/       # Remotion video rendering
│   ├── api/             # API routes
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Collection

```typescript
import { researchTopic } from '@/lib/research/auto-scan';

// Automatically research a topic
async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });

  return {
    insights: research.insights,
    data: research.rawData,
    trends: research.trends
  };
}

// Usage
const insights = await gatherInsights('AI automation trends');
console.log(insights.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Create a ${format} article about ${topic} in ${language}.
  Use the following structure:
  - Engaging headline
  - Introduction with hook
  - Main content (3-5 sections)
  - Conclusion with CTA
  
  Tone: Professional yet approachable`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

// Usage
const article = await generateContent(
  'Marketing automation with AI',
  'how-to',
  'en'
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateScript(topic: string, style: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${style} content.`
      },
      {
        role: 'user',
        content: `Create a short-form video script about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 1000
  });

  return completion.choices[0].message.content;
}

// Usage
const script = await generateScript('AI content tools', 'educational');
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoProps {
  title: string;
  content: string[];
  theme: 'dark' | 'light';
}

async function generateVideo(props: VideoProps) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: props,
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: props,
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Tools',
  content: ['Tool 1', 'Tool 2', 'Tool 3', 'Tool 4', 'Tool 5'],
  theme: 'dark'
});
```

### 5. Complete Pipeline Integration

```typescript
import { researchTopic } from '@/lib/research/auto-scan';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('📡 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
      language: 'en'
    });

    // Step 2: Generate content
    console.log('🧠 Generating content...');
    const content = await generateContent(
      keyword,
      'toplist',
      'en'
    );

    // Step 3: Parse content for video
    const contentArray = content.split('\n\n').filter(Boolean);

    // Step 4: Generate video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: keyword,
      content: contentArray.slice(0, 5),
      theme: 'dark'
    });

    return {
      research: research.insights,
      article: content,
      video: videoPath,
      status: 'success'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline('AI marketing automation');
console.log('Pipeline complete:', result);
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/auto-scan';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  const data = await researchTopic({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeframe: timeframe || '24h',
    language: 'en'
  });

  return NextResponse.json(data);
}
```

## Common Patterns

### Content Format Templates

```typescript
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

const formatTemplates: Record<ContentFormat, string> = {
  'toplist': `# Top {N} {Topic}
  
  Introduction explaining why this matters...
  
  ## 1. First Item
  Details...
  
  ## 2. Second Item
  Details...`,
  
  'pov': `# My Take on {Topic}
  
  Personal perspective introduction...
  
  ## Why This Matters
  Main argument...
  
  ## What Others Miss
  Unique insight...`,
  
  'case-study': `# Case Study: {Topic}
  
  ## Background
  Context...
  
  ## Challenge
  Problem faced...
  
  ## Solution
  How it was solved...
  
  ## Results
  Outcomes and metrics...`,
  
  'how-to': `# How to {Topic}
  
  Introduction to the process...
  
  ## Step 1: {Action}
  Details...
  
  ## Step 2: {Action}
  Details...
  
  ## Conclusion
  Summary and next steps...`
};
```

### Multi-Language Support

```typescript
interface ContentConfig {
  language: 'en' | 'vi';
  tone: 'professional' | 'casual' | 'humorous';
  audience: 'beginner' | 'intermediate' | 'expert';
}

async function generateLocalizedContent(
  topic: string,
  config: ContentConfig
) {
  const languagePrompts = {
    en: 'Write in clear, concise English',
    vi: 'Viết bằng tiếng Việt tự nhiên, dễ hiểu'
  };

  const tonePrompts = {
    professional: 'Use professional, authoritative tone',
    casual: 'Use friendly, conversational tone',
    humorous: 'Use light humor and engaging style'
  };

  const prompt = `${languagePrompts[config.language]}.
  ${tonePrompts[config.tone]}.
  Target audience: ${config.audience}.
  Topic: ${topic}`;

  // Generate with your chosen AI provider
  return await generateContent(topic, 'how-to', config.language);
}
```

### Video Aspect Ratios

```typescript
const platformConfigs = {
  reels: { width: 1080, height: 1920, fps: 30 }, // 9:16
  tiktok: { width: 1080, height: 1920, fps: 30 }, // 9:16
  shorts: { width: 1080, height: 1920, fps: 30 }, // 9:16
  youtube: { width: 1920, height: 1080, fps: 30 }, // 16:9
  instagram: { width: 1080, height: 1080, fps: 30 } // 1:1
};

async function renderForPlatform(
  content: VideoProps,
  platform: keyof typeof platformConfigs
) {
  const config = platformConfigs[platform];
  
  return await renderMedia({
    composition: {
      ...composition,
      width: config.width,
      height: config.height,
      fps: config.fps
    },
    // ... other options
  });
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Start Remotion studio (for video preview)
npm run remotion:studio

# Build for production
npm run build

# Start production server
npm start
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      if (!this.processing) {
        this.process();
      }
    });
  }
  
  private async process() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1 req/sec
    }
    
    this.processing = false;
  }
}

const limiter = new RateLimiter();
const result = await limiter.add(() => generateContent(topic, 'how-to', 'en'));
```

### Memory Issues with Video Rendering

```typescript
// Use smaller chunks and clear cache
import { clearCache } from '@remotion/renderer';

async function renderLargeVideo(props: VideoProps) {
  try {
    const result = await generateVideo(props);
    await clearCache(); // Clear Remotion cache after rendering
    return result;
  } catch (error) {
    if (error.message.includes('memory')) {
      console.log('Retrying with reduced quality...');
      // Implement fallback logic
    }
    throw error;
  }
}
```

### Content Quality Issues

```typescript
// Add validation layer
function validateContent(content: string): boolean {
  const checks = {
    minLength: content.length > 300,
    hasHeadline: /^#+\s.+/m.test(content),
    hasSections: content.split('\n\n').length >= 3,
    noPlaceholders: !content.includes('[TODO]')
  };
  
  return Object.values(checks).every(Boolean);
}

async function generateValidatedContent(topic: string) {
  let attempts = 0;
  const maxAttempts = 3;
  
  while (attempts < maxAttempts) {
    const content = await generateContent(topic, 'how-to', 'en');
    
    if (validateContent(content)) {
      return content;
    }
    
    attempts++;
    console.log(`Content validation failed, retry ${attempts}/${maxAttempts}`);
  }
  
  throw new Error('Failed to generate valid content after max attempts');
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement error handling** at each pipeline stage
3. **Cache research results** to avoid redundant API calls
4. **Monitor API usage** to stay within rate limits
5. **Validate generated content** before rendering videos
6. **Use TypeScript types** for better type safety
7. **Test video outputs** before bulk generation
