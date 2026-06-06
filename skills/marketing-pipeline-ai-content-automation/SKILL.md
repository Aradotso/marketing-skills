---
name: marketing-pipeline-ai-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate marketing content from research to video
  - set up AI content automation workflow
  - create automated content pipeline with Claude
  - build content research and video generation system
  - implement marketing automation with AI agents
  - configure content pipeline with OpenAI and Remotion
  - automate social media content creation end-to-end
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into complete content packages including articles, images, and videos.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-crawls** recent news and trends from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates** multi-format content (toplist, POV, case studies, how-tos) in multiple languages
- **Renders** videos and infographics automatically using Remotion
- **Optimizes** output for various platforms (Reels, TikTok, Shorts)

The entire workflow is driven by AI agents that handle research, writing, and multimedia generation.

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

Create a `.env.local` file in the project root:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# RapidAPI for news/data crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_here

# Next.js configuration
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI agent integrations
│   │   ├── crawler/     # Content crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── config/          # Configuration files
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNewsSource } from '@/lib/crawler/newsScanner';
import { analyzeContent } from '@/lib/ai/contentAnalyzer';

// Crawl latest news on a topic
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  const articles = await Promise.all(
    sources.map(source => 
      crawlNewsSource({
        source,
        keyword,
        timeRange: '24h',
        limit: 10
      })
    )
  );

  // Flatten and analyze
  const allArticles = articles.flat();
  const insights = await analyzeContent(allArticles, {
    provider: 'anthropic', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return {
    articles: allArticles,
    insights
  };
}
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/generator/contentEngine';

// Generate multi-format content
async function createContentPipeline(keyword: string, insights: any) {
  const content = await generateContent({
    keyword,
    insights,
    formats: ['toplist', 'how-to', 'case-study'],
    languages: ['en', 'vi'],
    tone: 'professional', // or 'friendly', 'humorous'
    aiProvider: 'anthropic',
    model: 'claude-3-opus-20240229'
  });

  return content;
}

// Example response structure
interface GeneratedContent {
  format: string;
  language: string;
  title: string;
  body: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    keywords: string[];
  };
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

// Render video from content
async function generateVideoContent(content: GeneratedContent) {
  const videoConfig = {
    composition: 'ContentVideo',
    props: {
      title: content.title,
      highlights: extractHighlights(content.body),
      duration: 30, // seconds
      aspectRatio: '9:16', // for Reels/TikTok
      theme: 'modern'
    },
    output: `output/${content.format}-${Date.now()}.mp4`
  };

  const result = await renderVideo(videoConfig);
  
  return {
    videoPath: result.output,
    thumbnail: result.thumbnail,
    duration: result.duration
  };
}

function extractHighlights(body: string): string[] {
  // Extract key points for video slides
  const sentences = body.split('. ');
  return sentences.slice(0, 5).map(s => s.trim());
}
```

## Complete Workflow Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY!,
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await pipeline.research({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generate({
      research,
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi'],
      aiProvider: 'anthropic'
    });

    // Step 3: Create videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.renderVideos({
      contents: content,
      platforms: ['tiktok', 'reels', 'shorts']
    });

    // Step 4: Save results
    const results = await pipeline.save({
      content,
      videos,
      metadata: {
        keyword,
        createdAt: new Date(),
        sources: research.sources
      }
    });

    console.log('✅ Pipeline complete!');
    return results;

  } catch (error) {
    console.error('❌ Pipeline error:', error);
    throw error;
  }
}

// Usage
runFullPipeline('AI Marketing Automation 2026')
  .then(results => {
    console.log(`Generated ${results.content.length} articles`);
    console.log(`Rendered ${results.videos.length} videos`);
  });
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, formats, languages } = await request.json();

    const pipeline = new ContentPipeline({
      openaiKey: process.env.OPENAI_API_KEY!,
      anthropicKey: process.env.ANTHROPIC_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!
    });

    const research = await pipeline.research({ keyword });
    const content = await pipeline.generate({
      research,
      formats: formats || ['toplist'],
      languages: languages || ['en']
    });

    return NextResponse.json({
      success: true,
      data: content
    });

  } catch (error: any) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNewsSource } from '@/lib/crawler/newsScanner';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const source = searchParams.get('source') || 'techcrunch';

  if (!keyword) {
    return NextResponse.json({
      error: 'Keyword is required'
    }, { status: 400 });
  }

  const articles = await crawlNewsSource({
    source,
    keyword,
    timeRange: '24h'
  });

  return NextResponse.json({
    keyword,
    source,
    count: articles.length,
    articles
  });
}
```

## Common Patterns

### Custom AI Prompt Templates

```typescript
// src/lib/ai/prompts.ts
export const CONTENT_PROMPTS = {
  toplist: `Create a compelling top 10 list about {keyword}.
    Use data from: {research}
    Language: {language}
    Tone: {tone}
    
    Format:
    - Catchy headline
    - Brief intro
    - 10 numbered items with explanations
    - Conclusion with CTA`,

  howTo: `Write a comprehensive how-to guide about {keyword}.
    Based on insights: {research}
    Language: {language}
    
    Structure:
    - Problem statement
    - Step-by-step instructions
    - Tips and best practices
    - Common mistakes to avoid`,

  caseStudy: `Develop a detailed case study on {keyword}.
    Research data: {research}
    
    Include:
    - Background/Challenge
    - Solution approach
    - Results with metrics
    - Key takeaways`
};

export function buildPrompt(
  template: keyof typeof CONTENT_PROMPTS,
  variables: Record<string, string>
): string {
  let prompt = CONTENT_PROMPTS[template];
  
  Object.entries(variables).forEach(([key, value]) => {
    prompt = prompt.replace(`{${key}}`, value);
  });
  
  return prompt;
}
```

### Batch Processing

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function batchProcess(keywords: string[]) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY!,
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  const results = [];

  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    try {
      const result = await pipeline.run({
        keyword,
        formats: ['toplist', 'how-to'],
        languages: ['en', 'vi']
      });
      
      results.push({
        keyword,
        status: 'success',
        data: result
      });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
      
    } catch (error: any) {
      results.push({
        keyword,
        status: 'failed',
        error: error.message
      });
    }
  }

  return results;
}
```

### Custom Video Templates

```typescript
// remotion/compositions/CustomBrand.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface CustomBrandProps {
  title: string;
  highlights: string[];
  brandColor: string;
}

