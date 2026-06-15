---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I generate automated content with AI pipeline
  - set up content automation from research to video
  - use Claude and OpenAI for content generation
  - automate content creation with remotion video
  - build AI content workflow with research crawling
  - generate multilingual content with AI pipeline
  - create automated social media content pipeline
  - set up AI-powered marketing content system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from researching trending topics, generating multilingual articles, to rendering videos automatically. It leverages Claude 3, OpenAI, and Remotion to transform keywords into publish-ready content.

## What This Project Does

- **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multilingual Support**: Generates content in both English and Vietnamese simultaneously
- **Video Rendering**: Automatically creates infographics and short videos from articles using Remotion
- **Multi-Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

## Environment Configuration

Create a `.env` file in the root directory:

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── api/           # API integrations
│   ├── components/    # React components
│   ├── lib/           # Core utilities
│   ├── pipelines/     # Content pipelines
│   └── remotion/      # Video rendering
├── public/
└── .env
```

## Key APIs and Usage

### 1. Content Research Pipeline

```typescript
import { ResearchService } from '@/lib/research';

const research = new ResearchService({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch trending topics from the last 24 hours
const trendingData = await research.fetchTrending({
  keyword: 'artificial intelligence',
  timeframe: '24h',
  maxResults: 50
});

// Analyze and extract insights
const insights = await research.analyzeInsights(trendingData);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string) {
  const message = await anthropic.messages.create({
    model: "claude-3-opus-20240229",
    max_tokens: 4096,
    messages: [{
      role: "user",
      content: `Generate a ${format} article about ${topic} based on these insights: ${JSON.stringify(insights)}`
    }],
    system: "You are an expert content writer specializing in marketing and tech."
  });

  return message.content[0].text;
}

// Generate different formats
const toplistArticle = await generateContent('AI trends 2024', 'toplist');
const caseStudy = await generateContent('AI trends 2024', 'case-study');
```

### 3. Multilingual Content Generation

```typescript
interface ContentOptions {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: string[];
}

async function generateMultilingualContent(options: ContentOptions) {
  const results = {};
  
  for (const lang of options.languages) {
    const prompt = `Write a ${options.format} article in ${lang} about ${options.topic} with a ${options.tone} tone.`;
    
    const content = await anthropic.messages.create({
      model: "claude-3-opus-20240229",
      max_tokens: 4096,
      messages: [{ role: "user", content: prompt }]
    });
    
    results[lang] = content.content[0].text;
  }
  
  return results;
}

// Usage
const multilingualContent = await generateMultilingualContent({
  topic: 'AI Marketing Automation',
  format: 'how-to',
  tone: 'friendly',
  languages: ['en', 'vi']
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(article: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: article.title,
      content: article.content,
      insights: article.insights
    }
  });

  // Render video
  const outputLocation = path.join(process.cwd(), 'output', 'video.mp4');
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.inputProps,
  });

  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  insights: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  insights
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ padding: 60, opacity }}>
        <h1 style={{ color: 'white', fontSize: 64, marginBottom: 40 }}>
          {title}
        </h1>
        <ul style={{ color: '#00ff88', fontSize: 32 }}>
          {insights.map((insight, i) => (
            <li key={i} style={{ marginBottom: 20 }}>{insight}</li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Integration

```typescript
import { ResearchService } from '@/lib/research';
import { ContentGenerator } from '@/lib/content';
import { VideoRenderer } from '@/lib/video';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  const research = new ResearchService({
    apiKey: process.env.RAPIDAPI_KEY
  });
  const trendingData = await research.fetchTrending({ keyword });
  const insights = await research.analyzeInsights(trendingData);

  // Step 2: Generate Content
  const generator = new ContentGenerator({
    claudeKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY
  });
  
  const article = await generator.create({
    topic: keyword,
    format: 'toplist',
    tone: 'expert',
    insights: insights,
    languages: ['en', 'vi']
  });

  // Step 3: Render Video
  const renderer = new VideoRenderer();
  const videoPath = await renderer.render({
    title: article.en.title,
    content: article.en.content,
    insights: insights.slice(0, 5),
    platform: 'tiktok' // or 'reels', 'shorts'
  });

  return {
    article,
    videoPath,
    insights
  };
}

// Execute pipeline
const result = await runContentPipeline('AI marketing automation');
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages } = req.body;

  try {
    const result = await runContentPipeline(keyword);
    
    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

### Usage from Frontend

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi']
      })
    });

    const data = await response.json();
    setResult(data.data);
    setLoading(false);
  };

  return (
    <div>
      <input
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && (
        <div>
          <h2>{result.article.en.title}</h2>
          <video src={result.videoPath} controls />
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Custom Content Formats

```typescript
const formatTemplates = {
  toplist: {
    structure: 'numbered list with detailed explanations',
    tone: 'authoritative',
    length: 'medium'
  },
  pov: {
    structure: 'personal perspective with examples',
    tone: 'conversational',
    length: 'short'
  },
  caseStudy: {
    structure: 'problem-solution-results',
    tone: 'analytical',
    length: 'long'
  },
  howTo: {
    structure: 'step-by-step guide',
    tone: 'instructional',
    length: 'medium'
  }
};

async function generateWithTemplate(topic: string, formatKey: string) {
  const template = formatTemplates[formatKey];
  
  const prompt = `Write a ${template.structure} article about ${topic}.
    Tone: ${template.tone}
    Target length: ${template.length}
    Include data-backed insights and real examples.`;
  
  return await generateContent(prompt);
}
```

### Batch Processing

```typescript
async function processBatchKeywords(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const content = await runContentPipeline(keyword);
    results.push({
      keyword,
      ...content
    });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (development)
npm run remotion:preview

# Render Remotion video (production)
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function fetchWithRetry(fn: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
    }
  }
}

const data = await fetchWithRetry(() => research.fetchTrending({ keyword }));
```

### Video Rendering Memory Issues

```typescript
// Configure Remotion for lower memory usage
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  concurrency: 1, // Reduce concurrency
  quality: 80, // Lower quality if needed
  outputLocation,
});
```

### Multilingual Content Quality

```typescript
// Add language-specific validation
function validateContent(content: string, language: string): boolean {
  const minLengths = { en: 500, vi: 300 };
  return content.length >= minLengths[language];
}

// Regenerate if validation fails
let content = await generateContent(topic, lang);
if (!validateContent(content, lang)) {
  content = await generateContent(topic, lang); // Retry
}
```

### Claude API Errors

```typescript
// Handle Claude-specific errors
try {
  const message = await anthropic.messages.create({...});
} catch (error) {
  if (error.status === 529) {
    // Overloaded, retry after delay
    await new Promise(r => setTimeout(r, 5000));
  } else if (error.status === 400) {
    // Invalid request, check prompt length
    console.error('Prompt too long or invalid format');
  }
  throw error;
}
```
