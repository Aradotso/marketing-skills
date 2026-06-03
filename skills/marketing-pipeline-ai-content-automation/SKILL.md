---
name: marketing-pipeline-ai-content-automation
description: AI-powered content pipeline that automates research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI
  - generate marketing content from research to video
  - build an AI content pipeline
  - create automated video content with Remotion
  - scrape news and generate content automatically
  - set up AI marketing automation workflow
  - generate multilingual content with Claude
  - automate social media content production
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation. The system leverages Claude 3, OpenAI, and Remotion to create multilingual content (English/Vietnamese) across multiple formats.

## What This Project Does

The Marketing Pipeline automates:

1. **Auto-Research**: Crawls and analyzes recent content from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
3. **Multilingual Output**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
5. **Multi-Platform Optimization**: Exports video in formats optimized for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research crawlers
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript types
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
import { ResearchCrawler } from '@/lib/crawler/research-crawler';

// Initialize crawler
const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  apiKey: process.env.RAPIDAPI_KEY
});

// Crawl content by keyword
async function crawlResearch(keyword: string) {
  const results = await crawler.scan({
    keyword,
    limit: 20,
    includeMetrics: true
  });

  return {
    articles: results.articles,
    insights: results.extractedInsights,
    trends: results.trendingTopics
  };
}

// Example usage
const research = await crawlResearch('AI marketing automation');
console.log(research.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationParams {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(params: ContentGenerationParams) {
  const prompt = buildContentPrompt(params);
  
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

  return {
    content: message.content[0].text,
    usage: message.usage
  };
}

function buildContentPrompt(params: ContentGenerationParams): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with compelling headlines',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze with data-backed examples and outcomes',
    'how-to': 'Provide step-by-step actionable instructions'
  };

  return `
You are an expert content creator. Generate a ${params.format} article about "${params.keyword}".

Language: ${params.language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${params.tone}
Format: ${formatInstructions[params.format]}

Research Data:
${JSON.stringify(params.researchData, null, 2)}

Requirements:
- Use recent data and statistics from the research
- Include compelling headlines and subheadings
- Optimize for SEO and engagement
- Length: 1500-2000 words
- Include actionable takeaways
`;
}
```

### 3. OpenAI Alternative Implementation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(params: ContentGenerationParams) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content creator.'
      },
      {
        role: 'user',
        content: buildContentPrompt(params)
      }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });

  return {
    content: completion.choices[0].message.content,
    usage: completion.usage
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  points: string[];
  style: 'infographic' | 'slideshow' | 'animated';
  duration: number; // in seconds
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[config.platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.style,
    inputProps: {
      title: config.title,
      points: config.points,
      durationInSeconds: config.duration
    },
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'output',
    `${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      points: config.points,
    },
  });

  return outputPath;
}

// Example usage
const videoPath = await generateVideo({
  title: '5 AI Marketing Trends',
  points: [
    'Automated content creation',
    'Predictive analytics',
    'Personalization at scale',
    'Chatbot integration',
    'Voice search optimization'
  ],
  style: 'infographic',
  duration: 30,
  platform: 'reels'
});
```

## Complete Workflow Example

```typescript
import { ContentPipeline } from '@/lib/content-pipeline';

