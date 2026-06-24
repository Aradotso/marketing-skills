---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - create automated marketing content from scratch
  - build AI content workflow with remotion
  - automate content research and video creation
  - generate multi-format content with Claude
  - create AI-powered content automation system
  - set up automated content creation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from researching trending topics across multiple sources (TechCrunch, a16z, Twitter, LinkedIn), to generating written content in multiple formats and languages using Claude/OpenAI, to automatically rendering videos and infographics with Remotion.

**Key Capabilities:**
- Auto-crawl and analyze real-time data from major news sources
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Support for both English and Vietnamese with customizable tone
- Automatic video and infographic rendering for social media
- Next.js-based UI for easy content management

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

Create a `.env.local` file in the root directory:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for content research
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database configuration
DATABASE_URL=your_database_url

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run Remotion preview (for video rendering)
npm run remotion:preview
```

## Core Architecture

### 1. Content Research Pipeline

The system automatically crawls and analyzes content from multiple sources:

```typescript
// lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchConfig {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeRange: '24h' | '7d' | '30d';
  maxResults: number;
}

export async function performResearch(config: ResearchConfig) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    config.sources.map(async (source) => {
      return await client.searchContent({
        source,
        keyword: config.keyword,
        timeRange: config.timeRange,
        limit: config.maxResults
      });
    })
  );
  
  return aggregateAndAnalyze(results);
}

async function aggregateAndAnalyze(results: any[]) {
  // Combine results from all sources
  const allContent = results.flat();
  
  // Extract insights, trends, and key data points
  return {
    insights: extractInsights(allContent),
    trends: identifyTrends(allContent),
    dataPoints: extractDataPoints(allContent),
    sources: allContent.map(item => ({
      title: item.title,
      url: item.url,
      publishedAt: item.publishedAt
    }))
  };
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationConfig {
  research: ResearchData;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience: string;
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generate(config: ContentGenerationConfig, provider: 'claude' | 'openai' = 'claude') {
    const prompt = this.buildPrompt(config);
    
    if (provider === 'claude') {
      return await this.generateWithClaude(prompt);
    } else {
      return await this.generateWithOpenAI(prompt);
    }
  }
  
  private buildPrompt(config: ContentGenerationConfig): string {
    const formatInstructions = this.getFormatInstructions(config.format);
    const toneGuidelines = this.getToneGuidelines(config.tone);
    
    return `
You are a professional content writer creating content in ${config.language}.

Target Audience: ${config.targetAudience}
Content Format: ${config.format}
Tone: ${config.tone}

Research Data:
${JSON.stringify(config.research, null, 2)}

${formatInstructions}
${toneGuidelines}

Please create comprehensive, engaging content that:
1. Incorporates the latest insights from the research
2. Uses data points to back up claims
3. Maintains the specified tone throughout
4. Follows the format structure precisely
5. Includes proper citations for sources
`;
  }
  
  private async generateWithClaude(prompt: string) {
    const response = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return {
      content: response.content[0].text,
      provider: 'claude',
      model: response.model,
      usage: response.usage
    };
  }
  
  private async generateWithOpenAI(prompt: string) {
    const response = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4000
    });
    
    return {
      content: response.choices[0].message.content,
      provider: 'openai',
      model: response.model,
      usage: response.usage
    };
  }
  
  private getFormatInstructions(format: ContentFormat): string {
    const instructions = {
      'toplist': 'Create a numbered list format with detailed explanations for each item.',
      'pov': 'Write from a specific perspective, sharing unique insights and opinions.',
      'case-study': 'Structure as: Problem → Solution → Results with specific metrics.',
      'how-to': 'Create step-by-step instructions with clear, actionable advice.'
    };
    return instructions[format];
  }
  
  private getToneGuidelines(tone: Tone): string {
    const guidelines = {
      'expert': 'Use professional language, industry terminology, and authoritative voice.',
      'friendly': 'Write conversationally, use "you" and "we", be approachable.',
      'humorous': 'Include wit and humor where appropriate, use analogies and light jokes.'
    };
    return guidelines[tone];
  }
}
```

### 3. Video Generation with Remotion

Automatically render videos from content:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
  brandColor: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  brandColor,
  format
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
        opacity: titleOpacity
      }}>
        <h1 style={{
          fontSize: 72,
          color: brandColor,
          textAlign: 'center',
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        {content.map((text, index) => {
          const startFrame = 60 + (index * 90);
          const opacity = interpolate(
            frame,
            [startFrame, startFrame + 30],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <p key={index} style={{
              fontSize: 48,
              color: '#fff',
              opacity,
              marginBottom: 30,
              textAlign: 'center'
            }}>
              {text}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderConfig {
  title: string;
  content: string[];
  format: 'reels' | 'tiktok' | 'shorts';
  brandColor?: string;
}

export async function renderContentVideo(config: VideoRenderConfig) {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Get composition details
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: config
  });
  
  // Render the video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.format}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: config
  });
  
  return {
    path: outputPath,
    url: `/videos/${path.basename(outputPath)}`
  };
}
```

