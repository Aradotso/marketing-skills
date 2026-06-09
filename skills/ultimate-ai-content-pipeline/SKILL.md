---
name: ultimate-ai-content-pipeline
description: AI-powered content automation system that researches, generates scripts, and creates videos with Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up content pipeline with video generation
  - research and generate content automatically
  - create AI content workflow with Remotion
  - build automated marketing content system
  - generate videos from text content automatically
  - integrate Claude and OpenAI for content automation
  - automate social media content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based automation system that handles the complete content creation workflow: from research and script generation to automatic video rendering and social media posting.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation platform that:

- **Auto-researches** latest news from TechCrunch, a16z, Twitter, LinkedIn using web crawling
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 and OpenAI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Manages posting** with scheduling and multi-platform distribution

Built with Next.js, TypeScript, and integrates with OpenAI, Anthropic Claude, RapidAPI, and Remotion.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment template
cp .env.example .env.local
```

## Configuration

Set up your `.env.local` with required API keys:

```bash
# AI Providers
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here

# Research APIs
RAPIDAPI_KEY=your_key_here

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_key_here

# Social Media APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_token_here
TWITTER_API_KEY=your_key_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (OpenAI, Claude)
│   │   ├── research/    # Web scraping & research
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Components

### 1. Research Module

Automatically scrape and analyze recent content from multiple sources:

```typescript
// src/lib/research/scraper.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchResult[]> {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const articles = await rapidAPI.searchNews({
      query: keyword,
      source: source,
      timeframe: '24h'
    });
    
    results.push(...articles);
  }
  
  return results;
}

// Usage in API route
export async function POST(req: Request) {
  const { keyword } = await req.json();
  
  const research = await researchTopic(keyword, [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ]);
  
  return Response.json({ research });
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with multiple format support:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchResult[];
}

interface GeneratedContent {
  title: string;
  body: string;
  summary: string;
  keywords: string[];
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateWithClaude(
    request: ContentRequest
  ): Promise<GeneratedContent> {
    const researchContext = request.researchData
      .map(r => `${r.title}: ${r.summary}`)
      .join('\n\n');
    
    const prompt = this.buildPrompt(request, researchContext);
    
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    const content = message.content[0].text;
    return this.parseContent(content);
  }
  
  async generateWithOpenAI(
    request: ContentRequest
  ): Promise<GeneratedContent> {
    const researchContext = request.researchData
      .map(r => `${r.title}: ${r.summary}`)
      .join('\n\n');
    
    const prompt = this.buildPrompt(request, researchContext);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator and marketer.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      temperature: 0.7
    });
    
    const content = completion.choices[0].message.content!;
    return this.parseContent(content);
  }
  
  private buildPrompt(
    request: ContentRequest,
    researchContext: string
  ): string {
    return `Create a ${request.format} article about "${request.topic}" in ${request.language} with a ${request.tone} tone.

Research Context:
${researchContext}

Requirements:
- Format: ${request.format}
- Language: ${request.language}
- Tone: ${request.tone}
- Include data-backed insights from the research
- Structure with clear headings and sections
- Add relevant keywords for SEO

Output format:
TITLE: [title here]
SUMMARY: [summary here]
KEYWORDS: [comma-separated keywords]
BODY:
[full article content]`;
  }
  
  private parseContent(rawContent: string): GeneratedContent {
    const titleMatch = rawContent.match(/TITLE:\s*(.+)/);
    const summaryMatch = rawContent.match(/SUMMARY:\s*(.+)/);
    const keywordsMatch = rawContent.match(/KEYWORDS:\s*(.+)/);
    const bodyMatch = rawContent.match(/BODY:\s*([\s\S]+)/);
    
    return {
      title: titleMatch?.[1] || '',
      summary: summaryMatch?.[1] || '',
      keywords: keywordsMatch?.[1]?.split(',').map(k => k.trim()) || [],
      body: bodyMatch?.[1] || ''
    };
  }
}
```

### 3. Video Generation with Remotion

Transform content into engaging video formats:

