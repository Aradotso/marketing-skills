---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline
  - create automated content with marketing pipeline
  - generate videos from content automatically
  - research and write content with AI
  - automate content creation workflow
  - set up remotion video generation
  - crawl news and generate articles
  - build AI content automation system
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that handles the entire content creation workflow: from researching trending topics, generating multi-format articles, to automatically rendering videos. It integrates Claude 3, OpenAI, web scraping, and Remotion for video generation.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Bilingual support (English & Vietnamese)
- Automatic video/infographic rendering via Remotion
- Next.js frontend for easy content management

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
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion preview (for video development)
npm run remotion:preview

# Render video
npm run remotion:render
```

## Core Components

### 1. Research & Content Scraping

**Location:** `src/lib/research/scraper.ts`

```typescript
import { scrapeNews } from '@/lib/research/scraper';

// Scrape trending news from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await scrapeNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });
  
  return results; // Array of articles with title, url, summary, date
}

// Example usage
const aiNews = await gatherResearch('artificial intelligence');
console.log(aiNews);
```

### 2. AI Content Generation

**Location:** `src/lib/ai/content-generator.ts`

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Generate content using Claude
async function generateWithClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous',
  researchData: any[]
) {
  const prompt = buildPrompt(topic, format, language, tone, researchData);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].text;
}

// Generate content using OpenAI
async function generateWithOpenAI(
  topic: string,
  format: string,
  language: string,
  tone: string,
  researchData: any[]
) {
  const prompt = buildPrompt(topic, format, language, tone, researchData);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator and marketer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}

function buildPrompt(
  topic: string,
  format: string,
  language: string,
  tone: string,
  researchData: any[]
) {
  const research = researchData.map(r => 
    `- ${r.title}: ${r.summary} (${r.url})`
  ).join('\n');
  
  return `
Create a ${format} article about "${topic}" in ${language} with a ${tone} tone.

Based on this recent research:
${research}

Requirements:
- Use data-backed insights from the research
- Include relevant statistics and examples
- Format: ${format}
- Length: 1000-1500 words
- Include engaging headlines and subheadings
`;
}
```

### 3. Complete Content Pipeline

**Location:** `src/lib/pipeline/content-pipeline.ts`

```typescript
import { scrapeNews } from '@/lib/research/scraper';
import { generateWithClaude } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Researching...');
    const research = await scrapeNews({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      limit: 15
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateWithClaude(
      config.keyword,
      config.format,
      config.language,
      config.tone,
      research
    );
    
    // Step 3: Parse content into structured format
    const structuredContent = parseContent(content);
    
    // Step 4: Generate Video (optional)
    let videoUrl = null;
    if (config.generateVideo) {
      console.log('🎬 Rendering video...');
      videoUrl = await renderVideo({
        title: structuredContent.title,
        sections: structuredContent.sections,
        format: 'reels' // or 'tiktok', 'youtube-short'
      });
    }
    
    return {
      success: true,
      content: structuredContent,
      videoUrl,
      research: research.slice(0, 5) // Include top sources
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function parseContent(rawContent: string) {
  // Parse AI-generated content into structured format
  const lines = rawContent.split('\n');
  const title = lines.find(l => l.startsWith('#'))?.replace('#', '').trim();
  
  return {
    title,
    content: rawContent,
    sections: extractSections(rawContent),
    metadata: {
      generatedAt: new Date().toISOString(),
      wordCount: rawContent.split(' ').length
    }
  };
}

function extractSections(content: string) {
  // Extract key sections for video generation
  const sections = [];
  const lines = content.split('\n');
  
  for (let i = 0; i < lines.length; i++) {
    if (lines[i].startsWith('##')) {
      sections.push({
        heading: lines[i].replace('##', '').trim(),
        content: lines[i + 1] || ''
      });
    }
  }
  
  return sections;
}
```

### 4. Remotion Video Generation

**Location:** `src/remotion/compositions/ContentVideo.tsx`

```typescript
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';

interface ContentVideoProps {
  title: string;
  sections: Array<{
    heading: string;
    content: string;
  }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, sections }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity
          }}
        >
          <h1
            style={{
              fontSize: 80,
              color: 'white',
              fontWeight: 'bold',
              textAlign: 'center',
              padding: '0 100px'
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {sections.map((section, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 100
            }}
          >
            <div style={{ color: 'white' }}>
              <h2 style={{ fontSize: 60, marginBottom: 30 }}>
                {section.heading}
              </h2>
              <p style={{ fontSize: 36, lineHeight: 1.6 }}>
                {section.content}
              </p>
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

**Remotion Config:** `remotion.config.ts`

```typescript
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

