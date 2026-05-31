---
name: pennydinh-marketing-pipeline-automation
description: Automated AI content pipeline for research, script generation, and video creation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - set up AI marketing content pipeline
  - generate videos from blog posts automatically
  - crawl news and create content with AI
  - build automated content workflow with Claude
  - create multilingual content with video rendering
  - automate social media content generation
  - setup Remotion video pipeline with AI
---

# Pennydinh Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use an automated content creation pipeline that handles everything from web scraping and research to AI-generated scripts and automated video rendering.

## Overview

The Ultimate AI Content Pipeline is a TypeScript-based system that automates the entire content creation workflow:

1. **Research Phase**: Auto-crawl news from sources (TechCrunch, Twitter, LinkedIn, a16z)
2. **Content Generation**: Use Claude/OpenAI to generate scripts in multiple formats and languages
3. **Video Rendering**: Convert text content to videos using Remotion
4. **Distribution**: Ready-to-publish content for various platforms

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

Create a `.env.local` file with required API credentials:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

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
│   │   ├── crawlers/    # Web scraping modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlTechCrunch, crawlTwitter, crawlLinkedIn } from '@/lib/crawlers';

async function gatherResearch(keyword: string, timeframe: string = '24h') {
  const sources = await Promise.all([
    crawlTechCrunch(keyword, timeframe),
    crawlTwitter(keyword, timeframe),
    crawlLinkedIn(keyword, timeframe)
  ]);

  const insights = sources.flat().map(article => ({
    title: article.title,
    url: article.url,
    excerpt: article.excerpt,
    publishedAt: article.publishedAt,
    relevanceScore: calculateRelevance(article, keyword)
  }));

  return insights.sort((a, b) => b.relevanceScore - a.relevanceScore);
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  insights: any[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(insights, format, language);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return {
    content,
    metadata: {
      format,
      language,
      wordCount: content.split(' ').length,
      generatedAt: new Date().toISOString()
    }
  };
}

function buildPrompt(insights: any[], format: string, language: string): string {
  const insightsSummary = insights.map(i => 
    `- ${i.title} (${i.url}): ${i.excerpt}`
  ).join('\n');

  const formatInstructions = {
    'toplist': 'Create a numbered list article highlighting the top insights',
    'pov': 'Write an opinion piece from an expert perspective',
    'case-study': 'Develop an in-depth case study with data analysis',
    'how-to': 'Create a step-by-step guide based on the insights'
  };

  const languageInstruction = language === 'vi' 
    ? 'Write in Vietnamese with a professional yet approachable tone.'
    : 'Write in English with a professional yet engaging tone.';

  return `
${formatInstructions[format]}

Research Data:
${insightsSummary}

Requirements:
- ${languageInstruction}
- Include data points and statistics from the research
- Make it SEO-optimized with clear headings
- Add a compelling introduction and conclusion
- Cite sources where appropriate
`;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  insights: any[],
  format: string,
  language: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in data-driven marketing content.'
      },
      {
        role: 'user',
        content: buildPrompt(insights, format, language)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: {
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
}) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });

  const outputLocation = path.resolve(`./public/videos/${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: content,
  });

  return outputLocation;
}
```

### 5. Remotion Video Component Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
}

export const ContentVideo: React.FC<VideoProps> = ({ title, keyPoints, stats }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={styles.titleSlide}>
          <h1 style={{
            fontSize: 60,
            color: '#fff',
            textAlign: 'center',
            opacity: Math.min(1, frame / 30)
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key Points */}
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 4)}
          durationInFrames={fps * 4}
        >
          <AbsoluteFill style={styles.pointSlide}>
            <h2 style={styles.pointText}>{point}</h2>
          </AbsoluteFill>
        </Sequence>
      ))}

      {/* Stats Sequence */}
      <Sequence from={fps * (3 + keyPoints.length * 4)} durationInFrames={fps * 5}>
        <AbsoluteFill style={styles.statsSlide}>
          {stats.map((stat, index) => (
            <div key={index} style={styles.statItem}>
              <div style={styles.statValue}>{stat.value}</div>
              <div style={styles.statLabel}>{stat.label}</div>
            </div>
          ))}
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};

