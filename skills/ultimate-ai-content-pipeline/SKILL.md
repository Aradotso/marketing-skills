---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, multi-format content generation, and video rendering
triggers:
  - "generate content from keyword research"
  - "create AI-powered marketing content with video"
  - "automate content pipeline with Claude and OpenAI"
  - "build content automation workflow"
  - "generate multilingual marketing content"
  - "create video content from text automatically"
  - "set up AI content research and generation"
  - "automate social media content creation"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript/Next.js project provides an end-to-end automated content creation pipeline that:
1. **Crawls and researches** trending topics from sources like TechCrunch, a16z, Twitter, LinkedIn
2. **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Renders videos** automatically using Remotion for social media platforms
4. Supports **multilingual output** (English & Vietnamese) with customizable tone of voice

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# App Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (OpenAI, Claude)
│   │   ├── crawler/     # Web scraping utilities
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core Functionality

### 1. Content Research & Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trends
  };
}
```

### 2. AI Content Generation

**Using Claude 3:**

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `
Based on the following research data, create a ${format} article in ${language}:

Research: ${JSON.stringify(research)}

Requirements:
- Use data-backed insights
- Include statistics and real examples
- Tone: Professional yet engaging
- Length: 1500-2000 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
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

**Using OpenAI:**

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  research: any,
  options: {
    format: string;
    language: string;
    tone: 'expert' | 'friendly' | 'humorous';
  }
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${options.format} articles. Write in ${options.language} with a ${options.tone} tone.`
      },
      {
        role: 'user',
        content: `Create content based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 3. Multi-Format Content Pipeline

```typescript
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationOptions {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone?: Tone;
  aiProvider: 'claude' | 'openai';
}

async function generateContent(options: ContentGenerationOptions) {
  // Step 1: Research
  const research = await researchTopic({
    keyword: options.keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Step 2: Generate content
  let content: string;
  
  if (options.aiProvider === 'claude') {
    content = await generateContentWithClaude(
      research,
      options.format,
      options.language
    );
  } else {
    content = await generateContentWithOpenAI(research, {
      format: options.format,
      language: options.language,
      tone: options.tone || 'expert'
    });
  }

  // Step 3: Extract key points for video
  const keyPoints = await extractKeyPoints(content);

  return {
    content,
    research,
    keyPoints,
    metadata: {
      wordCount: content.split(' ').length,
      readingTime: Math.ceil(content.split(' ').length / 200),
      format: options.format,
      language: options.language
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoOptions {
  content: string;
  keyPoints: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(options: VideoOptions) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[options.platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      keyPoints: options.keyPoints,
      content: options.content.substring(0, 500) // First 500 chars
    },
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `output-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    overwrite: true,
  });

  return {
    videoPath: outputPath,
    duration: composition.durationInFrames / composition.fps,
    dimensions: { width, height }
  };
}
```

### 5. Complete Pipeline Example

```typescript
import { generateContent } from '@/lib/content/generator';
import { generateVideo } from '@/lib/video/renderer';

async function runContentPipeline(keyword: string) {
  try {
    // Generate English version
    const englishContent = await generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      aiProvider: 'claude',
      tone: 'expert'
    });

    // Generate Vietnamese version
    const vietnameseContent = await generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      aiProvider: 'claude',
      tone: 'friendly'
    });

    // Generate videos for both languages
    const englishVideo = await generateVideo({
      content: englishContent.content,
      keyPoints: englishContent.keyPoints,
      platform: 'reels'
    });

    const vietnameseVideo = await generateVideo({
      content: vietnameseContent.content,
      keyPoints: vietnameseContent.keyPoints,
      platform: 'tiktok'
    });

    return {
      english: {
        content: englishContent,
        video: englishVideo
      },
      vietnamese: {
        content: vietnameseContent,
        video: vietnameseVideo
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI Marketing Automation 2024')
  .then(result => {
    console.log('Content generated successfully!');
    console.log('English video:', result.english.video.videoPath);
    console.log('Vietnamese video:', result.vietnamese.video.videoPath);
  });
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, aiProvider } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await generateContent({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      aiProvider: aiProvider || 'claude'
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { content, keyPoints, platform } = body;

    const video = await generateVideo({
      content,
      keyPoints,
      platform: platform || 'reels'
    });

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(video.videoPath)}`,
      metadata: video
    });
  } catch (error) {
    console.error('Video generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate video' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render video (if using Remotion CLI)
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        aiProvider: 'claude'
      })
    )
  );

  return results;
}
```

### Scheduling Content Creation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline(topic);
  }
});
```

### Custom Tone Configuration

```typescript
const tonePrompts = {
  expert: 'Write with authority, using industry jargon and deep insights',
  friendly: 'Use conversational language, be approachable and warm',
  humorous: 'Include witty remarks and light humor while staying informative'
};

async function generateWithCustomTone(
  research: any,
  tone: keyof typeof tonePrompts
) {
  const systemPrompt = `${tonePrompts[tone]}. ${baseInstructions}`;
  // Continue with generation...
}
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateWithRateLimit(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword => 
      limit(() => generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        aiProvider: 'claude'
      }))
    )
  );
}
```

### Video Rendering Memory Issues

```typescript
// Increase Node.js memory limit
// In package.json scripts:
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### Research Data Validation

```typescript
function validateResearchData(research: any) {
  if (!research.articles || research.articles.length === 0) {
    throw new Error('No research articles found');
  }

  if (!research.insights || research.insights.length === 0) {
    console.warn('Limited insights available, using fallback data');
    research.insights = generateFallbackInsights(research.articles);
  }

  return research;
}
```

### Error Handling Best Practices

```typescript
async function robustContentGeneration(options: ContentGenerationOptions) {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      const result = await generateContent(options);
      return result;
    } catch (error) {
      attempt++;
      console.error(`Attempt ${attempt} failed:`, error);
      
      if (attempt >= maxRetries) {
        throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
      }
      
      // Exponential backoff
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
}
```
