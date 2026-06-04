---
name: marketing-pipeline-share-automation
description: TypeScript automation pipeline for AI-powered content creation from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create videos from articles automatically
  - set up content automation workflow
  - research and write content with Claude
  - build marketing content pipeline
  - auto-generate social media videos
  - scrape news and create content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a complete TypeScript-based automation system that handles content creation from research through article generation to video rendering. The pipeline integrates Claude 3, OpenAI, RapidAPI for research, and Remotion for video generation.

## What This Project Does

Marketing Pipeline Share automates the entire content lifecycle:
1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn for recent insights
2. **AI Content Generation**: Uses Claude/OpenAI to write articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates Vietnamese and English content simultaneously
4. **Video Rendering**: Converts articles to videos/infographics using Remotion
5. **Multi-platform Optimization**: Exports for Reels, TikTok, Shorts

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Service Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Optional: Social Media Auto-posting
FACEBOOK_PAGE_TOKEN=your_fb_token
LINKEDIN_TOKEN=your_linkedin_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/       # Web scraping & data collection
│   ├── generation/     # AI content generation
│   ├── render/         # Remotion video rendering
│   ├── api/           # API routes
│   └── utils/         # Helper functions
├── pages/             # Next.js pages
├── public/            # Static assets
└── remotion/          # Video templates
```

## Core Usage Patterns

### 1. Research Module - Auto Content Discovery

```typescript
// src/research/newsScraper.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  category: string;
}

export async function scrapeLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`https://api.rapidapi.com/news/${source}`, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        },
        params: {
          q: keyword,
          freshness: 'Day' // Last 24 hours
        }
      });
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return articles;
}

export async function extractInsights(articles: NewsArticle[]): Promise<Insight[]> {
  // Extract key insights, stats, and trends
  const insights = articles.map(article => ({
    title: article.title,
    summary: article.description,
    stats: extractStatistics(article.content),
    trends: identifyTrends(article.content),
    source: article.source
  }));
  
  return insights;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/generation/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  insights: Insight[];
}

export async function generateArticle(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a specific perspective with personal insights',
    'case-study': 'Analyze a real-world example with data',
    'how-to': 'Provide step-by-step instructions'
  };
  
  const toneInstructions = {
    'expert': 'professional and authoritative',
    'friendly': 'conversational and approachable',
    'humorous': 'witty and entertaining'
  };
  
  return `
You are a ${toneInstructions[request.tone]} content writer.

Topic: ${request.keyword}
Format: ${formatInstructions[request.format]}
Language: ${request.language === 'vi' ? 'Vietnamese' : 'English'}

Recent Insights:
${request.insights.map(i => `- ${i.title}: ${i.summary}`).join('\n')}

Write a comprehensive article (1500-2000 words) incorporating these insights.
Include:
1. Attention-grabbing headline
2. Data-backed statements
3. Actionable takeaways
4. SEO-optimized structure
`;
}
```

### 3. OpenAI Alternative Generation

```typescript
// src/generation/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithGPT(
  prompt: string,
  model: string = 'gpt-4-turbo-preview'
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model,
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content writer specialized in creating engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Dual-Language Generation

```typescript
// src/generation/multiLanguage.ts
export async function generateBilingualContent(
  keyword: string,
  insights: Insight[],
  format: ContentFormat
): Promise<{ vi: string; en: string }> {
  const [vietnameseContent, englishContent] = await Promise.all([
    generateArticle({
      keyword,
      format,
      language: 'vi',
      tone: 'friendly',
      insights
    }),
    generateArticle({
      keyword,
      format,
      language: 'en',
      tone: 'expert',
      insights
    })
  ]);
  
  return {
    vi: vietnameseContent,
    en: englishContent
  };
}
```

### 5. Video Rendering with Remotion

```typescript
// src/render/videoGenerator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
  duration: number;
}

export async function renderContentVideo(
  article: string,
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  // Extract video metadata
  const inputProps = {
    title: config.title,
    keyPoints: config.keyPoints,
    stats: config.stats
  };
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps
  });
  
  return outputLocation;
}
```

