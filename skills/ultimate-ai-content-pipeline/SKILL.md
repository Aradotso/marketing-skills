---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated content pipeline with video generation
  - create content from research to video automatically
  - integrate Claude and OpenAI for content automation
  - generate videos from written content with Remotion
  - build automated marketing content workflow
  - scrape news and create AI-powered content
  - automate blog posts and social media videos
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

This TypeScript project automates the entire content creation workflow: from researching trending topics, generating multilingual content in various formats (toplist, POV, case studies), to rendering videos and infographics using Remotion. It leverages Claude 3, OpenAI, and web scraping to create data-backed, trend-leading content.

## What It Does

**Ultimate AI Content Pipeline** is an end-to-end content automation system that:

- **Auto-crawls** news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates content** in multiple formats (toplist, POV, how-to, case studies)
- **Supports multilingual** output (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for Reels, TikTok, Shorts
- **Provides Next.js UI** for managing the pipeline with a few clicks

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

Create a `.env.local` file in the project root:

```env
# AI Services
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=your_database_url_here

# Remotion License (if using Remotion Cloud)
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Key Architecture

```
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integrations (Claude, OpenAI)
│   ├── scraper/           # Web scraping modules
│   └── video/             # Remotion video rendering
├── remotion/              # Remotion video templates
└── public/                # Static assets
```

## Core Modules

### 1. Research & Scraping

```typescript
// lib/scraper/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  timeframe: string;
}

export async function crawlTrendingNews(
  keyword: string,
  sources: NewsSource[]
): Promise<Array<{ title: string; url: string; summary: string }>> {
  const results = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword, hours: 24 },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-scraper.p.rapidapi.com'
        }
      });
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source.url}:`, error);
    }
  }
  
  return results;
}

// Usage
const news = await crawlTrendingNews('AI marketing', [
  { url: 'https://techcrunch.com/api', selector: '.article', timeframe: '24h' },
  { url: 'https://a16z.com/api', selector: '.post', timeframe: '24h' }
]);
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  research: Array<{ title: string; summary: string }>;
}

export async function generateContent(
  keyword: string,
  config: ContentConfig
): Promise<{ title: string; content: string; metadata: object }> {
  
  const researchContext = config.research
    .map(r => `- ${r.title}: ${r.summary}`)
    .join('\n');
  
  const prompt = `
You are an expert content creator. Based on this research:

${researchContext}

Create a ${config.format} article about "${keyword}" in ${config.language} with a ${config.tone} tone.

Requirements:
- Data-backed insights
- SEO-optimized
- Engaging and actionable
- Include relevant statistics from research

Format: JSON with title, content (markdown), and metadata (tags, readTime).
`;

  try {
    // Use Claude for longer, analytical content
    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    const result = JSON.parse(response.content[0].text);
    return result;
    
  } catch (error) {
    // Fallback to OpenAI
    const response = await openai.chat.completions.create({
      model: 'gpt-4-turbo',
      messages: [{ role: 'user', content: prompt }],
      response_format: { type: 'json_object' }
    });
    
    return JSON.parse(response.choices[0].message.content);
  }
}

// Usage
const article = await generateContent('AI automation', {
  format: 'toplist',
  language: 'vi',
  tone: 'professional',
  research: newsData
});
```

### 3. Video Generation with Remotion

```typescript
// lib/video/render-content.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  points: string[];
  brandColor: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export async function renderContentVideo(
  content: VideoConfig,
  outputPath: string
): Promise<string> {
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content
  });
  
  // Render video
  const outputFile = path.join(outputPath, `video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputFile,
    inputProps: content
  });
  
  return outputFile;
}

