---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion for multi-format content creation
triggers:
  - automate content creation pipeline
  - generate video from content with ai
  - auto research and write articles
  - create multi-format content with claude
  - build automated marketing pipeline
  - scrape news and generate content
  - render videos from text automatically
  - setup ai content automation system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete AI-powered content automation system that handles research, scriptwriting, multi-format content generation, and automatic video rendering. Leverages Claude 3, OpenAI, and Remotion to transform keywords into publication-ready content.

## What This Project Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto Research**: Crawls recent news from TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in English and Vietnamese
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Platform Optimization**: Outputs content optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── scraper/     # Web scraping modules
│   │   ├── renderer/    # Remotion video rendering
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core Modules

### 1. Research & Scraping

```typescript
// src/lib/scraper/news-aggregator.ts
import axios from 'axios';

interface NewsSource {
  title: string;
  content: string;
  url: string;
  publishedAt: string;
  source: string;
}

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/search`,
        {
          params: {
            q: keyword,
            source: source,
            timeRange: '24h'
          },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
            'X-RapidAPI-Host': 'news-api.rapidapi.com'
          }
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Error scraping ${source}:`, error);
    }
  }
  
  return results;
}

export async function extractInsights(articles: NewsSource[]): Promise<string> {
  const content = articles.map(a => `${a.title}\n${a.content}`).join('\n\n');
  
  return `
    Found ${articles.length} recent articles.
    Key topics: ${extractKeywords(content)}
    Latest trends: ${analyzeTrends(content)}
  `;
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: string;
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const { keyword, format, language, tone, researchData } = request;
  
  const prompt = buildPrompt(format, language, tone, keyword, researchData);
  
  // Use Claude for long-form content
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(
  format: ContentFormat,
  language: Language,
  tone: Tone,
  keyword: string,
  researchData: string
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write from a specific perspective or opinion',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step instructions'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terms',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and light humor while staying informative'
  };
  
  return `
You are an expert content writer. Create a ${format} article about "${keyword}" in ${language === 'en' ? 'English' : 'Vietnamese'}.

Tone: ${toneInstructions[tone]}
Format: ${formatInstructions[format]}

Research Data:
${researchData}

Requirements:
- Use recent data and insights from the research
- Include specific examples and statistics
- Structure with clear headings and subheadings
- Optimize for readability and engagement
- Length: 1500-2000 words
`;
}
```

### 3. Dual Language Generation

```typescript
// src/lib/ai/multilingual-generator.ts
interface BilingualContent {
  english: string;
  vietnamese: string;
}

export async function generateBilingualContent(
  keyword: string,
  format: ContentFormat,
  tone: Tone,
  researchData: string
): Promise<BilingualContent> {
  // Generate English version first
  const english = await generateContent({
    keyword,
    format,
    language: 'en',
    tone,
    researchData
  });
  
  // Generate Vietnamese version (not just translation)
  const vietnamese = await generateContent({
    keyword,
    format,
    language: 'vi',
    tone,
    researchData
  });
  
  return { english, vietnamese };
}
```

### 4. Video Rendering with Remotion

```typescript
// src/lib/renderer/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'infographic' | 'text-animated' | 'slideshow';
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export async function generateVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = `${config.style}-${config.aspectRatio}`;
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });
  
  // Render video
  const outputLocation = `./public/videos/${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });
  
  return outputLocation;
}
```

### 5. Remotion Composition Example

```typescript
// remotion/compositions/InfographicVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

