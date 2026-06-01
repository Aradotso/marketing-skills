---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do i automate content creation with AI
  - set up automated marketing pipeline with video generation
  - create AI-powered content workflow with Remotion
  - build automated content research and scriptwriting system
  - generate videos from text content automatically
  - use Claude and OpenAI for content automation
  - configure automated social media content pipeline
  - integrate Remotion for video rendering in marketing
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the Ultimate AI Content Pipeline, a TypeScript-based automated content creation system that handles research, scriptwriting, and video generation end-to-end using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline Share is a complete content automation system that:

- **Auto-crawls research data** from TechCrunch, a16z, X (Twitter), LinkedIn within the last 24 hours
- **Generates multi-format content** (Toplist, POV, Case Study, How-to) in multiple languages (English/Vietnamese)
- **Automatically renders videos** using Remotion for social media platforms (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire content pipeline

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for Research Crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion License (if applicable)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # Research crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── content/     # Content generation pipeline
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research Crawling

```typescript
import { crawlLatestNews } from '@/lib/crawler/news-crawler';

async function gatherResearch(keyword: string) {
  const sources = {
    techcrunch: true,
    twitter: true,
    linkedin: true,
    a16z: true
  };
  
  const researchData = await crawlLatestNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 50
  });
  
  return researchData;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(
  topic: string, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const systemPrompt = `You are an expert content writer creating ${format} articles in ${language}.
Use the research data provided to create data-backed, insightful content.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about: ${topic}`
    }]
  });
  
  return message.content[0].text;
}
```

### 3. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(prompt: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a marketing content writer with a ${tone} tone.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
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

async function generateVideo(
  scriptContent: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      script: scriptContent,
      platform
    }
  });

  const aspectRatios = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  await renderMedia({
    composition: {
      ...composition,
      ...aspectRatios[platform]
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      script: scriptContent,
      platform
    }
  });
}
```

## Complete Content Pipeline Example

```typescript
import { crawlLatestNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { generateVideo } from '@/lib/video/remotion-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  platforms: ('reels' | 'tiktok' | 'shorts')[];
  tone: 'expert' | 'friendly' | 'humorous';
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Crawling research data...');
    const research = await crawlLatestNews({
      keyword: config.keyword,
      sources: {
        techcrunch: true,
        twitter: true,
        linkedin: true,
        a16z: true
      },
      timeframe: '24h',
      limit: 50
    });

    // Step 2: Generate Content in Multiple Languages
    const contents: Record<string, string> = {};
    
    for (const lang of config.languages) {
      console.log(`✍️ Generating ${lang} content...`);
      const content = await generateContent(
        config.keyword,
        config.format,
        lang,
        research,
        config.tone
      );
      contents[lang] = content;
    }

    // Step 3: Generate Videos for Each Platform
    const videos: string[] = [];
    
    for (const platform of config.platforms) {
      for (const lang of config.languages) {
        console.log(`🎬 Rendering ${platform} video (${lang})...`);
        const outputPath = `./output/${config.keyword}_${platform}_${lang}.mp4`;
        
        await generateVideo(
          contents[lang],
          outputPath,
          platform
        );
        
        videos.push(outputPath);
      }
    }

    return {
      research,
      contents,
      videos,
      success: true
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI automation trends 2026',
  format: 'toplist',
  languages: ['en', 'vi'],
  platforms: ['reels', 'tiktok'],
  tone: 'expert'
});
```

## Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en'],
      platforms: body.platforms || ['reels'],
      tone: body.tone || 'expert'
    });

    return NextResponse.json({
      success: true,
      data: result
    });
    
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Content Format Templates

### Toplist Format

```typescript
const toplistPrompt = `
Create a toplist article with:
- Engaging headline with numbers (e.g., "Top 10...")
- Brief introduction setting context
- Each item with: Title, Description (100-150 words), Key stats/data
- Conclusion with actionable takeaways
- Use research data: ${JSON.stringify(researchData)}
`;
```

### POV (Point of View) Format

```typescript
const povPrompt = `
Create a POV article with:
- Strong opinion-led headline
- Personal/expert perspective introduction
- 3-4 main arguments with supporting evidence
- Counter-arguments addressed
- Conclusion with call-to-action
- Tone: ${tone}
- Research context: ${JSON.stringify(researchData)}
`;
```

### Case Study Format

```typescript
const caseStudyPrompt = `
Create a case study with:
- Company/Project overview
- Challenge/Problem statement
- Solution implemented (detailed)
- Results with metrics and data
- Key learnings and best practices
- Based on: ${JSON.stringify(researchData)}
`;
```

## Remotion Video Template Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface ContentVideoProps {
  script: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ script, platform }) => {
  const frame = useCurrentFrame();
  
  const sections = script.split('\n\n').filter(s => s.trim());
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {sections.map((section, index) => (
        <Sequence
          key={index}
          from={index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60
            }}
          >
            <h2 style={{
              color: '#fff',
              fontSize: 48,
              textAlign: 'center',
              fontFamily: 'Inter, sans-serif'
            }}>
              {section}
            </h2>
          </AbsoluteFill>
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

# Access the application
# http://localhost:3000
```

