---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude AI, OpenAI, and Remotion
triggers:
  - automate content creation with AI
  - generate videos from articles automatically
  - research and write marketing content
  - create social media content pipeline
  - build AI content automation system
  - scrape news and generate content
  - make automated video content
  - set up content marketing workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from web scraping and research, to AI-powered content generation, to automated video rendering. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into publishable content and videos.

## What It Does

This pipeline automates:
- **News & Data Scraping**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending content
- **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos)
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Converts written content into social media videos using Remotion
- **Platform Optimization**: Exports videos formatted for TikTok, Reels, YouTube Shorts

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

## Configuration

Create a `.env.local` file in the root directory:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database for content storage
DATABASE_URL=your_database_connection_string

# Remotion configuration
REMOTION_CODEC=h264
REMOTION_CRF=18
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Video rendering with Remotion
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Web Scraping & Research

```typescript
import { scrapeNewsArticles } from '@/lib/scraper/news-scraper';

// Scrape recent articles by keyword
async function researchTopic(keyword: string) {
  const articles = await scrapeNewsArticles({
    keyword: keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    limit: 10
  });
  
  return articles;
}

// Example usage
const aiArticles = await researchTopic('artificial intelligence');
console.log(`Found ${aiArticles.length} relevant articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  research: any[], 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(research, format, language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }],
    temperature: 0.7
  });
  
  return message.content[0].text;
}

function buildPrompt(research: any[], format: string, language: string) {
  const researchData = research.map(r => 
    `Title: ${r.title}\nSummary: ${r.summary}\nURL: ${r.url}`
  ).join('\n\n');
  
  return `Based on this research:\n${researchData}\n\n` +
    `Create a ${format} article in ${language === 'en' ? 'English' : 'Vietnamese'}. ` +
    `Include data-backed insights, quotes, and actionable takeaways.`;
}
```

### 3. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer creating engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.8,
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

async function generateVideo(articleContent: {
  title: string;
  points: string[];
  images: string[];
}) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: articleContent,
  });
  
  // Render video
  const outputLocation = path.join(process.cwd(), 'public/videos/output.mp4');
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: articleContent,
  });
  
  return outputLocation;
}
```

### 5. Complete Pipeline Example

```typescript
import { scrapeNewsArticles } from '@/lib/scraper/news-scraper';
import { generateArticle } from '@/lib/ai/claude-generator';
import { generateVideo } from '@/lib/video/remotion-renderer';
import { parseArticleForVideo } from '@/lib/content/parser';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await scrapeNewsArticles({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h',
      limit: 5
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating article...');
    const articleEN = await generateArticle(research, 'toplist', 'en');
    const articleVI = await generateArticle(research, 'toplist', 'vi');
    
    // Step 3: Parse for Video
    console.log('🎬 Preparing video content...');
    const videoContent = parseArticleForVideo(articleEN);
    
    // Step 4: Render Video
    console.log('🎥 Rendering video...');
    const videoPath = await generateVideo(videoContent);
    
    return {
      articles: {
        english: articleEN,
        vietnamese: articleVI
      },
      video: videoPath
    };
  } catch (error) {
    console.error('Pipeline failed:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI marketing trends')
  .then(result => {
    console.log('✅ Pipeline complete!');
    console.log('Video:', result.video);
  });
```

## Common Patterns

### Custom Content Templates

```typescript
interface ContentTemplate {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'casual' | 'humorous';
  wordCount: number;
  includeStats: boolean;
  includeCTA: boolean;
}

async function generateCustomContent(
  research: any[],
  template: ContentTemplate,
  language: 'en' | 'vi'
) {
  const systemPrompt = `You are creating a ${template.format} article with a ${template.tone} tone. 
Target length: ${template.wordCount} words.
${template.includeStats ? 'Include relevant statistics and data.' : ''}
${template.includeCTA ? 'End with a strong call-to-action.' : ''}`;

  const userPrompt = `Research data:\n${JSON.stringify(research, null, 2)}`;
  
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{ role: 'user', content: userPrompt }]
  });
  
  return message.content[0].text;
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const research = await scrapeNewsArticles({ keyword, limit: 5 });
    const content = await generateArticle(research, 'toplist', 'en');
    
    results.push({
      keyword,
      content,
      generatedAt: new Date()
    });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Video Template Customization

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: 'white', fontSize: 48 }}>{title}</h1>
      </Sequence>
      
      {points.map((point, i) => (
        <Sequence
          key={i}
          from={60 + i * 90}
          durationInFrames={90}
        >
          <div style={{ color: 'white', fontSize: 32 }}>
            {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the UI at http://localhost:3000
```

## API Routes (Next.js)

Create API endpoints for the pipeline:

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function callWithRetry(fn: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
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
}
```

### Video Rendering Errors

```typescript
// Check Remotion installation
import { getCompositions } from '@remotion/renderer';

async function validateRemotionSetup() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions.map(c => c.id));
  } catch (error) {
    console.error('Remotion setup error:', error);
    console.log('Run: npx remotion upgrade');
  }
}
```

### Memory Issues with Large Scrapes

```typescript
// Process in chunks
async function scrapeInChunks(keywords: string[], chunkSize = 5) {
  const results = [];
  
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(kw => scrapeNewsArticles({ keyword: kw }))
    );
    results.push(...chunkResults.flat());
    
    // Clear memory
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Performance Tips

- Use streaming for long-form content generation
- Cache research results to avoid redundant scraping
- Implement queue system for video rendering (use Bull or BullMQ)
- Use CDN for video delivery
- Enable incremental static regeneration for Next.js pages
