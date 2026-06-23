---
name: ultimate-ai-content-pipeline
description: Automated content creation system with AI research, multi-format writing, and video generation using Claude, OpenAI, and Remotion
triggers:
  - generate automated content with ai research
  - create video content from written articles
  - scrape and analyze news for content generation
  - build ai powered marketing content pipeline
  - automate content creation with claude and openai
  - render videos automatically from blog posts
  - create multilingual content with ai
  - setup automated content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from research and article generation to video rendering. The pipeline leverages Claude 3, OpenAI, web scraping, and Remotion for video generation.

## What This Project Does

The Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates AI content**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
- **Multi-language support**: Generates content in English and Vietnamese simultaneously
- **Video rendering**: Automatically creates infographics and short videos from articles using Remotion
- **Platform optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Setup environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Application
NODE_ENV=development
PORT=3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── api/           # API routes and endpoints
│   ├── components/    # React components
│   ├── lib/          
│   │   ├── ai/       # AI integration (Claude, OpenAI)
│   │   ├── scraper/  # Web scraping modules
│   │   └── video/    # Remotion video generation
│   ├── pages/        # Next.js pages
│   └── utils/        # Utility functions
├── remotion/         # Video templates
└── public/           # Static assets
```

## Core Modules

### 1. Research & Data Collection

```typescript
// src/lib/scraper/newsCollector.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  category: string;
  date: Date;
}

export async function collectRecentNews(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<NewsSource[]> {
  const sources = [
    'techcrunch.com',
    'a16z.com/blog',
    'twitter.com/search'
  ];
  
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/scraper/v1/search`,
        {
          params: { q: keyword, source, timeframe },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
            'X-RapidAPI-Host': 'web-scraper.p.rapidapi.com'
          }
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to fetch from ${source}:`, error);
    }
  }
  
  return results;
}

export async function extractInsights(articles: NewsSource[]): Promise<string> {
  // Process and summarize key insights
  const summaries = articles.map(a => a.content).join('\n\n');
  return summaries;
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string;
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateWithClaude(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    const message = await this.anthropic.messages.create({
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
  
  async generateWithOpenAI(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'system',
        content: 'You are an expert content creator and marketer.'
      }, {
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 3000
    });
    
    return completion.choices[0].message.content || '';
  }
  
  private buildPrompt(request: ContentRequest): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear rankings',
      'pov': 'Write from a personal perspective with unique insights',
      'case-study': 'Analyze with data, examples, and actionable takeaways',
      'how-to': 'Provide step-by-step instructions with examples'
    };
    
    const toneInstructions = {
      'expert': 'Use professional, authoritative language',
      'friendly': 'Use conversational, approachable tone',
      'humorous': 'Include wit and engaging storytelling'
    };
    
    return `
Create a ${request.format} article about "${request.keyword}" in ${request.language}.

Tone: ${toneInstructions[request.tone]}
Format: ${formatInstructions[request.format]}

Research Data:
${request.researchData}

Requirements:
- Use the latest data from the research
- Include specific examples and statistics
- Add actionable insights
- Optimize for SEO
${request.language === 'vi' ? '- Write in Vietnamese' : '- Write in English'}
`;
  }
  
  async generateBilingual(request: ContentRequest) {
    const [english, vietnamese] = await Promise.all([
      this.generateWithClaude({ ...request, language: 'en' }),
      this.generateWithClaude({ ...request, language: 'vi' })
    ]);
    
    return { english, vietnamese };
  }
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'infographic' | 'text-overlay' | 'animated';
  platform: 'reels' | 'tiktok' | 'shorts';
}