```typescript
// remotion/compositions.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot = () => {
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
          title: 'Your Title',
          points: [],
          backgroundColor: '#000000'
        }}
      />
    </>
  );
};
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  backgroundColor
}) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60
      }}>
        <h1 style={{
          fontSize: 80,
          fontWeight: 'bold',
          color: 'white',
          textAlign: 'center',
          opacity: titleOpacity
        }}>
          {title}
        </h1>
        
        <div style={{ marginTop: 60 }}>
          {points.map((point, index) => {
            const pointOpacity = interpolate(
              frame,
              [30 + index * 20, 50 + index * 20],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div
                key={index}
                style={{
                  fontSize: 40,
                  color: 'white',
                  marginBottom: 30,
                  opacity: pointOpacity
                }}
              >
                {point}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      points: content.body.split('\n').filter(Boolean).slice(0, 5),
      backgroundColor: '#1a1a1a'
    }
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.props
  });
  
  return outputLocation;
}
```

### 4. Complete Workflow Integration

Orchestrate the entire pipeline from research to publishing:

```typescript
// src/lib/pipeline/content-pipeline.ts
import { researchTopic } from '@/lib/research/scraper';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  autoPost: boolean;
}

export class ContentPipeline {
  private generator: ContentGenerator;
  
  constructor() {
    this.generator = new ContentGenerator();
  }
  
  async execute(config: PipelineConfig) {
    // Step 1: Research
    console.log('🔍 Researching topic:', config.keyword);
    const research = await researchTopic(config.keyword);
    
    // Step 2: Generate content for each language
    const contents = [];
    for (const language of config.languages) {
      console.log(`✍️ Generating ${language} content...`);
      
      const content = await this.generator.generateWithClaude({
        topic: config.keyword,
        format: config.format,
        language,
        tone: 'expert',
        researchData: research
      });
      
      contents.push({ language, content });
    }
    
    // Step 3: Generate videos if enabled
    const videos = [];
    if (config.generateVideo) {
      for (const { language, content } of contents) {
        console.log(`🎬 Rendering ${language} video...`);
        const videoPath = await renderContentVideo(content);
        videos.push({ language, videoPath });
      }
    }
    
    // Step 4: Auto-post if enabled
    if (config.autoPost) {
      console.log('📤 Publishing content...');
      // Implementation for posting to social media
    }
    
    return {
      research,
      contents,
      videos
    };
  }
}
```

### 5. API Routes

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(req: NextRequest) {
  try {
    const config = await req.json();
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(config);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Pattern 1: Single Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';
import { researchTopic } from '@/lib/research/scraper';

async function createSinglePost(keyword: string) {
  const research = await researchTopic(keyword);
  const generator = new ContentGenerator();
  
  const content = await generator.generateWithClaude({
    topic: keyword,
    format: 'toplist',
    language: 'en',
    tone: 'friendly',
    researchData: research
  });
  
  return content;
}
```

### Pattern 2: Bilingual Content with Video

```typescript
async function createBilingualVideoContent(keyword: string) {
  const pipeline = new ContentPipeline();
  
  const result = await pipeline.execute({
    keyword,
    format: 'how-to',
    languages: ['en', 'vi'],
    generateVideo: true,
    autoPost: false
  });
  
  return result;
}
```

### Pattern 3: Scheduled Content Batch

```typescript
async function scheduleBatchContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];
  
  for (const keyword of keywords) {
    const result = await pipeline.execute({
      keyword,
      format: 'pov',
      languages: ['en'],
      generateVideo: true,
      autoPost: true
    });
    
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

## CLI Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render specific Remotion composition
npx remotion render ContentVideo output.mp4

# Preview Remotion composition
npx remotion preview
```

## Troubleshooting

### API Rate Limits

```typescript
// Add exponential backoff for API calls
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering with memory limits
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps: composition.props,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  },
  envVariables: {
    NODE_OPTIONS: '--max-old-space-size=4096'
  }
});
```

### Research Scraping Failures

```typescript
// Handle scraping errors gracefully
try {
  const research = await researchTopic(keyword);
  if (research.length === 0) {
    console.warn('No research results found, using fallback');
    // Use cached or default data
  }
} catch (error) {
  console.error('Research failed:', error);
  // Proceed with generation using existing knowledge
}
```

### Content Quality Validation

```typescript
function validateContent(content: GeneratedContent): boolean {
  return (
    content.title.length > 10 &&
    content.body.length > 500 &&
    content.keywords.length >= 3 &&
    content.summary.length > 50
  );
}
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache research results** to avoid redundant API calls
3. **Use environment variables** for all sensitive credentials
4. **Implement rate limiting** when batch processing content
5. **Monitor video render times** and adjust composition complexity accordingly
6. **Test content output** in multiple languages before automation
7. **Set up error notifications** for failed pipeline executions
