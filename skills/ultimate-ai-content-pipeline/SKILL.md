---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing with Claude/OpenAI
triggers:
  - how do I use the AI content pipeline to generate articles
  - set up automated content research and video generation
  - create multi-language content with Claude and OpenAI
  - generate videos from text using Remotion in the content pipeline
  - configure the marketing content automation system
  - scrape news and generate social media content automatically
  - build an AI-powered content creation workflow
  - automate content from research to video publishing
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics across TechCrunch, Twitter, LinkedIn, to generating multi-format articles (listicles, case studies, how-tos) in multiple languages, to rendering videos with Remotion. It integrates Claude 3, OpenAI, and RapidAPI for a complete content factory.

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
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content scraping & analysis
│   │   ├── render/      # Remotion video generation
│   │   └── utils/       # Helpers
│   └── types/           # TypeScript definitions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Features

### 1. Automated Research & Scraping

**Scrape trending topics from multiple sources:**

```typescript
// src/lib/research/scraper.ts
import { RapidAPI } from '@/lib/api/rapid';

interface ResearchSource {
  platform: 'techcrunch' | 'twitter' | 'linkedin' | 'a16z';
  keyword: string;
  timeRange?: '24h' | '7d' | '30d';
}

export async function scrapeContent(sources: ResearchSource[]) {
  const results = await Promise.all(
    sources.map(async (source) => {
      const api = new RapidAPI(process.env.RAPIDAPI_KEY!);
      
      switch (source.platform) {
        case 'techcrunch':
          return api.fetchTechCrunchNews(source.keyword, source.timeRange);
        case 'twitter':
          return api.fetchTwitterTrends(source.keyword);
        case 'linkedin':
          return api.fetchLinkedInPosts(source.keyword);
        default:
          throw new Error(`Unsupported platform: ${source.platform}`);
      }
    })
  );
  
  return results.flat();
}

// Usage
const trendingData = await scrapeContent([
  { platform: 'techcrunch', keyword: 'AI automation', timeRange: '24h' },
  { platform: 'twitter', keyword: 'marketing AI' }
]);
```

### 2. AI Content Generation with Claude/OpenAI

**Generate multi-format articles:**

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  keyword: string;
  researchData: any[];
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
  }

  async generateWithClaude(config: ContentConfig) {
    const systemPrompt = this.buildSystemPrompt(config);
    const userPrompt = this.buildUserPrompt(config);

    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      system: systemPrompt,
      messages: [
        { role: 'user', content: userPrompt }
      ]
    });

    return this.parseContent(message.content[0].text);
  }

  async generateWithOpenAI(config: ContentConfig) {
    const systemPrompt = this.buildSystemPrompt(config);
    const userPrompt = this.buildUserPrompt(config);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt }
      ],
      temperature: 0.7
    });

    return this.parseContent(completion.choices[0].message.content!);
  }

  private buildSystemPrompt(config: ContentConfig): string {
    const toneMap = {
      expert: 'professional and authoritative',
      friendly: 'conversational and approachable',
      humorous: 'witty and entertaining'
    };

    return `You are an expert ${config.language === 'vi' ? 'Vietnamese' : 'English'} content writer.
Create a ${config.format} article with a ${toneMap[config.tone]} tone.
Use the provided research data to ensure accuracy and timeliness.`;
  }

  private buildUserPrompt(config: ContentConfig): string {
    return `Topic: ${config.keyword}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create a comprehensive ${config.format} article that:
1. Incorporates recent data and trends from the research
2. Provides actionable insights
3. Is optimized for engagement
4. Includes relevant statistics and examples

${config.format === 'toplist' ? 'Format as a numbered list with detailed explanations.' : ''}
${config.format === 'how-to' ? 'Provide step-by-step instructions with examples.' : ''}`;
  }

  private parseContent(text: string) {
    // Extract title, sections, key points
    return {
      title: text.match(/^#\s+(.+)$/m)?.[1] || '',
      content: text,
      sections: this.extractSections(text),
      metadata: {
        wordCount: text.split(/\s+/).length,
        readTime: Math.ceil(text.split(/\s+/).length / 200)
      }
    };
  }

  private extractSections(text: string) {
    const sections = [];
    const lines = text.split('\n');
    
    for (const line of lines) {
      if (line.startsWith('##')) {
        sections.push(line.replace(/^##\s+/, ''));
      }
    }
    
    return sections;
  }
}
```

**Usage example:**

```typescript
// src/app/api/generate/route.ts
import { ContentGenerator } from '@/lib/ai/content-generator';
import { scrapeContent } from '@/lib/research/scraper';

export async function POST(request: Request) {
  const { keyword, format, language, tone, provider } = await request.json();

  // Step 1: Research
  const researchData = await scrapeContent([
    { platform: 'techcrunch', keyword, timeRange: '24h' }
  ]);

  // Step 2: Generate Content
  const generator = new ContentGenerator();
  const content = provider === 'claude'
    ? await generator.generateWithClaude({
        format,
        language,
        tone,
        keyword,
        researchData
      })
    : await generator.generateWithOpenAI({
        format,
        language,
        tone,
        keyword,
        researchData
      });

  return Response.json(content);
}
```

### 3. Video Generation with Remotion

**Create videos from article content:**

```typescript
// src/lib/render/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  sections: string[];
  aspectRatio: '16:9' | '9:16' | '1:1';
  platform: 'youtube' | 'tiktok' | 'instagram';
}

export class VideoGenerator {
  async createVideoFromContent(config: VideoConfig) {
    const compositionId = this.getCompositionId(config.platform);
    
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config
    });

    // Select composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        title: config.title,
        sections: config.sections,
        aspectRatio: config.aspectRatio
      }
    });

    // Render video
    const outputLocation = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}-${config.platform}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: composition.defaultProps
    });

    return outputLocation;
  }

  private getCompositionId(platform: string): string {
    const compositionMap = {
      youtube: 'YouTubeShort',
      tiktok: 'TikTokVideo',
      instagram: 'InstagramReel'
    };
    return compositionMap[platform] || 'YouTubeShort';
  }
}
```

**Remotion composition example:**

```typescript
// remotion/compositions/YouTubeShort.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, spring } from 'remotion';
import React from 'react';