const styles = {
  titleSlide: { justifyContent: 'center', alignItems: 'center' },
  pointSlide: { justifyContent: 'center', alignItems: 'center', padding: 40 },
  pointText: { fontSize: 40, color: '#fff', textAlign: 'center' as const },
  statsSlide: { flexDirection: 'row' as const, justifyContent: 'space-around', alignItems: 'center' },
  statItem: { textAlign: 'center' as const },
  statValue: { fontSize: 72, color: '#00ff88', fontWeight: 'bold' },
  statLabel: { fontSize: 24, color: '#fff', marginTop: 10 }
};
```

## Complete Pipeline Workflow

```typescript
// src/lib/pipeline/runPipeline.ts
export async function runContentPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    languages: ('en' | 'vi')[];
    generateVideo: boolean;
  }
) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const insights = await gatherResearch(keyword, '24h');

  // Step 2: Generate content in multiple languages
  console.log('✍️ Generating content...');
  const contentVersions = await Promise.all(
    options.languages.map(lang => 
      generateContent(insights, options.format, lang)
    )
  );

  const results = contentVersions.map((content, index) => ({
    language: options.languages[index],
    ...content
  }));

  // Step 3: Generate video if requested
  if (options.generateVideo) {
    console.log('🎬 Rendering video...');
    const videoContent = {
      title: extractTitle(results[0].content),
      keyPoints: extractKeyPoints(results[0].content),
      stats: extractStats(insights)
    };

    const videoPath = await generateVideo(videoContent);
    results[0].videoUrl = videoPath;
  }

  return results;
}

function extractKeyPoints(content: string): string[] {
  // Extract main points from content (simple regex-based)
  const matches = content.match(/^[-*]\s+(.+)$/gm);
  return matches ? matches.slice(0, 5).map(m => m.replace(/^[-*]\s+/, '')) : [];
}

function extractStats(insights: any[]): { label: string; value: string }[] {
  return insights.slice(0, 3).map(insight => ({
    label: insight.title.substring(0, 30) + '...',
    value: insight.relevanceScore.toFixed(0) + '%'
  }));
}

function extractTitle(content: string): string {
  const match = content.match(/^#\s+(.+)$/m);
  return match ? match[1] : 'Generated Content';
}
```

## Usage in Next.js API Route

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/runPipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages, generateVideo } = await request.json();

    const results = await runContentPipeline(keyword, {
      format,
      languages,
      generateVideo
    });

    return NextResponse.json({
      success: true,
      data: results
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Client-Side Usage

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: 'AI in marketing',
        format: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: true
      })
    });

    const data = await response.json();
    setResults(data);
    setLoading(false);
  }

  return (
    <div>
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {results && (
        <div>
          {results.data.map((item, index) => (
            <div key={index}>
              <h3>{item.language.toUpperCase()}</h3>
              <div dangerouslySetInnerHTML={{ __html: item.content }} />
              {item.videoUrl && (
                <video src={item.videoUrl} controls />
              )}
            </div>
          ))}
        </div>
      )}
    </div>
  );
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

# Render Remotion video preview
npm run remotion
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const content = await runContentPipeline(keyword, {
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: false
    });
    
    results.push({ keyword, content });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Content Templates

```typescript
const CONTENT_TEMPLATES = {
  startup: {
    format: 'case-study',
    tone: 'professional',
    sections: ['problem', 'solution', 'results', 'learnings']
  },
  tutorial: {
    format: 'how-to',
    tone: 'friendly',
    sections: ['intro', 'prerequisites', 'steps', 'conclusion']
  }
};

async function generateFromTemplate(
  templateName: keyof typeof CONTENT_TEMPLATES,
  data: any
) {
  const template = CONTENT_TEMPLATES[templateName];
  // Apply template-specific logic
  return generateContent(data, template.format, 'en');
}
```

## Troubleshooting

**API Rate Limits**: Implement exponential backoff:
```typescript
async function withRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
    }
  }
}
```

**Remotion Memory Issues**: Reduce video resolution or duration:
```typescript
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
  inputProps: { ...content, quality: 'medium' }, // Add quality control
});
```

**Crawler Blocked**: Use rotating proxies or headers:
```typescript
const headers = {
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
};
```

This skill provides comprehensive coverage for building automated content marketing pipelines with AI-powered research, generation, and video rendering capabilities.
