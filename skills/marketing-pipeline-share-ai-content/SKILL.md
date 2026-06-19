---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, script generation, video creation, and multi-platform publishing
triggers:
  - "set up automated content generation pipeline"
  - "create AI-powered marketing content workflow"
  - "generate videos from text automatically"
  - "crawl news and research for content ideas"
  - "build content automation with Claude and OpenAI"
  - "render marketing videos with Remotion"
  - "automate content creation from research to publishing"
  - "generate multilingual content with AI"
---

# Marketing Pipeline Share AI Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based system automates the entire content creation pipeline: from news research and data crawling, through AI-powered content generation (Claude/OpenAI), to automatic video rendering (Remotion) and multi-platform publishing.

## What This Project Does

**Ultimate AI Content Pipeline** is an all-in-one content automation system that:

- **Auto-scans research**: Crawls fresh data from TechCrunch, a16z, X (Twitter), LinkedIn within the last 24 hours
- **AI content generation**: Creates diverse content formats (toplist, POV, case study, how-to) in multiple languages using Claude 3 and OpenAI
- **Video rendering**: Automatically converts text content into videos and infographics using Remotion
- **Multi-platform optimization**: Exports content optimized for Reels, TikTok, Shorts, and other platforms

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
cp .env.example .env
```

### Required Environment Variables

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom configurations
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Video rendering (Remotion)
npm run render
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Video templates
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── .env                 # Environment configuration
```

## Core Components

### 1. Research & Crawling

The system crawls multiple news sources for fresh content:

```typescript
// lib/crawler/newsCrawler.ts
import { fetchFromRapidAPI } from './api';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary: string;
}

export async function crawlNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const results = await fetchFromRapidAPI({
        endpoint: `/news/${source}`,
        params: {
          q: keyword,
          sortBy: 'publishedAt',
          from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
        }
      });
      
      articles.push(...results.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return articles;
}

// lib/crawler/api.ts
export async function fetchFromRapidAPI(options: {
  endpoint: string;
  params: Record<string, any>;
}) {
  const response = await fetch(
    `https://api.rapidapi.com${options.endpoint}?${new URLSearchParams(options.params)}`,
    {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'X-RapidAPI-Host': 'news-api.rapidapi.com'
      }
    }
  );
  
  return response.json();
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  keyword: string;
  research: string[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  provider?: 'claude' | 'openai';
}

export async function generateContent(options: GenerateContentOptions) {
  const {
    keyword,
    research,
    format,
    language,
    tone,
    provider = 'claude'
  } = options;
  
  const prompt = buildPrompt({ keyword, research, format, language, tone });
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content || '';
}

function buildPrompt(options: {
  keyword: string;
  research: string[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
}): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step instructions with actionable tips'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Write in a conversational, approachable style',
    'humorous': 'Include wit and humor while maintaining informativeness'
  };
  
  return `
You are a content creator specializing in ${options.keyword}.

Research data from the last 24 hours:
${options.research.join('\n\n')}

Task: Create a ${options.format} article about "${options.keyword}"
Language: ${options.language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${toneInstructions[options.tone]}

Format instructions: ${formatInstructions[options.format]}

Requirements:
- Include specific data and insights from the research
- Make it engaging and actionable
- Add relevant statistics and examples
- Structure with clear headings and subheadings
- Include a compelling introduction and conclusion

Output the complete article in markdown format.
`;
}
```

### 3. Multilingual Content Pipeline

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/content/multilingualPipeline.ts
import { generateContent, ContentFormat, Tone } from '../ai/contentGenerator';
import { crawlNews } from '../crawler/newsCrawler';

interface ContentOutput {
  english: string;
  vietnamese: string;
  metadata: {
    keyword: string;
    format: ContentFormat;
    createdAt: string;
    sources: number;
  };
}

export async function createMultilingualContent(
  keyword: string,
  format: ContentFormat,
  tone: Tone = 'expert'
): Promise<ContentOutput> {
  // Step 1: Crawl fresh research
  console.log(`Crawling news for: ${keyword}`);
  const articles = await crawlNews(keyword);
  const research = articles.map(a => `${a.title}\n${a.summary}\nSource: ${a.source}`);
  
  // Step 2: Generate English version
  console.log('Generating English content...');
  const englishContent = await generateContent({
    keyword,
    research,
    format,
    language: 'en',
    tone,
    provider: 'claude'
  });
  
  // Step 3: Generate Vietnamese version
  console.log('Generating Vietnamese content...');
  const vietnameseContent = await generateContent({
    keyword,
    research,
    format,
    language: 'vi',
    tone,
    provider: 'claude'
  });
  
  return {
    english: englishContent,
    vietnamese: vietnameseContent,
    metadata: {
      keyword,
      format,
      createdAt: new Date().toISOString(),
      sources: articles.length
    }
  };
}
```

