---
name: ultimate-ai-content-pipeline
description: Complete AI-powered content automation pipeline for research, scriptwriting, and video generation with Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from written content automatically
  - scrape news and create content from latest trends
  - build automated content pipeline with Claude
  - create social media videos with Remotion
  - set up AI content marketing automation
  - generate multilingual content with AI
  - automate research to video content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

The Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that transforms keywords into fully-realized content pieces. It crawls trending news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates articles in multiple formats and languages using Claude 3 and OpenAI, and automatically renders videos using Remotion.

## What It Does

This pipeline automates the entire content creation workflow:

1. **Research Phase**: Automatically scrapes and analyzes recent data from major news sources
2. **Content Generation**: Creates articles in various formats (listicles, POV pieces, case studies, how-tos) in both English and Vietnamese
3. **Video Rendering**: Converts written content into videos and infographics optimized for Reels, TikTok, and Shorts
4. **Multi-platform Output**: Generates content ready for immediate publishing across social media platforms

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

Create a `.env.local` file in the project root with the following environment variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Key Architecture

The system typically follows this structure:

```
/app
  /api
    /research      # Auto-scan research endpoints
    /generate      # Content generation endpoints
    /video         # Video rendering endpoints
/lib
  /ai              # Claude/OpenAI integrations
  /scrapers        # Web scraping utilities
  /remotion        # Video composition templates
/components        # React UI components
```

## Core API Endpoints

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  // Scrape recent data
  const scrapedData = await scrapeMultipleSources(sources, keyword);
  
  // Analyze with Claude
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Analyze these recent articles about ${keyword} and extract key insights:\n${JSON.stringify(scrapedData)}`
    }]
  });

  return NextResponse.json({
    insights: message.content,
    sources: scrapedData
  });
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import OpenAI from 'openai';
import Anthropic from '@anthropic-ai/sdk';

export async function POST(req: NextRequest) {
  const { insights, format, language, tone } = await req.json();
  
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const prompt = buildPrompt(insights, format, language, tone);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are an expert content writer.' },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
  });

  const content = completion.choices[0].message.content;

  return NextResponse.json({
    content,
    format,
    language
  });
}

function buildPrompt(insights: string, format: string, language: string, tone: string): string {
  const formatInstructions = {
    toplist: 'Create a numbered list article with clear sections',
    pov: 'Write from a unique perspective with personal insights',
    casestudy: 'Structure as a detailed case study with problem-solution-results',
    howto: 'Create a step-by-step tutorial format'
  };

  return `
Based on these insights: ${insights}

Create a ${format} article in ${language} with a ${tone} tone.
Format guidelines: ${formatInstructions[format] || formatInstructions.toplist}

Include:
- Engaging headline
- Clear structure with subheadings
- Data-backed arguments
- Actionable takeaways
`;
}
```

### Video Rendering with Remotion

```typescript
// lib/remotion/compositions.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: '',
          content: '',
          bgColor: '#1a1a1a',
        }}
      />
    </>
  );
};
```

```typescript
// lib/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
  bgColor: string;
}> = ({ title, content, bgColor }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        opacity, 
        padding: 60,
        color: 'white',
        fontFamily: 'Inter, sans-serif'
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>{title}</h1>
        <p style={{ fontSize: 36, lineHeight: 1.6 }}>{content}</p>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(req: NextRequest) {
  const { title, content, format } = await req.json();

  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'lib/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { title, content, bgColor: '#1a1a1a' },
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: { title, content, bgColor: '#1a1a1a' },
  });

  return NextResponse.json({
    videoUrl: `/videos/${path.basename(outputLocation)}`
  });
}
```

## Web Scraping Implementation

