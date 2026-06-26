---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline for marketing
  - generate videos from blog posts automatically
  - research and create content with Claude AI
  - build automated content workflow with Remotion
  - crawl news sources for content ideas
  - create multilingual content with AI
  - automate social media video generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is an end-to-end automated content generation system that:
- **Crawls** real-time data from news sources (TechCrunch, a16z, Twitter, LinkedIn)
- **Generates** multilingual content (English/Vietnamese) using Claude 3 or OpenAI
- **Renders** videos and infographics automatically using Remotion
- **Supports** multiple content formats: Toplist, POV, Case Study, How-to

Built with TypeScript, Next.js, and integrated with powerful AI APIs for a complete content automation workflow.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Content Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI service integrations
│   ├── scraper/           # Content crawling logic
│   └── video/             # Remotion video rendering
├── remotion/              # Remotion video templates
└── public/                # Static assets
```

## Core Workflows

### 1. Content Research & Crawling

```typescript
// lib/scraper/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  source: string;
}

export async function crawlNewsSource(keyword: string, sources: NewsSource[]) {
  const articles = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: {
          q: keyword,
          timeRange: '24h'
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
        }
      });
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source.source}:`, error);
    }
  }
  
  return articles;
}

// Usage in API route
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const keyword = searchParams.get('keyword');
  
  const sources = [
    { url: 'https://api.techcrunch.com/search', selector: 'article', source: 'TechCrunch' },
    { url: 'https://api.twitter.com/2/tweets/search', selector: 'tweet', source: 'Twitter' }
  ];
  
  const articles = await crawlNewsSource(keyword, sources);
  
  return Response.json({ articles, count: articles.length });
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ToneOfVoice = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerateContentParams {
  keyword: string;
  researchData: any[];
  format: ContentFormat;
  tone: ToneOfVoice;
  language: Language;
  provider?: 'claude' | 'openai';
}

export async function generateContent({
  keyword,
  researchData,
  format,
  tone,
  language,
  provider = 'claude'
}: GenerateContentParams) {
  const prompt = buildPrompt(keyword, researchData, format, tone, language);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].text;
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7
    });
    
    return completion.choices[0].message.content;
  }
}

function buildPrompt(
  keyword: string,
  researchData: any[],
  format: ContentFormat,
  tone: ToneOfVoice,
  language: Language
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective or angle on this topic',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step instructions with actionable tips'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Write in a conversational, approachable style',
    'humorous': 'Include wit and light humor while maintaining informativeness'
  };
  
  const languageInstructions = {
    'en': 'Write in English',
    'vi': 'Viết bằng tiếng Việt'
  };
  
  return `
You are an expert content creator. Create a ${format} article about "${keyword}".

Research Data:
${JSON.stringify(researchData, null, 2)}

Format: ${formatInstructions[format]}
Tone: ${toneInstructions[tone]}
Language: ${languageInstructions[language]}

Requirements:
- Use data and insights from the research provided
- Include statistics and examples
- Make it engaging and actionable
- Optimize for SEO
- Length: 1500-2000 words
`;
}
```