**Render function:** `src/lib/video/remotion-renderer.ts`

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderVideo(props: {
  title: string;
  sections: any[];
  format: 'reels' | 'tiktok' | 'youtube-short';
}) {
  const bundled = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  const dimensions = getFormatDimensions(props.format);
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: props,
  });
  
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: props,
    ...dimensions
  });
  
  return `/videos/${path.basename(outputPath)}`;
}

function getFormatDimensions(format: string) {
  const formats = {
    'reels': { width: 1080, height: 1920, fps: 30 },
    'tiktok': { width: 1080, height: 1920, fps: 30 },
    'youtube-short': { width: 1080, height: 1920, fps: 30 }
  };
  
  return formats[format] || formats['reels'];
}
```

### 5. API Routes

**Location:** `src/app/api/generate/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, generateVideo } = body;
    
    // Validate input
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'expert',
      generateVideo: generateVideo || false
    });
    
    return NextResponse.json(result);
    
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
// In a React component or API route
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

async function generateContent() {
  const result = await runContentPipeline({
    keyword: 'AI automation tools',
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    generateVideo: false
  });
  
  console.log('Generated:', result.content.title);
  console.log('Sources:', result.research.length);
}
```

### Full Pipeline with Video

```typescript
async function createFullContentPackage() {
  const result = await runContentPipeline({
    keyword: 'marketing automation trends 2026',
    format: 'case-study',
    language: 'vi',
    tone: 'friendly',
    generateVideo: true
  });
  
  // Save to database or CMS
  await saveToDatabase({
    title: result.content.title,
    body: result.content.content,
    videoUrl: result.videoUrl,
    sources: result.research
  });
  
  return result;
}
```

### Custom Research Sources

```typescript
import { scrapeNews } from '@/lib/research/scraper';

// Scrape specific sources
const customResearch = await scrapeNews({
  keyword: 'SaaS growth',
  sources: ['techcrunch', 'a16z'],
  timeRange: '48h',
  limit: 10,
  filters: {
    minEngagement: 100,
    excludeDomains: ['spam-site.com']
  }
});
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateMultipleArticles(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: false
      })
    )
  );
  
  return results;
}

// Usage
const topics = ['AI tools', 'Marketing automation', 'SEO strategies'];
const articles = await generateMultipleArticles(topics);
```

### Bilingual Content Creation

```typescript
async function createBilingualContent(keyword: string) {
  const [english, vietnamese] = await Promise.all([
    runContentPipeline({
      keyword,
      format: 'how-to',
      language: 'en',
      tone: 'friendly',
      generateVideo: false
    }),
    runContentPipeline({
      keyword,
      format: 'how-to',
      language: 'vi',
      tone: 'friendly',
      generateVideo: false
    })
  ]);
  
  return { english, vietnamese };
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline({
      keyword: topic,
      format: 'pov',
      language: 'en',
      tone: 'expert',
      generateVideo: true
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic
async function generateWithRetry(config: PipelineConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await runContentPipeline(config);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        // Rate limited, wait and retry
        await new Promise(resolve => setTimeout(resolve, 5000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Issues

```bash
# Check Remotion installation
npm list @remotion/renderer

# Test video rendering
npm run remotion:preview

# Ensure ffmpeg is installed
ffmpeg -version
```

### Memory Issues with Large Scrapes

```typescript
// Process in chunks
async function scrapeInChunks(keywords: string[], chunkSize = 5) {
  const results = [];
  
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(kw => scrapeNews({ keyword: kw, limit: 10 }))
    );
    results.push(...chunkResults.flat());
    
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return results;
}
```

### Claude API Errors

```typescript
// Handle Claude-specific errors
try {
  const content = await generateWithClaude(/* ... */);
} catch (error) {
  if (error.status === 529) {
    console.log('Claude overloaded, falling back to OpenAI');
    return await generateWithOpenAI(/* ... */);
  }
  throw error;
}
```

## Best Practices

1. **Use environment variables** for all API keys
2. **Implement caching** for research results to avoid redundant scraping
3. **Rate limit** your API calls to avoid hitting quotas
4. **Store generated content** in a database for reuse
5. **Monitor costs** for AI API usage (Claude/OpenAI can be expensive at scale)
6. **Validate input** before running the pipeline
7. **Use streaming** for long content generation to improve UX
8. **Optimize video rendering** settings based on target platform
