---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, auto-posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with marketing pipeline
  - set up AI content automation with research and video generation
  - create automated content workflow from research to video
  - use marketing pipeline for auto content generation
  - integrate Claude and OpenAI for content automation
  - automate social media content with AI pipeline
  - generate videos automatically from content scripts
  - build content automation pipeline with Remotion
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline leverages Claude 3, OpenAI, and Remotion to crawl news sources, generate multi-format content, and render videos automatically.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-scans and researches** trending topics from sources like TechCrunch, a16z, X (Twitter), and LinkedIn
- **Generates diverse content formats** (top lists, POV articles, case studies, how-tos) in multiple languages using Claude/OpenAI
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multi-platform** distribution (Reels, TikTok, Shorts)

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
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research & Data Collection
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Architecture Components

### 1. Research Module (Auto-Scan)

The research module crawls and analyzes content from various sources:

```typescript
// src/services/research.service.ts
import { Anthropic } from '@anthropic-ai/sdk';

export class ResearchService {
  private anthropic: Anthropic;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async scanTrendingTopics(keyword: string, sources: string[] = []) {
    const crawledData = await this.crawlSources(sources);
    
    const analysis = await this.anthropic.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: `Analyze these articles about "${keyword}" and extract key insights, trends, and data points:\n\n${crawledData}`,
        },
      ],
    });

    return this.parseInsights(analysis.content);
  }

  private async crawlSources(sources: string[]) {
    // Implementation for crawling TechCrunch, a16z, Twitter, LinkedIn
    const results = await Promise.all(
      sources.map(source => this.fetchFromSource(source))
    );
    return results.join('\n\n');
  }

  private async fetchFromSource(source: string) {
    // RapidAPI integration for fetching real-time data
    const response = await fetch(`https://api.rapidapi.com/${source}`, {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      },
    });
    return response.json();
  }

  private parseInsights(content: any) {
    // Extract structured insights from Claude's response
    return {
      trends: [],
      dataPoints: [],
      keyInsights: [],
    };
  }
}
```

### 2. Content Generation Module

Generate content in multiple formats and languages:

```typescript
// src/services/content-generator.service.ts
import OpenAI from 'openai';
import { Anthropic } from '@anthropic-ai/sdk';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type ToneOfVoice = 'expert' | 'friendly' | 'humorous';

export class ContentGeneratorService {
  private openai: OpenAI;
  private anthropic: Anthropic;

  constructor() {
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async generateContent(params: {
    keyword: string;
    research: any;
    format: ContentFormat;
    language: Language;
    tone: ToneOfVoice;
    provider?: 'openai' | 'claude';
  }) {
    const prompt = this.buildPrompt(params);

    if (params.provider === 'openai') {
      return this.generateWithOpenAI(prompt, params);
    }
    
    return this.generateWithClaude(prompt, params);
  }

  private buildPrompt(params: {
    keyword: string;
    research: any;
    format: ContentFormat;
    language: Language;
    tone: ToneOfVoice;
  }) {
    const formatInstructions = {
      'toplist': 'Create a top 10 list article with detailed explanations',
      'pov': 'Write a thought-leadership piece expressing a unique perspective',
      'case-study': 'Develop an in-depth case study with data and examples',
      'how-to': 'Create a step-by-step tutorial guide',
    };

    const toneInstructions = {
      'expert': 'Use professional, authoritative language',
      'friendly': 'Use conversational, approachable language',
      'humorous': 'Use light, engaging humor while staying informative',
    };

    return `
You are writing a ${params.format} article about "${params.keyword}" in ${params.language === 'en' ? 'English' : 'Vietnamese'}.

${formatInstructions[params.format]}
${toneInstructions[params.tone]}

Research data:
${JSON.stringify(params.research, null, 2)}

Create engaging, SEO-optimized content that:
- Incorporates the latest insights from the research
- Includes data points and statistics
- Has a compelling headline and structure
- Is optimized for social media sharing
`;
  }

  private async generateWithOpenAI(prompt: string, params: any) {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator and marketer.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      temperature: 0.7,
      max_tokens: 2000,
    });