### 3. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  backgroundColor?: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ 
  title, 
  points,
  backgroundColor = '#1a1a1a' 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity
          }}
        >
          <h1 style={{ 
            fontSize: 80, 
            color: 'white',
            textAlign: 'center',
            padding: '0 100px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence 
          key={index}
          from={90 + index * 120} 
          durationInFrames={120}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '0 150px'
            }}
          >
            <div style={{
              fontSize: 50,
              color: 'white',
              lineHeight: 1.6
            }}>
              <span style={{ 
                color: '#00d4ff',
                fontSize: 70,
                fontWeight: 'bold'
              }}>
                {index + 1}.
              </span>{' '}
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  title: string,
  points: string[],
  outputPath: string
) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { title, points }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { title, points }
  });
  
  return outputPath;
}
```

### 4. Complete Pipeline Integration

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNewsSource } from '@/lib/scraper/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { extractKeyPoints } from '@/lib/utils/content-parser';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, language, includeVideo } = await request.json();
    
    // Step 1: Research
    console.log('📡 Crawling news sources...');
    const researchData = await crawlNewsSource(keyword, [
      { url: 'https://api.techcrunch.com/search', selector: 'article', source: 'TechCrunch' }
    ]);
    
    // Step 2: Generate Content
    console.log('🧠 Generating content with AI...');
    const content = await generateContent({
      keyword,
      researchData,
      format,
      tone,
      language,
      provider: 'claude'
    });
    
    let videoPath = null;
    
    // Step 3: Generate Video (optional)
    if (includeVideo) {
      console.log('🎬 Rendering video...');
      const keyPoints = extractKeyPoints(content);
      videoPath = `/videos/${Date.now()}.mp4`;
      
      await renderContentVideo(
        keyword,
        keyPoints,
        path.join(process.cwd(), 'public', videoPath)
      );
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoPath,
      researchCount: researchData.length
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Generate Multi-Language Content

```typescript
// Generate both English and Vietnamese versions
const languages = ['en', 'vi'] as const;

const contents = await Promise.all(
  languages.map(lang => 
    generateContent({
      keyword: 'AI Marketing Trends 2024',
      researchData,
      format: 'toplist',
      tone: 'expert',
      language: lang
    })
  )
);

const [englishContent, vietnameseContent] = contents;
```

### Batch Video Generation

```typescript
// Generate videos for multiple social platforms
const platforms = [
  { name: 'reels', width: 1080, height: 1920 },
  { name: 'youtube', width: 1920, height: 1080 },
  { name: 'tiktok', width: 1080, height: 1920 }
];

for (const platform of platforms) {
  await renderContentVideo(
    title,
    points,
    `/videos/${platform.name}_${Date.now()}.mp4`,
    { width: platform.width, height: platform.height }
  );
}
```

### Schedule Content Pipeline

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run daily at 6 AM
  cron.schedule('0 6 * * *', async () => {
    console.log('🤖 Running scheduled content generation...');
    
    for (const keyword of keywords) {
      const researchData = await crawlNewsSource(keyword, sources);
      
      const content = await generateContent({
        keyword,
        researchData,
        format: 'toplist',
        tone: 'expert',
        language: 'en'
      });
      
      // Save to database or CMS
      await saveContent({ keyword, content });
    }
  });
}
```

## Configuration

### AI Provider Selection

```typescript
// lib/config/ai-config.ts
export const aiConfig = {
  defaultProvider: process.env.DEFAULT_AI_PROVIDER || 'claude',
  fallbackProvider: 'openai',
  maxRetries: 3,
  timeout: 60000,
  models: {
    claude: 'claude-3-5-sonnet-20241022',
    openai: 'gpt-4-turbo-preview'
  }
};
```

### Remotion Video Settings

```typescript
// remotion/config.ts
export const videoConfig = {
  width: 1920,
  height: 1080,
  fps: 30,
  durationInFrames: 900, // 30 seconds
  codec: 'h264',
  quality: 80
};
```

## Troubleshooting

### AI API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(params: GenerateContentParams, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(params);
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

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  inputProps,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-dev-shm-usage']
  },
  concurrency: 2, // Reduce concurrent rendering
  enforceAudioTrack: false
});
```

### Failed Content Crawling

```typescript
// Add fallback data sources
async function crawlWithFallback(keyword: string) {
  const primarySources = [...];
  const fallbackSources = [...];
  
  try {
    const data = await crawlNewsSource(keyword, primarySources);
    if (data.length > 0) return data;
  } catch (error) {
    console.warn('Primary sources failed, trying fallback...');
  }
  
  return await crawlNewsSource(keyword, fallbackSources);
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a test video
npm run remotion:render
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Queue video rendering** for resource-intensive operations
3. **Monitor AI token usage** to control costs
4. **Version control prompts** for consistent output quality
5. **Implement content approval workflow** before publishing
