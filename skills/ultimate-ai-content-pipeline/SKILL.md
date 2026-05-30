---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, script generation, video rendering, and multi-platform publishing
triggers:
  - how do i use the ultimate ai content pipeline
  - automate content creation with ai research
  - generate videos from blog posts automatically
  - set up marketing content automation pipeline
  - create multilingual content with claude and openai
  - render videos with remotion for social media
  - build automated content workflow
  - scrape trending news and generate articles
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline**, a complete automation system for content creation that handles research, scriptwriting, multi-language article generation, and video rendering. The pipeline crawls real-time news, generates content using Claude/OpenAI, and creates social media videos using Remotion.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

- **Auto Research**: Crawls news sources (TechCrunch, Twitter, LinkedIn) for trending topics
- **AI Content Generation**: Creates articles in multiple formats (listicles, case studies, how-tos) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically creates social media videos and infographics using Remotion
- **Platform Optimization**: Outputs content optimized for Reels, TikTok, and Shorts

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
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key Components & Usage

### 1. Research & Content Scraping

```typescript
// src/lib/scraper/newsScanner.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  content: string;
  source: string;
  publishedAt: string;
  url: string;
}

export async function scanTrendingNews(
  topic: string,
  sources: string[] = ['techcrunch', 'twitter', 'linkedin']
): Promise<NewsArticle[]> {
  const headers = {
    'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
    'X-RapidAPI-Host': 'news-api.rapidapi.com'
  };

  const articles: NewsArticle[] = [];

  for (const source of sources) {
    try {
      const response = await axios.get(`https://news-api.rapidapi.com/search`, {
        headers,
        params: {
          q: topic,
          source: source,
          time: '24h'
        }
      });

      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Error fetching from ${source}:`, error);
    }
  }

  return articles;
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  topic: string;
  research: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  aiProvider?: 'claude' | 'openai';
}

export async function generateContent(options: GenerateContentOptions): Promise<string> {
  const { topic, research, format, language, tone, aiProvider = 'claude' } = options;

  const prompt = buildPrompt(topic, research, format, language, tone);

  if (aiProvider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-sonnet-20240229',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ]
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content writer specializing in marketing and tech content.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      max_tokens: 4096
    });

    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(
  topic: string,
  research: string,
  format: ContentFormat,
  language: Language,
  tone: Tone
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear headings',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and actionable insights',
    'how-to': 'Provide step-by-step instructions with clear outcomes'
  };

  const languageInstruction = language === 'vi' 
    ? 'Write in Vietnamese with natural, engaging language.'
    : 'Write in English with clear, professional language.';

  const toneInstruction = {
    'expert': 'Use authoritative, data-driven language.',
    'friendly': 'Use conversational, approachable language.',
    'humorous': 'Use light, entertaining language with appropriate humor.'
  };

  return `
Topic: ${topic}

Research Data:
${research}

Instructions:
- Format: ${formatInstructions[format]}
- Language: ${languageInstruction}
- Tone: ${toneInstruction[tone]}
- Include relevant statistics and data from the research
- Make it engaging and shareable
- Optimize for SEO with natural keyword integration