    return this.parseGeneratedContent(completion.choices[0].message.content);
  }

  private async generateWithClaude(prompt: string, params: any) {
    const message = await this.anthropic.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return this.parseGeneratedContent(message.content);
  }

  private parseGeneratedContent(content: any) {
    return {
      title: '',
      body: content,
      metadata: {
        generatedAt: new Date(),
      },
    };
  }
}
```

### 3. Video Generation Module (Remotion)

Automatically render videos from content:

```typescript
// src/services/video-generator.service.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export class VideoGeneratorService {
  async generateVideo(params: {
    content: any;
    platform: 'reels' | 'tiktok' | 'shorts';
  }) {
    const composition = this.getCompositionConfig(params.platform);
    
    // Bundle Remotion composition
    const bundled = await bundle(
      path.join(process.cwd(), 'src/remotion/index.tsx')
    );

    // Select composition
    const comp = await selectComposition({
      serveUrl: bundled,
      id: composition.id,
      inputProps: {
        content: params.content,
      },
    });

    // Render video
    const outputLocation = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}.mp4`
    );

    await renderMedia({
      composition: comp,
      serveUrl: bundled,
      codec: 'h264',
      outputLocation,
      inputProps: {
        content: params.content,
      },
    });

    return {
      videoUrl: outputLocation,
      duration: comp.durationInFrames / comp.fps,
    };
  }

  private getCompositionConfig(platform: string) {
    const configs = {
      reels: { id: 'Reels', width: 1080, height: 1920, fps: 30 },
      tiktok: { id: 'TikTok', width: 1080, height: 1920, fps: 30 },
      shorts: { id: 'Shorts', width: 1080, height: 1920, fps: 30 },
    };
    return configs[platform as keyof typeof configs];
  }
}
```

### 4. Remotion Composition Example

```typescript
// src/remotion/Composition.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

export const ContentVideoComposition: React.FC<{
  content: {
    title: string;
    body: string;
    dataPoints: string[];
  };
}> = ({ content }) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Arial, sans-serif',
      }}
    >
      <div style={{ opacity: titleOpacity, textAlign: 'center', padding: 40 }}>
        <h1 style={{ color: '#fff', fontSize: 72, marginBottom: 40 }}>
          {content.title}
        </h1>
        <div style={{ color: '#eee', fontSize: 32, lineHeight: 1.6 }}>
          {content.dataPoints.map((point, index) => (
            <p key={index} style={{ marginBottom: 20 }}>
              • {point}
            </p>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Integration

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/services/research.service';
import { ContentGeneratorService } from '@/services/content-generator.service';
import { VideoGeneratorService } from '@/services/video-generator.service';

export async function POST(request: NextRequest) {
  const body = await request.json();
  const { keyword, format, language, tone, platform } = body;

  try {
    // Step 1: Research
    const researchService = new ResearchService();
    const research = await researchService.scanTrendingTopics(keyword, [
      'techcrunch',
      'a16z',
      'twitter',
      'linkedin',
    ]);

    // Step 2: Generate Content
    const contentService = new ContentGeneratorService();
    const content = await contentService.generateContent({
      keyword,
      research,
      format,
      language,
      tone,
      provider: 'claude',
    });

    // Step 3: Generate Video
    const videoService = new VideoGeneratorService();
    const video = await videoService.generateVideo({
      content,
      platform,
    });

    return NextResponse.json({
      success: true,
      data: {
        content,
        video,
      },
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Development
npm run dev          # Start Next.js development server
npm run build        # Build for production
npm run start        # Start production server

# Remotion
npm run remotion     # Open Remotion Studio for video preview
npm run render       # Render videos from command line
```

## Common Usage Patterns

### Basic Content Pipeline

```typescript
import { ResearchService } from './services/research.service';
import { ContentGeneratorService } from './services/content-generator.service';

async function createContent(keyword: string) {
  const research = new ResearchService();
  const generator = new ContentGeneratorService();

  const insights = await research.scanTrendingTopics(keyword);
  
  const article = await generator.generateContent({
    keyword,
    research: insights,
    format: 'how-to',
    language: 'en',
    tone: 'expert',
  });

  return article;
}
```

### Batch Video Generation

```typescript
async function generateMultiPlatformVideos(content: any) {
  const videoService = new VideoGeneratorService();
  
  const platforms = ['reels', 'tiktok', 'shorts'] as const;
  
  const videos = await Promise.all(
    platforms.map(platform =>
      videoService.generateVideo({ content, platform })
    )
  );

  return videos;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Add retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Remotion Rendering Issues

Ensure you have proper codecs installed:

```bash
# Install FFmpeg (required for Remotion)
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows (use Chocolatey)
choco install ffmpeg
```

### Memory Issues with Large Content

```typescript
// Implement streaming for large content generation
async function generateLargeContent(params: any) {
  const stream = await openai.chat.completions.create({
    ...params,
    stream: true,
  });

  let content = '';
  for await (const chunk of stream) {
    content += chunk.choices[0]?.delta?.content || '';
  }

  return content;
}
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use environment-specific configs** for different AI providers
3. **Implement job queues** for video rendering (CPU-intensive)
4. **Monitor API usage** to stay within budget
5. **Version control your prompts** for reproducibility
