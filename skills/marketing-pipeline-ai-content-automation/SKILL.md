---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated content
  - create automated content workflow from research to video
  - build AI-powered content automation system
  - generate videos from AI-written content automatically
  - configure Claude and OpenAI for content pipeline
  - deploy automated marketing content generation
  - research and write content automatically with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an all-in-one automated content pipeline that transforms keywords into complete content pieces with research, AI-generated scripts, and rendered videos. It crawls fresh data from sources like TechCrunch, a16z, Twitter, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats (toplist, POV, case studies, how-to guides) in both English and Vietnamese, and finally renders videos using Remotion.

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database for caching
DATABASE_URL=your_database_connection_string

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Core Architecture

The pipeline consists of three main stages:

1. **Research Stage**: Crawls and aggregates real-time data
2. **Content Generation Stage**: Uses AI to create scripts/articles
3. **Rendering Stage**: Generates videos and graphics using Remotion

## Key Components

### 1. Auto-Scan Research Module

```typescript
// lib/research/crawler.ts
import { searchNews } from './apis/news-api';
import { scrapeSocialMedia } from './apis/social-scraper';

interface ResearchConfig {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeRange: number; // hours
  limit: number;
}

export async function conductResearch(config: ResearchConfig) {
  const results = await Promise.all([
    searchNews(config.keyword, config.timeRange),
    scrapeSocialMedia(config.keyword, config.sources)
  ]);
  
  return {
    articles: results[0],
    socialPosts: results[1],
    aggregatedInsights: analyzeData(results)
  };
}

// Example usage
const research = await conductResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter'],
  timeRange: 24,
  limit: 50
});
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(config);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return {
    content: message.content[0].text,
    metadata: {
      format: config.format,
      language: config.language,
      wordCount: message.content[0].text.split(' ').length
    }
  };
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a specific perspective with personal insights',
    'case-study': 'Analyze with data, examples, and outcomes',
    'how-to': 'Step-by-step guide with actionable instructions'
  };

  return `
You are a ${config.tone} content writer.
Format: ${formatInstructions[config.format]}
Language: ${config.language === 'en' ? 'English' : 'Vietnamese'}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create comprehensive, engaging content based on this research.
`;
}
```

### 3. Bilingual Content Pipeline

```typescript
// lib/ai/bilingual-generator.ts
export async function generateBilingualContent(researchData: any, format: ContentFormat) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format,
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      format,
      language: 'vi',
      tone: 'friendly',
      researchData
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
    translations: {
      title: {
        en: extractTitle(englishContent.content),
        vi: extractTitle(vietnameseContent.content)
      }
    }
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

const VIDEO_DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function renderContentVideo(config: VideoConfig) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      ...VIDEO_DIMENSIONS[config.format]
    },
  });

  const outputLocation = `out/${Date.now()}-${config.format}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content
    },
  });

  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  content: string;
  width: number;
  height: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, width, height }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30, 120, 150],
    [0, 1, 1, 0]
  );

  const sections = parseContentSections(content);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity, padding: 60, maxWidth: width * 0.8 }}>
        {sections.map((section, idx) => (
          <div key={idx} style={{ marginBottom: 40 }}>
            <h2 style={{ color: '#fff', fontSize: 48, fontWeight: 'bold' }}>
              {section.title}
            </h2>
            <p style={{ color: '#ccc', fontSize: 32, lineHeight: 1.6 }}>
              {section.text}
            </p>
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};

function parseContentSections(content: string) {
  // Parse markdown-style content into sections
  const lines = content.split('\n');
  const sections = [];
  let currentSection = { title: '', text: '' };

  for (const line of lines) {
    if (line.startsWith('##')) {
      if (currentSection.title) sections.push(currentSection);
      currentSection = { title: line.replace('##', '').trim(), text: '' };
    } else if (line.trim()) {
      currentSection.text += line + ' ';
    }
  }
  
  if (currentSection.title) sections.push(currentSection);
  return sections;
}
```

## Complete Pipeline Workflow

```typescript
// app/api/pipeline/route.ts
import { conductResearch } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: Request) {
  const { keyword, format, videoFormat } = await req.json();

  try {
    // Step 1: Research
    const research = await conductResearch({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeRange: 24,
      limit: 50
    });

    // Step 2: Generate Content
    const content = await generateBilingualContent(research, format);

    // Step 3: Render Video (optional)
    let videoPath = null;
    if (videoFormat) {
      videoPath = await renderContentVideo({
        content: content.en.content,
        format: videoFormat,
        duration: 60
      });
    }

    return Response.json({
      success: true,
      data: {
        research: research.aggregatedInsights,
        content,
        video: videoPath
      }
    });
  } catch (error) {
    return Response.json({ 
      success: false, 
      error: error.message 
    }, { status: 500 });
  }
}
```

## Next.js API Routes

```typescript
// app/api/content/generate/route.ts
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const body = await request.json();
  
  const { keyword, format = 'toplist', languages = ['en', 'vi'] } = body;

  // Validate input
  if (!keyword) {
    return Response.json({ error: 'Keyword is required' }, { status: 400 });
  }

  // Execute pipeline
  const research = await conductResearch({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeRange: 24,
    limit: 30
  });

  const results = {};
  for (const lang of languages) {
    results[lang] = await generateContent({
      format,
      language: lang,
      tone: 'expert',
      researchData: research
    });
  }

  return Response.json({ success: true, results });
}
```

## Client-Side Usage

```typescript
// components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setLoading(true);

    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        videoFormat: 'reels'
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded"
      />
      <button type="submit" disabled={loading} className="ml-2 px-4 py-2 bg-blue-500 text-white rounded">
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-4">
          <h3>Generated Content</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </form>
  );
}
```

## Common Patterns

### Scheduling Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 8 AM
  cron.schedule('0 8 * * *', async () => {
    for (const keyword of keywords) {
      await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format: 'toplist' })
      });
    }
  });
}
```

### Caching Research Results

```typescript
// lib/cache/research-cache.ts
const cache = new Map();

export async function getCachedResearch(keyword: string, maxAge: number = 3600000) {
  const cached = cache.get(keyword);
  
  if (cached && Date.now() - cached.timestamp < maxAge) {
    return cached.data;
  }

  const fresh = await conductResearch({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeRange: 24,
    limit: 50
  });

  cache.set(keyword, { data: fresh, timestamp: Date.now() });
  return fresh;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting errors:

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

export async function batchGenerate(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword => 
      limit(() => generateContent({ 
        keyword, 
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: {}
      }))
    )
  );
}
```

### Remotion Rendering Failures

Ensure sufficient memory for video rendering:

```typescript
// Increase Node.js memory limit
// package.json scripts:
{
  "scripts": {
    "render": "node --max-old-space-size=8192 scripts/render.js"
  }
}
```

### AI Response Quality

Improve prompt engineering for better results:

```typescript
const enhancedPrompt = `
Context: You are an expert content strategist with 10+ years of experience.

Research Data Summary:
${summarizeResearch(researchData)}

Task: Create a ${format} article that:
- Uses data from the last 24 hours
- Includes specific examples and numbers
- Targets ${targetAudience}
- Maintains a ${tone} tone

Output in ${language}.
`;
```

### Database Persistence

Store generated content for later use:

```typescript
// lib/db/content-store.ts
import { prisma } from './prisma';

export async function saveContent(content: any) {
  return prisma.content.create({
    data: {
      keyword: content.keyword,
      format: content.format,
      language: content.language,
      text: content.content,
      videoUrl: content.videoPath,
      metadata: content.metadata
    }
  });
}
```