interface YouTubeShortProps {
  title: string;
  sections: string[];
  aspectRatio: string;
}

export const YouTubeShort: React.FC<YouTubeShortProps> = ({
  title,
  sections
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = spring({
    frame,
    fps,
    from: 0,
    to: 1,
    durationInFrames: 30
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        padding: 60, 
        color: 'white',
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center'
      }}>
        <h1 style={{ 
          fontSize: 64, 
          fontWeight: 'bold',
          opacity: titleOpacity,
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        {sections.map((section, index) => {
          const sectionOpacity = spring({
            frame: frame - (index * 60),
            fps,
            from: 0,
            to: 1,
            durationInFrames: 30
          });

          return (
            <p key={index} style={{ 
              fontSize: 32,
              opacity: sectionOpacity,
              marginBottom: 20,
              lineHeight: 1.5
            }}>
              • {section}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

**Register compositions:**

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { Composition } from 'remotion';
import { YouTubeShort } from './compositions/YouTubeShort';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="YouTubeShort"
        component={YouTubeShort}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Sample Title',
          sections: ['Point 1', 'Point 2', 'Point 3'],
          aspectRatio: '9:16'
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

## Complete Workflow Example

```typescript
// src/app/api/pipeline/route.ts
import { scrapeContent } from '@/lib/research/scraper';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoGenerator } from '@/lib/render/video-generator';

export async function POST(request: Request) {
  const { keyword, platforms } = await request.json();

  try {
    // Step 1: Research
    console.log('🔍 Researching content...');
    const researchData = await scrapeContent([
      { platform: 'techcrunch', keyword, timeRange: '24h' },
      { platform: 'twitter', keyword }
    ]);

    // Step 2: Generate Content (English & Vietnamese)
    console.log('✍️ Generating content...');
    const generator = new ContentGenerator();
    
    const englishContent = await generator.generateWithClaude({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      keyword,
      researchData
    });

    const vietnameseContent = await generator.generateWithClaude({
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      keyword,
      researchData
    });

    // Step 3: Generate Videos for Each Platform
    console.log('🎬 Rendering videos...');
    const videoGen = new VideoGenerator();
    
    const videos = await Promise.all(
      platforms.map((platform: string) =>
        videoGen.createVideoFromContent({
          title: englishContent.title,
          sections: englishContent.sections,
          aspectRatio: platform === 'youtube' ? '9:16' : '1:1',
          platform
        })
      )
    );

    return Response.json({
      success: true,
      content: {
        english: englishContent,
        vietnamese: vietnameseContent
      },
      videos
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return Response.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video (standalone)
npm run render
```

## Key API Endpoints

### Generate Content
```
POST /api/generate
Body: {
  "keyword": "AI automation",
  "format": "toplist",
  "language": "en",
  "tone": "expert",
  "provider": "claude"
}
```

### Full Pipeline
```
POST /api/pipeline
Body: {
  "keyword": "marketing trends 2024",
  "platforms": ["youtube", "tiktok", "instagram"]
}
```

### Research Only
```
POST /api/research
Body: {
  "keyword": "AI tools",
  "sources": ["techcrunch", "twitter"],
  "timeRange": "24h"
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const generator = new ContentGenerator();
  
  return Promise.all(
    keywords.map(async (keyword) => {
      const research = await scrapeContent([
        { platform: 'techcrunch', keyword, timeRange: '24h' }
      ]);
      
      return generator.generateWithClaude({
        format: 'how-to',
        language: 'en',
        tone: 'friendly',
        keyword,
        researchData: research
      });
    })
  );
}
```

### Multi-Language Content Pipeline

```typescript
async function generateMultiLanguageContent(keyword: string) {
  const generator = new ContentGenerator();
  const research = await scrapeContent([
    { platform: 'techcrunch', keyword, timeRange: '24h' }
  ]);

  const [english, vietnamese] = await Promise.all([
    generator.generateWithClaude({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      keyword,
      researchData: research
    }),
    generator.generateWithClaude({
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      keyword,
      researchData: research
    })
  ]);

  return { english, vietnamese };
}
```

## Troubleshooting

### API Rate Limits
```typescript
// Implement rate limiting for AI providers
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => generator.generateWithClaude(config))
  )
);
```

### Remotion Rendering Errors
```bash
# Ensure ffmpeg is installed
brew install ffmpeg  # macOS
sudo apt install ffmpeg  # Ubuntu

# Check Remotion license
echo $REMOTION_LICENSE_KEY
```

### Memory Issues with Large Videos
```typescript
// Adjust composition settings
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: compositionId,
  inputProps: {
    // Reduce quality for faster rendering
    quality: 70,
    // Limit frame rate
    fps: 24
  }
});
```

### Claude API Timeout
```typescript
// Add retry logic
async function generateWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generator.generateWithClaude(config);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cached = cache.get(keyword);
  if (cached) return cached;
  
  const research = await scrapeContent([
    { platform: 'techcrunch', keyword, timeRange: '24h' }
  ]);
  
  cache.set(keyword, research);
  return research;
}
```
