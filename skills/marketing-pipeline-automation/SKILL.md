---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, social media posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate my content creation workflow
  - set up AI marketing pipeline
  - generate videos from content automatically
  - create social media posts with AI
  - research and write content with Claude
  - build automated content generation system
  - set up remotion video rendering pipeline
  - create AI-powered content workflow
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers use the Ultimate AI Content Pipeline - an end-to-end automated content creation system that handles research, content generation, social media posting, and video rendering.

## What This Project Does

The Marketing Pipeline automation system provides:

- **Auto-Research**: Crawls and analyzes fresh data from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates multi-format content (lists, POV articles, case studies, how-tos) in multiple languages using Claude 3 and OpenAI
- **Video Generation**: Automatically renders videos and infographics from written content using Remotion
- **Social Media Integration**: Auto-posts to Facebook Pages and other platforms
- **Multi-language Support**: Generates content in English and Vietnamese with customizable tone

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

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Service Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Social Media
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
FACEBOOK_PAGE_ID=your_page_id

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Rendering
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── services/
│   │   ├── research/       # Content crawling & research
│   │   ├── ai/             # Claude/OpenAI integrations
│   │   ├── video/          # Remotion video generation
│   │   └── social/         # Social media posting
│   ├── lib/
│   │   ├── prompts/        # AI prompt templates
│   │   └── utils/          # Helper functions
│   └── app/                # Next.js pages
├── remotion/               # Video templates
└── public/                 # Static assets
```

## Core API Usage

### Research & Content Crawling

```typescript
import { researchService } from '@/services/research';

// Crawl recent news on a topic
async function gatherResearch(keyword: string) {
  const research = await researchService.crawl({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return research;
}

// Example usage
const aiResearch = await gatherResearch('artificial intelligence trends');
console.log(aiResearch.insights); // Extracted insights
console.log(aiResearch.dataPoints); // Statistical data
```

### AI Content Generation with Claude

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string, research: any) {
  const systemPrompt = `You are an expert content creator specializing in ${format} format.
Use the provided research data to create engaging, data-backed content.`;

  const userPrompt = `Topic: ${topic}
Format: ${format}
Research Data: ${JSON.stringify(research)}

Create a compelling article with:
- Attention-grabbing headline
- Data-driven insights
- Actionable takeaways
- SEO-optimized structure`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userPrompt
    }]
  });

  return message.content[0].text;
}

// Generate toplist article
const content = await generateContent(
  'Top AI Tools 2024',
  'toplist',
  aiResearch
);
```

### Multi-Language Content Generation

```typescript
interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: string[];
}

async function generateMultiLanguageContent(request: ContentRequest) {
  const results = {};
  
  for (const lang of request.languages) {
    const prompt = `Create ${request.format} content about ${request.topic} in ${lang}.
Tone: ${request.tone}
Include cultural nuances appropriate for ${lang} speakers.`;

    const content = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    results[lang] = content.content[0].text;
  }
  
  return results;
}

// Generate English and Vietnamese versions
const bilingualContent = await generateMultiLanguageContent({
  topic: 'AI Marketing Automation',
  format: 'how-to',
  tone: 'expert',
  languages: ['en', 'vi']
});
```

### Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

interface VideoConfig {
  content: string;
  title: string;
  duration: number;
  format: '9:16' | '16:9' | '1:1'; // Reels, YouTube, Instagram
}

async function generateVideo(config: VideoConfig) {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      title: config.title,
    },
  });

  // Render video
  const outputLocation = `./output/${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content,
      title: config.title,
    },
  });

  return outputLocation;
}

// Create Reels-format video
const videoPath = await generateVideo({
  content: bilingualContent.en,
  title: 'AI Marketing Tips',
  duration: 60,
  format: '9:16'
});
```

### Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface Props {
  content: string;
  title: string;
}