Please generate the complete article now.
  `.trim();
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoOptions {
  title: string;
  keyPoints: string[];
  statistics: Array<{ label: string; value: string }>;
  platform: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(options: VideoOptions): Promise<string> {
  const { title, keyPoints, statistics, platform } = options;

  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[platform];

  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      keyPoints,
      statistics
    },
  });

  // Output path
  const outputLocation = path.resolve(
    `./public/videos/${Date.now()}-${platform}.mp4`
  );

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      keyPoints,
      statistics
    },
  });

  return outputLocation;
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  statistics: Array<{ label: string; value: string }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  statistics
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60,
            opacity: titleOpacity,
          }}
        >
          <h1
            style={{
              fontSize: 72,
              fontWeight: 'bold',
              color: '#ffffff',
              textAlign: 'center',
              lineHeight: 1.2,
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key Points Sequence */}
      {keyPoints.map((point, index) => (
        <Sequence key={index} from={90 + index * 90} durationInFrames={90}>
          <KeyPointSlide point={point} index={index + 1} />
        </Sequence>
      ))}

      {/* Statistics Sequence */}
      <Sequence from={90 + keyPoints.length * 90} durationInFrames={120}>
        <StatisticsSlide statistics={statistics} />
      </Sequence>
    </AbsoluteFill>
  );
};

const KeyPointSlide: React.FC<{ point: string; index: number }> = ({ point, index }) => {
  const frame = useCurrentFrame();
  const scale = interpolate(frame, [0, 20], [0.8, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        padding: 80,
      }}
    >
      <div style={{ transform: `scale(${scale})` }}>
        <div
          style={{
            fontSize: 48,
            color: '#00d9ff',
            fontWeight: 'bold',
            marginBottom: 30,
          }}
        >
          Point {index}
        </div>
        <p
          style={{
            fontSize: 56,
            color: '#ffffff',
            textAlign: 'center',
            lineHeight: 1.4,
          }}
        >
          {point}
        </p>
      </div>
    </AbsoluteFill>
  );
};

const StatisticsSlide: React.FC<{ statistics: Array<{ label: string; value: string }> }> = ({
  statistics,
}) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div style={{ width: '100%' }}>
        {statistics.map((stat, index) => {
          const opacity = interpolate(
            frame,
            [index * 15, index * 15 + 15],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );

          return (
            <div
              key={index}
              style={{
                marginBottom: 60,
                opacity,
              }}
            >
              <div
                style={{
                  fontSize: 96,
                  fontWeight: 'bold',
                  color: '#00d9ff',
                  textAlign: 'center',
                }}
              >
                {stat.value}
              </div>
              <div
                style={{
                  fontSize: 40,
                  color: '#ffffff',
                  textAlign: 'center',
                  marginTop: 20,
                }}
              >
                {stat.label}
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/contentPipeline.ts
import { scanTrendingNews } from '../scraper/newsScanner';
import { generateContent } from '../ai/contentGenerator';
import { renderContentVideo } from '../video/videoRenderer';

interface PipelineOptions {
  topic: string;
  format: ContentFormat;
  languages: Language[];
  platforms: ('reels' | 'tiktok' | 'shorts')[];
}

export async function runContentPipeline(options: PipelineOptions) {
  const { topic, format, languages, platforms } = options;

  console.log(`Starting pipeline for topic: ${topic}`);

  // Step 1: Research
  console.log('Step 1: Scanning trending news...');
  const articles = await scanTrendingNews(topic);
  const researchData = articles
    .map(a => `${a.title}\n${a.content}`)
    .join('\n\n---\n\n');

  // Step 2: Generate content in multiple languages
  console.log('Step 2: Generating content...');
  const content: Record<Language, string> = {} as any;

  for (const language of languages) {
    content[language] = await generateContent({
      topic,
      research: researchData,
      format,
      language,
      tone: 'expert',
      aiProvider: 'claude'
    });
  }

  // Step 3: Extract key points for video
  console.log('Step 3: Extracting key points...');
  const keyPoints = extractKeyPoints(content.en || content.vi);
  const statistics = extractStatistics(researchData);

  // Step 4: Render videos for each platform
  console.log('Step 4: Rendering videos...');
  const videos: Record<string, string> = {};

  for (const platform of platforms) {
    const videoPath = await renderContentVideo({
      title: topic,
      keyPoints,
      statistics,
      platform
    });
    videos[platform] = videoPath;
  }

  console.log('Pipeline completed successfully!');

  return {
    content,
    videos,
    researchSources: articles.length
  };
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or numbered items from content
  const lines = content.split('\n');
  const points: string[] = [];

  for (const line of lines) {
    const trimmed = line.trim();
    if (
      trimmed.match(/^[\d]+\./) || 
      trimmed.match(/^[-*]/) ||
      trimmed.match(/^##/)
    ) {
      const cleaned = trimmed
        .replace(/^[\d]+\./, '')
        .replace(/^[-*]/, '')
        .replace(/^##/, '')
        .trim();
      
      if (cleaned.length > 10 && cleaned.length < 100) {
        points.push(cleaned);
      }
    }
  }

  return points.slice(0, 5); // Top 5 key points
}

function extractStatistics(research: string): Array<{ label: string; value: string }> {
  const stats: Array<{ label: string; value: string }> = [];
  const numberPattern = /(\d+(?:,\d+)*(?:\.\d+)?)\s*(%|million|billion|thousand)/gi;
  
  const matches = research.match(numberPattern);
  if (matches) {
    matches.slice(0, 3).forEach(match => {
      stats.push({
        value: match,
        label: 'Growth Metric'
      });
    });
  }

  return stats;
}
```