export class VideoRenderer {
  async renderVideo(config: VideoConfig): Promise<string> {
    const bundleLocation = await bundle(
      path.join(process.cwd(), 'remotion/index.ts')
    );
    
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: config.style,
      inputProps: {
        title: config.title,
        content: config.content,
        platform: config.platform
      }
    });
    
    const outputPath = path.join(
      process.cwd(), 
      'public', 
      'videos',
      `${Date.now()}-${config.platform}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: composition.defaultProps
    });
    
    return outputPath;
  }
  
  getPlatformDimensions(platform: string) {
    const dimensions = {
      'reels': { width: 1080, height: 1920 },
      'tiktok': { width: 1080, height: 1920 },
      'shorts': { width: 1080, height: 1920 },
      'youtube': { width: 1920, height: 1080 }
    };
    
    return dimensions[platform] || dimensions.youtube;
  }
}
```

```tsx
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, spring } from 'remotion';

export const InfographicVideo: React.FC<{
  title: string;
  content: string[];
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = spring({
    frame,
    fps,
    from: 0,
    to: 1,
    durationInFrames: 30
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: '60px',
        color: 'white'
      }}>
        <h1 style={{
          fontSize: '72px',
          fontWeight: 'bold',
          textAlign: 'center',
          opacity: titleOpacity,
          marginBottom: '60px'
        }}>
          {title}
        </h1>
        
        {content.map((item, index) => {
          const itemOpacity = spring({
            frame: frame - (30 * (index + 1)),
            fps,
            from: 0,
            to: 1,
            durationInFrames: 30
          });
          
          return (
            <div key={index} style={{
              fontSize: '48px',
              margin: '20px 0',
              opacity: itemOpacity,
              transform: `translateY(${(1 - itemOpacity) * 50}px)`
            }}>
              {item}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { ContentGenerator } from '../ai/contentGenerator';
import { collectRecentNews, extractInsights } from '../scraper/newsCollector';
import { VideoRenderer } from '../video/renderer';

export class ContentPipeline {
  private contentGen: ContentGenerator;
  private videoRenderer: VideoRenderer;
  
  constructor() {
    this.contentGen = new ContentGenerator();
    this.videoRenderer = new VideoRenderer();
  }
  
  async executeFullPipeline(params: {
    keyword: string;
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    generateVideo: boolean;
  }) {
    console.log(`Starting pipeline for keyword: ${params.keyword}`);
    
    // Step 1: Research
    console.log('Step 1: Collecting research data...');
    const articles = await collectRecentNews(params.keyword, '24h');
    const insights = await extractInsights(articles);
    
    // Step 2: Generate Content
    console.log('Step 2: Generating content...');
    const content = await this.contentGen.generateBilingual({
      keyword: params.keyword,
      format: params.format,
      tone: 'expert',
      language: 'en',
      researchData: insights
    });
    
    // Step 3: Generate Video (optional)
    let videoPath: string | null = null;
    if (params.generateVideo) {
      console.log('Step 3: Rendering video...');
      
      const bulletPoints = this.extractBulletPoints(content.english);
      
      videoPath = await this.videoRenderer.renderVideo({
        title: params.keyword,
        content: bulletPoints,
        style: 'infographic',
        platform: 'reels'
      });
    }
    
    return {
      articles: articles.length,
      content: {
        english: content.english,
        vietnamese: content.vietnamese
      },
      video: videoPath,
      timestamp: new Date().toISOString()
    };
  }
  
  private extractBulletPoints(content: string): string[] {
    // Extract key points from content for video
    const lines = content.split('\n');
    const bullets = lines
      .filter(line => line.match(/^[\d\-\*]/))
      .slice(0, 5)
      .map(line => line.replace(/^[\d\-\*\.]\s*/, ''));
    
    return bullets;
  }
}
```

## API Routes (Next.js)

```typescript
// src/pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, generateVideo } = req.body;
  
  if (!keyword || !format) {
    return res.status(400).json({ error: 'Missing required fields' });
  }
  
  try {
    const pipeline = new ContentPipeline();
    const result = await pipeline.executeFullPipeline({
      keyword,
      format,
      generateVideo: generateVideo || false
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      details: error.message
    });
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

# Render video only
npm run remotion:render
```

## CLI Usage (if available)

```bash
# Generate content from CLI
npm run generate -- --keyword "AI Marketing" --format toplist

# Generate with video
npm run generate -- --keyword "AI Marketing" --format toplist --video

# Specify language
npm run generate -- --keyword "AI Marketing" --format toplist --lang vi
```

## Common Patterns

### Pattern 1: Quick Article Generation

```typescript
import { ContentGenerator } from './lib/ai/contentGenerator';
import { collectRecentNews, extractInsights } from './lib/scraper/newsCollector';

async function quickArticle(topic: string) {
  const generator = new ContentGenerator();
  const news = await collectRecentNews(topic);
  const insights = await extractInsights(news);
  
  const article = await generator.generateWithClaude({
    keyword: topic,
    format: 'how-to',
    tone: 'friendly',
    language: 'en',
    researchData: insights
  });
  
  console.log(article);
}
```

### Pattern 2: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.all(
    keywords.map(keyword => 
      pipeline.executeFullPipeline({
        keyword,
        format: 'toplist',
        generateVideo: false
      })
    )
  );
  
  return results;
}
```

### Pattern 3: Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const pipeline = new ContentPipeline();
  
  const dailyTopics = ['AI trends', 'Marketing automation', 'Content strategy'];
  
  for (const topic of dailyTopics) {
    await pipeline.executeFullPipeline({
      keyword: topic,
      format: 'pov',
      generateVideo: true
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10 // limit each IP to 10 requests per windowMs
});

// Apply to API routes
app.use('/api/', limiter);
```

### Video Rendering Memory Issues

```typescript
// Adjust Remotion memory settings
export const renderVideo = async (config: VideoConfig) => {
  await renderMedia({
    // ... other config
    chromiumOptions: {
      headless: true,
      args: ['--no-sandbox', '--disable-dev-shm-usage']
    },
    // Limit concurrent rendering
    concurrency: 1
  });
};
```

### AI API Timeouts

```typescript
// Add timeout handling
const generateWithRetry = async (
  generator: ContentGenerator,
  request: ContentRequest,
  maxRetries = 3
) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await Promise.race([
        generator.generateWithClaude(request),
        new Promise((_, reject) => 
          setTimeout(() => reject(new Error('Timeout')), 30000)
        )
      ]);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
};
```

### Web Scraping Failures

```typescript
// Fallback mechanism
async function collectNewsWithFallback(keyword: string) {
  try {
    return await collectRecentNews(keyword, '24h');
  } catch (error) {
    console.warn('Primary scraper failed, using fallback');
    // Use cached data or alternative source
    return await getCachedNews(keyword);
  }
}
```

## Best Practices

1. **API Key Management**: Always use environment variables, never commit keys
2. **Error Handling**: Wrap all API calls in try-catch blocks
3. **Rate Limiting**: Implement delays between API calls to avoid rate limits
4. **Caching**: Cache research data to reduce API calls
5. **Video Optimization**: Render videos at lower quality for previews, high quality for final output
6. **Content Validation**: Validate AI-generated content before publishing
7. **Monitoring**: Log all pipeline executions for debugging

This skill provides comprehensive guidance for working with the Ultimate AI Content Pipeline project, enabling AI coding agents to effectively assist developers in implementing automated content creation workflows.
