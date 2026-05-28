---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline for marketing
  - generate videos from text automatically
  - crawl news and create content with AI
  - build automated content workflow with Claude
  - use Remotion for AI-generated videos
  - create multilingual content with OpenAI
  - automate research and scriptwriting
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive AI-powered content automation system that handles the entire content lifecycle: from researching trending topics, generating multi-format articles in multiple languages, to rendering videos automatically. It integrates Claude 3, OpenAI, RapidAPI for data crawling, and Remotion for video generation.

## What It Does

The Ultimate AI Content Pipeline automates:
- **News crawling** from sources like TechCrunch, a16z, Twitter/X, LinkedIn (24h recent data)
- **Content generation** in multiple formats (toplist, POV, case study, how-to)
- **Multilingual output** (English & Vietnamese) with customizable tone
- **Automatic video rendering** using Remotion for social media (Reels, TikTok, Shorts)
- **End-to-end workflow** from keyword input to publishable content

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

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Data Crawling (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Remotion for video rendering
REMOTION_LICENSE_KEY=your_remotion_license_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
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
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
// src/lib/crawler/newsCrawler.ts
import axios from 'axios';

interface NewsSource {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
}

export async function crawlRecentNews(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/v1/news/${source}`,
        {
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
            'X-RapidAPI-Host': 'news-api.rapidapi.com'
          },
          params: {
            q: topic,
            from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(),
            sortBy: 'publishedAt'
          }
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return results;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: string;
}

export async function generateContent(req: ContentRequest): Promise<string> {
  const formatPrompts = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a specific point of view with strong opinions',
    'case-study': 'Analyze a real example with data and outcomes',
    'how-to': 'Provide step-by-step instructions'
  };
  
  const tonePrompts = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include witty remarks and engaging humor'
  };
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `You are a professional content creator. 
        
Topic: ${req.topic}
Format: ${formatPrompts[req.format]}
Language: ${req.language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${tonePrompts[req.tone]}

Research Data:
${req.research}

Write a comprehensive, engaging article based on the research above. Include:
- Attention-grabbing headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure`
      }
    ]
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. Alternative: OpenAI Content Generation

```typescript
// src/lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithGPT(
  topic: string,
  research: string,
  language: 'en' | 'vi'
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a marketing content expert creating ${language === 'en' ? 'English' : 'Vietnamese'} content.`
      },
      {
        role: 'user',
        content: `Topic: ${topic}\n\nResearch:\n${research}\n\nCreate an engaging article.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  points: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(content: VideoContent): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: content,
  });
  
  const composition = compositions[0];
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${content.style}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: content,
  });
  
  return outputLocation;
}
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/contentPipeline.ts
import { crawlRecentNews } from '@/lib/crawler/newsCrawler';
import { generateContent } from '@/lib/ai/claudeGenerator';
import { renderContentVideo } from '@/lib/video/videoRenderer';

export async function runContentPipeline(
  topic: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    languages: ('en' | 'vi')[];
    generateVideo: boolean;
  }
) {
  // Step 1: Research
  console.log('🔍 Crawling recent news...');
  const news = await crawlRecentNews(topic);
  const research = news.map(n => `${n.title}\n${n.content}`).join('\n\n');
  
  // Step 2: Generate content for each language
  console.log('✍️ Generating content...');
  const contents: Record<string, string> = {};
  
  for (const lang of options.languages) {
    contents[lang] = await generateContent({
      topic,
      format: options.format,
      language: lang,
      tone: 'expert',
      research
    });
  }
  
  // Step 3: Optional video generation
  let videoPath: string | null = null;
  if (options.generateVideo) {
    console.log('🎬 Rendering video...');
    const videoContent = extractVideoPoints(contents['en'] || contents['vi']);
    videoPath = await renderContentVideo(videoContent);
  }
  
  return {
    contents,
    videoPath,
    research: news
  };
}

function extractVideoPoints(content: string): any {
  // Extract key points from content for video
  const lines = content.split('\n').filter(l => l.trim());
  const points = lines
    .filter(l => /^\d+\./.test(l) || /^-/.test(l))
    .slice(0, 5)
    .map(p => p.replace(/^\d+\.\s*/, '').replace(/^-\s*/, ''));
  
  return {
    title: lines[0] || 'Content Video',
    points,
    style: 'reels'
  };
}
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(req: NextRequest) {
  try {
    const { topic, format, languages, generateVideo } = await req.json();
    
    if (!topic) {
      return NextResponse.json(
        { error: 'Topic is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(topic, {
      format: format || 'toplist',
      languages: languages || ['en', 'vi'],
      generateVideo: generateVideo ?? false
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [topic, setTopic] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic,
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true
        })
      });
      
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      <input
        value={topic}
        onChange={(e) => setTopic(e.target.value)}
        placeholder="Enter topic..."
        disabled={loading}
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h3>English Content:</h3>
          <pre>{result.contents.en}</pre>
          
          <h3>Vietnamese Content:</h3>
          <pre>{result.contents.vi}</pre>
          
          {result.videoPath && (
            <video src={result.videoPath} controls />
          )}
        </div>
      )}
    </div>
  );
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

# Render videos (if using Remotion separately)
npx remotion render
```

## Troubleshooting

### API Key Issues
- Ensure all API keys are properly set in `.env.local`
- Check API quota limits for OpenAI/Anthropic/RapidAPI
- Verify API keys have proper permissions

### Crawling Failures
```typescript
// Add retry logic
async function crawlWithRetry(topic: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await crawlRecentNews(topic);
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}
```

### Video Rendering Memory Issues
- Reduce video resolution in Remotion config
- Use `--max-old-space-size=4096` flag when running Node
- Process videos in batches

### Rate Limiting
```typescript
// Implement basic rate limiting
const rateLimiter = new Map<string, number>();

function checkRateLimit(ip: string, limit = 10): boolean {
  const now = Date.now();
  const lastCall = rateLimiter.get(ip) || 0;
  
  if (now - lastCall < 60000 / limit) {
    return false;
  }
  
  rateLimiter.set(ip, now);
  return true;
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Queue video rendering** jobs for better performance
3. **Implement proper error handling** for each pipeline stage
4. **Monitor API usage** to stay within quota limits
5. **Use webhooks** for long-running video generation tasks
6. **Store generated content** in a database for reuse
