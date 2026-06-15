---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - generate content with AI pipeline
  - automate content research and writing
  - create video from text content
  - set up AI content automation
  - crawl news and generate articles
  - build content marketing pipeline
  - automate social media content creation
  - research and write with Claude AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end automated content creation system that performs research by crawling news sources, generates articles in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion. Built with TypeScript and Next.js.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Scan Research**: Crawls recent articles from TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (Top List, POV, Case Study, How-to) using Claude or OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Converts written content into videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

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

# Video rendering (Remotion)
npm run render
```

## Key Components & Usage

### 1. Content Research Module

The research module crawls and analyzes recent articles:

```typescript
import { researchContent } from '@/lib/research';

async function performResearch(keyword: string) {
  const researchData = await researchContent({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return researchData;
}
```

### 2. AI Content Generation with Claude

Generate articles using Claude API:

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  topic: string, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Write a ${format} article about ${topic} in ${language}. 
                Include data-backed insights and recent trends.`
    }]
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

Alternative using OpenAI:

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, tone: string = 'professional') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content writer with a ${tone} tone. Create engaging, data-driven content.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. News Crawling with RapidAPI

Fetch recent news articles:

```typescript
import axios from 'axios';

async function fetchNewsArticles(keyword: string) {
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/v2/search/articles',
    params: {
      query: keyword,
      language: 'en',
      time_published: '24h'
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };

  const response = await axios.request(options);
  return response.data.articles;
}
```

### 5. Video Rendering with Remotion

Create a Remotion composition:

```typescript
// remotion/Composition.tsx
import { Composition } from 'remotion';
import { ArticleVideo } from './ArticleVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ArticleVideo"
        component={ArticleVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Article Title',
          content: 'Article content...',
          format: 'reels'
        }}
      />
    </>
  );
};
```

```typescript
// remotion/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ArticleVideo: React.FC<{
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ color: 'white', fontSize: 48 }}>{title}</h1>
        <p style={{ color: '#ccc', fontSize: 24 }}>{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```

Render the video:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderArticleVideo(articleData: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: articleData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${articleData.title}.mp4`,
    inputProps: articleData,
  });
}
```

## Complete Pipeline Example

Full workflow from research to video:

```typescript
import { researchContent } from '@/lib/research';
import { generateArticle } from '@/lib/ai/claude';
import { renderArticleVideo } from '@/lib/video/render';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('Starting research...');
    const research = await researchContent({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h',
      maxResults: 10
    });

    // Step 2: Generate Content
    console.log('Generating article...');
    const articleEN = await generateArticle(
      keyword,
      'toplist',
      'en'
    );
    
    const articleVI = await generateArticle(
      keyword,
      'toplist',
      'vi'
    );

    // Step 3: Save to Database
    const savedArticle = await saveArticle({
      titleEN: extractTitle(articleEN),
      titleVI: extractTitle(articleVI),
      contentEN: articleEN,
      contentVI: articleVI,
      researchData: research,
      keyword
    });

    // Step 4: Render Video
    console.log('Rendering video...');
    await renderArticleVideo({
      title: savedArticle.titleEN,
      content: savedArticle.contentEN,
      format: 'reels'
    });

    console.log('Pipeline completed successfully!');
    return savedArticle;
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute
runContentPipeline('AI automation trends').then(result => {
  console.log('Article ID:', result.id);
});
```

## API Routes (Next.js)

### Generate Content API

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateArticle } from '@/lib/ai/claude';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, language } = req.body;

  try {
    const article = await generateArticle(keyword, format, language);
    
    res.status(200).json({
      success: true,
      article,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(500).json({ 
      success: false, 
      error: error.message 
    });
  }
}
```

### Research API

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { researchContent } from '@/lib/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword } = req.query;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword required' });
  }

  try {
    const data = await researchContent({
      keyword: keyword as string,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });
    
    res.status(200).json({ success: true, data });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Common Patterns

### Content Format Templates

```typescript
const CONTENT_FORMATS = {
  toplist: {
    structure: 'numbered list with explanations',
    tone: 'authoritative',
    length: 1500
  },
  pov: {
    structure: 'personal perspective with arguments',
    tone: 'conversational',
    length: 1200
  },
  caseStudy: {
    structure: 'problem-solution-results',
    tone: 'analytical',
    length: 2000
  },
  howTo: {
    structure: 'step-by-step guide',
    tone: 'instructional',
    length: 1800
  }
};

function buildPrompt(format: keyof typeof CONTENT_FORMATS, topic: string) {
  const config = CONTENT_FORMATS[format];
  return `Write a ${config.structure} article about ${topic}. 
          Use a ${config.tone} tone. Target length: ${config.length} words.`;
}
```

### Rate Limiting & Error Handling

```typescript
import { sleep } from '@/lib/utils';

async function generateWithRetry(
  prompt: string, 
  maxRetries: number = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const result = await generateArticle(prompt, 'toplist', 'en');
      return result;
    } catch (error) {
      if (error.status === 429) {
        // Rate limited - wait and retry
        const waitTime = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await sleep(waitTime);
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const article = await generateArticle(keyword, 'toplist', 'en');
      results.push({ keyword, article, success: true });
      
      // Respect rate limits
      await sleep(1000);
    } catch (error) {
      results.push({ keyword, error: error.message, success: false });
    }
  }
  
  return results;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
function checkAPIKeys() {
  const required = ['ANTHROPIC_API_KEY', 'OPENAI_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
}
```

### Remotion Rendering Errors

```bash
# If video rendering fails, check Remotion installation
npx remotion versions

# Clear Remotion cache
rm -rf node_modules/.cache/remotion

# Test render with simpler composition
npx remotion render src/index.ts ArticleVideo out/test.mp4
```

### Memory Issues with Large Batches

```typescript
// Process in smaller chunks
async function processInChunks<T>(
  items: T[], 
  chunkSize: number, 
  processor: (item: T) => Promise<any>
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    
    // Allow garbage collection
    await sleep(100);
  }
  
  return results;
}
```

### Claude API Token Limits

```typescript
// Chunk long content for Claude
function chunkContent(content: string, maxTokens: number = 3000) {
  const chunks = [];
  const sentences = content.split('. ');
  let currentChunk = '';
  
  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > maxTokens) {
      chunks.push(currentChunk);
      currentChunk = sentence;
    } else {
      currentChunk += sentence + '. ';
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

This skill enables AI agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research through video generation.
