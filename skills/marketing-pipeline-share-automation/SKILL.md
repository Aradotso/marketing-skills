---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an automated marketing content pipeline
  - generate videos from blog posts automatically
  - research and create content with Claude and OpenAI
  - build an AI content workflow from research to video
  - automate social media content generation
  - create AI-powered content pipeline with Remotion
  - set up automated content research and writing
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for video rendering to create a complete content automation workflow.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-scans and researches** news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
- **Supports multiple languages** (English and Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Setup Steps

1. Clone the repository:

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
```

2. Install dependencies:

```bash
npm install
# or
yarn install
```

3. Create environment file:

```bash
cp .env.example .env
```

4. Configure required environment variables:

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Content Generation
DEFAULT_LANGUAGE=en
SUPPORTED_LANGUAGES=en,vi

# Video Rendering
REMOTION_LAMBDA_REGION=us-east-1
```

5. Run the development server:

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key API & Usage Patterns

### 1. Research Module

Automatically scrape and analyze content from news sources:

```typescript
import { ResearchService } from '@/lib/research/research-service';

interface ResearchParams {
  keyword: string;
  sources?: string[];
  timeframe?: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

async function performResearch(params: ResearchParams) {
  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });

  const results = await researchService.scan({
    keyword: params.keyword,
    sources: params.sources || ['techcrunch', 'a16z', 'twitter'],
    timeframe: params.timeframe || '24h',
  });

  // Extract insights
  const insights = await researchService.extractInsights(results);
  
  return {
    rawData: results,
    insights: insights,
    sources: results.map(r => r.source),
  };
}

// Usage
const research = await performResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter'],
  timeframe: '24h',
});
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/content/generator';
import { ContentFormat, ContentTone } from '@/types/content';

interface GenerateContentParams {
  research: any;
  format: ContentFormat;
  tone: ContentTone;
  language: 'en' | 'vi';
  aiProvider: 'claude' | 'openai';
}

async function generateContent(params: GenerateContentParams) {
  const generator = new ContentGenerator({
    claudeApiKey: process.env.ANTHROPIC_API_KEY,
    openaiApiKey: process.env.OPENAI_API_KEY,
  });

  const content = await generator.create({
    researchData: params.research,
    format: params.format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    tone: params.tone, // 'expert' | 'friendly' | 'humorous'
    language: params.language,
    provider: params.aiProvider,
  });

  return content;
}

// Usage
const article = await generateContent({
  research: research.insights,
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  aiProvider: 'claude',
});
```

### 3. Claude Integration

Working directly with Claude API:

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: `You are an expert content creator specializing in marketing content. 
             Use the following research context: ${context}`,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Usage for content outline
const outline = await generateWithClaude(
  'Create a detailed outline for a top 10 list about AI automation tools',
  JSON.stringify(research.insights)
);
```

### 4. OpenAI Integration

Using OpenAI for content generation:

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  prompt: string, 
  systemContext: string,
  model: string = 'gpt-4-turbo-preview'
) {
  const completion = await openai.chat.completions.create({
    model: model,
    messages: [
      {
        role: 'system',
        content: systemContext,
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}

// Usage for multilingual content
const vietnameseVersion = await generateWithOpenAI(
  'Translate and adapt this content for Vietnamese audience: ' + article,
  'You are a professional translator and content adapter specializing in marketing content.'
);
```

### 5. Video Generation with Remotion

Render videos from generated content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  contentData: any;
  template: 'infographic' | 'social-short' | 'explainer';
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig) {
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.template,
    inputProps: {
      content: config.contentData,
      platform: config.platform,
    },
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
    inputProps: {
      content: config.contentData,
      platform: config.platform,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  contentData: article,
  template: 'infographic',
  platform: 'reels',
});
```

### 6. Complete Pipeline Orchestration

Run the entire pipeline from research to video:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

interface PipelineConfig {
  keyword: string;
  contentFormats: ContentFormat[];
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  platforms?: ('reels' | 'tiktok' | 'shorts')[];
}

async function runCompletePipeline(config: PipelineConfig) {
  const pipeline = new ContentPipeline({
    claudeApiKey: process.env.ANTHROPIC_API_KEY,
    openaiApiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });

  // Execute full pipeline
  const result = await pipeline.execute({
    keyword: config.keyword,
    contentFormats: config.contentFormats,
    languages: config.languages,
    generateVideo: config.generateVideo,
    videoPlatforms: config.platforms,
  });

  return {
    research: result.research,
    content: result.generatedContent,
    videos: result.videos,
    metadata: result.metadata,
  };
}

// Usage - Complete automation
const output = await runCompletePipeline({
  keyword: 'AI marketing tools 2024',
  contentFormats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['reels', 'tiktok'],
});
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research/research-service';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, sources, timeframe } = body;

    const researchService = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY,
    });

    const results = await researchService.scan({
      keyword,
      sources,
      timeframe,
    });

    return NextResponse.json({
      success: true,
      data: results,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { research, format, tone, language, provider } = body;

    const generator = new ContentGenerator({
      claudeApiKey: process.env.ANTHROPIC_API_KEY,
      openaiApiKey: process.env.OPENAI_API_KEY,
    });

    const content = await generator.create({
      researchData: research,
      format,
      tone,
      language,
      provider,
    });

    return NextResponse.json({
      success: true,
      content,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { contentData, template, platform } = body;

    const videoPath = await generateVideo({
      contentData,
      template,
      platform,
    });

    return NextResponse.json({
      success: true,
      videoUrl: videoPath,
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
export type ContentFormat = 
  | 'toplist' 
  | 'pov' 
  | 'case-study' 
  | 'how-to'
  | 'comparison'
  | 'news-analysis';

export type ContentTone = 
  | 'expert' 
  | 'friendly' 
  | 'humorous'
  | 'professional'
  | 'casual';

export type Language = 'en' | 'vi';

export type VideoPlatform = 'reels' | 'tiktok' | 'shorts' | 'youtube';

export interface ContentMetadata {
  title: string;
  description: string;
  keywords: string[];
  format: ContentFormat;
  tone: ContentTone;
  language: Language;
  wordCount: number;
  readingTime: number;
  createdAt: Date;
}

export interface GeneratedContent {
  content: string;
  metadata: ContentMetadata;
  htmlContent?: string;
  markdown?: string;
  seoData?: {
    metaTitle: string;
    metaDescription: string;
    ogImage?: string;
  };
}

export interface ResearchResult {
  sources: Array<{
    url: string;
    title: string;
    snippet: string;
    publishedAt: Date;
    source: string;
  }>;
  insights: string[];
  trends: string[];
  statistics: Array<{
    metric: string;
    value: string | number;
    source: string;
  }>;
}
```