### 6. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { topic, format, languages, platforms } = body;

    if (!topic || !format) {
      return NextResponse.json(
        { error: 'Missing required fields: topic, format' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      topic,
      format: format || 'toplist',
      languages: languages || ['en', 'vi'],
      platforms: platforms || ['reels', 'tiktok']
    });

    return NextResponse.json({
      success: true,
      data: result
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video preview
npm run remotion:preview

# Render specific video composition
npm run remotion:render
```

## Common Patterns

### Pattern 1: Custom Content Templates

```typescript
// Create custom format templates
const customTemplates = {
  'linkedin-post': {
    structure: 'hook -> story -> lesson -> cta',
    maxLength: 1300,
    tone: 'professional'
  },
  'twitter-thread': {
    structure: 'hook -> points -> conclusion',
    maxLength: 280,
    tone: 'casual'
  }
};
```

### Pattern 2: Batch Processing

```typescript
// Process multiple topics simultaneously
const topics = ['AI Marketing', 'Content Automation', 'Video SEO'];

const results = await Promise.all(
  topics.map(topic =>
    runContentPipeline({
      topic,
      format: 'how-to',
      languages: ['en'],
      platforms: ['reels']
    })
  )
);
```

### Pattern 3: Content Scheduling

```typescript
// Schedule content generation
interface ScheduledContent {
  topic: string;
  publishDate: Date;
  platforms: string[];
}

async function scheduleContentGeneration(items: ScheduledContent[]) {
  for (const item of items) {
    const delay = item.publishDate.getTime() - Date.now();
    
    setTimeout(async () => {
      await runContentPipeline({
        topic: item.topic,
        format: 'toplist',
        languages: ['en', 'vi'],
        platforms: item.platforms as any
      });
    }, delay);
  }
}
```

## Troubleshooting

### API Rate Limits
If you encounter rate limiting:
```typescript
// Add delay between API calls
async function withRateLimit<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delay));
  return fn();
}
```

### Video Rendering Memory Issues
For large video renders, increase Node memory:
```bash
NODE_OPTIONS=--max-old-space-size=4096 npm run remotion:render
```

### Missing Environment Variables
Validate environment variables at startup:
```typescript
const requiredEnvVars = ['ANTHROPIC_API_KEY', 'OPENAI_API_KEY', 'RAPIDAPI_KEY'];

requiredEnvVars.forEach(varName => {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
});
```

### Content Quality Issues
Improve output by refining prompts:
```typescript
// Add more specific instructions to prompts
const enhancedPrompt = `
${basePrompt}

Additional Requirements:
- Include specific examples and case studies
- Use data from the last 30 days only
- Focus on actionable insights
- Keep paragraphs under 100 words
`;
```

This skill covers the complete workflow for automating content creation using the Ultimate AI Content Pipeline, from research to video generation and multi-platform publishing.
