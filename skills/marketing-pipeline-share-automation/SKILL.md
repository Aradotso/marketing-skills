---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate video from blog posts automatically
  - scrape news and create content pipeline
  - build AI marketing automation system
  - create multi-format content with Claude
  - set up automated content research pipeline
  - generate videos with Remotion from AI content
  - automate social media content generation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to automatic posting and video generation.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-scans and researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates AI content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline

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
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering
npm run render
```

## Core Architecture

### 1. Research Module

The research module scrapes and analyzes real-time data from multiple sources:

```typescript
// src/lib/research/scraper.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  selector?: string;
}

export async function scrapeNews(keyword: string, sources: ResearchSource[]) {
  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const response = await axios.get(source.url, {
          params: { q: keyword },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          },
        });
        return {
          source: source.name,
          articles: response.data.articles || [],
        };
      } catch (error) {
        console.error(`Error scraping ${source.name}:`, error);
        return { source: source.name, articles: [] };
      }
    })
  );
  
  return results;
}

export async function analyzeResearch(articles: any[]) {
  const insights = articles.map(article => ({
    title: article.title,
    summary: article.description,
    url: article.url,
    publishedAt: article.publishedAt,
  }));
  
  return insights;
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with multiple format options:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: any[];
}

export async function generateContent(
  keyword: string,
  options: ContentOptions
) {
  const systemPrompt = buildSystemPrompt(options);
  const userPrompt = buildUserPrompt(keyword, options.research);

  // Using Claude
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return {
    content,
    metadata: {
      model: 'claude-3-5-sonnet',
      format: options.format,
      language: options.language,
    },
  };
}

function buildSystemPrompt(options: ContentOptions): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with detailed explanations for each item',
    'pov': 'Write from a specific perspective or viewpoint with personal insights',
    'case-study': 'Structure as a case study with problem, solution, and results',
    'how-to': 'Create a step-by-step tutorial format',
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Use light, engaging language with wit',
  };

  return `You are an expert content creator. ${formatInstructions[options.format]}. ${toneInstructions[options.tone]}. Language: ${options.language === 'vi' ? 'Vietnamese' : 'English'}.`;
}

function buildUserPrompt(keyword: string, research: any[]): string {
  const researchSummary = research
    .map(r => `- ${r.title}: ${r.summary}`)
    .join('\n');

  return `Create content about "${keyword}" based on this research:\n\n${researchSummary}\n\nInclude data-backed insights and recent trends.`;
}
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// src/lib/ai/bilingual-generator.ts
export async function generateBilingualContent(
  keyword: string,
  research: any[],
  format: string
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(keyword, {
      format: format as any,
      language: 'en',
      tone: 'expert',
      research,
    }),
    generateContent(keyword, {
      format: format as any,
      language: 'vi',
      tone: 'expert',
      research,
    }),
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### 4. Video Generation with Remotion

Transform content into videos automatically:

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  duration,
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <div style={{ padding: 60, maxWidth: 1080 }}>
        <h1 style={{ 
          fontSize: 72, 
          color: '#fff',
          marginBottom: 40,
          fontWeight: 'bold',
        }}>
          {title}
        </h1>
        
        {points.map((point, index) => {
          const pointOpacity = interpolate(
            frame,
            [30 + index * 20, 50 + index * 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <p
              key={index}
              style={{
                fontSize: 36,
                color: '#e0e0e0',
                marginBottom: 24,
                opacity: pointOpacity,
              }}
            >
              {index + 1}. {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// src/remotion/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: { title: string; points: string[] },
  outputPath: string
) {
  const bundled = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: content,
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content,
  });

  return outputPath;
}
```

### 5. Complete Pipeline API

Create an API endpoint that orchestrates the entire pipeline:

```typescript
// src/app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNews, analyzeResearch } from '@/lib/research/scraper';
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';
import { renderContentVideo } from '@/remotion/render-video';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, sources } = await request.json();

    // Step 1: Research
    const newsData = await scrapeNews(keyword, sources);
    const research = await analyzeResearch(
      newsData.flatMap(d => d.articles)
    );

    // Step 2: Generate content
    const content = await generateBilingualContent(
      keyword,
      research,
      format
    );

    // Step 3: Extract key points for video
    const points = extractKeyPoints(content.en.content);

    // Step 4: Render video
    const videoPath = await renderContentVideo(
      {
        title: keyword,
        points: points.slice(0, 5),
      },
      `./public/videos/${Date.now()}.mp4`
    );

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl: videoPath.replace('./public', ''),
        research: research.slice(0, 10),
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - split by numbered lists
  const matches = content.match(/\d+\.\s+([^\n]+)/g);
  return matches 
    ? matches.map(m => m.replace(/^\d+\.\s+/, '')) 
    : [];
}
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
// Quick single-language content
import { generateContent } from '@/lib/ai/content-generator';

const content = await generateContent('AI Marketing Trends 2024', {
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  research: [],
});
```

### Pattern 2: Full Research Pipeline

```typescript
// Complete pipeline with research
const sources = [
  { name: 'TechCrunch', url: 'https://api.example.com/techcrunch' },
  { name: 'a16z', url: 'https://api.example.com/a16z' },
];

const research = await scrapeNews('artificial intelligence', sources);
const analyzed = await analyzeResearch(research.flatMap(r => r.articles));
const content = await generateBilingualContent(
  'AI in Marketing',
  analyzed,
  'case-study'
);
```

### Pattern 3: Video-First Workflow

```typescript
// Generate content optimized for video
const videoContent = await generateContent('Top 5 AI Tools', {
  format: 'toplist',
  language: 'en',
  tone: 'friendly',
  research: [],
});

const points = extractKeyPoints(videoContent.content);
await renderContentVideo(
  { title: 'Top 5 AI Tools', points },
  './output/video.mp4'
);
```

## CLI Commands (if available)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI Marketing" --format toplist

# Render video from existing content
npm run render -- --input content.json --output video.mp4

# Run full pipeline
npm run pipeline -- --keyword "Tech Trends" --sources all
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```typescript
// Add exponential backoff
async function generateWithRetry(keyword: string, options: ContentOptions, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(keyword, options);
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

For large videos, increase Node memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run render
```

### Research Scraping Failures

Handle failed sources gracefully:

```typescript
const safeResearch = research.filter(r => r.articles.length > 0);
if (safeResearch.length === 0) {
  // Fallback to AI knowledge only
  console.warn('No research data available, using AI knowledge');
}
```

## Advanced Configuration

### Custom Content Templates

```typescript
// src/lib/templates/custom-template.ts
export const customTemplates = {
  'viral-thread': {
    system: 'Create viral Twitter/X thread format with hooks',
    structure: ['Hook', 'Context', 'Value Points', 'CTA'],
  },
  'linkedin-post': {
    system: 'Create professional LinkedIn post with storytelling',
    structure: ['Personal Story', 'Lesson', 'Application', 'Question'],
  },
};
```

### Multi-Platform Video Export

```typescript
// Different aspect ratios for platforms
const platforms = {
  tiktok: { width: 1080, height: 1920 }, // 9:16
  youtube: { width: 1920, height: 1080 }, // 16:9
  instagram: { width: 1080, height: 1080 }, // 1:1
};

export async function renderForPlatform(
  content: any,
  platform: keyof typeof platforms
) {
  const dimensions = platforms[platform];
  // Pass dimensions to Remotion composition
}
```

This skill enables comprehensive automation of content marketing workflows using TypeScript, AI models, and video generation technology.