## Configuration

### Content Generation Config

```typescript
// config/content.config.ts
export const contentConfig = {
  formats: {
    toplist: {
      minItems: 5,
      maxItems: 15,
      includeIntro: true,
      includeConclusion: true,
    },
    'case-study': {
      sections: ['background', 'challenge', 'solution', 'results'],
      includeMetrics: true,
    },
    'how-to': {
      includeSteps: true,
      includeTips: true,
      includeImages: true,
    },
  },
  tones: {
    expert: {
      vocabulary: 'advanced',
      includeData: true,
      formalLanguage: true,
    },
    friendly: {
      vocabulary: 'accessible',
      conversational: true,
      includeExamples: true,
    },
  },
  aiProviders: {
    claude: {
      model: 'claude-3-5-sonnet-20241022',
      maxTokens: 4096,
      temperature: 0.7,
    },
    openai: {
      model: 'gpt-4-turbo-preview',
      maxTokens: 3000,
      temperature: 0.7,
    },
  },
};
```

### Video Rendering Config

```typescript
// remotion/config.ts
export const videoConfig = {
  platforms: {
    reels: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationInFrames: 900, // 30 seconds
    },
    tiktok: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationInFrames: 900,
    },
    shorts: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationInFrames: 1800, // 60 seconds
    },
  },
  templates: {
    infographic: {
      scenes: ['intro', 'main-points', 'conclusion'],
      transitions: true,
      music: true,
    },
  },
};
```

## Common Patterns

### Pattern 1: Research → Generate → Publish

```typescript
async function researchToPublish(keyword: string) {
  // Step 1: Research
  const research = await performResearch({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h',
  });

  // Step 2: Generate content
  const content = await generateContent({
    research: research.insights,
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    aiProvider: 'claude',
  });

  // Step 3: Generate video
  const video = await generateVideo({
    contentData: content,
    template: 'infographic',
    platform: 'reels',
  });

  // Step 4: Return publish-ready assets
  return {
    article: content,
    videoUrl: video,
    metadata: {
      keyword,
      createdAt: new Date(),
    },
  };
}
```