## Building for Production

```bash
# Build the Next.js application
npm run build

# Start production server
npm run start
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultilingualContent(
  topic: string,
  targetLanguages: string[]
) {
  const contents = await Promise.all(
    targetLanguages.map(lang =>
      generateContent(topic, 'toplist', lang as 'en' | 'vi')
    )
  );
  
  return Object.fromEntries(
    targetLanguages.map((lang, idx) => [lang, contents[idx]])
  );
}
```

### Batch Video Generation

```typescript
async function batchGenerateVideos(
  scripts: Array<{ content: string; platform: string; id: string }>
) {
  const videoPromises = scripts.map(script =>
    generateVideo(
      script.content,
      `./output/video_${script.id}.mp4`,
      script.platform as 'reels' | 'tiktok' | 'shorts'
    )
  );
  
  return await Promise.allSettled(videoPromises);
}
```

### Research Data Filtering

```typescript
interface ResearchItem {
  title: string;
  source: string;
  date: string;
  content: string;
  relevance: number;
}

function filterRelevantResearch(
  data: ResearchItem[],
  minRelevance: number = 0.7
): ResearchItem[] {
  return data
    .filter(item => item.relevance >= minRelevance)
    .sort((a, b) => b.relevance - a.relevance)
    .slice(0, 10);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const waitTime = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${waitTime}ms`);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
  throw new Error('Max retries reached');
}
```

### Memory Issues with Large Videos

```typescript
// Process videos in chunks
async function generateVideoSafely(
  script: string,
  outputPath: string,
  platform: string
) {
  const maxScriptLength = 5000;
  
  if (script.length > maxScriptLength) {
    const chunks = script.match(new RegExp(`.{1,${maxScriptLength}}`, 'g')) || [];
    
    for (let i = 0; i < chunks.length; i++) {
      await generateVideo(
        chunks[i],
        `${outputPath}_part${i}.mp4`,
        platform as 'reels' | 'tiktok' | 'shorts'
      );
    }
  } else {
    await generateVideo(script, outputPath, platform as 'reels' | 'tiktok' | 'shorts');
  }
}
```

### Claude/OpenAI API Errors

```typescript
// Graceful fallback between providers
async function generateContentWithFallback(
  topic: string,
  format: string
) {
  try {
    return await generateContent(topic, format as any, 'en');
  } catch (claudeError) {
    console.warn('Claude failed, trying OpenAI:', claudeError);
    try {
      return await generateWithGPT(
        `Create a ${format} article about: ${topic}`,
        'expert'
      );
    } catch (openaiError) {
      console.error('Both AI providers failed');
      throw new Error('Content generation failed');
    }
  }
}
```

### Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file'
    );
  }
}

validateEnvironment();
```

## Best Practices

1. **Always validate API keys** before starting the pipeline
2. **Cache research data** to avoid redundant API calls
3. **Use queues** for batch video generation to prevent memory overflow
4. **Implement proper error handling** with fallback mechanisms
5. **Monitor API usage** to stay within rate limits
6. **Test video outputs** on target platforms before bulk generation
7. **Version control your prompts** for reproducible content quality