```typescript
// lib/scrapers/news-scraper.ts
import axios from 'axios';

interface ScraperSource {
  name: string;
  url: string;
  apiEndpoint?: string;
}

export async function scrapeMultipleSources(
  sources: ScraperSource[],
  keyword: string
): Promise<any[]> {
  const results = await Promise.all(
    sources.map(source => scrapeSingleSource(source, keyword))
  );
  
  return results.flat();
}

async function scrapeSingleSource(
  source: ScraperSource,
  keyword: string
): Promise<any[]> {
  try {
    const response = await axios.get(source.apiEndpoint || source.url, {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': 'google-news13.p.rapidapi.com'
      },
      params: {
        query: keyword,
        language: 'en',
        time: '24h'
      }
    });

    return response.data.items || [];
  } catch (error) {
    console.error(`Failed to scrape ${source.name}:`, error);
    return [];
  }
}
```

## Complete Pipeline Usage Example

```typescript
// app/actions/create-content-pipeline.ts
'use server';

import { scrapeMultipleSources } from '@/lib/scrapers/news-scraper';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export async function runContentPipeline(keyword: string) {
  // Step 1: Research Phase
  const sources = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', apiEndpoint: '...' },
    { name: 'a16z', url: 'https://a16z.com', apiEndpoint: '...' },
  ];
  
  const scrapedData = await scrapeMultipleSources(sources, keyword);

  // Step 2: Extract Insights with Claude
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const insights = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Extract key insights from: ${JSON.stringify(scrapedData)}`
    }]
  });

  // Step 3: Generate Content with OpenAI
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const article = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are an expert content writer.' },
      { 
        role: 'user', 
        content: `Write a toplist article about ${keyword} based on: ${insights.content[0].text}` 
      }
    ],
  });

  // Step 4: Generate Video
  const videoResponse = await fetch('/api/video/render', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      title: `Top Trends: ${keyword}`,
      content: article.choices[0].message.content?.substring(0, 200),
    })
  });

  const { videoUrl } = await videoResponse.json();

  return {
    article: article.choices[0].message.content,
    insights: insights.content[0].text,
    videoUrl
  };
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Start Remotion Studio (for video preview)
npm run remotion:studio

# Render a video composition
npm run remotion:render
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateMultilingualContent(insights: string) {
  const languages = ['en', 'vi'];
  
  const content = await Promise.all(
    languages.map(async (lang) => {
      const response = await openai.chat.completions.create({
        model: 'gpt-4-turbo-preview',
        messages: [
          { 
            role: 'user', 
            content: `Write in ${lang}: ${insights}` 
          }
        ],
      });
      
      return {
        language: lang,
        content: response.choices[0].message.content
      };
    })
  );
  
  return content;
}
```

### Format-Specific Templates

```typescript
const contentFormats = {
  toplist: {
    structure: ['Introduction', 'Item 1-5', 'Conclusion'],
    prompt: 'Create a numbered list with clear benefits for each item'
  },
  pov: {
    structure: ['Hook', 'Personal Angle', 'Arguments', 'Call to Action'],
    prompt: 'Write from a unique perspective with personal insights'
  },
  casestudy: {
    structure: ['Background', 'Challenge', 'Solution', 'Results', 'Takeaways'],
    prompt: 'Detail a real-world example with metrics'
  },
  howto: {
    structure: ['Overview', 'Prerequisites', 'Steps 1-N', 'Tips', 'Conclusion'],
    prompt: 'Provide actionable step-by-step instructions'
  }
};
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```typescript
// lib/utils/rate-limiter.ts
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

For large video renders, increase Node.js memory:

```json
// package.json
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Scraping Failures

Implement fallback sources:

```typescript
async function scrapeSingleSource(source: ScraperSource, keyword: string) {
  try {
    // Primary scraping method
    return await primaryScrape(source, keyword);
  } catch (error) {
    // Fallback to alternative method
    return await fallbackScrape(source, keyword);
  }
}
```

### Claude/OpenAI Context Length

For long research data, chunk the content:

```typescript
function chunkContent(content: string, maxTokens = 3000): string[] {
  const chunks = [];
  const words = content.split(' ');
  let currentChunk = '';
  
  for (const word of words) {
    if ((currentChunk + word).length > maxTokens * 4) {
      chunks.push(currentChunk);
      currentChunk = word;
    } else {
      currentChunk += ' ' + word;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```