### 4. Video Rendering with Remotion

Transform text content into videos:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundColor?: string;
  textColor?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  backgroundColor = '#1a1a1a',
  textColor = '#ffffff'
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  const pointsPerFrame = Math.floor(durationInFrames / points.length);
  
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <div style={{ padding: 60 }}>
        <h1
          style={{
            fontSize: 72,
            fontWeight: 'bold',
            color: textColor,
            opacity: titleOpacity,
            marginBottom: 60
          }}
        >
          {title}
        </h1>
        
        <div style={{ fontSize: 36, color: textColor }}>
          {points.map((point, index) => {
            const startFrame = 30 + index * pointsPerFrame;
            const opacity = interpolate(
              frame,
              [startFrame, startFrame + 20],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div
                key={index}
                style={{
                  opacity,
                  marginBottom: 30,
                  transform: `translateY(${interpolate(
                    frame,
                    [startFrame, startFrame + 20],
                    [20, 0],
                    { extrapolateRight: 'clamp' }
                  )}px)`
                }}
              >
                <strong>{index + 1}.</strong> {point}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};

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
    path.join(process.cwd(), 'remotion/index.ts')
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
  
  console.log(`Video rendered: ${outputPath}`);
}
```

### 5. Complete Pipeline Execution

```typescript
// lib/pipeline/executor.ts
import { createMultilingualContent } from '../content/multilingualPipeline';
import { renderContentVideo } from '../video/renderer';
import { extractKeyPoints } from '../content/parser';
import path from 'path';

export async function executeContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  console.log(`🚀 Starting content pipeline for: ${keyword}`);
  
  // Step 1: Generate multilingual content
  const content = await createMultilingualContent(keyword, format);
  
  console.log('✅ Content generated in both languages');
  
  // Step 2: Extract key points for video
  const keyPoints = extractKeyPoints(content.english);
  
  // Step 3: Render video
  const videoPath = path.join(
    process.cwd(),
    'output',
    `${keyword.replace(/\s+/g, '-')}-${Date.now()}.mp4`
  );
  
  await renderContentVideo(keyword, keyPoints, videoPath);
  
  console.log('✅ Video rendered successfully');
  
  return {
    content,
    videoPath,
    summary: {
      keyword,
      format,
      sourcesUsed: content.metadata.sources,
      englishWordCount: content.english.split(/\s+/).length,
      vietnameseWordCount: content.vietnamese.split(/\s+/).length,
      videoPath
    }
  };
}

// Usage in API route or CLI
export async function runPipeline() {
  const result = await executeContentPipeline(
    'AI Marketing Automation',
    'toplist'
  );
  
  console.log('Pipeline complete:', result.summary);
  return result;
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { executeContentPipeline } from '@/lib/pipeline/executor';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format } = await request.json();
    
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing keyword or format' },
        { status: 400 }
      );
    }
    
    const result = await executeContentPipeline(keyword, format);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}

// app/api/crawl/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/newsCrawler';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const keyword = searchParams.get('keyword');
  
  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword required' },
      { status: 400 }
    );
  }
  
  const articles = await crawlNews(keyword);
  
  return NextResponse.json({
    keyword,
    count: articles.length,
    articles
  });
}
```

## CLI Usage

```typescript
// scripts/generate.ts
import { executeContentPipeline } from '../lib/pipeline/executor';

async function main() {
  const args = process.argv.slice(2);
  const keyword = args[0];
  const format = args[1] as any || 'toplist';
  
  if (!keyword) {
    console.error('Usage: npm run generate <keyword> [format]');
    process.exit(1);
  }
  
  await executeContentPipeline(keyword, format);
}

main();
```

```bash
# Run from command line
npm run generate "AI Content Marketing" toplist
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await executeContentPipeline(keyword, 'toplist');
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// Create custom Remotion compositions
export const ShortFormVideo: React.FC<{text: string}> = ({text}) => {
  // 9:16 aspect ratio for TikTok/Reels
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Custom short-form content */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

**API Rate Limits:**
- Implement retry logic with exponential backoff
- Use caching for research data
- Batch API calls where possible

**Video Rendering Fails:**
- Ensure ffmpeg is installed: `brew install ffmpeg` or `apt-get install ffmpeg`
- Check memory limits for large renders
- Reduce video resolution if needed

**Content Quality Issues:**
- Adjust prompts in `buildPrompt()` function
- Provide more specific research data
- Experiment with different AI models (Claude vs OpenAI)

**Environment Setup:**
- Verify all API keys are set correctly
- Check API quota/limits for each service
- Ensure Node.js version >= 18