// Initialize the complete pipeline
const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  outputFormats: ['article', 'video'],
  languages: ['en', 'vi']
});

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research
    });
    
    // Step 3: Translate (if needed)
    console.log('🌐 Translating...');
    const translatedContent = await pipeline.translate(content, 'vi');
    
    // Step 4: Generate video
    console.log('🎬 Creating video...');
    const video = await pipeline.generateVideo({
      title: content.title,
      points: content.keyPoints,
      style: 'infographic',
      duration: 30,
      platform: 'reels'
    });
    
    return {
      englishArticle: content,
      vietnameseArticle: translatedContent,
      videoPath: video.path,
      metadata: {
        generatedAt: new Date(),
        sources: research.sources,
        aiTokensUsed: content.usage.total_tokens
      }
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
const result = await runContentPipeline('AI content automation');
console.log('✅ Pipeline complete:', result);
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;

    const pipeline = new ContentPipeline({
      aiProvider: process.env.AI_PROVIDER || 'claude',
      outputFormats: ['article'],
      languages: [language]
    });

    const result = await pipeline.run({
      keyword,
      format,
      language,
      tone
    });

    return NextResponse.json({
      success: true,
      data: result
    });

  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// src/app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { title, points, style, platform } = body;

    const videoPath = await generateVideo({
      title,
      points,
      style,
      duration: 30,
      platform
    });

    return NextResponse.json({
      success: true,
      videoUrl: `/output/${path.basename(videoPath)}`
    });

  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}

// Generate content for multiple topics
const topics = [
  'AI marketing trends',
  'Social media automation',
  'Content creation tools'
];

const batchResults = await batchGenerateContent(topics);
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Schedule daily content generation at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('🕐 Running scheduled content generation...');
  
  const trendingTopics = await fetchTrendingTopics();
  const results = await batchGenerateContent(trendingTopics.slice(0, 3));
  
  console.log('✅ Scheduled generation complete:', results);
});
```

### Custom Video Templates

```typescript
// remotion/templates/CustomInfographic.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomInfographic: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ color: '#fff', fontSize: 72 }}>{title}</h1>
        <ul style={{ color: '#fff', fontSize: 48, marginTop: 40 }}>
          {points.map((point, index) => {
            const pointOpacity = interpolate(
              frame,
              [30 + index * 20, 50 + index * 20],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <li key={index} style={{ opacity: pointOpacity, marginBottom: 30 }}>
                {point}
              </li>
            );
          })}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Options

### Pipeline Configuration

```typescript
interface PipelineConfig {
  // AI Provider
  aiProvider: 'claude' | 'openai';
  claudeModel?: 'claude-3-5-sonnet-20241022' | 'claude-3-opus-20240229';
  openaiModel?: 'gpt-4-turbo-preview' | 'gpt-4';
  
  // Output settings
  outputFormats: Array<'article' | 'video' | 'infographic'>;
  languages: Array<'en' | 'vi'>;
  
  // Research settings
  researchSources: string[];
  researchTimeRange: '24h' | '7d' | '30d';
  
  // Video settings
  videoStyle: 'infographic' | 'slideshow' | 'animated';
  videoPlatforms: Array<'reels' | 'tiktok' | 'shorts'>;
  
  // Rate limiting
  maxConcurrentRequests: number;
  requestDelay: number;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateWithRateLimit(keywords: string[]) {
  const promises = keywords.map(keyword =>
    limit(() => runContentPipeline(keyword))
  );
  
  return Promise.all(promises);
}
```

### Error Handling

```typescript
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'video',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

async function safeRunPipeline(keyword: string) {
  try {
    return await runContentPipeline(keyword);
  } catch (error) {
    if (error.message.includes('rate_limit')) {
      console.log('⏳ Rate limited, waiting 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeRunPipeline(keyword);
    }
    throw new PipelineError(
      'Pipeline execution failed',
      'generation',
      error
    );
  }
}
```

### Memory Management for Large Batches

```typescript
async function* generateContentStream(keywords: string[]) {
  for (const keyword of keywords) {
    yield await runContentPipeline(keyword);
  }
}

// Use streaming for large batches
for await (const result of generateContentStream(largeKeywordList)) {
  console.log('Generated:', result.englishArticle.title);
  // Process or save immediately to avoid memory issues
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render video locally (Remotion)
npm run remotion:render

# Preview video composition
npm run remotion:preview

# Type checking
npm run type-check

# Linting
npm run lint
```

This skill enables AI agents to effectively implement, customize, and troubleshoot the Marketing Pipeline AI Content Automation system for automated content creation workflows.