### 6. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  stats
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ padding: 60, color: 'white' }}>
        {/* Title Animation */}
        <h1 style={{ 
          opacity: titleOpacity,
          fontSize: 48,
          fontWeight: 'bold',
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        {/* Key Points */}
        {keyPoints.map((point, i) => {
          const startFrame = 60 + (i * 60);
          const opacity = interpolate(
            frame,
            [startFrame, startFrame + 30],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div key={i} style={{ opacity, marginBottom: 20 }}>
              <p style={{ fontSize: 24 }}>✓ {point}</p>
            </div>
          );
        })}
        
        {/* Stats Display */}
        <div style={{ 
          marginTop: 60,
          display: 'flex',
          gap: 40 
        }}>
          {stats.map((stat, i) => (
            <div key={i} style={{ textAlign: 'center' }}>
              <div style={{ fontSize: 36, fontWeight: 'bold', color: '#00ff88' }}>
                {stat.value}
              </div>
              <div style={{ fontSize: 16, opacity: 0.7 }}>
                {stat.label}
              </div>
            </div>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 7. Complete Pipeline API Route

```typescript
// pages/api/generate-content.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { scrapeLatestNews, extractInsights } from '@/src/research/newsScraper';
import { generateBilingualContent } from '@/src/generation/multiLanguage';
import { renderContentVideo } from '@/src/render/videoGenerator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, generateVideo } = req.body;
    
    // Step 1: Research
    console.log('📡 Starting research...');
    const articles = await scrapeLatestNews(keyword);
    const insights = await extractInsights(articles);
    
    // Step 2: Generate Content
    console.log('🧠 Generating content...');
    const content = await generateBilingualContent(
      keyword,
      insights,
      format
    );
    
    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (generateVideo) {
      console.log('🎬 Rendering video...');
      const keyPoints = extractKeyPoints(content.en);
      const stats = extractStats(insights);
      
      videoUrl = await renderContentVideo(content.en, {
        title: keyword,
        keyPoints,
        stats,
        duration: 30
      });
    }
    
    return res.status(200).json({
      success: true,
      data: {
        content,
        insights: insights.length,
        videoUrl
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({ 
      error: 'Content generation failed',
      details: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or key sentences
  const lines = content.split('\n');
  return lines
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('•'))
    .map(line => line.replace(/^[-•]\s*/, '').trim())
    .slice(0, 5);
}

function extractStats(insights: Insight[]): { label: string; value: string }[] {
  return insights
    .flatMap(i => i.stats)
    .slice(0, 3)
    .map(stat => ({
      label: stat.metric,
      value: stat.value
    }));
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion Studio (video preview)
npm run remotion

# Render a video composition
npx remotion render src/index.ts ContentVideo output.mp4
```

## Common Workflows

### Workflow 1: Full Automation

```typescript
// scripts/automate.ts
import { runFullPipeline } from '@/src/pipeline';

async function main() {
  const keywords = ['AI Marketing', 'Content Automation', 'Video Marketing'];
  
  for (const keyword of keywords) {
    console.log(`\n🚀 Processing: ${keyword}`);
    
    const result = await runFullPipeline({
      keyword,
      format: 'toplist',
      languages: ['vi', 'en'],
      generateVideo: true,
      autoPost: false // Set true to auto-post to social media
    });
    
    console.log(`✅ Generated:`, result);
  }
}

main();
```

### Workflow 2: Research Only

```typescript
import { scrapeLatestNews, extractInsights } from '@/src/research/newsScraper';

const articles = await scrapeLatestNews('Web3 Marketing', [
  'techcrunch',
  'a16z',
  'twitter'
]);

const insights = await extractInsights(articles);

console.log(`Found ${insights.length} insights`);
insights.forEach(i => console.log(`- ${i.title}`));
```

### Workflow 3: Content Generation Only

```typescript
import { generateArticle } from '@/src/generation/contentGenerator';

const article = await generateArticle({
  keyword: 'AI Content Marketing',
  format: 'how-to',
  language: 'en',
  tone: 'expert',
  insights: [] // Use empty if no research data
});

console.log(article);
```

## Configuration Options

### Content Formats

- `toplist`: Numbered list articles with rankings
- `pov`: Opinion pieces with personal perspective
- `case-study`: Data-driven analysis of real examples
- `how-to`: Step-by-step tutorials

### Tone Options

- `expert`: Professional, authoritative voice
- `friendly`: Conversational, approachable
- `humorous`: Entertaining, witty

### Video Export Formats

```typescript
// remotion.config.ts
export const videoFormats = {
  reels: { width: 1080, height: 1920 }, // 9:16
  tiktok: { width: 1080, height: 1920 }, // 9:16
  shorts: { width: 1080, height: 1920 }, // 9:16
  landscape: { width: 1920, height: 1080 }, // 16:9
  square: { width: 1080, height: 1080 } // 1:1
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/utils/rateLimiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  delayMs: number = 1000
): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return fn();
}

// Usage
const article = await withRateLimit(
  () => generateArticle(config),
  2000 // 2 second delay
);
```

### Video Rendering Memory Issues

```typescript
// Reduce video quality if memory issues occur
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps,
  chromiumOptions: {
    gl: 'angle' // Use ANGLE for better compatibility
  },
  envVariables: {
    NODE_OPTIONS: '--max-old-space-size=4096'
  }
});
```

### Claude API Errors

```typescript
// Add retry logic
async function generateWithRetry(
  request: ContentRequest,
  maxRetries: number = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateArticle(request);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Missing Research Data

```typescript
// Fallback to cached or manual insights
export async function getInsightsWithFallback(
  keyword: string
): Promise<Insight[]> {
  try {
    const articles = await scrapeLatestNews(keyword);
    return await extractInsights(articles);
  } catch (error) {
    console.warn('Research failed, using fallback data');
    return getFallbackInsights(keyword);
  }
}
```

## Performance Optimization

```typescript
// Parallel processing for speed
async function generateMultipleArticles(
  keywords: string[]
): Promise<Article[]> {
  const articles = await Promise.all(
    keywords.map(keyword => 
      generateBilingualContent(keyword, [], 'toplist')
    )
  );
  
  return articles;
}

// Cache frequently used data
import NodeCache from 'node-cache';
const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

export async function getCachedInsights(keyword: string): Promise<Insight[]> {
  const cached = cache.get<Insight[]>(keyword);
  if (cached) return cached;
  
  const insights = await scrapeAndExtract(keyword);
  cache.set(keyword, insights);
  return insights;
}
```

This skill enables AI agents to help developers build, configure, and extend automated content marketing pipelines with research, AI generation, and video rendering capabilities.