export const CustomBrandVideo: React.FC<CustomBrandProps> = ({
  title,
  highlights,
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));
  const currentSlide = Math.floor(frame / (fps * 3));

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{
        opacity,
        padding: 60,
        color: brandColor
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        {highlights[currentSlide] && (
          <p style={{ fontSize: 36, lineHeight: 1.6 }}>
            {highlights[currentSlide]}
          </p>
        )}
      </div>
    </AbsoluteFill>
  );
};
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build
npm run start

# Render videos (Remotion)
npx remotion render ContentVideo output/video.mp4

# Render with custom props
npx remotion render ContentVideo output/custom.mp4 --props='{"title":"My Title"}'
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Memory Issues with Video Rendering

```typescript
// Process videos in chunks
async function renderVideosInChunks(
  contents: GeneratedContent[],
  chunkSize = 3
) {
  const results = [];
  
  for (let i = 0; i < contents.length; i += chunkSize) {
    const chunk = contents.slice(i, i + chunkSize);
    const videos = await Promise.all(
      chunk.map(content => generateVideoContent(content))
    );
    results.push(...videos);
    
    // Garbage collection hint
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Claude API Token Limits

```typescript
// Split long content for Claude
function splitIntoChunks(text: string, maxTokens = 4000): string[] {
  const chunks: string[] = [];
  const sentences = text.split('. ');
  let currentChunk = '';
  
  for (const sentence of sentences) {
    const estimated = (currentChunk + sentence).length / 4; // rough token estimate
    
    if (estimated > maxTokens) {
      chunks.push(currentChunk);
      currentChunk = sentence;
    } else {
      currentChunk += (currentChunk ? '. ' : '') + sentence;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

This skill provides comprehensive coverage of the Marketing Pipeline AI Content Automation system, enabling AI agents to assist developers in implementing end-to-end content automation workflows.