### Pattern 2: Multilingual Content Generation

```typescript
async function generateMultilingualContent(keyword: string) {
  const research = await performResearch({ keyword, timeframe: '24h' });
  
  const languages: Language[] = ['en', 'vi'];
  const contents = await Promise.all(
    languages.map(async (language) => {
      const content = await generateContent({
        research: research.insights,
        format: 'how-to',
        tone: 'friendly',
        language,
        aiProvider: language === 'en' ? 'claude' : 'openai',
      });

      return {
        language,
        content,
      };
    })
  );

  return contents;
}
```

### Pattern 3: Batch Video Generation

```typescript
async function batchGenerateVideos(
  content: GeneratedContent,
  platforms: VideoPlatform[]
) {
  const videos = await Promise.all(
    platforms.map(async (platform) => {
      const videoPath = await generateVideo({
        contentData: content,
        template: 'social-short',
        platform,
      });

      return {
        platform,
        path: videoPath,
        dimensions: videoConfig.platforms[platform],
      };
    })
  );

  return videos;
}
```

### Pattern 4: Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

function setupScheduledPipeline(keywords: string[]) {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    for (const keyword of keywords) {
      try {
        const result = await runCompletePipeline({
          keyword,
          contentFormats: ['toplist', 'how-to'],
          languages: ['en'],
          generateVideo: true,
          platforms: ['reels', 'tiktok'],
        });

        console.log(`Pipeline completed for: ${keyword}`);
        // Store or publish result
      } catch (error) {
        console.error(`Pipeline failed for ${keyword}:`, error);
      }
    }
  });
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys before running pipeline
function validateApiKeys() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required API keys: ${missing.join(', ')}\n` +
      `Please set them in your .env file`
    );
  }
}

// Use before pipeline execution
validateApiKeys();
```

### Rate Limiting

```typescript
// Implement retry logic with exponential backoff
async function apiCallWithRetry<T>(
  apiCall: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await apiCall();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await apiCallWithRetry(() =>
  generateWithClaude(prompt, context)
);
```

### Memory Issues with Video Rendering

```typescript
// Process videos sequentially to avoid memory issues
async function renderVideosSequentially(
  contents: GeneratedContent[],
  platform: VideoPlatform
) {
  const results = [];
  
  for (const content of contents) {
    const video = await generateVideo({
      contentData: content,
      template: 'infographic',
      platform,
    });
    
    results.push(video);
    
    // Force garbage collection if available
    if (global.gc) {
      global.gc();
    }
  }
  
  return results;
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearchQuality(research: ResearchResult) {
  // Check minimum data requirements
  if (research.sources.length < 3) {
    throw new Error('Insufficient research sources. Need at least 3 sources.');
  }

  if (research.insights.length === 0) {
    throw new Error('No insights extracted from research data.');
  }

  // Filter out low-quality sources
  const qualitySources = research.sources.filter(source => 
    source.snippet.length > 100 &&
    source.title.length > 10
  );

  return {
    ...research,
    sources: qualitySources,
  };
}
```

### Error Handling Wrapper

```typescript
// Comprehensive error handling for pipeline
async function safePipelineExecution(config: PipelineConfig) {
  try {
    validateApiKeys();
    
    const result = await runCompletePipeline(config);
    
    return {
      success: true,
      data: result,
    };
  } catch (error) {
    console.error('Pipeline execution failed:', error);
    
    // Log to monitoring service
    if (error.status === 429) {
      return {
        success: false,
        error: 'Rate limit exceeded. Please try again later.',
        retryable: true,
      };
    }
    
    if (error.code === 'INSUFFICIENT_QUOTA') {
      return {
        success: false,
        error: 'API quota exceeded. Please check your billing.',
        retryable: false,
      };
    }
    
    return {
      success: false,
      error: error.message || 'Unknown error occurred',
      retryable: true,
    };
  }
}
```

This skill provides comprehensive coverage for working with the Marketing Pipeline Share automation system, enabling AI coding agents to effectively assist developers in implementing end-to-end content automation workflows.