// Usage
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Marketing Tools',
  points: [
    'Claude for content generation',
    'Midjourney for visuals',
    'ElevenLabs for voiceovers',
    'Remotion for videos',
    'Make.com for automation'
  ],
  brandColor: '#FF6B6B',
  aspectRatio: '9:16'
}, './output');
```

### 4. Remotion Video Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: titleOpacity 
        }}>
          <h1 style={{ 
            color: brandColor, 
            fontSize: 60,
            fontWeight: 'bold',
            textAlign: 'center',
            padding: 40
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Points Sequences */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill style={{ 
            justifyContent: 'center', 
            alignItems: 'center' 
          }}>
            <div style={{
              backgroundColor: brandColor,
              padding: 30,
              borderRadius: 20,
              maxWidth: '80%'
            }}>
              <h2 style={{ 
                color: '#fff', 
                fontSize: 40,
                margin: 0 
              }}>
                {index + 1}. {point}
              </h2>
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { crawlTrendingNews } from '../scraper/news-crawler';
import { generateContent } from '../ai/content-generator';
import { renderContentVideo } from '../video/render-content';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoAspectRatio?: '9:16' | '16:9' | '1:1';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);
  
  // Step 1: Research
  console.log('📡 Crawling trending news...');
  const research = await crawlTrendingNews(config.keyword, [
    { url: 'techcrunch-api', selector: '.article', timeframe: '24h' },
    { url: 'a16z-api', selector: '.post', timeframe: '24h' }
  ]);
  
  const results = [];
  
  // Step 2: Generate content for each language
  for (const language of config.languages) {
    console.log(`🧠 Generating ${language} content...`);
    
    const content = await generateContent(config.keyword, {
      format: config.contentFormat,
      language,
      tone: 'professional',
      research
    });
    
    results.push({ language, content });
    
    // Step 3: Generate video if requested
    if (config.generateVideo) {
      console.log(`🎬 Rendering ${language} video...`);
      
      // Extract key points from content
      const points = extractKeyPoints(content.content);
      
      const videoPath = await renderContentVideo({
        title: content.title,
        points,
        brandColor: '#4F46E5',
        aspectRatio: config.videoAspectRatio || '9:16'
      }, `./output/${language}`);
      
      results[results.length - 1].videoPath = videoPath;
    }
  }
  
  console.log('✅ Pipeline complete!');
  return results;
}

function extractKeyPoints(markdownContent: string): string[] {
  // Extract bullet points or numbered lists
  const matches = markdownContent.match(/^[-*\d.]\s+(.+)$/gm) || [];
  return matches.slice(0, 5).map(m => m.replace(/^[-*\d.]\s+/, ''));
}

// Usage
const results = await runContentPipeline({
  keyword: 'AI Marketing Automation',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoAspectRatio: '9:16'
});
```

## Next.js API Routes

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const results = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
      videoAspectRatio: body.aspectRatio || '9:16'
    });
    
    return NextResponse.json({ 
      success: true, 
      results 
    });
    
  } catch (error) {
    return NextResponse.json({ 
      success: false, 
      error: error.message 
    }, { status: 500 });
  }
}
```

## Running the Project

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion:preview

# Render specific composition
npm run remotion:render ContentVideo output/video.mp4
```

## Common Patterns

### Custom Content Formats

```typescript
// Define custom format templates
const formatTemplates = {
  'toplist': 'Create a numbered list of top items with explanations',
  'pov': 'Write from a unique perspective with personal insights',
  'case-study': 'Analyze a real-world example with data and outcomes',
  'how-to': 'Step-by-step guide with actionable instructions'
};

// Extend with your own
formatTemplates['comparison'] = 'Compare multiple options with pros/cons';
```

### Multi-Platform Video Exports

```typescript
// Generate videos for all platforms
const aspectRatios = {
  instagram: '9:16',
  youtube: '16:9',
  linkedin: '1:1'
};

for (const [platform, ratio] of Object.entries(aspectRatios)) {
  await renderContentVideo({
    ...videoConfig,
    aspectRatio: ratio
  }, `./output/${platform}`);
}
```

### Batch Processing

```typescript
// Process multiple keywords
const keywords = ['AI Marketing', 'Content Automation', 'Video Generation'];

const batchResults = await Promise.all(
  keywords.map(keyword => 
    runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true
    })
  )
);
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent API calls

const results = await Promise.all(
  tasks.map(task => limit(() => apiCall(task)))
);
```

### Video Rendering Timeouts

```typescript
// Increase timeout for complex videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputFile,
  timeoutInMilliseconds: 120000, // 2 minutes
  inputProps: content
});
```

### Memory Issues with Large Content

```typescript
// Process in chunks
const chunkSize = 5;
for (let i = 0; i < items.length; i += chunkSize) {
  const chunk = items.slice(i, i + chunkSize);
  await processChunk(chunk);
}
```

### Missing Environment Variables

```typescript
// Validate before starting
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

for (const varName of requiredEnvVars) {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
}
```

## Best Practices

1. **Cache research results** to avoid repeated API calls
2. **Queue video rendering** for resource-intensive operations
3. **Version your prompts** for reproducible content generation
4. **Log pipeline steps** for debugging and monitoring
5. **Store generated content** in a database for reuse and analytics

This skill enables AI agents to help developers build automated content creation pipelines that transform keywords into polished articles and videos across multiple platforms and languages.