### 4. Complete Pipeline Integration

```typescript
// lib/pipeline/content-pipeline.ts
import { performResearch } from '@/lib/research/crawler';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  includeVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export class ContentPipeline {
  private generator: ContentGenerator;
  
  constructor() {
    this.generator = new ContentGenerator();
  }
  
  async execute(config: PipelineConfig) {
    console.log('🔍 Starting research phase...');
    const research = await performResearch({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeRange: '24h',
      maxResults: 10
    });
    
    console.log('✍️ Generating content...');
    const content = await this.generator.generate({
      research,
      format: config.format,
      language: config.language,
      tone: config.tone,
      targetAudience: 'Marketing professionals and content creators'
    });
    
    let video = null;
    if (config.includeVideo && config.videoFormat) {
      console.log('🎬 Rendering video...');
      video = await renderContentVideo({
        title: content.title,
        content: this.extractKeyPoints(content.content),
        format: config.videoFormat,
        brandColor: '#3b82f6'
      });
    }
    
    console.log('✅ Pipeline complete!');
    return {
      research,
      content,
      video,
      metadata: {
        keyword: config.keyword,
        generatedAt: new Date().toISOString(),
        provider: content.provider,
        usage: content.usage
      }
    };
  }
  
  private extractKeyPoints(content: string): string[] {
    // Extract 5-7 key points from the content for video
    const sentences = content.split(/[.!?]+/).filter(s => s.trim().length > 0);
    return sentences.slice(0, 7).map(s => s.trim());
  }
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: Request) {
  try {
    const config = await request.json();
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(config);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Common Usage Patterns

### Basic Content Generation

```typescript
const pipeline = new ContentPipeline();

const result = await pipeline.execute({
  keyword: 'AI marketing automation',
  format: 'how-to',
  language: 'en',
  tone: 'expert',
  includeVideo: false
});

console.log(result.content.content);
```

### Full Pipeline with Video

```typescript
const result = await pipeline.execute({
  keyword: 'social media trends 2024',
  format: 'toplist',
  language: 'vi',
  tone: 'friendly',
  includeVideo: true,
  videoFormat: 'reels'
});

console.log('Content:', result.content);
console.log('Video URL:', result.video.url);
```

### Bilingual Content Generation

```typescript
const enContent = await generator.generate({
  research: researchData,
  format: 'pov',
  language: 'en',
  tone: 'expert',
  targetAudience: 'Marketing professionals'
});

const viContent = await generator.generate({
  research: researchData,
  format: 'pov',
  language: 'vi',
  tone: 'expert',
  targetAudience: 'Marketing professionals'
});
```

## Troubleshooting

### API Key Issues

```typescript
// Verify environment variables are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is not set');
}

if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY is not set');
}
```

### Rate Limiting

```typescript
// Implement retry logic for API calls
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (maxRetries === 0) throw error;
    
    if (error.status === 429) {
      await new Promise(resolve => setTimeout(resolve, delay));
      return withRetry(fn, maxRetries - 1, delay * 2);
    }
    throw error;
  }
}
```

### Video Rendering Issues

```bash
# Ensure ffmpeg is installed
brew install ffmpeg  # macOS
apt-get install ffmpeg  # Ubuntu/Debian

# Check Remotion installation
npm list @remotion/bundler @remotion/renderer
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
async function generateLargeContent(config: ContentGenerationConfig) {
  const chunks = splitResearchIntoChunks(config.research, 5);
  
  const results = [];
  for (const chunk of chunks) {
    const result = await generator.generate({
      ...config,
      research: chunk
    });
    results.push(result);
  }
  
  return combineResults(results);
}
```

## Best Practices

1. **Cache Research Results**: Store research data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue for video generation to avoid blocking
3. **Validate Content**: Implement content validation before publishing
4. **Monitor API Usage**: Track usage across Claude/OpenAI to manage costs
5. **Version Content**: Keep versions of generated content for A/B testing
