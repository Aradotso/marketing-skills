---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for multi-format content pipelines.
triggers:
  - how do I automate content research and generation
  - set up AI content pipeline with video rendering
  - create automated marketing content workflow
  - generate videos from written content automatically
  - build content pipeline with Claude and OpenAI
  - automate social media content creation end-to-end
  - research and generate multilingual content with AI
  - set up Remotion video generation pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive content automation system that handles research, scriptwriting, multi-format content generation, and automatic video rendering using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

The Marketing Pipeline Share system automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Support**: Generates parallel English and Vietnamese versions
4. **Video Generation**: Automatically renders infographics and short videos using Remotion
5. **Multi-Platform Optimization**: Exports videos in formats optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```env
# AI Service APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research Data Sources
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── generators/  # Content format generators
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research Module

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Crawl and research a topic
const researchData = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  language: 'en'
});

console.log(researchData);
// {
//   articles: [...],
//   insights: [...],
//   trending: [...],
//   dataPoints: [...]
// }
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Using Claude
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const claudeContent = await generateContent({
  client: anthropic,
  provider: 'claude',
  topic: 'AI in Marketing',
  format: 'toplist',
  tone: 'professional',
  language: 'en',
  researchData: researchData
});

// Using OpenAI
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

const openaiContent = await generateContent({
  client: openai,
  provider: 'openai',
  topic: 'AI in Marketing',
  format: 'case-study',
  tone: 'friendly',
  language: 'vi',
  researchData: researchData
});
```

### 3. Multi-Format Content Generation

```typescript
import { generateMultiFormat } from '@/lib/generators/multi-format';

const contentFormats = await generateMultiFormat({
  topic: 'Marketing Automation Trends 2024',
  formats: ['toplist', 'pov', 'how-to', 'case-study'],
  languages: ['en', 'vi'],
  aiProvider: 'claude',
  researchData: researchData
});

// Returns content in all specified formats and languages
contentFormats.forEach(content => {
  console.log(`Format: ${content.format}, Language: ${content.language}`);
  console.log(content.title);
  console.log(content.body);
});
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';

// Render video from content
const videoOutput = await renderVideo({
  contentId: 'content-123',
  template: 'infographic',
  content: {
    title: 'Top 5 AI Marketing Tools',
    points: [
      'Tool 1: Description',
      'Tool 2: Description',
      // ...
    ],
    branding: {
      logo: '/logo.png',
      colors: {
        primary: '#FF6B6B',
        secondary: '#4ECDC4'
      }
    }
  },
  platform: 'reels', // or 'tiktok', 'shorts'
  duration: 30
});

console.log(`Video rendered: ${videoOutput.url}`);
```

## Common Workflows

### Complete Content Pipeline

```typescript
import { runFullPipeline } from '@/lib/pipeline';

async function createCompleteContent(keyword: string) {
  try {
    // Step 1: Research
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter', 'a16z'],
      timeframe: '24h'
    });

    // Step 2: Generate content in multiple formats
    const contents = await generateMultiFormat({
      topic: keyword,
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi'],
      aiProvider: 'claude',
      researchData: research
    });

    // Step 3: Generate videos for each content piece
    const videos = await Promise.all(
      contents.map(content => 
        renderVideo({
          contentId: content.id,
          template: getTemplateForFormat(content.format),
          content: content,
          platform: 'reels'
        })
      )
    );

    return {
      research,
      contents,
      videos
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function getTemplateForFormat(format: string): string {
  const templates = {
    'toplist': 'numbered-list',
    'how-to': 'step-by-step',
    'case-study': 'storytelling',
    'pov': 'opinion-piece'
  };
  return templates[format] || 'default';
}
```

### Scheduled Content Generation

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

// Schedule daily content generation
const schedule = await scheduleContentGeneration({
  keywords: ['AI marketing', 'content automation', 'social media trends'],
  frequency: 'daily',
  time: '09:00',
  formats: ['toplist', 'pov'],
  languages: ['en', 'vi'],
  autoPost: false, // Set to true to auto-post to platforms
  platforms: ['facebook', 'linkedin']
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language, aiProvider } = await request.json();

    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const content = await generateContent({
      client: anthropic,
      provider: aiProvider || 'claude',
      topic,
      format,
      language: language || 'en'
    });

    return NextResponse.json({ 
      success: true, 
      content 
    });
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
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();

    const researchData = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h'
    });

    return NextResponse.json({ 
      success: true, 
      data: researchData 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## TypeScript Types

```typescript
// types/content.ts
export interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  aiProvider: 'claude' | 'openai';
}

export interface GeneratedContent {
  id: string;
  title: string;
  body: string;
  format: string;
  language: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    keywords: string[];
  };
  createdAt: Date;
}

export interface ResearchData {
  articles: Article[];
  insights: string[];
  trending: string[];
  dataPoints: DataPoint[];
}

export interface VideoRenderOptions {
  contentId: string;
  template: string;
  content: any;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Start Remotion Studio (for video preview/editing)
npm run remotion:studio

# Render videos in production
npm run remotion:render
```

## Remotion Video Templates

```typescript
// remotion/compositions/Infographic.tsx
import { Composition } from 'remotion';
import { InfoGraphicTemplate } from './templates/InfoGraphic';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="Infographic"
        component={InfoGraphicTemplate}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // Reels/TikTok aspect ratio
        defaultProps={{
          title: 'Your Content Title',
          points: [],
          branding: {}
        }}
      />
    </>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI APIs
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
});

async function generateWithRateLimit(params) {
  const { success } = await ratelimit.limit(params.userId);
  
  if (!success) {
    throw new Error('Rate limit exceeded. Please try again later.');
  }
  
  return await generateContent(params);
}
```

### Error Handling

```typescript
// Robust error handling for content generation
async function safeContentGeneration(params) {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      return await generateContent(params);
    } catch (error) {
      attempt++;
      
      if (error.message.includes('rate_limit')) {
        await new Promise(resolve => setTimeout(resolve, 2000 * attempt));
        continue;
      }
      
      if (attempt === maxRetries) {
        throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
      }
    }
  }
}
```

### Video Rendering Issues

```typescript
// Handle Remotion rendering errors
import { renderMedia } from '@remotion/renderer';

async function safeVideoRender(options) {
  try {
    const output = await renderMedia(options);
    return output;
  } catch (error) {
    if (error.message.includes('memory')) {
      console.error('Insufficient memory for video rendering');
      // Reduce quality or duration
      return await renderMedia({
        ...options,
        quality: 50 // Lower quality
      });
    }
    throw error;
  }
}
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache research results** to avoid redundant API calls
3. **Use environment variables** for all API keys and secrets
4. **Implement proper error boundaries** in React components
5. **Monitor AI token usage** to control costs
6. **Batch video rendering** for efficiency
7. **Store generated content** in a database for reuse and analytics