export const InfographicVideo: React.FC<{
  title: string;
  content: string[];
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity,
          }}
        >
          <h1 style={{ 
            color: 'white', 
            fontSize: '80px',
            textAlign: 'center',
            padding: '0 100px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Sequences */}
      {content.map((text, index) => (
        <Sequence key={index} from={60 + index * 90} durationInFrames={90}>
          <ContentSlide text={text} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const ContentSlide: React.FC<{ text: string; index: number }> = ({
  text,
  index,
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);
  const scale = interpolate(frame, [0, 15], [0.8, 1]);
  
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
        transform: `scale(${scale})`,
      }}
    >
      <div style={{ padding: '0 150px', color: 'white' }}>
        <h2 style={{ fontSize: '60px', marginBottom: '30px' }}>
          {index}
        </h2>
        <p style={{ fontSize: '40px', lineHeight: 1.5 }}>
          {text}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/content-pipeline.ts
interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: Tone;
  generateVideo: boolean;
  videoStyle?: 'infographic' | 'text-animated' | 'slideshow';
  aspectRatio?: '9:16' | '16:9' | '1:1';
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  research: string;
  content: BilingualContent;
  videoUrl?: string;
}> {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);
  
  // Step 1: Research
  console.log('📡 Gathering research...');
  const articles = await scrapeRecentNews(config.keyword);
  const research = await extractInsights(articles);
  
  // Step 2: Generate Content
  console.log('🧠 Generating content...');
  const content = await generateBilingualContent(
    config.keyword,
    config.format,
    config.tone,
    research
  );
  
  // Step 3: Generate Video (optional)
  let videoUrl: string | undefined;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    
    // Extract key points for video
    const keyPoints = extractKeyPoints(content.english);
    
    videoUrl = await generateVideo({
      title: config.keyword,
      content: keyPoints,
      style: config.videoStyle || 'infographic',
      aspectRatio: config.aspectRatio || '9:16',
    });
  }
  
  console.log('✅ Pipeline completed!');
  
  return { research, content, videoUrl };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - split by headings or paragraphs
  const points = content
    .split('\n')
    .filter(line => line.trim().length > 50 && line.trim().length < 200)
    .slice(0, 5);
  
  return points;
}
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, tone, generateVideo, videoStyle, aspectRatio } = body;
    
    // Validate input
    if (!keyword || !format || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      tone,
      generateVideo: generateVideo || false,
      videoStyle,
      aspectRatio,
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

async function createArticle() {
  const result = await runContentPipeline({
    keyword: 'AI in Marketing 2024',
    format: 'toplist',
    tone: 'expert',
    generateVideo: false,
  });
  
  console.log('English version:', result.content.english);
  console.log('Vietnamese version:', result.content.vietnamese);
}
```

### Generate Content with Video

```typescript
async function createContentWithVideo() {
  const result = await runContentPipeline({
    keyword: 'Social Media Trends',
    format: 'how-to',
    tone: 'friendly',
    generateVideo: true,
    videoStyle: 'infographic',
    aspectRatio: '9:16', // For TikTok/Reels
  });
  
  console.log('Video URL:', result.videoUrl);
}
```

### Custom Research Sources

```typescript
import { scrapeRecentNews, extractInsights } from '@/lib/scraper/news-aggregator';
import { generateContent } from '@/lib/ai/content-generator';

async function customResearch() {
  // Scrape from specific sources
  const articles = await scrapeRecentNews('startup funding', [
    'techcrunch',
    'venturebeat',
    'a16z'
  ]);
  
  const insights = await extractInsights(articles);
  
  // Generate content
  const content = await generateContent({
    keyword: 'Startup Funding Trends',
    format: 'case-study',
    language: 'en',
    tone: 'expert',
    researchData: insights,
  });
  
  return content;
}
```

## Running the Application

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video locally
npx remotion render src/index.ts InfographicVideo out/video.mp4
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'friendly',
        generateVideo: true,
      })
    )
  );
  
  return results;
}
```

### Content Scheduling

```typescript
interface ScheduledContent {
  content: BilingualContent;
  publishAt: Date;
  platforms: ('facebook' | 'twitter' | 'linkedin')[];
}

async function scheduleContent(
  keyword: string,
  publishAt: Date
): Promise<ScheduledContent> {
  const result = await runContentPipeline({
    keyword,
    format: 'pov',
    tone: 'expert',
    generateVideo: false,
  });
  
  // Save to database with schedule
  return {
    content: result.content,
    publishAt,
    platforms: ['facebook', 'linkedin'],
  };
}
```

### Video Optimization for Platforms

```typescript
async function generatePlatformVideos(keyword: string, content: string[]) {
  const platforms = [
    { name: 'TikTok', ratio: '9:16' as const },
    { name: 'YouTube', ratio: '16:9' as const },
    { name: 'Instagram', ratio: '1:1' as const },
  ];
  
  const videos = await Promise.all(
    platforms.map(platform =>
      generateVideo({
        title: keyword,
        content,
        style: 'infographic',
        aspectRatio: platform.ratio,
      })
    )
  );
  
  return videos.map((url, i) => ({
    platform: platforms[i].name,
    videoUrl: url,
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function rateLimitedGeneration(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword =>
      limit(() => runContentPipeline({ keyword, format: 'toplist', tone: 'friendly', generateVideo: false }))
    )
  );
}
```

### Remotion Rendering Issues

```bash
# Install required dependencies for video rendering
# On Ubuntu/Debian:
sudo apt-get install -y chromium-browser

# On macOS:
brew install chromium

# Set Chromium path in environment
export PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
async function generateLargeContent(keyword: string) {
  // Split into smaller sections
  const sections = ['introduction', 'main-points', 'conclusion'];
  
  const contents = await Promise.all(
    sections.map(section =>
      generateContent({
        keyword: `${keyword} - ${section}`,
        format: 'how-to',
        language: 'en',
        tone: 'expert',
        researchData: '',
      })
    )
  );
  
  return contents.join('\n\n');
}
```

### Error Handling

```typescript
async function robustPipeline(config: PipelineConfig) {
  try {
    return await runContentPipeline(config);
  } catch (error) {
    if (error.message.includes('rate limit')) {
      console.log('Rate limited, waiting 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return robustPipeline(config);
    }
    
    if (error.message.includes('API key')) {
      throw new Error('Invalid API credentials. Check your .env.local file');
    }
    
    throw error;
  }
}
```

## Best Practices

1. **API Key Management**: Always use environment variables, never hardcode
2. **Rate Limiting**: Implement delays between API calls to avoid rate limits
3. **Error Handling**: Wrap all API calls in try-catch blocks
4. **Caching**: Cache research results to avoid redundant scraping
5. **Video Optimization**: Use appropriate aspect ratios for each platform
6. **Content Quality**: Always review AI-generated content before publishing
7. **Monitoring**: Log pipeline execution for debugging and optimization
