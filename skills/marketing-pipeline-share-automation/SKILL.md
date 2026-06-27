---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation with multi-language support
triggers:
  - automate content creation with AI research
  - generate marketing videos from scripts automatically
  - create multilingual content with Claude and OpenAI
  - crawl and analyze news for content insights
  - build automated content pipeline with Remotion
  - scrape trending topics for content generation
  - set up AI content research workflow
  - render videos from text content automatically
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using **marketing-pipeline-share**, an all-in-one AI content automation system that handles research, scriptwriting, multi-format content generation, and automated video rendering. The pipeline crawls real-time data from sources like TechCrunch, Twitter, and LinkedIn, then uses Claude 3 and OpenAI to generate content in multiple languages and formats, finally rendering videos with Remotion.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based automation system that:

- **Auto-crawls** trending news and insights from tech media and social platforms
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports bilingual output** (English and Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms
- **Provides a Next.js interface** for managing the entire pipeline

Perfect for content creators, marketers, and agencies who need to produce high-quality, data-backed content at scale.

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
cp .env.example .env
```

### Required Environment Variables

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Optional: Remotion Cloud (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── lib/
│   │   ├── research/      # News crawling and data extraction
│   │   ├── ai/            # Claude/OpenAI integrations
│   │   ├── video/         # Remotion video generation
│   │   └── formats/       # Content format templates
│   ├── pages/
│   │   └── api/           # API routes for pipeline
│   └── remotion/          # Video composition templates
├── public/
└── package.json
```

## Core API & Usage Patterns

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

// Crawl trending topics from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'twitter', 'linkedin'];
  
  const rawData = await researchTopic({
    keyword,
    sources,
    timeframe: '24h',
    limit: 50
  });
  
  // Extract actionable insights
  const insights = await analyzeInsights(rawData, {
    extractStats: true,
    findTrends: true,
    identifyExperts: true
  });
  
  return insights;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  insights: any,
  format: 'toplist' | 'pov' | 'casestudy' | 'howto',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(insights, format, language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
    system: `You are an expert content creator specializing in ${format} format. 
             Write in ${language === 'vi' ? 'Vietnamese' : 'English'} with a professional yet engaging tone.`
  });
  
  return message.content[0].text;
}
```

### 3. Multi-Format Content Templates

```typescript
// lib/formats/templates.ts
export const formatTemplates = {
  toplist: {
    structure: [
      'hook',
      'introduction',
      'items', // 5-10 items with data
      'conclusion',
      'cta'
    ],
    tone: 'authoritative',
  },
  
  pov: {
    structure: [
      'provocative_statement',
      'personal_experience',
      'supporting_data',
      'counterargument',
      'conclusion'
    ],
    tone: 'conversational',
  },
  
  casestudy: {
    structure: [
      'problem',
      'solution',
      'implementation',
      'results',
      'lessons_learned'
    ],
    tone: 'analytical',
  },
  
  howto: {
    structure: [
      'overview',
      'prerequisites',
      'step_by_step', // numbered steps
      'tips_and_tricks',
      'conclusion'
    ],
    tone: 'instructional',
  }
};

function buildPrompt(insights: any, format: keyof typeof formatTemplates, language: string) {
  const template = formatTemplates[format];
  
  return `
Create a ${format} article based on these insights:

${JSON.stringify(insights, null, 2)}

Structure: ${template.structure.join(' -> ')}
Tone: ${template.tone}
Language: ${language}
Requirements:
- Include specific data points and statistics
- Reference real examples from the research
- Keep paragraphs under 3 sentences for readability
- Add relevant subheadings
${language === 'vi' ? '- Use natural Vietnamese expressions, avoid direct translation' : ''}
  `.trim();
}
```

### 4. Bilingual Content Pipeline

```typescript
// lib/pipeline/bilingual.ts
async function generateBilingualContent(keyword: string, format: string) {
  // Step 1: Research (language-agnostic)
  const research = await gatherResearch(keyword);
  
  // Step 2: Generate both versions in parallel
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, format as any, 'en'),
    generateContent(research, format as any, 'vi')
  ]);
  
  return {
    en: {
      title: extractTitle(englishContent),
      content: englishContent,
      metadata: extractMetadata(englishContent)
    },
    vi: {
      title: extractTitle(vietnameseContent),
      content: vietnameseContent,
      metadata: extractMetadata(vietnameseContent)
    },
    research: research
  };
}
```

### 5. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  style: 'reels' | 'tiktok' | 'shorts'
) {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: parseContentForVideo(content),
      style,
      duration: calculateDuration(content)
    }
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${style}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });
  
  return outputLocation;
}

function parseContentForVideo(content: string) {
  // Extract key points for video frames
  const sections = content.split('\n\n');
  
  return sections
    .filter(s => s.trim().length > 0)
    .slice(0, 8) // Max 8 frames for short video
    .map(text => ({
      text: text.substring(0, 100), // Trim for readability
      duration: 30 // frames per slide
    }));
}
```

### 6. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface ContentVideoProps {
  content: Array<{ text: string; duration: number }>;
  style: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, style }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {content.map((slide, index) => {
        const startFrame = content
          .slice(0, index)
          .reduce((sum, s) => sum + s.duration, 0);
        
        return (
          <Sequence
            key={index}
            from={startFrame}
            durationInFrames={slide.duration}
          >
            <SlideContent text={slide.text} style={style} />
          </Sequence>
        );
      })}
    </AbsoluteFill>
  );
};

