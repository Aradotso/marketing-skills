---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate blog posts and videos from keywords automatically
  - use Claude and OpenAI for content research and generation
  - build AI content automation workflow
  - create automatic video content from text with Remotion
  - integrate content research crawling into marketing pipeline
  - automate social media content generation and posting
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, scriptwriting, content generation, and video rendering. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate multi-format content (blog posts, case studies, how-tos) in multiple languages, and finally renders videos using Remotion.

## What This Project Does

- **Auto-Research**: Crawls recent news and data from major tech/marketing sources
- **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
- **Multi-language**: Generates content in English and Vietnamese with customizable tone
- **Video Generation**: Automatically renders infographics and short-form videos using Remotion
- **Platform Optimization**: Exports video in formats suitable for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Required Node.js version
node >= 18.x
npm or pnpm
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Content Sources (optional custom endpoints)
TECHCRUNCH_API_URL=https://api.techcrunch.com
TWITTER_API_KEY=your_twitter_api_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video generation)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # Content scraping modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
// src/lib/crawlers/research.ts
import { crawlTechCrunch, crawlTwitter, crawlLinkedIn } from './sources';

interface ResearchResult {
  title: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export async function performResearch(keyword: string): Promise<ResearchResult[]> {
  const [techCrunchData, twitterData, linkedInData] = await Promise.all([
    crawlTechCrunch(keyword, { timeframe: '24h' }),
    crawlTwitter(keyword, { maxResults: 50 }),
    crawlLinkedIn(keyword, { industry: 'tech' })
  ]);

  return aggregateAndAnalyze([
    ...techCrunchData,
    ...twitterData,
    ...linkedInData
  ]);
}

// Usage in API route
export async function POST(req: Request) {
  const { keyword } = await req.json();
  const researchData = await performResearch(keyword);
  
  return Response.json({ 
    success: true, 
    data: researchData 
  });
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(
  keyword: string, 
  options: ContentGenerationOptions
): Promise<string> {
  const systemPrompt = buildSystemPrompt(options);
  const userPrompt = buildUserPrompt(keyword, options.researchData);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(options: ContentGenerationOptions): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list with detailed explanations',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Structure as problem-solution-results with data',
    'how-to': 'Step-by-step instructional format with examples'
  };

  const toneInstructions = {
    'expert': 'Use technical terminology and authoritative voice',
    'friendly': 'Conversational and approachable tone',
    'humorous': 'Light, engaging with appropriate humor'
  };

  return `You are a professional content creator. 
Format: ${formatInstructions[options.format]}
Tone: ${toneInstructions[options.tone]}
Language: ${options.language === 'vi' ? 'Vietnamese' : 'English'}
Use data and insights from the provided research.`;
}
```

### 3. OpenAI Content Generation Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithGPT(
  keyword: string,
  options: ContentGenerationOptions
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(options)
      },
      {
        role: 'user',
        content: buildUserPrompt(keyword, options.researchData)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Full Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { performResearch } from '../crawlers/research';
import { generateContent } from '../ai/claude-generator';
import { renderVideo } from '../video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  tone: 'expert' | 'friendly' | 'humorous';
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('Starting research phase...');
  const researchData = await performResearch(config.keyword);

  // Step 2: Generate content for each language
  console.log('Generating content...');
  const contentResults = await Promise.all(
    config.languages.map(async (language) => {
      const content = await generateContent(config.keyword, {
        format: config.contentFormat,
        language,
        tone: config.tone,
        researchData
      });

      return { language, content };
    })
  );

  // Step 3: Generate video if requested
  let videoUrls: string[] = [];
  if (config.generateVideo) {
    console.log('Rendering videos...');
    videoUrls = await Promise.all(
      contentResults.map(async ({ language, content }) => {
        return await renderVideo({
          content,
          language,
          format: 'vertical', // for Reels/TikTok/Shorts
          duration: 60
        });
      })
    );
  }

  return {
    research: researchData,
    content: contentResults,
    videos: videoUrls
  };
}
```

### 5. Remotion Video Generation

```typescript
// src/lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  content: string;
  language: 'en' | 'vi';
  format: 'vertical' | 'horizontal' | 'square';
  duration: number;
}

export async function renderVideo(options: VideoRenderOptions): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts'),
    () => undefined,
    {
      webpackOverride: (config) => config,
    }
  );

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: options.content,
      language: options.language,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${options.language}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: options.content,
      language: options.language,
      aspectRatio: getAspectRatio(options.format),
    },
  });

  return outputLocation;
}

function getAspectRatio(format: string): { width: number; height: number } {
  const ratios = {
    vertical: { width: 1080, height: 1920 },   // 9:16 for Reels/TikTok
    horizontal: { width: 1920, height: 1080 }, // 16:9 for YouTube
    square: { width: 1080, height: 1080 },     // 1:1 for Instagram
  };
  return ratios[format] || ratios.vertical;
}
```

### 6. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  language: 'en' | 'vi';
  aspectRatio: { width: number; height: number };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  content,
  language,
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const scale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

