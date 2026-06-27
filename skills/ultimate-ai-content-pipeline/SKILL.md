---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - generate automated content from keyword research
  - create video content with AI scriptwriting
  - automate marketing content pipeline with remotion
  - research and generate blog posts automatically
  - build ai-powered content automation system
  - scrape trending topics and create videos
  - set up automated content generation workflow
  - integrate claude and openai for content creation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript system that automates content creation from research to video generation. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion video rendering to create a complete content automation workflow.

## What This Project Does

The Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research** from sources like TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
- **Generates multi-format content** (toplists, POV articles, case studies, how-tos) in multiple languages
- **Renders videos automatically** using Remotion from generated scripts
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts
- **Provides a Next.js interface** for managing the entire content pipeline

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

### Environment Variables

Create a `.env.local` file with the following keys:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run render
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key Components and APIs

### 1. Research & Scraping Module

```typescript
// src/lib/scraper/research.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  url: string;
  snippet: string;
  publishedDate: string;
  source: string;
}

export async function scrapeLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/search`,
        {
          params: {
            q: keyword,
            source: source,
            timeframe: '24h'
          },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-api.p.rapidapi.com'
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

// Extract insights from research
export async function extractInsights(
  articles: ResearchResult[]
): Promise<string> {
  const articlesText = articles
    .map(a => `${a.title}\n${a.snippet}`)
    .join('\n\n');
    
  return articlesText;
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ContentTone = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerateContentOptions {
  keyword: string;
  research: string;
  format: ContentFormat;
  tone: ContentTone;
  language: Language;
  aiProvider?: 'claude' | 'openai';
}

export async function generateContent(
  options: GenerateContentOptions
): Promise<string> {
  const {
    keyword,
    research,
    format,
    tone,
    language,
    aiProvider = 'claude'
  } = options;
  
  const prompt = buildPrompt(keyword, research, format, tone, language);
  
  if (aiProvider === 'claude') {
    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  } else {
    const response = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return response.choices[0].message.content || '';
  }
}

function buildPrompt(
  keyword: string,
  research: string,
  format: ContentFormat,
  tone: ContentTone,
  language: Language
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with actionable items',
    'pov': 'Write a perspective piece with strong opinions and insights',
    'case-study': 'Develop a detailed case study with data and examples',
    'how-to': 'Create a step-by-step tutorial guide'
  };
  
  const toneInstructions = {
    'expert': 'Use authoritative, professional language',
    'friendly': 'Write in a conversational, approachable tone',
    'humorous': 'Include wit and light humor while staying informative'
  };
  
  const languageInstruction = language === 'vi' 
    ? 'Write in Vietnamese' 
    : 'Write in English';
  
  return `You are an expert content writer. Create content about "${keyword}" based on this research:

${research}

Format: ${formatInstructions[format]}
Tone: ${toneInstructions[tone]}
Language: ${languageInstruction}

Requirements:
- Use data and insights from the research
- Include specific examples and statistics
- Make it engaging and valuable for readers
- Optimize for SEO with natural keyword usage
- Structure with clear headings and sections`;
}
```

### 3. Video Script Generation

```typescript
// src/lib/ai/script-generator.ts
import { generateContent } from './content-generator';

interface VideoScript {
  title: string;
  hook: string;
  scenes: ScriptScene[];
  cta: string;
}

interface ScriptScene {
  duration: number; // in seconds
  text: string;
  visualDescription: string;
}

export async function generateVideoScript(
  keyword: string,
  research: string,
  platform: 'reels' | 'tiktok' | 'shorts' = 'shorts'
): Promise<VideoScript> {
  const maxDuration = platform === 'reels' ? 60 : 
                      platform === 'tiktok' ? 60 : 60;
  
  const prompt = `Create a ${maxDuration}-second video script about "${keyword}" based on this research:

${research}

Format as JSON with this structure:
{
  "title": "Catchy title",
  "hook": "Attention-grabbing first 3 seconds",
  "scenes": [
    {
      "duration": 5,
      "text": "What the narrator says",
      "visualDescription": "What viewers see on screen"
    }
  ],
  "cta": "Call to action"
}

Make it engaging, fast-paced, and optimized for ${platform}.`;

  const response = await generateContent({
    keyword,
    research: prompt,
    format: 'how-to',
    tone: 'friendly',
    language: 'en',
    aiProvider: 'claude'
  });
  
  // Parse JSON response
  const jsonMatch = response.match(/\{[\s\S]*\}/);
  if (jsonMatch) {
    return JSON.parse(jsonMatch[0]);
  }
  
  throw new Error('Failed to generate valid video script');
}
```

### 4. Remotion Video Rendering

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Audio, Img, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoScene {
  text: string;
  visualDescription: string;
  duration: number;
}

export const VideoComposition: React.FC<{
  scenes: VideoScene[];
  title: string;
}> = ({ scenes, title }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  let currentFrame = 0;
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Scene */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            fontSize: 60,
            color: 'white',
            fontWeight: 'bold',
            textAlign: 'center',
            padding: 40
          }}
        >
          {title}
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Scenes */}
      {scenes.map((scene, index) => {
        const from = currentFrame + (index === 0 ? fps * 3 : 0);
        const durationInFrames = Math.floor(scene.duration * fps);
        currentFrame += durationInFrames;
        
        return (
          <Sequence
            key={index}
            from={from}
            durationInFrames={durationInFrames}
          >
            <AbsoluteFill
              style={{
                justifyContent: 'center',
                alignItems: 'center',
                backgroundColor: `hsl(${(index * 40) % 360}, 70%, 20%)`
              }}
            >
              <div
                style={{
                  fontSize: 40,
                  color: 'white',
                  padding: 60,
                  textAlign: 'center',
                  maxWidth: '80%'
                }}
              >
                {scene.text}
              </div>
            </AbsoluteFill>
          </Sequence>
        );
      })}
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';
import { VideoScript } from '../ai/script-generator';

export async function renderVideo(
  script: VideoScript,
  outputPath: string
): Promise<string> {
  const compositionId = 'VideoComposition';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      scenes: script.scenes,
      title: script.title
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      scenes: script.scenes,
      title: script.title
    }
  });
  
  return outputPath;
}
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scrapeLatestNews, extractInsights } from '../scraper/research';
import { generateContent, generateVideoScript } from '../ai';
import { renderVideo } from '../video/renderer';
import path from 'path';

export interface PipelineOptions {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  includeVideo: boolean;
  languages: ('en' | 'vi')[];
}

export interface PipelineResult {
  keyword: string;
  research: string;
  content: {
    en?: string;
    vi?: string;
  };
  videoPath?: string;
}

export async function runContentPipeline(
  options: PipelineOptions
): Promise<PipelineResult> {
  const { keyword, contentFormat, includeVideo, languages } = options;
  
  console.log(`🔍 Researching: ${keyword}`);
  
  // Step 1: Research
  const articles = await scrapeLatestNews(keyword);
  const research = await extractInsights(articles);
  
  console.log(`📊 Found ${articles.length} articles`);
  
  // Step 2: Generate content in multiple languages
  const content: { en?: string; vi?: string } = {};
  
  for (const language of languages) {
    console.log(`✍️ Generating ${language.toUpperCase()} content...`);
    
    content[language] = await generateContent({
      keyword,
      research,
      format: contentFormat,
      tone: 'friendly',
      language,
      aiProvider: 'claude'
    });
  }
  
  // Step 3: Generate video if requested
  let videoPath: string | undefined;
  
  if (includeVideo) {
    console.log(`🎬 Generating video script...`);
    
    const script = await generateVideoScript(keyword, research, 'shorts');
    
    console.log(`🎥 Rendering video...`);
    
    const outputPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${keyword.replace(/\s+/g, '-')}-${Date.now()}.mp4`
    );
    
    videoPath = await renderVideo(script, outputPath);
    
    console.log(`✅ Video rendered: ${videoPath}`);
  }
  
  return {
    keyword,
    research,
    content,
    videoPath
  };
}
```

### 6. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      keyword,
      contentFormat = 'how-to',
      includeVideo = false,
      languages = ['en']
    } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      contentFormat,
      includeVideo,
      languages
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          keyword,
          contentFormat: 'how-to',
          includeVideo: true,
          languages: ['en', 'vi']
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">
        AI Content Pipeline
      </h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
      </div>
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-500 text-white px-6 py-3 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Results</h2>
          
          {result.content.en && (
            <div className="mb-6">
              <h3 className="text-xl font-semibold mb-2">English</h3>
              <div className="prose max-w-none">
                {result.content.en}
              </div>
            </div>
          )}
          
          {result.content.vi && (
            <div className="mb-6">
              <h3 className="text-xl font-semibold mb-2">Vietnamese</h3>
              <div className="prose max-w-none">
                {result.content.vi}
              </div>
            </div>
          )}
          
          {result.videoPath && (
            <div>
              <h3 className="text-xl font-semibold mb-2">Video</h3>
              <video
                src={result.videoPath}
                controls
                className="w-full max-w-2xl"
              />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate content for multiple keywords
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      includeVideo: false,
      languages: ['en']
    });
    
    results.push(result);
    
    // Add delay to avoid rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Research Sources

```typescript
// Add custom scraping logic
async function scrapeCustomSource(keyword: string) {
  // Implement your own scraping logic
  // using Puppeteer, Cheerio, or other tools
  
  return {
    title: 'Custom Article',
    snippet: 'Content preview...',
    url: 'https://example.com',
    source: 'custom'
  };
}
```

## Troubleshooting

### API Rate Limits
- Implement exponential backoff for API calls
- Use caching for research results
- Consider using multiple API keys in rotation

### Video Rendering Issues
- Ensure sufficient disk space for video output
- Check Remotion dependencies are installed correctly
- Verify AWS credentials if using cloud rendering

### Memory Issues
- Process large batches in chunks
- Use streaming for large content pieces
- Clear cache between operations

### AI Response Quality
- Provide more detailed research context
- Adjust temperature parameters (lower = more focused)
- Use system prompts to guide output format
- Validate and retry if response is malformed
