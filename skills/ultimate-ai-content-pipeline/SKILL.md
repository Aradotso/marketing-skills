---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, auto-posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation from research to video
  - set up AI content pipeline with video generation
  - crawl news and generate content automatically
  - create multilingual content with AI research
  - generate videos from written content using Remotion
  - build automated marketing content workflow
  - scrape trending topics and create posts
  - auto-generate social media content and videos
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based automation system that handles the entire content creation workflow: from researching trending topics (TechCrunch, Twitter, LinkedIn), generating multilingual blog posts/scripts using Claude/OpenAI, to rendering videos automatically with Remotion. Perfect for marketers, content creators, and agencies looking to scale content production by 90%.

## What It Does

- **Auto-Research**: Crawls real-time data from major tech news sources and social platforms
- **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) in both English and Vietnamese
- **Voice/Tone Customization**: Adapts writing style for different audiences (expert, friendly, humorous)
- **Video Rendering**: Converts written content into videos/infographics optimized for Reels, TikTok, Shorts
- **Multi-Platform Publishing**: Auto-scheduling and posting capabilities

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Setup

Create a `.env.local` file:

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Key Architecture

```
src/
├── components/          # React UI components
├── lib/
│   ├── ai/             # AI service integrations (Claude, OpenAI)
│   ├── research/       # Web scraping and data collection
│   ├── video/          # Remotion video generation
│   └── utils/          # Helper functions
├── pages/
│   ├── api/            # API routes
│   └── *.tsx           # Next.js pages
└── remotion/           # Video templates
```

## Core Workflows

### 1. Research & Data Collection

```typescript
// lib/research/crawler.ts
import { fetchTechCrunchArticles, fetchTwitterTrends } from './sources';

interface ResearchData {
  topic: string;
  articles: Article[];
  trends: Trend[];
  insights: string[];
}

export async function conductResearch(keyword: string): Promise<ResearchData> {
  const [articles, trends] = await Promise.all([
    fetchTechCrunchArticles(keyword, { days: 1 }),
    fetchTwitterTrends(keyword, { limit: 20 })
  ]);

  const insights = await extractInsights(articles, trends);

  return {
    topic: keyword,
    articles,
    trends,
    insights
  };
}

// Usage in API route
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { conductResearch } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword } = req.query;
  
  if (!keyword || typeof keyword !== 'string') {
    return res.status(400).json({ error: 'Keyword required' });
  }

  try {
    const data = await conductResearch(keyword);
    res.status(200).json(data);
  } catch (error) {
    res.status(500).json({ error: 'Research failed' });
  }
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchData;
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);

  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
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
  } else {
    const completion = await openai.chat.completions.create({
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
      max_tokens: 4096
    });

    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(request: ContentRequest): string {
  const { topic, format, language, tone, researchData } = request;
  
  const dataContext = `
Research Data:
- Articles: ${researchData.articles.map(a => `${a.title}: ${a.summary}`).join('\n')}
- Trends: ${researchData.trends.map(t => t.name).join(', ')}
- Key Insights: ${researchData.insights.join('\n')}
`;

  return `
Create a ${format} article about "${topic}" in ${language === 'vi' ? 'Vietnamese' : 'English'}.
Tone: ${tone}
${dataContext}

Requirements:
- Use real data from the research above
- Include statistics and evidence
- Make it engaging and actionable
- Optimize for social media sharing
${format === 'toplist' ? '- Create a numbered list format' : ''}
${format === 'case-study' ? '- Include real examples and outcomes' : ''}
${format === 'how-to' ? '- Provide step-by-step instructions' : ''}
`;
}
```

### 3. Complete Pipeline Workflow

```typescript
// lib/pipeline/content-pipeline.ts
import { conductResearch } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { renderVideo } from '../video/renderer';
import { publishToSocial } from '../publish/social-publisher';

interface PipelineConfig {
  keyword: string;
  formats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>;
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  autoPublish: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  const researchData = await conductResearch(config.keyword);
  console.log(`Research complete: ${researchData.articles.length} articles found`);

  const results = [];

  // Step 2: Generate content for each format/language combination
  for (const format of config.formats) {
    for (const language of config.languages) {
      console.log(`Generating ${language} ${format}...`);
      
      const content = await generateContent({
        topic: config.keyword,
        format,
        language,
        tone: 'expert',
        researchData
      });

      const result = {
        format,
        language,
        content,
        videoUrl: null as string | null
      };

      // Step 3: Generate video if requested
      if (config.generateVideo) {
        console.log(`Rendering video for ${language} ${format}...`);
        result.videoUrl = await renderVideo({
          content,
          language,
          format
        });
      }

      // Step 4: Auto-publish if enabled
      if (config.autoPublish) {
        await publishToSocial({
          content,
          videoUrl: result.videoUrl,
          platform: 'facebook' // or other platforms
        });
      }

      results.push(result);
    }
  }

  return {
    keyword: config.keyword,
    researchData,
    results
  };
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  language: 'en' | 'vi';
  format: string;
}

export async function renderVideo(config: VideoConfig): Promise<string> {
  const { content, language, format } = config;

  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo', // Composition ID
    inputProps: {
      content,
      language,
      format
    },
  });

  // Output path
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${format}.mp4`
  );

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      language,
      format
    },
  });

  return `/videos/${path.basename(outputLocation)}`;
}
```

```tsx
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  content: string;
  language: 'en' | 'vi';
  format: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  content,
  language,
  format
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Extract key points from content
  const keyPoints = content.split('\n').filter(line => 
    line.trim().length > 0 && line.trim().length < 100
  ).slice(0, 5);

  const currentPoint = Math.floor(frame / (fps * 3)) % keyPoints.length;
  const opacity = Math.min(1, (frame % (fps * 3)) / 30);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div
        style={{
          fontSize: 48,
          color: 'white',
          textAlign: 'center',
          fontWeight: 'bold',
          opacity,
        }}
      >
        {keyPoints[currentPoint]}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Frontend Integration