  // Parse content into segments
  const segments = parseContentIntoSegments(content);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          padding: '40px',
          maxWidth: '80%',
        }}
      >
        {segments.map((segment, idx) => (
          <AnimatedSegment
            key={idx}
            segment={segment}
            frame={frame}
            delay={idx * 90}
          />
        ))}
      </div>
    </AbsoluteFill>
  );
};

function parseContentIntoSegments(content: string): string[] {
  // Split content into digestible segments
  return content.split('\n\n').filter(s => s.trim().length > 0);
}

const AnimatedSegment: React.FC<{
  segment: string;
  frame: number;
  delay: number;
}> = ({ segment, frame, delay }) => {
  const opacity = interpolate(
    frame,
    [delay, delay + 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <p
      style={{
        fontSize: '32px',
        color: 'white',
        marginBottom: '20px',
        opacity,
        lineHeight: 1.5,
      }}
    >
      {segment}
    </p>
  );
};
```

### 7. API Route Example

```typescript
// src/app/api/generate/route.ts
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(req: Request) {
  try {
    const body = await req.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'how-to',
      languages: body.languages || ['en', 'vi'],
      generateVideo: body.generateVideo !== false,
      tone: body.tone || 'expert',
    });

    return Response.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return Response.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Project

### Development Server

```bash
npm run dev
# or
pnpm dev

# Access at http://localhost:3000
```

### Build for Production

```bash
npm run build
npm start
```

### Video Rendering (Remotion)

```bash
# Render a single video
npx remotion render ContentVideo out/video.mp4

# Start Remotion Studio for preview
npx remotion studio
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
// Generate multiple content pieces at once
const keywords = ['AI Marketing', 'Content Automation', 'Video Generation'];

const batchResults = await Promise.all(
  keywords.map(keyword => 
    runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      languages: ['en'],
      generateVideo: false,
      tone: 'expert'
    })
  )
);
```

### Pattern 2: Scheduled Content Generation

```typescript
// Use with cron or similar scheduler
import cron from 'node-cron';

// Generate content daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline({
      keyword: topic,
      contentFormat: 'pov',
      languages: ['en', 'vi'],
      generateVideo: true,
      tone: 'friendly'
    });
  }
});
```

### Pattern 3: Custom Content Templates

```typescript
// Define reusable content templates
const contentTemplates = {
  productLaunch: {
    format: 'case-study',
    tone: 'expert',
    sections: ['problem', 'solution', 'results', 'cta']
  },
  thoughtLeadership: {
    format: 'pov',
    tone: 'expert',
    sections: ['insight', 'analysis', 'prediction']
  },
  tutorial: {
    format: 'how-to',
    tone: 'friendly',
    sections: ['intro', 'steps', 'tips', 'conclusion']
  }
};

// Use template
const result = await generateContent(keyword, {
  ...contentTemplates.productLaunch,
  language: 'en',
  researchData
});
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting and retry logic
import pRetry from 'p-retry';

async function generateWithRetry(keyword: string, options: any) {
  return pRetry(
    () => generateContent(keyword, options),
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Issue: Video Rendering Memory Issues

```typescript
// Reduce memory usage by rendering in chunks
const MAX_CONCURRENT_RENDERS = 2;

async function renderVideosInBatches(videoConfigs: any[]) {
  const batches = chunk(videoConfigs, MAX_CONCURRENT_RENDERS);
  
  for (const batch of batches) {
    await Promise.all(batch.map(config => renderVideo(config)));
    // Allow garbage collection between batches
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}
```

### Issue: Content Quality Consistency

```typescript
// Add validation and quality checks
function validateGeneratedContent(content: string): boolean {
  const minLength = 500;
  const hasHeadings = /#{1,3}\s/.test(content);
  const hasData = /\d+%|\d+\s(users|customers|dollars)/.test(content);
  
  return (
    content.length >= minLength &&
    hasHeadings &&
    hasData
  );
}

// Regenerate if quality is insufficient
let content = await generateContent(keyword, options);
let attempts = 0;

while (!validateGeneratedContent(content) && attempts < 3) {
  console.log(`Content quality insufficient, regenerating... (attempt ${attempts + 1})`);
  content = await generateContent(keyword, options);
  attempts++;
}
```

### Issue: Research Data Freshness

```typescript
// Cache research data with TTL
import NodeCache from 'node-cache';

const researchCache = new NodeCache({ stdTTL: 3600 }); // 1 hour TTL

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  let data = researchCache.get(cacheKey);
  
  if (!data) {
    data = await performResearch(keyword);
    researchCache.set(cacheKey, data);
  }
  
  return data;
}
```

## Key Commands

```bash
# Development
npm run dev              # Start development server
npm run build           # Build for production
npm run start           # Start production server

# Remotion
npx remotion studio     # Open Remotion preview
npx remotion render     # Render video from CLI

# Testing
npm run test            # Run tests
npm run lint            # Lint code

# Database (if using)
npx prisma migrate dev  # Run database migrations
npx prisma studio       # Open database GUI
```

## Configuration Tips

1. **API Keys**: Always use environment variables, never commit keys
2. **Rate Limiting**: Configure appropriate rate limits for each AI provider
3. **Video Quality**: Adjust Remotion codec settings based on output platform
4. **Content Format**: Test different formats to find what resonates with your audience
5. **Caching**: Implement caching for research data to reduce API costs