const SlideContent: React.FC<{ text: string; style: string }> = ({ text, style }) => {
  const frame = useCurrentFrame();
  const opacity = Math.min(1, frame / 10); // Fade in
  
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
        opacity
      }}
    >
      <h1 style={{
        fontSize: style === 'tiktok' ? 48 : 60,
        color: '#fff',
        textAlign: 'center',
        lineHeight: 1.4
      }}>
        {text}
      </h1>
    </AbsoluteFill>
  );
};
```

### 7. Complete Pipeline API Route

```typescript
// pages/api/pipeline/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, includeVideo, videoStyle } = req.body;
  
  try {
    // Step 1: Research
    res.write(JSON.stringify({ stage: 'research', progress: 10 }));
    const research = await gatherResearch(keyword);
    
    // Step 2: Content generation
    res.write(JSON.stringify({ stage: 'generation', progress: 40 }));
    const content = await generateBilingualContent(keyword, format);
    
    // Step 3: Video rendering (optional)
    let videoUrl = null;
    if (includeVideo) {
      res.write(JSON.stringify({ stage: 'video', progress: 70 }));
      videoUrl = await renderContentVideo(
        content.en.content,
        videoStyle || 'reels'
      );
    }
    
    // Final response
    res.status(200).json({
      success: true,
      data: {
        content,
        videoUrl,
        research: research.summary
      }
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({
      error: 'Pipeline failed',
      message: error.message
    });
  }
}
```

## Configuration

### Customizing Content Tone

```typescript
// lib/config/tones.ts
export const tonePresets = {
  professional: {
    vocabulary: 'formal',
    sentenceLength: 'medium-long',
    emoticons: false,
    personalPronouns: 'minimal'
  },
  
  friendly: {
    vocabulary: 'casual',
    sentenceLength: 'short-medium',
    emoticons: true,
    personalPronouns: 'frequent'
  },
  
  humorous: {
    vocabulary: 'playful',
    sentenceLength: 'varied',
    emoticons: true,
    personalPronouns: 'frequent',
    includeJokes: true
  },
  
  expert: {
    vocabulary: 'technical',
    sentenceLength: 'long',
    emoticons: false,
    personalPronouns: 'minimal',
    includeCitations: true
  }
};
```

### Video Style Presets

```typescript
// lib/config/video-styles.ts
export const videoStyles = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationPerSlide: 30,
    backgroundColor: '#000',
    textColor: '#fff'
  },
  
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationPerSlide: 25,
    backgroundColor: '#000',
    textColor: '#fff'
  },
  
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationPerSlide: 35,
    backgroundColor: '#000',
    textColor: '#fff'
  }
};
```

## Common Workflows

### Full Automation Workflow

```typescript
async function automatedContentCreation(topic: string) {
  // 1. Research phase
  const research = await gatherResearch(topic);
  
  // 2. Generate multiple formats in parallel
  const formats = ['toplist', 'pov', 'casestudy', 'howto'];
  const allContent = await Promise.all(
    formats.map(format => 
      generateBilingualContent(topic, format)
    )
  );
  
  // 3. Render videos for social media
  const videos = await Promise.all(
    allContent.map(content =>
      renderContentVideo(content.en.content, 'reels')
    )
  );
  
  // 4. Return complete package
  return {
    research,
    articles: allContent,
    videos
  };
}
```

### Custom Research Sources

```typescript
// lib/research/custom-sources.ts
import axios from 'axios';

async function crawlCustomSource(url: string, selector: string) {
  const response = await axios.get(url, {
    headers: {
      'User-Agent': 'Mozilla/5.0'
    }
  });
  
  // Use cheerio or puppeteer to parse HTML
  // Return structured data
  return parsedData;
}

// Add to main research function
async function extendedResearch(keyword: string) {
  const defaultResearch = await gatherResearch(keyword);
  
  const customData = await Promise.all([
    crawlCustomSource('https://news.ycombinator.com', '.storylink'),
    crawlCustomSource('https://reddit.com/r/technology', '.title')
  ]);
  
  return {
    ...defaultResearch,
    custom: customData
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process(delay);
    });
  }
  
  private async process(delay: number) {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, delay));
    }
    
    this.processing = false;
    this.process(delay);
  }
}

export const rateLimiter = new RateLimiter();
```

### Video Rendering Failures

```typescript
// Retry logic for Remotion rendering
async function renderWithRetry(
  content: string,
  style: string,
  maxRetries: number = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderContentVideo(content, style as any);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      
      if (i === maxRetries - 1) throw error;
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, 5000 * (i + 1)));
    }
  }
}
```

### Memory Issues with Large Content

```typescript
// Stream processing for large datasets
import { Readable } from 'stream';

async function processLargeResearch(keyword: string) {
  const stream = new Readable({ objectMode: true });
  
  // Process in chunks
  const chunkSize = 10;
  let offset = 0;
  
  while (true) {
    const chunk = await fetchResearchChunk(keyword, offset, chunkSize);
    
    if (chunk.length === 0) break;
    
    for (const item of chunk) {
      stream.push(item);
    }
    
    offset += chunkSize;
  }
  
  stream.push(null); // End stream
  return stream;
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Access the UI
open http://localhost:3000

# Render a video manually (Remotion)
npm run remotion:preview
```

## Best Practices

1. **Always cache research data** to avoid redundant API calls
2. **Use streaming responses** for real-time progress updates
3. **Implement proper error handling** for each pipeline stage
4. **Test video rendering locally** before deploying to production
5. **Monitor API usage** to stay within rate limits
6. **Version control your prompts** for consistent content quality

This skill enables AI agents to effectively use the marketing-pipeline-share system for automated, high-quality content creation at scale.