```tsx
// pages/index.tsx
import { useState } from 'react';
import type { NextPage } from 'next';

const Home: NextPage = () => {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          formats: ['toplist', 'how-to'],
          languages: ['en', 'vi'],
          generateVideo: true,
          autoPublish: false
        })
      });

      const data = await response.json();
      setResults(data);
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="container">
      <h1>Ultimate AI Content Pipeline</h1>
      
      <div className="input-section">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI Marketing)"
          disabled={loading}
        />
        <button onClick={handleGenerate} disabled={loading || !keyword}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {results && (
        <div className="results">
          <h2>Generated Content</h2>
          {results.results.map((result, idx) => (
            <div key={idx} className="result-item">
              <h3>{result.language.toUpperCase()} - {result.format}</h3>
              <div className="content">{result.content}</div>
              {result.videoUrl && (
                <video src={result.videoUrl} controls width="100%" />
              )}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default Home;
```

## API Routes

### POST /api/pipeline

Run the complete content generation pipeline.

**Request Body:**
```json
{
  "keyword": "AI Marketing Trends 2024",
  "formats": ["toplist", "case-study"],
  "languages": ["en", "vi"],
  "generateVideo": true,
  "autoPublish": false
}
```

**Response:**
```json
{
  "keyword": "AI Marketing Trends 2024",
  "researchData": {
    "articles": [...],
    "trends": [...],
    "insights": [...]
  },
  "results": [
    {
      "format": "toplist",
      "language": "en",
      "content": "...",
      "videoUrl": "/videos/1234567890-toplist.mp4"
    }
  ]
}
```

### GET /api/research?keyword={keyword}

Fetch research data only.

### POST /api/generate

Generate content without running full pipeline.

## Configuration

### Content Formats

Available formats with their characteristics:

- **toplist**: Numbered lists, data-driven, scannable
- **pov**: Opinion pieces, thought leadership
- **case-study**: Real examples with outcomes and analysis
- **how-to**: Step-by-step tutorials and guides

### Tone Options

- **expert**: Professional, authoritative, data-backed
- **friendly**: Conversational, approachable, engaging
- **humorous**: Light-hearted, entertaining, relatable

### Video Settings

Configure in `remotion/config.ts`:

```typescript
export const VIDEO_CONFIG = {
  fps: 30,
  durationInFrames: 300, // 10 seconds at 30fps
  width: 1080,
  height: 1920, // 9:16 for Reels/TikTok/Shorts
  backgroundColor: '#1a1a1a'
};
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultilingualContent(keyword: string) {
  const researchData = await conductResearch(keyword);
  
  const [enContent, viContent] = await Promise.all([
    generateContent({
      topic: keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      topic: keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData
    })
  ]);

  return { en: enContent, vi: viContent };
}
```

### Batch Processing Multiple Keywords

```typescript
async function batchProcessKeywords(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        formats: ['toplist'],
        languages: ['en', 'vi'],
        generateVideo: true,
        autoPublish: false
      })
    )
  );

  return results.map((result, idx) => ({
    keyword: keywords[idx],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null
  }));
}
```

## Troubleshooting

### API Rate Limits

If you hit rate limits:

```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  retries = 3,
  delay = 2000
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (retries > 0 && error.status === 429) {
      await new Promise(resolve => setTimeout(resolve, delay));
      return withRateLimit(fn, retries - 1, delay * 2);
    }
    throw error;
  }
}
```

### Video Rendering Memory Issues

For large content, split into smaller chunks:

```typescript
// Adjust Remotion concurrency
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  concurrency: 1, // Reduce from default
  inputProps: { content, language, format },
});
```

### Claude/OpenAI Token Limits

Truncate research data if needed:

```typescript
function truncateResearchData(data: ResearchData, maxTokens = 3000): ResearchData {
  return {
    ...data,
    articles: data.articles.slice(0, 5),
    insights: data.insights.slice(0, 10)
  };
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in project root and restart dev server:

```bash
# Verify environment
npm run dev -- --debug
```

## Production Deployment

```bash
# Build for production
npm run build

# Start production server
npm start

# Or deploy to Vercel
vercel deploy --prod
```

Remember to set all environment variables in your hosting platform (Vercel, Railway, etc.).