export const ContentVideo: React.FC<Props> = ({ content, title }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  // Split content into key points
  const points = content.split('\n').filter(p => p.trim());

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div style={{ 
        padding: 60, 
        color: 'white',
        opacity 
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>{title}</h1>
        {points.slice(0, 3).map((point, i) => (
          <div
            key={i}
            style={{
              fontSize: 36,
              marginBottom: 30,
              opacity: interpolate(
                frame,
                [30 + i * 30, 60 + i * 30],
                [0, 1],
                { extrapolateRight: 'clamp' }
              ),
            }}
          >
            • {point}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

### Social Media Auto-Posting

```typescript
import axios from 'axios';

interface FacebookPost {
  message: string;
  link?: string;
  videoPath?: string;
}

async function postToFacebook(post: FacebookPost) {
  const pageId = process.env.FACEBOOK_PAGE_ID;
  const accessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;

  if (post.videoPath) {
    // Upload video
    const formData = new FormData();
    formData.append('file', fs.createReadStream(post.videoPath));
    formData.append('description', post.message);
    formData.append('access_token', accessToken);

    const response = await axios.post(
      `https://graph.facebook.com/v18.0/${pageId}/videos`,
      formData
    );

    return response.data;
  } else {
    // Post text/link
    const response = await axios.post(
      `https://graph.facebook.com/v18.0/${pageId}/feed`,
      {
        message: post.message,
        link: post.link,
        access_token: accessToken
      }
    );

    return response.data;
  }
}

// Post generated content with video
await postToFacebook({
  message: bilingualContent.en,
  videoPath
});
```

## Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/services/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline();

  try {
    // 1. Research phase
    console.log('🔍 Researching...');
    const research = await pipeline.research({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h'
    });

    // 2. Content generation
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      topic: keyword,
      research,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert'
    });

    // 3. Video generation
    console.log('🎬 Creating video...');
    const videos = await pipeline.generateVideos({
      content: content.en,
      formats: ['9:16', '1:1'], // Reels and Instagram
      duration: 60
    });

    // 4. Auto-post to social media
    console.log('📤 Posting to social media...');
    await pipeline.publish({
      platforms: ['facebook', 'twitter'],
      content,
      videos,
      schedule: 'immediate' // or provide Date
    });

    return {
      research,
      content,
      videos,
      status: 'published'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute full pipeline
const result = await runFullPipeline('AI content creation tools 2024');
```

## Development Server

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio for video preview
npm run remotion
```

## Common Patterns

### Custom AI Prompts

```typescript
// lib/prompts/templates.ts
export const PROMPT_TEMPLATES = {
  toplist: (topic: string, data: any) => `
Create a comprehensive toplist article about "${topic}".

Requirements:
- Include 7-10 items
- Data-driven rankings based on: ${JSON.stringify(data.metrics)}
- Each item needs: name, description, pros/cons, price
- Add expert commentary
- Include call-to-action

Recent data: ${JSON.stringify(data.insights)}
`,
  
  caseStudy: (company: string, data: any) => `
Write an in-depth case study about ${company}.

Structure:
1. Challenge/Problem
2. Solution approach
3. Implementation details
4. Results (use these metrics: ${JSON.stringify(data.metrics)})
5. Key takeaways

Make it actionable for readers.
`
};

// Use custom prompts
import { PROMPT_TEMPLATES } from '@/lib/prompts/templates';

const content = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  messages: [{
    role: 'user',
    content: PROMPT_TEMPLATES.toplist('Best CRM Software', research)
  }]
});
```

### Scheduled Content Publishing

```typescript
interface ScheduledPost {
  content: string;
  publishAt: Date;
  platforms: string[];
  videoPath?: string;
}

async function schedulePost(post: ScheduledPost) {
  // Store in database for scheduled execution
  await db.scheduledPosts.create({
    data: {
      content: post.content,
      publishAt: post.publishAt,
      platforms: post.platforms,
      videoPath: post.videoPath,
      status: 'pending'
    }
  });

  // Set up cron job or use service like Vercel Cron
  return { scheduled: true, id: post.id };
}

// Schedule for tomorrow at 9 AM
const tomorrow9AM = new Date();
tomorrow9AM.setDate(tomorrow9AM.getDate() + 1);
tomorrow9AM.setHours(9, 0, 0, 0);

await schedulePost({
  content: bilingualContent.en,
  publishAt: tomorrow9AM,
  platforms: ['facebook', 'twitter'],
  videoPath
});
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

const contents = await Promise.all(
  topics.map(topic =>
    limit(() => generateContent(topic, 'toplist', research))
  )
);
```

### Video Rendering Memory Issues

```typescript
// Use smaller chunks for long content
function chunkContent(content: string, maxLength: number = 500) {
  const sentences = content.split('. ');
  const chunks = [];
  let currentChunk = '';

  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > maxLength) {
      chunks.push(currentChunk);
      currentChunk = sentence;
    } else {
      currentChunk += sentence + '. ';
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}

// Render multiple shorter videos instead of one long video
const chunks = chunkContent(content);
const videos = await Promise.all(
  chunks.map((chunk, i) =>
    generateVideo({
      content: chunk,
      title: `Part ${i + 1}`,
      duration: 30,
      format: '9:16'
    })
  )
);
```

### Claude/OpenAI Timeout Handling

```typescript
async function generateWithRetry(
  prompt: string,
  maxRetries: number = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const message = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
        timeout: 60000 // 60 seconds
      });

      return message.content[0].text;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      // Exponential backoff
      await new Promise(resolve =>
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
}
```

### Environment Variable Validation

```typescript
// lib/config.ts
export function validateConfig() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'FACEBOOK_PAGE_ACCESS_TOKEN'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env file.'
    );
  }
}

// Call at startup
validateConfig();
```

This skill equips AI agents to help developers implement comprehensive automated content marketing pipelines using Claude, OpenAI, Remotion, and social media APIs.
