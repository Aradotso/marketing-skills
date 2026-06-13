---
name: ultimate-ai-content-pipeline
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing
triggers:
  - how do I generate automated content with AI
  - set up the marketing content pipeline
  - create videos from text automatically
  - research and write blog posts with AI
  - automate content creation workflow
  - generate multilingual content with Claude
  - create social media videos with Remotion
  - build an AI content automation system
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the Ultimate AI Content Pipeline, a comprehensive TypeScript-based content automation system that handles research, scriptwriting, content generation, and video rendering using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** news and trends from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports multilingual** output (English and Vietnamese)
- **Renders videos** automatically using Remotion for social media platforms
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
node >= 18.x
npm or yarn
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

### Environment Configuration

Edit `.env` and configure the following:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Key Features & API Usage

### 1. Content Research Module

The research module crawls and analyzes content from multiple sources:

```typescript
import { researchContent } from './services/research';

// Research a topic
const research = await researchContent({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

// Returns structured data
console.log(research.insights);
console.log(research.trends);
console.log(research.dataPoints);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from './services/content-generator';

const content = await generateContent({
  topic: 'AI Content Automation',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  language: 'en', // 'en' | 'vi'
  aiProvider: 'claude', // 'claude' | 'openai'
  researchData: research // Optional: include research data
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Multi-language Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
import { generateMultilingualContent } from './services/content-generator';

const multiContent = await generateMultilingualContent({
  topic: 'Marketing Automation',
  format: 'how-to',
  tone: 'friendly'
});

console.log(multiContent.en.title);
console.log(multiContent.vi.title);
```

### 4. Video Generation with Remotion

Automatically render videos from content:

```typescript
import { renderVideo } from './services/video-renderer';

const video = await renderVideo({
  content: content,
  template: 'reels', // 'reels' | 'tiktok' | 'shorts'
  aspectRatio: '9:16',
  duration: 30, // seconds
  style: {
    theme: 'modern',
    colors: ['#FF6B6B', '#4ECDC4'],
    font: 'Inter'
  }
});

// video.url contains the rendered video URL
console.log(video.url);
console.log(video.thumbnail);
```

## Common Patterns

### Complete Content Pipeline

Full workflow from research to video:

```typescript
import { 
  researchContent, 
  generateContent, 
  renderVideo,
  publishContent 
} from './services';

async function createContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchContent({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: keyword,
      format: 'toplist',
      tone: 'professional',
      language: 'en',
      aiProvider: 'claude',
      researchData: research
    });

    // Step 3: Render video
    console.log('🎬 Rendering video...');
    const video = await renderVideo({
      content,
      template: 'reels',
      aspectRatio: '9:16'
    });

    // Step 4: Publish (optional)
    console.log('📤 Publishing...');
    const published = await publishContent({
      content,
      video,
      platforms: ['facebook', 'instagram']
    });

    return {
      content,
      video,
      published
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Use the pipeline
createContentPipeline('AI Marketing Tools 2024')
  .then(result => console.log('✅ Pipeline complete!', result));
```

### Batch Content Generation

Generate multiple content pieces at once:

```typescript
import { generateBatchContent } from './services/content-generator';

const topics = [
  'AI Marketing Automation',
  'Social Media Trends 2024',
  'Content Creation Tools'
];

const batchResults = await generateBatchContent({
  topics,
  format: 'toplist',
  tone: 'professional',
  languages: ['en', 'vi'],
  aiProvider: 'claude'
});

batchResults.forEach((result, index) => {
  console.log(`Topic ${index + 1}:`, result.en.title);
  console.log(`Vietnamese:`, result.vi.title);
});
```

### Custom Content Format

Define custom content templates:

```typescript
import { generateContent } from './services/content-generator';

const customContent = await generateContent({
  topic: 'Email Marketing Best Practices',
  format: 'custom',
  customPrompt: `
    Create a detailed guide with:
    - Executive summary
    - 5 actionable tips
    - Real-world examples
    - Metrics and KPIs
    - Tools recommendations
  `,
  tone: 'expert',
  language: 'en',
  aiProvider: 'openai'
});
```

## API Integration Examples

### Using Claude (Anthropic)

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### Using OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are a content marketing expert.' },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### Remotion Video Component

```typescript
import { Composition } from 'remotion';
import { ContentVideo } from './components/ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Content Title',
          points: [
            'Point 1',
            'Point 2',
            'Point 3'
          ],
          theme: 'modern'
        }}
      />
    </>
  );
};
```

## Configuration

### Content Format Templates

Available format templates in `config/formats.ts`:

```typescript
export const contentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10
  },
  pov: {
    structure: ['hook', 'perspective', 'arguments', 'conclusion'],
    tone: ['opinionated', 'thought-provoking']
  },
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true
  },
  howTo: {
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    minSteps: 3,
    maxSteps: 10
  }
};
```

### Video Templates

Configure video templates in `config/video.ts`:

```typescript
export const videoTemplates = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 30,
    aspectRatio: '9:16'
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60,
    aspectRatio: '9:16'
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60,
    aspectRatio: '9:16'
  }
};
```

## CLI Commands

If the project includes CLI tools:

```bash
# Generate content from command line
npm run generate -- --topic "AI Marketing" --format toplist --lang en

# Run research only
npm run research -- --keyword "content automation" --sources techcrunch,twitter

# Render video
npm run render -- --input ./content.json --template reels --output ./video.mp4

# Run full pipeline
npm run pipeline -- --keyword "AI tools" --platforms facebook,instagram
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      return await generateContent({ topic: prompt });
    },
    {
      retries: 3,
      minTimeout: 1000,
      maxTimeout: 5000,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`);
      }
    }
  );
}
```

### Memory Issues with Video Rendering

```typescript
// Use streaming for large video renders
import { renderMedia } from '@remotion/renderer';

await renderMedia({
  composition: 'ContentVideo',
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: './output.mp4',
  chromiumOptions: {
    // Reduce memory usage
    headless: true,
    args: [
      '--disable-dev-shm-usage',
      '--no-sandbox'
    ]
  }
});
```

### Content Quality Issues

```typescript
// Add content validation
function validateContent(content: GeneratedContent): boolean {
  const checks = {
    hasTitle: content.title && content.title.length > 10,
    hasBody: content.body && content.body.length > 100,
    hasStructure: content.sections && content.sections.length >= 3,
    hasMetadata: content.metadata && content.metadata.keywords
  };

  const passed = Object.values(checks).every(check => check);
  
  if (!passed) {
    console.error('Content validation failed:', checks);
  }
  
  return passed;
}
```

### Research Data Not Found

```typescript
// Fallback to alternative sources
async function robustResearch(keyword: string) {
  const sources = ['techcrunch', 'twitter', 'linkedin', 'reddit'];
  
  for (const source of sources) {
    try {
      const data = await researchContent({
        keyword,
        sources: [source],
        timeframe: '48h' // Extend timeframe if needed
      });
      
      if (data.insights.length > 0) {
        return data;
      }
    } catch (error) {
      console.warn(`Source ${source} failed, trying next...`);
      continue;
    }
  }
  
  throw new Error('No research data available from any source');
}
```

## Best Practices

1. **Always validate research data** before content generation
2. **Use environment variables** for all API keys
3. **Implement rate limiting** to avoid API quota issues
4. **Cache research results** to reduce API calls
5. **Test video templates** with different content lengths
6. **Monitor AI token usage** to manage costs
7. **Version control your prompts** for reproducibility
